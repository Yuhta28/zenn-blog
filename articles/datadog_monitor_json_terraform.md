---
title: "TerraformでDatadogモニターを実装するならdatadog_monitor_jsonを使おう"
emoji: "🐶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["terraform","datadog","監視","iac"]
published: false
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

