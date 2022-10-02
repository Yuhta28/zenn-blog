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

## コスト
想定されるコストは以下のリソース使用量が挙げられます。
- API Gatewayリクエスト数
- Lambdaリクエスト数
- CloudFrontからの通信トラフィック
- Cognitoのアクティブユーザー数

全部合わせても月の合計コストは約$1くらいです。

:::message alert
当然ながらEFSの利用料は別途かかりますのでお忘れなく
:::

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
![](/images/simple-filemanager-efs/image9.png)

# 改良
このままでもすぐに実践に使えるソリューションですが、より使いやすくするように改良していきます。

## 独自ドメイン追加
作成されたWeb UIのドメインはCloudFrontが生成したドメインになります。
![](/images/simple-filemanager-efs/image2.png)

視認性も悪いので、独自ドメインを追加しましょう。CFnで作成したCloudFrontをコンソールから編集します。
![](/images/simple-filemanager-efs/image10.png)

:::message
ACMを東京リージョンで作成する人は多いと思いますが、他のAWSリソースにインポートする場合バージニアリージョンのACMが必要になるケースがありますので、バージニアリージョンでも所有するドメインのカスタムSSL証明書を発行したほうがいいです。
:::

Route 53で取得したドメインでしたら、Route 53のホストのレコードに設定したドメインとCloudFrontの追加します。
![](/images/simple-filemanager-efs/image11.png)

レコード情報が更新されましたら設定したドメインでSimple File Managerにアクセスできるようになります。
![](/images/simple-filemanager-efs/image12.png)

## できなかったこと
Simple File Managerへのログイン方法がメールアドレス認証しかなかったので、Cognitoの設定を変更してGoogle認証を追加できないか検証しました。
![](/images/simple-filemanager-efs/image13.png)

ただCognito側でGoogleのIDプロバイダを追加してもログイン画面にGoogleログインのポップアップが追加されませんでした。
ログイン画面のフロントデザインはS3から取得しているので、ここのソースコードを修正できれば実装できる気がしますが、よくわからず断念。。。

# 所感
Simple File Managerについて紹介して独自ドメインを実装して多少改良してみました。
EFSへファイルを送る場合、EC2からマウントしてFTPで転送する必要がありましたが、Web UIがあることで非エンジニアにも簡単にファイルアップロード、共有ができるようになり便利になったかと思います。ログイン方法がメールアドレス認証だけだと運用コストが高くなりますので、何とかしてGoogle認証できるようにしたいと考えています。


# 参考資料
https://d1.awsstatic.com/Solutions/ja_JP/simple-file-manager-for-amazon-efs.pdf