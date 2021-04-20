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

※2021年4月20日現在
|                | Qiita  | Zenn |
| ----           | ----   | ---- |
| CloudFormation | 900件  | 31件 |
| Ansible        | 2804件 | 33件 |

ですが、AnsibleはOSSでありAWSのマネージドサービスではありません。
AWSのマネージドサービスでAnsibleにあたるのがAWS OpsWorksとよばれるサービスです。
https://aws.amazon.com/jp/opsworks/
OpsWorksを使うことで、サーバー構築やデプロイなどを自動化できます。
ところが、このOpsWorksですが紹介記事が全然見つからず、QiitaやZennで検索しても圧倒的に少ない状況です。

※2021年4月20日現在
|                | Qiita  | Zenn |
| ----           | ----   | ---- |
| OpsWorks | 95件  | 1件 |

どうしてここまで利用者が少ないのか、実際に触って使用感を試してみたいと思います。

# ハンズオン内容
表題の通りEC2インスタンスにLAMP環境を構築し、WordPressを導入してみる一連の流れをOpsWorksで挑戦してみたいと思います。

- Ubuntu 20.04
- Apache
- MariaDB
- PHP 7.4

スクラップで連載しているAWSインフラ構築日記の構成を基にしてWordPressを構築していきます。
https://zenn.dev/yuta28/scraps/1e14a65a26626b

# OpsWorksについて
冒頭でも説明しましたが、OpsWorksについて詳しく解説していきます。
OpsWorksには3つのバージョンが提供されています。

- AWS OpsWorks for Chef Automate
- AWS OpsWorks for Puppet Enterprise
- AWS OpsWorksスタック

ChefとPuppetですが、この2つは構成管理ツールつまりAnsibleと同様の機能を持っています。
大きく異なる点ですが、Ansibleはエージェントレスであるのに対しこの2つは変更先であるクライアントにエージェントをインストールして導入します。
ほかにもAnsibleはYAMLで処理内容を記述しますが、ChefはRuby、PuppetはDSLという独自言語を用いて処理内容を記述します。
