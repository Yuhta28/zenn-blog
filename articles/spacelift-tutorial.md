---
title: "CIをマネージしてくれるサービスSpaceliftを触ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CI","spacelift"]
published: false
---

# 概要
以前既存インフラをコード化するときに気を付けることという記事を書きました。
https://zenn.dev/yuta28/articles/iac-existing-infrastructure
そしてその記事を英訳したものをdev.toにも投稿しました。
https://dev.to/yuta28/keep-in-mind-when-coding-existing-infrastructure-15b3

するとLinkedInで私の記事をご覧になったエンジニアからメッセージを頂き、やり取りしていたのですがそのエンジニアが所属されている企業のプロダクトが面白そうでしたので、ハンズオンしてみて感想を書いてみました。
https://spacelift.io/

# Spaceliftとは
Spaceliftは、Spacelift社が開発したクラウドインフラのIaCを支援するサービスです。
日本語情報が一切なく公式ページの概要を都度都度DeepL翻訳しながら、どうやら[Terraform Cloud](https://cloud.hashicorp.com/products/terraform)や[Atlantis](https://www.runatlantis.io/)のようにTerraformの自動実行を提供してくれるサービスのようです。

また昨今注目されている`Policy as Code`と呼ばれるコードでポリシーを定義、管理して柔軟なアクセス制御を実現しインフラ構築の権限管理もできるようです。
