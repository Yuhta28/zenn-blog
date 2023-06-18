---
title: "ターミナルからEC2 instance Connectを使ってSSH接続してみる"
emoji: "🔓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","ec2","ssh"]
published: false
---

# 概要
2023/6/13-14(現地時間)にアメリカアナハイムで開催された[AWS re:Inforce](https://reinforce.awsevents.com/)にてEC2 Instance Connectに新たな機能が発表されました。従来EC2 Instance Connectで接続できるEC2はパブリックIPアドレスがあるものに限られていました。この度発表されたEC2 Instance Connect EndpointはパブリックIPアドレスを持たないEC2に対してもSSH/RDP接続ができるようになりました。
またEC2 Instance Connectは今まではブラウザからでしか接続できませんでしたが、AWS CLI経由でターミナルからも接続できるようになりました。
すでにEC2 Instance Connect Endpointについて紹介している記事がいくつか存在していますがターミナルから接続できることについて述べた記事はまだ少なかったのでAWS CLIを使用してターミナルからプライベートサブネットのEC2にSSH接続した手順を紹介いたします。


# 対象読者
- EC2 Instance Connect Endpointについて知りたい人
- SSH秘密鍵の管理が面倒と感じている人

# 目次
- EC2 Instance Connect Endpointについて
- AWS CLI経由でのSSH接続
- 所感
- [参考文献](#参考文献)





# 参考文献
https://dev.classmethod.jp/articles/demystifying-ec2-instance-connect-implementation/
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html#SSH-using-EC2-Instance-Connect
https://aws.amazon.com/blogs/compute/secure-connectivity-from-public-to-private-introducing-ec2-instance-connect-endpoint-june-13-2023/