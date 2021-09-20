---
title: "WindowsPCでWeb開発におススメなツールを紹介してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows","scoop","WSL2"]
published: false
---
# 概要
プログラミングの勉強を始める際、何となくMacじゃないといけないと考えている人は一定数いるのかなと思います。
プログラミングスクールでもPCをMac指定しているところもありますし、Windowsでは開発できないと思っている人もいそうです。
ただWindowsも近年色々といい感じな開発ツールが出始めてきていますので、Macを買い替えなくてもお手元のWindowsPCでも導入できる開発支援ツールを入れてみてプログラミング勉強してみてもいいかなと思い、私が使っているツール群について紹介します。

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
私自身ガッツリPowerShellを触ってきたわけではないため、PowerShell Coreにどういった新機能があるのか上手く説明できる自信がありませんが、代表的なものに[Pipeline Chain Operators](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pipeline_chain_operators?view=powershell-7.1)があります。
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
#### WSL導入を有効化

```powershell:PowerShell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
## 再起動する
```
#### Linuxカーネル更新プログラムパッケージをダウンロード
https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi

#### WSL2を規定のバージョンとして設定

```powershell:powershell
wsl --set-default-version 2
```
#### Microsoft StoreからLinuxディストリビューションをインストール
![](/images/windows-development-env/image3.png)

インストール後起動しますと、新規ユーザー登録とパスワード登録を要求されますので設定します。
初期設定が完了しますとWSLが使えるようになります。
![](/images/windows-development-env/image4.png)
WSLは複数のLinuxディストリビューションを使い分けることができますので、Ubuntu以外にも軽量OSとして有名なAlpineも使ってみるのも面白いと思います。
https://www.alpinelinux.org/
ちなみに余談ですが、Macのパッケージ管理システムとして有名なHomebrewですがLinuxでも使うことができます。
インストール方法は簡単で以下のワンライナーを実行するだけでOKです。
(少し時間はかかります)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```


# 参考文献
https://dev.classmethod.jp/articles/powershell-7-generally-available/
https://dev.classmethod.jp/articles/powershell-7-pipeline-chain-operator/
https://jsnotice.com/posts/2021-05-31/
https://brew.sh/index_ja