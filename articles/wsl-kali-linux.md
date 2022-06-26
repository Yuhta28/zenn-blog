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

私がその存在を知ったのは4,5年くらい前のセキュリティセミナーでしたが、当時は日本語の情報が少なく公式のドキュメントもあまり豊富とは言えませんでした。
その後、日本語情報が揃いはじめて公式ドキュメントもかなり増えてきて導入の敷居も下がっていると思います。

情報セキュリティのニーズがますます高まる昨今、攻撃者がどのように脆弱性攻撃を仕掛けているのか理解するために自ら脆弱性攻撃を体験してみてセキュアなシステム開発について理解を深めてみてください。

# Kali Linux導入
https://www.kali.org/get-kali/
WSLにKali Linuxを導入してみます。上記のページから媒体に合わせた複数のインストール方法が紹介されています。昔は単にISOイメージのダウンロードページしかなかったのですが、コンテナやクラウドなど色々紹介されていびっくりしました。([Live USBの方法も公式で紹介されてる！](https://www.kali.org/get-kali/#kali-live))
公式ドキュメントではWin-KeXについて紹介していますが、Microsoft StoreからKali Linux Appをインストールしておきましょう。
https://apps.microsoft.com/store/detail/kali-linux/9PKR34TNCV07?hl=en-us&gl=US

:::message
ちなみにWSLを動かす場合、Windows Terminal上で動かすことを推奨します。同じように[Storeからインストール](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=ja-jp&gl=JP)できますので、まだの人はこちらを先にインストールしてください。
:::

Kali Linuxをインストールし、起動しますと他のLinuxディストリビューション同様にユーザー名とパスワードを聞かれますので設定します。
![](/images/wsl-kali-linux/image2.png)
最初からある程度プロンプト設定が組み込まれているみたいで、同じDebian系のUbuntuとは勝手が違いますね。
私は色々とWSLにLinuxディストリビューション入れていますので一目でKali Linuxのターミナルだと分かるようにデザインを変えてみました。
![](/images/wsl-kali-linux/image3.png)

# Win-Kex導入
Kali LinuxをWSLに導入しましたが、Kali