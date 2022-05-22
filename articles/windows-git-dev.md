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