---
title: "GitHub ActionsにAWSクレデンシャル情報を渡さずにTerraformでCI/CDをやってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","terraform","GitHubActions"]
published: false
---

# 概要
先日、非公式ながらGitHub ActionsのAWSアクションで以下のような面白い機能が発表されました。
https://twitter.com/__steele/status/1437984026145427461

よりわかりやすく嚙み砕くとこういうことです。
https://twitter.com/toricls/status/1438120050167189510

CirleCIやGitHub ActionsでAWSを使う場合は事前に環境変数にアクセスキーとシークレットキーを登録させてCIを動かしてきましたが、そのためにIAMユーザーを発行して鍵を管理するのは手間だったのでこれはいいアップデートです。

今回はTerraformとGitHub Actionsを組み合わせたCI/CDにこの機能を取り入れてGitHub ActionsにIAMロールを渡してEC2構築のCI/CDを実装してみようと思います。
https://github.com/aws-actions/configure-aws-credentials

# GitHub Actionsを用いたTerraformのCI/CD
TerraformでAWSリソースを構築する際GitHub ActionsやCircle CIをでCI/CDを構築する場合、事前に環境変数にIAMユーザーのアクセスキーとシークレットキーを登録する必要がありました。

#### 従来のTerraform+GitHub Actions
![](/images/terraform-gha/image1.png)
*GitHub Actionsの環境変数登録*

```yml:workflow.yml
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }} #アクセスキー
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }} #シークレットキー
```

この場合アクセスキーとシークレットキーを発行するためにIAMユーザーを新規作成しないといけないこと、マルチアカウントでTerraformを展開したい場合アクセスキーとシークレットキーを登録しなおさないといけないことがあり手間がかかります。

## 下準備
まずはAWSでGitHubに渡すためのIAMロールと認証連携のためのIDプロバイダーを作成します。
IAMのコンソール画面から作成しても大丈夫ですがクラスメソッドさんのブログ[^1]にいい感じのCloudFormationスタックテンプレートがありましたのでそれをお借りします。

[^1]: https://dev.classmethod.jp/articles/github-actions-without-permanent-credential/

#### IAMロール&IDプロバイダー作成

```yml:CloudFormation
AWSTemplateFormatVersion: "2010-09-09"
Description: "IAM Role for GHA"

Parameters:
  RepoName:
    Type: String
    Default: Yuhta28/terraform-githubaction-ci

Resources:
  Role:
    Type: AWS::IAM::Role
    Properties:
      RoleName: ExampleGithubRole
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Action: sts:AssumeRoleWithWebIdentity
            Principal:
              Federated: !Ref GithubOidc
            Condition:
              StringLike:
                vstoken.actions.githubusercontent.com:sub: !Sub repo:${RepoName}:*

  Policy:
    Type: AWS::IAM::Policy
    Properties:
      PolicyName: test-gha
      Roles:
       - !Ref Role
      PolicyDocument: 
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Action:
              - 'ec2:*'
              - 'sts:GetCallerIdentity'
              - 's3:*'
            Resource: '*'

  GithubOidc:
    Type: AWS::IAM::OIDCProvider
    Properties:
      Url: https://vstoken.actions.githubusercontent.com
      ClientIdList: [sigstore]
      ThumbprintList: [a031c46782e6e6c662c2c87c76da9aa62ccabd8e]

Outputs:
  Role:
    Value: !GetAtt Role.Arn
```



# 参考文献
https://dev.classmethod.jp/articles/github-actions-without-permanent-credential/
https://zenn.dev/yutaro1985/articles/b012f69b49bec095b9f1

