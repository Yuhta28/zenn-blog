---
title: "CloudTrailのログを直接クエリ検索できる新サービスCloudTrail Lakeを触ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws"]
published: false
---

# 概要
先日のアップデートで新しくCloudTrail Lakeという機能がリリースされました。
https://aws.amazon.com/jp/about-aws/whats-new/2022/01/aws-cloudtrail-lake-audit-security/
CloudTrail Lakeでどんなことができるのか、従来のAthenaでのクエリ検索と何が違うのか検証してみました。

# CloudTrailとAthenaの連携
CloudTrailはAWSのオペレーションログを記録してくれるサービスです。
私はよく使っていまして、例えば誰が立ち上げたのかわからないEC2インスタンスの起動ユーザーを確認して削除してよいかどうか調査したり、夜間に検証環境のEC2インスタンスが自動で停止されているのか確認したいときなどに使います。

ただCloudTrailで検索できるログ内容は過去90日以内のものしかなく、クエリ検索もできないので複雑な条件でログ検索をしたい場合はS3にログを転送して、Athenaでログを分析する必要がありました。
![](/images/cloudtraillake-athena/image2.png)
*CloudTrailログをS3へ転送する*
![](/images/cloudtraillake-athena/image1.png)
*CloudTrailログからAthenaテーブルを作成*

AthenaにCloudTrailログのテーブルが作成されるとクエリ検索で過去のAWS操作記録を調べることができます。

### サンプルクエリ

```sql
SELECT
 eventtime,
 useridentity.userName,
 eventname
FROM cloudtrail_logs_aws_cloudtrail_logs_152231080651_2bc74aa9 #テーブル名は一意
order by eventtime desc
LIMIT 100
```
![](/images/cloudtraillake-athena/image4.png)

### EC2インスタンスを作成したユーザーを検索
```sql
SELECT
 eventtime,
 useridentity.userName,
 eventname
FROM cloudtrail_logs_aws_cloudtrail_logs_152231080651_2bc74aa9
where eventname like 'RunInstances'
order by eventtime desc
LIMIT 100
```

### AWSにログインしたユーザーを検索

```sql
SELECT
 eventtime,
 useridentity.userName,
 eventname
FROM cloudtrail_logs_aws_cloudtrail_logs_152231080651_2bc74aa9
where eventname like 'ConsoleLogin'
order by eventtime desc
LIMIT 100
```

こんな感じでSQLクエリを使ってAWSを操作したユーザー履歴を追うことができます。

# CloudTrail Lakeについて
新しくリリースされたCloudTrail LakeはAthenaテーブルを作成することなくSQLクエリ検索が可能になります。
CloudTrailのコンソール画面に`Lake`項目があり、クリックすると初期設定では`Event data stores`タブに飛ぶのでイベントデータストアを作成します。

![](/images/cloudtraillake-athena/image6.png)
今回はデータストア名と`イベントデータストアに現在のリージョン (ap-northeast-1) のみを含める`の欄にチェックを入れて、残りはデフォルトの設定のまま作成しました。
各項目の詳しい説明内容は[AWS Blog](https://aws.amazon.com/jp/blogs/mt/announcing-aws-cloudtrail-lake-a-managed-audit-and-security-lake/?s=09)やクラスメソッドさんの[記事](https://dev.classmethod.jp/articles/aws-cloudtrail-lake-audit-security/)が参考になると思います。
イベントデータストアを作成後、エディタータブをクリックするとAthenaライクなエディター画面が見えるようになります。
![](/images/cloudtraillake-athena/image7.png)

:::message
作成されるイベントデータストアのカラム内容はこちらのドキュメントを参照してください。
https://docs.aws.amazon.com/awscloudtrail/latest/userguide/query-limitations.html
:::

### サンプルクエリ
CloudTrail Lakeでクエリ検索を実施する場合は、イベントデータストア名の下にあるストアIDを指定して検索します。

```sql
SELECT eventTime, userIdentity.username, eventSource, eventName, awsRegion FROM 880143b0-368b-4ef8-a6e1-887a48146de7 
WHERE eventTime >= '2022-01-22 00:00:30' AND eventTime <= '2022-01-22 05:00:00' 
order by eventtime desc 
LIMIT 1000;
```

![](/images/cloudtraillake-athena/image8.png)

#### 特定ユーザーの全操作記録を検索

```sql
SELECT eventTime, eventSource, eventName, awsRegion FROM 880143b0-368b-4ef8-a6e1-887a48146de7 
WHERE eventTime >= '2021-01-22 00:00:30' AND 
eventTime <= '2022-01-22 06:40:00' AND 
userIdentity.username like 'Yuta0128' AND 
awsRegion like 'ap-northeast-1' 
order by eventtime desc 
limit 1000;
```

もし他のリージョンのログも出力する設定にしていますとログの件数が増えると思いますので、`awsRegion`カラムで自身のリージョンを指定して検索することを推奨します。

# 気になったこと
CloudTrail Lakeの機能はAthenaでできることを一通り備えています。
クエリ履歴を使えば直近使ったクエリ内容を確認できますし、よく使うクエリを保存する機能もあります。

気になったのが料金体系です。
CloudTrail Lakeもスキャン数に応じて課金され、1GBあたり0.005 USDと1TBで5USDのAthenaと実質同じ料金です。
ただしCloudTrail Lakeはイベントデータストアを作成し、そこにログを取り込むため保存料が別途かかります。
![](/images/cloudtraillake-athena/image9.png)
# 参考文献
https://dev.classmethod.jp/articles/aws-cloudtrail-lake-audit-security/
https://docs.aws.amazon.com/ja_jp/athena/latest/ug/cloudtrail-logs.html
https://docs.aws.amazon.com/awscloudtrail/latest/userguide/query-limitations.html
https://aws.amazon.com/jp/cloudtrail/pricing/