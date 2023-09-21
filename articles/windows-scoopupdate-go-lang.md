---
title: "Scoopインストーラーのアップデートからキャッシュ削除までまとめて実行プログラム"
emoji: "🥄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows","scoop","go","chatgpt"]
published: false
---

# 序論

私は[以前の記事](https://zenn.dev/yuta28/articles/windows-development-env)でも書いたとおりWindowsでツールをインストールする場合、Scoopを利用したコマンドラインベースのインストールを利用しています。

https://scoop.sh/

コマンドラインベースでツールをインストールするメリットの一つにアップデートが簡単にできることが挙げられます。GUIでツール提供元からインストールするといつバージョン情報の更新があったのか、自分からサイトにアクセスしないとわかりませんが、Scoop経由でインストールしたらコマンドを叩くだけで最新バージョンがリリースされているか一目でわかります。
