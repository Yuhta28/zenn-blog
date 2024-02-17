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

# 参考文献
https://it-trend.jp/log_management/article/kind-of-purpose_and_the-log_of_the-log-management
https://betterstack.com/community/guides/logging/logging-best-practices/