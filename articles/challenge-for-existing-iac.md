---
title: "既存インフラのIaC実現へ向けた取り組み"
emoji: "🐭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["terraform","AWS","IaC"]
published: false
---

# 概要
この記事は以前書いた記事の続き物みたいなものです。
https://zenn.dev/yuta28/articles/iac-existing-infrastructure

会社の既存AWSリソースをTerraformを活用してIaC推進していますが、そのなかで躓いた部分やどのようなTerraformディレクトリ構成にしているか紹介していこうと思います。

# ディレクトリ設計
Terraformを活用したIaCで悩まれるポイントなのがディレクトリ設計だと思います。
今現在Hasicorp社でもTerraformのディレクトリ設計についてベストプラクティスと呼べるようなものは発表されておらず、いくつかの企業の事例なども調べましたが用途・目的に応じてさまざまな設計で運用しているみたいです。
またTerraformは0.12系以前と0.13系以上でバージョン非互換があり、あまり古い記事のソースコードですとシンタックスエラーを起こしてしまいます。
ディレクトリ設計するうえで運用を意識した設計を大事にしたいと考えましたので、以下の要望を満たせるようなディレクトリ設計にしようと考えました。

- 本番環境と検証環境で似た構成のAWSリソースは一つのTerraformリソースを使いまわせる設計にしたい
- 本番環境と検証環境はAWSアカウント単位で異なるため、別アカウントへのTerraform実行はセキュアに実行できるようにしたい
- チームでTerraformに詳しい人は少なく初めて触る人もいるので、複雑な設計にはしない

いきなりすべてのAWSリソースをIaC化するのではなく、メインプロダクト関連のリソースのみをTerraformで管理することから始めてみて小さく設計するようにしました。

```bash: ディレクトリ構成一部
$ tree
.
├── README.md
├── env
│   ├── prd
│   │   ├── ec2.tf
│   │   ├── main.tf
│   │   └── vpc.tf
│   └── stg
│       ├── ec2.tf
│       ├── main.tf
│       └── vpc.tf
└── modules
    ├── ec2
    │   ├── main.tf
    │   └── variables.tf
    └── vpc
        ├── main.tf
        ├── output.tf
        └── variables.tf
```

弊社では`module`を使ったディレクトリ設計でIaC化してみました。


## moduleについて
モジュールは複数のリソースをまとめたテンプレートみたいなものです。
https://www.terraform.io/language/modules/syntax
GitHubのようにTerraformのレジストリ上に、他の人が作ったモジュールが公開されて利用者は自由にダウンロードして活用できます。

ただ今回は既存インフラリソースをインポートする必要があるため、モジュールを自作し、その中に含めることにしました。

# Terraformアーキテクチャ
今回設計したTerraformアーキテクチャは以下のようになります。
![](/images/challenge-for-existing-iac/image1.png)

`env`ディレクトリ配下を環境単位で分けて、そのなかで`terraform import`して既存リソースをコード管理していきます。

また会社では環境ごとにAWSアカウントを使い分けるマルチアカウントで運用しております。
このTerraformの実行場所はStg環境内のEC2インスタンスですのでStg環境内のリソースでしたらEC2インスタンスにIAMロールをアタッチさせるだけで問題ありませんが、異なるAWSアカウントへのリソース作成を行なう場合Assume Role[^1]を使ってEC2インスタンスにPrdアカウント内のAWSリソース作成権限を渡していきます。
[^1]: https://dev.classmethod.jp/articles/iam-role-passrole-assumerole/

```json: env/prd/main.tf
provider "aws" {
  region = "ap-northeast-1"
  assume_role {
    role_arn  =  "arn:aws:iam::XXXXXXXXXXXXXXXXX:role/Terraform-Prd-Switch"
  }
}
```

# 各種ファイルについて
`terraform apply`を実行する場所は`env`ディレクトリ配下の環境ディレクトリ直下で行ないます。リソース単位でtfファイルを作成しており、例として`vpc.tf`の中身は以下のようになります。

```hcl: vpc.tf
resource "aws_vpc" "terraform-vpc" {
  cidr_block = var.cidr_block
  tags = {
    Name      = "${var.Tag_Name}-vpc"
    Terraform = "True"
  }
}

resource "aws_internet_gateway" "terraform-igw" {
  vpc_id = aws_vpc.terraform-vpc.id
  tags = {
    Name      = "${var.Tag_Name}-igw"
    Terraform = "True"
  }
}

resource "aws_subnet" "terraform-public-subnet" {
  for_each                = var.public-AZ
  vpc_id                  = aws_vpc.terraform-vpc.id
  cidr_block              = each.value
  availability_zone       = "ap-northeast-1${each.key}"
  map_public_ip_on_launch = true
  tags = {
    Name      = "terraform-${var.Tag_Name}-public-subnet-${each.key}"
    Terraform = "True"
  }
}

resource "aws_subnet" "terraform-private-subnet" {
  for_each          = var.private-AZ
  vpc_id            = aws_vpc.terraform-vpc.id
  cidr_block        = each.value
  availability_zone = "ap-northeast-1${each.key}"
  tags = {
    Name      = "terraform-${var.Tag_Name}-private-subnet-${each.key}"
    Terraform = "True"
  }
}

resource "aws_eip" "terraform-nat-eip" {
  for_each = toset(var.eip-NAT-AZ)
  tags = {
    Name = "${var.Tag_Name}-eip-${each.key}"
  }
  depends_on = [
    aws_internet_gateway.terraform-igw
  ]
}

resource "aws_nat_gateway" "terraform-nat" {
  for_each  = toset(var.eip-NAT-AZ)
  subnet_id = aws_subnet.terraform-public-subnet[each.key].id
  depends_on = [
    aws_internet_gateway.terraform-igw
  ]
  allocation_id = aws_eip.terraform-nat-eip[each.key].id
  tags = {
    Name      = "${var.Tag_Name}-nat-${each.key}"
    Terraform = "True"
  }
}

resource "aws_route_table" "terraform-public-rt" {
  for_each = var.public-AZ
  vpc_id   = aws_vpc.terraform-vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.terraform-igw.id
  }
  tags = {
    Name      = "${var.Tag_Name}-public-rt-${each.key}"
    Terraform = "True"
  }
}

resource "aws_route_table_association" "terraform-public-rt-assoc" {
  for_each       = var.public-AZ
  subnet_id      = aws_subnet.terraform-public-subnet[each.key].id
  route_table_id = aws_route_table.terraform-public-rt[each.key].id
}

resource "aws_route_table" "terraform-private-rt" {
  for_each = var.private-AZ
  vpc_id   = aws_vpc.terraform-vpc.id
  tags = {
    Name      = "${var.Tag_Name}-private-rt-${each.key}"
    Terraform = "True"
  }
}
```