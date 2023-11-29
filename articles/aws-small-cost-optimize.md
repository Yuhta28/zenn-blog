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

AWSのベストプラクティスを定めたWell-Architected[^2]フレームワークでも独立した章がある、AWSコスト最適に特化した書籍があるなどコスト最適化はAWS上でサービスを運用していく中で意識していかなければならない分野です。

https://aws.amazon.com/jp/aws-cost-management/cost-optimization/

https://www.kadokawa.co.jp/product/322104000266/

とはいってもAWSコスト最適化の方法は多岐にわたり、どれから手を付けていけばいいかわからないこともあります。
まずは個人からでも気軽に始められる余剰コンピューティングリソースの自動起動停止に焦点を絞ったコスト削減方法について紹介していきます。

[^1]: https://jp.tradingview.com/symbols/USDJPY/
[^2]: https://aws.amazon.com/jp/architecture/well-architected/?wa-lens-whitepapers.sort-by=item.additionalFields.sortDate&wa-lens-whitepapers.sort-order=desc&wa-guidance-whitepapers.sort-by=item.additionalFields.sortDate&wa-guidance-whitepapers.sort-order=desc

# 対象読者

- AWSコストについて関心がある
- EC2、RDS、ECSの稼働時間を楽に減らしたい

# EC2のコスト最適化

EC2やRDS、ECSなどのコンピューティングリソースをプロビジョニングさせて課金させるリソースは、夜中や早朝の時間帯も起動させたままでは料金が発生してしまいます。
本番稼働させているものでしたら24時間稼働させ続けますが、業務時間中しか使わない検証環境のコンピューティングリソースも24時間稼働させたままにしておくのは非常にもったいないコストです。

しかし、毎日の業後に検証環境のコンピューティングリソースを一台一台停止するのも運用が辛く万が一誤って本番環境のリソースを停止させてしまっては大問題です。

以前Instance Scheduler[^3]を使ったリソース稼働時間の自動起動停止の方法について[紹介](https://zenn.dev/yuta28/articles/ec2-schedule)しましたが、その後Systems Managerでもっと簡単にEC2の自動起動停止を実現できるResource Schedulerが登場しました。

https://aws.amazon.com/jp/about-aws/whats-new/2022/12/aws-systems-manager-quick-setup-resource-scheduler/

Resource SchedulerはInstance Scheduler程自由にカスタマイズはできませんが、シンプルに設定できるのでAWSを始めたばかりの人にもおススメのサービスになります。
Systems Managerのコンソール画面の左サイドバーから高速セットアップという項目がありますのでこちらをクリックします。
![](/images/aws-small-cost-optimize/image2.png)

高速セットアップには運用業務を楽にしてくれるいくつかのソリューションが提供されており、その中にResource Schedulerも含まれています。
![](/images/aws-small-cost-optimize/image3.png)

Resource Schedulerでは対象にしたいEC2のタグを指定し、EC2の起動時刻、停止時刻、適用曜日を設定できます。
![](/images/aws-small-cost-optimize/image4.png)

残念ながら2023年11月時点ではタグは1つしか設定できず、複数タグによるAND条件やOR条件でのタグフィルタリングはできません。

[^3]: https://aws.amazon.com/jp/solutions/implementations/instance-scheduler-on-aws/

# 参考文献
https://dev.classmethod.jp/articles/ssm-quick-setup-resource-scheduler/