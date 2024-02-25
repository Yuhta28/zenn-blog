---
title: "Terramateを使えばIaCは豊かになれるのか?"
emoji: "🧑🏻‍🤝‍🧑🏻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["terramate","terraform","aws","iac"]
published: false
---

# 序論

先日LinkedInで気になるIaCツールを紹介してもらいました。
https://terramate.io/

TerramateはIaCオーケストレーションツールであり、Terraformを小さく分離し、並列に実行することでパイプラインの高速化、Blast Radius[^1]の減少を抑えていると記載されています。

Terramate経由でTerraformを実行すると聞くとTerragrunt[^2]のようなラッパーツールかと思いましたが、TerramateにはTerragruntとは明確に違う部分がありましたのでTerramateを使ったIaC開発について紹介いたします。


# Terramateについて

[GitHub](https://github.com/terramate-io/terramate/tree/main)で公開されているOSS版を見るとTerramateはGoで作られているようです。創業者のブログを見るとリリースしたのは2022年5月みたいで3年ほど前にリリースされた比較的新しい開発ツールのようです。

https://blog.terramate.io/introducing-terramate-an-orchestrator-and-code-generator-for-terraform-5e538c9ee055


[^1]: クラウドコンピューティング上でセキュリティインシデントが発生した時の影響範囲
[^2]: https://terragrunt.gruntwork.io/