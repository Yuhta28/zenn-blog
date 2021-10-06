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
ですが、AWS CLIのサブコマンドは各種AWSリソース毎に統一されておらず他のリソースで活用したサブコマンドやオプションを使いまわすことができず、AWS CLIの学習は今日大変なものとなっています。

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

使用した