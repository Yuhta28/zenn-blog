---
title: "AWS Cloud Control APIがリリースされていたので触ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","awscli","awscloudcontrol"]
published: false
---
# 概要
先月末にAWSからこんな機能がリリースされていました。
https://aws.amazon.com/jp/blogs/news/announcing-aws-cloud-control-api/

今回はAWS Cloud Control APIを使ってAWSリソースの取得や作成などを試してみます。


# 従来のAPI操作
AWSリソースはコンソール画面から作成や削除ができますが、AWSにはAPIが提供されておりAWS CLIを使ってAPI操作でAWSリソースの作成や削除ができます。
ですが、AWS CLIのサブコマンドは各種AWSリソースごとに統一されておらず他のリソースで活用したサブコマンドやオプションを使いまわすことができず、AWS CLIの学習は今日大変なものとなっています。

例えば特定のLambda関数の詳細を確認する場合、以下のAPIを使ってLambda関数の詳細を取得します。

```bash:Lambda
aws lambda get-function --function-name Yuta-rds-auto-stop
{
    "Configuration": {
        "FunctionName": "Yuta-rds-auto-stop",
        "FunctionArn": "arn:aws:lambda:ap-northeast-1:Account_id:function:Yuta-rds-auto-stop",
        "Runtime": "python3.7",
        "Role": "arn:aws:iam::Account_id:role/Yuta-rds-auto-stop",
        "Handler": "lambda_function.lambda_handler",
        "CodeSize": 1071,
        "Description": "",
        "Timeout": 3,
        "MemorySize": 128,
        "LastModified": "2021-06-28T15:34:07.858+0000",
        "CodeSha256": "7uhkZRPbpJc8Q/pSLk3xbZE8LbxuYm1P10qPngLz6Lw=",
        "Version": "$LATEST",
        "Environment": {
            "Variables": {
                "db_cluster": "database-yuta",
                "ACCOUNTID": "Account_id",
                "TAGKEY": "AUTO_STARTSTOP",
                "TAGVALUE": "true"
            }
        },
        "TracingConfig": {
            "Mode": "PassThrough"
        },


~~~~~~~~~~

```

ここではAPIとして`get-function`、オプションとして`fucntion-name`を指定して情報を取得しています。

次にCloudWatch Logsの特定のロググループの詳細を取得します。

```bash:CloudWatch Logs
 aws logs describe-log-groups --log-group-name-prefix /aws/lambda/Yuta-rds-auto-stop
{
    "logGroups": [
        {
            "logGroupName": "/aws/lambda/Yuta-rds-auto-stop",
            "creationTime": 1629018014661,
            "retentionInDays": 7,
            "metricFilterCount": 0,
            "arn": "arn:aws:logs:ap-northeast-1:Account_id:log-group:/aws/lambda/Yuta-rds-auto-stop:*",
            "storedBytes": 1860581
        }
    ]
}
```

使用したAPIは`describe-log-groups`でオプションとして`log-group-name-prefix`でロググループの名前を指定して詳細情報を取得しています。

指定するAPIはもちろん出力されるJSON形式もリソースごとに異なっていました。
仮にjqコマンドでJSONデータを加工するとき、加工するスクリプトファイルが使いまわしできず煩雑になってしまいます。

# Cloud ControlによるAPI操作
それでは新しくリリースされたAWS Cloud ControlによるAPI操作で同じAWSリソースを出力させてみます。

```bash:Lambda
 aws cloudcontrol get-resource --type-name AWS::Lambda::Function --identifier Yuta-rds-auto-stop
{
    "TypeName": "AWS::Lambda::Function",
    "ResourceDescription": {
        "Identifier": "Yuta-rds-auto-stop",
        "Properties": "{\"MemorySize\":128,\"Description\":\"\",\"TracingConfig\":{\"Mode\":\"PassThrough\"},\"Timeout\":3,\"Handler\":\"lambda_function.lambda_handler\",\"Role\":\"arn:aws:iam::Account_id:role/Yuta-rds-auto-stop\",\"FileSystemConfigs\":[],\"FunctionName\":\"Yuta-rds-auto-stop\",\"Runtime\":\"python3.7\",\"PackageType\":\"Zip\",\"Environment\":{\"Variables\":{\"db_cluster\":\"database-yuta\",\"ACCOUNTID\":\"Account_id\",\"TAGKEY\":\"AUTO_STARTSTOP\",\"TAGVALUE\":\"true\"}},\"Arn\":\"arn:aws:lambda:ap-northeast-1:Account_id:function:Yuta-rds-auto-stop\",\"Architectures\":[\"x86_64\"]}"
    }
}
```

```bash:CloudWatch Logs
aws cloudcontrol get-resource --type-name AWS::Logs::LogGroup --identifier /aws/lambda/Yuta-rds-auto-stop
{
    "TypeName": "AWS::Logs::LogGroup",
    "ResourceDescription": {
        "Identifier": "/aws/lambda/Yuta-rds-auto-stop",
        "Properties": "{\"RetentionInDays\":7,\"LogGroupName\":\"/aws/lambda/Yuta-rds-auto-stop\",\"Arn\":\"arn:aws:logs:ap-northeast-1:Account_id:log-group:/aws/lambda/Yuta-rds-auto-stop:*\"}"
    }
}
```

`get-resource`という共通のAPIを使って同じJSON出力形式で2つのAWSリソースの詳細情報を取得しました。

ここでAWS Cloud Control APIについて詳しく見てみましょう。

# AWS Cloud Control APIについて
Cloud Control APIは前述のとおり膨大なAWSリソースや数十のサードパーティーのサービス全体で、リソースの作成、読み取り、更新、削除および一覧表示を実行するための標準的なAPIセットです。
`aws cloudcontrol help`で現在使用できるサブコマンドが一覧表示されます。

```bash:helpコマンド
Available Commands
******************

* cancel-resource-request

* create-resource

* delete-resource

* get-resource

* get-resource-request-status

* help

* list-resource-requests

* list-resources

* update-resource

* wait
```

CRUDL操作(CRUD+List)は以下のコマンドです。

- create-resource
- get-resource
- update-resource
- delete-resource
- list-resources

# 参考文献
https://aws.amazon.com/jp/cloudcontrolapi/
https://dev.classmethod.jp/articles/aws-cloud-control-api-kinesis/
https://blog.serverworks.co.jp/aws-cli-cloudcontrol_
https://codezine.jp/article/detail/14970