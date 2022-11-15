---
title: "SSL脆弱性診断テストでA評価をとってみよう"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ssl","セキュリティ"]
published: false
---

# 概要
Googleが常時SSL化しているサイトへのSEO評価を高めると発表してからずいぶん時間が経ちました。[^1]
日本国内の上場企業およそ9割がサイトの常時SSL化を対応しています。[^2]
個人開発でも自己ドメインを取得し、SSL証明書を発行される人は多いと思います。そのSSLの設定が正しくできているか無料で利用できるサイトがQualys SSL Labs社が提供するSSL Server Testです。
https://www.ssllabs.com/ssltest/index.html


[^1]: https://developers.google.com/search/blog/2014/08/https-as-ranking-signal?hl=ja

[^2]: https://www.feedtailor.jp/report_aossl/