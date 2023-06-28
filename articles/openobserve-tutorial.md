---
title: "ログコスパ最強!?OpenObserveを触ってみた"
emoji: "🔭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["openobserve","aws","ログ","オブザーバビリティ","初心者"]
published: true
---

# 概要
昨今オブザーバビリティに関する需要は益々高まっています。Googleトレンドで`Observability`について調べても右肩上がりに増加していることがわかります。
![](/images/openobserve-tutorial/image1.png)
*過去5年間の検索推移*

その一方でオブザーバビリティツールの選定につきましては機能面、コスト面などさまざまな事情を考慮する必要がありどれを選べばいいのかわからない人も多いでしょう。
その中でもログ監視につきましてはストレージコストや運用設計など頭を悩ます項目が多いかと思われます。

会社でも私が入社してからログの保存先に関しては、`CloudWatch Logs`→`OpenSearch`→`Datadog`と転々と変更してきました。
今のところDatadogのLogs管理につきましては、使いやすく機能面では満足しています。ただ一方でストレージコストが高いこともあり、どうにかストレージコストを下げるために保存量や保持期間の調整など工夫を凝らしている最中です。

もう少しログコストが安く済むようなサービスがないかなと思いましたが、先日気になるOSSが登場しました。
それがこちらOpenObserveです。
https://openobserve.ai/

まず目につくのが10倍簡単、1/140以下の安いストレージコスト、ペタバイト相当のスケーリング、セットアップ完了が2分というアピールポイントです。

いったいどのようなサービスなのか実際に触ってみて試してみました。

# 対象読者
- OpenObserveについて知りたい人
- ログ運用でコストに悩んでいる人
- 新しいもの好きな人

# OpenObserveとは
公式ページの説明によるとログ、メトリクス、トレース監視を効率化できるRust製のオブザーバビリティプラットフォームと紹介されています。
GitHubページ[^1]によるとElasticsearch、Splunk、Datadogの代替を目指しているそうです。ElasticSearchとは[Bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)で互換性があり、FluentdやFluent Bitなどのログフォワーダーが使用できます。
注目すべき点はログのストレージコストです。冒頭で1/140以下と紹介されていますが、これは冗長化構成されたElasticSearchを自前でAWSリソースを使って構築した場合と比較した場合の結果です。
![](/images/openobserve-tutorial/image2.png)
*ESとの比較[^2]*

約180GBのログをElasticSearchとOpenObserveに取り込んだ比較表ですが、OpenObserveはデータをインデックスせずに圧縮してストレージに保存します。そのためストレージへの保存量が少なく済みストレージコストが大幅に抑えられます。OpenObserveはS3にログを取り込むことができ、HA構成で3台EBSを用意した場合と比較して1/140ほどコストが下げられるという理屈になります。
まだアルファ版で機能開発中のサービスですが、ログ検索以外にもダッシュボードの提供、アラート機能、Prometheusからのメトリクスを取り込みなど活発な開発が行なわれています。

利用する場合はホストを自前で用意してセルフ運用するか、すぐに始められるクラウド版の二通りの方法があります。
(セットアップ完了2分はクラウド版を想定しての記載です。)
まずはクラウド版を利用して使用感を試してみます。

[^1]: https://github.com/openobserve/openobserve
[^2]: https://openobserve.ai/docs/#how-does-openobserve-compare-to-elasticsearch

# セットアップ
[https://cloud.openobserve.ai/](https://cloud.openobserve.ai/)にアクセスするとサインアップを求められます。
![](/images/openobserve-tutorial/image3.png)
*Google認証可*

サインアップできましたらホーム画面が表示されます。
この手のサービスとしては珍しく多言語対応しており、日本語でも表示されました。
![](/images/openobserve-tutorial/image4.png)
当然ながらまだ何も取り込んでいませんのでログは存在してませんが、最初からデモ組織が用意されており、すぐにログ検索をしたい場合は組織を切り替えれば表示されます。

![](/images/openobserve-tutorial/image5.png)

それでは自分の組織に戻って、ログを取り込んでみます。

## ログ取り込み
ログの取り込み方法としてはFluentd[^3]などのデータコレクターを利用します。

![](/images/openobserve-tutorial/image6.png)

ただそうなるとデータコレクターをインストールできないサーバーレスサービスのログは取り込められないと思うかもしれません。
Kinesis Data Firehose[^4]やPub/Sub[^5]を転送できるサービスならOpenObserveにログを送信できるみたいです。
FargateコンテナのログをFluentBitで、CloudTrailのログをKinesis Data Firehoseで送信してみます。

### Fargateコンテナログの取り込み
ECSにはFireLensというコンテナログを他に送信できるログルーターがあります。このログルーター機能はFluent Bitで動いておりAWS公式にてイメージが配布されています。[^6]

ECSからのログをOpenObserveに送る手順もこのFireLensを使った手順が紹介されています。
以下はFluent Bitのコンテナとnginxのコンテナを記述したタスク定義ファイルになります。

:::details タスク定義ファイル
```json: nginx_firelens_zo_task_def.json
{
  "family": "nginx_firelens_zo1",
  "taskRoleArn": "arn:aws:iam::xxxxxxxxxxxx:role/ecsTaskBasicRole",
  "executionRoleArn": "arn:aws:iam::xxxxxxxxxxxx:role/ecsTaskExecutionRole",
  "cpu": "512",
  "memory": "1024",
  "requiresCompatibilities": ["FARGATE"],
  "networkMode": "awsvpc",
  "containerDefinitions": [
    {
      "name": "log_router",
      "image": "docker.io/amazon/aws-for-fluent-bit:latest",
      "essential": true,
      "firelensConfiguration": {
        "type": "fluentbit",
        "options": {
          "enable-ecs-log-metadata": "true"
        }
      },
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-create-group": "true",
          "awslogs-group": "firelens-container",
          "awslogs-region": "ap-northeast-1",
          "awslogs-stream-prefix": "firelens"
        }
      }
    },
    {
      "name": "nginx",
      "image": "nginx",
      "portMappings": [
        {
          "containerPort": 80
        }
      ],
      "essential": true,
      "environment": [],
      "logConfiguration": {
        "logDriver": "awsfirelens",
        "secretOptions": [],
        "options": {
          "Name": "http",
          "Match": "*",
          "uri": "/api/org_name/ecs_firelens1/_json",
          "host": "api.openobserve.ai",
          "Port": "443",
          "Format": "json",
          "tls": "on",
          "Json_date_key": "_timestamp",
          "Json_date_format": "iso8601",
          "http_User": "account_mail",
          "http_Passwd": "XXXXXXXXXXXXXXXXXX"
        }
      }
    }
  ]
}
```

nginxコンテナのログ設定にOpenObserveの認証情報を記載します。
>          "http_User": "account_mail",
>          "http_Passwd": "XXXXXXXXXXXXXXXXXX"
:::

Fargateへのデプロイが完了し、nginxにアクセスするとログがOpenObserveに記録されます。

![](/images/openobserve-tutorial/image7.png)
*Nginxページ*

![](/images/openobserve-tutorial/image8.png)
*Fargateのログ*

### CloudTrailの取り込み
CloudTrailのログをKinesis Data Firehoseに送るにはログをCloudWatch Logsに格納する必要があります。
![](/images/openobserve-tutorial/image9.png)

CloudTrailのログをCloudWatch Logsに送信する手順は以下のドキュメントを参照してください。
https://docs.aws.amazon.com/ja_jp/awscloudtrail/latest/userguide/send-cloudtrail-events-to-cloudwatch-logs.html

Kinesis Data Firehoseを作成する場合ソースをDirect Put、送信先をHTTPエンドポイントを指定します。
![](/images/openobserve-tutorial/image10.png)

送信先の設定にてOpenObserveのIngestionページに記載されているKinesis用のOpenObserve HTTPエンドポイントとアクセスキーを記載します。
![](/images/openobserve-tutorial/image11.png)

CloudWatch LogsからKinesis Data Firehoseへの書き込みは以下のドキュメントを参照してください。
https://docs.aws.amazon.com/ja_jp/firehose/latest/dev/writing-with-cloudwatch-logs.html

CloudWatch LogsからKinesis Data Firehoseへログが配信できましたらOpenObserveにログが届くようになります。
![](/images/openobserve-tutorial/image12.png)
*CloudTrailのログ*

[^3]: https://www.fluentd.org/
[^4]: https://aws.amazon.com/jp/kinesis/data-firehose/
[^5]: https://cloud.google.com/pubsub/docs/overview?hl=ja
[^6]: https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/userguide/firelens-using-fluentbit.html

## ダッシュボード構築
OpenObserveはダッシュボードを構築してデータを集約して把握できます。
![](/images/openobserve-tutorial/image13.png)
*CloudTrailダッシュボード*

# 所感
OpenObserveを使ってログ基盤を構築してみました。
DatadogやSumo Logic[^7]のようにログに対応したインテグレーション機能はなく、FluentdのようなOSSデータコレクターを使うか、マネージドなストリーミングETLサービスを使ってログ転送するしかなく実装は難しいと感じました。

またドキュメントにも記載されていましたが、OpenObserveはログ検索に特化したものでElasticSearchのようにアプリ検索など汎用的な検索エンジンには適していないそうです。

>Elasticsearch is a general purpose search engine which can be used app search or log search. OpenObserve is built specifically for log search. 

冒頭で述べた通りまだアルファ版ですので業務利用は難しいですが、個人開発で監視設計を意識したプロダクトを作りたい場合に使ってみてもいいかもしれません。
クラウド版なら簡単に始められるので皆さんもお試しで触ってみてください。
[^7]: https://www.sumologic.jp/

# LT資料
LTのスライド資料です。
@[speakerdeck](f0ce80504f7248b49aa31056240dd45d)

# 参考文献
https://zenn.dev/y_k/articles/0528b27b0d0512
https://github.com/openobserve/openobserve
https://www.youtube.com/watch?v=fZ-ErfMdF-o
https://docs.fluentbit.io/manual/pipeline/outputs/http