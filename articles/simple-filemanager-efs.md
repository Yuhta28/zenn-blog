---
title: "EFSのファイル管理をGUIで管理するAWSソリューションを使ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","efs"]
published: false
---

# 概要
AWSは[ソリューションライブラリ](https://aws.amazon.com/jp/solutions/)とよばれるビジネス課題、技術的課題を解決するためのテンプレートソリューションを数多く提供しています。

その中でEFSのファイル管理をGUIから管理できる「Simple File Manager for Amazon EFS」というものを見つけました。
https://aws.amazon.com/jp/solutions/implementations/simple-file-manager-for-amazon-efs/

![](/images/simple-filemanager-efs/image1.png)
*アーキテクチャ図(上記サイトより引用)*

通常EFSにあるファイル管理はマウントされたEC2を経由してファイルコピーや削除をする必要があります。このソリューションではCloudFrontをフロントに配置することで、web UIを提供し任意のデバイスから自由にEFSへアクセスできるようにします。

# 実装
実装ガイドに飛んでステップ1のスタック起動からソリューション起動をクリックしますとCloudFormationのスタックテンプレートが起動します。
![](/images/simple-filemanager-efs/image3.png)
(起動リージョンがデフォルトでバージニアになっていますので東京リージョンに変えても問題ありません)
パラメーターのメールアドレスではSimple File Managerへログインするためのメールアドレスを記載します。
![](/images/simple-filemanager-efs/image4.png)
# 参考資料
https://d1.awsstatic.com/Solutions/ja_JP/simple-file-manager-for-amazon-efs.pdf