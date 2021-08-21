---
title: "Instance Schedulerを使って検証環境のEC2の稼働時間を管理してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","ec2"]
published: false
---

# 概要
会社で使っているEC2はSavings Plansを採用していてオンデマンドで使うよりも安い料金でEC2を利用しています。
ただそれでもEC2の数が段々と増えていて、当初の予想よりもコンピューティングリソースの消費が大きくなりEC2の利用料が大きくなってきました。
そこで検証環境のEC2インスタンスの稼働時間を減らし、コンピューティングリソースの消費を抑えることでEC2の利用料を節約しようと思いました。
前職では先輩が作成したLambdaで21:00~翌9:00の時間帯は自動でシャットダウンする仕組みができており、当初はそれを真似しようと思いましたがInstance Schedulerなるものを今の会社のリーダーから教わりましたので、それを使って検証環境のEC2の稼働時間を減らしてみたいと思います。

# Instance Schedulerについて
先にお話しするとInstance SchedulerというAWSリソースはありません。
複数のサービスを組み合わせて実装したAWSソリューションです。
https://aws.amazon.com/jp/solutions/implementations/instance-scheduler/

イメージとしてはEC2に専用のタグを付与して、LambdaがDynamoDBに保存されているタグごとに定義されたスケジュール情報を読み取り、DynamoDBのタグ情報と一致したタグを持つEC2に対してLambdaが停止、起動するという仕組みになっています。
![](/images/ec2-schedule/image1.png)
*アーキテクチャ*

[上記ページから引用](https://aws.amazon.com/jp/solutions/implementations/instance-scheduler/)
CloudFormationテンプレートファイルが用意されていますので、自前で上記アーキテクチャを構築する必要はなく、簡単に用意できますのでまずはテンプレートファイルをインストールし、CloudFormationを実行します。
###### テンプレートファイル
https://s3.amazonaws.com/solutions-reference/aws-instance-scheduler/latest/aws-instance-scheduler.template

# CloudFormation実行
テンプレートファイルをアップロードし、スタック作成を進めていくと以下のパラメータ入力が求められます。

#### Shceduler
- TagName
  - インスタンススケジューラー対象となるタグの定義付け
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


# 参考文献
https://d1.awsstatic.com/Solutions/ja_JP/instance-scheduler.pdf