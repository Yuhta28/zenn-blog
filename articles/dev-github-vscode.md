---
title: "dev.toの記事もGitHubで管理してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["devto","GitHub","GitHubActions","vscode"]
published: false
---

# 概要

私はZenn以外にも[dev.to](https://dev.to/yuta28)でブログを書いています。
基本的にZennの一部の記事を英訳してdev.toに投稿しているのですが、Zennのようにdev.toの記事もGitHubで管理して、可能ならGitHub Actionsを使って自動で投稿できるようにしたいと考えていました。
調べたところ、dev.toはAPIが公開されていて、記事の公開もAPIを使えばGitHub Actionsで自動投稿できることがわかりました。
さらに調べると、dev.toでまさに私がやりたかったdev.toの記事をGitHubで管理する方法を紹介している人がいましたので、その人の記事を参考にしながらさっそく管理してみることにしました。
https://developers.forem.com/api


# 参考文献

https://dev.to/maxime1992/manage-your-dev-to-blog-posts-from-a-git-repo-and-use-continuous-deployment-to-auto-publish-update-them-143j?signin=true
https://dev.to/beeman/automate-your-dev-posts-using-github-actions-4hp3