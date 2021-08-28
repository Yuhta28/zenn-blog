---
title: "AWSのSavings Plansに気を付けよう"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","EC2","コスト最適化"]
published: true
---
# 概要
この記事は実質的に[RDSのリザーブドインスタンスに気を付けよう](https://zenn.dev/yuta28/articles/rds-attention)の続きもの、というか執筆後に発覚したやらかしを記述したものになります。
Savings Plans(SP)の詳しい説明は先の記事をご覧いただき、ここではSPで私がやってしまったミスについて紹介いたします。

# 事象
前の記事でも紹介したとおり6/25に会社で運用しているRIの有効期限が切れますので、新しく更新する必要がでてきました。
弊社では今年からEC2はRIからSPに切り替える運用にしてきましたので、順次有効期限が切れたEC2インスタンスを、コンピューティングリソース分新しくSPで申請しています。
前回の記事も書いたとおりSPは事前予約もできますので、6月中旬頃に7/1に適用できるように購入しました。
![](https://storage.googleapis.com/zenn-user-upload/8164da74ee389aaa8bd9e4ec.png)

そして７月になったので利用明細を確認したのですが、６月に利用されていることになっていました。
![](https://storage.googleapis.com/zenn-user-upload/67af7b07bb814aea2371e6b6.png)

# 原因
原因ですがAWSは請求の時間を協定世界時刻(UTC)を基準に計算しています。[^1]
> Q. AWSの請求の時間はJST/日本標準時ですか？
> いいえ、協定世界時（UTC）を基準に請求金額を計算しています。
> 日本標準（JST）と協定世界時（UTC）との時差は9時間となります。
> 4月1日8時台（JST）のご利用料金は3月分の料金として計上され、
> 4月1日9時（JST）のご利用料金から4月分の料金として計上されます。

[^1]: https://aws.amazon.com/jp/aws-jp-faq/

私は、日本時間の7/1の0時からプランを適用させたかったのでUTCは6/30 15:00から適用開始するように購入しました。

そのためAWSは6月に利用したと判断してSPの利用明細が6月利用となったということでした。

# 所感
日本で生活していると当たり前のようにJST基準で時刻を決めますが日付の変わり目は特に気を付けたほうが良さそうです。

# 追記(2021/8/28)
友人から教えてもらいましたが、EC2のRIは日付指定はできても時刻指定はできないとのことでした。日付を指定した場合、GMTの0:00つまり日本時間の朝9時にRIが適用されるということです。

RDSのRIとEC2のRI、SPの三者はそれぞれでできる部分、できない部分が微妙に異なっていましたので図で整理します。

#### 比較表

| | RDS RI | EC2 RI | EC2 SP |
| --| -- |  -- | --|
| 日付指定 | × | ○ | ○ |
| 時刻指定 | × | × | ○|

これは意外と見落としがちな罠だと思いますので、今後RI/SPを申請するときは気を付けたほうがよさそうです。

## 追記の追記
EC2のRIですが、AWS CLIからでしたら時刻指定ができるみたいです。

https://twitter.com/ohsawa0515/status/1431467827249090560?conversation=none

AWS CLIのドキュメントを参照しますと、`purchase-reserved-instances-offering`コマンドのこのオプションを使えば時刻指定ができるみたいです。

--purchase-time (timestamp)
> The time at which to purchase the Reserved Instance, in UTC format (for example, YYYY -MM -DD T*HH* :MM :SS Z).

```
purchase-reserved-instances-offering
--instance-count <value> \
--reserved-instances-offering-id <value> \
--purchase-time <value>
```

AWSはコンソールではできないこともCLIならできることが多々あります。
ちなみにRDSのRIもCLIなら日付指定できるかなと思って調べましたが、残念ながらCLIでもRDSのRIは日付指定できないみたいです。

いつかRDSのRIも日時の指定ができることを祈っています。
# 参考文献
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-scheduled-instances.html
https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/purchase-reserved-instances-offering.html
https://awscli.amazonaws.com/v2/documentation/api/latest/reference/rds/purchase-reserved-db-instances-offering.html