---
title: "私のPowerShellプロファイルを紹介する"
emoji: "🐚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["powershell","windows"]
published: false
---

# 序論
過去何回かWindowsでWeb開発する場合のおすすめ初期セットアップについて紹介しました。
https://zenn.dev/yuta28/articles/windows-git-dev

https://zenn.dev/yuta28/articles/windows-development-env

今回は私のWindowsPCに設定しているPowerShellプロファイルの設定について紹介していきます。

:::message
一部以前の記事と重複した内容になります。
:::

# 対象読者
- Windowsを普段使う
- Windows Terminalを普段使う

# プロファイルについて
Macを使っている人ならシェルをbash[^1]かzsh[^2]にしているかと思います。ホームディレクトリに`~/.bashrc`か`~/.zshrc`を作成し、シェルを起動時に独自変数の読み込みやコマンドラインタブ補完コマンドを有効化して開発生産性を高める人も多いでしょう。
ターミナル起動時にシェルを実行し、シェル環境をカスタマイズできるファイルがプロファイルです。

Windowsは標準でPowerShell[^3]を使っており、PowerShellのプロファイルは`$PROFILE`変数で簡単に呼び出せます。

```powershell
ls $PROFILE

Directory: C:\Users\yuta_\OneDrive\Documents\PowerShell


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a---        2023/08/30      8:09           2905   Microsoft.PowerShell_profile.ps1
```

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_profiles

Windowsターミナル[^4]でPowerShellを起動すると最初にPowerShellプロファイルのスクリプトが起動し、Gitプロンプトの見た目を編集、モジュールのセットアップが行われターミナル操作が楽になります。

[^1]: https://www.gnu.org/software/bash/
[^2]: https://www.zsh.org/
[^3]: https://learn.microsoft.com/ja-jp/powershell/scripting/overview
[^4]: https://learn.microsoft.com/ja-jp/windows/terminal/


# My PowerShellプロファイル

本題となる私のPowerShellのプロファイルの中身です。
順番に説明していきますが冗長になりますので下記の目次から気になる部分だけチェックしてみてください。

- posh-git
- タブ補完
- リポジトリ管理エイリアスコマンド

@[gist](https://gist.github.com/Yuhta28/6b4455937b0098e6b573bcf9a506798c)

## posh-git

https://github.com/dahlbyk/posh-git

posh-gitは[以前の記事](https://zenn.dev/yuta28/articles/windows-git-dev#posh-git)でも紹介しましたが、GitとPowerShellを統合したPowerShellモジュールです。Gitのステータス情報をターミナル上に表示してくれるのでブランチ間違いや未pushに気が付きやすくなります。
パッケージマネージャーからposh-gitをインストールすれば自動でプロファイルにインポートモジュールコマンドが追加されるのでユーザーによる操作は不要です。

```powershell:scoopによるインストール手順
scoop bucket add extras
scoop install posh-git
```

PowerShellのデフォルトのコマンドタブ補完は少々使いにくいので`Set-PSReadLineKeyHandler`モジュール[^5]でタブ補完モードを変更します。
私はzshのタブ補完が気に入っているのでzshライクなタブ補完にしました。
![](/images/powershell-profile/image1.gif)

[^5]: https://learn.microsoft.com/ja-jp/powershell/module/psreadline/set-psreadlinekeyhandler?view=powershell-7.4

## タブ補完

7行目から55行目まではツール特化のタブ補完になります。基本的に公式サイトドキュメント[^6][^7][^8]に記載されている手順をそのままコピーしました。ただいくつかのツールは残念ながら公式ではbashかzshしか対応しておりません。[^9]

Terraformのタブ補完はクラスメソッドさんの記事にPowerShell向けのTerraformタブ補完を有効化してくれる手順が紹介されてましたのでこちらを採用しました。
https://dev.classmethod.jp/articles/how-to-set-terraform-autocomplete-on-powershell/

DockerコマンドのPowerShellタブ補完は以下のリポジトリを参考に設定しました。
https://github.com/matt9ucci/DockerCompletion?tab=readme-ov-file#installation

Scoopのタブ補完についても[以前私の記事](https://zenn.dev/yuta28/articles/windows-git-dev#scoop%E3%81%AE%E3%82%BF%E3%83%96%E8%A3%9C%E5%AE%8C)で紹介しましたので、こちらをご覧ください。

[^6]: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-completion.html#cli-command-completion-windows
[^7]: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli#enable-tab-completion-in-powershell
[^8]: https://cli.github.com/manual/gh_completion
[^9]: https://developer.hashicorp.com/terraform/cli/commands#shell-tab-completion

## リポジトリ管理エイリアスコマンド
GitHubからリポジトリをcloneする際あちらこちらにcloneしてしまうとローカルリポジトリがとっ散らかってしまいどこにあるのか探すのに苦労します。
私は[ghq](https://github.com/x-motemen/ghq)を使ってローカルのリポジトリをcloneし管理しています。ghqならどこのパスからcloneしても特定フォルダ内にリポジトリをcloneしてくれます。
さらに[peco](https://github.com/peco/peco)と組み合わせて使うことでいつでもローカルリポジトリに移動できるエイリアスコマンドを作りました。

```powershell
# Alias
function ghcd() {
    Set-Location "$(ghq root)/$(ghq list | peco)"
}
```

# 参考文献
https://qiita.com/FKbelm/items/2edb23d4f57e8c0d4fb4