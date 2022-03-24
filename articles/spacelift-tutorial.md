---
title: "TerraformのCI/CD基盤を自動構築してくれるサービスSpaceliftを触ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CI","CD","spacelift","IaC","terraform"]
published: true
---

# 概要
以前既存インフラをコード化するときに気を付けることという記事を書きました。
https://zenn.dev/yuta28/articles/iac-existing-infrastructure
そしてその記事を英訳したものをdev.toにも投稿しました。
https://dev.to/yuta28/keep-in-mind-when-coding-existing-infrastructure-15b3

するとLinkedInで私の記事をご覧になったエンジニアからメッセージを頂き、やり取りしていたのですがそのエンジニアが所属されている企業のプロダクトが面白そうでしたので、ハンズオンをして感想を書いてみました。
https://spacelift.io/

# Spaceliftとは
Spaceliftは、Spacelift社が開発したクラウドインフラのIaCを支援するサービスです。
日本語情報が一切なく公式ページの概要を翻訳して読んでみましたが、どうやら[Terraform Cloud](https://cloud.hashicorp.com/products/terraform)や[Atlantis](https://www.runatlantis.io/)のようにTerraformの自動実行を提供してくれるサービスのようです。(IaCのためのCI/CDプラットフォームとサイトには記載されています)

また昨今注目されている`Policy as Code`と呼ばれるコードでポリシーを定義、管理して柔軟なアクセス制御を実現してインフラ構築の権限管理もできるようです。
https://spacelift.io/product

2022年現在ではTerraform以外にもPulumiやCloudFormation向けのCI/CDプラットフォームの構築も実現できるみたいです。(サイトを見るとAnsibleも近々実装される？)
ソース管理もGitHubやBitbucketなど一般的なGitベースのソース管理システムでバージョン管理できます。
https://spacelift.io/integrations

試しに適当なリポジトリを連携させてどんな感じか調べてみました。

# ハンズオン

## リポジトリ構成

```bash
$ tree
.
├── README.md
├── provider.tf
├── terraform.tfstate
└── vpc.tf

0 directories, 4 files
```

```hcl: provider.tf
provider "aws" {
  region = "ap-northeast-1"
}
```

```hcl: vpc.tf
resource "aws_vpc" "terraform-vpc" {

  cidr_block = "192.168.0.0/16"
  tags = {
    Name      = "terraform-vpc",
    Terraform = "True"
  }
  enable_dns_hostnames = true

}
```

適当なリポジトリを作成して、AWS VPCを作成するTerraformファイルを1個用意しました。
これをローカルで実行しますとAWSにVPCが作成されます。
https://github.com/Yuhta28/mini-mini-Iac_terraform

:::details Terraform実行

```powershell
terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  # aws_vpc.terraform-vpc will be created
  + resource "aws_vpc" "terraform-vpc" {
      + arn                                  = (known after apply)
      + cidr_block                           = "192.168.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_classiclink                   = (known after apply)
      + enable_classiclink_dns_support       = (known after apply)
      + enable_dns_hostnames                 = true
      + enable_dns_support                   = true
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Name"      = "terraform-vpc"
          + "Terraform" = "True"
        }
      + tags_all                             = {
          + "Name"      = "terraform-vpc"
          + "Terraform" = "True"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_vpc.terraform-vpc: Creating...
aws_vpc.terraform-vpc: Still creating... [10s elapsed]
aws_vpc.terraform-vpc: Creation complete after 13s [id=vpc-053ef2c5d26d2052a]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

:::

![](/images/spacelift-tutorial/image1.png)
*VPC作成*

一度このVPCを`terraform destroy`で削除しておきます。

## Spacelift登録
[公式ページ](https://spacelift.io/)から`Get started`をクリックして、自身のGitHubアカウントと連携します。
![](/images/spacelift-tutorial/image2.png)
*登録ページ*

### トップページ
![](/images/spacelift-tutorial/image3.png)

## Stacks作成
次にStacksを作成します。
StacksはSpaceliftのコンポーネントの中でもっとも重要な要素です。Stacksにインフラ情報のソースコードや現在のインフラ状態を保持させることでインフラリソースの管理運用ができます。
https://docs.spacelift.io/concepts/stack
このStacksに先ほど作成したGitHubのリポジトリを登録します。
`ADD STACK`で新規にStacksを作成します。

![](/images/spacelift-tutorial/image4.png)
最初に連携したいVCSとリポジトリ、トリガーとなるブランチを選択します。

![](/images/spacelift-tutorial/image5.png)
次にバックエンドの設定を行ないます。Terraform以外にもCloudFormationやPulumiなども選択できるみたいです。

![](/images/spacelift-tutorial/image6.png)
ここではStacksの動きについて定義させています。作成するStackに管理者権限を持たせるとリソースの作成、更新、削除ができるようになります。他にもこのアカウント内で作成した別のStacksで管理しているリソースに対しても干渉できますので基本的には１個以上の管理者権限を持つStacksを作成する必要があるようです。
デフォルトだとオフになっていますので管理者権限をオンにします。
https://docs.spacelift.io/concepts/stack/stack-settings#administrative

![](/images/spacelift-tutorial/image7.png)
最後にこのStacksの名前やラベル付けなどを設定して保存します。

![](/images/spacelift-tutorial/image8.png)
*作成完了画面*

## トリガー実行
完了画面に記載されているとおり`master`ブランチに新しくコミットされると、Stacksは起動されますし、右上のTRIGGERをクリックしても起動できます。ここではボタンクリックでStacksを起動してみます。
![](/images/spacelift-tutorial/image9.png)
失敗してしまいました。
先に`Preparing`を確認しましたが、どうやらDockerコンテナを起動してそのなかでStacksで管理しているTerraformを起動しているようです。

```bash: Preparing
[01FW52JWBQ6VTT4XCWG7STE43K] Pulling Docker image public.ecr.aws/spacelift/runner-terraform:latest...
[01FW52JWBQ6VTT4XCWG7STE43K] Docker image is GO
[01FW52JWBQ6VTT4XCWG7STE43K] Downloading Terraform 1.1.6...
[01FW52JWBQ6VTT4XCWG7STE43K] Terraform 1.1.6 download is GO (/bin/terraform)
[01FW52JWBQ6VTT4XCWG7STE43K] Creating Docker container...
[01FW52JWBQ6VTT4XCWG7STE43K] Starting Docker container...
[01FW52JWBQ6VTT4XCWG7STE43K] Docker container is GO
```
`Planning`を確認しましたが、AWSのクレデンシャル情報を渡していないため、Spaceliftが私のAWSアカウントにリソースを作成する権限がなくエラーが起きたようです。

```bash: Planning
[01FW52JWBQ6VTT4XCWG7STE43K] Planning changes with 0 custom hooks...
╷
│ Error: error configuring Terraform AWS Provider: no valid credential sources for Terraform AWS Provider found.
│ 
│ Please see https://registry.terraform.io/providers/hashicorp/aws
│ for more information about providing credentials.
│ 
│ Error: no EC2 IMDS role found, operation error ec2imds: GetMetadata, request send failed, Get "http://169.254.169.254/latest/meta-data/iam/security-credentials/": dial tcp 169.254.169.254:80: i/o timeout
│ 
│ 
│   with provider["registry.terraform.io/hashicorp/aws"],
│   on provider.tf line 1, in provider "aws":
│    1: provider "aws" {
│ 
╵
[01FW52JWBQ6VTT4XCWG7STE43K] Unexpected exit code when planning changes: 1
```

なのでSpaceliftに私のAWSアカウントのリソース作成権限を渡してみます。
上部メニュータブのSettingsをクリックします。
![](/images/spacelift-tutorial/image10.png)
## IAMロール権限設定
![](/images/spacelift-tutorial/image11.png)
*[ドキュメントから引用](https://docs.spacelift.io/integrations/cloud-providers/aws#setting-up-the-aws-integration)*


CirlceCIによるCI/CD構築やTerraformをローカル端末で実行する場合、AWSからアクセスキーとシークレットキーをダウンロードして環境変数に登録させて実行権限を付与させていましたが、SpaceliftはIAMロールを渡すことでクレデンシャル情報を渡すことなくIAMの権限を付与できます。
上に書かれている信頼ポリシーステートメントを新規ロールに付与してAdminポリシーをアタッチさせます。

:::message alert
今回はハンズオンなのでadminポリシーですが、権限過多ですので実運用では適切な権限ポリシーをアタッチしてください。
:::
そして作成したIAMのARNを下の空欄に記載します。
![](/images/spacelift-tutorial/image12.png)
これでIAMロールを設定しましたので、もう一度失敗したRunをリトライします。

::: details Planning

```bash:planning
[01FW52JWBQ6VTT4XCWG7STE43K] Planning changes with 0 custom hooks...

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_vpc.terraform-vpc will be created
  + resource "aws_vpc" "terraform-vpc" {
      + arn                                  = (known after apply)
      + cidr_block                           = "192.168.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_classiclink                   = (known after apply)
      + enable_classiclink_dns_support       = (known after apply)
      + enable_dns_hostnames                 = true
      + enable_dns_support                   = true
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Name"      = "terraform-vpc"
          + "Terraform" = "True"
        }
      + tags_all                             = {
          + "Name"      = "terraform-vpc"
          + "Terraform" = "True"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────

Saved the plan to: spacelift.plan

To perform exactly these actions, run the following command to apply:
    terraform apply "spacelift.plan"
[01FW52JWBQ6VTT4XCWG7STE43K] Changes are GO
[01FW52JWBQ6VTT4XCWG7STE43K] Uploading the list of managed resources...
[01FW52JWBQ6VTT4XCWG7STE43K] Please be aware that Run changes calculation includes Terraform output changes.
[01FW52JWBQ6VTT4XCWG7STE43K] Resource list upload is GO
[01FW52JWBQ6VTT4XCWG7STE43K] Generating JSON representation of the plan...
[01FW52JWBQ6VTT4XCWG7STE43K] JSON representation is GO
[01FW52JWBQ6VTT4XCWG7STE43K] No plan policies to evaluate
[01FW52JWBQ6VTT4XCWG7STE43K] Encrypting workspace...
[01FW52JWBQ6VTT4XCWG7STE43K] Uploading workspace...
[01FW52JWBQ6VTT4XCWG7STE43K] Workspace upload is GO
```

:::

![](/images/spacelift-tutorial/image13.png)

Planがとおり、`terraform apply`直前の状態になりました。
これで右上のCONFIRMボタンをクリックして、AWSのリソースを作成します。
## リソース作成
![](/images/spacelift-tutorial/image14.png)
*作成完了画面*

![](/images/spacelift-tutorial/image15.png)
*VPC作成完了画面*

無事にSpaceliftからVPCを作成できました。
もちろんGitHubでブランチを切って`master`ブランチへPRがマージされたときも自動でAWSリソースが更新されます。
![](/images/spacelift-tutorial/image16.png)
*devブランチからTag名を変更してPR*

![](/images/spacelift-tutorial/image17.png)
*Tag名が変更された*

Spaceliftを使うことでTerraformのCI/CD基盤の構築が簡単にできることがわかりました。

## タスク実行
SpaceliftにはStacksに対してコマンドを実行できるタスクと呼ばれる機能があります。
![](/images/spacelift-tutorial/image19.png)
*state一覧情報*
![](/images/spacelift-tutorial/image20.png)
*リソース削除も可能*

`terraform destroy`も実行できてしまうので扱いに注意が必要です。
間違った操作でリソースの変更や削除が発生しないようにポリシーでルールを設定します。
このポリシーもコードで設定でき、前述した`Policy as Code`を実現します。

次にもう一つの機能である`Policy as Code`について述べていきます。

# Policy as Code
冒頭でも触れましたが、Policy as Codeはポリシーと定義された組織が守るべきルールやガイドラインを宣言型の言語を使って記述・管理するアプローチです。
リソースに対するアクセス制限やリソースの使い方、起動更新削除権限などをコードで定義することで自動化し、日々変わる環境の変化に運用を対応させていく考え方です。
コードで管理できるメリットはIaCと同じです。
#### メリット
- 変更履歴が記録される
- 以前のポリシーへのロールバックが容易
- GitHubなどのバージョン管理と連携させることでコードレビュー、CI/CDを実現

Spaceliftはポリシーもコードで管理できるCI/CDプラットフォームとしての一面もあります。
#### 定義できるポリシー一例
- Spaceliftによる誰がログインできるか
- Stacksへのアクセス権限
- Push権限
- Stacksの起動権限

Spaceliftでは[OPA](https://www.openpolicyagent.org/)(Open Policy Agent)と呼ばれるポリシー管理ツールと[Rego](https://www.openpolicyagent.org/docs/latest/policy-language/)と呼ばれるポリシー言語を使ってPaCを実現しています。

ちょっと初めて触る場合、勝手がわからず混乱しますがSpaceliftにはPaCを生成するチュートリアルリポジトリがありましたのでそれを使ってポリシーを作ってみます。
https://github.com/spacelift-io/terraform-starter

## ポリシー作成
チュートリアルを進めますと以下のようにポリシーが作成されます。
![](/images/spacelift-tutorial/image18.png)

例えば`Allow only safe commands`というポリシーの中身を見てみます。

```rego: Allow only safe commands
package spacelift

# This task policy only allows you to exectute a few selected commands.
#
# You can read more about task policies here:
#
# https://docs.spacelift.io/concepts/policy/task-run-policy

allowlist := {
    "ls",
    "terraform taint random_password.secret",
}

allowed { allowlist[_] == input.request.command }
deny["Only selected commands are allowed"] { not allowed }

# Learn more about sampling policy evaluations here:
#
# https://docs.spacelift.io/concepts/policy#sampling-policy-inputs
sample { true }
```

これはタスクで実行できるコマンドについて制限をかけているポリシーで、`ls`と`terraform taint`コマンドのみが実行できるようになっています。
`Settings`タブから`POLICIES`を選択して、Stacksにアタッチさせたいポリシーを選択します。
![](/images/spacelift-tutorial/image23.png)
ポリシーがアタッチされたStacksは操作権限や参照権限を管理できます。

![](/images/spacelift-tutorial/image21.png)
*lsコマンド*
![](/images/spacelift-tutorial/image22.png)
*実行拒否される*

# 所感
Spaceliftを使ってTerraformのCI/CD基盤の実装と`Policy as Code`を体験してみました。
インフラリソースをコード化した後は、GitHubに管理してCI/CD基盤を実装することが目標とする会社も多いですが、CI/CD基盤の実装は作りこみも多く時間がかかる作業です。
SpaceliftではそうしたCI/CD基盤の実装をマネージドに実現してかつポリシーもコード管理できるのでよさそうなサービスだと感じました。
とは言え今のところ日本語情報が一切なくポリシー言語も独特で私は全く理解できなかったので、まだまだこうしたサービスが広まるのは先になるのかなと個人的には思っています。

個別説明会も実施しているみたいですので英語が堪能で、サービスに興味がある人は申し込んでみてはいかがでしょうか。
https://spacelift.io/schedule-demo

# 参考文献
https://jp.techcrunch.com/2021/02/12/2021-02-11-cloud-automation-startup-spacelift-raises-6m-series-a-led-by-blossom-capital/
https://gihyo.jp/magazine/SD/archive/2022/202202