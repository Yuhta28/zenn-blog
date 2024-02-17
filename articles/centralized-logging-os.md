---
title: "OpenSearchを使用した統合ログ管理基盤を構築してみた"
emoji: "🪵"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","log","opensearch"]
published: false
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

コンソール画面が表示されましたら事前準備完了です。

## OpenSearch構築
次にログ集約用のOpenSearchを構築します。
この辺の構築手順はソリューションライブラリのドキュメントでは省略されていましたのでOpenSearchに不慣れな人はつまずくポイントになると思います。


# 参考文献
https://it-trend.jp/log_management/article/kind-of-purpose_and_the-log_of_the-log-management
https://betterstack.com/community/guides/logging/logging-best-practices/