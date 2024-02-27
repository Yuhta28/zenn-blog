---
title: "Terramateを使えばIaCは豊かになれるのか?"
emoji: "🧑🏻‍🤝‍🧑🏻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["terramate","terraform","aws","iac"]
published: false
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

TerraformでIaC開発を進めていくとStateファイルが肥大化し、`terraform plan`や`terraform apply`に時間がかかってしまうことが多々あると思います。Stateファイルを分割し、モジュールとして共通化し、複数環境で使いまわせるにようにすることをTerraform開発元のHashiCorp社も推奨しております。[^2]

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

もしくは前述のTerragruntを使ってラッピングすることでStateファイルを細かく分離するという方法があります。
https://zenn.dev/ring_belle/articles/terragrunt-start

Terramateでは分離した個々のStateファイルのことをStacksと表現していますが、小さく独立したStacksに分離したことに対するトレードオフについて創業者はブログで以下のように述べています。

> But splitting your code into smaller independent units, that we call stacks, has some significant tradeoffs and it will require you to:
> - Share data and code across all stacks to keep the code DRY (Don’t Repeat Yourself).
> - Orchestrate your stacks so that not all stacks are executed all the time, but only stacks that have changed within a specific pull request (PR) will be executed (planned/applied).

DeepL:) しかし、コードをスタックと呼ばれる小さな独立した単位に分割するには、いくつかの重大なトレードオフがある：
- すべてのスタックでデータとコードを共有し、コードをDRY（Don't Repeat Yourself）に保つ。
- すべてのスタックが常に実行されるのではなく、特定のプルリクエスト（PR）内で変更されたスタックだけが実行（計画／適用）されるように、スタックをオーケストレーションする。

私も経験があるのでわかりますが、CI/CDパイプライン上にTerraform実行基盤を乗せて、`terraform apply`すると更新されていないモジュールに対してもデプロイが実行されてしまい、完了までの時間が段々と長くなってきます。TerramateのTerragruntとの大きな違いは、Gitと連携することでブランチ単位で行われた変更を検出し、必要なStateのみデプロイすることで実行時間の短縮を目的としていること、Terraformのラッパーではなく任意のコマンドを実行できることです。(これによりTerraform以外のIaCツールのコマンドも実行できます。)

TerramateとTerragruntの違いについては創業者が同じように詳しくブログで解説していますので詳細を知りたい人はこちらの記事をご覧ください。

https://blog.terramate.io/terramate-and-terragrunt-f27f2ec4032f

[^2]: https://developer.hashicorp.com/terraform/language/modules/develop/structure

# Terramate使い方
Terramateを実際に使ってみたいと思います。
インストール手順は[こちら](https://terramate.io/docs/cli/installation)です。MacならHomebrew経由でインストールできます。

```terminal
$ brew install terramate
$ terramate version
0.4.5
```

またデフォルトシェルがbash,zsh,fishの場合コマンドの自動補完が使えます。

```terminal
$ terramate install-completions
```

公式のクイックスタートではTerraformのnullリソース[^3]を作成する複数のTerraformに対して一括デプロイを実行します。nullリソースならGitHubとの連携やAWSやGoogle Cloudの認証を使う必要がなくすぐに始められます。

https://terramate.io/docs/cli/getting-started/

事前にGit[^4]とTerraform[^5]をローカルマシンで実行できるようにしておいてください。

[^3]: https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource
[^4]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
[^5]: https://developer.hashicorp.com/terraform/install