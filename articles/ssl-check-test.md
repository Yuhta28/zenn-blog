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
もちろん諸外国でも同様の動きがみられます。
![](/images/ssl-check-test/image3.png)
*Google Transparency Reportより[^3]*
個人開発でも自己ドメインを取得し、SSL証明書を発行される人は多いと思います。そのSSLの設定が正しくできているか無料で利用できるサイトがQualys SSL Labs社が提供するSSL Server Testです。
https://www.ssllabs.com/ssltest/index.html
このサイトを使って自分のWebサイトが正しくSSL化できているか試してみましょう。

[^1]: https://developers.google.com/search/blog/2014/08/https-as-ranking-signal?hl=ja

[^2]: https://www.feedtailor.jp/report_aossl/

[^3]: https://transparencyreport.google.com/https/overview
# SSL Server Testについて
前述したとおりSSL Server Testはドメインを入力することでそのサイトのSSL設定が正しく設定されているか診断してくれるサイトになります。
![](/images/ssl-check-test/image1.png)
適当にEC2+RDSでWordPressを立ててAWS ACMの証明書でSSL化した私のサイトはB評価と出ています。最高評価がA+になり、B以下には脆弱性が含まれる結果となりました。このサイトをA+に持っていけるようにSSL設定の見直しを行なっていきます。

# 問題個所
各項目部分を確認するとProtocol Supportの評価が低くなっていることがわかります。
>This server supports TLS 1.0 and TLS 1.1. Grade capped to B. 

TLS 1.0とTLS 1.1をサポートしていると最大でもBまでの評価しか得られないと記載されています。
![](/images/ssl-check-test/image2.png)

## TLSとSSL
あらためてSSLについて説明しますと、SSL(Secure Socket Layer)とはデータを暗号化して送受信できるプロトコルです。1990年代にリリースされましたが深刻な脆弱性があるため今はほとんど使われていません。現在はSSLの後継のTLS(Transport Layer Security)をつかってサイトSSL化を実現しています。

::: message
厳密性を求めるとTLS通信と表現するのが正しいのかもしれませんが、SSLの名称が広く使用されてきた経緯がありますので、今でもTLSのことをSSLもしくはTLS/SSLと表記することが多いです。
:::

TLSが標準となった後も脆弱性対応で定期的な最新バージョンのリリースが行なわれ2022年現在の最新バージョンはTLS 1.3になります。

| SSL/TLSバージョン | リリース年 | 推奨非推奨 |
| ---- | ---- | ---- |
| SSL 1.0 | - | 非推奨(未リリース) |
| SSL 2.0 | 1994年 | 非推奨 |


# 参考文献
https://tech.synapse.jp/entry/2020/11/03/120000
https://seven.ne.jp/qualys-ssl-labs-%E3%81%AE-ssl%E8%84%86%E5%BC%B1%E6%80%A7%E8%A8%BA%E6%96%ADssl-server-test%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6/
https://www.infraexpert.com/study/security7.html