---
title: "監視からオブザーバビリティへ"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["監視","オブザーバビリティ","AWS"]
published: false
---

# 概要
先日弊社の監視運用をMackerelからDatadogへ切り替えました。その時監視だけでなく、OpenSearchによるログ集約運用も見直しそちらもDatadogにまとめました。
![](/images/cloud-observability/image1.png)

Datadogは前の会社でも使っていましたが、すでに導入済みで特に私自身使う機会がなかったので今回の導入に向けて色々とDatadogの導入方法や監視そのものの運用について色々と調べてみました。
そのなかでオブザーバビリティという用語が気になりましたので、私が調べて感じた事、AWSで実践できるワークショップの紹介も兼ねて紹介いたします。

# オブザーバビリティとは
日本語訳では可観測性とも紹介され観察する能力という意味合いになります。何を観察するか。エンジニアがシステムやプロダクトを
リリースした後の運用でログの取得やCPU使用率、パフォーマンスデータの確認など多くの指標を取得すると思います。
これらを取得する目的はなんでしょうか。おそらく企業ごとに取得する目的は異なると思います。
https://www.datadoghq.com/ja/resources/business-value-of-observability-ebook/
Datadog社がクライアント企業の導入事例をベースに以下のように説明しています。
- ITコストの削減
- ITのアップタイムの向上
- 市場投入までの時間短縮
- 顧客転換率の向上
- セキュリティリスクの低減

AWSブログでも観測する目的としてシステムのどこで何が起こっているかを知ることが大事だと述べています。
https://aws.amazon.com/jp/products/management-and-governance/use-cases/monitoring-and-observability/
> オブザーバビリティの主な目的は、システムのどこで何が起こっているか知ることです。

## オブザーバビリティと監視の違い
オブザーバビリティという新しい言葉を使わなくてもログ取得やメトリクス取得は監視という運用で昔からサービス運用で使われてきたと思います。
この2つの違いはなんでしょうか。
いくつかの記事を参照するとこの2つは比較するものではなく、オブザーバビリティは考え方であり監視は手段であると述べています。オブザーバビリティは複雑なシステムを適切に管理するためにも重要です。特にクラウド上で稼働するシステムはマイクロサービス化が進み、障害やパフォーマンス低下時にどこから調査すればいいかわからなくなります。

**監視はシステムの状態を見続けることに対し、オブザーバビリティはシステムで何が起きているのかを把握するためのものと言えます。**

## オブザーバビリティの三本柱
オブザーバビリティには何が、どこで、なぜ発生したのかを把握するために3つの要素があります。

- メトリクス
    - 特定期間のデータの集計値(何が起きたか)
- トレース
    - リクエスト、トランザクションの経過を観察(どこで起きたか)
- ログ
    - 特定地点の記録を示す(なぜ起きたか)

# AWSでのオブザーバビリティ
AWSでオブザーバビリティを実現するためにはさまざまなマネージドサービスやオープンソースが提供されています。
![](/images/cloud-observability/image2.png)
さらにAWSでオブザーバビリティを実現するためのワークショップもあります。
https://catalog.workshops.aws/observability/ja-JP
こちらのワークショップではX-Ray、CloudWatch、Prometheusなどボリュームたっぷりな内容になっています。

ワークショップではデモ用のペット保護のボランティアサイトを構築して、サイトのオブザーバビリティを実装します。
![](/images/cloud-observability/image3.png)
*デモサイト*
![](/images/cloud-observability/image14.png)
*アーキテクチャ図*
EKS上で構築されたペットサイトに対して、GrafanaやPrometheusなどの監視サービスを利用し、モニタリングしていきます。

:::message alert
説明では3~4時間と記載されていましたが、全部を一通りやろうとすると1日超えました。扱うリソースが多く、何日も放置したままにするとAWSコストが結構かかりましたのでハンズオンは余裕をもって始めたほうがよろしいです。
:::

# オブザーバビリティサービス紹介
ワークショップにそっていくつかのサービスを紹介します。
全部を紹介するとなると膨大な量になりますので全部気になる人はぜひワークショップに挑戦してみてください。

## CloudWatch ServiceLens
https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/ServiceLens.html
CloudWatch ServiceLensはトレース、メトリクス、ログなどのリソースヘルス情報を1か所に統合しサービスアプリケーションの監視強化を行ないます。サービスマップが各リソースとエンドポイント間のトラフィックやレイテンシー、エラー状態を表示しアプリケーションの影響を迅速に調査できます。
![](/images/cloud-observability/image4.png)
*デモサイトのサービスマップ*

## CloudWatch Synthetics
https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html
CloudWatch SyntheticsはCanaryとよばれるスケジュールに沿って実行されるNode.jsまたはPythonのスクリプトを作成し、エンドポイントとAPIをモニタリングできます。Canaryで顧客が操作するアクションを記述し、顧客が遭遇する問題にいちはやく発見できます。
![](/images/cloud-observability/image5.png)
*Canary成否結果*
![](/images/cloud-observability/image15.png)
*自分で書かなくても事前に提供されているブループリントから簡易的なテストを実装できる*

## CloudWatch RUM
https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/CloudWatch-RUM.html
RUMとはリアルユーザーモニタリング(Real User Monitoring)を意味し、ユーザーが体験するアプリケーションパフォーマンスと信頼性をモニタリングします。ユーザーセッション情報を収集し、ページ読み込み時間、クライアントエラー、ユーザー活動などを可視化、分析できます。Datadogにも同様の機能[^1]がマネージドサービスとして提供されており、UX最適化のために大事なモニタリングサービスになります。
弊社でもRUMを実装して、WPコンテンツのレスポンス速度やユーザージャーニーでユーザーのページ遷移をビジネス分析できないか検討している最中になります。

![](/images/cloud-observability/image7.png)
*ウェブバイタル情報 入力レイテンシーがどれくらいかわかる*

## Container Insights
https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/ContainerInsights.html
CloudWatch Container InsightsはECSやEKS、Fargateなどコンテナ上で生成されるメトリクスやログを収集、集計できます。収集したメトリクスはCloudWatchダッシュボード上でも表示できますので、アラート監視を設定しメトリクス異常が見つかったらSlackに通知するようにしてモニタリングできます。
![](/images/cloud-observability/image8.png)
*コンテナマップ ノードをクリックするとメトリクスが表示される*

![](/images/cloud-observability/image10.png)
*EKSポッドのCPU使用率情報*



[^1]: https://docs.datadoghq.com/ja/real_user_monitoring/browser/
# 参考資料
https://licensecounter.jp/devops-hub/blog/splunk1/
https://www.oreilly.co.jp/books/9784873118642/
https://newrelic.com/jp/topics/what-is-observability
https://www.splunk.com/ja_jp/data-insider/what-is-observability.html
https://d1.awsstatic.com/events/jp/2020/innovate/pdf/S-8_AWSInnovate_Online_Conference_2020_Spring_management_monitoring.pdf