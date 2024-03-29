---
title: "SSL脆弱性診断テストでA評価をとってみよう"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ssl","セキュリティ"]
published: true
published_at: 2022-11-16 08:00
---

# 概要
Googleが常時SSL化しているサイトへのSEO評価を高めると発表してからずいぶん時間が経ちました。[^1]
日本国内のおよそ9割の上場企業がサイトの常時SSL化を対応しています。[^2]
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
適当にEC2+RDSでWordPressを立ててACM[^4]の証明書でSSL化した私のサイトはB評価と出ています。最高評価がA+で、B以下には脆弱性が含まれます。このサイトをA+に持っていけるようにSSL設定の見直しを行なっていきます。

[^4]: https://aws.amazon.com/jp/certificate-manager/

# 問題個所
各項目部分を確認するとProtocol Supportの評価が低くなっていることがわかります。
>This server supports TLS 1.0 and TLS 1.1. Grade capped to B. 

TLS 1.0とTLS 1.1をサポートしていると最大でもBまでの評価しか得られないと記載されています。
![](/images/ssl-check-test/image2.png)

## TLSとSSL
今一度SSLについて説明しますと、SSL(Secure Socket Layer)とはデータを暗号化して送受信できるプロトコルです。1990年代にリリースされましたが深刻な脆弱性があるため今はほとんど使われていません。現在はSSLの後継のTLS(Transport Layer Security)をつかってサイトSSL化を実現しています。

::: message
厳密性を求めるとTLS通信と表現するのが正しいのかもしれませんが、SSLの名称が広く使用されてきた経緯がありますので、今でもTLSのことをSSLもしくはTLS/SSLと表記することが多いです。
:::

TLSが標準となった後も脆弱性対応で定期的な最新バージョンのリリースが行なわれ2022年現在の最新バージョンはTLS 1.3になります。

| SSL/TLSバージョン | リリース年 | 利用状況 |
| :---: | :---: | :---: |
| SSL 1.0 | - | 利用不可 |
| SSL 2.0 | 1994年 | 利用不可 |
| SSL 3.0 | 1995年 | 利用不可 |
| TLS 1.0 | 1999年 | 非推奨 |
| TLS 1.1 | 2006年 | 非推奨 |
| TLS 1.2 | 2008年 | 推奨 |
| TLS 1.3 | 2018年 | 推奨 |

話をSSL Server Testの結果に戻しますと私のサイトは現在非推奨になっているTLS1.0/1.1が有効化されていることから評価が低いということになります。

## なぜTLS1.0/1.1が有効化されているか
AWS上でサイトを運用している場合、ALBを使ってロードバランシングしているかと思います。
ALBのHTTPSリスナーを作成するときになにも設定しなければデフォルトのセキュリティポリシーである`ELBSecurityPolicy-2016-08`をアタッチしてSSL通信を実現しています。
![](/images/ssl-check-test/image4.png)
AWSのドキュメントを参照するとこのポリシーはTLS1.0/1.1をサポートしていると記載しています。[^5]
セキュリティポリシーを変更しTLS1.0/1.1のサポートを無効化しますとTLS2.0以上に対応していないブラウザはサイト表示できなくなることから、AWSは互換性のため`ELBSecurityPolicy-2016-08`をお勧めしています。ただ主要なブラウザはTLS1.2以上のサポートを対応を完了しているのでTLS1.0/1.1のサポートを残す必要はないと考えています。
::: message
SSL Server Testに対応ブラウザの結果も出力されますが、よっぼどマイナーか5,6世代前のブラウザでない限りTLS2.0以上に対応しています。
:::

[^5]: https://docs.aws.amazon.com/ja_jp/elasticloadbalancing/latest/application/create-https-listener.html#describe-ssl-policies

# セキュリティポリシー変更
デフォルトのセキュリティポリシーを変更してもういちどSSL Server Testで診断してみます。
![](/images/ssl-check-test/image5.png)
![](/images/ssl-check-test/image6.png)

TLS1.0/1.1を無効化したことで評価がAに変わりました。
![](/images/ssl-check-test/image7.png)
セキュリティポリシーを変更しても私のスマートフォンやPCからは問題なく表示されていました。

::: message
TLS1.3が有効化されていませんが、AWSの最新セキュリティポリシーがまだ非対応みたいです。ちなみにAzureのL7Load BalancerであるApplication GatewayもTLS1.3はまだプレビュー段階でした。[^6] GCPは対応しているみたいです。[^7]
:::

[^6]: https://learn.microsoft.com/ja-jp/azure/application-gateway/application-gateway-ssl-policy-overview

[^7]: https://cloud.google.com/load-balancing/docs/ssl-policies-concepts?hl=ja

# HSTS Preload
セキュリティポリシーを変更しましたがまだ最高評価のA+になっていません。ここからさらに評価を上げるためにHSTS Preloadにドメインを登録します。

## HSTS
HSTS(Hypertext Strict Transport Security)は、HTTPでの通信を強制的にHTTPS通信にリダイレクトする仕組みです。Apacheやnginxの設定でレスポンスヘッダーに`Strict-Transport-Secutiry`を追加することでHSTSが有効化されます。

## HSTSの問題点
HSTSの仕組み上初回のアクセスはHTTP通信を許してしまうことが挙げられます。はじめてそのサイトにアクセスしたとき、そのアクセスがHTTP通信の場合レスポンス時に次回以降にHTTPS通信でアクセスするように返します。なので最初のアクセス時にセキュアな情報を持っていた場合、悪意ある第三者に盗み見られるリスクが出てきます。
この問題点を解決する仕組みがHSTS Preloadです。
https://hstspreload.org/

HTST PreloadはGoogleが提供する先読みサービスです。レスポンスヘッダーに`Strict-Transport-Security`を追加する設定をWebサーバーに設定した後、対象ドメインをHTST Preloadリストに登録することでブラウザ側でHTST対象のドメインを先読みし初回アクセスからHTTPS通信を実現します。

# HSTS設定
それではWordPressを動かしているApacheのレスポンスヘッダーに`Strict-Transport-Security`を追加します。
`.htaccess`に以下の記述を追加するだけです。

```conf
Header set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
```

WordPressにアクセスし、デベロッパーツールでレスポンスヘッダーを確認すると`Strict-Transport-Security`が追加されていることが確認できます。
![](/images/ssl-check-test/image9.png)

[HSTS Preloadリストの申請フォーム](https://hstspreload.org/)にドメインを入力して確認事項にチェックを入れてPreloadに申請します。
![](/images/ssl-check-test/image10.png)

申請ボタンを押しましたらチェックボタンを押して、ドメインが申請中になりましたらPreload申請完了となります。
![](/images/ssl-check-test/image11.png)
*実際に申請が完了するのは数か月後になります*

再度SSL Server Testで診断してみます。
![](/images/ssl-check-test/image12.png)

A＋評価になりました。

# 所感
SSL Server Testを活用してSSL設定の見直しを実施しました。
サイトをSSL化することの重要性は非エンジニアの人たちにも理解されているかと思いますが、適当設定すると逆に脆弱性が潜んでいる危険性があります。
皆さんもSSL Server Testを使って自分のサイトが問題なくSSL通信実現できているかぜひ確かめてみてください。

# 参考文献
https://tech.synapse.jp/entry/2020/11/03/120000
https://seven.ne.jp/qualys-ssl-labs-%E3%81%AE-ssl%E8%84%86%E5%BC%B1%E6%80%A7%E8%A8%BA%E6%96%ADssl-server-test%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6/
https://www.infraexpert.com/study/security7.html
https://www.ipa.go.jp/security/ipg/documents/ipa-cryptrec-gl-3001-3.0.1.pdf
https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Strict-Transport-Security
https://labcoat.jp/hsts-preload/