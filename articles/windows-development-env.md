---
title: "WindowsPCでWeb開発におススメなツールを紹介してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows","scoop","WSL2","powershell"]
published: true
---
# 概要
プログラミングの勉強を始める際、何となくMacじゃないといけないと考えている人は一定数いるのかなと思います。
プログラミングスクールでもPCをMac指定しているところもありますし、Windowsでは開発できないと思っている人もいそうです。
ただWindowsも近年色々といい感じな開発ツールが出始めてきていますので、Macを買い替えなくてもお手元のWindowsPCでも導入できる開発支援ツールを入れてみてプログラミング勉強してみてもいいかなと考え、私が使っているツール群について紹介します。

:::message alert
当初開発環境構築までを記事にしようとしましたが、私のスキル・知識では正確に説明できないので、ツールの紹介に留めます。
:::

# PowerShell
https://docs.microsoft.com/ja-jp/powershell/
WindowsにはデフォルトでWindows PowerShellがインストールされています。
PowerShellはスクリプト言語でもあり、コマンドラインツールでもある密結合な設定管理フレームワークです。
ですが、デフォルトでインストールされているWindows PowerShellは5系と古くWindowsOSでしか利用できないものとなっています。
現在のPowerShellはPowerShell Coreと呼ばれWindows以外のOSでも利用できる7系ですが、手動でインストールする必要があります。
基本的に新規開発された機能などはPowerShell Coreに実装されますのでまずはPowerShell Coreをインストールします。

Windows PowerShellを起動すると冒頭で新しいPowerShellをお試しくださいというメッセージが表示されますので案内先のリンクをクリックします。
ただ、正直リンク先のMicrosoftドキュメントだとどこからインストールページに行けばいいのかわかりにくかったので、PowerShellのGitHubページを記載しておきます。
こちらのページにOSごとにダウンロードリンクが用意されていますので、Windowsのインストーラーをクリックします。
https://github.com/PowerShell/PowerShell
ダウンロード後、インストーラーを起動しましたら標準設定のまま次へをクリックしインストールを完了させます。
PowerShell Coreを起動しますと黒い画面が表示され、以下にもUNIXライクな見た目のコマンドラインが表示されます。
![](/images/windows-development-env/image2.png)
私自身ガッツリPowerShellを触ってきたわけではないため、PowerShell Coreにどういった新機能があるのか上手く説明できる自信がありませんが、一例として[Pipeline Chain Operators](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pipeline_chain_operators?view=powershell-7.1)を挙げます。
Linuxでは`&&`や`||`で複数コマンドの実行結果を可能としておりましたが、Windows PowerShellでは今までこの機能は実装されていませんでした。

```powershell:Windows PowerShell
C:\Users\yuta_>  Write-Output "Hello" && Write-Output "World"
発生場所 行:1 文字:23
+  Write-Output "Hello" && Write-Output "World"
+                       ~~
トークン '&&' は、このバージョンでは有効なステートメント区切りではありません。
    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : InvalidEndOfLine
```

PowerShell Coreでは実装され複数コマンドの実行が可能となっています。

```powershell:PowerShell Core
PS C:\Users\yuta_> Write-Output "Hello" && Write-Output "World"
Hello
World
```
# WSL2
https://docs.microsoft.com/ja-jp/windows/wsl/
従来WindowsでLinux環境を試す場合、VirtualBoxやVMwareなどの仮想マシンとISOイメージをインストールして起動させる必要がありました。
個人的に仮想マシンは起動に時間がかかりあまり気軽に使えなくて微妙な気持ちでした。
WSLは起動が早く、Windowsホストにあるファイルも共有しやすいので一番気に入っているツール(system?)です。
バージョン1と2があり基本的にバージョン2のほうが機能に優位性があります。
https://docs.microsoft.com/ja-jp/windows/wsl/compare-versions
詳細なインストール手順は公式ドキュメントの以下が参考になります。
https://docs.microsoft.com/ja-jp/windows/wsl/install-win10

### 手順だけ見たい人向け

#### 2021/12/31追記
いつのまにかWSLの導入が簡単になっていました。
上のインストール手順を見ればわかりますが、下の手順を実施しなくてもワンライナーで一発で実行できます。
`wsl --install`
このコマンドを実行すれば自動でUbuntuディストリビューションの最新版インストールまで完了します。

インストール後起動しますと、新規ユーザー登録とパスワード登録を要求されますので設定します。
初期設定が完了しますとWSLが使えるようになります。
![](/images/windows-development-env/image4.png)
余談ですが、HomebrewはLinuxでも使うことができます。
インストール方法は簡単で以下のワンライナーを実行するだけでOKです。
(少し時間はかかります)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

# Docker Desktop for Windows
https://docs.docker.com/desktop/
WSL2を導入しましたらDockerをインストールしましょう。
WindowsでDockerを動かす場合WSL2+Docker Desktopの組み合わせが一番簡単に構築しやすいのでWSL2を導入してからインストールすることをおススメします。
インストール方法はこちらの公式に沿って進めば簡単です。
https://docs.docker.com/desktop/windows/install/
インストール完了後、デスクトップにアイコンが生成されますのでクリックします。
![](/images/windows-development-env/image9.png)

DockerにはDocker HubというDockerイメージを格納するGit Hubのようなものがありますので、アカウントを登録してDocker Desktopと連携すれば便利です。
https://hub.docker.com/

# Git for Windows
https://gitforwindows.org/
MacですとGitは標準に入っているかもしれませんが、Windowsには入っていません。
とはいえWindowsでのGitのインストール方法は難しくないので↑の公式ページからインストーラーをダウンロードしてインストールするだけでOKです。
WindowsのGitには`Git Bash`という便利なものがあります。
![](/images/windows-development-env/image6.png)
Windowsでもbashコマンドを使えるように提供されたアプリケーションです。
デフォルト設定のままGit for Windowsをインストールしていけば最初から入っています。
※一応GUIもあるのですが、私は使い方わかりません。。。
GUIのおススメの機能ご存じでしたらアドバイスください。

# Windows Terminal
https://github.com/microsoft/terminal
ここまでPowerShellやWSL、`Git Bash`など色々なターミナルソフトをインストールしました。
ただ複数のターミナルソフトを起動すると管理が大変です。
ここでWindowsならではのターミナルソフトとしてWindows Terminalを活用します。
Windows Terminalは上記のターミナル、コマンドラインツールを統合的に管理できるターミナルソフトです。
2020年にリリースされたことでだいぶWindowsでのターミナル操作も楽になったと思います。
インストール方法は簡単で[Microsoft Store](https://www.microsoft.com/ja-jp/p/windows-terminal/9n0dx20hk701?activetab=pivot:overviewtab)からインストールするだけでOKです。
最初に起動するとWindows PowerShell(古いほう)が表示されます。
![](/images/windows-development-env/image7.png)
タブの▽ボタンから設定を選べばカスタマイズできます。
ここでターミナルの見た目を変えたり、デフォルトの起動をWSLに変更するなど色々できます。
私はターミナルごとに背景色を変えたり、最初にPowerShell Coreにするなど自分好みに変えました。
![](/images/windows-development-env/image8.png)
ちなみにMicrosoftの公式ドキュメントを参照するとおすすめのターミナル拡張ツールが紹介されていますのでそちらもおすすめです。
https://docs.microsoft.com/ja-jp/windows/terminal/tutorials/custom-prompt-setup

私は`posh-git`でGitコマンドのタブ補完と`Oh My Posh`で多少おしゃれなターミナルにカスタマイズしています。

# Scoop
https://scoop.sh/
Windowsでソフトウェアをインストールする際はGUIでブラウザ画面からダブルクリックしてダウンロードしてインストールすることが一般的です。
ですが、それですと自動化がしにくくて面倒かつ手順も残しにくいです。
WindowsにもDebian系のAPTやRHELのYUM(DNF)のようなパッケージ管理ツールがありまして、それがScoopとよばれるものです。
Scoopのインストール方法はPowerShellから以下のコマンドを叩くだけです。

```powershell:powershell
PS: Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
PS: iwr -useb get.scoop.sh | iex
```

インストールできましたら試しにいくつかコマンドを実行してみます。

##### パッケージ検索
```powershell:powershell
PS C:\Users\yuta_> scoop search touch

'main' bucket:
    busybox (4264-gc79f13025) --> includes 'touch'
    coreutils (5.97.3) --> includes 'touch.exe'
    gow (0.8.0) --> includes 'touch.exe'
    psutils (0.2020.02.27) --> includes 'touch.ps1'
    touch (0.2018.07.25)
    unxutils (2007.03.01) --> includes 'touch.exe'
```

#### パッケージインストール
```powershell:powershell
PS C:\Users\yuta_> scoop install touch
Installing 'touch' (0.2018.07.25) [64bit]
Loading touch.ps1 from cache
Checking hash of touch.ps1 ... ok.
Linking ~\scoop\apps\touch\current => ~\scoop\apps\touch\0.2018.07.25
Creating shim for 'touch'.
'touch' (0.2018.07.25) was installed successfully!
```

#### インストール済みパッケージ一覧
```powershell:powershell
PS C:\Users\yuta_> scoop list
Installed apps:

  gh 2.4.0 [main]
  ghq 1.2.1 [main]
  go 1.17.5 [main]
  peco 0.5.10 [main]
  terraform 1.1.2 [main]
  which 2.20 [main]
  ```

Scoopはbucketという単位でアプリを管理しており、`main`bucket以外のbucketを追加すればより多くのアプリをインストールできます。
https://github.com/lukesampson/scoop#known-application-buckets
#### リポジトリの追加
```powershell:powershell
scoop bucket add extras

PS C:\Users\yuta_> scoop search firefox
# firefoxはextras bucketからインストールできます。
'extras' bucket:
    firefox-beta (93.0b6)
    firefox-developer (93.0b6)
    firefox-esr-portable (78.14.0)
    firefox-esr (78.14.0)
    firefox-nightly (94.0a1.20210919212908)
    firefox-portable (92.0)
    firefox (92.0)
    tor-browser (10.5.6) --> includes 'firefox.exe'
```

# 所感
WindowsPCでWeb開発するときにおススメのツール群について紹介しました。
私はもともとプライベートPCはLinuxを入れて使っていましたが、新しくWindowsPCを購入し、Windowsでもできることが色々と広がっていたので最近はWindowsPCでもくもく会などの勉強会に参加しています。
まずはWindowsで環境構築を経験するのも楽しいと思いますので、皆さんもぜひ試してみてください。

# 参考文献
https://dev.classmethod.jp/articles/powershell-7-generally-available/
https://dev.classmethod.jp/articles/powershell-7-pipeline-chain-operator/
https://jsnotice.com/posts/2021-05-31/
https://brew.sh/index_ja
https://www.atlassian.com/ja/git/tutorials/git-bash
https://docs.microsoft.com/ja-jp/windows/terminal/
https://qiita.com/Dooteeen/items/12dc8fb14042888113d0
https://nodachisoft.com/common/jp/article/jp000009/
