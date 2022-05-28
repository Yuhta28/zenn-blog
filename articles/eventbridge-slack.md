---
title: "EventBridgeで監視してChatbot経由でSlackに検知させてみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","eventbridge","slack","chatbot"]
published: true
---

# 概要
弊社では監視サービスとしてはてな社の[Mackerel](https://ja.mackerel.io/)を利用してAWSリソースを監視しています。
先日Mackerelがメンテナンスのため3時間ほどサービスが利用できないとアナウンスがありました。
https://mackerel.io/ja/blog/entry/announcement/20211108
システム監視するサービスであるとのことからメンテナンス時間は平日水曜日の14:00~17:30でした。
この時間帯にEC2が落ちているなど異常があった場合、Mackerelで検知できないため一時的な代替措置が必要となりました。
あくまでも代替的な措置なのであまり作りこまずにサクッとできるもので運用しようと思い、Amazon EventBrdigeでEC2のステータスを監視して停止状態になったらAWS ChatbotでSlackに通知する機能を実装してみました。

# AWS Chatbotについて
https://aws.amazon.com/jp/chatbot/
AWS Chatbotは2020年にGA(一般公開)された「ChatOps」[^1]を実現するためのインタラクティブエージェントであり、SlackやAmazon ChimeにAWSリソースのモニタリング結果を簡単に連携できます。
2021年に大幅なアップデートが行なわれEventBridgeによって処理されるすべてのサービスイベントを通知させることが可能となりました。
https://dev.classmethod.jp/articles/chatbot-supports-all-events-from-eventbridge/

これによってEC2のステータスが変更されたイベントをトリガーにSNSでChatbotに知らせてSlackに通知を送れるようになりました。
次から実際に実装した方法について紹介します。

[^1]: チャット(Chat)をベースにシステム運用(Ops)を意味する造語

# アーキテクチャ
今回構築するアーキテクチャは以下の通りです。
![](/images/eventbridge-slack/image1.png)
EC2が障害などで停止した場合、EC2のステータスが変更されたイベントをトリガーにSNSでChatbotに知らせてSlackに通知を送れるようにしています。
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
SNSを確認すると自動でサブスクリプションも登録されているので、これで連携していることが確認できます。
![](/images/eventbridge-slack/image6.png)

## EventBridge
もともとEventBrdigeはCloudWatch Eventsをベースに構築されたサービスです。
CloudWatch Eventsと同じAPI、エンドポイントを利用していますのでCloudWatch Eventsを使っていた人なら問題なく慣れると思います。
メニューサイドバーからルールを選び、ルール作成画面からパターン定義を決めます。
一定時間ごとのスケジュール呼び出しかイベント条件によって呼び出すイベントパターンがありますが、ここではイベント条件を指定します。
![](/images/eventbridge-slack/image5.png)
図ではEC2のステータスが`stopped`になったらトリガーするイベントを指定しています。
:::message
右を見ればわかりますが、イベントパターンはJSON形式で書くこともできますので、使いまわしするイベントは保存して別のパターン定義にコピーもできます。
:::

# 検証
一連のAWSリソースが作成できましたので、EC2をシャットダウンさせてSlackに通知がくるか確認します。

## 起動中
![](/images/eventbridge-slack/image7.png)

## 停止直後
![](/images/eventbridge-slack/image8.png)
![](/images/eventbridge-slack/image9.png)
SlackにEC2が停止した通知が来ました。

# できないこと
Slackに通知を簡単に飛ばしてくれるChatbotですが、その分カスタマイズ性があまりよろしくありません。
例えば上図のメッセージ内容を編集したり、@hereをつけてメンション通知することは残念ながらできません。

> Q: AWS Chatbot通知にカスタムフォーマットを追加できますか?
> いいえ。AWS Chatbotの通知のフォーマットをカスタマイズすることはできません。

https://aws.amazon.com/jp/chatbot/faqs/

こうしたカスタマイズをしたい場合、従来通りLambda関数を作成してSlackに通知させます。

# 所感
EventBridgeとChatbotを使ってEC2に対して簡易的な監視を実装しました。
本来ならCloudWatchエージェントをインストールして、メトリクス監視も行ない異常検知することが大事ですが、今回のように一時的な監視であればこれくらいで大丈夫かなと思います。
幸いなことにMackerelメンテナンス期間中に大きな障害は起こらず、会社のEC2は問題なく稼働していたのでよかったです。
MackerelやDatadogのようなSaaS系監視サービスを導入している企業も多いと思いますが、サービス提供元の都合でサービスが使えなくなる可能性もありますのでその時に備えて簡易的な代替機能は用意したほうがよさそうです。

# 余談
昔はこんな感じでCloudWatch Event(当時)からLambda連携してSlackに通知させていましたが、あれからChatbotの機能もアップデートされてできることが増えました。
昔の記事と今とでは状況が異なりますので、自分も記事作成する際やほかの人のブログ記事を参照する場合は気を付けようと思います。
https://zenn.dev/yuta28/articles/d6abcdf226107944197b

# 参考文献
https://www.itmedia.co.jp/enterprise/articles/1707/14/news044.html
https://aws.amazon.com/jp/builders-flash/202006/slack-chatbot/?awsf.filter-name=*all
https://tech-cci.io/archives/6411