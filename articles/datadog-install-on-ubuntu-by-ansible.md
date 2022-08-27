---
title: "Ansible経由でUbuntuにDatadogをインストールするときの注意点"
emoji: "🐶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["datadog","ubuntu","ansible"]
published: false
---

# 概要
会社で新しくDatadogを導入しようとしたときにちょっとしたトラップに出くわしましたので詳細と回避策について紹介します。

# Datadog
https://www.datadoghq.com/ja/
詳しい説明はドキュメントなり他の人が書いた記事などがありますので、割愛しますが監視SaaSとして人気のあるサービスです。
単なるモニタリング以外にもログ分析やトレーシングなどがありまして弊社でもメトリクス監視以外の機能も活用しようとして導入を進めている最中です。
いわゆるエージェント方式で監視対象ホストにエージェントを導入することで、Datadogの管理画面にメトリクスやログ、プロセス情報を収集できます。公式でインストール方法が細かく紹介されていまして、直接ホストにインストールする方法以外にもAnsible経由やDockerでのインストール方法も紹介されています。
https://docs.datadoghq.com/ja/agent/

# インストール
弊社では複数のメディアサーバーに共通のミドルウェアを導入するときはAnsibleの入ったDockerコンテナからインストールするようにしています。
Datadog用のロールを用意してplaybookで追加するだけで簡単にインストールできると思い、Ansibleからインストールする方法を選択しました。
![](/images/datadog-install-on-ubuntu-by-ansible/image1.png)