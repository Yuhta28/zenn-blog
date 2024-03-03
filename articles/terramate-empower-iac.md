---
title: "Terramateを使えばIaCは豊かになれるのか?"
emoji: "🧑🏻‍🤝‍🧑🏻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["terramate","terraform","aws","iac"]
published: true
---

# 序論

先日LinkedInで面白そうなIaCツールを紹介してもらいました。
https://terramate.io/

マネージドサービス版はまだクローズドベータで一般利用できませんでしたが、GitHubにCLI版がオープンソールとして公開されておりました。

https://github.com/terramate-io/terramate

一見、Terraformの実行を代行するTerragrunt[^1]のようなラッパーツールかと思いましたが、Terramateは単なるラッパーツールではなくオーケストレーションツールとしてIaC開発を楽にしてくれるさまざまな機能が提供されていましたので、Terramateを使ったIaC開発について紹介いたします。

:::message alert
オープンソース版に限定した機能の紹介になります。
:::

[^1]: https://terragrunt.gruntwork.io/

# 対象読者
- 複数のIaCツールの管理に苦労している人
- (後述のハンズオンのため)Terraformの基礎知識がある人

# Terramateについて

創業者のブログを見るとリリースしたのは2022年5月みたいで3年ほど前にリリースされた比較的新しいGo製の開発ツールのようです。

https://blog.terramate.io/introducing-terramate-an-orchestrator-and-code-generator-for-terraform-5e538c9ee055

[ドキュメント](https://terramate.io/docs/cli/introduction)ではTerramateについてこう述べられています。

> Terramate CLI is an open-source Infrastructure as Code (IaC) orchestration tool for Terraform, OpenTofu, Terragrunt, Kubernetes, Pulumi, AWS Cloud Formation, AWS Cloud Development Kit (CDK), Azure Resource Manager (ARM), Biceps, and others.

DeepL訳:) Terramate CLIは、Terraform、OpenTofu、Terragrunt、Kubernetes、Pulumi、AWS Cloud Formation、AWS Cloud Development Kit (CDK)、Azure Resource Manager (ARM)、BicepsなどのためのオープンソースのInfrastructure as Code (IaC)オーケストレーションツールです。

オーケストレーションツールとはどういう意味でしょうか🤔
詳しく説明する前に冒頭の創業者のブログ記事でTerraformの課題について述べている箇所がありましたのでそちらを見ていきます。

> Terraform provides the concept of modules to allow reusing code, but modules on their own do not provide isolation on plan and apply time. As long as you keep a centralized state for all your Terraform code, all modules used in this code will always be planned and applied.

DeepL訳:) Terraformはコードの再利用を可能にするためモジュールの概念を提供していますが、モジュール単体では計画・適用時の分離はできません。すべてのTerraformコードの状態を一元管理している限り、このコードで使われているすべてのモジュールは常に計画・適用されます。

TerraformでIaC開発を進めていくとstateファイルが肥大化し、`terraform plan`や`terraform apply`に時間がかかってしまうことが多々あると思います。stateファイルを分割しモジュールとして共通化し、複数環境で使いまわせるようにすることをTerraform開発元のHashiCorp社も推奨しております。[^2]

```terminal
tree complete-module/
.
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
├── ...
├── modules/
│   ├── nestedA/
│   │   ├── README.md
│   │   ├── variables.tf
│   │   ├── main.tf
│   │   ├── outputs.tf
│   ├── nestedB/
│   ├── .../
├── examples/
│   ├── exampleA/
│   │   ├── main.tf
│   ├── exampleB/
│   ├── .../
```

もしくは前述のTerragruntを使ってラッピングすることでstateファイルを細かく分離するという方法があります。
https://zenn.dev/ring_belle/articles/terragrunt-start

Terramateでは分離した個々のstateファイルのことを[Stacks(以下スタック表記)](https://terramate.io/docs/cli/stacks/)と表現していますが、小さく独立したスタックに分離したことに対するトレードオフについて創業者はブログで以下のように述べています。

> But splitting your code into smaller independent units, that we call stacks, has some significant tradeoffs and it will require you to:
> - Share data and code across all stacks to keep the code DRY (Don’t Repeat Yourself).
> - Orchestrate your stacks so that not all stacks are executed all the time, but only stacks that have changed within a specific pull request (PR) will be executed (planned/applied).

DeepL:) しかし、コードをスタックと呼ばれる小さな独立した単位に分割するには、いくつかの重大なトレードオフがある。
- すべてのスタックでデータとコードを共有し、コードをDRY（Don't Repeat Yourself）に保つ。
- すべてのスタックが常に実行されるのではなく、特定のPR内で変更されたスタックだけが実行（計画／適用）されるように、スタックをオーケストレーションする。

私も経験があるのでわかりますが、CI/CDパイプライン上にTerraform実行基盤を乗せて、`terraform apply`を実行すると更新されていないモジュールに対してもデプロイが実行されてしまい、完了までの時間が段々と長くなってしまいます。
TerramateとTerragruntの大きな違いは、Gitと連携しブランチ単位で行われた変更を検出し、必要なstateのみをデプロイすることで実行時間の短縮を目的としていること、Terraformのラッパーではなく任意のコマンドを実行できることです。(これによりTerraform以外のIaCツールのコマンドも実行できます。)

TerramateとTerragruntの違いは創業者が詳しくブログにて解説していますので詳細を知りたい人はこちらの記事をご覧ください。

https://blog.terramate.io/terramate-and-terragrunt-f27f2ec4032f

[^2]: https://developer.hashicorp.com/terraform/language/modules/develop/structure

# Terramateの使い方
Terramateを実際に使ってみたいと思います。
インストール手順は[こちら](https://terramate.io/docs/cli/installation)です。MacならHomebrew経由でインストールできます。

```terminal:インストール方法
$ brew install terramate
$ terramate version
0.4.5
```

またデフォルトシェルがbash,zsh,fishの場合コマンドの自動補完が使えます。

```terminal:コマンド補完
$ terramate install-completions
```

公式のクイックスタートではTerraformのnullリソース[^3]を作成する複数のTerraformに対して一括デプロイを実行します。nullリソースならGitHubとの連携やAWSやGoogle Cloudの認証を使う必要がなく、すぐに始められます。
事前にGit[^4]とTerraform[^5]をローカルマシンで実行できるようにしておいてください。

## クイックスタート
https://terramate.io/docs/cli/getting-started/

### 初期セットアップ
クイックスタートではローカルGitリポジトリを作成し、リポジトリ内に2つのスタックを作成します。それぞれのスタックではTerraformのnullリソースを作成し変更のあったTerraformリソースのみに対してTerramateが差分検知して余計なデプロイをなくすことが体験できます。

```bash:リポジトリ作成
$ git init terramate-quickstart
$ cd terramate-quickstart
$ git commit --allow-empty -m 'Initial empty commit'
```

### 最初のスタック作成
リポジトリ作成後、最初のスタックを作成します。スタック作成コマンドは`terramate create <作成先パス指定>`になります。
オプションを指定しなければスタック名は指定先パスのディレクトリ名になります。

```bash:スタック作成
$ terramate create \
  --name "A first stack" \
  --description "The very first Terramate Stack of the Quickstart guide" \
  stacks/first
```

`stacks/first`ディレクトリに`stack.tm.hcl`というスタックファイルが作成されます。

```hcl:stack.tm.hcl
stack {
  name        = "A first stack"
  description = "The very first Terramate Stack of the QuickStart guide"
  id          = "8e98ba37-fc0d-4700-a71a-234f49542d3c"
}
```

`id`はオプションでユーザーが設定できますが、一意である必要があるため自動生成を推奨しています。
作成したスタックをTerramateが認識出るようにリポジトリへコミットします。

```bash
$ git add stacks/first/stack.tm.hcl
$ git commit -m 'Create the first terramate stack'

$ terramate list
stacks/first
```

### 2番目のスタック作成

最初と同じように2番目のスタックを作成します。

```bash
$ terramate create \
  --name "A second stack" \
  --description "The second Terramate Stack of the Quickstart guide" \
  stacks/second

$ git add stacks/second/stack.tm.hcl
$ git commit -m 'Create a second terramate stack'
```

コミットまで完了したらTerramateが2番目のスタックを認識できるようになります。

```bash
$ terramate list
stacks/first
stacks/second
```

### 差分検知

TerramateはGitの変更履歴を元に差分検知を行ない、リポジトリ内の最新コミットで変更されたスタックのみを検出できます。

```bash
#最初のコミットから2回目のコミットまでの変更した部分のみを検出
$ terramate list --changed
stacks/second
```


### コード生成

TerramateにはスタックをDRY[^9]に保てるコード生成機能が提供されています。例えば作成した2つのスタックをそれぞれローカルマシンの中にTerraform stateファイルを生成できるようにするローカルバックエンドを設定したいとします。

`stacks`ディレクトリにTerraformのバックエンドを指定する`backend.tm.hcl`を作成し、生成コマンドを実行することで自動的に各スタック配下にTerraformのバックエンド設定を記述したTerraform stateファイルが生成されます。

```hcl:stacks/backend.tm.hcl
#
generate_hcl "backend.tf" {
  content {
    terraform {
      backend "local" {}
    }
  }
}
```

```bash
$ terramate generate

Code generation report

Successes:

- /stacks/first
	[+] backend.tf

- /stacks/second
	[+] backend.tf

Hint: '+', '~' and '-' mean the file was created, changed and deleted, respectively.
```

```powershell
$ ls .\stacks\first\  

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a---        2024/03/03     15:48             89 backend.tf # ←自動生成されたバックエンドファイル
-a---        2024/03/03     15:47            165 stack.tm.hcl

$ ls .\stacks\second\

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a---        2024/03/03     15:48             89 backend.tf # ←自動生成されたバックエンドファイル
-a---        2024/03/03     15:46            167 stack.tm.hcl
```

```hcl:backend.tf
// TERRAMATE: GENERATED AUTOMATICALLY DO NOT EDIT

terraform {
  backend "local" {
  }
}
```

自動生成したコードもコミットします。

```bash
$ git add stacks
$ git commit -m 'Add a Terraform Backend Config to all stacks'
```

### オーケストレーション実行
Terraformのバックエンド設定を記述したのでそれぞれのスタック内で`terraform init`を実行する必要があります。ここでTerramateの真価を発揮するのがTerramateオーケストレーションです。
`terramate run`でスタック内のコマンドを実行できる機能が提供されており、個別に`terraform init`をする必要がなくなります。

:::message
Terragruntでもできるのでマッパーコマンドと思われるかもしれませんが、Terraform以外のコマンドも実行できます。
:::

`terramate run`を実行する前に、`.gitignore`ファイルを作成してtfstateファイル各種をGitリポジトリへコミットすることを防ぎます。

```text:.gitignore
.terraform
.terraform.lock.hcl
terraform.tfstate
```

```bash
$ git add .gitignore
$ git commit -m 'Add .gitignore'
```

`.gitignore`をコミットしたらTerramateオーケストレーションでTerraformの初期化を実行します。

```bash:複数スタックに対してterraform init実行
$ terramate run terraform init

terramate: Entering stack in /stacks/first
terramate: Executing command "terraform init"

Initializing the backend...

Successfully configured the backend "local"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
terramate: Entering stack in /stacks/second
terramate: Executing command "terraform init"

Initializing the backend...

Successfully configured the backend "local"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

スタック配下に`.terraform/terraform.tfstate`が作成されるはずです。

`run`コマンドで`terraform plan`を実行してみます。

```bash:複数スタックに対してterraform plan実行
$ terramate run terraform plan

terramate: Entering stack in /stacks/first
terramate: Executing command "terraform plan"

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
terramate: Entering stack in /stacks/second
terramate: Executing command "terraform plan"

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
```

Terramateオーケストレーションで複数のスタックに対してTerraformコマンドが実行できることを確認できました。

### Terraformリソース作成

まだTerraformリソースを作成していないので前述の`terramate run terraform plan`ではNo changesと表記されていました。
nullリソースを作成するTerraformファイルを作成します。
ここではそれぞれのスタックで異なるリソースを作成することを想定し、2番目のスタック内にnull.tfファイルを作成します。

```hcl:stacks/second/null.tf
resource "null_resource" "quickstart" {
}
```

```bash:コミット
$ git add stacks/second/null.tf
$ git commit -m "Add a null resource"
```

nullリソースを記述したのでnullプロバイダーをインストールするためもう一度`terraform init`が必要になります。

ここで`terramate run terraform plan`を再度実行すると変更のなかった最初のスタックにも`terraform init`を実行し、実行時間に無駄が発生してしまいます。
TerramateはGitベースで差分検知が実行できるので変更を含むスタックのみに対してオーケストレーションコマンドを実行することで実行時間を抑えられます。

```bash:再初期化
$ terramate run --changed terraform init

#変更のあった2番目のスタックに対して再初期化を実行している。
terramate: Entering stack in /stacks/second
terramate: Executing command "terraform init"

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/null...
- Installing hashicorp/null v3.2.2...
- Installed hashicorp/null v3.2.2 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

```bash:plan
$ terramate run --changed terraform plan
terramate: Entering stack in /stacks/second
terramate: Executing command "terraform plan"

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # null_resource.quickstart will be created
  + resource "null_resource" "quickstart" {
      + id = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

`terraform plan`が実行できることを確認したら`terraform apply`を実行します。

```bash:apply
$ terramate run --changed terraform apply -auto-approve

terramate: Entering stack in /stacks/second
terramate: Executing command "terraform apply -auto-approve"

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # null_resource.quickstart will be created
  + resource "null_resource" "quickstart" {
      + id = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
null_resource.quickstart: Creating...
null_resource.quickstart: Creation complete after 0s [id=6372471593458417750]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

```bash:作成したリソースを確認
$ terramate run --changed terraform state list
terramate: Entering stack in /stacks/second
terramate: Executing command "terraform state list"
null_resource.quickstart

$ terramate run --changed terraform state show null_resource.quickstart
terramate: Entering stack in /stacks/second
terramate: Executing command "terraform state show null_resource.quickstart"
# null_resource.quickstart:
resource "null_resource" "quickstart" {
    id = "6372471593458417750"
}
```

再度`terraform apply`を実行してもリソースに更新がないためNo changesを返します。

```bash:冪等性を持つ
$ terramate run --changed terraform apply
terramate: Entering stack in /stacks/second
terramate: Executing command "terraform apply"
null_resource.quickstart: Refreshing state... [id=6372471593458417750]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are
needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
```

[^3]: https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource
[^4]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
[^5]: https://developer.hashicorp.com/terraform/install
[^6]: https://opentofu.org/
[^7]: https://kubernetes.io/ja/
[^8]: https://aws.amazon.com/jp/cdk/
[^9]: https://e-words.jp/w/DRY%E5%8E%9F%E5%89%87.html

# 所感

Terramateの基本コンセプトとクイックスタートについて紹介しました。前職でTerraformのCI/CDパイプラインを構築した時はGitHub Actionsを使用しましたが、変更差分のあったTerraformモジュールだけデプロイできるようにワークフロー内でif文で分岐する作りこみがあり大変だった記憶があります。

Terramateを使ったオーケストレーションなら実行時間の短縮と運用コスト削減を実現でき、複数のIaCツールを1つのリポジトリの中でまとめられるようにもなります。

まだ開発段階のツールなので不足している部分もありすぐの業務利用は難しいですがこれからの発展に期待したいところです。

