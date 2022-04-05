---
title: "進化したGitHubのコード検索機能を試してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHub"]
published: true
---

# 概要
昨年、GitHubはコード検索を強化しました。具体的にどう強化されたのか紹介いたします。
https://twitter.com/github/status/1468626754474594316

# 従来の検索
従来、GitHubの検索機能は、検索Wordを入力するだけでリポジトリ名、言語、ファイル名、ユーザー名といった細かいフィルタリングはできませんでした。
https://github.com/search
例えば、自分のリポジトリ内にある`README.md`を検索しようとしても、ソースコード内に`README.md`という文字列が含まれているファイルしか検索できませんでした。
![](/images/gh-codesearch-preview/image1.png)

# コード検索機能
昨年末にプレビュー版がリリースされたコード検索は、便利な機能を兼ね備えています。
プレビュー版ですので、いきなり使うことはできず事前登録する必要があります。
後日GitHubからコード検索のプレビューが利用可能になる案内がきますので、記載されているリンクをクリックします。
![](/images/gh-codesearch-preview/image2.png)
*利用案内メール*

こちらが初期画面になります。
![](/images/gh-codesearch-preview/image3.png)
https://cs.github.com/
いくつか例文が記載されていることからわかるとおり、正規表現による検索や言語別、オーガナイザー別での検索もできます。
例えばオーガナイザーがAWS、Goで記述されたファイルを検索したいときは、検索バーに`org:AWS language:Go`と入力します。
![](/images/gh-codesearch-preview/image4.png)
*検索結果*

他に、最初の例示で自分の全リポジトリの`README.md`を検索したいときは、検索スコープを検索バーに`owner:Yuhta28  path:README.md`と入力します。
![](/images/gh-codesearch-preview/image5.png)
*自分のリポジトリ内に存在するREADMEファイル*

# Tips
コード検索の検索範囲はデフォルトではGitHub上の全リポジトリが対象ですが、変更もできます。
![](/images/gh-codesearch-preview/image6.png)
*自分のリポジトリ内のみを検索範囲にする*
スコープのカスタイマイズをすることで自分所有以外のリポジトリを検索範囲に含めて検索できます。
![](/images/gh-codesearch-preview/image7.png)
![](/images/gh-codesearch-preview/image8.png)
*Microsoftリポジトリ内のJavaファイルのみを検索に含める*

![](/images/gh-codesearch-preview/image9.png)
*Javaで作成されたAzure関係のファイルを検索*

# 所感
強化されたGitHubのコード検索機能について体験してみました。
今までGitHub内のファイル検索は使いにくいところもあり、自社内のリポジトリ内から特定のファイルを探すのも苦労しましたが、この機能なら探すのが簡単になりそうなので、皆さんも興味ありましたらぜひ使ってみてください。


# 紹介動画
https://www.youtube.com/watch?v=UOIPBfPXkus

# 参考文献
https://www.publickey1.jp/blog/21/github_8.html