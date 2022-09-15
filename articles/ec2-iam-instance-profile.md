---
title: "IAMインスタンスプロファイルって？"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "IAM", "EC2"]
published: true
---

# 概要

AWS リソースに IAM ポリシー権限を渡すときは IAM ポリシーがアタッチされている IAM ロールを作成し、その IAM ロールを AWS リソースに付与することで付与された AWS リソースは他のリソースへの操作権限が与えられます。
![](/images/ec2-iam-instance-profile/image3.png)

ですが EC2 を作成する画面に IAM ロールをアタッチする箇所がなく、IAM インスタンスプロファイルを設定する箇所が存在します。

![](/images/ec2-iam-instance-profile/image1.png)

![](/images/ec2-iam-instance-profile/image2.png)

AWS CLI でもパラメータに`IamInstanceProfile`という項目があり、Arn にも`instance-profile`と記載されています。

```powershell
 aws ec2 describe-instances --query "Reservations[].Instances[].IamInstanceProfile.Arn"
[
    "arn:aws:iam::XXXXXXXXXXXX:instance-profile/Yuta20210911"
]
```

この IAM インスタンスプロファイルというものは何なのでしょうか？

# IAM インスタンスプロファイルについて

https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html
AWS のドキュメントに以下記載がありました。

> Amazon EC2 は、IAM ロールのコンテナとしてインスタンスプロファイルを使用します。IAM コンソールを使用して IAM ロールを作成すると、コンソールによりインスタンスプロファイルが自動的に作成され、対応するロールと同じ名前が付けられます。

インスタンスプロファイルは IAM ロールを格納するコンテナであり、そのコンテナを EC2 にアタッチすると記載されています。
インスタンスプロファイルは EC2 のみに使われる概念であり、IAM ロールを作成すると自動で同名の IAM インスタンスプロファイルが作成されます。

普段 AWS を扱う中で IAM インスタンスプロファイルと IAM ロールの違いについて意識することはないと思います。気を付ける場面でいうとインフラのコード化、すなわち**Infrastructure as Code(IaC)** を実践するときだと思います。

# IaC 開発時の注意点

Lambda や CodePipeline に IAM ロールをアタッチするときは CloudFormation や Terraform で role パラメータに IAM ロールの Arn を設定します。

## Lambda を作成する IaC

```yml:CloudFormation
AWSTemplateFormatVersion: "2010-09-09"
Resources:
    LambdaFunction:
        Type: "AWS::Lambda::Function"
        Properties:
            FunctionName: "efs-s3"
            Role: !GetAtt IAMRole.Arn

    IAMRole:
        Type: "AWS::IAM::Role"
        Properties:
            Path: "/"
            RoleName: "efs-S3"
            AssumeRolePolicyDocument: "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"lambda.amazonaws.com\"},\"Action\":\"sts:AssumeRole\"}]}"
```

```hcl:Terraform
resource "aws_lambda_function" "LambdaFunction" {
    function_name = "efs-s3"
    role = "${aws_iam_role.IAMRole.arn}"
}

resource "aws_iam_role" "IAMRole" {
    path = "/"
    name = "efs-S3"
    assume_role_policy = "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"lambda.amazonaws.com\"},\"Action\":\"sts:AssumeRole\"}]}"
}
```

リソースによって多少表記が異なりますが、だいたい`role`だったり`role_arn`ですので感覚的に理解できると思います。作成した IAM ロールの Arn 情報を AWS リソースに渡せば他リソースへの操作権限が付与されます。
一方、EC2 に IAM 権限を渡すのは IAM ロールではなく IAM インスタンスプロファイルです。IaC で AWS リソースを作成する場合明示的に IAM インスタンスプロファイルを作成しなければなりません。

## EC2 を作成する IaC

```yml:CloudFormation
AWSTemplateFormatVersion: "2010-09-09"
Resources:
    EC2Instance:
        Type: "AWS::EC2::Instance"
        Properties:
            IamInstanceProfile: !Ref IAMRole
            Tags:
              -
                Key: "Name"
                Value: "Test"

    IAMRole:
        Type: "AWS::IAM::Role"
        Properties:
            Path: "/"
            RoleName: "Yuta20220815"
            AssumeRolePolicyDocument: "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"ec2.amazonaws.com\"},\"Action\":\"sts:AssumeRole\"}]}"

    # IAMインスタンスプロファイルを作成する必要がある
    IAMInstanceProfile:
        Type: "AWS::IAM::InstanceProfile"
        Properties:
            Path: "/"
            InstanceProfileName: !Ref IAMRole
            Roles:
              - !Ref IAMRole
```

```hcl:Terraform
resource "aws_instance" "EC2Instance" {
    tags = {
        Name = "Test"
    }
    iam_instance_profile = aws_iam_role.IAMRole.name
}

resource "aws_iam_role" "IAMRole" {
    path = "/"
    name = "Yuta20220815"
    assume_role_policy = "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"ec2.amazonaws.com\"},\"Action\":\"sts:AssumeRole\"}]}"
}

# IAMインスタンスプロファイルを作成する必要がある
resource "aws_iam_instance_profile" "IAMInstanceProfile" {
    path = "/"
    name = aws_iam_role.IAMRole.name
    role = aws_iam_role.IAMRole.name
}
```

もし IAM インスタンスプロファイルを作成せず、EC2 インスタンスと IAM ロールだけを作成するコードをデプロイするとエラーとなります。

![](/images/ec2-iam-instance-profile/image4.png)
_デプロイ失敗(CloudFormation)_

```console:デプロイ失敗(Terraform)
│ Error: creating EC2 Instance: InvalidParameterValue: Value (Yuta20220815) for parameter iamInstanceProfile.name is invalid. Invalid IAM Instance Profile name
│       status code: 400, request id: 7fe3f4c5-b023-40e9-8d31-69e799d5b18a
│
│   with aws_instance.EC2Instance,
│   on ec2.tf line 1, in resource "aws_instance" "EC2Instance":
│    1: resource "aws_instance" "EC2Instance" {
```

EC2 を構築するときはインスタンスプロファイルのリソースを明示的に指定する必要があります。

# 所感

IAM インスタンスプロファイルについて紹介しました。
コンソールから EC2 に IAM ロールの権限を渡すときは IAM ロールを選択するのであまり意識することはありませんが、IaC で AWS リソースを構築するときに思わぬ落とし穴が潜んでいましたので皆さんも注意してみてください。

疑問点としてなぜ EC2 は IAM ロールからではなく IAM インスタンスプロファイル経由で権限を渡しているのかが気になりましたが、納得のいく回答は見つけられませんでした。

[この記事](https://medium.com/devops-dudes/the-difference-between-an-aws-role-and-an-instance-profile-ae81abd700d)には以下のように IAM インスタンスプロファイルは自分が誰なのかを定義するための EC2 版 IAM ユーザーと説明されています。

> An instance profile, on the other hand, defines “who am I?” Just like an IAM user represents a person, an instance profile represents EC2 instances. The only permissions an EC2 instance profile has is the power to assume a role.

ただ数ある AWS リソースのなかで EC2 だけが IAM インスタンスプロファイルで対象者を明示的に定義付けしているかまではわかりませんでした。
(古くからある AWS リソースだから当時の設計で仕方ない面などがあるのですかね？)
詳しい人がいましたら教えてくれるとうれしいです 🙇

# 英語記事

英訳しました。
https://dev.to/yuta28/what-is-iam-instance-profile-3h28

# 参考文献

https://dev.classmethod.jp/articles/do_you_know_iaminstanceprofile/

https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html

https://docs.aws.amazon.com/ja_jp/managedservices/latest/userguide/defaults-instance-profile.html

https://medium.com/devops-dudes/the-difference-between-an-aws-role-and-an-instance-profile-ae81abd700d
