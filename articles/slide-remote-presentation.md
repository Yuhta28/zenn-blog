---
title: "Google Slidesをスマホから操作したい"
emoji: "📲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googleslides", "発表", "スマホ"]
published: false
---

# 背景
LTを発表する際に利用するMicrosoft PowerPoint/Google Slidesにはプレゼンテーションモードがあります。
最近オフラインでもLTを実施する機会が増え、私もよく自分のPCを持ち込んで発表しています。オンラインの時はスライドの切り替えに直接PCの矢印キーを使っていましたがオフラインだとPCの前から離れにくく、スクリーンと登壇席が遠いとスライドへの指さしができないのでちょっと不便だなと最近感じました。
![](/images/slide-remote-presentation/image1.png)
*こんな感じで指さししたい*
市販のレーザーポインターを使えば遠くからでも強調でき、スライドの切り替えもポインタからできることも実現できます。
![](/images/slide-remote-presentation/image2.webp)
*こういうのとか良さそう[^1]*

[^1]: https://www.microsoft.com/ja-jp/d/microsoft-%E3%83%95%E3%83%AC%E3%82%BB%E3%83%B3%E3%82%BF%E3%83%BC/8w5mkmpdxm3v?activetab=pivot:%E6%A6%82%E8%A6%81tab

ただ個人のLTでここまで器具を用意するのも少し面倒で有名メーカーの安いものでも4,000-5,000円くらいしますので、できればスマホから操作して簡単に切り替えたいと思いました。
いくつかそうしたツールやアプリがないかなと探しまして良さそうなものを見つけましたので紹介いたします。

# Unified Remote
https://www.unifiedremote.com/
このアプリはPC操作をスマホから行なえるリモートデスクトップアプリになります。Windows、Mac、Linux、ラズベリーパイなど幅広いホストに対応し、スマホアプリもiOS、Android共にあります。(Windows Phoneもありました)
## 使い方
PC側でUnified Remoteサーバーを起動しましたら、スマホ側でクライアントアプリを立ち上げます。
![](/images/slide-remote-presentation/image3.png)
*PC側*

![](/images/slide-remote-presentation/image4.png)
*スマホアプリからリモート先PCを選択*
しばらくするとスマホアプリ側で操作メニューが立ち上がります。
![](/images/slide-remote-presentation/image6.png)
*接続された状態のスマホアプリ*

Basic Inputがマウス操作になります。1タップで左クリックしたことになりますのでスマホからスライドページを次に移動できます。ただこれでは戻るができないので誤タップした場合スマホからでは戻せない問題があります。広告なしの有料版を購入すれば仮想キーボードを使え、矢印キー操作ができます。
![](/images/slide-remote-presentation/image7.png)
*左下の拡張キーボードは有料版で利用可能*


# Remote for Slides
https://chrome.google.com/webstore/detail/remote-for-slides/pojijacppbhikhkmegdoechbfiiibppi
Google Slides専用のクローム拡張機能になりますが、Remote for Slidesを使えばスマホからスライドページを戻すこともできます。
