---
title: "TerraformでDatadogモニターを実装するならdatadog_monitor_jsonを使おう"
emoji: "🐶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["terraform","datadog","監視","iac"]
published: true
---

# 序論

Terraform[^1]にはDatadog[^2]の設定をコード管理できるプロパイダが提供されています。

https://registry.terraform.io/providers/DataDog/datadog/latest/docs

Terraformの監視を管理画面から設定すると担当者毎によって微妙に閾値の値がズレてたり、アラート通知のメッセージ表記ゆれがあるなどバラバラになりがちです。
昨今のクラウドインフラのIaCのようにDatadogの設定もTerraformでIaCすれば設定値が統一され、監視設定履歴もGit管理できるので新しく監視基盤をDatadogに移行する際にTerraformでコード化して監視設定やSLOを実装しました。
この時、Datadogの監視設定をTerraformでコード管理するために`datadog_monitor`リソースを使って実装しました。

https://registry.terraform.io/providers/DataDog/datadog/latest/docs/resources/monitor

以下一例ですがEC2のCPU使用率を監視するメトリクス監視をTerraformで書く場合このようになります。

```hcl:CPUモニター監視
resource "datadog_monitor" "cpu_alert" {
  name    = "EC2 CPU Monitor"
  query   = "avg(last_5m):avg:system.cpu.user{*} by {host} > 80"
  type    = "metric alert"
  message = <<-EOT
  {{#is_alert}}   
   @slack-infratest    
   <!here> High CPU! The host is {{host}}.  
   If you are able to analyze and resolve, 
   please chat for this alert message and then start to work on it.    
   {{/is_alert}}    
   
   {{#is_recovery}}  
   @slack-infratest  
    resolved  
  {{/is_recovery}}
  EOT

  renotify_interval = 60
  monitor_thresholds {
    critical          = 80
    critical_recovery = 70
  }
}
```

![](/images/datadog_monitor_json_terraform/image1.png)
*モニター画面*

ただDatadogの設定をTerraformのtfファイルに書くのはとても大変で`terraform plan`では問題なくてもapply時にエラーが起きて失敗するということが多々ありました。
例えば`query`に監視アラートを発砲するトリガー条件を書くのですがここに架空のメトリクスを書いても`terraform plan`は問題なく通ります。
ここはTerraformの文法上とは関係なくTerraform側でDatadogのメトリクス情報を持っていないから適当な値でも通ってしまうからです。
![](/images/datadog_monitor_json_terraform/image2.png)
*適当なメトリクス値*

会社ではTerraformの実行環境としてGitHub Actions上で動かし、mainブランチにPull Requestが入ったら`terraform plan`が実行し、問題なければマージして`terraform apply`を実行するというCI/CDパイプラインを構築していたのですが、マージしたのにエラーが発生しもう一度ブランチを切ってやり直すのは面倒で大変でした。

Datadogの監視設定はJSON出力が可能でコンソールから簡単に確認できるのですが、この出力結果をそのまま`datadog_monitor`に使うことは文法上できません。
![](/images/datadog_monitor_json_terraform/image3.png)

Datadog使い始めの時は一度手作業で監視設定を作った後にJSON出力してTerraformで必要なパラメータと見比べて記述したのですが、もう少し楽にできる方法がないのかなと思っていました。

その後TerraformのDatadogリソースに`datadog_monitor_json `リソースというJSON出力された監視設定値をそのまま使える便利なリソースがあることに気づきこっちを先に使えば良かったと後悔しました。
https://registry.terraform.io/providers/DataDog/datadog/latest/docs/resources/monitor_json

これからDatadogの設定をTerraformで構築する予定の人にはぜひこの`datadog_monitor_json`を使って実装してほしいと考えていますので`datadog_monitor_json`を使ったDatadog監視の方法について紹介いたします。


[^1]: https://www.terraform.io/
[^2]: https://www.datadoghq.com/ja/

# 対象読者

- Datadogで監視している人
- Terraformの使い方に慣れている人
- Datadogの設定をTerraformで管理したい人

# datadog_monitor_jsonの使い方

公式ドキュメントのサンプルコードを見ますと`monitor`プロパティ値にヒアドキュメントを使ってJSONを入力しています。

```hcl:サンプルコード
resource "datadog_monitor_json" "monitor_json" {
  monitor = <<-EOF
{
    "name": "Example monitor - service check",
    "type": "service check",
    "query": "\"ntp.in_sync\".by(\"*\").last(2).count_by_status()",
    "message": "Change the message triggers if any host's clock goes out of sync with the time given by NTP. The offset threshold is configured in the Agent's 'ntp.yaml' file.\n\nSee [Troubleshooting NTP Offset issues](https://docs.datadoghq.com/agent/troubleshooting/ntp for more details on cause and resolution.",
    "tags": [],
    "multi": true,
    "restricted_roles": null,
    "options": {
        "include_tags": true,
        "locked": false,
        "new_host_delay": 150,
        "notify_audit": false,
        "notify_no_data": false,
        "thresholds": {
            "warning": 1,
            "ok": 1,
            "critical": 1
        }
    },
    "priority": null,
    "classification": "custom"
}
EOF
}
```

もしくはfile関数[^3]を使ってJSONを外だしにする方法もあります。

```hcl:CPUモニター監視
resource "datadog_monitor_json" "cpu_alert" {
  monitor = file("./monitor.json")
}
```

```json:monitor.json
{
	"name": "EC2 CPU Monitor",
	"type": "query alert",
	"query": "avg(last_5m):avg:system.cpu.user{*} by {host} > 80",
	"message": "{{#is_alert}}   \r\n @slack-infratest    \r\n <!here> High CPU! The host is {{host}}.  \r\n If you are able to analyze and resolve, \r\n please chat for this alert message and then start to work on it.    \r\n {{/is_alert}}    \r\n   \r\n {{#is_recovery}}  \r\n @slack-infratest  \r\n  resolved  \r\n{{/is_recovery}}",
	"tags": [],
	"options": {
		"thresholds": {
			"critical": 80,
			"critical_recovery": 70
		},
		"notify_audit": false,
		"include_tags": true,
		"notify_no_data": false,
		"renotify_interval": 60,
		"require_full_window": true,
		"silenced": {},
		"new_group_delay": 300,
		"escalation_message": ""
	},
	"priority": null,
	"restricted_roles": null
}
```

ただ素のJSONファイルですと変数を入れることができず動的な値入力ができません。
その場合templatefile関数[^4]を使うことでJSONファイル内にTerraform変数を埋めることができ動的に設定を変更できます。

`monitor.json`ファイルを以下のように書き換えます。

```json:monitor.json
{
	"name": "${name}",
	"type": "query alert",
	"query": "avg(last_5m):avg:system.cpu.user{*} by {host} > 80",
	"message": "{{#is_alert}}   \n ${send_to_slack}    \n <!here> High CPU! The host is {{host}}.  \n If you are able to analyze and resolve, \n please chat for this alert message and then start to work on it.    \n {{/is_alert}}    \n   \n {{#is_recovery}}  \n ${send_to_slack}  \n  resolved  \n{{/is_recovery}}",
	"tags": [],
	"options": {
		"thresholds": {
			"critical": 80,
			"critical_recovery": 70
		},
		"notify_audit": false,
		"include_tags": true,
		"notify_no_data": false,
		"renotify_interval": 60,
		"require_full_window": true,
		"silenced": {},
		"new_group_delay": 300,
		"escalation_message": ""
	},
	"priority": null,
	"restricted_roles": null
}
```

モニター名を変数`name`、メッセージ内のSlackの通知先チャンネルを変数`send_to_slack`にしました。
先ほどの`datadog_monitor_json`を以下のように書き換えます。

```hcl:CPUモニター監視
resource "datadog_monitor_json" "monitor_json" {
  monitor = templatefile("./monitor.json", { name = "CPU Monitor",send_to_slack = "@slack-infratest" })
}
```

templatefile関数は(path,var)の引数で指定した変数値をJSON内の${...}で括られた変数に入れられます。
引数の中に直接変数値を入れずTerraformで宣言した変数もセットできます。

```hcl:CPUモニター監視
resource "datadog_monitor_json" "monitor_json" {
  monitor = templatefile("./monitor.json", { name = var.monitor_name,send_to_slack = var.notify_channel })
}
```

```hcl:variable.tf
variable "monitor_name" {
  type = string
}

variable "notify_channnel" {
  type = string
}
```

![](/images/datadog_monitor_json_terraform/image4.png)
*変数を入力*

[^3]: https://developer.hashicorp.com/terraform/language/functions/file
[^4]: https://developer.hashicorp.com/terraform/language/functions/templatefile

# 所感
`datadog_monitor_json`を使ったDatadogモニターの監視設定について紹介しました。
会社ではもう一通りの監視を実装し終えた後にこのリソースの存在に気づき残念ながら使う機会がなかったのが悔やまれます。
このリソースを使えば以下のように各監視モニターJSONファイルを監視タイプごとに格納してTerraformは監視モニターJSONファイルを値を入れて呼び出すようにすれば効率的な監視モニターのIaCが実現できると考えています。


```bash:サンプルディレクトリ
% tree
├── init.tf
├── metrics
│   ├── cpu.json
│   └── memory.json
├── process
│   ├── apache.json
│   └── nginx.json
└── monitor.tf
```

```hcl:monitor.tf
resource "datadog_monitor_json" "cpu_alert" {
  monitor = templatefile("./metrics/cpu.json", {name = "CPUモニター"})
}

resource "datadog_monitor_json" "memory_alert" {
  monitor = templatefile("./metrics/memory.json", {name = "メモリモニター"})
}

resource "datadog_monitor_json" "apache_alert" {
  monitor = templatefile("./process/apache.json", {name = "Apacheプロセスモニター"})
}

resource "datadog_monitor_json" "nginx_alert" {
  monitor = templatefile("./process/nginx.json", {name = "Nginxプロセスモニター"})
}
```

これからDatadogの監視をTerraformでIaC管理してみたい人はぜひtfファイル内に監視設定を書きだすのではなく、JSONファイルからの値呼び出しで設定するようにしてみてください。

# 参考文献
https://dev.classmethod.jp/articles/writing-iam-policy-with-terraform/