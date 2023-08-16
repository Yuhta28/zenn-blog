---
title: "TerraformからPulumiへの移行方法"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","terraform","typescript","aws"]
published: false
---

# 序論
先日、Terraform開発元であるHashiCorp社から今後リリースするすべての製品においてライセンスを変更するアナウンスが発表されました。
https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license

ライセンス変更による影響はこちらの記事などをご覧頂ければわかりやすいと思います。
https://zenn.dev/the_exile/articles/b90fe8c5c41694

このライセンス変更はHashiCorp社と競合する企業に対して競合プロダクト内にコミュニティ版HashiCorp製品を無償で利用できなくなるもので、Terraformなどを利用するユーザー自身には影響はありません。Terraform Cloudと競合するサービスを提供するSpacelift[^1]やenv0[^2]がこのライセンス変更に対して自身らのプロダクトの今後について述べております。2社ともこのライセンス変更に対して賛成の立場をとり、今後自分たちのプロダクトについてどうするか弁護士と相談すると記載しています。[^3][^4]

一方で同じプロビジョニングツールとして知られるPulumiへの関心が高まっています。
私が以前書いたPulumiの記事[^5]に急にイイねが増えており、閲覧数も急上昇していました。
![](/images/migration-terraform-to-pulumi/image1.png)
*8/7~8/16の記事表示回数推移*

先ほどのライセンス変更の詳細を紹介した記事が公開されて以降日本でもPulumiについて検索する回数が増えているようです。
![](/images/migration-terraform-to-pulumi/image2.png)
*GoogleTrendsより*

しかしすでに多く企業、個人でインフラのコード化を実現する場合ツールにTerraformを採用している人は多いと思います。
Pulumiは他のプロビジョニングツールからの移行を提供する手順について紹介[^6]していますので、TerraformからPulumiに移行する方法について紹介していきます。

[^1]: https://spacelift.io/
[^2]: https://www.env0.com/
[^3]: https://spacelift.io/blog/hashicorps-license-change
[^4]: https://www.env0.com/blog/hashicorp-license-change
[^5]: https://zenn.dev/yuta28/articles/pulumi-ai-revolutionary
[^6]: https://www.pulumi.com/docs/using-pulumi/adopting-pulumi/migrating-to-pulumi/

# 対象読者

- Pulumiについて知りたい人
- TerraformからPulumiへの移行に興味がある人

# Pulumiとは
https://www.pulumi.com/

先にPulumiについて簡単に紹介しますとプログラミング言語でインフラ開発できるプロビジョニングツールとなります。以前[私が書いた記事](https://zenn.dev/yuta28/articles/pulumi-ai-revolutionary)と重複している部分もありますのでよろしければそちらもご覧ください。
Terraformで利用されるHCLのような独自言語とは異なり、TypeScriptやPython、Goなど開発者が慣れ親しんだ言語でインフラをコーディングできます。
## 対応言語[^7]
- Node.js(JavaScript,TypeScript)
- Python
- Go
- .NET(C#,F#,Visual Basic)
- Java
- Pulumi YAML

プログラミング言語以外にもYAMLでも書けますので、プログラミングが不得手なインフラエンジニアでも利用できると思います。
[^7]: https://www.pulumi.com/docs/languages-sdks/

AWSやGoogle Cloud、Azureといった主要なパブリックラウド以外にもDatadogやAuth0などのクラウドSaaSといった100以上ものパッケージを提供しています。
https://www.pulumi.com/registry/

## ステート管理方法
プロビジョニングツールで構築された実際のインフラ情報を記録したメタデータのことをステート情報と呼び、ステート情報には秘文情報も含まれるため取り扱いには注意が必要です。Terraformはデフォルトではデプロイを実行したローカルマシン内にステートファイルが生成されます。チームでインフラコード開発する際はステートファイルをAmazon S3やCloud Storageなどの外部ストレージに保存するかTerraform Cloudにおいてチームで管理できるようにします。

Pulumiは標準でPulumi Cloudと呼ばれるステート管理やマネージドなCI/CDを提供してくれるSaaSを提供してくれます。
https://www.pulumi.com/product/pulumi-cloud/

![](/images/migration-terraform-to-pulumi/image3.png)
*コンソール画面*

# 他ツールからの移行
冒頭でも紹介したとおりPulumiは他のプロビジョニングツールのコードをPulumiに移行するための手段が公式で紹介されています。
以前は[tf2pulumi](https://github.com/pulumi/tf2pulumi)と呼ばれるコンバーターツールを公式が提供していましたが、2023年6月に新たに`pulumi convert`サブコマンドの実装を発表しました。[^8]
https://www.pulumi.com/blog/converting-full-terraform-programs-to-pulumi/

`pulumi convert`を使ってTerraformで書いたコードをPulumiに移行してみようと思います。

[^8]: これにともないtf2pulumiの使用は非推奨になりました

## サンプルコード
サンプル用にS3バケット作成するシンプルなTerraformコードを用意しました。

```hcl:main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "3.5.1"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "ap-northeast-1"
}
provider "random" {}

# 一意のバケット名となるSuffix
resource "random_string" "s3_bucket_name" {
  length  = 8
  special = false
  upper   = false
}
# Create S3
resource "aws_s3_bucket" "migration_S3" {
  bucket = "yuta-${random_string.s3_bucket_name.id}"
}
```

`terraform apply`コマンドでデプロイするとS3が作成されます。
![](/images/migration-terraform-to-pulumi/image4.png)

このTerraformファイルが配置されているディレクトリ上でPulumiのコンバーターコマンドを実行します。

```console
pulumi convert --from terraform --language typescript
```

`--language`部分に対応言語で述べたプログラミング言語を指定することでTerraformをPulumiへ変換できます。
上記コマンドを実行するとディレクトリ直下にPulumiで使われるプロジェクト一式が生成されます。

```console
$ tree -L 1
.
├── Pulumi.yaml
├── index.ts
├── main.tf
├── node_modules
├── package-lock.json
├── package.json
├── terraform.tfstate
└── tsconfig.json

1 directory, 9 files
```

`index.ts`ファイルが構成ファイルになります。

```typescript:index.ts
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";
import * as random from "@pulumi/random";

const s3BucketName = new random.RandomString("s3BucketName", {
    length: 8,
    special: false,
    upper: false,
});
// Create S3
const migrationS3 = new aws.s3.BucketV2("migrationS3", {bucket: pulumi.interpolate`yuta-${s3BucketName.id}`});
```

`pulumi up`コマンドでS3を作成します。

:::message
初めてPulumiコマンドを実行するとPulumi Cloudへログインが求められます。
:::

```powershell
$ pulumi up
Please choose a stack, or create a new one: <create a new stack>
Please enter your desired stack name.
To create a stack in an organization, use the format <org-name>/<stack-name> (e.g. `acmecorp/dev`): Yuhta28/migration-terraform
Created stack 'migration-terraform'
Previewing update (migration-terraform)

     Type                          Name                      Plan
 +   pulumi:pulumi:Stack           temp-migration-terraform  create
 +   ├─ random:index:RandomString  s3BucketName              create
 +   └─ aws:s3:BucketV2            migrationS3               create

Resources:
    + 3 to create

Do you want to perform this update? yes
Updating (migration-terraform)

     Type                          Name                      Status
 +   pulumi:pulumi:Stack           temp-migration-terraform  created (1s)
 +   ├─ random:index:RandomString  s3BucketName              created (0.23s)
 +   └─ aws:s3:BucketV2            migrationS3               created (1s)

Resources:
    + 3 created

Duration: 8s
```

# 所感

# 参考文献
https://www.pulumi.com/docs/concepts/
https://www.pulumi.com/docs/pulumi-cloud/
https://www.pulumi.com/docs/cli/commands/pulumi_convert/