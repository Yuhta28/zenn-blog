---
title: "WindowsにKali Linuxをセットアップしてみた"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows", "WSL2", "kalilinux"]
published: false
---

# 概要
WSLで動かせるLinux OSはMicrosoft Storeからインストールできますが、Kali LinuxもStoreからインストールしてWSLで操作できます。
![](/images/wsl-kali-linux/image1.png)

Kali Linuxは脆弱性診断に特化したDebian系Linuxディストリビューションで、最初からハッキング系のツールがプリセットされています。
昔はVirtualBox上で動かしていましたが、動かし続けていくうちにメモリ消費が大きくなり、動作がもっさりしました。USBにOSを入れてLiveUSBで遊んだこともありますが、USBからの読み込みですとかなり処理が遅く、APTパッケージのアップデートに何時間もかかることもありました。
WSLがLinuxのGUIアプリもサポートされるようになり、Kali LinuxのデスクトップモードがWindowsで使えるようになりましたので、WSL上でのKali Linuxのセットアップについて紹介します。

# Kali Linuxについて
WSLで動かす前に簡単にKali Linuxについて説明します。
https://www.kali.org/
冒頭で述べたとおりKali Linuxは脆弱性診断に特化したLinuxディストリビューションで[Burp Suite](https://portswigger.net/)や[Wireshrak](https://www.wireshark.org/)など数多くの診断ツールが最初から入っています。(その数なんと600以上！)
https://www.kali.org/tools/

私がその存在を知ったのは4,5年くらい前のセキュリティセミナーでしたが、当時は日本語の情報が少なく公式のドキュメントもあまり豊富とは言えませんでしたが、だいぶ日本語情報が揃いはじめて、公式ドキュメントもかなり増えてきて導入の敷居も下がっていると思います。

情報セキュリティのニーズがますます高まる昨今、攻撃者側がどのように脆弱性攻撃を仕掛けているのか理解するために自ら脆弱性攻撃を体験してみてセキュアなシステム開発について理解を深めてみてください。

# Kali Linuxの導入
https://www.kali.org/get-kali/
WSLにKali Linuxを導入してみます。