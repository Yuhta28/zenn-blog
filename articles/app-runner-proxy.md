---
title: "ドキュメントから読み取れなかったApp Runnerで気になったこと"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "apprunner", "datadog"]
published: true
---

:::message alert
この記事は結論がありません。調べてみましたが原因がわからなかったのでApp Runnerに詳しい人がいましたらアドバイスいただけますと幸いです。
:::

# 背景
会社でDatadogによるモニタリング基盤の構築を実装しています。最初はメディアサーバーやDBを監視するためにエージェント起動用のEC2を用意してそこからサーバーへの外形監視やDB監視を構築していました。
ただ導入しているエージェントを動かすためだけにサーバーを用意するのはもったいなかったのでコンテナにDatadogエージェントを導入し、そこから起動するようにしました。
せっかくなので昨年リリースされたApp Runner[^1]でDatadogエージェントコンテナを構築してみました。
[^1]: https://aws.amazon.com/jp/apprunner/

![](/images/app-runner-proxy/image7.png)
*アーキテクチャ図*

App RunnerならECR[^2]にコンテナイメージをセットするだけで面倒なネットワーク周りの設定も簡略化でき、監視メディアを追加する際もレジストリにプッシュするだけでコンテナの自動デプロイで簡単に増やすことができるので、運用が楽になると思ったのが採用理由になります。
:::message
EC2からECSへのDatadogエージェント移行は後日会社のテックブログに詳しい解説を掲載します。
:::

ただApp Runnerでメディアサーバーを外形監視すると奇妙なことがありました。
[^2]: https://aws.amazon.com/jp/ecr/

# レスポンスタイム劣化
App Runner上のDatadogエージェントから弊社のメディアサーバーに外形監視し、取得したレスポンスタイムのグラフ結果は以下のようになります。
![](/images/app-runner-proxy/image3.png)
*メディアサーバー毎のレスポンスタイム取得値*

最初に数メディアをECS Fargateから監視し挙動を確認した後に、DatadogエージェントをApp Runnerに乗せ換えてみました。それがお昼時くらいのことですが移行直後から急激にレスポンスタイムが上昇していることがわかります。最初は移行直後だからと様子見しましたが、丸一日経っても改善しなかったため翌日にFargateに切り戻ししました。

# Curl検証
DatadogエージェントがApp Runnerと相性が良くないのではと思いましたが、curlで単純なレスポンスタイムを取得するコンテナ[^3]を用意してFargateとApp Runnerからそれぞれ計測してみることにしました。
[^3]: https://hub.docker.com/r/curlimages/curl

```Dockerfile:Dockerfile
FROM  curlimages/curl:latest

CMD  curl -s -o /dev/null -w '%{time_starttransfer}' <うちのメディアURL>
```

```console:ローカルからの実行例
$ docker build -t curl-yuta .
$ docker run --rm curl-yuta
0.252717
```

FargateとApp Runnerから取得できたレスポンスタイムはこちらです。

### Fargateコンテナ
![](/images/app-runner-proxy/image1.png)

### App Runnerコンテナ
![](/images/app-runner-proxy/image2.png)

Datadogエージェントほど露骨ではありませんが、curlでもApp Runner経由で取得した方がレスポンスタイムが遅くなっていることがわかります。

# 謎のコンテナ
Datadogではエージェントを導入したホスト/コンテナが自動的にダッシュボードに登録されます。

### Fargateコンテナ
![](/images/app-runner-proxy/image4.png)

FargateでDatadogエージェントコンテナを導入したときは1台登録されていました。
次にApp Runner上でDatadogエージェントをデプロイしたときの画面です。

### App Runnerコンテナ
![](/images/app-runner-proxy/image5.png)

不思議なことに2台のエージェントコンテナが動いています。`aws-fargate-request-proxy`というコンテナが動いていますが、このコンテナの詳細がApp Runnerのドキュメントからは見つけることができませんでした。
あくまでも私の予想になるのですが、App Runner上にデプロイされたコンテナは直接外部通信するのではなく、あいだにプロキシコンテナが存在しその分時間が遅くなるのではないかと考えています。

![](/images/app-runner-proxy/image8.png)
*予想アーキテクチャ図*

もちろん私の推測なので違っていましたらアドバイスいただけますと幸いです。

# 現状
事業部から新規メディアリリースに伴う監視対象の追加依頼が発生した際は監視メディアのURLを編集したリストをGitHubにプッシュ、ECRにイメージがビルドされるとApp Runnerがトリガーとなり追加されたメディアのURLを監視するといった自動化を実現したかったのですが、ユーザー体感に近いレスポンスタイムを取得できないとなると諦めざるをえませんでした。
![](/images/app-runner-proxy/image9.png)
*想定していた自動図*

現在はFargate上でDatadogエージェントをデプロイし、監視するようにしています。
![](/images/app-runner-proxy/image6.png)
幸いなことにGitHub ActionsからECS Fargateへの自動デプロイはテンプレートがありましたのでやりたかったことはすんなり実現できました。
こちらに関しては会社のテックブログに近日公開しようと思いますので興味ありましたらそちらもご覧ください。

# 所感
App Runnerをチュートリアル以外ではじめて使ってみましたが、残念ながら仕事に活用できませんでした。
個人的には簡単にコンテナのデプロイができたので、別の機会で使ってみたいなと思います。

# 参考文献
https://zenn.dev/taxin/articles/curl-time-measure
https://www.thousandeyes.com/ja/blog/measuring-performance-with-http-proxies
https://docs.datadoghq.com/ja/integrations/http_check/