---
title: "EventBridgeで監視してChatbot経由でSlackに検知させてみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","eventbridge","slack","chatbot"]
published: false
---

# 概要
弊社では監視サービスとしてはてな社の[Mackerel](https://ja.mackerel.io/)を利用してAWSリソースを監視しています。
先日Mackerelがメンテナンスのため3時間ほどサービスが利用できないとアナウンスがありました。
https://mackerel.io/ja/blog/entry/announcement/20211108
システム監視をするサービスであるとのことからメンテナンス時間は平日水曜日の14:00~17:30でした。
この時間帯にEC2が落ちているなど異常があった場合、Mackerelで検知できないため一時的な代替措置が必要となりました。
あくまでも代替的な措置なのであまり作りこまずにサクッとできるもので運用しようと思い、Amazon EventBrdigeでEC2のステータスを監視して停止状態になったらAWS ChatbotでSlackに通知する機能を実装してみました。
### 余談
昔はこんな感じでCloudWatch Event(当時)からLambda連携してSlackに通知させていましたが、あれからChatbotの機能もアップデートされてできることが増えました。
https://zenn.dev/yuta28/articles/d6abcdf226107944197b

# AWS Chatbotについて
https://aws.amazon.com/jp/chatbot/
AWS Chatbotは昨年GA(一般公開)された「ChatOps」[^1]を実現するためのインタラクティブエージェントであり、SlackやAmazon ChimeにAWSリソースのモニタリング結果を簡単に連携できます。
今年大幅なアップデートが行われEventBridgeによって処理されるすべてのサービスイベントを通知させることが可能となりました。
https://dev.classmethod.jp/articles/chatbot-supports-all-events-from-eventbridge/

これによってEC2のステータスが変更されたイベントをトリガーにSNSでChatbotに知らせてSlackに通知を送れるようになりました。
次から実際に実装した方法について紹介します。

[^1]: チャット(Chat)をベースにシステム運用(Ops)を意味する造語

# アーキテクチャ
今回構築するアーキテクチャは以下の通りです。
![](/images/eventbridge-slack/image1.png)
EC2が障害か何かで停止した場合、EC2のステータスが変更されたイベントをトリガーにSNSでChatbotに知らせてSlackに通知を送れるようにしています。
## SNS
まず最初にSNSの作成からはじめます。
SNSトピックの作成は特別な設定は不要でスタンダードタイプを選択し、任意の名前をつけるだけでOKです。
![](/images/eventbridge-slack/image3.png)

## Chatbot
ChatbotのページからSlackをクライアントツールとして選択し、連携します。
![](/images/eventbridge-slack/image2.png)
ChatbotはIncoming Webhook不要でSlackへの権限リクエストを許可すれば連携できますので、Slackに詳しくない人でも簡単に連携できると思います。
自分のSlack名が連携されたら通知先チャンネルの設定ができますので、順番に設定します。
アクセス許可部分は新規に作成する場合テンプレートで自動作成してくれますので、任意のロール名を決めるだけでOKです。
最後の通知オプションでは先ほど作成したSNSトピックを選択することで、SNSトピックに登録されたイベントを受け取ることができます。
![](/images/eventbridge-slack/image4.png)

## EventBridge
元々EventBrdigeはCloudWatch Eventsをベースに構築されたサービスです。
CloudWatch Eventsと同じAPI、エンドポイントを利用していますのでCloudWatch Eventsを使っていた人なら問題なく慣れると思います。
メニューサイドバーからルールを選び、ルール作成画面からパターン定義を決めます。
一定時間ごとのスケジュール呼び出しかイベント条件によって呼び出すイベントパターンがありますが、ここではイベント条件を指定します。
![](/images/eventbridge-slack/image5.png)
図ではEC2のステータスが`stopped`になったらトリガーするイベントを指定しています。
:::message
右を見ればわかりますが、イベントパターンはJSON形式で書くこともできますので、使いまわしするイベントは保存して別のパターン定義にコピーもできます。
:::


# 参考文献
https://www.itmedia.co.jp/enterprise/articles/1707/14/news044.html
https://aws.amazon.com/jp/builders-flash/202006/slack-chatbot/?awsf.filter-name=*all
https://tech-cci.io/archives/6411