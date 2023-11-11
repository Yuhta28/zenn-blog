---
title: "Pulumi ESCで秘匿情報をセキュアに扱う"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","security"]
published: false
---

# 序論
Pulumiから新しい機能がリリースされました。
https://www.pulumi.com/product/esc/

IaCでDBのパスワードといった認証情報をハードコーディングする方法はセキュリティ上推奨されておらず、環境変数から値を入れる。AWS Secrets Manager[^1]などを使って外部から値を取得するという運用が一般的です。

[^1]: https://aws.amazon.com/jp/secrets-manager/