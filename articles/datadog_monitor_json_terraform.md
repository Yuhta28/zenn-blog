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

Terraformの監視設定を管理画面から設定するとその時の担当者によって微妙に閾値の値がズレてたり、アラート通知メッセージに表記ゆれがあるなどバラバラになりがちです。
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


[^1]: https://www.terraform.io/
[^2]: https://www.datadoghq.com/ja/