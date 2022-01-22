---
title: "CloudTrail LakeとAthenaでのログ分析を比べてみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws"]
published: false
---

# 概要
先日のアップデートで新しくCloudTrail Lakeという機能がリリースされました。
https://aws.amazon.com/jp/about-aws/whats-new/2022/01/aws-cloudtrail-lake-audit-security/
CloudTrail Lakeと従来CloudTrailのログを分析する際に使用されたAthenaとの比較をしてみようと思います。

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
# 参考文献
https://dev.classmethod.jp/articles/aws-cloudtrail-lake-audit-security/
https://docs.aws.amazon.com/ja_jp/athena/latest/ug/cloudtrail-logs.html