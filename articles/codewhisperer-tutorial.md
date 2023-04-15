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
現在2つのプランが提供されており、個人利用とプロフェッショナル利用になります。
https://aws.amazon.com/jp/codewhisperer/pricing/

個人利用でしたら**無料**で利用できます。プロフェッショナル利用は1ユーザーあたり$19/月になります。

## 個人利用とプロフェッショナルの違い
基本的な機能に大きな違いはありません。細かいところではコードセキュリティスキャンの実行回数制限やログイン方法、AWS Organization配下のポリシー管理ができるかどうかになります。

|プラン|個人|プロフェッショナル|
|:---|:---:|:---:|
|料金|無料|$19/月|
|ログイン方法|Builder ID| IAM Identity Center|
|対応言語| すべて | すべて|
|コードセキュリティスキャン| 1ユーザー50回/月 | 1ユーザー500回/月 |
|組織的ライセンス管理| 不可 | 可能|
|組織的方針管理| 不可 | 可能|

複数アカウントを運用する場合AWS Organizationを利用する企業は多いと思いますが、組織内のどのユーザーがCodeWhispererを利用できるかはを一元的に管理したい場合はプロフェッショナルプランを利用するほうが良さそうです。

# 参考文献
https://docs.aws.amazon.com/codewhisperer/latest/userguide/what-is-cwspr.html
https://dev.classmethod.jp/articles/amazon-codewhisperer-ga/