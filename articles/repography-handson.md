---
title: "Gitコミットをイイ感じに視覚化してくれるRepographyを触ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHub","Repography","Git", "markdown"]
published: true
---

# 概要
GitHubにはリポジトリにインストールして、各種アプリケーションにアクセスを許可するGitHub Appsという仕組みがあります。
[GitHub marketplace](https://github.com/marketplace)では、さまざまなAppsを手に入れることができます。

昨今、ソフトウェア開発チームのパフォーマンスを示す4つの要素(=Four Keys[^1])が重要視され、エンジニア組織はいかに速くビジネス成果を出すかが問われています。
このアプリを導入することで開発チームがどれだけコミットできているか可視化してくれるためパフォーマンス成果を対外アピールする時に役立ちます。
Repographyがどういったアプリケーションなのか使い方を含めて紹介いたします。

[^1]: https://cloud.google.com/blog/ja/products/gcp/using-the-four-keys-to-measure-your-devops-performance

# 対象読者
- 

# Repographyとは
https://github.com/marketplace/repography
Repographyは、マークダウン形式のヴィジュアライズなダッシュボードを提供してくれるGitHub Appsです。`README.md`に埋め込むだけで過去のコミットやIssue、上位コントリビューターのランキングが表示されます。
[Repography](https://repography.com/)のサイトにアクセスすればデモ用にいくつかのオープンソースのGitHubリポジトリにインストール済みのRepographyが確認できます。
![](/images/repography-handson/image1.png)
- [clap](https://github.com/clap-rs/clap)
- [neovim](https://github.com/neovim/neovim)
- [strawberry](https://github.com/strawberry-graphql/strawberry)
- [xdebug](https://github.com/xdebug/xdebug)

# 使い方
Repography Appをインストール後にリポジトリの選択ができますので自分のGitHubリポジトリで追加したいリポジトリを選びます。

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
インストールできましたら、Repographyと連携させたいリポジトリを選択します。
![](/images/repography-handson/image3.png)
後はデモで用意されたリポジトリと同じようにダッシュボードが出力されます。
![](/images/repography-handson/image4.png)

右のフォーマットからマークダウン形式かreStructuredText形式を選択できます。
コードをコピーして、`README.md`に張り付ければGitHubリポジトリからもダッシュボードの内容を確認できます。
# ポスター
ダッシュボード作成メニューの下にポスターという項目があります。
![](/images/repography-handson/image5.png)
かなり変わったサービスですが、Repographyで出力した内容をポスターとしてオフィスに飾ることができるようです。
![](/images/repography-handson/image6.png)
![](/images/repography-handson/image7.png)
*ポスターイメージ*

ポスター料金は60ユーロなので大体8000円くらいですね。送料込みみたいですが、日本へ発送してくれるのでしょうか🤔
と思ったら、PNGイメージとしてダウンロードできるみたいですので画像だけダウンロードして、印刷は地元のポスター制作屋さんに注文してみるのも良さそうです。PCの壁紙に最適な16:9のサイズもダウンロードできますので、自分のPCに張り付けて思い出にするのも良いでしょう。
![](/images/repography-handson/image8.png)
*値段は5€(700円)でリーズナブル*
![](/images/repography-handson/image9.png)

# 所感
Repositoryについて紹介してみました。
自分のコミット量をダッシュボードで可視化してくれるので、日々の学習のモチベーションやチームでOSS開発するときに導入すれば、頑張りが可視化されるのでモチベーション向上にもつながるかもしれませんね。
ポスター制作は日本まで送ってくれるかは不明ですが、画像イメージだけダウンロードすれば印刷を外注できますので、気になる企業さんは検討してみてはいかがでしょうか💡
オシャレなテック企業っぽくなり自社の宣伝になるかもしれませんね。
# LT資料
LTのスライド資料です。
@[speakerdeck](704db58db30d49a8ae9f90b3631c6c08)

# 英語記事
英訳しました。
https://dev.to/yuta28/repography-makes-github-repository-beautiful-3dn3
# 参考文献
https://eng-blog.iij.ad.jp/archives/2242