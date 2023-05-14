---
title: "Azure Export for Terraformで既存リソースを楽々IaC化"
emoji: "🐈‍⬛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["azure", "terraform"]
published: true
---

# 概要
過去何回かAWSの既存リソースをIaC管理する記事を書いたことがあります。
https://zenn.dev/yuta28/articles/iac-existing-infrastructure
https://zenn.dev/yuta28/articles/challenge-for-existing-iac

記事の総括で既存リソースをIaC化することは難しいと述べました。
というのもTerraformは複数リソースを一括でインポートする機能がなく、リソースに対して1回ずつ`terraform import ~`を実施しなければならず手間がかかるからです。

私のリポジトリからの引用ですが、コンソールからVPCを作成する際に自動的に作成されるサブネットやインターネットゲートウェイ、ルートテーブルなども全部Terraformに落とし込むとするとこのように数が増え手間と労力がかかります。
https://github.com/Yuhta28/terraform-on-AWS/blob/main/modules/vpc/network.tf

Azureも同じ苦労を持っているのかと思いましたが、Azureにはリソースグループ単位で既存リソースをTerraform配下にエクスポートしてくれる**Azure Export for Terraform**というOSSツールがありました。
このツールを使ってAzureのリソースをTerraform配下に管理する方法について紹介します。

# Azure Export for Terraformについて
https://learn.microsoft.com/ja-jp/azure/developer/terraform/overview
> Azure Export for Terraform は、Azure と Terraform の概念間の翻訳の摩擦を軽減するために設計されたツールです。

Azure Export for Terraformは次の3つの方法でリソースをTerraformにエクスポートできます。
| タスク | 説明 | 例示 |
| --- | --- | --- |
| single resource | 1つのリソースをエクスポート | `aztfexport resource <リソースid>` |
| resource group  | リソースグループ単位でエクスポート | `aztfexport resource-group <リソースグループ名>` |
| query | Azure Resource Graph[^1]クエリを使用してエクスポート | `aztfexport query <ARG where predicate>` |


Azureは管理グループの下にサブスクリプションがあり、サブスクリプションの中に複数のリソースグループ、そしてリソースグループの中にリソースが含まれます。
![](/images/azure-export-terraform/image1.png)
*関係図[^2]*

リソースグループはWeb層、DB層といったいわゆるn層アーキテクチャー[^3]別に分離して分けることが多いと思います。
リソースグループ別にエクスポートすることでTerraformのディレクトリ構成がスッキリするメリットもありますのでこちらを使ってAzureの既存リソースをIaC化してみます。

[^1]: https://learn.microsoft.com/ja-jp/azure/governance/resource-graph/overview
[^2]: https://learn.microsoft.com/ja-jp/azure/cloud-adoption-framework/ready/azure-setup-guide/organize-resources
[^3]: https://learn.microsoft.com/ja-jp/azure/architecture/guide/architecture-styles/n-tier

## インストール方法
Azure Export for TerraformはGo製のツールですのでGoコマンドでインストールできます。
```terminal:Go toolchain
go install github.com/Azure/aztfexport@latest
```
他にもWindows、Mac、Linux別にパッケージマネージャーからインストールも可能です。

```terminal:Windows
scoop install aztfexport
```

```terminal:Mac
brew install aztfexport
```

```terminal:Linux
# dnf package
rpm --import https://packages.microsoft.com/keys/microsoft.asc
ver=8 # or 9
dnf install -y https://packages.microsoft.com/config/rhel/${ver}/packages-microsoft-prod.rpm
dnf install aztfexport

# apt package
curl -sSL https://packages.microsoft.com/keys/microsoft.asc > /etc/apt/trusted.gpg.d/microsoft.asc
ver=20.04 # or 22.04
apt-add-repository https://packages.microsoft.com/ubuntu/${ver}/prod
apt-get install aztfexport
```

# ハンズオン
### 1 リソース作成
まずは手作業でAzureリソースを作成します。Azure Portalから作成してもいいですし、以下のようにAzure CLIコマンドでVMマシンを作成しても構いません。

```terminal:Azure CLI
az group create --name terraform-export-RG --location japaneast

az vm create \
   --resource-group terraform-export-RG \
   --name terraform-export-VM \
   --image Debian \
   --admin-username azureadmin \
   --generate-ssh-keys \
   --public-ip-sku Standard
```

![](/images/azure-export-terraform/image2.png)
*作成されたリソースグループ*

### 2 リソースエクスポート
手動で作成したリソースをAzure Export for Terraformでエクスポートします。
エクスポート先となる空のディレクトリを用意します。
![](/images/azure-export-terraform/image3.png)

このディレクトリ内でエクスポートコマンドを実行します。
```terminal
aztfexport resource-group terraform-export-RG
```

オプションを使わない場合対話形式で実行され対象となるリソース一覧が表示されます。
![](/images/azure-export-terraform/image4.png)
エクスポート不要なリソースがありましたらdeleteキーでスキップできます。
wコマンドでエクスポートを開始します。
![](/images/azure-export-terraform/image5.png)

エクスポートが完了するとディレクトリ配下にHCLファイルが生成されます。

```terminal
ls
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----        2023/05/14     15:19                  .terraform
-a---        2023/05/14     15:19           1185   .terraform.lock.hcl
-a---        2023/05/14     15:27           4180   aztfexportResourceMapping.json
-a---        2023/05/14     15:27           4720   main.tf
-a---        2023/05/14     15:19             40   provider.tf
-a---        2023/05/14     15:19            144   terraform.tf
-a---        2023/05/14     15:27          18449   terraform.tfstate
```

`terraform init`して、`terraform plan`を実行後tfstateファイルとAzureリソース間で差異が出ないと表示されれば無事AzureのリソースがTerraformにエクスポートされたことになります。
![](/images/azure-export-terraform/image6.png)

# 所感
Azure Export for Terraformで既存AzureリソースをTerraformにエクスポートしました。AWSの時と比較すると簡単に既存リソースをIaC化できとても便利なツールだと思います。
ただ、Azure Export for Terraformは完全にリソースをエクスポートできることを保証していません。

> The Terraform configurations generated by aztfexport are not meant to be comprehensive and do not ensure that the infrastructure can be fully reproduced from the generated configurations. For details, please refer to the limitation.[^4]
DeepL訳：aztfexportが生成するTerraformの構成は、包括的なものではありませんし、生成された構成からインフラを完全に再現できることを保証するものではありません。詳細については、制限事項をご参照ください。

例えばSQLデータベースの作成で管理者パスワードをポータルでは入力しますが、認証情報など機密性の高い値はエクスポートできません。他にも暗黙的な依存関係も認識できません。
これはTerraform自体がそういう仕様なので当然かもしれません。

>However, some dependencies cannot be recognized implicitly in configuration. For example, if Terraform must manage access control policies and take actions that require those policies to be present, there is a hidden dependency between the access policy and a resource whose creation depends on it. In these rare cases, the depends_on meta-argument can explicitly specify a dependency.[^5]
DeepL訳:しかし、設定において暗黙のうちに認識できない依存関係もあります。例えば、Terraformがアクセスコントロールポリシーを管理し、それらのポリシーが存在することを必要とするアクションを取らなければならない場合、アクセスポリシーと、その作成が依存するリソースの間に隠れた依存関係が存在する。このような稀なケースでは、depends_onメタ引数で依存関係を明示的に指定できます。

Azure Export for Terraformは便利ですがあくまでもIaC化へのサポートツールですので全部任せるのではなく、補助的な役割であることを理解したうえで是非AzureリソースのIaCを進めてみてください。

[^4]: https://github.com/Azure/aztfexport#limitation
[^5]: https://developer.hashicorp.com/terraform/language/resources/behavior

# 参考文献
https://qiita.com/koudaiii/items/10ae59a817abc438b71c
https://learn.microsoft.com/ja-jp/azure/developer/terraform/azure-export-for-terraform/export-first-resources?tabs=azure-cli
https://github.com/Azure/aztfexport