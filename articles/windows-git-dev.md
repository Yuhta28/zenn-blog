---
title: "WindowsではじめるGit生活"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows", "git", "wsl", "powershell"]
published: false
---

# 概要
以前WindowsでおすすめのWeb開発ツールをブログで書きました。
https://zenn.dev/yuta28/articles/windows-development-env

多少重複しますが、WindowsでGitを使う場合の便利なツール群やWindows Terminalのカスタマイズに焦点をあてて紹介します。

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


# 参考文献
https://docs.microsoft.com/ja-jp/windows/terminal/tutorials/custom-prompt-setup