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

会社の既存AWSリソースをTerraformを活用してIaC推進していますが、その中で躓いた部分やどのようなTerraformディレクトリ構成にしているか紹介していこうと思います。

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
モジュールはTerraformの設計上で、複数のファイルをまとめて管理することができます。
https://www.terraform.io/language/modules/syntax
GitHubのようにTerraformのレジストリ上に他の人が作ったモジュールが、公開されてダウンロードでき自由に組み込めます。

ただ今回は既存インフラリソースをインポートする必要があるため、モジュールを自作し、その中に含めることにしました。

# Terraformアーキテクチャ
今回設計したTerraformアーキテクチャは以下のようになります。
![](/images/challenge-for-existing-iac/image1.png)

`env`ディレクトリ配下を環境単位で分けて、その中で`terraform import`して既存リソースをコード管理していきます。