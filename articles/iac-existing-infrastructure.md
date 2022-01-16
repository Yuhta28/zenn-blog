---
title: "既存インフラをコード化する時に気を付けること"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","Terraform","IaC"]
published: false
---

:::message
新年最初のJAWS-UGイベントに参加したときに、発表した内容をより詳しく補足している記事になります。
:::

https://jawsug-bgnr.connpass.com/event/235497/

# 概要

昨年の12月から社内のAWSインフラシステムをコード管理(IaC=Infrastructure as Code)しようという動きが始まりました。
新規リソースをTerraformやAWS CDKなどを活用してコード化する事例は数多く紹介されていましたが、意外と既存インフラをコード化する方法については数が少なく結構大変なことがありました。
この記事では私が直面した既存インフラのコード化でハマった点について紹介していきます。

# 背景
2020年まで社内のインフラチームは2人しかおらず、うち1人はマネージャーで基本的にビジネス職とのコミュニケーションがメインだったため実質1人でインフラ構築や運用、新規メディアサーバーの作成を担当していました。
1人で検証環境のEC2インスタンス含め数百台のサーバーを運用管理、新規メディアのサーバーを構築していました。
当然時間も足りず、スピードを優先してAWSコンソール画面からの操作でリソース構築していました。

![](/images/iac-existing-infrastructure/image1.png)

昨年から人数も増え、新しいマネージャーがジョインされたことでチームとして体制が整いサーバー構築従来よりも早く作れるようになったことからIaCに取り組もうということになりました。
マネージャーからはチームに適したIaCツールの選定と運用方法について考えてほしいと頼まれましたので、12月は各種ツールの技術選定と検証を行ないました。

# 技術選定調査
まずはAWSリソースをIaCする場合どんなツールが使われているのか調べてみました。
その中で以下のツールが他社事例や日本語記事も多くありましたので、学習コストの敷居の低さからこの3つの中から検証してみました。

- AWS CloudFormation
- AWS CDK
- Terraform

また選定基準としては、既存リソースをコード化する場合の難易度、コードの可読性、将来的な運用の拡張性をポイントにしました。
将来的な運用の拡張性ですが、今後Git上にCI/CD環境基盤に構成管理ツールを実装して、`git push`したら自動デプロイされて、インフラリソースが更新されるようになるようにしたいと考えていました。

## AWS CloudFormation
https://aws.amazon.com/jp/cloudformation/
CloudFormationは昨年のメディアサーバー構築でも部分的に使われていて、AWSリソースだけをIaCしたい場合はピッタリなツールです。
ただ既存リソースをインポートする場合、事前にリソースに合ったテンプレートファイルを用意しなければいけないということがあり、既存リソースのコード化には難しいと感じました。

## AWS CDK
https://aws.amazon.com/jp/cdk/
AWS CDKはプログラミング言語を使用してAWSリソースを構築してくれます。
2022年1月現在は以下のプログラミング言語が対応されています。

- JavaScript
- TypeScript
- Python
- Java
- C#
- Go(preview)

ただチームで対応されている言語を扱ったことのあるメンバーが少なく、私自身もプログラミングがあまり得意ではなかったので、AWS CDKを使ってIaCを行なうことは断念しました。
またAWS CDKには既存リソースをインポートする機能がなく、OSSの[former2](https://former2.com/)を使えば実現できますが、L1 Constructという抽象度の低いコードが生成されてしまうため、CDKの恩恵が受けづらいと感じました。[^1]
(あと、生成されるコードが1系でそのまま`cdk`コマンドを実行すると失敗したのですが、あまりPythonに詳しくない私では原因がわからなかったこともあります)
AWS CDKには下記の日本語のワークショップも用意されていて、少ない記述で簡単にAWSリソースを構築できたのは感動しましたので、後述のCDK for Terraformで再挑戦したみたいと思います。

https://catalog.us-east-1.prod.workshops.aws/v2/workshops/99731164-1d19-4d2e-9319-727a130e2d57/ja-JP/

[^1]: https://dev.classmethod.jp/articles/aws-cdk-layer/

## Terraform
https://www.terraform.io/
最終的にTerraformを使ってIaC化を実現することが今のチームにもっとも適していると思いました。
Terraformには`import`サブコマンドを使って既存リソースのインポートができます。
ただ`import`コマンドはEC2やVPC、RDSなど複数のリソースをまとめてインポートできないため、対象リソースの数が多いと辛くなるという問題がありました。

https://zenn.dev/yuta28/articles/terraform-gha
もう一つ面白そうだと思ったのが、CDK for TerraformというCDKを使ってTerraformを実行できる機能です。
https://www.terraform.io/cdktf
まだベータ版ですので本番運用するのは難しそうですが、これがGAされればTerraformからAWS CDKへの移行も容易になるのかなと思います。

# ハマった点
ただそれでも既存リソースでハマった点が出てきましたので、その部分について紹介します。

# 参考文献
https://beyondjapan.com/blog/2020/05/terraform-resource-import/
https://beyondjapan.com/blog/2020/05/terraformer-import-existing-infrastructure/