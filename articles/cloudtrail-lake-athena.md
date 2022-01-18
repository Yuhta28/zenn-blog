---
title: "CloudTrail LakeとAthenaでのログ分析を比べてみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws"]
published: false
---

# 概要
先日のアップデートで新しくCloudTrail Lakeという機能がリリースされました。
https://aws.amazon.com/jp/about-aws/whats-new/2022/01/aws-cloudtrail-lake-audit-security/

CloudTrailはAWSのオペレーションログを記録してくれるサービスです。
私はよく使っていまして、例えば誰が立ち上げたのかわからないEC2インスタンスの起動ユーザーを確認して削除してよいかどうか調査したり、夜間に検証環境のEC2インスタンスが自動で停止されているのか確認したいときなどに使います。

ただCloudTrailで検索できるログ内容は過去90日以内のものしかなく、クエリ検索もできないので複雑な条件でログ検索をしたい場合はS3にログを転送して、Athenaでログを分析する必要がありました。

# 参考文献
https://dev.classmethod.jp/articles/aws-cloudtrail-lake-audit-security/