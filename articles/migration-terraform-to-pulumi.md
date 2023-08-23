---
title: "TerraformからPulumiへの移行方法"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","terraform","typescript","aws"]
published: true
published_at: 2023-08-17 11:00
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

このコマンドは2023年8月現在でTypeScript、Python、Go、C#に対応しています。`pulumi convert`を使ってTerraformで書いたコードをPulumiに移行してます。

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
Terraform関係のファイルと一緒の場所に置きたくない場合、`--out <ディレクトリ名>`で違う場所にプロジェクトが生成されます。

```powershell
$ pulumi convert --from terraform --language typescript --out typescript-convert
Converting from terraform...
Converting to nodejs...
Installing dependencies...

added 215 packages, and audited 216 packages in 45s

68 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
Finished installing dependencies

$ ls .\typescript-convert\

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----        2023/08/16     22:38                  node_modules
-a---        2023/08/16     22:38             20   .gitignore
-a---        2023/08/16     22:38            366   index.ts
-a---        2023/08/16     22:38          83688   package-lock.json
-a---        2023/08/16     22:38            203   package.json
-a---        2023/08/16     22:38             75   Pulumi.yaml
-a---        2023/08/16     22:38            354   tsconfig.json
```

`index.ts`ファイルにインフラの構成情報が記載されます。

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

せっかくなので他の言語でS3を作成する構成情報ファイルをまとめました。本題とはそれるのでトグルで丸めてますが興味ありましたらご覧ください。
:::details おまけ

### Python
```python:__main__.py
import pulumi
import pulumi_aws as aws
import pulumi_random as random

s3_bucket_name = random.RandomString("s3BucketName",
    length=8,
    special=False,
    upper=False)
# Create S3
migration_s3 = aws.s3.BucketV2("migrationS3", bucket=s3_bucket_name.id.apply(lambda id: f"yuta-{id}"))
```

### Go
```go:main.go
package main

import (
	"fmt"

	"github.com/pulumi/pulumi-aws/sdk/v5/go/aws/s3"
	"github.com/pulumi/pulumi-random/sdk/v4/go/random"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {
		s3BucketName, err := random.NewRandomString(ctx, "s3BucketName", &random.RandomStringArgs{
			Length:  pulumi.Int(8),
			Special: pulumi.Bool(false),
			Upper:   pulumi.Bool(false),
		})
		if err != nil {
			return err
		}
		_, err = s3.NewBucketV2(ctx, "migrationS3", &s3.BucketV2Args{
			Bucket: s3BucketName.ID().ApplyT(func(id string) (string, error) {
				return fmt.Sprintf("yuta-%v", id), nil
			}).(pulumi.StringOutput),
		})
		if err != nil {
			return err
		}
		return nil
	})
}
```

### C#
```cs:Program.rs
using System.Collections.Generic;
using System.Linq;
using Pulumi;
using Aws = Pulumi.Aws;
using Random = Pulumi.Random;

return await Deployment.RunAsync(() => 
{
    var s3BucketName = new Random.RandomString("s3BucketName", new()
    {
        Length = 8,
        Special = false,
        Upper = false,
    });

    // Create S3
    var migrationS3 = new Aws.S3.BucketV2("migrationS3", new()
    {
        Bucket = s3BucketName.Id.Apply(id => $"yuta-{id}"),
    });

});
```
:::

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

デプロイが完了するとS3が新規に作成されます。
![](/images/migration-terraform-to-pulumi/image5.png)

# 既存リソースの移行
生成されたコードをデプロイすると新規にリソースが作成されます。ただ実際はTerraformで作成した既存リソースをPulumiに移したいと考える人も多いと思います。
Pulumiにはステートファイルに保存されている既存リソース情報をPulumiのステートファイルにインポートする手順が用意されています。
まず始めにTerraformのステートファイルをローカルに用意します。外部ストレージに保存している場合`terraform state pull`コマンドでローカルにステートファイルをダウンロードできます。

下記の`import.ts`ファイルをステートファイルが存在するディレクトリにコピーします。
https://github.com/pulumi/tf2pulumi/blob/master/misc/import/import.ts

`index.ts`ファイルの先頭に次のコードを追加します。
```ts:index.ts
import "./import"
```

次にTerraformのステート情報を`import.ts`の`importFromStatefile`変数にセットすることで既存リソースをインポートできます。

```powershell
$ pulumi config set importFromStatefile ./terraform.tfstate
```

`pulumi up`コマンドでインポートしたリソースをPulumiで管理できるようになります。
と、ドキュメントにはそう書いていますが実際のところ以下のようなエラーが発生し、インポートができませんでした。
![](/images/migration-terraform-to-pulumi/image6.png)

エラー内容を確認するとステートファイルのバージョンが3でないため失敗したと出ています。現在のTerraformのステートファイルはバージョン4ですが、バージョン3はTerraformのバージョンが0.13以下のもので現在のバージョンとは互換性の保証がありません。ドキュメントで参照している`import.ts`はアーカイブされたリポジトリにあるものでメンテナンスされていないファイルです。
個人的にこれはドキュメントの記載ミスではないかと思いましたのでissueを立てて問い合わせている最中です。
https://github.com/pulumi/pulumi-hugo/issues/3265

続報がありましたら更新いたします。

### 2023/8/24追記
回答がありましてどうやらこの方法はもうサポートしないということでした。ドキュメントも更新されまして上記の`import.ts`ファイルへの導線もドキュメントからなくなりました。

>This is the same issue that had been tracked in pulumi/tf2pulumi#241. However, we've since moved to a new model for conversion from Terraform as part of https://www.pulumi.com/blog/converting-full-terraform-programs-to-pulumi/. That new model does not yet have a replacement for the state conversion. We are tracking adding that in pulumi/pulumi#5953 (comment) and pulumi/pulumi-converter-terraform#23.
In the meantime, we will want to update the docs here to not point at the archived (and not currently working for v4 state) import scripts.

https://github.com/pulumi/pulumi-hugo/issues/3265#issuecomment-1688608046

PulumiにはTerraformと同じように`pulumi import`コマンドも提供されており、このコマンドを実行すれば既存リソースのインポートができます。
https://www.pulumi.com/docs/using-pulumi/adopting-pulumi/import/

Terraformで作成したS3を`pulumi import`でPulumi配下にインポートしてみます。
## pulumi import手順
![](/images/migration-terraform-to-pulumi/image7.png)
*インポート対象S3*

`pulumi import`コマンドを実行する場合最初にスタックを作成する必要があります。
スタックは`pulumi stack init <スタック名>`で作成できます。

```terminal
$ pulumi stack init import-s3
Created stack 'import-s3'
```

スタックを作成したディレクトリ上でインポートしたいリソースのタイプ、リソース名、任意のPulumi IDを引数に指定します。先ほどのS3をインポートしたい場合以下のコマンドを実行します。

```terminal
$ pulumi import aws:s3/bucket:Bucket import-s3 yuta-fmajk8l4
Previewing import (import-s3)

     Type                 Name            Plan
 +   pulumi:pulumi:Stack  temp-import-s3  create
 =   └─ aws:s3:Bucket     import-s3       import

Resources:
    + 1 to create
    = 1 to import
    2 changes

Do you want to perform this import? yes
Importing (import-s3)

     Type                 Name            Status
 +   pulumi:pulumi:Stack  temp-import-s3  created (3s)
 =   └─ aws:s3:Bucket     import-s3       imported (1s)

Resources:
    + 1 created
    = 1 imported
    2 changes

Duration: 4s

Please copy the following code into your Pulumi application. Not doing so
will cause Pulumi to report that an update will happen on the next update command.

Please note that the imported resources are marked as protected. To destroy them
you will need to remove the `protect` option and run `pulumi update` *before*
the destroy will take effect.
```

インポートが成功しますとターミナル上にPulumiのソースコードが表示されます。

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

const imoprt_s3 = new aws.s3.Bucket("import-s3", {
    arn: "arn:aws:s3:::yuta-fmajk8l4",
    bucket: "yuta-fmajk8l4",
    hostedZoneId: "Z2M4EHUR26P7ZW",
    requestPayer: "BucketOwner",
    serverSideEncryptionConfiguration: {
        rule: {
            applyServerSideEncryptionByDefault: {
                sseAlgorithm: "AES256",
            },
        },
    },
}, {
    protect: true,
});
```

Pulumi Cloud上にも作成したスタックとインポートしたリソース情報が記録されています。
![](/images/migration-terraform-to-pulumi/image8.png)

ただTerraformでのインポート作業を経験した人ならご存知だと思いますがこのインポートコマンドは一つ一つのリソース毎にコマンド実行しなければならないため手間がかかります。別ツールからの移行は労力のかかる作業なのでPulumi社も移行支援するサービスを提供しているようです。
https://www.pulumi.com/migrate/

さすがに日本語対応はしていないと思いますが、もし興味ありましたら問い合わせた感想などお待ちしております。

# 所感
TerraformからPulumiへの移行方法について説明しました。
Pulumiは機能開発が活発で昨年あたりから多くの機能アップデートが発表されています。Terraformのライセンス変更自体はユーザーに影響はないものと考えていますが、Pulumiも面白いツールだと思いますのでぜひとも手に取ってみてください。

# 参考文献
https://www.pulumi.com/docs/concepts/
https://www.pulumi.com/docs/pulumi-cloud/
https://www.pulumi.com/docs/cli/commands/pulumi_convert/
https://www.pulumi.com/docs/cli/commands/pulumi_import/