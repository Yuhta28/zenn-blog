---
title: "TerraformからforkされるOpenTFについて"
emoji: "🍴"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["OSS","Terraform","OpenTF"]
published: false
---

# 概要
2023年8月にTerraformなどのOSSを開発しているHashiCorp社は自社製品のライセンスを変更することを発表しました。
https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license
以前もこのことについて[記事](https://zenn.dev/yuta28/articles/migration-terraform-to-pulumi)で紹介していますが、このライセンス変更でTerraformなどを利用するHashiCorpユーザーに直接の影響はありません。

ただHashiCorp社のライセンス変更に反対する動きもありTerraformからフォークされた真のオープンソースを目指してる団体が出てきました。
それがOpenTFです。
https://opentf.org/

OpenTFは上記のマニフェスト(宣言書)によるとTerraformを真のオープンソースであり続けるためにHashiCorp社にライセンス変更の撤廃を要求、それができないならTerraformからフォークたOSSを管理することを明言しています。

https://github.com/opentffoundation/manifesto