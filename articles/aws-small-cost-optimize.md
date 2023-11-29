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

作成するとResource Schedulerのデプロイが行なわれ、条件に一致したタグがついたEC2を対象に起動停止のスケジューリングが実行されます。
![](/images/aws-small-cost-optimize/image5.png)
![](/images/aws-small-cost-optimize/image6.png)


ただ残念ながら2023年11月時点ではタグは1つしか設定できず、複数タグによるAND条件やOR条件でのタグフィルタリングはできません。またInstance SchedulerはRDSも含めて自動起動停止が設定できましたが、Resource Schedulerは現状EC2のみを対象としています。

RDSについてはこの後紹介するEventBridge Schedulerを使うことでECSと同様のソリューションでコンピューティングリソースの最適化を図れます。

[^3]: https://aws.amazon.com/jp/solutions/implementations/instance-scheduler-on-aws/

# RDS/ECSのコスト最適化

https://aws.amazon.com/jp/about-aws/whats-new/2022/11/amazon-eventbridge-launches-new-scheduler/

EventBridge SchedulerはスケジュールをトリガーにAWSのAPIを実行できるサーバレススケジューラです。よくあるユースケースとしてLambdaを定期実行したいときにEventBridgeでセットした時間になったら実行したいLambdaをターゲットにEventBridgeがLambdaを叩くというバッチ処理が簡単に設定できます。

EventBridge Schedulerを使うことで先のResource Schedulerと同じように深夜帯になったら停止APIを実行し、業務開始時間になったら起動APIを実行するといった定期処理を実現できます。

実際にEventBridge Schedulerで起動スケジューラーと停止スケジューラー2つを作る手順を紹介します。

## RDSスケジューラー作成手順
スケジュールの作成を選択するとスケジュールパターンについて設定します。
例として月曜から金曜の20時になったらEventBridge Schedulerが起動するようにパターンを作ります。
![](/images/aws-small-cost-optimize/image7.png)

次にAPIターゲットの選択です。すべてのAPIからAWSサービス別の大項目を選択し、そのサービス内のAPIから実行したいAPIを選択します。
例としてRDSからDBを停止したいAPIを選択します。
:::message
RDSならStopDBInstance、AuroraDBならStopDBClusterを選択します。
起動の場合`Stop`を`Start`に置き換えてください。
:::

![](/images/aws-small-cost-optimize/image8.png)

API選択後にそのAPIを渡したいDBリソースを指定します。ここはDB識別子の値しか入力できず、例えばタグ別による複数検索ができないか過去に問い合わせたことがあるのですが、残念ながらタグによる一括指定はできずDB識別子を1個ずつ指定する必要があります。

![](/images/aws-small-cost-optimize/image9.png)
*停止したいRDSインスタンスを指定*

EventBridge Schedulerで一点気を付けなければならないのはアクセス許可設定です。
![](/images/aws-small-cost-optimize/image10.png)

多くのAWSサービスでは新規リソース作成時にそのリソースが他のAWSサービスへのアクセスを許可するIAMロールの新規作成ができるのですが、EventBridge Schedulerはなぜか新しいロールの作成がグレーアウトされており事前にIAMロールを作成する必要があります。

IAMロールでEventBridge Schedulerにアクセス許可を付与するロールを作成しようとしてもコンソールからでは信頼されたエンティティにEventBridge Scheduler向けの選択肢がありません。
![](/images/aws-small-cost-optimize/image11.png)

ここはJSON記法で直接記述する必要があります。

```json:EventBridge Scheduler向け信頼されたエンティティ
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "scheduler.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

:::message alert
EventBridgeとEventBridge SchedulerはAPIが異なりますので注意してください。
```json:EventBridge向け信頼されたエンティティ
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "sts:AssumeRole"
            ],
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "apidestinations.events.amazonaws.com"
                ]
            }
        }
    ]
}
```
:::

作成したロールにDB操作できるIAMポリシーをアタッチし、先ほどのアクセス許可で作成したIAMロールを付与すればEventBridge SchedulerがRDSに対して起動停止できるようになります。

## ECSスケジューラー作成手順
ECSのスケジューラー作成もRDSとほとんど同じです。ECSのAPIからUpdateService APIを選択し、対象ECSクラスタ、サービス、実行希望数を入力します。
![](/images/aws-small-cost-optimize/image12.png)
*特定時間になったら対象サービスを0にする*

後の流れはRDSのスケジューラー設定と同じです。[^4]

[^4]: IAMロール選択時にECSへのアクセス許可が付与されているEventBridge Scheduler向けIAMロールを用意することを忘れずに

# 所感
小さなことから始められるAWSコスト最適化について紹介しました。
テスト用に動かしたリソースを消し忘れて気が付いたら課金されていたという経験をされた人は多いと思います。
AWSコストの管理はユーザー自身の責任範囲ですので小規模からでも良いので是非AWSコストを最適化できるように意識してAWSを動かしてみてください。

# 参考文献
https://dev.classmethod.jp/articles/ssm-quick-setup-resource-scheduler/
https://dev.classmethod.jp/articles/eventbridge-scheduler-and-event-bridge-rule-difference/