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

今回のJAWS Security支部主催のCTFはAWSに特化したCTFとなっておりAWSインフラ上で構成されたサービスの脆弱性をつくものから権限が限定されたIAMユーザーを使って旗(スコア)を手に入れるものでCTF初心者でも楽しめる内容となっていました。

すでにイベント用環境は破棄されているのでもう一度解くことはできませんが、イベント当日のアーカイブ動画は配信され、問題を作成された作問者たちの解説記事も公開されていますので当日の雰囲気やどういった問題が出題された気になる人は覗いてみてください。
- アーカイブ動画

https://youtu.be/y7RnYJq02iM

- 作問者による解説記事

https://ken5scal.notion.site/Security-JAWS-Days-CTF-Write-up-9da13cc1f7534bd0a4add75ec3f16f22
https://speakerdeck.com/tigerszk/security-jaws-days-ctf-zuo-wen-zhe-jie-shuo
https://scgajge12.hatenablog.com/entry/security_jaws_days_2023

私が解けた問題のいくつかについて感想を述べていきます。

::: message
ターミナルコマンドは私の環境で再現したものになります。
:::

# Show IAM policy
あるIAMユーザーに含まれているIAMポリシーについて調べる問題です。アクセスキーとシークレットキー情報だけ貰えた状態で解法を求められたのでAWS CLIコマンドを使ってアタッチされているIAMポリシーについて調べていきます。
IAMユーザーにアタッチされているポリシー情報を調べる場合は`aws iam list-attached-user-policies`コマンドを使えばいいと考えました。ですがアタッチされているポリシーが出てきません。

```terminal
$ aws iam list-attached-user-policies --user-name CTF-user
{
    "AttachedPolicies": []
}
```

このことからIAMユーザーには直接ポリシーがアタッチされておらず、IAMユーザーグループに属しているのではないかと考え、所属グループについて調べてみました。

```
# 所属IAMユーザーグループを確認
$ aws iam list-groups-for-user --user-name CTF-user
{
    "Groups": [
        {
            "Path": "/",
            "GroupName": "CTF-Group",
            "GroupId": "AGPASG4NN6LFS67TPLCJT",
            "Arn": "arn:aws:iam::XXXXXXXXX:group/CTF-group",
            "CreateDate": "2023-09-03T02:30:39+00:00"
        }
    ]
}

# IAMユーザーグループにアタッチされているポリシーを確認
$ aws iam list-group-policies --group-name CTF-group
{
    "PolicyNames": [
        "selfcheck"
    ]
}

# アタッチされているポリシーの詳細を確認
$ aws iam get-group-policy --group-name CTF-group --policy-name selfcheck --query 'PolicyDocument'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "SGVsbG8gQ1RG",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

アタッチされているIAMポリシー`selfcheck`が怪しいと感じましたがこの後どうすればいいかしばらく悩みました。
ここで注目するポイントは`Sid`です。ここはユーザー自身が任意で名づけられるステートメント識別子であり、何も設定しなかった場合も`Statement1`みたいなものが付与されます。わざわざ法則性のない文字の羅列がついているということは何か意味があるのではと思いました。じつは、これBase64[^1]方式にエンコードされた文字列です。
この文字列を`base64 -d`コマンドでデコードすれば読める文字が出力されます。(WindowsPCだとちょっと面倒でしたので変換してくれるオンラインエディターを使いました。)
![](/images/jaws-ctf-report/image1.png)
*実際はフラッグ情報が出てきました*

Base64はAWS外の知識だったためIAMポリシーまで判明してもここからどうやってフラッグ情報を取得すればいいか悩んだ回答者多かったらしいです。

[^1]: https://e-words.jp/w/Base64.html

# Find data
S3に隠されたフラッグを見つける問題です。全部で3問ありまして私は問1と問3が解けました。

# 参考文献
https://cybersecurity-jp.com/column/33780
