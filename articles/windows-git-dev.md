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
# 参考文献
https://docs.microsoft.com/ja-jp/windows/terminal/tutorials/custom-prompt-setup