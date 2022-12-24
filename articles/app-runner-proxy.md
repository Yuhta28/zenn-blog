---
title: "App Runnerで気になったところ"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "apprunner"]
published: false
---

:::message alert
この記事は結論がありません。調べてみましたが原因がわからなかったので詳しい人いましたらアドバイスいただけますと幸いです。
:::

# 背景
会社でDatadogによるモニタリング基盤の構築を実装しています。最初はメディアサーバーやDBを監視するためにエージェント起動用のEC2を用意してそこからサーバーへの外形監視やDB監視を構築していました。
ただ導入しているエージェントを動かすためだけにサーバーを用意するのものもったいなかったのでコンテナにDatadogエージェントを導入し、そこから起動するようにしました。
せっかくなので昨年リリースされたApp Runner[^1]でDatadogエージェントコンテナを構築してみました。
[^1]: https://aws.amazon.com/jp/apprunner/

![](/images/app-runner-proxy/image7.png)
*アーキテクチャ図*

App RunnerならECR[^2]にコンテナイメージをセットするだけで面倒なネットワーク周りの設定も簡略化でき、監視メディアを追加する際もリポジトリにプッシュするだけでコンテナの自動デプロイで簡単に増やすことができるので、運用が楽になると思ったのが採用理由になります。
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
DatadogエージェントがApp Runnerと相性が良くないのではと思いましたが、curlで単純なレスポンスタイムを取得するコンテナを用意してFargateからとApp Runnerからそれぞれ計測してみることにしました。

```Dockerfile:Dockerfile
FROM  curlimages/curl:latest

CMD  curl -s -o /dev/null -w '%{time_starttransfer}' <うちのメディアURL>
```

ローカルPCからこのコンテナを起動すると対象URLのレスポンスタイムを取得してくれます。

```console
$ docker build -t curl-yuta .
$ docker run --rm curl-yuta
0.252717
```

FargateとApp Runnerから取得できたレスポンスタイムはこちらです。

### Fargateコンテナ
![](/images/app-runner-proxy/image1.png)

### App Runnerコンテナ
![](/images/app-runner-proxy/image2.png)

Datadogエージェントほど露骨ではありませんが、curlでもApp Runner経由で実行した方がレスポンスタイムが遅くなっていることがわかります。

# 謎のコンテナ
Datadogではエージェントを導入したホスト/コンテナが自動的にダッシュボードに登録されます。
![](/images/app-runner-proxy/image4.png)
*コンテナ画面*

FargateでDatadogエージェントコンテナを導入した時は1台登録されていました。
