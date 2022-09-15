---
title: "IAMインスタンスプロファイルって？"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "IAM", "EC2"]
published: true
---

# 概要

AWSリソースにIAMポリシー権限を渡すときは IAM ポリシーがアタッチされているIAMロールを作成し、そのIAMロールをAWSリソースに付与することで付与されたAWSリソースは他のリソースへの操作権限が与えられます。
![](/images/ec2-iam-instance-profile/image3.png)

ですがEC2を作成する画面に IAM ロールをアタッチする箇所がなく、IAMインスタンスプロファイルを設定する箇所が存在します。

![](/images/ec2-iam-instance-profile/image1.png)

![](/images/ec2-iam-instance-profile/image2.png)

AWS CLIでもパラメータに`IamInstanceProfile`という項目があり、Arnにも`instance-profile`と記載されています。

```powershell
 aws ec2 describe-instances --query "Reservations[].Instances[].IamInstanceProfile.Arn"
[
    "arn:aws:iam::XXXXXXXXXXXX:instance-profile/Yuta20210911"
]
```

このIAMインスタンスプロファイルというものは何なのでしょうか？

# IAMインスタンスプロファイルについて

https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html
AWSのドキュメントに以下記載がありました。

> Amazon EC2は、IAMロールのコンテナとしてインスタンスプロファイルを使用します。IAMコンソールを使用してIAMロールを作成すると、コンソールによりインスタンスプロファイルが自動的に作成され、対応するロールと同じ名前が付けられます。

インスタンスプロファイルはIAMロールを格納するコンテナであり、そのコンテナをEC2にアタッチすると記載されています。
インスタンスプロファイルはEC2のみに使われる概念であり、IAMロールを作成すると自動で同名のIAMインスタンスプロファイルが作成されます。

普段AWSを操作する時にIAMインスタンスプロファイルとIAMロールの違いについて意識することはないと思います。気を付ける場面でいうとインフラのコード化、すなわち**Infrastructure as Code(IaC)** を実践するときだと思います。

# IaC開発時の注意点

LambdaやCodePipelineにIAMロールをアタッチするときはCloudFormationやTerraformでroleパラメータにIAMロールのArnを設定します。

## Lambdaを作成するIaC

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

リソースによって多少表記が異なりますが、だいたい`role`だったり`role_arn`ですので感覚的に理解できると思います。作成したIAMロールのArn情報をAWSリソースに渡せば他リソースへの操作権限が付与されます。
一方、EC2にIAM権限を渡すのはIAMロールではなくIAMインスタンスプロファイルです。IaCでAWSリソースを作成する場合明示的にIAMインスタンスプロファイルを作成しなければなりません。

## EC2を作成するIaC

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

もしIAMインスタンスプロファイルを作成せず、EC2インスタンスとIAMロールだけを作成するコードをデプロイするとエラーとなります。

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

EC2を構築するときはインスタンスプロファイルのリソースを明示的に指定する必要があります。

# 所感

IAMインスタンスプロファイルについて紹介しました。
コンソールからEC2にIAMロールの権限を渡すときはIAMロールを選択するのであまり意識することはありませんが、IaCでAWSリソースを構築するときに思わぬ落とし穴が潜んでいましたので皆さんも注意してみてください。

疑問点としてなぜEC2はIAMロールからではなくIAMインスタンスプロファイル経由で権限を渡しているのかが気になりましたが、納得のいく回答は見つけられませんでした。

[この記事](https://medium.com/devops-dudes/the-difference-between-an-aws-role-and-an-instance-profile-ae81abd700d)には以下のようにIAMインスタンスプロファイルは自分が誰なのかを定義するためのEC2版IAMユーザーと説明されています。

> An instance profile, on the other hand, defines “who am I?” Just like an IAM user represents a person, an instance profile represents EC2 instances. The only permissions an EC2 instance profile has is the power to assume a role.

ただ数あるAWSリソースのなかでEC2だけがIAMインスタンスプロファイルで対象者を明示的に定義付けしているかまではわかりませんでした。
(古くからあるAWSリソースだから当時の設計で仕方ない面などがあるのですかね？)
詳しい人がいましたら教えてくれるとうれしいです 🙇

# 英語記事

英訳しました。
https://dev.to/yuta28/what-is-iam-instance-profile-3h28

# 参考文献

https://dev.classmethod.jp/articles/do_you_know_iaminstanceprofile/

https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html

https://docs.aws.amazon.com/ja_jp/managedservices/latest/userguide/defaults-instance-profile.html

https://medium.com/devops-dudes/the-difference-between-an-aws-role-and-an-instance-profile-ae81abd700d
