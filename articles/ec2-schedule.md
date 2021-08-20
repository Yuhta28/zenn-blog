---
title: "Instance Schedulerを使って検証環境のEC2の稼働時間を管理してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","ec2"]
published: false
---

# 概要
会社で使っているEC2にはSavings Planを採用していてオンデマンドで使うよりも安い料金でEC2を利用しています。
ただそれでもEC2の数が段々と増えていて、当初の予想よりもコンピューティングリソースの消費が大きくなりEC2の利用料が大きくなってきましたので、検証環境のEC2インスタンスの稼働時間を減らすことで、コンピューティングリソースの消費を抑えようと思いました。
前職では先輩が作成したLambdaで21:00~翌9:00の時間帯は自動でシャットダウンする仕組みができており、当初はそれを真似しようと思いましたがInstance Schedulerなるものを今の会社のリーダーから教わりましたのしたので、それを使って検証環境のEC2の稼働時間を減らしてみたいと思います。

# Instance Schedulerについて
先にお話しするとInstance SchedulerというAWSリソースはありません。
複数のサービスを組み合わせて実装したAWSソリューションです。
https://aws.amazon.com/jp/solutions/implementations/instance-scheduler/

イメージとしてはEC2に専用のタグを付与して、LambdaがDynamoDBに保存されているタグごとに定義されたスケジュール情報を読み取り、DynamoDBのタグ情報と一致したタグを持つEC2に対してLambdaが停止、起動を実行するという仕組みになっています。
(上記公式ページのアーキテクチャ図参照)