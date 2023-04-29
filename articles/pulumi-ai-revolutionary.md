---
title: "PulumiはIaCの革命児になれるか"
emoji: "☠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","ai"]
published: false
---

# 概要
2023年4月にPulumi社はPulumiサービスにAIを組み込んだ機能を実装したと発表しました。
https://twitter.com/PulumiCorp/status/1646530231639867392?s=20

ここ最近、コード生成系AIが増えてきましたがIaCに特化したツールは珍しいと思いましたので触って感想を述べていきます。

# Pulumiとは
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

### VS AWS CDK
プログラミング言語でインフラストラクチャーを構成するIaCツールといえばAWS CDKがあります。AWS CDKと違いについてPUlumiではこのように述べています。

| 項目 | Pulumi | AWS CDK |
| --- | --- | --- |
| サポート対象 | AWS以外にも60以上のクラウドやSaaSをサポート | AWSのみ |
| デプロイエンジン | Pulumi CLI（オープンソース） | AWS CloudFormation（プロプライエタリ） |
| テスト | ユニットテスト、プロパティテストなど一般的なテストフレームワークをサポート | 合成されたYAML/JSONに対してアサートするユニットテストのみ |
| 既存リソースインポート | 可 | プレビュー対応 |

[^1]: https://www.terraform.io/
[^2]: https://aws.amazon.com/jp/cdk/
[^3]: https://www.pulumi.com/docs/intro/vs/terraform/#dynamic-providers
[^4]:https://www.pulumi.com/docs/intro/languages/





# 所感

PulumiはTerraformやAWS CDKに比べると知名度が弱い気がしますが、2つの良いところを取り入れてる部分もありますので、是非今回のアップデートを機にIaCツール採用に検討してみてください。

# 参考文献
https://www.pulumi.com/blog/pulumi-ai