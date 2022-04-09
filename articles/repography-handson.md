---
title: "Gitコミットをイイ感じに視覚化してくれるRepographyをさわってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHub","Repography","Git"]
published: false
---

# 概要
GitHubにはリポジトリにインストールして、各種アプリケーションにアクセスを許可するGitHub Appsという仕組みがあります。
[GitHub marketplace](https://github.com/marketplace)では、さまざまなAppsを手に入れることができます。
今回は、GitHubリポジトリへのコミットを視覚化し、`README.md`をリッチにしてくれるRepographyというアプリケーションをさわってみました。

# Repographyとは
https://github.com/marketplace/repography
Repographyは、マークダウン形式のヴィジュアライズなダッシュボードを提供してくれるGitHub Appsです。`README.md`に埋め込むだけで過去のコミットやIssue、上位コントリビューターのランキングが表示されます。
[Repography](https://repography.com/)のサイトにアクセスすればデモ用にいくつかのオープンソースのGitHubリポジトリにインストール済みのRepographyが確認できます。
![](/images/repography-handson/image1.png)
- [clap](https://github.com/clap-rs/clap)
- [neovim](https://github.com/neovim/neovim)
- [strawberry](https://github.com/strawberry-graphql/strawberry)
- [xdebug](https://github.com/xdebug/xdebug)

適当なインストール済みのOSSをクリックすれば、そのOSSのアウトプット情報をRepographyがダッシュボードに出力してくれます。
## [![Repography logo](https://images.repography.com/logo.svg)](https://repography.com) / Recent activity [![Time period](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_badge.svg)](https://repography.com)
[![Timeline graph](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_timeline.svg)](https://github.com/neovim/neovim/commits)
[![Issue status graph](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_issues.svg)](https://github.com/neovim/neovim/issues)
[![Pull request status graph](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_prs.svg)](https://github.com/neovim/neovim/pulls)
[![Trending topics](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_words.svg)](https://github.com/neovim/neovim/commits)
[![Top contributors](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_users.svg)](https://github.com/neovim/neovim/graphs/contributors)

## [![Repography logo](https://images.repography.com/logo.svg)](https://repography.com) / Structure
[![Structure](https://images.repography.com/0/clap-rs/clap/structure/f00e021e8d4f56f5a659737a2301b4c1_table.svg)](https://github.com/clap-rs/clap)

## [![Repography logo](https://images.repography.com/logo.svg)](https://repography.com) / Top contributors
[![Top contributors](https://images.repography.com/0/clap-rs/clap/top-contributors/d751713988987e9331980363e24189ce_table.svg)](https://github.com/clap-rs/clap/graphs/contributors)

GitHub MarketplaceからRepographyをクリックしますと、いくつかプランを選択します。
![](/images/repography-handson/image2.png)
無料プランと複数の有料プランがありますが、基本的な機能に大きな違いはなく、プライベートリポジトリの数の上限を増やすことができるようです。

# 参考文献
https://eng-blog.iij.ad.jp/archives/2242