---
title: "WindowsにKali Linuxをセットアップしてみた"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows", "WSL2", "kalilinux"]
published: false
---

# 概要
WSLで動かせるLinuxはMicrosoft Storeからインストールできますが、Kali LinuxもStoreからインストールしてWSLで操作できます。
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
Kali LinuxをWSLに導入しましたが、Kali LinuxはGUIアプリが多いのでターミナル上で動かすよりもデスクトップ環境が望ましいです。
Win-KexはWSL上のKali Linuxをデスクトップで動かすことができるツールでして、公式ドキュメントに紹介されていますのでこちらを導入します。
https://www.kali.org/docs/wsl/win-kex/

APTパッケージがありますのでAPTコマンドでインストールします。

```bash
# Win-Kexインストール
sudo apt install -y kali-win-kex

# Win-Kex起動
kex --win -s
```

起動コマンドを実行しますとKali Linuxのデスクトップが表示されます。
![](/images/wsl-kali-linux/image4.png)
Win-Kexは3種類のモードがサポートされて、上記コマンドがデスクトップセッションを別ウィンドウで実行するWindowsモード、Windows環境とKali Linux環境を分離し、armデバイスでも使えるESMモード、Windows環境上でシームレスにKali Linuxを利用できるシームレスモードがあります。

### Windowsモード
[![Windows](/images/wsl-kali-linux/image5.png)](https://www.kali.org/docs/wsl/win-kex-win/)

### ESMモード
[![ESM](/images/wsl-kali-linux/image6.png)](https://www.kali.org/docs/wsl/win-kex-esm/)

### シームレスモード
[![シームレス](/images/wsl-kali-linux/image7.png)](https://www.kali.org/docs/wsl/win-kex-sl/)

# ターミナルからの即時起動
毎回Win-Kexをターミナルからコマンドを打ち込んで起動するのは少々面倒ですので、ターミナルの設定からすぐに起動できるようにします。
一度ターミナルを閉じて再度起動後、`ctrl+,`で設定画面を開き「新しいプロファイルを追加します」を選びます。
![](/images/wsl-kali-linux/image8.png)

新規プロファイル作成画面になりますので、コマンドラインにKali Linux起動コマンドと先ほどのWin-Kexコマンドを入力します。
![](/images/wsl-kali-linux/image9.png)

保存しますとターミナルの一覧からWin-Kexの起動プロファイルが選べますので次回以降はここから選ぶだけですぐにデスクトップでのKali Linuxが使えます。
![](/images/wsl-kali-linux/image10.png)

# 所感
WSLでKali Linuxのセットアップについて紹介しました。
本格的なKali Linuxの使い方は別の記事であらためて説明しようと思います。
Kali Linuxは昔に比べて大分使うことに対する敷居が下がってきましたので、これを気に本格的に学習してみようかなと思います。

最後に諸注意ですがKali Linuxで勝手に脆弱性診断をするのは犯罪行為ですので、くれぐれも自分が管理していないサーバーに対して使うことは控えてください。