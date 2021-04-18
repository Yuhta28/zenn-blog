---
title: "OpsWorksを使ってEC2インスタンスにWordPressを導入してみた"
emoji: "🐀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","OpsWorks","WordPress"]
published: false
---

# 概要
AWSでInfrastructure as Codeを実践する場合、CloudFormationを使う機会が多いと思います。
CloudFormationでEC2やRDSを簡単にプロビジョニングでき、コンソール画面から作成する手間が省けます。
CloudFormationはOS以下のレイヤーの構築をサポートしてくれますが、アプリケーションレベルのプロビジョニングは別サービスを使います。
弊社でも使っていますが、有名なのがAnsibleによるプロビジョニングです。
CloudFormationやAnsibleは日本語記事も豊富でQiitaやZennで検索するとたくさんの記事がヒットします。

※2021年4月18日現在
|                | Qiita  | Zenn |
| ----           | ----   | ---- |
| CloudFormation | 899件  | 31件 |
| Ansible        | 2804件 | 33件 |

ですが、AnsibleはOSSでありAWSのマネージドサービスではありません。
AWSのマネージドサービスでAnsibleにあたるのがAWS OpsWorksとよばれるサービスです。
https://aws.amazon.com/jp/opsworks/
OpsWorksを使うことで、サーバー構築やデプロイなどを自動化できます。
