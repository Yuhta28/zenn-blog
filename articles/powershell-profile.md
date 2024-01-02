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


# 対象読者
- Windowsを普段使う
- Windows Terminalを普段使う

# プロファイルについて
Macを使っている人ならシェルをbash[^1]かzsh[^2]にしているかと思います。ホームディレクトリに`~/.bashrc`か`~/.zshrc`を作成し、シェルを起動時に独自変数の読み込みやコマンドラインタブ補完コマンドを有効化して開発生産性を高める人も多いでしょう。
ターミナル起動時にシェルを実行し、シェル環境をカスタマイズできるファイルがプロファイルとよばれます。

Windowsは標準でPowerShell[^3]を使っており、PowerShellのプロファイルは`$PROFILE`変数で簡単に呼び出せます。

```powershell
ls $PROFILE

Directory: C:\Users\yuta_\OneDrive\Documents\PowerShell


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a---        2023/08/30      8:09           2905   Microsoft.PowerShell_profile.ps1
```

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_profiles

Windowsターミナル[^4]でPowerShellを起動するときに最初にPowerShellプロファイルに記載されているスクリプトが起動し、Gitプロンプトの見た目を編集、モジュールのセットアップが行われターミナル操作が楽になります。

[^1]: https://www.gnu.org/software/bash/
[^2]: https://www.zsh.org/
[^3]: https://learn.microsoft.com/ja-jp/powershell/scripting/overview
[^4]: https://learn.microsoft.com/ja-jp/windows/terminal/