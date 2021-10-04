---
title: "GitHub ActionsにAWSクレデンシャル情報を渡さずにTerraformでCI/CDをやってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","terraform","GitHubActions"]
published: false
---

# 概要
先日、非公式ながらAWSで以下のような面白い機能が発表されました。
https://twitter.com/__steele/status/1437984026145427461

よりわかりやすく嚙み砕くとこういうことです。
https://twitter.com/toricls/status/1438120050167189510

CirleCIやGitHub ActionsでAWSを使う場合は事前に環境変数にアクセスキーとシークレットキーを登録させてCIを動かしてきましたが、そのためにIAMユーザーを発行して鍵を管理するのは手間だったのでこれはいいアップデートです。

今回はTerraformとGitHub Actionsを組み合わせたCI/CDにこの機能を取り入れてGitHub ActionsにIAMロールを渡してEC2構築のCI/CDを実装してみようと思います。


# 参考文献
https://dev.classmethod.jp/articles/github-actions-without-permanent-credential/
https://zenn.dev/yutaro1985/articles/b012f69b49bec095b9f1

