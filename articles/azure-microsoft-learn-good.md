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

機能の比較だけでなく、AWS/GCPからAzureへの移行やAzureでAWSのコスト管理する場合についての方法についても紹介していますので、クラウドマイグレーション、マルチクラウド運用に興味があれば一読の価値があります。

[^1]: https://www.crn.com/news/cloud/top-5-cloud-market-share-leaders-aws-microsoft-google-in-q3-2022/1
[^2]: https://www.publickey1.jp/blog/22/awssiersiersi.html
[^3]: https://explore.skillbuilder.aws/learn
[^4]: レバテックフリーランスで案件数を調べたらAWSが4,231件、Azureが730件、GCPが546件でした(2023/2月時点)
[^5]: https://www.srgresearch.com/articles/as-quarterly-cloud-spending-jumps-to-over-50b-microsoft-looms-larger-in-amazons-rear-mirror
[^6]: https://www.redhat.com/ja/topics/cloud-computing/what-is-multicloud

# 参考文献
https://qiita.com/lazy-kz/items/699ee26b7b6b89c6614d