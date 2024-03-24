---
title: "OpenTofuに独自機能が実装されました"
emoji: "📛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["opentofu","iac","oss"]
published: false
---

# 序論
[以前紹介したOpenTofu](https://zenn.dev/yuta28/articles/fork-opentf-from-terraform)が2024年1月からGAされています。[^1]
GAされた時点ではTerraform 1.6系に追随する形でOpenTofu1.6としてリリースされましたが、OpenTofu1.7からは独自の機能がいくつか実装されることになりました。2024年3月時点では1.7はα版としてプレリリースされており本番利用は非推奨ながらも自由に使えるようになっています。
どのような機能が実装されているか触ってみて確かめてみたいと思います。

[^1]: https://opentofu.org/blog/opentofu-is-going-ga/

# 対象読者
- OpenTofuの独自機能について知りたい
- Terraformについての基礎知識を持っている

# OpenTofu 1.7系について
前述した通りOpenTofu1.7.0は現在α版としてのプレリリースとなっています。[^2]

https://opentofu.org/blog/help-us-test-opentofu-1-7-0-alpha1/

本番環境での利用は推奨されておらずあくまでも非本番環境で使い、[フィードバックを募集している](https://github.com/opentofu/opentofu/issues/new?assignees=&labels=preview-release-feedback&projects=&template=1_7_0_alpha1_feedback.yml)段階となっています。
当然ながら各種パッケージマネージャーからはインストールできず、[GitHub Releaseページ](https://github.com/opentofu/opentofu/releases/tag/v1.7.0-alpha1)からダウンロードすることのみ利用できます。

[^2]: 2024年3月時点

# 追加機能について
1.7では新機能や新しい関数などTerraformにはないものが組み込まれています。

### 一例

- Stateファイル暗号化
- Stateファイルからのリソースを削除するremovedブロック
- 外だしファイルに変数を埋め込む`templatestring`関数
- IPアドレスが定義付けられたCIDR範囲内に含まれているかどうか判定する`cidrcontains`関数


# 参考文献
https://zenn.dev/yukionodera/articles/opentofu-ga-v1-6