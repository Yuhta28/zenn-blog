---
title: "監視から可観測性へ"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["監視","可観測性","AWS"]
published: false
---

# 概要
先日弊社の監視運用をMackerelからDatadogへ切り替えました。その時監視だけでなく、OpenSearchによるログ集約運用も見直しそちらもDatadogの中にまとめました。
![](/images/cloud-observability/image1.png)

Datadogは前の会社でも使っていましたが、すでに導入済みで特に私自身使う機会がなかったので今回の導入に向けて色々とDatadogの導入方法や監視そのものの運用について色々と調べてみました。
そのなかで可観測性という用語が気になりましたので、私が調べて感じた事、AWSで実践できるワークショップの紹介も兼ねて紹介いたします。

# 可観測性とは
英語ではObservabilityと紹介され、直訳すると観察する能力という意味合いになります。何を観察するか。エンジニアがシステムやプロダクトを
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

# 可観測性と監視の違い
可観測性という新しい言葉を使わなくてもログ取得やメトリクス取得は監視という運用で昔からサービス運用で使われてきたと思います。
この2つの違いはなんでしょうか。
## 可観測性の三本柱

# 参考資料
https://licensecounter.jp/devops-hub/blog/splunk1/
https://www.oreilly.co.jp/books/9784873118642/