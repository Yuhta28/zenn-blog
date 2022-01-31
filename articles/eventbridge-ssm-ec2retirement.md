---
title: "EventBridgeとSystems ManagerでEC2 Retirement時の再起動作業を自動化してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","eventbrige","systemsmanager"]
published: true
---
# 背景
たまにAWSからこんなお知らせが会社に届くことがあります。
![](/images/eventbridge-ssm-ec2retirement/image1.jpg)

これはAWSが保有しているハードウェアのメンテナンスのため、中で稼働しているEC2インスタンスをリタイアメントさせるお知らせです。
この連絡が届いたらユーザーは約2週間以内にEC2インスタンスを再起動させないといけません。
弊社は検証環境に関しましては夜間に自動で停止し、翌朝に起動させる設定にしていますので問題はありませんが[^1]、本番環境に関しましては日中に再起動するとサービス影響が出るためビジネス部門からも避けてほしいという要望があります。
作業自体は再起動するだけの簡単なものですが、それだけのために深夜に作業するのも嫌なので、EventBridgeとSysyems Managerを使って自動化してみました。
[^1]: 詳しくは以前書いた記事をご覧ください。https://zenn.dev/yuta28/articles/ec2-schedule

# アーキテクチャ
今回のAWSアーキテクチャは以下のとおりです。
![](/images/eventbridge-ssm-ec2retirement/image2.png)

ビジネス部門から再起動の希望時間をヒアリングし、EventBridgeで時間になったらSystems Managerの機能の一つであるオートメーションを実行させて対象のEC2を再起動させます。

## Systems Manager
Systems Managerは[別の記事](https://zenn.dev/yuta28/articles/ssm-cloudwatch-get)でも使いましたが、その時は`Run Command`という機能を使ってエージェントソフトインストール作業を一括で実行していました。
Systems Managerは運用業務を自動化してくれるための色々な機能がありますが、今回は`オートメーション`を使ってEC2を再起動させることを自動化します。
### オートメーション
https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/systems-manager-automation.html
Systems Managerにはオートメーションというあらかじめ定義されたJSONまたはYAML形式で書かれたRunbookと呼ばれるドキュメントを実行させることでIT運用業務を自動で実行する機能を備えています。
RunbookはAWSが事前に用意してくれたもの以外に自分で作成もできます。
![](/images/eventbridge-ssm-ec2retirement/image4.png)
今回はAWSのRunbookを基に自分で新しいRunbookを作成していきます。

```json:EC2Retirement
{
  "description": "Stop EC2 instances(s)",
  "schemaVersion": "0.3",
  "assumeRole": "{{ AutomationAssumeRole }}",
  "parameters": {
    "InstanceId": {
      "type": "StringList",
      "description": "(Required) EC2 Instance(s) to stop"
    },
    "AutomationAssumeRole": {
      "type": "String",
      "description": "(Optional) The ARN of the role that allows Automation to perform the actions on your behalf.",
      "default": ""
    }
  },
  "mainSteps": [
    {
      "name": "stopInstances",
      "action": "aws:changeInstanceState",
      "onFailure": "Continue",
      "inputs": {
        "InstanceIds": "{{ InstanceId }}",
        "DesiredState": "stopped"
      }
    },
    {
      "name": "forceStopInstances",
      "action": "aws:changeInstanceState",
      "inputs": {
        "InstanceIds": "{{ InstanceId }}",
        "CheckStateOnly": false,
        "DesiredState": "stopped",
        "Force": true
      }
    },
    {
      "name": "startInstances",
      "action": "aws:changeInstanceState",
      "inputs": {
        "InstanceIds": "{{ InstanceId }}",
        "DesiredState": "running"
      }
    }
  ]
}
```

## EventBridge
EventBridgeも以前[別の記事](https://zenn.dev/yuta28/articles/eventbridge-slack)で似たような使い方をしました。
あの時は、特定イベントをトリガーにSNSへ通知を飛ばすイベントパターンを使いましたが、今回はスケジューリング設定を使って特定の時間になったらSystems Managerを呼び出してEC2インスタンスを再起動させるようにしました。
![](/images/eventbridge-ssm-ec2retirement/image3.png)
EventBridgeのスケジューリング設定ではcron式も使えるので定期的なバッチ処理の起動トリガーとしても優秀です。
ターゲットの選択ではSystems Managerオートメーションを選び、ドキュメント選択から先ほど作成したRunbookを選びます。
InstanceId部分は再起動したいEC2インスタンスのIDを入力します。
(そのほかはデフォルト設定で大丈夫です)
![](/images/eventbridge-ssm-ec2retirement/image5.png)
これでEventBridgeのルールが作成されました。

# 実行結果
ビジネス部門からは土曜の深夜に実施してほしいという要望でしたので、1/22(土)の深夜3時に実行するようにセットしました。
金曜日にEventBridgeで対象EC2のInstanceIdがセットされていることを確認して仕事を終えて翌朝念のためCloudTrailからイベントログを確認してみました。
## インスタンス停止
![](/images/eventbridge-ssm-ec2retirement/image6.png)
## インスタンス起動
![](/images/eventbridge-ssm-ec2retirement/image7.png)

問題なくログ上で再起動されていることが記録されました。
EC2イベントにもリタイアメント対象のEC2イベントステータスもcompleteされており、EC2リタイアメント通知後の退避作業を自動化せることができました。

# 課題
Systems ManagerとEventBridgeを使ってEC2の再起動を自動化させましたが課題はあります。
弊社ではサーバーの監視ツールにSaaSのMackerelを利用しています。
https://ja.mackerel.io/

外形監視とホストリソース監視をしていますが、EC2を停止する際に監視アラートが発砲し、Slackに通知が飛んできます。
![](/images/eventbridge-ssm-ec2retirement/image8.png)
*503エラーが起きたときのアラート検知*

現状の対応策としては仕事終わりに外形監視とホストリソースを監視を一時的に解除し、翌朝に再度監視設定を戻すようにしました。

MackerelにはCLI[^2]ツールがありますので、将来的にはEventBridgeでEC2を停止する前にLambdaで対象EC2の監視設定を解除して、起動後に監視設定を元に戻すような運用にしていきたいと考えています。
[^2]: https://mackerel.io/ja/docs/entry/advanced/cli

# 所感
EventBridgeとSystems Managerを使って定常作業の自動化を実現しました。
再起動時間に関してはビジネス部門の希望によって異なるのでEC2と時間のセットは手動にしていますが、セットした後は自分で作業しなくても勝手に夜間に再起動してくれるので運用業務改善を実現できて良かったです。

Systems Managerは運用業務自動化を実現できる機能が数多く用意されていて一番気に入っているサービスですので、皆さんも試してみてください。

# 参考文献
https://dev.classmethod.jp/articles/incident-manager-create-eventbridge/
https://dev.classmethod.jp/articles/tsnote-ec2-ssm-automation/
https://aws.amazon.com/jp/blogs/mt/automate-remediation-actions-for-amazon-ec2-notifications-and-beyond-using-ec2-systems-manager-automation-and-aws-health/