---
title: "インスタンスプロファイルって？"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "IAM", "EC2"]
published: false
---

# 概要
AWSリソースにIAMポリシー権限を渡す時、IAMポリシーがアタッチされているIAMロールを作成し、そのIAMロールをAWSリソースに付与することで付与されたAWSリソースは他のリソースに対して操作できます。
![](/images/ec2-iam-instance-profile/image3.png)

ですがEC2を作成する画面にIAMロールをアタッチする箇所がなく、IAMインスタンスプロファイルを設定する箇所が存在します。

![](/images/ec2-iam-instance-profile/image1.png)

![](/images/ec2-iam-instance-profile/image2.png)

AWS CLIでもパラメータに`IamInstanceProfile`という項目があり、Arnにも`instance-profile`と記載されています。

```powershell
 aws ec2 describe-instances --query "Reservations[].Instances[].IamInstanceProfile.Arn"
[
    "arn:aws:iam::XXXXXXXXXXXX:instance-profile/Yuta20210911"
]
```

このIAMインスタンスプロファイルについて紹介いたします。

# IAMインスタンスプロファイルについて
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html
AWSのドキュメントに以下記載がありました。

> Amazon EC2 は、IAM ロールのコンテナとしてインスタンスプロファイルを使用します。IAM コンソールを使用して IAM ロールを作成すると、コンソールによりインスタンスプロファイルが自動的に作成され、対応するロールと同じ名前が付けられます。

インスタンスプロファイルはIAMロールを格納するコンテナであり、そのコンテナをEC2にアタッチすると記載されています。
IAMインスタンスプロファイルはEC2のみに使われる概念であり、IAMロールを作成すると自動的に対応する同名のIAMインスタンスプロファイルが作成されます。

普段AWSを扱う中でIAMインスタンスプロファイルとIAMロールの違いについていしきすることはないと思います。気を付ける場面でいうとインフラのコード化、すなわち**Infrastructure as Code(IaC)** を実践するときだと思います。


# 参考文献
https://dev.classmethod.jp/articles/do_you_know_iaminstanceprofile/

https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html

https://docs.aws.amazon.com/ja_jp/managedservices/latest/userguide/defaults-instance-profile.html

https://medium.com/devops-dudes/the-difference-between-an-aws-role-and-an-instance-profile-ae81abd700d