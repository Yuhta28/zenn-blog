---
title: "AWS Cloud Control APIがリリースされていたので触ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","awscli","awscloudcontrol"]
published: true
---
# 概要
先月末にAWSからこんな機能がリリースされていました。
https://aws.amazon.com/jp/blogs/news/announcing-aws-cloud-control-api/

今回はAWS Cloud Control APIを使ってAWSリソースの取得や作成などを試してみます。
https://docs.aws.amazon.com/ja_jp/cloudcontrolapi/latest/userguide/what-is-cloudcontrolapi.html

# 従来のAPI操作
AWSリソースはコンソール画面から作成や削除ができますが、AWSにはAPIが提供されておりAWS CLIを使ってAPI操作でAWSリソースの作成や削除ができます。
ですが、AWS CLIのサブコマンドは各種AWSリソースごとに統一されておらず他のリソースで活用したサブコマンドやオプションを使いまわすことができず、AWS CLIの学習は今日大変なものとなっています。

例えば特定のLambda関数の詳細を確認する場合、以下のAPIを使ってLambda関数の詳細を取得します。

```powershell:Lambda
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

```powershell:CloudWatch Logs
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

```powershell:Lambda
 aws cloudcontrol get-resource --type-name AWS::Lambda::Function --identifier Yuta-rds-auto-stop
{
    "TypeName": "AWS::Lambda::Function",
    "ResourceDescription": {
        "Identifier": "Yuta-rds-auto-stop",
        "Properties": "{\"MemorySize\":128,\"Description\":\"\",\"TracingConfig\":{\"Mode\":\"PassThrough\"},\"Timeout\":3,\"Handler\":\"lambda_function.lambda_handler\",\"Role\":\"arn:aws:iam::Account_id:role/Yuta-rds-auto-stop\",\"FileSystemConfigs\":[],\"FunctionName\":\"Yuta-rds-auto-stop\",\"Runtime\":\"python3.7\",\"PackageType\":\"Zip\",\"Environment\":{\"Variables\":{\"db_cluster\":\"database-yuta\",\"ACCOUNTID\":\"Account_id\",\"TAGKEY\":\"AUTO_STARTSTOP\",\"TAGVALUE\":\"true\"}},\"Arn\":\"arn:aws:lambda:ap-northeast-1:Account_id:function:Yuta-rds-auto-stop\",\"Architectures\":[\"x86_64\"]}"
    }
}
```

```powershell:CloudWatch Logs
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

```powershell:helpコマンド
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

先ほどは`get-resource`を使ってAWSリソースの詳細情報を取得しました。
`--identifier`オプションにリソースの名前を指定します。そして必須オプションで大事なのが、`--type-name`です。
ここにはAWSリソースのタイプを指定しますが、CloudFormationを使ったことがある人なら見たことがあるプロパティだと思います。
2021年9月30日時点では以下のAWSリソースがサポートされているとドキュメントに記載があります。
https://docs.aws.amazon.com/ja_jp/cloudcontrolapi/latest/userguide/supported-resources.html

まだまだ数が少なくEC2インスタンスの起動もサポートされていない状態です。

リリースされたばかりですので定期的に確認して更新されているか確認しておきましょう。

公式ドキュメントにCloud Watch Logsを使ってCRUDL操作まで一通り経験できるチュートリアルがありましたので、早速始めてみたいと思います。
https://docs.aws.amazon.com/ja_jp/cloudcontrolapi/latest/userguide/resource-operations.html

# ハンズオン
AWS Cloud Control APIを使うためにはAWS CLIのバージョンが2.2.43以上である必要があります。
手元のバージョンが古い場合は更新しておきましょう。
## 事前準備
https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2.html

```bash
aws --version
aws-cli/2.2.44 Python/3.8.8 Linux/5.10.16.3-microsoft-standard-WSL2 exe/x86_64.ubuntu.20 prompt/off
```

## リソース作成

それではCloud Watch Logsを作成していきます。`create-resource`サブコマンドを指定し、`--type-name`でCloud Watch Logsのリソースタイプを指定します。
`--desired-state`にリソースごとに必要なプロパティを設定します。
ロググループ名とログの保持期間を設定しました。

```powershell
aws cloudcontrol create-resource --type-name AWS::Logs::LogGroup --desired-state "{LogGroupName: CloudApiLogGroup,RetentionInDays:90}"
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup",
        "RequestToken": "8c98858b-a70b-4381-850e-606005f8c5ee",
        "Operation": "CREATE",
        "OperationStatus": "IN_PROGRESS",
        "EventTime": "2021-10-07T21:36:48.335000+09:00"
    }
}
```

リソースを作成するとリクエストトークンが発行されます。
リソースの作成進捗を確認する場合はリクエストトークンを指定して`get-resource-request-status`を実行します。

```powershell
aws cloudcontrol get-resource-request-status --request-token 8c98858b-a70b-4381-850e-606005f8c5ee
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup",
        "RequestToken": "8c98858b-a70b-4381-850e-606005f8c5ee",
        "Operation": "CREATE",
        "OperationStatus": "SUCCESS",
        "EventTime": "2021-10-07T21:36:49.146000+09:00"
    }
}
```

![](/images/aws-cloudcontrol-handson/image1.png)
*コンソール画面*

ステータスが**SUCCESS**でコンソール画面上からも作成されていることが確認できました。

## リソース確認
リソース確認は先ほどの`get-resource`を使います。
```powershell
aws cloudcontrol get-resource --type-name AWS::Logs::LogGroup --identifier CloudApiLogGroup
{
    "TypeName": "AWS::Logs::LogGroup",
    "ResourceDescription": {
        "Identifier": "CloudApiLogGroup",
        "Properties": "{\"RetentionInDays\":90,\"LogGroupName\":\"CloudApiLogGroup\",\"Arn\":\"arn:aws:logs:ap-northeast-1:Account_id:log-group:CloudApiLogGroup:*\"}"
    }
}
```

## リソース更新
リソース更新は`update-resource`です。
新しい必須オプションとして`--patch-document`を記載します。
ここではログの保持期間を90日から180日に変更します。

### patch-document
`patch-document`にはJSONフォーマットで更新情報を記載します。
以下2つのプロパティを設定します。

- op
  - オペレーションタイプを指定
    - add,remove,replace,move,copy,testの中から選択
- path
  - リソースプロパティを指定
    - 今回は`RetentionInDays`を指定します

```bash
aws cloudcontrol update-resource --type-name AWS::Logs::LogGroup --identifier CloudApiLogGroup --patch-document "[{\"op\":\"replace\",\"path\":\"/RetentionInDays\",\"value\":180}]"
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup",
        "RequestToken": "e3781812-727f-4af7-9e3f-91fb6b7851e8",
        "Operation": "UPDATE",
        "OperationStatus": "IN_PROGRESS",
        "EventTime": "2021-10-07T23:31:09.989000+09:00",
        "ResourceModel": "{\"RetentionInDays\":180,\"LogGroupName\":\"CloudApiLogGroup\"}"
    }
}
```

またリクエストトークンが発行されましたので、`get-resource-request-status`でステータスを確認してみます。


```powershell
 aws cloudcontrol get-resource-request-status --request-token e3781812-727f-4af7-9e3f-91fb6b7851e8
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup",
        "RequestToken": "e3781812-727f-4af7-9e3f-91fb6b7851e8",
        "Operation": "UPDATE",
        "OperationStatus": "SUCCESS",
        "EventTime": "2021-10-07T23:31:10.933000+09:00"
    }
}
```

![](/images/aws-cloudcontrol-handson/image2.png)
*保持期間が6か月に変更された*

##### おまけ
PowerShellですとエスケープ文字がきかないのか、うまく更新されませんでした。
WindowsユーザーはWSL2上で実行してみてください。

```powershell
 aws cloudcontrol update-resource --type-name AWS::Logs::LogGroup --identifier CloudApiLogGroup --patch-document "[{`"op`":`"replace`",`"path`":`"/RetentionInDays`",`"value`":120}]"

An error occurred (InternalFailure) when calling the UpdateResource operation (reached max retries: 2): Internal Failure
```

## リソース一覧表示
特定のAWSリソースのリストを一挙見たい場合は`list-resoureces`を使います。

```powershell
aws cloudcontrol list-resources --type-name AWS::Logs::LogGroup
{
    "TypeName": "AWS::Logs::LogGroup",
    "ResourceDescriptions": [
        {
            "Identifier": "/aws/lambda/Yuta-rds-auto-stop",
            "Properties": "{\"RetentionInDays\":7,\"LogGroupName\":\"/aws/lambda/Yuta-rds-auto-stop\",\"Arn\":\"arn:aws:logs:ap-northeast-1:Account_id:log-group:/aws/lambda/Yuta-rds-auto-stop:*\"}"
        },
        {
            "Identifier": "/aws/lambda/myfunc01_handler",
            "Properties": "{\"RetentionInDays\":14,\"LogGroupName\":\"/aws/lambda/myfunc01_handler\",\"Arn\":\"arn:aws:logs:ap-northeast-1:Account_id:log-group:/aws/lambda/myfunc01_handler:*\"}"
        },
        {
            "Identifier": "CloudApiLogGroup",
            "Properties": "{\"RetentionInDays\":180,\"LogGroupName\":\"CloudApiLogGroup\",\"Arn\":\"arn:aws:logs:ap-northeast-1:Account_id:log-group:CloudApiLogGroup:*\"}"
        },
        {
            "Identifier": "aws-controltower/CloudTrailLogs",
            "Properties": "{\"RetentionInDays\":14,\"LogGroupName\":\"aws-controltower/CloudTrailLogs\",\"Arn\":\"arn:aws:logs:ap-northeast-1:Account_id:log-group:aws-controltower/CloudTrailLogs:*\"}"
        }
    ]
}
```

地味に`aws logs`にはそのような機能がなく、ターミナル上でCloud Watch Logsのロググループをリスト表示できるのはこれが初だったりします。(非公式なツールにはあるかも)

## リソース削除
リソースの削除は`delete-resource`です。

```powershell
aws cloudcontrol delete-resource --type-name AWS::Logs::LogGroup --identifier CloudApiLogGroup
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup",
        "RequestToken": "094f5413-ba56-4cec-8049-63fcdcb0288b",
        "Operation": "DELETE",
        "OperationStatus": "IN_PROGRESS",
        "EventTime": "2021-10-08T00:18:08.680000+09:00"
    }
}

# リクエストトークンで進捗確認
aws cloudcontrol get-resource-request-status --request-token 094f5413-ba56-4cec-8049-63fcdcb0288b
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup",
        "RequestToken": "094f5413-ba56-4cec-8049-63fcdcb0288b",
        "Operation": "DELETE",
        "OperationStatus": "SUCCESS",
        "EventTime": "2021-10-08T00:18:09.200000+09:00"
    }
}
```

![](/images/aws-cloudcontrol-handson/image3.png)
*削除*

# 冪等性を保つ
AWS Cloud Control APIの`create-resource`,`update-resource`,`delete-resource`にはクライアントトークンを発行させるパラメーターがあります。

```powershell
aws cloudcontrol create-resource `
--type-name AWS::Logs::LogGroup `
--desired-state "{LogGroupName: CloudApiLogGroup,RetentionInDays:90}" `
--client-token 123
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup",
        "RequestToken": "9262db04-0c2d-4483-8357-f68c9df3c37d",
        "Operation": "CREATE",
        "OperationStatus": "IN_PROGRESS",
        "EventTime": "2021-10-08T00:38:01.514000+09:00"
    }
}
```

このパラメーターはリクエストの冪等性を保証するために使用されるものです。
冪等性とは同じ操作を何回繰り返しても同じ結果になることを示す数学用語です。
ITではAnsibleなどの構成管理ツールでよく使われる言葉です。
冪等性が保証されることで、誤って同じ操作をしても予期せぬエラーを防ぐことが可能になります。

## クライアントトークンを発行しないケース
クライアントトークンを発行しない状態でCloud Watch Logsの同名のロググループを作成するコマンドを複数回実行したとします。

```powershell
aws cloudcontrol create-resource --type-name AWS::Logs::LogGroup --desired-state "{LogGroupName: CloudApiLogGroup-non-token,RetentionInDays:90}"
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup-non-token",
        "RequestToken": "2a13c377-57a3-4460-9433-2f80a1725649",
        "Operation": "CREATE",
        "OperationStatus": "IN_PROGRESS",
        "EventTime": "2021-10-08T00:49:48.628000+09:00"
    }
}

aws cloudcontrol create-resource --type-name AWS::Logs::LogGroup --desired-state "{LogGroupName: CloudApiLogGroup-non-token,RetentionInDays:90}"
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup-non-token",
        "RequestToken": "54bc582b-62ec-47e0-93d2-a7d442f98659",
        "Operation": "CREATE",
        "OperationStatus": "IN_PROGRESS",
        "EventTime": "2021-10-08T00:49:51.993000+09:00"
    }
}
```

実行するたびに異なるリクエストトークンが発行され、コマンドそのものは正常終了します。
ですが、`get-resource-request-status`でステータスを確認しますと最初のリクエストトークンは`SUCCESS`ですが、2回目に実行したコマンドのリクエストトークンは`FAILED`になっています。

```powershell
 aws cloudcontrol get-resource-request-status --request-token 2a13c377-57a3-4460-9433-2f80a1725649
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup-non-token",
        "RequestToken": "2a13c377-57a3-4460-9433-2f80a1725649",
        "Operation": "CREATE",
        "OperationStatus": "SUCCESS", #成功が返っている
        "EventTime": "2021-10-08T00:49:49.514000+09:00"
    }
}

aws cloudcontrol get-resource-request-status --request-token 54bc582b-62ec-47e0-93d2-a7d442f98659
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup-non-token",
        "RequestToken": "54bc582b-62ec-47e0-93d2-a7d442f98659",
        "Operation": "CREATE",
        "OperationStatus": "FAILED", 
        "EventTime": "2021-10-08T00:49:52.694000+09:00",
        "StatusMessage": "Resource of type 'AWS::Logs::LogGroup' with identifier '{\"/properties/LogGroupName\":\"CloudApiLogGroup-non-token\"}' already exists.",
        "ErrorCode": "AlreadyExists" #既に同名のロググループ名が存在しており、失敗ステータスを返している
    }
}
```

## クライアントトークンを発行するケース
クライアントトークンを発行するオプションを指定して同名のロググループを作成するコマンドを複数回実行します。

```powershell
aws cloudcontrol create-resource --type-name AWS::Logs::LogGroup --desired-state "{LogGroupName: CloudApiLogGroup,RetentionInDays:90}"  --client-token 123
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup",
        "RequestToken": "9262db04-0c2d-4483-8357-f68c9df3c37d",
        "Operation": "CREATE",
        "OperationStatus": "IN_PROGRESS",
        "EventTime": "2021-10-08T00:38:01.514000+09:00"
    }
}

aws cloudcontrol create-resource --type-name AWS::Logs::LogGroup --desired-state "{LogGroupName: CloudApiLogGroup,RetentionInDays:90}"  --client-token 123
{
    "ProgressEvent": {
        "TypeName": "AWS::Logs::LogGroup",
        "Identifier": "CloudApiLogGroup",
        "RequestToken": "9262db04-0c2d-4483-8357-f68c9df3c37d",
        "Operation": "CREATE",
        "OperationStatus": "SUCCESS", #成功ステータスを返している
        "EventTime": "2021-10-08T00:38:02.890000+09:00"
    }
}
```

クライアントトークンを発行した状態で２回目に同名のロググループを作成するコマンドを実行すると、ステータスが`IN_PROGRESS`ではなく、`SUCCESS`になっていることがわかります。
AWSでは冪等性を保つために**常に**クライアントトークンを渡すことを推奨しております。
クライアントトークンは36時間の間有効です。
最大128文字まで指定できますので、UUIDなど一意なクライアントトークンを発行するようにしたほうがよさそうです。

# 所感
AWS Cloud Control APIについて研究してみました。
GAされたとはいえ、まだまだ機能に不足されている面がありすぐに使えるものではありませんが従来のAWS CLIにはなかった冪等性の担保がありますのでコマンド実行時には見落とすエラーの発生防止になりそうです。

今後の機能アップデートを楽しみにしています。

# 参考文献
https://aws.amazon.com/jp/cloudcontrolapi/
https://dev.classmethod.jp/articles/aws-cloud-control-api-kinesis/
https://blog.serverworks.co.jp/aws-cli-cloudcontrol_
https://codezine.jp/article/detail/14970