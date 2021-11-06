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
EC2インスタンスやオンプレミスのサーバー管理をエージェントソフトによって管理し、面倒な運用タスクを自動化できます。

事前にSSMエージェントをインストールする必要がありますが、Ubuntu、Amazon Linux2の公式AMIには最初からプリインストールされています。

:::message alert
残念ながらCentOSにはプリインストールされていないので、エージェントをインストールする必要があります。
:::

System Managerでできることに以下の機能があります。

- SSH接続なしのサーバーログイン
- 事前定義されたコマンドドキュメントからのコマンド実行
- 明示的なバージョン指定でのパッチ適用
- 定常タスクのオートメーション

今回は`Run Command`という事前定義されたコマンドドキュメントからのコマンド実行機能を利用して、CloudWatchエージェントのインストールとエージェントの設定反映を行い、複数WebサーバーのApacheログをCloudWatch Logsに集約できるように実装します。

![](/images/ssm-cloudwatch/image1.png)

# 下準備
EC2とSystem Managerの通信を実現するにはEC2に以下の設定が必要です。

- SSMエージェント導入
- System Managerとの通信を許可するIAMロールのアタッチ

今回EC2はBitnamiのWordPressの公式AMIから起動したものを使いました。
OS情報は以下のとおりDebianです。

```bash
$ neofetch
       _,met$$$$$gg.          bitnami@ip-10-0-0-56
    ,g$$$$$$$$$$$$$$$P.       --------------------
  ,g$$P"     """Y$$.".        OS: Debian GNU/Linux 10 (buster) x86_64
 ,$$P'              `$$$.     Host: HVM domU 4.11.amazon
',$$P       ,ggs.     `$$b:   Kernel: 4.19.0-18-cloud-amd64
`d$$'     ,$P"'   .    $$$    Uptime: 2 hours, 3 mins
 $$P      d$'     ,    $$P    Packages: 435 (dpkg)
 $$:      $$.   -    ,d$$'    Shell: bash 5.0.3
 $$;      Y$b._   _,d$P'      Terminal: /dev/pts/0
 Y$$.    `.`"Y$$$$P"'         CPU: Intel Xeon E5-2676 v3 (1) @ 2.394GHz
 `$$b      "-.__              GPU: Cirrus Logic GD 5446
  `Y$$                        Memory: 276MiB / 983MiB
   `Y$$.
     `$$b.
       `Y$$b.
          `"Y$b._
              `"""

```

このAMIから3台のEC2インスタンスを起動させましたので、まずはそれぞれにSSMエージェントを導入していきます。
![](/images/ssm-cloudwatch/image2.png)

## SSMエージェントインストール
