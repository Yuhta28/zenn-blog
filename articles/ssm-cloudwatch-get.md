---
title: "System Managerを使ってCloudWatchエージェントを導入・設定してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","CloudWatch","SystemManager"]
published: false
---

# 背景
以前いた会社ではFluentdとAWS Athenaを使ったログ集約基盤を構築を経験しました。
→詳しい内容はこちらの記事をご覧ください。
https://zenn.dev/yuta28/articles/ae8b1379aa873c
今の会社でもログ集約基盤の実装を担当していますが、今回はそこまで複雑なクエリ分析が不要である点、数百台のEC2のログ集約を実装したいという要望を踏まえて、CloudWatchエージェントを導入しCloudWatch Logsにログを集約する実装にしました。
とはいえ一台一台にCloudWatchエージェントをインストールするのは大変ですし、CloudWatchエージェントの設定ファイルを反映するのも面倒なので、AWS System Managerを使って複数のEC2に対して一括で設定反映できる実装に取り組んでみました。

# AWS System Managerについて
https://aws.amazon.com/jp/systems-manager/
EC2インスタンスやオンプレミスのサーバ管理をエージェントソフトによって管理し、面倒な運用タスクを自動化できます。

事前にSSMエージェントをインストールする必要がありますが、Ubuntu、Amazon Linux2の公式AMIには最初からプリインストールされています。

:::message alert
残念ながらCent OSにはプリインストールされていないので、エージェントをインストールする必要があります。
:::

System Managerでできることに以下の機能があります。

- SSH接続なしのサーバログイン
- 事前定義されたコマンドドキュメントからのコマンド実行
- 明示的なバージョン指定でのパッチ適用
- 定常タスクのオートメーション

