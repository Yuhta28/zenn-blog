---
title: "OIDC認証でPulumi CloudとAWSを連携してみた"
emoji: "🌎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","aws","oidc"]
published: false
---

# 概要
2022年11月にPulumiはGitHubと連携することでCI/CDプラットフォームを自動構築してくれるPulumi DeploymentをPulumi Cloud上でリリースしました。
https://www.pulumi.com/blog/pulumi-deployments/

Pulumiで記述された構成ファイルが含まれるGitHubリポジトリと連携し、PRすることでPulumi Cloud上で`pulumi preview`(Terraformでいうところのplanコマンド)が実施され、マージすることで`pulumi up`(Terraformでいうところのapplyコマンド)が走り、GitHubフローにそったIaC開発が実現できました。

ただ当初はAWSのシークレットキーなどの秘文情報を変数化して構成ファイル内に渡さなければなりませんでした。
```ts:IAMユーザーの認証情報を渡す
const stack = await RemoteWorkspace.createOrSelectStack({
    stackName,
    url: "https://github.com/pulumi/examples.git",
    branch: "refs/heads/master",
    projectPath: project,
}, {
    envVars: {
        AWS_REGION:            "us-west-2",
        AWS_ACCESS_KEY_ID:     process.env.AWS_ACCESS_KEY_ID ?? "",
        AWS_SECRET_ACCESS_KEY: { secret: process.env.AWS_SECRET_ACCESS_KEY ?? "" },
        AWS_SESSION_TOKEN:     { secret: process.env.AWS_SESSION_TOKEN ?? "" },
    },
});

const upRes = await stack.up({ onOutput: console.log });
console.log(`url: ${upRes.outputs.websiteUrl.value}`);
```

専用のIAMユーザーを作成したらアクセスキーとシークレットキーの管理負担が重くなるところですが、2023年1月にPulumiはOpenID Connect(OIDC)との統合を開始しました。
https://www.pulumi.com/blog/oidc-blog/

これによりクレデンシャル情報を保存する必要性がなくなり、デプロイしたときだけPulumi側で一時的な認証情報を発行し、完了したら自動的に削除してくれるため最小限のアクセス管理ができます。
公式ドキュメントがわかりにくい部分がありましたので自作したCloudFormationテンプレートファイルに沿った手順でPulumi CloudとAWSとのOIDC連携を実現してみます。

# OIDC連携

早速ですがこちらのCloudFormationテンプレートファイルをデプロイすればPulumi Cloud向けのOIDCが作成されます。

```yaml: テンプレートファイル
AWSTemplateFormatVersion: '2010-09-09'
Description: "OIDC for Pulumi"

Parameters:
  OrgName:
    Type: String
    Description: the name of your Pulumi Organization
  ProjectName:
    Type: String
    Description: If you want to allow all projects, etner *. 

Resources:
  IAMOIDCProvider:
    Type: AWS::IAM::OIDCProvider
    Properties:
      Url: https://api.pulumi.com/oidc
      ClientIdList:
        - !Ref OrgName
      ThumbprintList:
        - 9e99a48a9960b14926bb7f3b02e22da2b0ab7280
  IAMRole:
    Type: AWS::IAM::Role
    Properties:
      Path: /
      RoleName: OpenID-PulumiCloud
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Federated: !Ref IAMOIDCProvider
            Action: sts:AssumeRoleWithWebIdentity
            Condition:
              StringLike:
                api.pulumi.com/oidc:sub: !Sub pulumi:deploy:org:${OrgName}:project:${ProjectName}:*
                api.pulumi.com/oidc:aud: !Ref OrgName
      MaxSessionDuration: 3600
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AdministratorAccess
```

# 参考文献
https://openid.net/foundation/