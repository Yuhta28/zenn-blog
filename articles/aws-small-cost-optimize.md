---
title: "小さなことから始めるAWSコスト最適入門"
emoji: "💰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","コスト最適化","systemsmanager","eventbridge"]
published: false
---

# 序論
AWSをドル建てで支払う場合為替レート変更による料金の増減があります。
昨今円安の影響で何もしていないにも関わらずどんどんAWSのコストが上がっていき頭を悩ますエンジニアも多いと思います。
![チャート図](/images/aws-small-cost-optimize/image1.png)
*ドル円チャート[^1]*

AWSのベストプラクティスを定めたWell-Architected[^2]フレームワークでも独立した章がある、AWSコスト最適に特化した書籍があるなどAWS上でサービスを運用していく中で意識していかなければならない分野です。

https://aws.amazon.com/jp/aws-cost-management/cost-optimization/

https://www.kadokawa.co.jp/product/322104000266/

とはいってもAWSコスト最適化の方法は多岐にわたり、どれから手を付けていけばいいかわからないこともあります。
まずは個人からでも気軽に始められる余剰コンピューティングリソースの自動起動停止に焦点を絞ったコスト削減方法について紹介していきます。

[^1]: https://jp.tradingview.com/symbols/USDJPY/
[^2]: https://aws.amazon.com/jp/architecture/well-architected/?wa-lens-whitepapers.sort-by=item.additionalFields.sortDate&wa-lens-whitepapers.sort-order=desc&wa-guidance-whitepapers.sort-by=item.additionalFields.sortDate&wa-guidance-whitepapers.sort-order=desc

# 対象読者

- AWSコストについて関心がある
- EC2、RDS、ECSの稼働時間を楽に減らしたい

