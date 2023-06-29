---
title: "Repographyで開発チームのパフォーマンスを可視化してみよう"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHub","Repography","Git", "markdown"]
published: true
---

# 概要
昨今、ソフトウェア開発チームのパフォーマンスを示す4つの要素(=Four Keys[^1])が重要視され、エンジニア組織はいかに速くビジネス成果を出すかが問われています。
Repographyを導入することで開発チームがどれだけコミットできているか可視化してくれるためパフォーマンス成果を対外にアピールする時に役立ちます。
Repographyがどういったアプリケーションなのか使い方を含めて紹介いたします。

[^1]: https://cloud.google.com/blog/ja/products/gcp/using-the-four-keys-to-measure-your-devops-performance

# 対象読者
- 自社のソフトウェア開発チームのパフォーマンスを可視化したい人
- 商用サービスを使う前にOSSで可視化できないか知りたい人
- オフィスをテック企業っぽくしたい人

# Repographyとは
https://github.com/marketplace/repography
Repographyは、リポジトリ単位でどれだけのユーザーがコミットしたかをグラフィカルに可視化してくれるGitHub Apps[^2]です。
マークダウン形式で出力されるので、対象リポジトリの`README.md`に埋め込めば過去のコミットやIssue、上位コントリビューターのランキングが表示されます。

無料版と有料版が存在しますが、機能的な違いはなくプライベートリポジトリの上限数のみとなっていまして無料版ならプライベートリポジトリは一個まで使えます。
![](/images/repography-handson/image12.png)
*パブリックリポジトリなら制限なし*

[^2]: https://repography.com/about

# 使い方
Repographyの使い方について説明します。

## Step1 リポジトリ選択
Repography Appをインストール後、リポジトリの選択ができますので自分のGitHubリポジトリで追加したいリポジトリを選びます。
![](/images/repography-handson/image10.png)

## Step2 プロダクト選択
次にダッシュボードを選択します。
後述しますが、コミット量に応じたポスター制作もここで選べます。
![](/images/repography-handson/image11.png)

## Step3 ダッシュボード設定
現在提供されているダッシュボードは3種類です。
私のリポジトリだとコミット数が少なく見栄えが悪いため、ここからはデモ用に提供されているNeovim[^3]のRepographyで説明します。

[^3]: https://neovim.io/

### Recent activity
Recent activityでは直近のコミット数が可視化されます。コミット数以外にもIssueやPRの起票数やステータス状況なども可視化されます。コードのPRやIssueがどれだけ残っているかが一目でわかるのでパフォーマンスの可視化に最適です。

 [![Time period](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_badge.svg)](https://repography.com)
[![Timeline graph](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_timeline.svg)](https://github.com/neovim/neovim/commits)
[![Issue status graph](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_issues.svg)](https://github.com/neovim/neovim/issues)
[![Pull request status graph](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_prs.svg)](https://github.com/neovim/neovim/pulls)
[![Trending topics](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_words.svg)](https://github.com/neovim/neovim/commits)
[![Top contributors](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_users.svg)](https://github.com/neovim/neovim/graphs/contributors)
[![Activity map](https://images.repography.com/0/neovim/neovim/recent-activity/d751713988987e9331980363e24189ce_map.svg)](https://github.com/neovim/neovim/commits)

### Structure
Structureではリポジトリ内のどのファイルに対してコミットされたかが平面図上で可視化されます。コミット数も表示されているので、特定機能のソースコードがどれだけコミットされたかが一目でわかります。

[![Structure](https://images.repography.com/0/neovim/neovim/structure/f00e021e8d4f56f5a659737a2301b4c1_table.svg)](https://github.com/neovim/neovim)

### Top contributors
Top contributorsはそのリポジトリに対するコミット数が多い人をランキング形式で表示してくれます。タイムラインも表示されるので直近の活動量についても把握できます。

[![Top contributors](https://images.repography.com/0/neovim/neovim/top-contributors/d751713988987e9331980363e24189ce_table.svg)](https://github.com/neovim/neovim/graphs/contributors)

出力されるマークダウンも記法をコピーできるのでRepographyにログインしなくても、`README.md`に張り付ければいつでも確認できます。
![](/images/repography-handson/image4.png)

# ポスター作成
開発チームのパフォーマンスをただ可視化するだけでなく、オフィスに飾ることでチーム内外にアピールしたいと考える人もいらっしゃるでしょう。(多分)

かなり変わったサービスですが、Repographyで出力した内容をポスターとしてオフィスに飾ることができるようです。
![](/images/repography-handson/image5.png)

そのリポジトリ内の活動がイラスト化されるので、誰がどれだけコミットしてきたかがポスターとして会社に飾ることができます。
![](/images/repography-handson/image6.png)
![](/images/repography-handson/image7.png)
*ポスターイメージ*

料金はポスターサイズによってまちまちで$150~$250のようです。
今のところEU、イギリス、アメリカ、オーストラリアは送料無料で送れるみたいですが日本にも配送を対応してくれるかは不明です。別途送料を支払えば送ってくれるのでしょうか🤔

一部ポスターはPNGイメージとしてダウンロードできるみたいですので画像ファイルだけダウンロードして、地元のポスター制作屋さんに印刷注文してみるほうが安く済ませられるかもしれません。
PCの壁紙に最適な16:9のサイズもダウンロードできますので、自分のPCに張り付けて思い出にするのも良いでしょう。
![](/images/repography-handson/image8.png)
*値段は$5(700円)でリーズナブル*

![](/images/repography-handson/image9.png)
*サンプルイメージ*

# 所感
Repographyについて紹介してみました。
自社のソフトウェア開発チームのパフォーマンスについて非エンジニアに説明するのは中々難しいと思います。
GitHubのコミット数やPR、Issueのステータスをビジュアライズすることでビジネスサイドへの理解も深まると思います。

開発チームの頑張りをポスターとしてオフィスに飾ればオシャレなテック企業っぽくなり自社の宣伝になるかもしれませんね💡
気になる人はぜひ試してみてください。

# LT資料
LTのスライド資料です。
@[speakerdeck](704db58db30d49a8ae9f90b3631c6c08)

# 英語記事
英訳しました。
https://dev.to/yuta28/repography-makes-github-repository-beautiful-3dn3
# 参考文献
https://eng-blog.iij.ad.jp/archives/2242
https://repography.com/blog