---
title: "インスタンスプロファイルって？"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "IAM", "EC2"]
published: false
---

# 概要
AWSリソースにIAMポリシー権限を渡すとき、IAMポリシーがアタッチされているIAMロールを作成し、そのIAMロールをAWSリソースに付与することで付与されたAWSリソースは他のリソースに対して操作できます。
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

リソースによって多少表記が異なりますが、だいたい`role`だったり`role_arn`ですので感覚的に理解できると思います。IAMロールを作成して、作成したIAMロールのArn情報を他のAWSリソースに渡せばいいのでシンプルに収まります。
ですが、EC2にIAM権限を渡すのはIAMロールではなく、IAMインスタンスプロファイルです。IaCでAWSリソースを作成する場合明示的にIAMインスタンスプロファイルを作成し、作成したIAMインスタンスプロファイルのArnをEC2に渡す必要が出てきます。

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
            RoleName: "Yuta20210911"
            AssumeRolePolicyDocument: "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"ec2.amazonaws.com\"},\"Action\":\"sts:AssumeRole\"}]}"

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
    iam_instance_profile = "${aws_iam_role.IAMRole.name}"
}

resource "aws_iam_role" "IAMRole" {
    path = "/"
    name = "Yuta20210911"
    assume_role_policy = "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"ec2.amazonaws.com\"},\"Action\":\"sts:AssumeRole\"}]}"
}

resource "aws_iam_instance_profile" "IAMInstanceProfile" {
    path = "/"
    name = "${aws_iam_role.IAMRole.name}"
    roles = [
        "${aws_iam_role.IAMRole.name}"
    ]
}
```

EC2構築のIaCにはIAMロールのArnを渡すパラメータは存在せず、IAMインスタンスプロファイルを渡すパラメータがあります。ここに __IAMロールのArnを記入してもエラー__ となります。

# 参考文献
https://dev.classmethod.jp/articles/do_you_know_iaminstanceprofile/

https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html

https://docs.aws.amazon.com/ja_jp/managedservices/latest/userguide/defaults-instance-profile.html

https://medium.com/devops-dudes/the-difference-between-an-aws-role-and-an-instance-profile-ae81abd700d