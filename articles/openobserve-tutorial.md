---
title: "ログコスパ最強!?OpenObserveを触ってみた"
emoji: "🔭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["openobserve","aws","ログ","オブザーバビリティ"]
published: false
---

# 概要
昨今オブザーバビリティに関する需要は益々高まっています。Googleトレンドで`Observability`について調べても右肩上がりに増加していることがわかります。
![](/images/openobserve-tutorial/image1.png)
*過去5年間の検索推移*

その一方でオブザーバビリティツールの選定につきましては機能面、コスト面など様々な事情を考慮する必要がありどれを選べばいいのかわからない人も多いでしょう。
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

# 目次
- [OpenObserveとは](#openobserveとは)
- セットアップ
- ログ取り込み
- ダッシュボード構築
- [所感](#所感)
- [参考文献](#参考文献)

# OpenObserveとは
公式ページの説明によるとログ、メトリクス、トレース監視を効率化できるRust製のオブザーバビリティプラットフォームと紹介されています。
GitHubページ[^1]によるとElasticsearch、Splunk、Datadogの代替を目指しているそうです。注目すべき点はログのストレージコストです。冒頭で1/140以下と紹介されていますが、これは冗長化構成されたElasticSearchを自前でAWSリソースを使って構築した場合と比較した場合の結果です。
![](/images/openobserve-tutorial/image2.png)
*ESとの比較[^2]*

約180GBのログをElasticSearchとOpenObserveに取り込んだ比較表ですが、OpenObserveはデータをインデックスせずに圧縮してストレージに保存します。そのためストレージへの保存量が少なく済みストレージコストが大幅に抑えられます。OpenObserveはS3にログを取り込むことができ、HA構成で3台EBSを用意した場合と比較して1/140ほどコストが下げられるという理屈になります。

利用する場合はホストを自前で用意してセルフ運用するか、すぐに始められるクラウド版の二通りの方法があります。
(セットアップ完了2分はクラウド版を想定しての記載です。)
まずはクラウド版を利用して使用感を試してみます。

[^1]: https://github.com/openobserve/openobserve
[^2]: https://openobserve.ai/docs/#how-does-openobserve-compare-to-elasticsearch
[^3]: https://aws.amazon.com/jp/opensearch-service/
# 所感

# 参考文献
https://zenn.dev/y_k/articles/0528b27b0d0512
https://github.com/openobserve/openobserve
https://www.youtube.com/watch?v=fZ-ErfMdF-o