---
title: "JAWS CTFに参加してきました"
emoji: "🦈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","ctf","セキュリティ"]
published: false
---

# 概要
2023/8/27(日)にJAWS Security支部主催のCapture The Flag(CTF)に参加してきました。
https://s-jaws.doorkeeper.jp/events/155025

CTFは初めてでしたが、結果として参加者約100人中15位と思ったより頑張った結果だったと思います。
https://twitter.com/Y0u281/status/1695736620018974935?s=20

すでに問題環境は削除されており、問題の解説や感想について書いても問題ないということでしたので参加した感想について述べていこうと思います。

# 対象読者
- CTFについて知りたい人

# CTFについて
CTFとはエンジニア向けの一種の競技イベントでありエンジニアはチームないし個人で参加し、クイズ形式で制限時間内に多くの問題を解いてスコアを競うゲームとなります。
日本ではSECCONと呼ばれるSecurity Contestが有名で毎年開催されています。
https://www.seccon.jp/2023/

ネットワークやプログラミング、暗号技術など幅広い分野の知識が求められますが初心者でも解ける問題もあり、初心者から上級者までさまざまなエンジニアが楽しむことができます。

今回のJAWS Security支部主催のCTFはAWSに特化したCTFとなっておりAWSインフラ上で構成されたサービスの脆弱性をつくものから権限が限定されたIAMユーザを使って旗(スコア)を手に入れるものでCTF初心者でも楽しめる内容となっていました。

すでにイベント用環境は破棄されているのでもう一度解くことはできませんが、イベント当日のアーカイブ動画は配信され、問題を作成された作問者たちの解説記事も公開されていますので当日の雰囲気やどういった問題が出題された気になる人は覗いてみてください。
- アーカイブ動画

https://youtu.be/y7RnYJq02iM

- 作問者解説記事

https://ken5scal.notion.site/Security-JAWS-Days-CTF-Write-up-9da13cc1f7534bd0a4add75ec3f16f22
https://speakerdeck.com/tigerszk/security-jaws-days-ctf-zuo-wen-zhe-jie-shuo
https://scgajge12.hatenablog.com/entry/security_jaws_days_2023

私が解けた問題のいくつかについて感想を述べていきます。

# Show IAM policy


# 参考文献
https://cybersecurity-jp.com/column/33780
