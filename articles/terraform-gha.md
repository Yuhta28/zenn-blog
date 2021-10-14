---
title: "GitHub ActionsにAWSクレデンシャル情報を渡さずにTerraformでCI/CDをやってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","terraform","GitHubActions","OIDC"]
published: true
---

# 概要
先日、非公式ながらGitHub ActionsのAWSアクションで以下のような面白い機能が発表されました。
https://twitter.com/__steele/status/1437984026145427461

よりわかりやすく嚙み砕くとこういうことです。
https://twitter.com/toricls/status/1438120050167189510

Circle CIやGitHub ActionsでAWSを使う場合は事前に環境変数にアクセスキーとシークレットキーを登録させてCIを動かしてきましたが、そのためにIAMユーザーを発行して鍵を管理するのは手間だったのでこれはいいアップデートです。

今回はTerraformとGitHub Actionsを組み合わせたCI/CDにこの機能を取り入れてGitHub ActionsにIAMロールを渡してEC2インスタンス構築のCI/CDを実装してみようと思います。
https://github.com/aws-actions/configure-aws-credentials

# GitHub Actionsを用いたTerraformのCI/CD
TerraformでAWSリソースを構築する際GitHub ActionsやCircle CIでCI/CDを構築する場合、事前に環境変数にIAMユーザーのアクセスキーとシークレットキーを登録する必要がありました。

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
まずはAWSでGitHubに渡すためのIAMロールと認証連携のためのOIDCプロバイダーを作成します。
IAMのコンソール画面から作成しても大丈夫ですがクラスメソッドさんのブログ[^1]にいい感じのCloudFormationスタックテンプレートがありましたのでそれをお借りします。

[^1]: https://dev.classmethod.jp/articles/github-actions-without-permanent-credential/

#### IAMロール&OIDCプロバイダー作成

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
                token.actions.githubusercontent.com:sub: !Sub repo:${RepoName}:*

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
      Url: https://token.actions.githubusercontent.com
      ClientIdList: [sigstore]
      ThumbprintList: [a031c46782e6e6c662c2c87c76da9aa62ccabd8e]

Outputs:
  Role:
    Value: !GetAtt Role.Arn
```

ブログ先のスタックテンプレートに合わせてこのIAMロールは特定のリポジトリ[^2]のみで動くようにしています。
IAMロールの内容ですが、STS以外にもEC2を作成するためと`terraform.tfstate`をS3に格納するためにS3のフルアクセス権限を渡しています。
このテンプレートを起動させてIAMのコンソール画面を確認します。
![](/images/terraform-gha/image2.png)

[^2]: ここでは`Yuhta28/terraform-githubaction-ci`にしています

IAMロール`ExampleGithubRole`が作成されてますので、ロールARNを手元に控えます。
OIDCプロバイダーも作成されていることも念のため確認しておきます。
![](/images/terraform-gha/image3.png)


## ワークフロー作成
次にGitHub Actionsを動かすためのワークフローファイルを作成します。
#### ワークフロー作成

新規でGitHub Actionsワークフローファイルを作成する場合、GitHubからいくつかテンプレートが用意されていますのでその中からTerraformを選べば↓のワークフローファイルのひな形が簡単に作れます。
![](/images/terraform-gha/image4.png)

```yml:workflow.yml
name: 'Terraform'
on:
  push:
    branches:
      - main
  pull_request:
    

jobs:
  terraformCICD:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
    - run: sleep 5

    - uses: actions/checkout@v2

    - name: Configure AWS
      run: |
        export AWS_ROLE_ARN=arn:aws:iam::<AWS_AccountID>:role/ExampleGithubRole
        export AWS_WEB_IDENTITY_TOKEN_FILE=/tmp/awscreds
        export AWS_DEFAULT_REGION=ap-northeast-1
     
        echo AWS_WEB_IDENTITY_TOKEN_FILE=$AWS_WEB_IDENTITY_TOKEN_FILE >> $GITHUB_ENV
        echo AWS_ROLE_ARN=$AWS_ROLE_ARN >> $GITHUB_ENV
        echo AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION >> $GITHUB_ENV
        curl -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=sigstore" | jq -r '.value' > $AWS_WEB_IDENTITY_TOKEN_FILE

    # Install the latest version of Terraform CLI and configure the Terraform CLI configuration file with a Terraform Cloud user API token
    - name: Setup Terraform
      uses: aws-actions/configure-aws-credentials@master
      with:
        role-to-assume: "${{ env.AWS_ROLE_ARN }}"
        web-identity-token-file: "${{ env.AWS_WEB_IDENTITY_TOKEN_FILE }}"
        aws-region: "${{ env.AWS_DEFAULT_REGION }}"
        role-duration-seconds: 900
        role-session-name: GitHubActionsTerraformCICD
    
    # Checks that all Terraform configuration files adhere to a canonical format
    - name: Terraform Format
      run: terraform fmt -check -diff

    # Initialize a new or existing Terraform working directory by creating initial files, loading any remote state, downloading modules, etc.
    - name: Terraform Init
      run: terraform init

    - name: Terraform Validate
      run: terraform validate -no-color

    # Generates an execution plan for Terraform
    - name: Terraform Plan
      if: github.event_name == 'pull_request'
      run: terraform plan -no-color
      continue-on-error: true

      # On push to main, build or change infrastructure according to Terraform configuration files
      # Note: It is recommended to set up a required "strict" status check in your repository for "Terraform Cloud". See the documentation on "strict" required status checks for more information: https://help.github.com/en/github/administering-a-repository/types-of-required-status-checks
    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve

```

注目部分は`Configure AWS`です。
#### 解説ポイント
GitHubに3つの環境変数をセットします。

- AWS_ROLE_ARN
- AWS_WEB_IDENTITY_TOKEN_FILE
- AWS_DEFAULT_REGION

##### AWS_ROLE_ARN
引き受けるロールのARNを指定
先ほど手元に控えたIAMロールのARNをここに貼り付けます。

##### AWS_WEB_IDENTITY_TOKEN_FILE
Web IDトークンファイルへのパス
詳しくは後述

##### AWS_DEFAULT_REGION
デフォルトのリージョン
東京リージョンを指定したい場合は`ap-northeast-1`

最後のcurl~でWeb IDトークンファイルにトークンを渡すためのパラメーターを渡しています。

- `ACTIONS_ID_TOKEN_REQUEST_TOKEN`
- `ACTIONS_ID_TOKEN_REQUEST_URL`

この2つはGitHub Actionsの環境変数のようですが、ドキュメントにも記載がなく詳細は不明でした。
作成したOIDCプロバイダー`token.actions.githubusercontent.com`に対してリクエストを投げてトークンを取得しているみたいですが、詳しいことはドキュメントに記載されたら確認します。

このトークンを先ほどのWeb IDトークンファイルへ格納し、次の`configure-aws-credentials`に渡しています。

```yml:ワークフローファイル抜粋
      uses: aws-actions/configure-aws-credentials@master
      with:
        role-to-assume: "${{ env.AWS_ROLE_ARN }}"
        web-identity-token-file: "${{ env.AWS_WEB_IDENTITY_TOKEN_FILE }}"
        aws-region: "${{ env.AWS_DEFAULT_REGION }}"
        role-duration-seconds: 900
        role-session-name: GitHubActionsTerraformCICD
```

冒頭に記載したワークフローファイルと比較するとAWSのアクセスキーとシークレットキーの環境変数がなくなり、IAMロールとOIDCのトークンをセットしています。
ロールのセッション名は必須ではありませんが、運用時にログ調査する際に名前があったほうが追いやすいので設定しておくとよいです。

## Terraform構築
ワークフローファイルが作成されましたので次はTerraformの実装です。
今回は単純にEC2インスタンスを1台作成して、`terraform.tfstate`をS3に格納する構成にします。

```bash:ディレクトリ構成
|
├── main.tf
└── variables.tf
```

```hcl:main.tf
terraform {
  backend "s3" {
    bucket = "バケット名を指定"
    key    = "terraform.tfstate"
    region = "ap-northeast-1"

  }
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.27"
    }
  }

  required_version = ">= 0.14.9"
}

provider "aws" {
  region = "ap-northeast-1"
}

resource "aws_instance" "app_server_yuta" {
  ami                    = "ami-00cb500575fd9f9be"
  instance_type          = "t2.micro"
  vpc_security_group_ids = ["sg-058cda7badca49175", "sg-0c5b21b30ff8741cd"]
  subnet_id              = "subnet-05b2284acd7a081db"

  tags = {
    Name = var.instance_name
  }
}
```

```hcl:variables.tf
variable "instance_name" {
  description = "Value of the Name tag for the EC2 instance"
  type        = string
  default     = "Yuta-ServerInstance"
}
```

Terraformに関してはAMIやセキュリティグループ箇所を自分のものに読み替えて設定してください。
セットアップできましたらブランチを切ってGitHubにpushします。

## GitHub Actions起動
pushしてmainブランチへPRを出しますとGitHub Actionsが動きます。

![](/images/terraform-gha/image5.png)

`terraform plan`ステップまで問題なく実行されればマージが可能になりますのでマージします。
mainブランチにマージされますと`terraform apply`ステップが走り、EC2インスタンスが作成されます。
![](/images/terraform-gha/image7.png)

作成できることを確認できましたら最後のrunをdestoryに変更して再度pushしてPRします。
同じようにCIのチェックが入り、異常がなければマージしてEC2インスタンスを削除します。
![](/images/terraform-gha/image8.png)
![](/images/terraform-gha/image9.png)

# 所感
Terraform+GitHub ActionsのCI/CD構築を一通り実践しました。
新しい機能の検証は中々大変でこの仕組みを試すのに大分時間がかかりました。
他の人のブログのおかげでCI/CDの構築が完了できたのでブログ発信の大事さが分かった気がします。
今回はEC2の作成とS3に`terraform.tfstate`を置くためにEC2とS3のフルアクセスを付与しましたが、TerraformでAWSリソースを作成することを考えるとAdmin権限をあげてもいい気がします。
ともあれこの機能はまだ新しく日本語記事も少ないと思いますので、この記事を読んで皆さんも是非手元で試してみてほしいと思います。

# LT資料
LTのスライド資料です。
@[speakerdeck](b12a92fc9ba3454085aab839e81aedd5)

# 参考文献
https://dev.classmethod.jp/articles/github-actions-without-permanent-credential/
https://zenn.dev/yutaro1985/articles/b012f69b49bec095b9f1
https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-role.html#cli-configure-role-oidc

