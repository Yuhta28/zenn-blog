---
title: "EFSのファイル管理をGUIで管理するAWSソリューションを使ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","efs"]
published: false
---

# 概要
AWSは[ソリューションライブラリ](https://aws.amazon.com/jp/solutions/)とよばれるビジネス課題、技術的課題を解決するためのテンプレートソリューションを数多く提供しています。

そのなかでEFSのファイル管理をGUIから管理できる「Simple File Manager for Amazon EFS」というものを見つけました。
https://aws.amazon.com/jp/solutions/implementations/simple-file-manager-for-amazon-efs/

![](/images/simple-filemanager-efs/image1.png)
*アーキテクチャ図(上記サイトより引用)*

通常EFSにあるファイル管理はマウントされたEC2を経由してファイルコピーや削除をする必要があります。このソリューションではCloudFrontをフロントに配置することで、Web UIを提供し任意のデバイスから自由にEFSへアクセスできるようにします。

# 実装
実装ガイドに飛んでステップ1のスタック起動からソリューション起動をクリックしますとCloudFormationのスタックテンプレートが起動します。
![](/images/simple-filemanager-efs/image3.png)
(起動リージョンがバージニアになっていますので必要に応じて東京リージョンに変えます)
パラメーターにはSimple File Managerへログインするためのメールアドレスを記載します。
![](/images/simple-filemanager-efs/image4.png)

後はデフォルトのまま次へを選択し、CloudFormationを作成します。
ネストされて複数のスタックがデプロイされるので15分間程で完了します。
記載したメールアドレスに初期パスワードが添付されたメールが届いていますので控えておきます。
CloudFormationの出力タブにWeb UIのログイン画面へのURLが表示されています。
![](/images/simple-filemanager-efs/image5.png)
ログイン画面でメールアドレスとパスワードを入力すれば作成されたEFSのリストが表示されます。
![](/images/simple-filemanager-efs/image6.png)

# 使い方
新規に作成したEFSのManagedステータスはfalseになっています。Web UIからファイル管理できるようにするためにTrueに変更します。false部分をクリックし、管理したいルートディレクトリを指定すればsubmitします。(UserIDとGroupIDはEFS側でよっぼど変な設定をしていない限りデフォルトで大丈夫です)
![](/images/simple-filemanager-efs/image7.png)
ManagedのステータスがTrueに変更されたらWeb UI経由でEFS内のファイルを管理できるようになります。EFS IDをクリックすればEFSディレクトリ構造を確認できます。
![](/images/simple-filemanager-efs/image8.png)
これでWeb UI経由でファイルのアップロード、ダウンロード、EFSのディレクトリの変更ができます。
# 参考資料
https://d1.awsstatic.com/Solutions/ja_JP/simple-file-manager-for-amazon-efs.pdf