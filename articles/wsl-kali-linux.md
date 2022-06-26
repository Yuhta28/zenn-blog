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
昔はVirtual Box上で動かしていましたが、動かし続けていくうちにメモリ消費が大きくなり、動作がもっさりしました。USBにOSを入れてLiveUSBで遊んだこともありますが、USBからの読み込みですとかなり処理が遅く、APTパッケージのアップデートに何時間もかかることもありました。
WSLがLinuxのGUIアプリもサポートされるようになり、Kali LinuxのデスクトップモードがWindowsで使えるようになりましたので、WSL上でのKali Linuxのセットアップについて紹介します。
