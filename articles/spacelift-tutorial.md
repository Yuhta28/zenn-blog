---
title: "CIをマネージしてくれるサービスSpaceliftを触ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CI","spacelift"]
published: false
---

# 概要
以前既存インフラをコード化する時に気を付けることという記事を書きました。
https://zenn.dev/yuta28/articles/iac-existing-infrastructure
そしてその記事を英訳したものをdev.toにも投稿しました。
https://dev.to/yuta28/keep-in-mind-when-coding-existing-infrastructure-15b3

するとLinkedInで私の記事をご覧になったエンジニアからメッセージを頂き、メッセージでやり取りしていたのですがそのエンジニアが所属されている企業のプロダクトに興味深いものがありました。
https://spacelift.io/

会社名を検索しても日本語記事はほとんど見つからず、詳細がわからなかったですが、会社ページのプロダクト概要に記載されているCI/CDのマネージドサービスが気になりましたので、チュートリアルを体験した感想について記載していきます。
