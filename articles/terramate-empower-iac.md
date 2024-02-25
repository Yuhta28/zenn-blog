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

一見、Terraformの実行を代行するTerragrunt[^1]のようなラッパーツールかと思いましたが、Terramateは単なるラッパーツールではなくオーケストレーションツールとしてIaC開発を楽にしてくれる様々な機能が提供されていましたので、Terramateを使ったIaC開発について紹介いたします。

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

DeepL訳:) Terraformはコードの再利用を可能にするためにモジュールの概念を提供していますが、モジュール単体では計画・適用時の分離はできません。すべてのTerraformコードの状態を一元管理している限り、このコードで使われているすべてのモジュールは常に計画・適用されます。

TerraformでIaC開発を進めていくとstateファイルが肥大化し、`terraform plan`や`terraform apply`に時間がかかってしまうことが多々あると思います。stateファイルを分割し、モジュールとして共通化することで複数環境で使いまわせるようにすることをTerraform開発元のHashiCorp社も推奨しております。[^2]

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

Terramateでは分離した個々のStateファイルのことをStacksと表現しています。

[^2]: https://developer.hashicorp.com/terraform/language/modules/develop/structure