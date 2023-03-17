---
title: "Datadog RUMを学んでみる"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["datadog", "監視", "RUM"]
published: true
published_at: 2023-03-08 11:30
---

# 概要
昨年会社の監視運用をDatadog[^1]に切り替えてから私が主な担当者としてモニタリング基盤の実装を進めています。
ホスト監視や外形監視、プロセス監視など前監視サービスで運用していた部分は大体移行が完了し、今はDatadogでよりビジネス分析に貢献できるようにDatadog RUMを使ったフロントエンド監視の実装を進めています。
Datadog RUMを使った監視運用自体はそのうち会社のテックブログで公開したいと考えていますが、私自身Datadog RUMで何ができるかもわからなかったので、どういったことができるのか調べてみようと考えました。
色々と調べたりしているうちにDatadogは学習サイトを提供しており、Datadogの使い方を学べるハンズオンやDatadogの資格など学ぶのにピッタリでしたので、今回はDatadogラーニングセンターでDatadog RUMを学んだことを紹介いたします。

[^1]: https://www.datadoghq.com/ja/

# RUMについて
Datadog RUMのハンズオンへ進む前にそもそもRUMとはなにかについて説明します。
RUMはReal User Monitoringの略で、ユーザーエクスペリエンスを測定することを目的としたフロントエンド監視で、ユーザーの行動、Webサイトアプリケーションパフォーマンスを収集・分析し**Webサイトのパフォーマンスがどのようにユーザー体験に影響を与えているのか**を調べることを目的としています。
以下の４つのユースケースを解決するのに役立ちます。
- パフォーマンス改善
    - Webページ、ユーザーアクション、フロントエンドコードのパフォーマンス追跡
- エラー原因特定
    - アプリケーションバグの監視、検知
- ユーザー分析
    - サイト利用者の位置情報、デバイスなどの収集、監視ページ訪問、クリック数の分析
- サポート
    - ユーザー問い合わせ対応に必要なセッション時間、リソースエラーの収集

2020年5月にGoogleはUXパフォーマンス監視の指標としてCore Web Vitalsを発表しました。
https://developers-jp.googleblog.com/2020/05/web-vitals.html

RUMはCore Web Vitalsを収集、評価することでユーザー体験向上に役立てることができます。

# Core Web Vitalsについて
Core Web Vitalsはユーザー体験を評価するために3つの指標があります。
- LCP(Largest Contentful Paint)[^2]
    - ページ内の最も大きい画像の描写時間の読み込み時間を測定
    - 2.5秒以下が目標値
- FID(First Input Delay)[^3]
    - ユーザーが最初にページ操作してから処理を開始するまでの時間を測定
    - 100ミリ秒以下が目標値
- CLS(Cumulative Layout Shift)[^4]
    - レイアウトのズレが発生する頻度を測定
    - 0.1以下が目標値

[^2]: https://web.dev/lcp/
[^3]: https://web.dev/fid/
[^4]: https://web.dev/cls/

もちろんDatadog RUMもCore Web Vitalsを測定し、ユーザーエクスペリエンス向上に役立つ分析ができます。

# Datadogラーニングセンター
https://learn.datadoghq.com/

前述したとおりDatadogには自前でDatadogを学べる学習サイトを提供しています。まだ数はそこまで多くありませんが、すべてのコースが無料で受講でき中には仮想環境上でDatadogを実際に触って理解を深められるコースもあります。
https://learn.datadoghq.com/collections

今回の記事の主題とは外れるので深掘りはしませんが、Datadogは認定資格も提供しているみたいでラーニングセンターから受験申し込みができます。
https://www.datadoghq.com/ja/certification/overview/
### 認定資格一覧
- Datadog Fundamentals[^5]
- Log Management Fundamentals[^6]
- APM and Distributed Tracing Fundamentals[^7]

受講できるコースの中にDatadog RUMを使って架空サイトのCore Web Vitalsを監視するというまさに私が学びたかったコースがありましたのでこちらを受講してみました。
https://learn.datadoghq.com/courses/core-web-vitals-lab



[^5]: https://www.datadoghq.com/pdf/Datadog_Fundamentals.pdf
[^6]: https://www.datadoghq.com/pdf/Log_Management_Fundamentals.pdf
[^7]: https://www.datadoghq.com/pdf/APM_Distributed_Tracing_Fundamentals.pdf

ここからはコースで学習したことをベースにメモを記します。

# Core Web Vitals Labハンズオン
## 1. デモ環境セットアップ
ハンズオンを開始すると仮想環境とデモ用のDatadogアカウントが発行されます。
![](/images/datadog-rum-handson/image1.png)
![](/images/datadog-rum-handson/image2.png)

デモ用の架空サイトを起動するためにdocker-composeコマンドを実行します。
![](/images/datadog-rum-handson/image3.png)
*デモ用ECサイト*


## 2. Google Lighthouseでの測定
Core Web VitalsはGoogle Lighthouse[^8]を使うことでブラウザ上からでも測定できます。
このハンズオンではGoogle Lighthouseによる測定方法も紹介しています。
![](/images/datadog-rum-handson/image4.png)
*Google LighthouseによるCWV測定*

[^8]: https://developer.chrome.com/docs/lighthouse/overview/


## 3. Datadog RUMでの測定
Datadog RUMを開始するためにはDatadog側でアプリケーションを作成し、アプリケーションにデプロイするためのコードスニペットを発行します。ハンズオンではすでにスニペットコードが挿入されていますのでアプリケーションを作成するだけで大丈夫です。

![](/images/datadog-rum-handson/image5.png)

アプリケーションをデプロイすると、冒頭のパフォーマンス改善やエラー分析など用途に応じたRUMのダッシュボードが作成されます。
![](/images/datadog-rum-handson/image6.png)
*パフォーマンス測定ダッシュボード*

![](/images/datadog-rum-handson/image7.png)
*エラー測定ダッシュボード*

測定以外にもDatadog RUMはユーザーのブラウジング体験をキャプチャして行動を視覚的に再生できるセッションリプレイという機能を提供しています。
https://docs.datadoghq.com/ja/real_user_monitoring/session_replay/

ユーザー毎に一意のセッションID付与し、そのユーザーの行動を追うことができます。
![](/images/datadog-rum-handson/image8.png)
*私のサイト操作を記録したセッションリプレイ*

## 4. Datadog RUMでCore Web Vitalsをチェック
パフォーマンスダッシュボードを確認しますとCore Web Vitalsを確認できますがLCPの値が目標値よりも悪いことがわかります。
![](/images/datadog-rum-handson/image9.png)
*読み込みに2.5秒以上かかる*

下の方までスクロールしますとページ別のCore Web Vitals情報が確認できトップページが最も遅いことがわかります。
![](/images/datadog-rum-handson/image10.png)

さらにスクロールしますとXHR[^9]やFetch API[^10]の実行についてもモニタリングしています。
LCPのスコアが高いのは`/discounts`と`/ads`リソースの実行感覚が長いことと関係しているのではないかと推測できます。
![](/images/datadog-rum-handson/image11.png)

セッションリプレイを使ってトップページでのRUMイベントについてチェックしてみます。
![](/images/datadog-rum-handson/image12.png)

Loading TimeがIn Progressではないものはデータ処理が完了しているイベントです。
より詳細を確認しますとJSやCSS、画像などの静的コンテンツでキャッシュが効いておらず時間がかかっていることがわかります。
![](/images/datadog-rum-handson/image13.png)

デモ環境を動かすソースコード修正できますのでキャッシュ設定を有効化した状態でdocker-composeを再起動しますとキャッシュが有効化されたデモ環境が開きます。
キャッシュが有効化された状態でLCPを確認しますとスコアが改善されていることがわかります。
![](/images/datadog-rum-handson/image14.png)

さらにハンズオン内で指示されている改善を続けていくとLCPスコアが目標値である2.5秒以下を出すことができました。
![](/images/datadog-rum-handson/image15.png)

以上がDatadog RUMを使ったアプリケーションパフォーマンス改善になります。

[^9]: https://developer.mozilla.org/ja/docs/Web/API/XMLHttpRequest
[^10]: https://developer.mozilla.org/ja/docs/Web/API/Fetch_API

# 所感
Datadog RUMのハンズオンを経験し、RUMの使い方や実装方法について学びました。
アプリケーションパフォーマンスの改善はSREとフロントエンドエンジニアが協力することで実現するということがわかりました。
今現在会社ではDatadog RUMの導入検証を進めていますが、ハンズオンで学んだことを基に会社サイトのパフォーマンス改善へつなげられるように頑張っていきます。
# 参考文献
https://web.dev/vitals-business-impact/
https://www.datadoghq.com/ja/product/real-user-monitoring/
https://www.splunk.com/ja_jp/data-insider/what-is-real-user-monitoring.html
https://memo.ag2works.tokyo/post-4424/