---
title: "CloudWatch Eventで監視してChatbot経由でSlackに検知させてみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","cloudwatch","slack","chatbot"]
published: false
---

# 概要
弊社では監視サービスとしてはてな社の[Mackerel](https://ja.mackerel.io/)を利用してAWSリソースを監視しています。
先日Mackerelがメンテナンスのため3時間ほどサービスが利用できないとアナウンスがありました。
https://mackerel.io/ja/blog/entry/announcement/20211108
システム監視をするサービスであるとのことからメンテナンス時間は平日水曜日の14:00~17:30となっています。
この時間帯にEC2が落ちているなど異常があった場合、Mackerelで検知できないため一時的な代替措置が必要となりました。
あくまでも代替的な措置なのであまり作りこまずにサクッとできるもので運用しようと思い、調べてみましたがCloudWatchで監視してAWS ChatbotでSlackに通知する機能を実装してみました。

# AWS Chatbotについて
https://aws.amazon.com/jp/chatbot/
AWS Chatbotは昨年GA(一般公開)された「ChatOps」[^1]を実現するためのインタラクティブエージェントであり、SlackやAmazon ChimeにAWSリソースのモニタリング結果を簡単に連携できます。
今年大きくアップデートが行われEventBridgeによって処理されるすべてのサービスイベントを通知させることが可能となりました。
https://dev.classmethod.jp/articles/chatbot-supports-all-events-from-eventbridge/
今回はCloudWatchメトリクスからイベントを取得してSNSからChatbotに送信してSlackに通知させますが、EventBridgeに対応するイベントでしたらLambdaを作る必要がなくなるのでChatOpsがかなり推進やすくなったと感じます。

[^1]: チャット(Chat)をベースにシステム運用(Ops)を意味する造語

# 実装



# 参考文献
https://www.itmedia.co.jp/enterprise/articles/1707/14/news044.html