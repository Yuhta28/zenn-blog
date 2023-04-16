---
title: "GAされたAmazon CodeWhispererを試してみる"
emoji: "😮‍💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","codewhisperer"]
published: false
---

# 概要
「Amazon re:MARS 2022[^1]」で発表されプレビュー版だったAmazon CodeWhispererが一般公開[^2]されました。
プレビュー版の時から気になってはいましたが、触らないままGAされましたので、興味があるうちに使ってみて感想を述べていきたいと思います。

[^1]: https://remars.amazonevents.com/
[^2]: https://aws.amazon.com/jp/about-aws/whats-new/2023/04/amazon-codewhisperer-generally-available/

# CodeWhispererについて
https://aws.amazon.com/jp/codewhisperer/

CodeWhispererはコーディング時にAIによるオートコンプリート機能を提供したコード自動生成サービスになります。(分かりやすくいうとAWS版GitHub Copilotになります)
GAされましたが使えるリージョンは今のところバージニアリージョンのみのようです。
![](/images/codewhisperer-tutorial/image1.png)
ただCodeWhispererはVS CodeやJetBrainsでも使えるのであまりリージョンのことは気にする必要はないかと思います。そのほかLambda関数やCloud9上でもCodeWhispererを有効化できます。

# 価格について
プレビュー版の時は無料で利用できたみたいですが、GAされたことで料金体系が正式に決まりました。
現在2つのプランが提供されており、個人プランとプロフェッショナルプランになります。
https://aws.amazon.com/jp/codewhisperer/pricing/

個人プランでしたら**無料**で利用できます。プロフェッショナルプランは1ユーザーあたり$19/月になります。

## 個人プランとプロフェッショナルプランの違い
基本的な機能に大きな違いはありません。細かいところではコードセキュリティスキャンの実行回数制限やログイン方法、AWS Organization配下のポリシー管理ができるかどうかになります。

|プラン|個人|プロフェッショナル|
|:---|:---:|:---:|
|料金|無料|$19/月|
|ログイン方法|Builder ID| IAM Identity Center|
|対応言語| すべて | すべて|
|コードセキュリティスキャン| 1ユーザー50回/月 | 1ユーザー500回/月 |
|組織によるライセンス管理| 不可 | 可能|
|組織による方針管理| 不可 | 可能|

個人プランでも今後のアップデートでIAM Identity Centerによるログインは予定されているみたいです。
> In the future, it will be possible for an individual developer to become a professional developer, by connecting their Builder ID credentials with IAM Identity Center credentials from their employer. 
訳:将来的には、開発者個人がBuilder IDのクレデンシャルと雇用主のIAM Identity Centerのクレデンシャルを結びつけることで、プロの開発者になることができるようになります。

https://docs.aws.amazon.com/codewhisperer/latest/userguide/user-types.html

GitHub Copilotと異なり、コード生成される言語は2023年4月時点で数が決まっているみたいで以下の言語がサポートされています。

- Python
- Java
- JavaScript
- TypeScript
- C#
- Rust
- Go
- Ruby
- Scala
- Kotlin
- PHP
- C
- C++
- Shell Script
- SQL

とは言えプレビュー版の時は、Python、Java、JavaScript、TypeScript、C#の5つのみでしたのでGAされて大幅に増えたのは嬉しいです。

:::message
ちなみにプレビュー版の頃からサポートされている言語のほうが学習リソースが多かったためか精度と質がいいみたいです。[^3]
:::

[^3]: https://dev.classmethod.jp/articles/amazon-codewhisperer-ga/

# VS Codeへの導入
VS Codeに導入してみます。ローカルのIDEにCodeWhispererを導入する場合AWSアカウントは必要でありません。個人プランではAWS Builder IDを利用しますので事前に作成しておきます。
詳しい手順についてはドキュメントをご覧ください。
https://docs.aws.amazon.com/codewhisperer/latest/userguide/whisper-setup-indv-devs.html

:::message

##### AWS Builder IDって?
AWS Builder ID[^4]はAWS上で構築する人のための新しい個人プロファイルです。会社ではAWSアカウント自体、皆さん共通のアカウントを使い、IAMユーザー毎に認証情報を持っていると思いますが、AWS Builder IDは開発者1人1人に紐づきます。
今のところAWS BUilder IDを利用するサービスは以下の通りです。
- Amazon CodeCatalyst[^5]
- AWS re:Post[^6]
- Amazon CodeWhisperer
:::

VS CodeをAWS Builderと紐づけましたら利用可能になります。
![](/images/codewhisperer-tutorial/image2.png)

[^4]: https://docs.aws.amazon.com/signin/latest/userguide/sign-in-aws_builder_id.html
[^5]: https://codecatalyst.aws/explore
[^6]: https://repost.aws/ja

# コード生成
試しにEC2インスタンスを作成するAWS CDK(TypeScript)を生成してみました。
最初のコメント部分が私が書いた日本語です。改行した後CodeWhispererが動いて次々とコードを生成してくれます。ただバージニアリージョンのせいかわかりませんが、動作がもっさりしていてコード生成も1行ずつの出力で毎回改行Enterを押していたのでGitHub Copilotほど爽快感のあるコード生成体験はできませんでした。
![](/images/codewhisperer-tutorial/demo1.gif)
*もう少し速く出力したい*

# 参考文献
https://docs.aws.amazon.com/codewhisperer/latest/userguide/what-is-cwspr.html
https://dev.classmethod.jp/articles/amazon-codewhisperer-ga/
https://dev.classmethod.jp/articles/aws-repost-builder-id/