---
title: "Azureの勉強はMicrosoft Learnがいいよねって話"
emoji: "🐈"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Azure", "microsoft", "初心者"]
published: false
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

Microsoft LearnはMicrosoft製品に関するドキュメントやハンズオントレーニング、認定資格をまとめた学習コンテンツサイトになります。
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
Azureを活用した設計ガイダンス集です。まずはここから入ってAzureの概念や設計原則について学習すると良いと思います。
:::message
AWSでいうところのデベロッパーセンター[^7]ですね
:::

### Microsoft Azure Well-Architected Framework
https://learn.microsoft.com/ja-jp/azure/architecture/framework/

AWS[^8]にもありますがクラウドサービスにはアーキテクチャ設計において指針となる要素があります。
Azureにも5つの柱が紹介されており、高品質で安定した効率的なクラウドアーキテクチャの構築するために大事なことをまとめています。

:::message
実は全部AWSと同じです。(AWSは持続可能性の柱が後から追加され6本ありますが、それ以外の5つはAzureと全く同じです)
:::

### Azure向けのMicrosoft Cloud導入フレームワーク
https://learn.microsoft.com/ja-jp/azure/cloud-adoption-framework/
ITプロフェッショナルやビジネス意思決定者が自社のシステムをクラウド化するにあたりどのように目標を達成するべきか道標となってくれるフレームワークです。
当たり前ですがすべてのシステムがクラウドに適しているわけではありませんので本当にクラウド移行するのがベストかどうかは移行前に入念に確認しておきましょう。
:::message
AWSにも(ry[^9]
:::

### キャリアパス
https://learn.microsoft.com/ja-jp/training/career-paths/

自分がなりたいキャリアとそのための技術スキルを身につけるためのトレーニングや資格情報をまとめたものになります。クラウド管理者やデータアナリスト、DevOpsエンジニアなど様々なキャリアが紹介されていますので、ご自身の興味のあるキャリアについて調べてみるといいでしょう。

# ラーニングパス
https://learn.microsoft.com/ja-jp/training/azure/
ラーニングパスには前述したMicrosoft製品が学べるハンズオンがまとめられています。製品別以外にもロール別、難易度別に検索できますので、ご自身のスキルに応じてフィルタリングしてどこから学ぶかの足がかりにしてもいいでしょう。
![](/images/azure-microsoft-learn-good/image2.jpg)

Azureの資格についても検索できますが、認定資格と試験が分けられていて個人的にわかりにくいとは思っています。
### 例 AZ-104試験に合格することでAzure Administrator Associate資格に認定
- AZ-104
https://learn.microsoft.com/ja-jp/certifications/exams/az-104/
- Azure Administrator Associate
https://learn.microsoft.com/ja-jp/certifications/azure-administrator/

# Azure資格
Microsoftの資格はAzure以外にもMicrosoft 365やWindows Serverなど数多くあり、すべての資格がラーニングパスに表示されるのでAzure特化した資格情報ページが中々見つからず分かりづらいです。
![](/images/azure-microsoft-learn-good/image3.jpg)

Azureだけに絞った資格表探しましたが、公式だとこれしか見つからなかったです。
![](/images/azure-microsoft-learn-good/image4.jpg)
試験別の概要についてはこちらの記事をご参照ください。
https://career.levtech.jp/guide/knowhow/article/639/

:::message alert
Azureの試験は頻繁に更新されるので古い記事ですと齟齬が出てきます。最初にMicrosoft公式ページを確認してから解説記事を探したほうが良いです。
:::

AzureがはじめてのクラウドでしたらまずはAzure Fundamentals取得を目標にするのがいいと思います。
https://learn.microsoft.com/ja-jp/certifications/azure-fundamentals/
この資格はMicrosoft Virtual Training Daysで試験代免除のクーポン付きセミナーが定期的に開催されますので、受験する場合はセミナーを受講したほうがいいです。
https://www.microsoft.com/ja-jp/events/top/training-days

他のクラウドサービスを経験済みかつAzureでクラウドアーキテクチャ設計を学びたい場合はAzure Solutions Architect Expertがありますが、このこの資格は上級資格でかつ前提条件としてAzure Administrator Associate資格保持が条件です。
https://learn.microsoft.com/ja-jp/certifications/azure-solutions-architect/

なので最初はAzure Administrator Associate資格から挑戦してみてもいいと思います。
https://learn.microsoft.com/ja-jp/certifications/azure-administrator/
# Azure Administrator Associate資格
先日AZ-104試験に合格し、Azure Administrator Associateに認定されました。
![](/images/azure-microsoft-learn-good/image5.jpg)
学習に利用したのは先ほど紹介したラーニングパスだけです。AZ-104と検索すればAZ-104の試験範囲を網羅したラーニングパスが出てきますのでこれを一通り触ることで理解を深めることができました。
https://learn.microsoft.com/ja-jp/training/browse/?terms=AZ-104

# 30日間で資格取得チャレンジ


[^1]: https://www.crn.com/news/cloud/top-5-cloud-market-share-leaders-aws-microsoft-google-in-q3-2022/1
[^2]: https://www.publickey1.jp/blog/22/awssiersiersi.html
[^3]: https://explore.skillbuilder.aws/learn
[^4]: レバテックフリーランスで案件数を調べたらAWSが4,231件、Azureが730件、GCPが546件でした(2023/2月時点)
[^5]: https://www.srgresearch.com/articles/as-quarterly-cloud-spending-jumps-to-over-50b-microsoft-looms-larger-in-amazons-rear-mirror
[^6]: https://www.redhat.com/ja/topics/cloud-computing/what-is-multicloud
[^7]: https://aws.amazon.com/jp/developer/
[^8]: https://aws.amazon.com/jp/architecture/well-architected/?wa-lens-whitepapers.sort-by=item.additionalFields.sortDate&wa-lens-whitepapers.sort-order=desc&wa-guidance-whitepapers.sort-by=item.additionalFields.sortDate&wa-guidance-whitepapers.sort-order=desc
[^9]: https://aws.amazon.com/jp/professional-services/CAF/
# 参考文献
https://qiita.com/lazy-kz/items/699ee26b7b6b89c6614d
https://www.whizlabs.com/blog/azure-certifications-path/