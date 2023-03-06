---
title: "Datadog RUMを学んでみる"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["datadog", "監視", "RUM"]
published: false
---

# 概要
昨年会社の監視運用をDatadog[^1]に切り替えてから私が主な担当者としてモニタリング基盤の実装を進めています。
ホスト監視や外形監視、プロセス監視など前監視サービスで運用していた部分は大体移行が完了し、今はDatadogでよりビジネス分析に貢献できるようにDatadog RUMを使ったフロントエンド監視の実装を進めています。
Datadog RUMを使った監視運用自体はそのうち会社のテックブログで公開したいと考えていますが、私自身Datadog RUMで何ができるかもわからなかったので、どういったことができるのか調べてみようと考えました。
色々と調べたりしているうちにDatadogは学習サイトを提供しており、Datadogの使い方を学べるハンズオンやDatadogの資格など学ぶのにピッタリでしたので、今回はDatadogラーニングセンターでDatadog RUMを学んだことを紹介いたします。

[^1]: https://www.datadoghq.com/ja/

# RUMについて
Datadog RUMについて話す前にそもそもRUMとはなにかについて説明します。
RUMはReal User Monitoringの略で、ユーザーエクスペリエンスを測定することを目的としたフロントエンド監視で、ユーザーの行動、Webサイトアプリケーションパフォーマンスを収集・分析し**Webサイトのパフォーマンスがどのようにユーザー体験に影響を与えているのか**を調べることを目的としています。
以下の４つのユースケースを解決するのに役立ちます。
- パフォーマンス改善
    - Webページ、ユーザーアクション、フロントエンドコードのパフォーマンス追跡
- エラー原因特定
    - アプリケーションバグの監視、検知
- ユーザー分析
    - サイト利用者の位置情報、デバイス等の収集、監視しページ訪問、クリック数の分析
- サポート
    - ユーザー問い合わせ対応に必要なセッション時間、リソースエラーの収集

2020年5月にGoogleはUXパフォーマンス監視の指標としてCore Web Vitalsを発表しました。
https://developers-jp.googleblog.com/2020/05/web-vitals.html

RUMはCore Web Vitalsを収集、評価することでユーザー体験向上に役立てることができます。

# Core Web Vitalsについて
Core Web Vitalsはユーザー体験を評価するために3つの指標があります。
- LCP(Largest Contentful Paint)
- FID(First Input Delay)
- CLS(Cumulative Layout Shift)
# Datadogラーニングセンター
https://learn.datadoghq.com/

前述したとおりDatadogには自前でDatadogを学べる学習サイトを提供しています。まだ数はそこまで多くありませんが、すべてのコースが無料で受講でき中には仮想環境上でDatadogを実際に触って理解を深められるコースもあります。
https://learn.datadoghq.com/collections

今回の記事の主題とは外れるので深掘りはしませんが、Datadogは認定資格も提供しているみたいでラーニングセンターから受験申し込みができます。
https://www.datadoghq.com/ja/certification/overview/
### 認定資格一覧
- Datadog Fundamentals[^2]
- Log Management Fundamentals[^3]
- APM and Distributed Tracing Fundamentals[^4]

受講できるコースの中にDatadog RUMを使って架空サイトのCore Web Vitalsを監視するというまさに私が学びたかったコースがありましたのでこちらを受講してみました。
https://learn.datadoghq.com/courses/core-web-vitals-lab



[^2]: https://www.datadoghq.com/pdf/Datadog_Fundamentals.pdf
[^3]: https://www.datadoghq.com/pdf/Log_Management_Fundamentals.pdf
[^4]: https://www.datadoghq.com/pdf/APM_Distributed_Tracing_Fundamentals.pdf

ここからはコースで学習したことをベースにメモを記します。

# Datadog RUMについて


Datadog RUM

- step1 コアウェブバイタルとは何か
- step2 コアウェブバイタルを取得するためのRUMのセット
- step3 コアウェブバイタルのトラッキング(ブラウザ)
- step4 コアウェブバイタルのトラッキング(Datadog RUM)
- step5 プロダクションでコアウェブバイタルレポートを理解するためにRUMを使う
- step6 悪いコアウェブバイタルレポートとは何か





# 参考文献
https://web.dev/vitals-business-impact/
https://www.datadoghq.com/ja/product/real-user-monitoring/
https://www.splunk.com/ja_jp/data-insider/what-is-real-user-monitoring.html