---
title: "TerraformからforkされるOpenTFがパブリックになりました"
emoji: "🍴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["OSS","Terraform","OpenTF"]
published: false
---

::: message alert
OpenTFは2023/9/9時点でアルファ版リリース前です。この記事は2023/9/9時点の情報を基にしてます。
:::
# 概要
2023年8月にTerraformなどのOSSを開発しているHashiCorp社は自社製品のライセンスを変更することを発表しました。
https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license
以前もこのことについて[記事](https://zenn.dev/yuta28/articles/migration-terraform-to-pulumi)で軽く触れていますが、このライセンス変更によってTerraformなどを利用するHashiCorpユーザーに直接の影響はありません。

ただHashiCorp社のライセンス変更に反対する動きもありTerraformからフォークされた真のオープンソースを目指してるという団体が出てきました。
それがOpenTF Foundationです。
https://opentf.org/

OpenTF Foundationは上記マニフェスト(宣言書)によるとTerraformを真のオープンソースであり続けるためにはHashiCorp社へライセンス変更の撤廃を要求、それができないならTerraformからフォークしたOSSであるOpenTFを管理すると宣言しています。

そして2023/9/6にOpenTFがフォークされました。
https://twitter.com/opentforg/status/1699076153968095494?s=20

OpenTFはどのようなOSSなのか、またなぜTerraformからフォークしようとしたのかマニフェストの中身を見てみます。

::: message
紛らわしいのでOpenTF Fundationは以降、財団と表記します。
:::

# 対象読者
- OpenTFについて知りたい人
- 財団の主張について知りたい人

# オープンソースのおさらい
上記マニフェストの中で何回か真のオープンソースという言葉(truly open source)が出てきました。
真のオープンソースというのはどういう意味なのかよく分からなかったので、オープンソースについて定義を確認してみました。
日本国内ではOpen Source Group Japan(OSG-JP)[^1]がオープンソースという用語を商標登録[^2]しています。
では、商標登録しているからオープンソースという言葉をビジネスの場で使う際に何かしらの制約が発生するのでしょうか？OSG-JPは以下のようにオープンソースという言葉は誰でも自由に使えることを認めています。その中でオープンソースの定義を満たした状態で「オープンソース」という言葉を使ってほしいと記載されています。

> オープンソース / Open Sourceという言葉の利用に 関して、我々が利用料を徴収したり、他に何らかの制限を加えるといった ことは一切行いません。Open Source Group Japanでは、 オープンソース / Open Source商標を誰もが自由に使うことを認めます。 ただし、我々はOSIの定める「オープンソースの定義」に沿う形で オープンソースという言葉が利用されることを希望しています。

オープンソースの定義とはどのようなものでしょうか。こちらにオープンソースの定義について紹介されています。[^3]
https://opensource.jp/osd/osd19/

訳された日が古かったので念のため原文のページも置いておきます。
https://opensource.org/definition-annotated/

以下の基準を満たしたものをオープンソースのプログラムと呼ばれます。

- 再頒布の自由
- ソースコード
- 派生ソフトウェア
- 作者のソースコードの完全性
- 個人、グループに対する差別の禁止
- 利用する分野に対する差別の禁止
- ライセンスの分配
- 特定製品でのみ有効なライセンスの禁止
- 他のソフトウェアを制限するライセンスの禁止
- ライセンスは技術中立でなければならない


# HashiCorp社のライセンス変更による影響

HashiCorp社が新たに採用したライセンス、Business Source License(BSL)[^4]はソースコードを公開していますが、商用利用を制限するライセンスです。利用を制限するライセンスのためBSLはオープンソースライセンスではないとライセンス元も明記しています。

> The BSL does not meet the Open Source Definition (OSD) maintained by the Open Source Initiative (OSI). [^5]

ただしHashiCorp社は競合する企業のプロダクトにTerraformを組み込むなどの利用用途以外に関しては商用利用を許可すると明記しています。

> All non-production uses are permitted. All production uses are allowed other than hosting or embedding the software in an offering competitive with HashiCorp commercial products, hosted or self-managed.[^6]

ここでいう組み込み(embedded)についてもHashiCorp社は以下のように回答しています。

> What does the term “embedded” mean under the HashiCorp BSL license?
Under the HashiCorp BSL license, the term “embedded” means including the source code or object code, including executable binaries, from a HashiCorp product in a competitive product. “Embedded” also means packaging the competitive product in such a way that the HashiCorp product must be accessed or downloaded for the competitive product to operate.

かいつまんで訳すとHashiCorp社のプロダクト内のソースコードを競合プロダクトのソースコード内に組み込むことを`embedded`と定義しているようです。

また勘違いしてはいけないのは、HashiCorpの競合プロダクトを作る場合でも、Terraformと競合するプロダクトをセキュアにすることを目的に同社のVaultを使う、Vaultと競合するプロダクトを構築するためにTerraformでデプロイすることは問題ないとのことです。[^7]

[^1]: https://opensource.jp/
[^2]: https://opensource.jp/osg/trademark/
[^3]: 2004年2月21日時点の日本語訳
[^4]: https://mariadb.com/bsl11/
[^5]: https://mariadb.com/bsl-faq-adopting/#osl
[^6]: https://www.hashicorp.com/license-faq#who-is-impacted
[^7]: https://www.hashicorp.com/license-faq#competitive-product-bsl-coverage

# 財団の主張

先のマニフェストで財団はHashiCorp社に対して、BSLはTerraformにとって毒薬だという強い言葉で非難しています。
> Our concern: the BUSL license is a poison pill for Terraform.

現時点でTerraformを利用するエンドユーザーやインテグレーションパートナーに対しては影響のないライセンス変更だとしても今後HashiCorp社がライセンス条項を変更したり、競合の解釈を変更することで自由に使えなくなるのではないのかと主張しています。

開発者、企業はインフラストラクチャーを管理するツール選定時にTerraformを選定から外すことで既存のTerraformでのIaCは負債となり、コミュニティも衰退することを危惧しています。

なので冒頭の主張に戻りますが財団はHashiCorp社へライセンス変更の撤廃を要求、それができないならTerraformからフォークしたOpenTFを管理すると述べています。

マニフェストの下部にはQ&Aがまとめられています。私が興味深いと感じた質問はこちらです。

> Didn't HashiCorp adopt BUSL to deter vendors who were using Terraform but not contributing back? 
訳)HashiCorpがBUSLを採用したのは、Terraformを使っていながら貢献しないベンダーを抑止するためではなかったのか？

HashiCorp社がライセンス変更した背景にはOSSにフリーライドするベンダーの存在がありました。オープンソースコミュニティに貢献せず自らの商業目的のためだけにOSSを利用することはオープンソースの精神に反するからだと[ブログ](https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license)に記載していました。

一方、HashiCorpの主張に対して財団はこう反論しています。

> This is inaccurate and misleading. First of all, many of the vendors affected by the change to BUSL have made considerable contributions to the Terraform community.
訳)これは不正確で誤解を招く。第一にBSLへの変更によって影響を受ける多くのベンダーはTerraformコミュニティに多大なる貢献をしてきた。

貢献してきた内容としてTerraformのバイナリやモジュール、Terragrunt[^8]やtflint[^9]などのサードパーティーツール、有志が作成した学習リソースなどを例に挙げています。

TerraformはLinuxやGoなどツールのように大きなエコシステムの一部であり、HashiCorp社以外のベンダーがコミュニティに貢献したからこそ発展してきた。
これはTerraformがオープンソースだからこそ発展してきたものであり、ベンダーの働きを認めないのは悪いことだと述べています。

またベンダーからの貢献に対して、Terraform CloudやTerraform Enterpriseへのマネタイズ誘導ができないからissueやPRを対応しきれていないことも問題と主張しています。


#### Terraformのマネタイズに関して
少し話がそれますがHashiCorp社がライセンスを変更した背景にはTerraformのマネタイズが上手くいっていないからではないか？と主張する記事もあります。
https://www.theregister.com/2023/08/28/opentf_forks_terraform_code/

Terraformのステート管理を効率化してくれるアメリカのSaaS企業Scalr[^10]のCEOであるStadil氏は記事内でこう述べています。
> Asked why HashiCorp opted for a change of license, Stadil claimed: "The reality here is that HashiCorp is not executing well on its Terraform cloud product. And because they're not executing well, other competitors like my company Scalr and a number of others have stepped in."
訳)HashiCorpがライセンス変更を選択した理由を尋ねられたStadil氏は、次のように主張した：「HashiCorpはTerraform Cloudでうまくいっていないというのが現実だ。HashiCorpがTerraform Cloudをうまく使いこなせていないため、私の会社Scalrやその他多くの競合他社が参入してきたのです」

正直なところTerraform Cloudは課金プランが変更されたことでやや使いにくくなりました。以前までなら1チーム5人までなら無料でTerraform Cloudを利用できましたが、新しい課金プランでは500リソースまで無料と人単位からリソース単位に課金体系が変更されました。500リソースだと直ぐに無料枠を超えてしまうため小規模な会社でも有料プランに切り替えたところは多いのではないかと思います。

[^8]: https://terragrunt.gruntwork.io/
[^9]: https://github.com/terraform-linters/tflint
[^10]: https://www.scalr.com/


# OpenTF

https://opentf.org/fork
2023/9/6にフォークされたOpenTFはすでに5,000以上ものスターを獲得しています。
![](/images/fork-opentf-from-terraform/image1.png)

2023/9時点でOpenTFはローカルでのテストと開発をサポートしています。

# 参考文献
https://ja.wikipedia.org/wiki/%E3%82%AA%E3%83%BC%E3%83%97%E3%83%B3%E3%82%BD%E3%83%BC%E3%82%B9%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E3%81%AE%E6%AD%B4%E5%8F%B2
https://zenn.dev/koduki/articles/45f65a5318f019
https://mariadb.com/ja/resources/blog/mariadb-bsl/
