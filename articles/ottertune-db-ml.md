---
title: "MLでDBをイイ感じにしてくれるOtterTuneを使ってみた"
emoji: "🦦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["database","ottertune","RDS"]
published: false
---

# 概要
先日ラクスさんが主催されるLTイベントに参加しました。
https://rakus.connpass.com/event/253656/

登壇者の[Hank Ehly](https://connpass.com/user/hankehly/)さんが発表したサービスが面白そうでしたので登録してみた感想を述べていきます。

https://speakerdeck.com/hankehly/ottertune-mldedbwozui-shi-hua-suruturufalseshao-jie


# DBのパフォーマンスチューニングは難しい

皆さんはDBは何を使われていますか？企業さん含めて今は自前でDBを持つよりもAWS RDSに代表されるマネージドなデータベースを利用されているかと思います。
DBのバージョンアップやバックアップの取得を自動で実行してくれて、パラメーターの設定もコンソールから簡単にできるのが魅力的です。
![](/images/ottertune-db-ml/image1.png)
*RDSのパラメータ設定画面*
ですが、そもそもDBのパラメーターチューニング作業が中々ハードルが高くデフォルト設定からの変更となりますとDBの専門的な知識が必要になり、またその設定変更がうまくいくかどうかは設定後の結果反映次第なため、本番環境になにかしらの影響を与えてしまうという問題点があります。

#### DBパフォーマンスチューニングの難しさ

- DBの専門的知識
- 設定変更後の本番影響の予測

結果として下手に設定値をいじるよりかはよりスペックの高いDBにアップグレードするというマネーで殴る💸対応にしがちです。

その問題点を解決するのが今回紹介するOtterTuneになります。

# OtterTune

https://ottertune.com/

OtterTuneは機械学習を利用して、DBの利用状況を分析し最適なパラメーターを提案し、改善してくれるSaaSプロダクトです。
2022年8月時点で対応しているDBはAWSのRDSしかないためまだ限定的なサービスです。

| Database System | Provider | Versions |
| ----       | ---- | ----              | 
| PostgreSQL | Amazon RDS | v10, v11, v12, v13, v14 |
| PostgreSQL | Amazon Aurora | v10, v11, v12, v13 |
| MySQL      | Amazon RDS    | v5.6, v5.7, v8.0 |
| MySQL      | Amazon Aurora | v5.6, v5.7 |

https://docs.ottertune.com/

# 使ってみる
OtterTuneはエージェントをRDSに導入し、メトリクスを取得することでパラメーターを機械学習して最適化します。
サインアップ後に最初にやることはAWSのIAMロールを作成し、自身のAWSアカウントとOtterTuneを連携するところから始めます。
![](/images/ottertune-db-ml/image2.png)
*IAMロール作成の案内*

CloudFormationから作る方法とTerraformから作る方法が公式から提供されていますので好みのツールでIAMロールを作成します。
![](/images/ottertune-db-ml/image3.png)
*CloudFormation作成画面*

IMAロール作成後セットアップ画面にIAMロールのARNを入力すればOtterTuneと自身のAWSアカウントの連携が始まり、ダッシュボードに稼働しているAWS RDSの一覧が表示されます。
![](/images/ottertune-db-ml/image4.png)
*ottertune ダッシュボード*

RDS一覧からottertuneと連携したいDBを有効化し、エージェントをDBにデプロイします。
DBへのデプロイ方法ですが、いくつか方法が紹介されていてもっとも簡単なのがCloudFormation経由でのFargateによるデプロイです。
https://docs.ottertune.com/info/connect-your-database-to-ottertune/add-database/agent
セットアップ画面にAPIキーが出力されていますのでコピーしてCloudFormationのスタックに張り付けて実行すればエージェントがDBにデプロイされます。
![](/images/ottertune-db-ml/image5.png)