---
title: "OpsWorksを使ってEC2インスタンスにWordPressを導入してみた"
emoji: "🐀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","OpsWorks","WordPress"]
published: true
---

# 概要
AWSでInfrastructure as Codeを実践する場合、CloudFormationを使う機会が多いと思います。
CloudFormationを使えばEC2やRDSを簡単にプロビジョニングでき、コンソール画面から作成する手間が省けます。
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

スクラップで連載しているAWSインフラ構築日記の構成を基にしてWordPressを構築していきます。
https://zenn.dev/yuta28/scraps/1e14a65a26626b

# OpsWorksについて
冒頭でも説明しましたが、OpsWorksについて詳しく解説していきます。
OpsWorksには3つのバージョンが提供されています。

- AWS OpsWorks for Chef Automate
- AWS OpsWorks for Puppet Enterprise
- AWS OpsWorks Stack

ChefとPuppetですが、この2つは構成管理ツール、つまりAnsibleと同様の機能を持っています。
大きく異なる点ですが、Ansibleはエージェントレスであるのに対しこの2つは変更先であるクライアントにエージェントをインストールして導入します。
ほかにもAnsibleはYAMLで処理内容を記述しますが、ChefはRuby、PuppetはDSLという独自言語を用いて処理内容を記述します。
ChefとPuppetの詳しい説明は割愛しますので、気になるかたはそれぞれのHPからご覧ください。
https://www.chef.io/products/chef-infra
https://puppet.com/

AWS OpsWorks for Chef AutomateとAWS OpsWorks for Puppet Enterpriseは使っているツールが異なるだけで使い方に違いはありません。
AWS OpsWorks StackもChefを使いますが、こちらはEC2インスタンスにインストールしたChef Soloクライアントを使うことで自分自身に設定変更命令を出して構成管理やプロビジョニングを実行するものになります。
https://aws.amazon.com/jp/opsworks/chefautomate/faqs/?nc=sn&loc=5

今回はWordPressを構築するEC2インスタンス内にChef Soloクライアントを導入して自身にChefを実行する形にしていきたいので、AWS OpsWorks Stackを使用していきたいと思います。
（以下、Stackと表記します）

# Stackの使い方
AWSコンソール画面からOpsWorksのページを開いてみます。
左のStackを選択します。
![](https://storage.googleapis.com/zenn-user-upload/mjbaiuq66vpnquhdd1iiz0aqjkbq)

Stackのダッシュボードが表示されますので、Add your first stackを選択します。
Stackの種類を選ぶことができ、サンプルとChefのバージョンを選ぶことができます。初めてOpsWorksを触るときは、Sampleを選んで使用感を試すのもいいと思います。
![](https://storage.googleapis.com/zenn-user-upload/5f8ok3g0i4bmoz5wfz1acqvqwn3x)
https://docs.aws.amazon.com/opsworks/latest/userguide/gettingstarted-intro.html
## Stackの作成
今回はChef12系を使ってEC2インスタンスにWordPressをデプロイしてみようと思います。
Chef 12 stackを選択するとStackの初期設定項目が表示されますので必要箇所を記載します。
![](https://storage.googleapis.com/zenn-user-upload/4ruw274xlj3jw1n9e24lpqzeap0x)
UbuntuにWordPressを導入するための、簡単なレシピを作りました。
https://github.com/Yuhta28/chef-for-opsworks
Chefのベストプラクティスがどういったものかわからないので、かなり単純なものになりましたがこちらのリポジトリを使うことで各種ミドルウェアとWordPressのインストールがChefで実行できるようになります。
Stackを作成したら次にLayerを作成します。
## Layerの作成
LayerはStackに一つ以上含まれているコンポーネントを表します。
![](https://storage.googleapis.com/zenn-user-upload/xmt9u7zhyaf5tkb454q09lm1d5eg)
OpsWorksタブを選択し、名前を作成します。
名前を作成後、LayerメニューからRecipesタブへ移動します。
![](https://storage.googleapis.com/zenn-user-upload/sixpzg7t1vaxu7wd41wu8ujaub16)
ここでChefのレシピをどのタイミングで実行したいか設定できます。
今回はEC2インスタンス起動時にWordPressに必要なミドルウェアとWordPressをインストールし、インスタンス起動時にWordPressがインストールできるようにしています。
## Instanceの追加
InstancesメニューからStackに登録するインスタンスを追加します。
![](https://storage.googleapis.com/zenn-user-upload/08f5uht21okjqj42w5zvu93u4lbv)
Newタブでインスタンス名とインスタンスタイプ、配置するサブネットを選択してインスタンスを追加します。
インスタンス追加後、メニュー画面からインスタンスを起動させます。(パブリックIPは隠しています)
![](https://storage.googleapis.com/zenn-user-upload/h5wmt3lyzpi9q1k8xp28t4nsuw1v)
# WordPress起動
http://パブリックIP/wordpress をURLに貼り付けるとWordPressのインストール画面が表示されて無事にWordPressが導入できました。

# 使いにくいと感じた点
OpsWorksを使ってみましたが、事前にGitでcookbookを格納したリポジトリが必要であることが少し手間だと感じます。
これはOpsWorksというよりもChefの思想によるものなので、なんとも言えませんが。。。
またChefは設定変更先にエージェントを導入する必要があり、新規インスタンスの場合は起動時に最初から入れてくれるので楽ですが、既存のインスタンスをOpsWorksに組み込む場合は自分の手でインストールする必要があります。
![](https://storage.googleapis.com/zenn-user-upload/oxalcn0tw1ccxsmkvdjhxfbvvpmf)
エージェントのインストール方法ですが変更先のインスタンスにAWS CLIを導入することが必要です。
サーバーを壊して新しく作り直すたびにエージェントをインストールし直すのは若干手間だと感じました。
エージェントをインストールしたインスタンスからAMIを作成して次回以降はそのAMIから起動するようにすれば多少はマシになりますが、OSを変更した場合また入れ直さないといけないのでどうしたものかと考えます。

もう一つ気になったのが、起動したEC2インスタンスをELBに紐付けることもできるのですがALBは非対応でCLBのみしか対応していないことです。
現在CLBを使う運用事例は少なく、ALBが使えないのは大きなマイナスだと思います。

# 所感
今回はOpsWorksを使って、EC2インスタンスにWordPressを導入しました。
AWS内ですべて完結はできますが、ローカルでAnsibleを使って導入したほうがいい気がしてきました。
Chefそのものもやや下降気味で、cookbookも更新されず放置されているものいくつかありました。
個人的にはOpsWorks for Ansibleが出てくることを願うばかりです。
