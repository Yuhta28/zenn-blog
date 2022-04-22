---
title: "dev.toの記事もGitHubで管理してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["devto","GitHub","GitHubActions","vscode"]
published: false
---

# 概要

私はZenn以外にも[dev.to](https://dev.to/yuta28)でもブログを書いています。
基本的にZennの記事を英訳してdev.toに投稿しており、Zennのようにdev.toの記事もGitHubで管理して、可能ならGitHub Actionsを使って自動で投稿できるようにしたいと考えていました。
調べると、dev.toでまさに私がやりたかったdev.toの記事をGitHubで管理する方法を紹介している人がいました。

https://dev.to/maxime1992/manage-your-dev-to-blog-posts-from-a-git-repo-and-use-continuous-deployment-to-auto-publish-update-them-143j?signin=true

またdev.toは[API](https://developers.forem.com/api)が公開されていて、記事の投稿もAPIを使うことで、GitHub Actionsで自動投稿できるみたいです。
https://dev.to/beeman/automate-your-dev-posts-using-github-actions-4hp3

これらの記事を参考にしてdev.toの記事もGitHubで管理してみました。

# 実装

基本的に上の記事の手順を訳して進めますが、一部アレンジした部分もありますのでそこを詳しく説明します。
## 1. テンプレートコピー
記事作成者のGitHubリポジトリテンプレートをコピーします。
https://github.com/maxime1992/dev.to

![](/images/dev-github-vscode/image1.png)
# 参考文献

https://developers.forem.com/api
