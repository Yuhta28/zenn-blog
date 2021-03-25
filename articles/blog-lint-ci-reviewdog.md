---
title: "GitHub Actionsでzennブログの校正を自動化してみた"
emoji: "🐀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [GitHubActions,reviewdog,Zenn,CI]
published: false
---
# 概要
前回の記事でも紹介したとおり、ZennはGitHubリポジトリと連携すると、自分の好きなエディタで記事を書くことができます。
https://zenn.dev/yuta28/articles/first-article-by-cli-yuta
ほかにもGitHubを連携することでCIツールを導入することが可能となり、記事作成の助けにもなります。
そこで今回は、GitHub Actionsとtextlintとreviewdogを使うことで、PRする度に自動ドキュメント校正してくれるCI環境の構築してみました。
# ドキュメント校正ツール
今回GitHub Actionsに組み込む校正ツールは以下の２つを使用します。

- textlint
- reviewdog

それぞれのツールについて簡単にはなりますが、紹介致します。
## textlint
`textlint`はMarkdown形式やプレーンテキストのファイルを校正してくれるJavaScriptで記述されたOSSツールです。
https://textlint.github.io/
