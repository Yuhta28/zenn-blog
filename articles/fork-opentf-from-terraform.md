---
title: "TerraformからforkされるOpenTFがパブリックになりました"
emoji: "🍴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["OSS","Terraform","OpenTF"]
published: false
---

::: message alert
OpenTFは2023年9月にパブリックになった新しいOSSです。この記事は2023/9/9時点の情報を基にしたものになります。
:::
# 概要
2023年8月にTerraformなどのOSSを開発しているHashiCorp社は自社製品のライセンスを変更することを発表しました。
https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license
以前もこのことについて[記事](https://zenn.dev/yuta28/articles/migration-terraform-to-pulumi)で軽く触れていますが、このライセンス変更によってTerraformなどを利用するHashiCorpユーザーに直接の影響はありません。

ただHashiCorp社のライセンス変更に反対する動きもありTerraformからフォークされた真のオープンソースを目指してるという団体が出てきました。
それがOpenTF Foundationです。
https://opentf.org/

OpenTF Foundationは上記マニフェスト(宣言書)によるとTerraformを真のオープンソースであり続けるためにHashiCorp社にライセンス変更の撤廃を要求、それができないならTerraformからフォークしたOSSであるOpenTFを管理することを宣言しています。

そして2023/9/6にOpenTFがフォークされパブリック利用できるようになりました。
https://twitter.com/opentforg/status/1699076153968095494?s=20

OpenTFはどのようなOSSなのか、またなぜTerraformからフォークしようとしたのかマニフェストの中身を見てみます。

# 対象読者
- OpenTFについて知りたい人

# オープンソースのおさらい
上記のマニフェストの中で何回か真のオープンソースという言葉(truly open source)が出てきました。
真のオープンソースというのはどういう意味なのかよく分からなかったので、オープンソースについて定義を確認してみました。
日本国内ではOpen Source Group Japan(OSG-JP)[^1]がオープンソースという用語を商標登録[^2]しています。
では、商標登録しているからオープンソースという言葉をビジネスの場で使う際に何かしらの制約が発生するのでしょうか？OSG-JPは以下のようにオープンソースという言葉は誰でも自由に使えることを認めています。その中でオープンソースの定義を満たした状態でオープンソースという言葉を使ってほしいと記載されています。

> オープンソース / Open Sourceという言葉の利用に 関して、我々が利用料を徴収したり、他に何らかの制限を加えるといった ことは一切行いません。Open Source Group Japanでは、 オープンソース / Open Source商標を誰もが自由に使うことを認めます。 ただし、我々はOSIの定める「オープンソースの定義」に沿う形で オープンソースという言葉が利用されることを希望しています。

オープンソースの定義とはどのようなものでしょうか。こちらにオープンソースの定義について紹介されています。[^3]
https://opensource.jp/osd/osd19/

訳された日が古かったので念のため原文のページも置いておきます。
https://opensource.org/osd/

以下の基準を満たしたものをオープンソースのプログラムと呼ばれます。

- 再頒布の自由
- ソースコード


[^1]: https://opensource.jp/
[^2]: https://opensource.jp/osg/trademark/
[^3]: 2004年2月21日時点の日本語訳

https://github.com/opentffoundation/manifesto


# 参考文献
https://ja.wikipedia.org/wiki/%E3%82%AA%E3%83%BC%E3%83%97%E3%83%B3%E3%82%BD%E3%83%BC%E3%82%B9%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E3%81%AE%E6%AD%B4%E5%8F%B2
https://zenn.dev/koduki/articles/45f65a5318f019
