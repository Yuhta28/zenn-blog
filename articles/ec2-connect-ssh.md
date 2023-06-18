---
title: "ターミナルからEC2 Instance Connectを使ってSSH接続してみる"
emoji: "🔓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","ec2","ssh"]
published: true
---

# 概要
2023/6/13-14(現地時間)にアメリカアナハイムで開催された[AWS re:Inforce](https://reinforce.awsevents.com/)にてEC2 Instance Connectに新たな機能が発表されました。従来EC2 Instance Connectで接続できるEC2はパブリックIPアドレスがあるものに限られていました。この度発表されたEC2 Instance Connect EndpointはパブリックIPアドレスを持たないEC2に対してもSSH/RDP接続ができるようになりました。
またEC2 Instance Connectは今まではブラウザからでしか接続できませんでしたが、AWS CLI経由でターミナルからも接続できるようになりました。
すでにEC2 Instance Connect Endpointについて紹介している記事がいくつか存在していますがターミナルから接続できることについて述べた記事はまだ少なかったのでAWS CLIを使用してターミナルからプライベートサブネットのEC2にSSH接続した手順を紹介いたします。


# 対象読者
- EC2 Instance Connect Endpointについて知りたい人
- SSH秘密鍵の管理が面倒と感じている人
- ターミナルからプライベートサブネットに配置されているEC2にSSH接続したい人

# 目次
- [EC2 Instance Connectについて](#ec2-instance-connectについて)
- [AWS CLI経由でSSH接続](#aws-cli経由でssh接続)
- [所感](#所感)
- [参考文献](#参考文献)

# EC2 Instance Connectについて
あらためてEC2 Instance Connectについて説明しますとコンソール画面からSSH秘密鍵が不要でIAMベースによる認証でEC2にSSHログインできる接続方式です。ローカルマシンに秘密鍵ファイルを保管する必要がないため新規ユーザーに秘密鍵を渡すことなくEC2に公開鍵認証ができるのは便利ですが、パブリックサブネットに配置されているEC2のみにしか対応していませんでした。
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/Connect-using-EC2-Instance-Connect.html

もしプライベートサブネットに配置されているEC2に秘密鍵なしでSSH接続する場合はSystems ManagerのSession Managerによる接続で実現できます[^1]が、インターフェース型のVPCエンドポイントを3つ作成しなければならず、VPCエンドポイント料金が約$30/月ほど発生しました。[^2]
検証環境などあまりコストをかけたくない環境ではSession Managerを使わず踏み台サーバー経由でプライベートサブネットに配置されているEC2に接続するという運用をとる人も多かったと思います。

そこで今回発表されたEC2 Instance Connect Endpointを使うことでプライベートサブネットに配置されているEC2に対してSSH接続が実現できました。VPCエンドポイントに新しくEC2 Instance Connect用のエンドポイントが用意されユーザーはこのエンドポイント経由でプライベートサブネットに配置されているEC2に接続できます。
![](/images/ec2-connect-ssh/image1.png)
*AWSブログより引用[^3]*

このエンドポイントはインターフェース型のVPCエンドポイントとは異なり無料で利用できます。
> There is no additional cost for using EIC endpoints. Standard data transfer charges apply.

https://aws.amazon.com/jp/about-aws/whats-new/2023/06/amazon-ec2-instance-connect-ssh-rdp-public-ip-address/

[^1]: https://dev.classmethod.jp/articles/tsnote-private-ec2-ssm-vpc-endpoint/
[^2]: 東京リージョン$0.014/hで計算
[^3]: https://aws.amazon.com/blogs/compute/secure-connectivity-from-public-to-private-introducing-ec2-instance-connect-endpoint-june-13-2023/

# AWS CLI経由でSSH接続
今回の主題はAWS CLI経由でターミナルからプライベートサブネットに配置されているEC2に対してSSH接続することを目標としています。
コンソールからSSH接続する方法につきましてはこちらの記事をご覧ください。
https://dev.classmethod.jp/articles/ec2-instance-connect-endpoint-private-access/

事前準備としては上記記事でも書いてあるとおりEC2 Instance Connect Endpoint用のVPCエンドポイント、EC2、VPCエンドポイントに割り当てるセキュリティグループも作成します。
## 事前準備
- EC2 Instance Connect Endpoint
- SecurityGroup
    - EC2用
    - VPCエンドポイント用
- EC2
- AWS CLI


![](/images/ec2-connect-ssh/image2.png)
*エンドポイントタイプがEC2 Instance Connect*
![](/images/ec2-connect-ssh/image3.png)
*パブリックIPアドレスがアタッチされていない*

```json:起動時にキーペアを割り当てていない
[
    {
        "InstanceId": "i-0a0ea20774c13e3ea",
        "KeyName": null,
        "PrivateIpAddress": "10.0.12.187",
        "PublicIpAddress": null
    }
]
```

準備できましたらローカルマシンからこのEC2に対してSSH接続してみます。

## 実行コマンド
新しくサポートされた[AWS CLIコマンドのドキュメント](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2-instance-connect/ssh.html)を確認すると以下のコマンドを実行することでローカルマシンからSSH接続できます。

```teminal
$  aws ec2-instance-connect ssh --instance-id <instance-id>
```
## 実際のコマンド結果
![](/images/ec2-connect-ssh/image4.png)

SSH公開鍵がないプライベートサブネットに配置されているEC2に対してターミナルからSSH接続できました。

# 所感
AWS CLI経由でターミナルからプライベートサブネットに配置されているEC2へSSH接続しました。
このアップデートによりSession Managerよりも簡単にプライベートサブネットのEC2にSSH接続ができるので嬉しい限りです。
また個人的に良かった点ですとコンソールから接続した場合クリップボードなどにコピーしたコマンドを`Ctrl+V`でペーストできず、プレーンテキストとして貼り付けないとペーストできなかったた地味にストレスでしたが、コンソールからなら`Ctrl+V`キーでコマンドを貼り付けができるのが嬉しかったです。
![](/images/ec2-connect-ssh/image5.png)
*この表記に共感してくれるはず*
簡単に試せますのでぜひターミナルからEC2 Instance Connect Endpointを試してみてください。

# 参考文献
https://dev.classmethod.jp/articles/demystifying-ec2-instance-connect-implementation/
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html#SSH-using-EC2-Instance-Connect
https://aws.amazon.com/blogs/compute/secure-connectivity-from-public-to-private-introducing-ec2-instance-connect-endpoint-june-13-2023/
https://github.com/aws/aws-cli/pull/7971