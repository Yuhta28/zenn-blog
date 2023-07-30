---
title: "Public IP Insightsで使っていないパブリックIPv4アドレスを整理しよう"
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

従来IPアドレスの課金体系はどのリソースにもアタッチされていないElastic IPアドレスを保持する場合かEC2のセカンダリとして追加でElastic IPをアタッチさせた場合に課金が発生しましたが、今回の発表で2024年2月からAWSが保有するすべてのパブリックIPアドレスに課金が発生します。

AWSは既存サービスの値下げを発表することはよくありましたが、今回のように実質的なサービスの値上げは珍しいものです。
この背景にはIPv4アドレスを取得するコストが年々高まっていることが理由に挙げられます。

>As you may know, IPv4 addresses are an increasingly scarce resource and the cost to acquire a single public IPv4 address has risen more than 300% over the past 5 years. This change reflects our own costs and is also intended to encourage you to be a bit more frugal with your use of public IPv4 addresses and to think about accelerating your adoption of IPv6 as a modernization and conservation measure.
DeepL訳)ご存じのように、IPv4アドレスはますます希少な資源となっており、単一のパブリックIPv4アドレスを取得するためのコストは、過去5年間で300％以上上昇しています。この変更は、私たち自身のコストを反映したものであると同時に、パブリックIPv4アドレスの使用をもう少し倹約し、近代化および保全対策としてIPv6の採用を加速することをお考えいただくためのものでもあります。

AWSはIPv4アドレスの利用を抑えてもらい、IPv6アドレスの採用を検討してほしいと同記事にて述べています。

一方でAWSはVPC IP Address Manager(IPAM)[^1]の機能に新しくPublic IP Insightsを発表しました。パブリックIPv4アドレスの監視や分析ができるため、この機能を使って社内で使われているパブリックIPアドレスの管理が効率的になります。
今回は自身のAWSアカウントでこのサービスを使いどのようにパブリックIPアドレスを管理できるか検証してみます。

[^1]: https://docs.aws.amazon.com/ja_jp/vpc/latest/ipam/what-it-is-ipam.html

# 対象読者
- AWSを利用するインフラ運用者
- クラウドコスト最適化を業務で担当される
- 社内のIPアドレスを管理される

# コスト試算
Public IP Insightsを利用する前に改めて今回のAWSで発表があったIPアドレスの課金体系の変更についておさらいします。
AWSで利用できるパブリックIPアドレスは4種類あります。

1. EC2のパブリックIPアドレス
2. Elastic IPアドレス
3. マネージドサービス IPアドレス
4. BYOIPアドレス

今まではどこにもアタッチされていない②のElastic IPアドレスもしくは稼働中のEC2でもセカンダリに登録した場合は料金が発生していました。
1個あたり$0.005/hで1か月あたり$3.6[^2]となります。

これが①でElastic IPアドレスを使っていない動的に変更されるパブリックIPアドレスを持つEC2にも課金が発生され、③のマネージドサービスが持つIPv4アドレス、つまりELB[^3]やNATゲートウェイ[^4]、Global Accelerators[^5]にアタッチされているIPv4アドレスにも課金が発生します。

④のBYOとはBring Your Ownの略でユーザー自身のIPアドレスを指します。ユーザーが自身のIPv4アドレスをAWSに導入して利用する場合[^6]は引き続き無料で利用できます。

## パブリックIPv4アドレスコスト早見表

| パブリックIPアドレスタイプ | 現在の課金体系 | 新しい課金体系 |
| ---- | ---- | ---- |
| EC2のパブリックIPアドレス | 無料 | $0.005/h |
| Elastic IPアドレス | $0.005/h | $0.005/h |
| マネージドサービス IPアドレス | 無料 | $0.005/h |
| BYOIPアドレス | 無料 | $0.005/h |

[^2]: 1か月720hで計算
[^3]: https://aws.amazon.com/jp/elasticloadbalancing/
[^4]: https://docs.aws.amazon.com/ja_jp/vpc/latest/userguide/vpc-nat-gateway.html
[^5]: https://aws.amazon.com/jp/global-accelerator/
[^6]: https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-byoip.html

## サンプルケース
以下のようなシンプルな構成で構築されたEC2+RDSアプリケーションを例にしてどれくらいパブリックIPv4アドレスの料金がかかるかみていきましょう。
![](/images/publicip-insights/image1.png)

このアプリケーションで使われているパブリックIPv4アドレスは以下のとおりです。

- NATゲートウェイ×2
- EC2×2
- ALB×2

ALBは作成時に2つ以上のサブネットを跨いだ状態で作成する必要があります。
複数のパブリックサブネットに跨いだ場合複数のパブリックIPアドレスが自動的に付与されます。

合計して6個のパブリックIPv4アドレスを利用しています。
1個あたり$0.005/hなので1か月稼働させた場合は$21.6でおよそ3,000円になります。
$0.005/h * 720h/月 * 6IP = $21.6 ≒ ￥3,024[^7]

[^7]: ￥140/$で計算

# Public IP Insights


# 所感

# 参考文献
https://zenn.dev/not75743/articles/bae546080bc543
https://aws.amazon.com/jp/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/
https://aws.amazon.com/jp/blogs/networking-and-content-delivery/identify-and-optimize-public-ipv4-address-usage-on-aws/