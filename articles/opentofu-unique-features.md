---
title: "OpenTofuに独自機能が実装されました"
emoji: "📛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["opentofu","iac","oss"]
published: false
---

# 序論
[以前紹介したOpenTofu](https://zenn.dev/yuta28/articles/fork-opentf-from-terraform)が2024年1月からGAされています。[^1]
GAされた時点ではTerraform 1.6系に追随する形でOpenTofu1.6としてリリースされましたが、OpenTofu1.7からは独自の機能がいくつか実装されることになりました。実際に触りながら機能の紹介をしていきます。

[^1]: https://opentofu.org/blog/opentofu-is-going-ga/

# 対象読者
- OpenTofuの独自機能について知りたい
- Terraformについての基礎知識を持っている

# 参考文献
https://zenn.dev/yukionodera/articles/opentofu-ga-v1-6