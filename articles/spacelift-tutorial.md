---
title: "TerraformのCI/CD基盤を自動構築してくれるサービスSpaceliftを触ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CI","CD","spacelift","IaC","terraform"]
published: false
---

# 概要
以前既存インフラをコード化するときに気を付けることという記事を書きました。
https://zenn.dev/yuta28/articles/iac-existing-infrastructure
そしてその記事を英訳したものをdev.toにも投稿しました。
https://dev.to/yuta28/keep-in-mind-when-coding-existing-infrastructure-15b3

するとLinkedInで私の記事をご覧になったエンジニアからメッセージを頂き、やり取りしていたのですがそのエンジニアが所属されている企業のプロダクトが面白そうでしたので、ハンズオンしてみて感想を書いてみました。
https://spacelift.io/

# Spaceliftとは
Spaceliftは、Spacelift社が開発したクラウドインフラのIaCを支援するサービスです。
日本語情報が一切なく公式ページの概要を都度都度DeepL翻訳しながら、どうやら[Terraform Cloud](https://cloud.hashicorp.com/products/terraform)や[Atlantis](https://www.runatlantis.io/)のようにTerraformの自動実行を提供してくれるサービスのようです。(IaCのためのCI/CDプラットフォームとサイトには記載されています)

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
StacksはSpaceliftのコンポーネントの中で最も重要な要素です。Stacksにインフラ情報のソースコードや現在のインフラ状態を保持させることでインフラリソースの管理運用ができます。
https://docs.spacelift.io/concepts/stack
このStacksに先ほど作成したGitHubのリポジトリを登録します。
`ADD STACK`で新規にStacksを作成します。

![](/images/spacelift-tutorial/image4.png)
最初に連携したいVCSとリポジトリ、トリガーとなるブランチを選択します。

![](/images/spacelift-tutorial/image5.png)
次にバックエンドの設定を行ないます。Terraform以外にもCloudFormationやPulumiなども選択できるみたいです。
# 参考文献
https://jp.techcrunch.com/2021/02/12/2021-02-11-cloud-automation-startup-spacelift-raises-6m-series-a-led-by-blossom-capital/