---
title: "TerraformからPulumiへの移行方法"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","terraform","typescript","aws"]
published: false
---

# 序論
先日、Terraform開発元であるHashiCorp社から今後リリースするすべての製品においてライセンスを変更するアナウンスが発表されました。
https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license

ライセンス変更による影響はこちらの記事などをご覧いただけるとわかりやすいと思います。
https://zenn.dev/the_exile/articles/b90fe8c5c41694

このライセンス変更はHashiCorp社と競合する企業に対して競合プロダクト内にコミュニティ版HashiCorp製品を無償で利用できなくなるもので、Terraformなどを利用するユーザー自身には影響はありません。Terraform Cloudと競合するサービスを提供するSpacelift[^1]やenv0[^2]がこのライセンス変更に対して自身らのプロダクトの今後について述べております。2社ともこのライセンス変更に対して賛成の立場をとり、今後自分たちのプロダクトについてどうするか弁護士と相談すると記載しています。[^3][^4]

一方で同じプロビジョニングツールとして知られるPulumiへの関心が高まっています。
私が以前書いたPulumiの記事[^5]に急にイイねが増えており、閲覧数も急上昇していました。
![](/images/migration-terraform-to-pulumi/image1.png)

先ほどのライセンス変更の詳細を紹介した記事が公開されて以降日本でもPulumiについて検索する回数が増えているようです。
![](/images/migration-terraform-to-pulumi/image2.png)
*GoogleTrendsより*

しかしすでに多く企業、個人でインフラのコード化を実現する場合ツールにTerraformを採用している人は多いと思います。
Pulumiは他のプロビジョニングツールからの移行を提供する手順について紹介[^6]していますので、TerraformからPulumiに移行する方法について紹介していきます。

[^1]: https://spacelift.io/
[^2]: https://www.env0.com/
[^3]: https://spacelift.io/blog/hashicorps-license-change
[^4]: https://www.env0.com/blog/hashicorp-license-change
[^5]: https://zenn.dev/yuta28/articles/pulumi-ai-revolutionary
[^6]: https://www.pulumi.com/docs/using-pulumi/adopting-pulumi/migrating-to-pulumi/

# 対象読者

- Pulumiについて知りたい人
- TerraformからPulumiへの移行に興味がある人

# 所感

# 参考文献
