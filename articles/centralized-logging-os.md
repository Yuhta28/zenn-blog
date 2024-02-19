---
title: "OpenSearchを使用した統合ログ管理基盤を構築してみた"
emoji: "🪵"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","log","opensearch"]
published: true
---

# 序論

AWSやGoogle Cloud、Azureなどのパブリッククラウドサービスのおかげで自社にて管理、メンテナンスするインフラストラクチャーが大幅に減り、Web/プロダクト開発のスピードが大幅に向上しました。
各種クラウドサービスが提供するマネージドサービスを利用することで、信頼性の高いスケーラブルなアプリケーションを素早くクラウド上で構築できます。

一方、複数のマネージドサービスを利用することでそれぞれのサービス内にログが分散し、障害発生時のトラブルシューティングやセキュリティインシデント発生時の調査対応が複雑化してしまう問題が出てきます。
ログを一か所に集めて集中管理することでインフラ全体の監視、分析、インシデント調査対応を簡素化し運用負荷を下げようと取り組む人は多いと思います。
その一方で散らばっているログを一か所に集めて分析するための統合ログ分析基盤を一から作るのは大変な作業です。

今回紹介するAWSのソリューションライブラリ[^1]にはOpenSearchを使用した統合ログ管理基盤を構築し、AWS上で生成されるサービスログに加えアプリケーションログの取り込み、処理、可視化を実現するソリューションが提供されています。

https://aws.amazon.com/jp/solutions/implementations/centralized-logging-with-opensearch/

このソリューションは比較的新しく日本語ドキュメント化されていない部分も多かったので実装時の注意点を交えてどのような統合ログ管理基盤が構築できたか紹介していきます。


[^1]: https://aws.amazon.com/jp/solutions/

# アーキテクチャ紹介
![](/images/centralized-logging-os/image1.png)
*ソリューションライブラリより引用*

このソリューションライブラリには対象となるAWSサービスログの選定やアプリケーションログをOpenSearchへ取り込むためのパイプラインを簡単に作成してくれるWebベースのコンソール画面が提供されています。

ライブラリにて提供されているCloudFormationをデプロイすることですぐに作成できます。
![](/images/centralized-logging-os/image2.png)
*コンソール画面*

基本的に[実装ガイド](https://docs.aws.amazon.com/ja_jp/solutions/latest/centralized-logging-with-opensearch/solution-overview.html)通りに進めばログ分析パイプラインコンソール画面は作成できますが、OpenSearch自体は自分で作成する必要があります。またカスタムドメインを使って外部からOpenSearchダッシュボードへアクセスできるアクセスプロキシも提供していますがドメインの事前購入も必要です。

字面だけではわかりにくいと思いますので順番に実装ガイドを見ながらユーザーが設定する項目について説明します。

# ソリューション構築

## 事前準備

実装前の準備としてコンソール画面への認証方法を選択する必要があります。
https://docs.aws.amazon.com/ja_jp/solutions/latest/centralized-logging-with-opensearch/automated-deployment.html

Cognito以外にも自前のOpenID Connectを使った認証基盤実装も可能です。

#### ドキュメント記載の対応OIDCクライアント一覧
- Authing[^2]
- Keycloak[^3]
- ADFS[^4]
- Auth0[^5]

準備の手間が少ないのはもちろんCognitoですので、私はCognitoで認証基盤を実装しました。ログ分析パイプラインコンソールを既存VPC上で動かすか、新規にVPCを作成してその上で動かすかで対象となるCloudFormationテンプレートが変わりますが新規にVPCを作成したほうが後述のOpenSearch構築も楽になります。

![](/images/centralized-logging-os/image3.png)
*スタックテンプレート*

このCloudFormationスタックではログイン用のメールアドレス入力欄があるだけですのでログイン時に利用するメールアドレスを入力したらその他はデフォルトのままCloudFormationをデプロイします。
![](/images/centralized-logging-os/image4.png)
*ログイン用メールアドレス入力*

デプロイが完了するとメールアドレス宛に一時パスワードが発行されるので出力結果から`WebConsoleUrl`に記載されているURLへアクセスし、メールアドレスとパスワードを入力します。
![](/images/centralized-logging-os/image5.png)

[^2]: https://www.authing.cn/
[^3]: https://github.com/aws-samples/keycloak-on-aws
[^4]: https://learn.microsoft.com/en-us/windows-server/identity/ad-fs/ad-fs-overview
[^5]: https://auth0.com/

また後述のアクセスプロキシ経由でOpenSearchダッシュボードへアクセスする場合、カスタムドメインとSSL証明書が必要になります。
**Route53やドメインレジストリサイトでドメインを入手し、ACMでSSL証明書を発行しておいてください。**

## OpenSearch構築
次にログ集約用のOpenSearchを構築します。
この辺の構築手順はソリューションライブラリのドキュメントでは省略されていましたのでOpenSearchに不慣れな人はつまずくポイントになると思います。

:::message alert
OpenSearchは高スペックなEC2インスタンスを複数台動かすようなものですので、使い終わったら必ずOpenSearchの削除を忘れないようにしましょう。
:::

OpenSearchのコンソール画面からドメインの作成を選択後、任意のドメイン名を決めます。ドメインの作成方法と開発テンプレートはそれぞれ「標準作成」、「開発/テスト」を選択します。

![](/images/centralized-logging-os/image6.png)

デプロイオプションに関して新規にVPCを作成した人は2個1組のサブネットが作成されていますので、アベイラビリティゾーン2-AZを選択できるようにスタンバイ開発が無効のドメインを選びます。

![](/images/centralized-logging-os/image7.png)
*利用AZは2個なのでデータノードの数も2の倍数に変更*


データノードはノードの数を2の倍数に合わせる以外はデフォルトの設定で問題ありません。
しばらくスクロールするとネットワーク設定の項目がありますが、ここはVPCを指定し、IPアドレスタイプをIPv4のみにします。
CloudFormationで新規VPCの作成したら対応するサブネットとセキュリティグループも一緒に作られており、サブネットはプライベートサブネット、セキュリティグループは`OpenSearchSecurityGroup`と名付けられているものを選択します。

![](/images/centralized-logging-os/image8.png)
*他のサブネットを指定するとOpenSearchダッシュボードへアクセスできないので注意*

きめ細かなアクセスコントロールではOpenSearchダッシュボードへアクセスするマスターユーザーになりますので、有効化しログイン用のマスターユーザーを作成します。
![](/images/centralized-logging-os/image9.png)

合わせてアクセスポリシーも`きめ細かなアクセスコントロールのみを使用`に変更します。
![](/images/centralized-logging-os/image10.png)

残りはデフォルトの設定で問題ありませんので、OpenSearchの作成をクリックします。10分ほどでOpenSearchの作成が完了します。

## OpenSearchドメインのインポート

https://docs.aws.amazon.com/ja_jp/solutions/latest/centralized-logging-with-opensearch/getting-started.html
ここからはソリューションライブラリのドキュメント手順に戻って事前準備で作成したログ分析パイプラインコンソールとOpenSearchの連携を行ないます。
コンソール画面トップから`Import domain`をクリックします。ドロップダウンリストから作成したOpenSearchを選択します。ネットワーク設定は`Automatic`を選択すれば各種必要なネットワーキング設定を自動で実装してくれます。
お好みでタグ付けなど設定したらOpenSearchドメインのインポートを実行します。

## アクセスプロキシ作成

作成したOpenSearchはVPCのプライベートサブネット内に配置されているため通常外からアクセスはできません。ドメイン画面上部の`Access Proxy`を有効化する必要があります。
![](/images/centralized-logging-os/image11.png)
*作成前はEnableと表記されています*

アクセスプロキシ作成画面へ移行したら配置サブネットの指定(表題通りパブリックサブネットを指定)、セキュリティグループを指定します。
セキュリティグループは`ProxySecurityGroup`と名付けられたセキュリティグループを指定します。

ドメインは事前準備で用意したカスタムドメイン、ロードバランサーSSL証明書はACMで発行したSSL証明書を選択します。

作成するとアクセスプロキシ用のAWSリソースを作成するCloudFormationがデプロイされ、関連リソースを作成してくれます。

![](/images/centralized-logging-os/image12.png)
*アクセスプロキシ作成リソース*

アクセスプロキシタブを確認すると設定情報が参照できるようになり、ALBのDNS名をカスタムドメインのCNAMEレコードに登録します。
![](/images/centralized-logging-os/image13.png)
*DNS名をコピー*

![](/images/centralized-logging-os/image14.png)
*サンプル画像はCloudFlare DNS*

カスタムドメインでOpenSearchダッシュボードへアクセスできるようになります。

![](/images/centralized-logging-os/image15.png)

## ログ取り込み

OpenSearchにログを取り込んでいきます。
AWSサービスのログを取り込みたい場合、`AWS Service Log Analytics Pipeline`にチェックを入れてパイプラインの作成を選択します。

![](/images/centralized-logging-os/image16.png)

取り込めるAWSサービスログは図の通りです。ドキュメントではCloudTrailを例に挙げているのでCloudTrailのログを取り込んでみます。

![](/images/centralized-logging-os/image17.png)
*以降のステップはデフォルトでOK*

ログパイプラインの作成も裏でCloudFormationがデプロイされ、ログをOpenSearchへ転送するAWSリソースが作成されます。パイプライン作成時に設定をデフォルト値のまま進めましたらOpenSearchのサンプルダッシュボードもセットで作成されます。

![](/images/centralized-logging-os/image18.png)
*パイプラインテンプレート(すごい複雑)*

## OpenSearchダッシュボードアクセス

初めてOpenSearchへアクセスする際、テナントの選択を聞かれますがGlobalテナントを選択してください。
Globalテナントであることを確認したら左サイドバーからOpenSearch Dashboards > DashboardsでCloudTrailログ用のダッシュボードを確認します。

![](/images/centralized-logging-os/image19.png)
*テナント確認忘れず*

サンプルダッシュボードのおかげでログを取り込むだけですぐに分析結果を可視化できます。

![](/images/centralized-logging-os/image20.png)
*CloudTrailログダッシュボード*

AWSサービスに応じたログ別のサンプルダッシュボードがありますのでログ毎の分析も容易に行なえます。

![](/images/centralized-logging-os/image21.png)
*ALBアクセスログダッシュボード*


## アプリケーションログ取り込み
AWSサービスのログ以外にサーバー内で生成されるログを取り組めるパイプラインも提供されています。
アプリケーションログのログソースはEC2、EKS、S3などが選べます。EC2やEKSの場合、Fluent Bit[^6]を使ってログ転送します。

![](/images/centralized-logging-os/image22.png)
*アプリケーションログ一覧*

EC2内で生成されるログをOpenSearchに取り組んでみます。事前にEC2 Instance Groupを作成する必要があります。

https://docs.aws.amazon.com/ja_jp/solutions/latest/centralized-logging-with-opensearch/log-sources.html#amazon-ec2-instance-group

ドキュメントによると必須要件としてSSMエージェントがインストールされていること、セッションマネージャーでEC2へアクセスできること、OpenSSL1.1がインストールされていることなどが挙げられています。
必須要件を満たしたEC2を用意すればEC2 Instance Group作成画面でFluent BitエージェントがインストールされたEC2が表示されるので対象グループに含めてグループ作成します。

![](/images/centralized-logging-os/image23.png)

またEC2のログをOpenSearchに転送する際、どのようにパースするのか定義するLog Configの設定も必要になります。

![](/images/centralized-logging-os/image24.png)
*Log Configs*

EC2 Instance GroupとLog Configの作成ができましたらアプリケーションログのパイプライン作成からEC2をソース元にし、転送対象ログを選択します。

![](/images/centralized-logging-os/image25.png)
*サンプルにApacheのアクセスログを転送*

ログパイプラインの作成が完了しましたらOpenSearchダッシュボードへアクセスし、アプリケーションログ用のログダッシュボードが作成されていることを確認します。

![](/images/centralized-logging-os/image26.png)
*Apacheアクセスログ用ダッシュボード*


[^6]: https://fluentbit.io/


# 所感
OpenSearchを使った統合ログ管理基盤を構築してみました。AWSにはSIEM on OpenSearch[^7]を使ったログソリューションがあり、以前社内でも検証したことがあります。あちらと比べるとやや機能が物足りない部分がありますが、アプリケーションログのログ分析パイプライン機能をWebコンソールとして提供してくれるのでAWS以外のログの分析をしたいときに便利かなという印象でした。
ドキュメントがややわかりにくい部分があり最初の構築につまずきましたが、一度構築すれば他のログも簡単に取り込めますので皆さんもお試しで実装してみてはいかがでしょうか。


[^7]: https://github.com/aws-samples/siem-on-amazon-opensearch-service/blob/main/README_ja.md

# 参考文献
https://it-trend.jp/log_management/article/kind-of-purpose_and_the-log_of_the-log-management
https://betterstack.com/community/guides/logging/logging-best-practices/