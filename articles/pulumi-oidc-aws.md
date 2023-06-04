---
title: "OIDC認証でPulumi CloudとAWSを連携してみた"
emoji: "🌎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","aws","oidc"]
published: publish
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

![](/images/pulumi-oidc-aws/image0.png)
*環境変数に値埋め込み*

専用のIAMユーザーを作成したらアクセスキーとシークレットキーを管理しなければなりませんが、2023年1月にPulumiはOpenID Connect(OIDC)[^1]との統合を開始しました。
https://www.pulumi.com/blog/oidc-blog/

これによりクレデンシャル情報を保存する必要性がなくなり、デプロイしたときだけPulumi側で一時的な認証情報を発行し、完了したら自動的に削除してくれるため最小限のアクセス管理ができます。

以前[GitHub ActionsとAWSでOIDC連携したこと](https://zenn.dev/yuta28/articles/terraform-gha)がありますので、Pulumi Cloudも同じようにOIDC認証してみました。

[^1]: https://openid.net/foundation/
# OIDC連携

## AWS操作
PulumiのドキュメントではAWSマネジメントコンソール上からIDプロバイダとIAMロールを作成しています。
https://www.pulumi.com/docs/pulumi-cloud/deployments/oidc/aws/

ただややわかりにくい表記でしたのでCloudFormationのテンプレートファイルを作成しました。

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

CloudFormationでスタックを作成すると自身のPulumi Cloudの組織名とプロジェクト名を入力します。
![](/images/pulumi-oidc-aws/image1.png)

![](/images/pulumi-oidc-aws/image2.png)
*Pulumi Cloud画面*

その他の設定は不要ですのでCloudFormationスタックを作成します。作成が完了したらPulumi用のIDプロバイダとIAMロールが作成されます。
![](/images/pulumi-oidc-aws/image3.png)
*IDプロバイダ*

![](/images/pulumi-oidc-aws/image4.png)
*IAMロール*

テンプレートファイルを確認すればわかりますがIAMロールにはAdminロールを渡しています。もし権限を最小限にしたい場合は適宜テンプレートファイルを修正してください。
作成できましたらPulumi Cloud側でOIDC認証の設定をします。

## Pulumi Cloud操作
プロジェクト内の連携するスタックの設定タブからDeploy画面を開きます。
![](/images/pulumi-oidc-aws/image5.png)

下の方にスクロールしますと各種パブリッククラウド毎にOIDC認証に関する設定タブがありますので、AWSのOIDC認証を有効化します。IAMロールをのARNとセッション名(なんでも可)をつけて保存します。
![](/images/pulumi-oidc-aws/image6.png)
*セッションの有効期限も設定可*

## GitHub PR
設定が完了しましたのでGitHubでPRしてマージしてみましょう。Pulumi CloudでCI/CDパイプラインが動いてAWSリソースが作成されます。
![](/images/pulumi-oidc-aws/image7.png)

# 所感
Pulumi CloudとAWSをOIDC認証で連携できるようにしました。OIDC認証はCI/CDプラットフォームによっては有料プラン以上しか利用できない場合もありますが、Pulumi Cloudは無料の個人プランでも使えるのが有難かったです。
前の記事でも述べましたが、PulumiはAWS CDKやTerraformにも負けないIaCツールだと思っていますので興味がありましたらぜひ使ってみてください。

余談ですが私が作成したCloudFormationテンプレートファイルをPulumiのドキュメントに組み込んでもらえないかOSSコミットしようと思います。
# 参考文献
https://youtu.be/0IcpWLhatoY