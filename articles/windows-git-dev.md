---
title: "Windowsではじめるターミナル生活"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows", "wsl", "powershell","windowsterminal"]
published: false
---

# 概要
以前WindowsでおすすめのWeb開発ツールをブログで書きました。
https://zenn.dev/yuta28/articles/windows-development-env

多少重複しますが、Windows Terminalをより便利にカスタマイズしてターミナル運用を楽しめるようにしました。

前提条件として以下のツールはインストール済とします。

- PowerShell 7系
- Git for Windows
- WSL2
- Scoop
- Windows Terminal

::: message
これらは上の記事で紹介していますのでまだの人はそちらを先にご覧ください
:::

# Windows Terminalカスタマイズ
Windows Terminalのデフォルトの背景画面は黒色なので、複数のWSLディストリビューションやPowerShellを入れると、どれがどのターミナルなのかわかりづらくなります。
設定の配色でターミナルのカラーや文字のフォントを変更できます。
![](/images/windows-git-dev/image1.png)

配色やフォントはデフォルトでも種類は多いですが、自分でカスタマイズして専用のターミナルを作るのも一興です。
参考までに私のPowerShellのターミナルの配色とフォントは以下のようにしました。

- 配色
  - One Half Dark(デフォルトで選択可)
- フォントフェイス
  - Nerd Font

Nerd Fontは開発向けのアイコンが豊富にそろっていて便利です。
https://www.nerdfonts.com/#home

こちらからダウンロードできます。
https://eng.fontke.com/font/24644369/

Windowsでフォントをインストールする方法ですが、個人用設定からフォントページに移動して、ダウンロードしたフォントファイルを指定された位置にドラッグアンドドロップします。([公式ページの方法](https://support.microsoft.com/en-us/office/add-a-font-b7c5f17c-4426-4b53-967f-455339c564c1)がわかりにくかった。。。)
![](/images/windows-git-dev/image2.png)

配色の設定はGUIでもできますし、JSONファイルに記述もできます。

```json: setting.json
{
  "background": "#282C34",
  "black": "#282C34",
  "blue": "#61AFEF",
  "brightBlack": "#5A6374",
  "brightBlue": "#61AFEF",
  "brightCyan": "#56B6C2",
  "brightGreen": "#98C379",
  "brightPurple": "#C678DD",
  "brightRed": "#E06C75",
  "brightWhite": "#DCDFE4",
  "brightYellow": "#E5C07B",
  "cursorColor": "#FFFFFF",
  "cyan": "#56B6C2",
  "foreground": "#DCDFE4",
  "green": "#98C379",
  "name": "One Half Dark",
  "purple": "#C678DD",
  "red": "#E06C75",
  "selectionBackground": "#FFFFFF",
  "white": "#DCDFE4",
  "yellow": "#E5C07B"
}
```

JSONで設定できるということはイイ感じのジェネレーターサイトないかなーと探しましたがありました。
https://windowsterminalthemes.dev/
ここでよさそうなテーマコードをコピーして`setting.json`に張り付ければあっという間にお気に入りのターミナルが作れます。
![](/images/windows-git-dev/image3.png)

# Gitカスタマイズ
ターミナルの外見の次はターミナルを使った開発作業を便利するツール群をインストールします。まずはGitです。
皆さんもGitでターミナル操作をする際、プロンプトの表示を見やすくしたり、タブの補完機能を有効活用されていると思います。Macでしたら、`.bashrc`や`.zshrc`ファイルに記述して実現されているでしょう。
PowerShellでもGit運用を便利にしてくれるツールがいくつかありますので導入していきましょう。

## oh-my-posh
https://ohmyposh.dev/  
oh-my-poshはbashやzsh、PowerShellのプロンプトをイケてる感じに見た目のカスタマイズをしてくれる素晴らしいツールです。

```powershell
scoop install oh-my-posh
```

 #### jandedobbeleer(私のお気に入り)
![](/images/windows-git-dev/image4.png)


インストールできましたらターミナル起動時にoh-my-poshのテーマを読み込ませる`oh-my-posh init pwsh --config <テーマファイル>`コマンドを`$PROFILE`に記述します。

```powershell: $PROFILE
oh-my-posh init pwsh --config ~/AppData/Local/Programs/oh-my-posh/themes/jandedobbeleer.omp.json | Invoke-Expression
```

テーマの種類は多くありますので、お気に入りのものを選択してください。
https://ohmyposh.dev/docs/themes

私は冒頭の`jandedobbeleer`というものを使っていますが、Macユーザーでしたら`Iterm2`が使いやすいでしょうか。
![](/images/windows-git-dev/image5.png)
*アイコンが半分見切れてる…*

## posh-git
https://github.com/dahlbyk/posh-git#overview  
posh-gitはGitとPowerShellを統合したPowerShellのモジュールです。Gitディレクトリ上に現在のブランチ名やステータス情報を表示してくれて、Gitコマンドのタブ補完機能も提供してくれます。

:::message
ブランチ名やステータス情報はoh-my-poshのテーマでも確認できます。
:::

```powershell
scoop install posh-git
```

こちらはインストールしたら`$PROFILE`にモジュール追加するだけ適用されます。

```powershell: $PROFILE
Import-Module posh-git
```

ただPowerShellのタブ補完はデフォルトでは、zshみたいに検索候補が一覧で表示されないので若干癖があります。
タブ補完時に検索候補を複数表示したいと思いましたので、以下のPowerShellコマンドを`$PROFILE`に追加します。

```powershell: $PROFILE
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
```

![](/images/windows-git-dev/image6.gif)

# Scoopのタブ補完
Scoopでアプリのインストールやアップデートをするときに何度も`scoop install ~`や`scoop update`を実行すると思いますが、デフォルトではScoopのコマンドのタブ補完がないため何度も最後までコマンドをうたなければなりません。
Scoopのタブ補完機能を追加してくれるアプリがありましたので、追加して有効化します。  
https://github.com/Moeologist/scoop-completion

```powershell
scoop install scoop-completion

# $PROFILEに追加
Import-Module "$($(Get-Item $(Get-Command scoop.ps1).Path).Directory.Parent.FullName)\modules\scoop-completion"
```

# ターミナルからアイコン表記
VS Codeからファイルのアイコンを表示してくれるプラグインを実装されている人はいらっしゃいますでしょうか。  
https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme  
VS Codeのリストからどんなファイルがあるか視覚的にわかりやすいのでとても便利なプラグインです。
![](/images/windows-git-dev/image7.png)
このアイコン表記をターミナル上からも表示できるようにしてくれるアプリがありますので気になる人はいれてみてください。  
https://github.com/devblackops/Terminal-Icons

```powershell
scoop install terminal-icons

# $PROFILEに追加
Import-Module Terminal-Icons
```

![](/images/windows-git-dev/image8.png)

# GitHub CLI
https://cli.github.com/  
ターミナル上で`git push`した後、GitHubリポジトリにマージするために、毎回ブラウザを開いてPRするのは面倒な気持ちになります。GitHubの操作もターミナルからできるようになればGitコミットからマージまでをターミナルですべて完結できるのでわずらわしさが減ります。
GitHub CLIはまさにGitHub上で今までやってきたPRやIssueの起票などをターミナル上で実現できます。

```powershell
scoop install gh
gh --version
gh version 2.11.3 (2022-05-25)
https://github.com/cli/cli/releases/tag/v2.11.3

#はじめてGitHub CLIを使う場合、ターミナルからの認証する必要があります。
gh auth login
# SSH秘密鍵もしくはトークン情報を渡してターミナルとGitHubを連携させる。

gh auth status
github.com
  ✓ Logged in to github.com as Yuhta28 (C:\Users\yuta_\AppData\Roaming\GitHub CLI\hosts.yml)
  ✓ Git operations for github.com configured to use ssh protocol.
  ✓ Token: *******************
```

GitHub CLIでマージすればローカルのリポジトリも自動的にmainブランチがpullされ、マージしたブランチを削除できますので不要なブランチが溜まるということがなくなり運用負荷の低下につながります。

```powershell: マージ実行
gh pr merge
? What merge method would you like to use? Create a merge commit
? Delete the branch locally and on GitHub? Yes #マージする際にローカルのブランチも削除するか聞かれるのでYesを選択
? What's next? Submit
✓ Merged pull request #139 (feature/add new blog windows git)
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (1/1), 648 bytes | 49.00 KiB/s, done.
From github.com:Yuhta28/zenn-blog
 * branch            main       -> FETCH_HEAD
   32be209..d7672f3  main       -> origin/main
Updating 32be209..d7672f3
Fast-forward
 .textlintrc                           |   3 +-
 articles/94744fc7a339cd.md            |   2 +-
 articles/eventbridge-slack.md         |   2 +-
 articles/first-article-by-cli-yuta.md |   2 +-
 articles/windows-git-dev.md           | 193 ++++++++++++++++++++++++++++++++++
 
✓ Deleted branch feature/add-new-blog-windows-git and switched to branch main #マージが完了するとローカルのブランチも削除されて、mainブランチも自動的にpullされる
```
# 参考文献
https://docs.microsoft.com/ja-jp/windows/terminal/tutorials/custom-prompt-setup  
https://git-scm.com/book/ja/v2/%E4%BB%98%E9%8C%B2-A%3A-%E3%81%9D%E3%81%AE%E4%BB%96%E3%81%AE%E7%92%B0%E5%A2%83%E3%81%A7%E3%81%AEGit-Powershell%E3%81%A7Git%E3%82%92%E4%BD%BF%E3%81%86  
https://qiita.com/FKbelm/items/2edb23d4f57e8c0d4fb4