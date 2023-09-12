---
title: "PulumiはIaCの革命児になれるか"
emoji: "☠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","ai","aws"]
published: true
---

# 序論
2023年4月にPulumi社はPulumiサービスにAIを組み込んだ機能を実装したと発表しました。
https://twitter.com/PulumiCorp/status/1646530231639867392?s=20

ここ最近、コード生成系AIが増えてきましたがIaCに特化したツールは珍しいと思いましたので触って感想を述べていきます。

# Pulumiとは
https://www.pulumi.com/
まずPulumiについて説明いたします。
Pulumiはプログラミング言語でインフラストラクチャーを作成できるプロビジョニングツールです。
プロビジョニングツールで有名なものですとTerraform[^1]やAWS CDK[^2]があり比較するとPulumiはやや知名度が低いです。
![](/images/pulumi-ai-revolutionary/image1.png)
*Googleトレンドより*

## 類似ツール比較
### VS Terraform
類似ツールとの違いについて[ドキュメント](https://www.pulumi.com/docs/intro/vs/)で紹介しています。Terraformとの主な違いについては以下の通りです。[^3]

| 項目 | Pulumi | Terraform |
| --- | --- | --- |
| 言語 | 汎用言語（Python, TypeScript, Goなど） | 独自言語（HCL） |
| インフラ再利用 | 関数、クラス、パッケージなど再利用可 | モジュールのみ再利用可 |
| テスト | ユニットテスト、プロパティテストなど一般的なテストフレームワークをサポート | 結合テストのみ |
| 他ツールからのインポート | 可 | 不可 |

Terraformの最大の障壁となるのがHCL(HashiCorp Language)と呼ばれる独自言語の学習だと考えています。開発者は普段自分が親しんでいるプログラミング言語でコードを書きたいという需要がありますが、2023年4月現在以下の言語をサポートしています。[^4]

- TypeScript/JavaScript(Node.js)
- Python
- Go
- .NET
- Java
- YAML

またPulumiはすでに他のツールで構築されているIaCソースコードをPulumiのソースコードに変換してくれるツールを提供してくれます。Terraformから移行する場合は`tf2pulumi`とよばれるマイグレーションツールがあります。
https://github.com/pulumi/tf2pulumi

::: message alert
2023年6月にconvertサブコマンドが登場したことによりtf2pulumiは非推奨になりました。
https://www.pulumi.com/blog/converting-full-terraform-programs-to-pulumi/
:::

### VS AWS CDK
プログラミング言語でインフラストラクチャーを構成するIaCツールといえばAWS CDKがあります。AWS CDKとの違いについてPUlumiではこのように述べています。

| 項目 | Pulumi | AWS CDK |
| --- | --- | --- |
| サポート対象 | AWS以外にも60以上のクラウドやSaaSをサポート | AWSのみ |
| デプロイエンジン | Pulumi CLI（オープンソース） | AWS CloudFormation（プロプライエタリ） |
| テスト | ユニットテスト、プロパティテストなど一般的なテストフレームワークをサポート | 合成されたYAML/JSONに対してアサートするユニットテストのみ |
| 既存リソースインポート | 可 | プレビュー対応 |

AWS CDKはAWSに対応したプロビジョニングツールなので他のクラウドサービスには対応していないことが大きな違いです。またAWS CDKは既存リソースに対するインポート機能を正式にまだリリースしていません。
:::message
CDK for Terraform(CDKTF)[^5]がGAされたのでこの問題は改善されたかもしれません。
:::

機密情報の保持についてはSecrets Manager[^6]を使って格納するようにしていますが、Pulumiはマネージドなクラウドサービスが提供する暗号化キーを使い、暗号化し保護します。

[^1]: https://www.terraform.io/
[^2]: https://aws.amazon.com/jp/cdk/
[^3]: https://www.pulumi.com/docs/intro/vs/terraform/#dynamic-providers
[^4]:https://www.pulumi.com/docs/intro/languages/
[^5]:https://developer.hashicorp.com/terraform/cdktf
[^6]:https://aws.amazon.com/jp/secrets-manager/


# Pulumi AIとは
https://www.pulumi.com/ai/

大規模言語モデルのGPTを活用して、自然言語でPulumiのコードを自動生成してくれるAIアシスタントツールになります。無料で使えユーザー登録も不要で上のURLから作りたい構成を質問すればPulumiのソースコードが生成されます。GPTはv3とv4の2つが選択できるので無料でV4を使えるというのはChatGPTよりも優位性があるのではないかと思います。
![](/images/pulumi-ai-revolutionary/image9.png)

## Pulumi Challenges
Pulumiには世界中のユーザーと競ってハンズオンを体験した感想をブログやSNSで発信した人の中からSWAGをプレゼントしてくれる企画型のワークショップがあります。Pulumi AIを学べるワークショップもありましたので早速やってみることにしました。

### 始め方
ワークショップではAWSのKinesis Data Streams[^7]、RDS[^8]、DynamoDB[^9]、Timestream[^10]をデプロイするコードを生成しています。
必要なものはPulumiとAWSアクセスキー、シークレットキー、OpenAIのAPIキーになります。

まずPulumiをローカルにインストールします。OS毎にインストーラーが用意されているので、インストールは難しくないと思います。
https://www.pulumi.com/docs/get-started/install/

AWSのアクセスキー、シークレットキー、OpenAIのAPIキーをセットします。

```shell
export AWS_ACCESS_KEY_ID=<YOUR_ACCESS_KEY_ID>
export AWS_SECRET_ACCESS_KEY=<YOUR_SECRET_ACCESS_KEY>
export OPENAI_API_KEY=<YOUR_API_KEY>
```

Pulumiを実行するためのプロジェクトを作成します。ワークショップではPythonで実施していますが慣れ親しんだプログラミング言語で問題ないと思います。私はTypeScriptで新規プロジェクトを作成しました。

```terminal
$ pulumi new typescript
This command will walk you through creating a new Pulumi project.

Enter a value or leave blank to accept the (default), and press <ENTER>.
Press ^C at any time to quit.

project name: (pulumi-tutorial)
project description: (A minimal TypeScript Pulumi program)
Created project 'pulumi-tutorial'

Please enter your desired stack name.
To create a stack in an organization, use the format <org-name>/<stack-name> (e.g. `acmecorp/dev`).
stack name: (dev)
Created stack 'dev'

Installing dependencies...


added 182 packages, and audited 183 packages in 28s

56 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
npm notice
npm notice New minor version of npm available! 9.2.0 -> 9.6.5
npm notice Changelog: <https://github.com/npm/cli/releases/tag/v9.6.5>
npm notice Run `npm install -g npm@9.6.5` to update!
npm notice
Finished installing dependencies

Your new project is ready to go!

To perform an initial deployment, run `pulumi up`
```
プロジェクトを作成するとPulumiを実行するために必要なテンプレートファイルが作成されます。
![](/images/pulumi-ai-revolutionary/image2.png)

準備ができましたのでPulumi AIを使ってソースコードを生成していきます。ワークショップと一言一句同じプロンプトを使いました。
`Deploy a Kinesis Data Stream, RDS database, DynamoDB database, and Timestream database.`
![](/images/pulumi-ai-revolutionary/image3.png)

言語の違いはありますが私が生成されたコードは以下の通りでした。

```ts:index.ts
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// Kinesis Data Stream
const kinesisStream = new aws.kinesis.Stream("my-stream", {
    shardCount: 1,
});

// RDS database
const dbSubnetGroup = new aws.rds.SubnetGroup("my-dbsubnetgroup", {
    subnetIds: ["subnet-12345678", "subnet-abcdefgh"], // Replace with your subnet IDs
});

const dbInstance = new aws.rds.Instance("my-db-instance", {
    engine: "mysql",
    instanceClass: "db.t2.micro",
    allocatedStorage: 20,
    username: "admin",
    password: "your-db-password", // Replace with a real password
    dbSubnetGroupName: dbSubnetGroup.name,
    skipFinalSnapshot: true,
});

// DynamoDB database
const dynamoTable = new aws.dynamodb.Table("my-dynamodb-table", {
    attributes: [
        { name: "id", type: "S" },
    ],
    hashKey: "id",
    readCapacity: 1,
    writeCapacity: 1,
});

// Timestream database
const timestreamDatabase = new aws.timestreamwrite.Database("my-timestream-db", {
    databaseName: "mytimestreamdb",
});

// Stack exports
export const kinesisStreamArn = kinesisStream.arn;
export const dbInstanceEndpoint = dbInstance.endpoint;
export const dynamoTableName = dynamoTable.name;
export const timestreamDatabaseName = timestreamDatabase.databaseName;
```

このコードを`index.ts`に張り付けてデプロイを実行するとデプロイ途中でエラーが出ました。
![](/images/pulumi-ai-revolutionary/image5.png)

ワークショップでも出力された内容は違いますがエラーが出ること自体は想定内のようです。ワークショップでは出力されたエラーをそのままPulumi AIに張り付けてエラーを修正してもらっています。私も同じように出力されたエラーをそのまま張り付けて`fix this error`と記載してみました。
![](/images/pulumi-ai-revolutionary/image4.png)

するとPulumi AIがエラーの内容を分析して原因と修正方法を提示してくれます。今回のエラー内容はDBサブネットを新規作成する際に既存のサブネットを指定している箇所があるのですが存在するサブネットIDを記載していないことが原因でした。
修正されたコードではわかりやすく既存のサブネットIDを指定するように修正されたコードを提示されました。

```ts:index.ts
// RDS database
const dbSubnetGroup = new aws.rds.SubnetGroup("my-dbsubnetgroup", {
    subnetIds: ["your-subnet-id-1", "your-subnet-id-2"], // Replace with your subnet IDs
});
```

既存サブネットIDを記載してもう一度デプロイすると今度はすべてのデプロイがうまくいきAWSリソースの作成に成功しました。
![](/images/pulumi-ai-revolutionary/image6.png)

これでこのチャレンジは成功です。リソースは`pulumi destroy -rf --remove`で強制的に削除できます。
このワークショップについてブログで発信していますが、日本在住の私にSWAGは貰えるのでしょうか🤔

[^7]: https://aws.amazon.com/jp/kinesis/data-streams/
[^8]: https://aws.amazon.com/jp/rds/
[^9]: https://aws.amazon.com/jp/dynamodb/
[^10]: https://aws.amazon.com/jp/timestream/

# 所感
Pulumi AIを使ってPulumiのIaCを体験してみました。気になった点ですが少し目を離すと接続がすぐに途切れてしまいスレッドが最初からやり直しになることが多々ありました。
![](/images/pulumi-ai-revolutionary/image8.png)
履歴を保存してくれるわけではないので一度やり直しになると作成途中のソースコードの質問ができなくなるので、もう少し接続時間を伸ばしてほしいという気持ちがあります。

やや挑戦的なタイトルになってしまいましたが、個人的にはもう少し今後のアップデートに期待したいと思いました。
PulumiはTerraformやAWS CDKと比べると知名度が弱いと思いますが、2つの良いところを取り入れてる部分もありますので、気になりましたら採用を検討してみてください。

# 参考文献
https://www.pulumi.com/blog/pulumi-ai
https://www.publickey1.jp/blog/23/pulumipulumi_aiawsazurecloudflarekubernetesdatadog130infra-as-code.html