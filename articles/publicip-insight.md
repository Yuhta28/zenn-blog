---
title: "AWSのIPアドレス料金体系が変わりました"
emoji: "😲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","ipv4"]
published: true
---

# 序論
先日AWSから以下のアナウンスがありました。
https://aws.amazon.com/jp/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/

> We are introducing a new charge for public IPv4 addresses. Effective February 1, 2024 there will be a charge of $0.005 per IP per hour for all public IPv4 addresses, whether attached to a service or not (there is already a charge for public IPv4 addresses you allocate in your account but don’t attach to an EC2 instance).
DeepL訳)パブリックIPv4アドレスに新しい料金を導入します。2024年2月1日より、サービスに接続されているかどうかにかかわらず、すべてのパブリックIPv4アドレスに対して1時間あたりIPあたり0.005ドルが課金されます（アカウントに割り当てられているがEC2インスタンスに接続されていないパブリックIPv4アドレスにはすでに課金があります）。

従来IPアドレスの料金体系はどのリソースにもアタッチされていないElastic IPアドレスを保持する場合かEC2のセカンダリとして追加でElastic IPをアタッチさせた場合に課金が発生しましたが、今回の発表で2024年2月からAWSが保有するすべてのパブリックIPアドレスに課金が発生します。

AWSは既存サービスの値下げを発表することはよくありましたが、今回のように実質的なサービスの値上げは珍しいものです。
この背景にはIPv4アドレスを取得するコストが年々高まっていることが理由に挙げられます。

>As you may know, IPv4 addresses are an increasingly scarce resource and the cost to acquire a single public IPv4 address has risen more than 300% over the past 5 years. This change reflects our own costs and is also intended to encourage you to be a bit more frugal with your use of public IPv4 addresses and to think about accelerating your adoption of IPv6 as a modernization and conservation measure.
DeepL訳)ご存じのように、IPv4アドレスはますます希少な資源となっており、単一のパブリックIPv4アドレスを取得するためのコストは、過去5年間で300％以上上昇しています。この変更は、私たち自身のコストを反映したものであると同時に、パブリックIPv4アドレスの使用をもう少し倹約し、近代化および保全対策としてIPv6の採用を加速することをお考えいただくためのものでもあります。

AWSはIPv4アドレスの利用を抑えてもらい、IPv6アドレスの採用を検討してほしいと同記事にて述べています。

一方でAWSはVPC IP Address Manager(IPAM)[^1]の機能に新しくPublic IP Insightsを発表しました。パブリックIPv4アドレスの監視や分析ができるため、この機能を使って社内で使われているパブリックIPアドレスの管理が効率的になります。
どのように料金体系が変わったのか、またPublic IP Insightsとはどのようなサービスなのか併せて調べてみます。

[^1]: https://docs.aws.amazon.com/ja_jp/vpc/latest/ipam/what-it-is-ipam.html

# 対象読者
- AWSを利用するインフラ運用者
- AWSのコストを気にする人
- 社内ので利用しているパブリックIPv4アドレスを管理する人

# コスト試算
Public IP Insightsを利用する前に改めて今回のAWSで発表があったIPアドレスの料金体系の変更についておさらいします。
AWSで利用できるパブリックIPアドレスは4種類あります。

1. EC2のパブリックIPアドレス
2. Elastic IPアドレス
3. サービス管理のパブリックIPアドレス
4. BYOIPアドレス

今まではどこにもアタッチされていない②のElastic IPアドレスもしくは稼働中のEC2でもセカンダリに登録した場合は料金が発生していました。
1個あたり$0.005/hで1か月あたり$3.6[^2]となります。

これが①でElastic IPアドレスを使っていない動的に変更されるパブリックIPアドレスを持つEC2にも課金が発生され、③のマネージドサービスが持つIPv4アドレス、つまりELB[^3]やNATゲートウェイ[^4]、Global Accelerators[^5]にアタッチされているIPv4アドレスにも課金が発生します。

④のBYOとはBring Your Ownの略でユーザー自身のIPアドレスを指します。ユーザーが自身のIPv4アドレスをAWSに導入して利用する場合[^6]は引き続き無料で利用できます。

## パブリックIPv4アドレスコスト早見表

| パブリックIPアドレスタイプ | 現在の料金体系 | 新しい料金体系 |
| ---- | ---- | ---- |
| EC2のパブリックIPアドレス | 無料 | $0.005/h |
| Elastic IPアドレス | $0.005/h | $0.005/h |
| サービス管理のパブリックIPアドレス | 無料 | $0.005/h |
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
https://aws.amazon.com/jp/about-aws/whats-new/2023/07/aws-public-ip-insights-vpc-ip-address-manager/

Public IP InsightsはIPAMに新しく追加された機能ですべてのパブリックIPv4アドレスを一元管理されたダッシュボードで可視化でき、監視や分析を容易にできます。リージョン単位でダッシュボードが分かれているため全リージョンを一括で可視化はできませんが、1リージョン内で使われているパブリックIPv4アドレスの使用状況や前述したパブリックIPアドレスタイプ別の分類ができます。

![](/images/publicip-insights/image2.png)
*Public IP Insightsコンソール画面*

現時点ではパブリックIPアドレスのタイプ別グラフ、Elastic IPアドレスの使用状況グラフ、リージョン内で利用しているパブリックIPアドレスのリスト表示ができるようです。
Public IP Insightsは今のところすべての機能を無料で使えるみたいです。

# 所感
新しく発表されたIPアドレスの料金体系とPublic IP Insightsについて紹介しました。
冒頭にも記載しましたが、AWSが既存サービスの値上げが生じる変更を発表するのは珍しく大変驚きました。
IPv4アドレスは2進数の32桁で表現され、アドレス総数は2の32乗個、約43億もの数があります。
インターネットの普及に伴い、PCやスマートフォン、ルーターなどの数が増えIPv4アドレスの供給以上に需要が発生するようになりました。

IPv4アドレス枯渇に伴いIPv6アドレスへの移行が叫ばれますが、IPv4アドレスとIPv6アドレスには互換性がなく移行は中々進んでいない状況です。

::: message
総務省の発表では国内事業者が運営するWebサイトにおけるIPv6アドレスの対応状況は9.8%程でした。[^8]
:::

AWSではIPv6アドレスを採用できるようにIPv6に対応したALBやNATゲートウェイの利用方法について紹介しています。
また先日発表されたEC2 Instance Connect Endpoint[^9]などパブリックIPv4アドレスがなくてもEC2にSSHやRDP接続できる機能がアップデートされています。

私もIPv6アドレスについて理解を深めてきたり料金体系変更までに備えたいと思います。

[^8]: https://www.soumu.go.jp/main_content/000891745.pdf
[^9]: https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/Connect-using-EC2-Instance-Connect-Endpoint.html

# 参考文献
https://zenn.dev/not75743/articles/bae546080bc543
https://aws.amazon.com/jp/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/
https://aws.amazon.com/jp/blogs/networking-and-content-delivery/identify-and-optimize-public-ipv4-address-usage-on-aws/
https://www.nic.ad.jp/ja/newsletter/No26/090.html#:~:text=IPv4%E3%82%A2%E3%83%89%E3%83%AC%E3%82%B9%E3%81%AF2%E9%80%B2%E6%95%B0,%E3%81%AB%E7%BD%AE%E3%81%8D%E6%8F%9B%E3%82%8F%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AF%E3%81%82%E3%82%8A%E3%81%BE%E3%81%9B%E3%82%93%E3%80%82