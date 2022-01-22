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

# 良かった点気になった点
CloudTrail Lakeの機能はAthenaでできることを一通り備えています。
クエリ履歴を使えば直近使ったクエリ内容を確認できますし、よく使うクエリを保存する機能もあります。

CloudTrail Lakeの優位性としてマルチアカウントに対応している点です。
Athenaを使ったクエリ分析ではマルチリージョンまではCloudTrailのコンソール画面からAthenaテーブル作成時に対応できますが、別のAWSアカウントのCloudTrailのログまでは対応しきれていません。
マルチアカウントのログをクエリ分析する場合、AWS Organizationでログアーカイブ用のアカウントを用意し、組織内のログをそのアカウントに一元化させる必要があります。
CloudTrail Lakeでしたらイベントデータストア作成時にチェックボックスで組織内のすべてのアカウントを含めさせるだけでよいので手間が減ります。

気になったのが料金体系です。
CloudTrail Lakeもスキャンサイズに応じて課金され、1GBあたり0.005 USDと1TBで5USDのAthenaと実質同じ料金です。
ただしCloudTrail Lakeはイベントデータストアを作成し、そこにログを取り込むときに別途料金がかかります。
![](/images/cloudtraillake-athena/image9.png)
若干わかりにくいですが、取り込むときの１回の操作に料金がかかりますので毎月2.5USDコストが請求されるわけではありません。ただし7年以上の長期保存されているログに対しては保存量に応じて課金されるという仕組みです。
言い換えると最初の7年間は取り込んだ料金のみが請求されます。
S3に１か月、1GB保存した場合の料金は0.025USD(東京リージョン)ですので仮に7年間1GBのログを保存しますと、 `0.025USD * 12 * 7 = 2.1USD` となりますのでCloudTrail Lakeに保存したほうが若干割高になります。
(そもそも7年もStandard S3に同一のログを格納するというケースもそんなにないと思います)
完全にAthenaからのクエリ分析を移行するというよりかはマルチアカウント運用時のログ収集を楽にするといった多少限定的なユースケースになるのかなと個人的には考えています。

ほかに分からなかったことですが、スキャンサイズの基準です。
Athenaはパーティションを定義することで、スキャン範囲を絞ることができスキャン速度改善、スキャンコスト最適化ができるのですが、CloudTrail Lakeにはパーティションを定義する設定がありません。
ただクエリ履歴のクエリサイズ部分を確認するとスキャン範囲が検索ごとに変わっていました。
![](/images/cloudtraillake-athena/image10.png)

これはAWS側でスキャンサイズを自動的に最適化してくれるのでしょうか🤔
ドキュメントや他の人のブログなど確認しましたが、クエリサイズについて触れているドキュメントは見つかりませんでした。
もしスキャンサイズの最適化をAWS側で最適化してくれるのならAthenaテーブル作成時にパーティションを定義をする必要がなくなり、パーティションを追加するという手間がなくなります。
以前S3に保存されているログをAWS Glueを使ってAthenaテーブルを作成し、パーティションを定義したことがあるのですが結構大変で理解が追いつかない部分があったのでここをAWSに任せられるなら便利かもしれません。
とは言えドキュメントにはっきりと記載もないので確かなことが言えないのでこの辺りわかる人がいましたらアドバイス頂けますと幸いです。

# 所感
2022年１月にリリースされたばかりのCloudTrail Lakeについて調べてみました。
リリースされたときはログ保存用のS3を用意してAthenaテーブルを作成する必要がなくなり、便利かなと思いましたがコスト的な面でいうと代替になるサービスとは言い難いです。
イベントデータストアを作成する必要がありますので作成以前のログはCloudTrail Lakeで検索することはできません。
弊社もCloudTrailのログはS3に保存して90日以前のものを検索したい場合はAthenaで検索していますので今のところこの機能を会社で使うことはないかなという所感です。

まだリリースされて新しいので機能のアップデートなどがあったらまた触ってみようかなと思います。

# 参考文献
https://dev.classmethod.jp/articles/aws-cloudtrail-lake-audit-security/
https://docs.aws.amazon.com/ja_jp/athena/latest/ug/cloudtrail-logs.html
https://docs.aws.amazon.com/awscloudtrail/latest/userguide/query-limitations.html
https://aws.amazon.com/jp/cloudtrail/pricing/
https://dev.classmethod.jp/articles/cloudtrail-lake-pricing/
https://ermetic.com/blog/aws/testing-the-waters-first-impressions-of-cloudtrail-lake/