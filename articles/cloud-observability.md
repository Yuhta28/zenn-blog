---
title: "監視から可観測性へ"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["監視","可観測性","AWS"]
published: false
---

# 概要
先日弊社の監視運用をMackerelからDatadogへ切り替えました。その時監視だけでなく、OpenSearchによるログ集約運用も見直しそちらもDatadogの中にまとめました。
![](/images/cloud-observability/image1.png)

Datadogは前の会社でも使っていましたが、すでに導入済みで特に私自身使う機会がなかったので今回の導入に向けて色々とDatadogの導入方法や監視そのものの運用について色々と調べてみました。
そのなかで可観測性という用語が気になりましたので、私が調べて感じた事、AWSで実践できるワークショップの紹介も兼ねて紹介いたします。



# 参考資料
https://aws.amazon.com/jp/products/management-and-governance/use-cases/monitoring-and-observability/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc&blog-posts-cards.sort-by=item.additionalFields.createdDate&blog-posts-cards.sort-order=desc
https://licensecounter.jp/devops-hub/blog/splunk1/
https://www.oreilly.co.jp/books/9784873118642/