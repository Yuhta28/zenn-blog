---
title: "GitHub Actionsでzennブログの校正を自動化してみた"
emoji: "🐀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [GitHubActions,reviewdog,Zenn,CI]
published: false
---
# 概要
前回の記事でも紹介したとおり、ZennはGitHubと連携することでリポジトリ管理することができ、自分の好きなエディタで記事を書くことができます。
https://zenn.dev/yuta28/articles/first-article-by-cli-yuta
他にもGitHubを連携することでCIツールを導入することができ、記事作成の助けになります。
そこで今回は、GitHub Actionsとtextlintとreviewdogを使うことで、PRする度に自動ドキュメント校正を行ってくれるCI環境の構築を行ってみました。