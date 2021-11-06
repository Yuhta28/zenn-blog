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
まず始めにSSMエージェントのインストールから始めます。
https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/agent-install-deb.html
SSMエージェントのインストール方法は公式ドキュメントに記載がありますが、これを一台一台に実行するのはやはり手間ですので簡単なAnsible Playbookを作成して一括導入します。

```yml:SSMエージェントインストールPlaybook
---
- hosts: all
  become: yes
  tasks:
  - name: install ssm-agent
    apt:
      deb: "https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/debian_amd64/amazon-ssm-agent.deb"
```

とても~~雑~~シンプルなPlaybookですがSSMエージェントをインストールするだけでしたら、これで大丈夫です。

インストールができたかどうかは`systemctl`コマンドで確認できます。

```bash
$ systemctl status amazon-ssm-agent
● amazon-ssm-agent.service - amazon-ssm-agent
   Loaded: loaded (/lib/systemd/system/amazon-ssm-agent.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2021-11-06 10:22:38 UTC; 7min ago
 Main PID: 2733 (amazon-ssm-agen)
    Tasks: 14 (limit: 1164)
   Memory: 30.3M
   CGroup: /system.slice/amazon-ssm-agent.service
           ├─2733 /usr/bin/amazon-ssm-agent
           └─2763 /usr/bin/ssm-agent-worker
```

インストールできましたら各EC2インスタンスに必要なIAMロールをアタッチさせます。
## IAMロールアタッチ
必要なIAMポリシーは`AmazonSSMManagedInstanceCore `になります。
これを新規に作成したIAMロールにアタッチさせて、このIAMロールもEC2インスタンスにアタッチさせます。
![](/images/ssm-cloudwatch/image3.png)

# System Manager実行
SSMエージェントをインストールし、IAMロールをアタッチさせましたらEC2がSystem Manager管理配下になります。
これでSystem Mananerの`Run Command`を使い、CloudWatchエージェントをインストールしてみます。

:::message alert
EC2とSystem Managerの接続に若干時間がかかります。
数分ほど待ってみてください。
:::

## Run Command実行
Run Commandではコマンドドキュメントという事前に定義されたコマンドを実行できます。
コマンドドキュメントはユーザーが自由に作成できますし、AWS側でテンプレートドキュメントも多く用意されています。
今回CloudWatchエージェントをインストールするドキュメントもすでに用意されていますのでそちらを利用します。
ドキュメント`AWS-ConfigureAWSPackage`を選択し、インストールするAWSパッケージにCloudWatchエージェントを指定後に、System Manager管理下のEC2インスタンスにエージェントインストールが実行されます。

![](/images/ssm-cloudwatch/image4.png)
![](/images/ssm-cloudwatch/image5.png)

実行が完了しましたらCloudWatchエージェントがインストールされていますので、同じように`systemctl`コマンドで確かめてみます。

```bash
$ systemctl status amazon-cloudwatch-agent
● amazon-cloudwatch-agent.service - Amazon CloudWatch Agent
   Loaded: loaded (/etc/systemd/system/amazon-cloudwatch-agent.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
```

インストールはされていますが、何も設定されていないので起動はされていない状態です。
CloudWatchエージェントにはウィザードで初期セットアップを簡単に済ませる機能がありますが、対象サーバーが多いと若干面倒な手順になります。
https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file-wizard.html

そこでここでもSystem Managerを使ってCloudWatchエージェントの設定ファイルを複数台に一括で適用させます。

## パラメータ作成
System Managerのパラメータストアにはパスワード情報やライセンスコードといった機密情報を格納したり、System Manager管理下のサーバーに設定ファイルを配置させることもできます。
パラメータストアにCloudWatchエージェントの設定ファイルを作成し、`Run Command`で配置、セットアップを行ないます。

```json:AmazonCloudWatch-linux
{
	"agent": {
		"run_as_user": "root"
	},
	"logs": {
		"logs_collected": {
			"files": {
				"collect_list": [
					{
						"file_path": "/home/bitnami/stack/apache/logs/access_log",
						"log_group_name": "Apache_access.log",
						"log_stream_name": "{local_hostname}"
					}
				]
			}
		}
	}
}
```

## Run Command再実行
ドキュメントは`AmazonCloudWatch-ManageAgent`を指定し、コマンドパラメータの`Optional Configuration Location`にパラメータストアで作成した設定ファイルを入力します。
![](/images/ssm-cloudwatch/image6.png)

コマンド実行が完了しますと、設定ファイルがEC2に格納されCloudWatchエージェントが起動されるようになります。

```bash
$ systemctl status amazon-cloudwatch-agent
● amazon-cloudwatch-agent.service - Amazon CloudWatch Agent
   Loaded: loaded (/etc/systemd/system/amazon-cloudwatch-agent.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2021-11-06 15:10:13 UTC; 14min ago
 Main PID: 2030 (amazon-cloudwat)
    Tasks: 7 (limit: 1164)
   Memory: 35.1M
   CGroup: /system.slice/amazon-cloudwatch-agent.service
           └─2030 /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent -config /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.toml -envconfig /opt/aws/amazon-cloudwatch-agent

Nov 06 15:10:13 Test-WordPress systemd[1]: Started Amazon CloudWatch Agent.
Nov 06 15:10:13 Test-WordPress start-amazon-cloudwatch-agent[2030]: /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json does not exist or cannot read. Skipping it.
Nov 06 15:10:13 Test-WordPress start-amazon-cloudwatch-agent[2030]: Valid Json input schema.
Nov 06 15:10:13 Test-WordPress start-amazon-cloudwatch-agent[2030]: I! Detecting run_as_user...

$ cat /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d/ssm_AmazonCloudWatch-linux
{
        "agent": {
                "run_as_user": "root"
        },
        "logs": {
                "logs_collected": {
                        "files": {
                                "collect_list": [
                                        {
                                                "file_path": "/home/bitnami/stack/apache/logs/access_log",
                                                "log_group_name": "Apache_access.log",
                                                "log_stream_name": "{local_hostname}"
                                        }
                                ]
                        }
                }
        }
}
```
これでCloudWatch Logsにログが転送されるようになりました。
![](/images/ssm-cloudwatch/image7.png)

# 所感
System Managerを使ってCloudWatchエージェントの導入・設定をしてみました。
CloudWatchエージェントの設定は面倒な手順も多く、Ansibleで一括反映するにしてもPlaybookの作りこみが面倒だと思いましたが、System Managerを使うことで当初想定していたよりもスムーズにログ集約基盤の構築が完了できました。

System Managerにはまだまだ色々な機能が備わっており、運用作業の自動化推進に向けて積極的に使っていこうと思います。

# 参考文献
https://github.com/dhoeric/ansible-aws-ssm
https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/systems-manager-parameter-store.html
https://www.sbcr.jp/product/4815609061/
