---
title: "Azureの勉強はMicrosoft Learnがいいよねって話"
emoji: "🐈"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Azure", "microsoft", "初心者"]
published: true
---
# 背景
昨今クラウドサービスを利用することが個人・企業問わず広く使われています。特に有名なのがAWS、Azure、GCPの3大クラウドサービスです。
2022年第3四半期時点のレポートでは、世界シェアの2/3を3つのクラウドサービスを占めているという報告があります。[^1]
より細かくみていくとAWSが34%占めていて圧倒的なシェアを誇っています。Azureが21%と伸びていてGCPが11%です。
私も仕事では普段AWSを触っており、日本国内でもクラスメソッドさん[^2]をはじめとする優秀なAWSパートナー企業が数多く存在し、学習コンテンツの豊富さ[^3]や案件数[^4]の多さなどから最初に勉強するクラウドサービスとしておすすめだと思っています。

ただ別の調査によりますと予測トレンドでAzureがAWSを猛追しているというデータが出ています。

![](/images/azure-microsoft-learn-good/image1.jpg)
*クラウドプロバイダーマーケットシェアトレンド[^5]*

企業規模別のクラウドサービスシェアという調査を見つけることができなかったので主観になりますが、AzureはMicrosoft 365を利用しているエンタープライズ系企業に好まれ、GCPはKubernetesベースのマイクロサービスアーキテクチャを活用する企業で採用、AWSは個人・企業問わず広く採用されるといった印象です。
また最近はマルチクラウド[^6]という言葉があるとおり複数のクラウドサービスを構築して運用する手法も増えてきています。私もAWSばかり学習していると偏りがあると感じて、他のクラウドサービスも触ろうと思いAzureのお勉強を最近はじめてみました。
AzureはAWSと比べて書籍や学習サイトが少なく自学自習が大変かと思われますが、Microsoftが提供しているMicrosoft Learnがとてもよかったので概要や良かった点について紹介していきます。

# Microsoft Learnについて
https://learn.microsoft.com/ja-jp/

Microsoft LearnはMicrosoft製品に関するドキュメントやハンズオントレーニング、認定資格をまとめた**無料**の学習コンテンツサイトになります。はじめてアクセスする時に何から始めたらいいかわからなければMicrosoftの公式YouTube動画を参考にしてみてください。
https://www.youtube.com/watch?v=9usUqNhK7xw

Microsoft製品はAzure、Windowsはもちろん、.NETやJava、JavaScriptなどのプログラミング言語もありますし、GitHubやGitHub Actions、Visual Studioを使ったWeb開発のについて説明もありますので普段Macを使ってWeb開発をしている人も見てみるとよいでしょう。またAWS、GCPを使っている人がAzureとの違いについて比較するための記事も用意されていますので違いについて簡単に知りたい場合は以下の記事をご覧ください。

 - AWSとAzureとの比較
https://learn.microsoft.com/ja-jp/azure/architecture/aws-professional/

- GCPとAzureとの比較
https://learn.microsoft.com/ja-jp/azure/architecture/gcp-professional/

機能の比較だけでなく、AWS/GCPからAzureへの移行やAzureでAWSのコスト管理する場合についての方法についても紹介していますので、クラウドマイグレーションやマルチクラウド運用に興味があれば読んでみることをおススメします。

## おすすめの記事
いくつかおすすめの記事について紹介します。

### Azureアーキテクチャセンター
https://learn.microsoft.com/ja-jp/azure/architecture/
Azureを活用した設計ガイダンス集です。まずはここから入ってAzureの概念や設計原則について学ぶとよいです。
:::message
AWSでいうところのデベロッパーセンター[^7]ですね。
:::

### Microsoft Azure Well-Architected Framework
https://learn.microsoft.com/ja-jp/azure/architecture/framework/

AWS[^8]にもありますがクラウドサービスにはアーキテクチャ設計において指針となる要素があります。
Azureにも5つの柱が紹介されており、高品質で安定した効率的なクラウドアーキテクチャの構築するために大事なことをまとめています。

:::message
実は全部AWSと同じです。(AWSは持続可能性の柱が後から追加され6本ありますが、それ以外の5つはAzureと全く同じです。)
:::

### Azure向けのMicrosoft Cloud導入フレームワーク
https://learn.microsoft.com/ja-jp/azure/cloud-adoption-framework/
ITプロフェッショナルやビジネス意思決定者が自社のシステムをクラウド化するにあたりどのようにして目標を達成するべきか道標となってくれるフレームワークです。
当たり前ですがすべてのシステムがクラウドに適しているわけではありませんので本当にクラウド移行するのがベストかどうかは移行前に入念に確認しておきましょう。
:::message
AWSにもry[^9]
:::

### キャリアパス
https://learn.microsoft.com/ja-jp/training/career-paths/

自分がなりたいキャリアとそのための技術スキルを身につけるためのトレーニングや資格情報をまとめたものになります。クラウド管理者やデータアナリスト、DevOpsエンジニアなどさまざまなキャリアが紹介されていますので、ご自身の興味のあるキャリアについて調べてみるといいでしょう。

# ラーニングパス
https://learn.microsoft.com/ja-jp/training/azure/
ラーニングパスには前述したMicrosoft製品が学べるハンズオンがまとめられています。製品別以外にもロール別、難易度別に検索できますので、ご自身のスキルに応じてフィルタリングしてどこから学ぶかの足がかりにしてもいいでしょう。
![](/images/azure-microsoft-learn-good/image2.jpg)

Azureの資格についても検索できますが、認定資格と試験が分けられていて個人的にわかりにくいと感じています。
### 例 AZ-104試験に合格することでAzure Administrator Associate資格に認定
- AZ-104
https://learn.microsoft.com/ja-jp/certifications/exams/az-104/
- Azure Administrator Associate
https://learn.microsoft.com/ja-jp/certifications/azure-administrator/

# Azure資格
Microsoftの資格はAzure以外にもMicrosoft 365やWindows Serverなど数多くあり、すべての資格がラーニングパスに表示されるのでAzure特化した資格情報ページが中々見つからず分かりづらいと思いました。
![](/images/azure-microsoft-learn-good/image3.jpg)

Azureだけに絞った資格表探しましたが、公式だとこれしか見つからなかったです。
![](/images/azure-microsoft-learn-good/image4.jpg)
試験別の概要についてはこちらの記事をご参照ください。
https://career.levtech.jp/guide/knowhow/article/639/

:::message alert
Azureの試験は頻繁に更新されるので古い記事ですと違うことが書かれていることがあります。最初にMicrosoft公式ページを確認してから解説記事を探したほうが良いです。
:::

AzureがはじめてのクラウドでしたらまずはAzure Fundamentals取得を目標にするのがいいと思います。
https://learn.microsoft.com/ja-jp/certifications/azure-fundamentals/
この資格はMicrosoft Virtual Training Daysで試験代免除のクーポン付きセミナーが定期的に開催されますので、受験する場合はセミナーを受講したほうがいいです。
https://www.microsoft.com/ja-jp/events/top/training-days

他のクラウドサービスを経験済みかつAzureでクラウドアーキテクチャ設計を学びたい場合はAzure Solutions Architect Expertがありますが、この資格は上級資格でかつ前提条件としてAzure Administrator Associate資格保持が条件です。
https://learn.microsoft.com/ja-jp/certifications/azure-solutions-architect/

なので最初はAzure Administrator Associate資格から挑戦してみてもいいと思います。
https://learn.microsoft.com/ja-jp/certifications/azure-administrator/
# Azure Administrator Associate資格
先日AZ-104試験に合格し、Azure Administrator Associateに認定されました。
![](/images/azure-microsoft-learn-good/image5.jpg)
学習に利用したのは先ほど紹介したラーニングパスだけです。AZ-104と検索すればAZ-104の試験範囲を網羅したラーニングパスが出てきますのでこれを一通り触ることで理解を深めることができました。注意点としてすべてのモジュールが日本語化されているわけではないので日本語ページだと達成度が100%になっていないにも関わらず、モジュールが見つからないということがありますので適宜英語で見直しましょう。
https://learn.microsoft.com/ja-jp/training/browse/?terms=AZ-104

![](/images/azure-microsoft-learn-good/image6.jpg)
Microsoftの日本語訳なので不安に思う人もいらっしゃると思いますが、そこまで不自然な日本語はなく学習する分には問題ありませんでした。ラーニングパスはすべて無料にも関わらずAzureやWeb開発向けの学習コンテンツが豊富に存在するので皆さんもぜひとも触ってみてください。

# 30日間で資格取得チャレンジ
MicrosoftはAzureのシェアを伸ばしたい戦略の一環なのかAzure資格の受験料クーポンを用意しています。
30 Days to Learn Itでは30日間以内に決められたモジュールをすべて完了すれば50%割引のクーポンチケットがもらえます。
https://developer.microsoft.com/ja-jp/offers/30-days-to-learn-it

昨年はMicrosoft Ignite開催を記念してエキスパートレベルの資格試験が無料で受験できるクーポンが手に入るThe Microsoft Learn Cloud Skills Challengeが用意されていました。
https://www.microsoft.com/ja-jp/cloudskillschallenge/ignite/registration/2022

また一度落ちても再受験が無料で受験できるExam Replyというのものあります。
https://www.microsoft.com/ja-jp/learning/local-exam-replay.aspx

お試しで受験して2回目に本気を出すということもできます。

# 所感
Microsoft Learnについて紹介しました。
無料でここまで学べるのはさすがMicrosoftだと思いますので、Azureに限らずGitHubやC#などを学びたいはこちらから手に取ってみるのが一番だと思います。AzureにはAWSとは違った特徴もありますので次回はAzureならではの機能について書こうかと思います。

[^1]: https://www.crn.com/news/cloud/top-5-cloud-market-share-leaders-aws-microsoft-google-in-q3-2022/1
[^2]: https://www.publickey1.jp/blog/22/awssiersiersi.html
[^3]: https://explore.skillbuilder.aws/learn
[^4]: レバテックフリーランスで案件数を調べたらAWSが4,231件、Azureが730件、GCPが546件でした(2023/1月時点)
[^5]: https://www.srgresearch.com/articles/as-quarterly-cloud-spending-jumps-to-over-50b-microsoft-looms-larger-in-amazons-rear-mirror
[^6]: https://www.redhat.com/ja/topics/cloud-computing/what-is-multicloud
[^7]: https://aws.amazon.com/jp/developer/
[^8]: https://aws.amazon.com/jp/architecture/well-architected/?wa-lens-whitepapers.sort-by=item.additionalFields.sortDate&wa-lens-whitepapers.sort-order=desc&wa-guidance-whitepapers.sort-by=item.additionalFields.sortDate&wa-guidance-whitepapers.sort-order=desc
[^9]: https://aws.amazon.com/jp/professional-services/CAF/
# 参考文献
https://qiita.com/lazy-kz/items/699ee26b7b6b89c6614d
https://qiita.com/lazy-kz/items/7dc9a61ddfa5b2a81632
https://www.whizlabs.com/blog/azure-certifications-path/