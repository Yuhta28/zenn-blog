---
title: "Pulumi ESCで秘匿情報をセキュアに扱う"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","security","aws"]
published: false
---

# 序論
Pulumiから新しい機能がリリースされました。
https://www.pulumi.com/product/esc/

従来IaCで認証情報を参照する場合、Git管理外の変数ファイルから参照するか、AWS Secrets Manager[^2]やHashiCorp Vault[^3]のように別サービスから認証情報を参照するのが一般的です。

今回リリースされたPulumi ESC(Environments, Secretes, Configuration)は名前のとおり、環境変数や認証情報を定義できるマネージドサービスです。

![architecture](/images/pulumi-esc-secure-protect/image1.png)
*アーキテクチャ[^1]*

Pulumi Cloudと組み合わせることで認証情報をPulumiエコシステム内で管理でき、運用の負荷を減らせるメリットがあります。

[^1]: https://www.pulumi.com/product/esc/
[^2]: https://aws.amazon.com/jp/secrets-manager/
[^3]: https://www.vaultproject.io/