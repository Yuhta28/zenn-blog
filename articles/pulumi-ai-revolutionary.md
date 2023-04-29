---
title: "PulumiはIaCの革命児になれるか"
emoji: "☠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","ai","aws"]
published: true
---

# 概要
2023年4月にPulumi社はPulumiサービスにAIを組み込んだ機能を実装したと発表しました。
https://twitter.com/PulumiCorp/status/1646530231639867392?s=20

ここ最近、コード生成系AIが増えてきましたがIaCに特化したツールは珍しいと思いましたので触って感想を述べていきます。

# Pulumiとは
まずPulumiについて説明いたします。
Pulumiはプログラミング言語でインフラストラクチャーを作成できるプロビジョニングツールです。
プロビジョニングツールで有名なものですとTerraform[^1]やAWS CDK[^2]があり比較するとPulumiはやや知名度が低いです。
![](/images/pulumi-ai-revolutionary/image1.png)
*Googleトレンドより*

## 類似ツール比較
### VS Terraform
類似ツールとの違いについて[ドキュメント](https://www.pulumi.com/docs/intro/vs/)で紹介しています。Terraformとの主な違いについては以下の通りです。[^3]

| 項目 | Pulumi | Terraform |
| --- | --- | --- |
| 言語 | 汎用言語（Python, TypeScript, Goなど） | 独自言語（HCL） |
| インフラ再利用 | 関数、クラス、パッケージなど再利用可 | モジュールのみ再利用可 |
| テスト | ユニットテスト、プロパティテストなど一般的なテストフレームワークをサポート | 結合テストのみ |
| 他ツールからのインポート | 可 | 不可 |

Terraformの最大の障壁となるのがHCL(HashiCorp Language)と呼ばれる独自言語の学習だと考えています。開発者は普段自分が親しんでいるプログラミング言語でコードを書きたいという需要がありますが、2023年4月現在以下の言語をサポートしています。[^4]

- TypeScript/JavaScript(Node.js)
- Python
- Go
- .NET
- Java
- YAML

またPulumiはすでに他のツールで構築されているIaCソースコードをPulumiのソースコードに変換してくれるツールを提供してくれます。Terraformから移行する場合は`tf2pulumi`とよばれるマイグレーションツールがあります。
https://github.com/pulumi/tf2pulumi

### VS AWS CDK
プログラミング言語でインフラストラクチャーを構成するIaCツールといえばAWS CDKがあります。AWS CDKとの違いについてPUlumiではこのように述べています。

| 項目 | Pulumi | AWS CDK |
| --- | --- | --- |
| サポート対象 | AWS以外にも60以上のクラウドやSaaSをサポート | AWSのみ |
| デプロイエンジン | Pulumi CLI（オープンソース） | AWS CloudFormation（プロプライエタリ） |
| テスト | ユニットテスト、プロパティテストなど一般的なテストフレームワークをサポート | 合成されたYAML/JSONに対してアサートするユニットテストのみ |
| 既存リソースインポート | 可 | プレビュー対応 |

AWS CDKはAWSに対応したプロビジョニングツールなので他のクラウドサービスには対応していないことが大きな違いです。またAWS CDKは既存リソースに対するインポート機能を正式にまだリリースしていません。
:::message alert
CDK for Terraform(CDKTF)[^5]がGAされたのでこの問題は改善されたかもしれません。まだ触れていないのでいつか検証します。
:::

機密情報の保持についてはSecrets Manager[^6]を使って格納するようにしていますが、Pulumiはマネージドなクラウドサービスが提供する暗号化キーを使い、暗号化し保護します。

[^1]: https://www.terraform.io/
[^2]: https://aws.amazon.com/jp/cdk/
[^3]: https://www.pulumi.com/docs/intro/vs/terraform/#dynamic-providers
[^4]:https://www.pulumi.com/docs/intro/languages/
[^5]:https://developer.hashicorp.com/terraform/cdktf
[^6]:https://aws.amazon.com/jp/secrets-manager/


# Pulumi AIとは
https://www.pulumi.com/ai/

大規模言語モデルのGPTを活用して、自然言語でPulumiのコードを自動生成してくれるAIアシスタントツールになります。無料で使えユーザー登録も不要で上のURLから作りたい構成を質問すればPulumiのソースコードが生成されます。GPTはv3とv4の2つが選択できるので無料でV4を使えるというのはChatGPTよりも優位性があるのではないかと思います。
![](/images/pulumi-ai-revolutionary/image9.png)

## 使い方
まずPulumiをローカルにインストールします。OS毎にインストーラーが用意されているので、インストールは難しくないと思います。
https://www.pulumi.com/docs/get-started/install/

次にPulumiを実行する作業ディレクトリを作成し、実行プロジェクトを作成します。ここではTypeScriptを使ったプロジェクトを作成します。

```terminal
$ pulumi new typescript
This command will walk you through creating a new Pulumi project.

Enter a value or leave blank to accept the (default), and press <ENTER>.
Press ^C at any time to quit.

project name: (pulumi-tutorial)
project description: (A minimal TypeScript Pulumi program)
Created project 'pulumi-tutorial'

Please enter your desired stack name.
To create a stack in an organization, use the format <org-name>/<stack-name> (e.g. `acmecorp/dev`).
stack name: (dev)
Created stack 'dev'

Installing dependencies...


added 182 packages, and audited 183 packages in 28s

56 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
npm notice
npm notice New minor version of npm available! 9.2.0 -> 9.6.5
npm notice Changelog: <https://github.com/npm/cli/releases/tag/v9.6.5>
npm notice Run `npm install -g npm@9.6.5` to update!
npm notice
Finished installing dependencies

Your new project is ready to go!

To perform an initial deployment, run `pulumi up`
```
プロジェクトを作成するとPulumiを実行するために必要なテンプレートファイルが作成されます。
![](/images/pulumi-ai-revolutionary/image2.png)

準備ができましたのでPulumi AIを使ってソースコードを生成していきます。
超シンプルな静的ホスティングサイトを公開するS3バケットを生成してもらいました。
![](/images/pulumi-ai-revolutionary/image3.png)

### 静的ホスティングS3バケット

AIで生成されたコードがこちらです。
```typescript:index.ts
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// S3バケットの作成
const websiteBucket = new aws.s3.BucketV2("websiteBucket", {});

// 静的ウェブサイト構成を設定
const websiteConfiguration = new aws.s3.BucketWebsiteConfigurationV2("websiteConfiguration", {
  bucket: websiteBucket.id,
  indexDocument: {
    suffix: "index.html",
  },
  errorDocument: {
    key: "error.html",
  },
});

// スタック出力
export const bucketName = websiteBucket.id;
export const websiteEndpoint = pulumi.interpolate`http://${websiteBucket.websiteEndpoint}`;
```

コードが生成された`pulumi up`でデプロイされS3が作成されます。
S3が作成されましたので後はindex.htmlファイルをS3にアップロードし、エンドポイントURLへアクセスしましたが403エラーが表示されました。バケットポリシーに何も記述がなく公開許可されていなかったことが原因でした。
![](/images/pulumi-ai-revolutionary/image4.png)

Pulumi AIはチャット形式で前の文章から引き続きコードの修正を命令できます。
![](/images/pulumi-ai-revolutionary/image6.png)

修正されたコードが生成されたのでもう一度デプロイしましたが、バケットポリシーの作成がうまくいきませんでした。
![](/images/pulumi-ai-revolutionary/image7.png)

一旦手作業でバケットポリシーを修正してバケットエンドポイントURLにアクセスすると静的サイトが表示されることを確認できました。
![](/images/pulumi-ai-revolutionary/image5.png)

やや消化不良気味でしたので今度は新規VPCの中にEC2、RDS、ALBを作成する基本的なLAMP構成のインフラストラクチャーを構築するように命令しました。

### LAMPアーキテクチャー

```typescript:index.ts
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// Create a new VPC
const vpc = new aws.ec2.Vpc("my-vpc", {
    cidrBlock: "10.0.0.0/16",
});

// Create an Internet Gateway
const internetGateway = new aws.ec2.InternetGateway("internetGateway", {
  vpcId: vpc.id,
});

// Attach the Internet Gateway to the VPC
const vpcGatewayAttachment = new aws.ec2.InternetGatewayAttachment("InternetGatewayAttachment", {
  vpcId: vpc.id,
  internetGatewayId: internetGateway.id,
});

// Create two subnets in different Availability Zones (AZs)
const subnet1 = new aws.ec2.Subnet("subnet-1", {
    cidrBlock: "10.0.1.0/24",
    availabilityZone: "ap-northeast-1a",
    vpcId: vpc.id,
});

const subnet2 = new aws.ec2.Subnet("subnet-2", {
    cidrBlock: "10.0.2.0/24",
    availabilityZone: "ap-northeast-1c",
    vpcId: vpc.id,
});

// Create a DB Subnet Group for the RDS instance
const dbSubnetGroup = new aws.rds.SubnetGroup("db-subnet-group", {
    subnetIds: [subnet1.id, subnet2.id],
});

// Create an EC2 instance with a LAMP stack in the first subnet
const userData = 
    `#!/bin/bash
     sudo yum -y update
     sudo yum -y install httpd mysql php
     sudo systemctl enable httpd
     sudo systemctl start httpd`;

const ec2SecurityGroup = new aws.ec2.SecurityGroup("ec2SG", {
    vpcId:vpc.id,
    ingress: [{
        description: "default",
        fromPort: 443,
        toPort: 443,
        protocol: "tcp",
        cidrBlocks: [vpc.cidrBlock],
    }]
})

const ec2Instance = new aws.ec2.Instance("web-server", {
    ami: "ami-01b32aa8589df6208", // Replace this with the ID of the Linux (Amazon Linux 2) AMI in your region
    instanceType: "t2.micro",
    subnetId: subnet1.id,
    userData: userData,
    vpcSecurityGroupIds: [ec2SecurityGroup.id],
});

const RDSSecurityGroup = new aws.ec2.SecurityGroup("RDSSG", {
    vpcId:vpc.id,
    ingress: [{
        description: "default",
        fromPort: 3306,
        toPort: 3306,
        protocol: "tcp",
        cidrBlocks: [vpc.cidrBlock],
    }]
})

// Create an RDS instance in the DB subnet group
const rdsInstance = new aws.rds.Instance("db-server", {
    engine: "mysql",
    instanceClass: "db.t2.micro",
    allocatedStorage: 20,
    name: "mydb",
    username: "admin",
    password: "yourd4t4b4sepa45w0rd",
    dbSubnetGroupName: dbSubnetGroup.name,
    vpcSecurityGroupIds: [RDSSecurityGroup.id],
});


const ALBSecurityGroup = new aws.ec2.SecurityGroup("ALBSG", {
    vpcId:vpc.id,
    ingress: [{
        description: "default",
        fromPort: 80,
        toPort: 80,
        protocol: "tcp",
        cidrBlocks: [vpc.cidrBlock],
    }]
})

// Create an ALB in VPC
const alb = new aws.lb.LoadBalancer("my-alb", {
    securityGroups: [ALBSecurityGroup.id],
    subnets: [subnet1.id, subnet2.id],
});

// Create a Target Group for instances
const targetGroup = new aws.lb.TargetGroup("target-group", {
    port: 80,
    protocol: "HTTP",
    targetType: "instance",
    vpcId: vpc.id,
});

// Create a Listener for ALB
const listener = new aws.lb.Listener("listener", {
    loadBalancerArn: alb.arn,
    port: 80,
    protocol: "HTTP",
    defaultActions: [{
        type: "forward",
        targetGroupArn: targetGroup.arn,
    }],
});

// Export the ALB DNS name
export const albDns = alb.dnsName;
```

こちらも一発で作成することができず、セキュリティグループやインターネットゲートウェイの作成を追加することでリソースの作成に成功しました。
中々一発でうまくいかず少なくともAIで生成されたコードだけでは実践で使うことは難しいように思えます。

# 所感
Pulumi AIを使ってPulumiでIaCを体験してみました。Pulumi AIですが少し目を離すと接続が途切れてしまいスレッドが最初からやり直しになることが多々ありました。
![](/images/pulumi-ai-revolutionary/image8.png)

まだ使い勝手は難しいと感じましたがPulumi公式でハンズオンがありましたのでもう少しこちらを使って理解を深めてみようと思います。
https://www.pulumi.com/challenge/ai-architecture/

やや挑戦的なタイトルになってしまいましたが、個人的にはもう少し機能拡張に期待したいなと思いました。
PulumiはTerraformやAWS CDKと比べると知名度が弱い気がしますが、2つの良いところを取り入れてる部分もありますので、気になりましたら今回のアップデートを機にIaCツールの採用にPulumiを検討してみてください。

# 参考文献
https://www.pulumi.com/blog/pulumi-ai