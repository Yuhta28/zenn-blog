---
title: "Public IP Insightsで使っていないIPアドレスを整理しよう"
emoji: "🤓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","ipv4"]
published: false
---

# 序論
先日AWSから以下のアナウンスがありました。
https://aws.amazon.com/jp/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/

> We are introducing a new charge for public IPv4 addresses. Effective February 1, 2024 there will be a charge of $0.005 per IP per hour for all public IPv4 addresses, whether attached to a service or not (there is already a charge for public IPv4 addresses you allocate in your account but don’t attach to an EC2 instance).
DeepL訳)パブリックIPv4アドレスに新しい料金を導入します。2024年2月1日より、サービスに接続されているかどうかにかかわらず、すべてのパブリックIPv4アドレスに対して1時間あたりIPあたり0.005ドルが課金されます（アカウントに割り当てられているがEC2インスタンスに接続されていないパブリックIPv4アドレスにはすでに課金があります）。

従来IPアドレスの課金体系はどのリソースにもアタッチされていないElastic IPアドレスを保持する場合のみに課金が発生しましたが、今回の発表でリソースにアタッチされているAWSが保有するすべてのパブリックIPアドレスに課金が発生します。

AWSは既存サービスの値下げを発表することはよくありましたが、今回のように実質的なサービスの値上げは珍しいものです。
この背景にはIPv4アドレスを取得するコストが年々高まっていることが理由に挙げられます。

>As you may know, IPv4 addresses are an increasingly scarce resource and the cost to acquire a single public IPv4 address has risen more than 300% over the past 5 years. This change reflects our own costs and is also intended to encourage you to be a bit more frugal with your use of public IPv4 addresses and to think about accelerating your adoption of IPv6 as a modernization and conservation measure.
DeepL訳)ご存じのように、IPv4アドレスはますます希少な資源となっており、単一のパブリックIPv4アドレスを取得するためのコストは、過去5年間で300％以上上昇しています。この変更は、私たち自身のコストを反映したものであると同時に、パブリックIPv4アドレスの使用をもう少し倹約し、近代化および保全対策としてIPv6の採用を加速することをお考えいただくためのものでもあります。

AWSはIPv4アドレスの利用を抑えてもらい、IPv6アドレスの採用を検討してほしいと同記事にて述べています。

一方でAWSはVPC IP Address Manager(IPAM)の機能に新しくPublic IP Insightsを発表しました。パブリックIPv4アドレスの監視や分析ができるため、この機能を使って社内で使われているパブリックIPアドレスの管理が効率的になります。
今回は自身のAWSアカウントでこのサービスを使いどのようにパブリックIPアドレスを管理できるか検証してみます。

# 対象読者
- AWSを利用するインフラ運用者
- クラウドコスト最適化を業務で担当される
- 社内のIPアドレスを管理される

# コスト試算
AWSで利用できるパブリックIPアドレスは4種類あります。

1. EC2のパブリックIPアドレス
2. Elastic IPアドレス
3. マネージドサービス IPアドレス
4. BYOIPアドレス


# 所感

# 参考文献
https://zenn.dev/not75743/articles/bae546080bc543
https://aws.amazon.com/jp/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/
https://aws.amazon.com/jp/blogs/networking-and-content-delivery/identify-and-optimize-public-ipv4-address-usage-on-aws/