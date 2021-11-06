---
title: "System Managerを使ってCloudWatchエージェントを導入してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","CloudWatch","SystemManager"]
published: false
---

# 背景
以前いた会社ではFluentdとAWS Athenaを使ったログ集約基盤を構築を経験しました。
→詳しい内容はこちらの記事をご覧ください。
https://zenn.dev/yuta28/articles/ae8b1379aa873c
今回はそこまで複雑なクエリ分析が不要である点、数百台のEC2のログ集約を実装したい点を踏まえて、CloudWatchエージェントを導入してCloudWatch Logsにログを集約する実装にしました。
