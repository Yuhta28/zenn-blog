---
title: "Instance Schedulerを使ってEC2の稼働時間を管理してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","ec2"]
published: true
---

# 概要
会社で使っているEC2はSavings Plansを採用していてオンデマンドで使うよりも安い料金でEC2を利用しています。
ただそれでもEC2の数が段々と増えていて、当初の予想よりもコンピューティングリソースの消費が大きくなりEC2の利用料が大きくなってきました。
そこで検証環境のEC2の稼働時間を減らし、コンピューティングリソースの消費を抑えることでEC2の利用料を節約しようと思いました。
前職では先輩が作成したLambdaで21:00~翌9:00の時間帯は自動でシャットダウンする仕組みができており、当初はそれを真似しようと思いましたがInstance Schedulerなるものを今の会社のリーダーから教わりました。
今回はInstance Schedulerを実際に使ってみてどういったものなのか試してみました。

# Instance Schedulerについて
先にお話しするとInstance SchedulerというAWSリソースはありません。
複数のサービスを組み合わせて実装したAWSソリューションです。
https://aws.amazon.com/jp/solutions/implementations/instance-scheduler/

イメージとしてはEC2に専用のタグを付与して、LambdaがDynamoDBに保存されているタグごとに定義されたスケジュール情報を読み取り、DynamoDBのタグ情報と一致したタグを持つEC2に対してLambdaが停止、起動するという仕組みになっています。
![](/images/ec2-schedule/image1.png)
*アーキテクチャ*

[上記ページから引用](https://aws.amazon.com/jp/solutions/implementations/instance-scheduler/)
CloudFormationテンプレートファイルが用意されていますので、自前で上記アーキテクチャを構築する必要はなく、簡単に作成できますのでまずはテンプレートファイルをダウンロードし、CloudFormationを実行します。
###### テンプレートファイル
https://s3.amazonaws.com/solutions-reference/aws-instance-scheduler/latest/aws-instance-scheduler.template

# CloudFormation実行
テンプレートファイルをアップロードし、スタック作成を進めていくと以下のパラメータ入力が求められます。

#### Scheduler
- TagName
  - インスタンススケジューラー対象となるタグの定義付け(デフォルトでScheduleが付与)
- ScheduledServices
  - 対象のAWSサービス(EC2、RDS、その両方の中から選択)
- ScheduleRdsClusters
  - Auroraクラスターのスケジューラーを有効可否
- CreateRdsSnapshot
  - 停止前にRDSのスナップショット取得可否
- SchedulingActive
  - スケジューラーの即時有効可否
- Regions
  - 対象リージョン選択
- DefaultTimezone
  - デフォルトのタイムゾーン設定
- CrossAccountRoles
  - クロスアカウントロール
- This account
  - 現在のアカウントのリソースに対してスケジューラーの有効可否
- SchedulerFrequency
  - Lambdaの実行頻度(分単位)
- MemorySize
  - Lambdaのメモリサイズ指定

#### Option
- UseCloudWatchMetrics
  - CloudWatchメトリクスによるインスタンスデータ収集の有効可否
- UseCloudWatchLogs
  - CloudWatchによるロギング設定の有効可否
- EnableSSMMaintenanceWindows
  - SSMメンテナンスウインドウをロードしてEC2のスケジューラーの設定有効可否

#### Other parameters
- LogRetentionDays
  - ログ保持期間
- StartedTags
  - 起動したインスタンスに追加するタグ
- StoppedTags
  - 停止したインスタンスに追加するタグ

基本的にデフォルト設定のままで問題ありません。
EC2インスタンスとRDSインスタンスの両方をインスタンススケジューラー対象にしたい場合、`Both`を選択します。
そのほかですと、Regionで東京を希望する場合は`ap-northeast-1`、日本時間でスケジューリングしたい場合はTimeZoneを`Asia/Tokyo`にします。
パラメータ入力を完了し、スタック作成の確認画面まで進むとIAMの新規作成のチェック確認がありますので忘れずにチェックします。
![](/images/ec2-schedule/image2.png)

これを起動することで上記図のアーキテクチャが構築されEC2(またはRDS)にTagに`Schedule`をつけたものが対応したスケジュールアクションを設定することで自動で停止起動が実現できます。

# サンプルハンズオン
起動できたらまずはDynamoDBを見てみます。
いくつかテーブルが作られていると思いますが、`ConfigTable`を見てます。
![](/images/ec2-schedule/image3.png)
注目するポイントはTypeの`period`と`schedule`です。
##### period
インスタンスの稼働時間を指定できます。
サンプルで作成された`office-hours`は平日月曜日~金曜日の10:00~19:00の間に稼働することを意味しています。

| フィールド| 値 |
| ---- | ---- |
| begintime | 10:00 |
| endtime | 19:00 |
| name | office-hours|
| weekdays | mon-fri |

![](/images/ec2-schedule/image4.png)

##### schedule
scheduleに複数のperiodをまとめることでインスタンスに複数のperiodのスケジューリング設定を反映させることができます。
サンプルである`jp-office-hours`はJST基準で`office-hours`のperiod設定を反映させる設定になります。

| フィールド| 値 |
| ---- | ---- |
| period | office-hours |
| timezone | Asia/Tokyo |
| name | jp-office-hours |

![](/images/ec2-schedule/image5.png)

このタグを稼働しているインスタンスに付与します。
このハンズオンを実施している時間帯は日曜日ですので、スケジュールタグ`Schedule`を付与したらEC2は自動で停止するようになります。
![](/images/ec2-schedule/image6.png)
*タグ付与直後*

![](/images/ec2-schedule/image7.png)
*数分後*

タグを付与してから数分後にEC2のステータスが変更し、停止されました。
ここで土曜日、日曜日は終日起動するというperiod`weekends`を`jp-office-hours`のscheduleに追加します。
![](/images/ec2-schedule/image8.png)
![](/images/ec2-schedule/image9.png)

テーブルの編集が完了したらあっという間にEC2が起動されました。
![](/images/ec2-schedule/image10.png)

このようにsheduleに複数のperiod設定をつけた場合どれか一つの期間を満たせばEC2が起動するようになっています。
これを活用することで例えば9:00-12:00の間起動し、12:00-13:00の間は停止、また13:00から起動するといった細やかな設定も可能になります。

# Scheduler CLIによるスケジューリング設定
スケジュールの設定は前述のとおりDyanamoDBのテーブルを編集することで新規にスケジュールを作成できますが、Instance Scheduerには専用のCLIコマンドも用意されています。
次にCLIコマンドを活用したスケジューリング設定について説明します。

## インストール方法
下記リンクからScheduler CLIパッケージをダウンロードしてローカルに展開します。
https://s3.amazonaws.com/solutions-reference/aws-instance-scheduler/latest/scheduler-cli.zip

展開したらディレクトリ`scheduler-cli`へ移動して`setup.py`を実行します。

```bash
Python setup.py install

# インストール完了後確認
scheduler-cli --version
scheduler-cli v1.2.0
```

## 使い方
コマンドの構造としてサブコマンドと変数指定があります。

```bash
scheduler-cli <sub-command> <arguments>
```

サブコマンドには以下のものがあります。

- create-period       
- create-schedule     
- delete-period       
- delete-schedule     
- describe-periods    
- describe-schedule-usage
- describe-schedules  
- update-period       
- update-schedule     

CLIコマンドを使ってperiodとscheduleを新規に作成していきます。

#### period作成

```bash
scheduler-cli create-period --name "sample-period" --begintime 10:00 --endtime 15:00 --weekdays mon-fri --stack test-schedule
{
   "Period": {
      "Begintime": "10:00",
      "Endtime": "15:00",
      "Name": "sample-period",
      "Weekdays": [
         "mon-fri"
      ],
      "Type": "period"
   }
}
```

![](/images/ec2-schedule/image11.png)

#### schedule作成

```bash
 scheduler-cli create-schedule --name sample-schedule --periods sample-period --timezone Asia/Tokyo --stack test-schedule
{
   "Schedule": {
      "Timezone": "Asia/Tokyo",
      "Name": "sample-schedule",
      "Periods": [
         "sample-period"
      ],
      "StopNewInstances": true,
      "UseMaintenanceWindow": false,
      "RetainRunning": false,
      "Enforced": false,
      "Hibernate": false,
      "UseMetrics": false,
      "Type": "schedule"
   }
}
```

![](/images/ec2-schedule/image12.png)

DynamoDBのテーブルにもCLIで作成したperiodとscheduleが表示されていることを確認できました。
これでCLIによるインスタンスのスケジューリング設定も行えることができます。

# 所感
Instance Schedulerを使ってEC2の稼働時間の管理ができました。
検証環境は基本的に深夜早朝、土日は動かす必要はないので停止したほうがいいですが、毎回仕事終わりにEC2を停止する運用にしても忘れることもありますし、検証環境の数が多いと一々手で止めるなんてことを行なうとオペレーションコストが高まってしまいます。
Instance SchedulerでEC2のコスト最適化を推進して無駄を減らしていこうと思います。

# 参考文献
https://d1.awsstatic.com/Solutions/ja_JP/instance-scheduler.pdf

https://www.youtube.com/watch?v=HqXMxdjv638

https://docs.aws.amazon.com/solutions/latest/instance-scheduler/scheduler-cli.html