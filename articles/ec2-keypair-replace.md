---
title: "EC2インスタンスのキーペア置換について調べてみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","ec2","ssh"]
published: false
---

# 概要
手持ちのAMIからEC2インスタンスを新規に起動する際にキーペアを変更したときに出くわしたEC2インスタンスの仕様について調べてみました。

# キーペアについて
EC2インスタンスを起動する最後にSSH接続するためのキーペアの選択があります。
![](/images/ec2-keypair-replace/image1.png)
EC2インスタンスは初期設定ではパスワードによるSSH接続が許可されていないため、新規作成したEC2インスタンスは基本的にキーペアを選択してEC2インスタンスを起動します。

```bash:sshd_config
# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no
PasswordAuthentication no
```
キーペアを選択してEC2インスタンスを起動しましたら、初期ユーザーで秘密鍵接続でSSHログインできるようになります。

```bash
 ssh -i ./NewWindows.pem ec2-user@18.182.24.156
Last login: Mon Aug  2 13:36:42 2021

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
```

# キーペアの置換
EC2インスタンスを起動後、AMIを作成しそのAMIからEC2インスタンスを起動するときに同じようにキーペアを選択する場面が出てきます。
![](/images/ec2-keypair-replace/image2.png)
実運用でキーペアを変更する必要が出てきた場合、新しくEC2を起動してキーペアを別のものに変更するというケースがあったとします。

この時別のキーペアを選択し、EC2インスタンスを起動すればもちろん別の秘密鍵でSSH接続できます。

```bash
 ssh -i .\tepkey.pem ec2-user@18.183.44.42
Last login: Mon Aug  2 13:45:37 2021

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
```

この時当然ながら`~/.ssh/authorized_keys`の中身は新規に選択した公開鍵だけかと考えました。

ところが調べてみるとそうではなく、過去の公開鍵も残っていました。

```bash:authorized_keys
ssh-rsa ~~~~~~~~~~~~~~~~~~~~~~~~~略~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ NewWindows
ssh-rsa ~~~~~~~~~~~~~~~~~~~~~~~~~略~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ tepkey
```

このように前の秘密鍵でもSSH接続できることを確認できました。

```bash
ssh -i .\NewWindows.pem ec2-user@18.183.44.42
Last login: Mon Aug  2 15:07:23 2021 

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
```
AWSのドキュメントを確認しましたが、キーペアを置換する場合以前の公開鍵は`~/.ssh/authorized_keys`から手動で削除する運用になっています。
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-key-pairs.html#replacing-key-pair

AMIの中に以前のキーペアの公開鍵が入っているとAWS側で置き換えてくれず、自分で削除する必要があるようです。

# 所感
EC2インスタンスのキーペア置換について調べてみました。
最初会社の人からこの話しをを聞いたときはそんなバカなと思いましたが、調べてみると本当でしたので驚きでした。
会社の人も話されていましたが、セキュリティ的にアカン気がしますが皆さん手動でEC2インスタンス内のキーペアを削除しているのでしょうか？

皆さんはどのように運用しているのか気になりますのでよろしければご意見をお聞かせください。