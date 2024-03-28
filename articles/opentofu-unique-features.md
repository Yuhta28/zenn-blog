---
title: "OpenTofuでStateファイルの暗号化が実装されました"
emoji: "📛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["opentofu","iac","oss"]
published: false
---

# 序論

[以前紹介したOpenTofu](https://zenn.dev/yuta28/articles/fork-opentf-from-terraform)が2024年1月からGAされています。[^1]
GAされた時点ではTerraform 1.6系に追随する形でOpenTofu1.6としてリリースされましたが、今後予定されるOpenTofu1.7からは独自の機能がいくつか実装されることになります。その中でも一番の目玉はStateファイルの暗号化でしょうか。
今まで平文で保存してきたStateファイルを暗号化して保存できるようになり、セキュリティが向上しました。プレリリースされたばかりですが、1.7で追加されたOpenTofuを触ってみこうと思います。

:::message alert
2024年3月時点で1.7はα版です。本番環境での利用は控えてください。
:::

[^1]: https://opentofu.org/blog/opentofu-is-going-ga/

# 対象読者

- Terraformについての基礎知識を持っている
- Stateファイルの取り扱いについて課題を持っている
- OpenTofuの独自機能について知りたい

# OpenTofu 1.7について

前述した通りOpenTofu1.7.0は現在α版としてプレリリースされています。[^2]

https://opentofu.org/blog/help-us-test-opentofu-1-7-0-alpha1/

本番環境での利用は推奨されておらずあくまでも非本番環境で使い、[フィードバックを募集している](https://github.com/opentofu/opentofu/issues/new?assignees=&labels=preview-release-feedback&projects=&template=1_7_0_alpha1_feedback.yml)段階となっています。
各種パッケージマネージャーからもインストールできず、[GitHub Releaseページ](https://github.com/opentofu/opentofu/releases/tag/v1.7.0-alpha1)からダウンロードすることのみ利用できます。

[^2]: 2024年3月時点

# 追加機能について

1.7では新機能や新しい関数などTerraformにはないものが組み込まれています。

### 主な追加機能

- Stateファイルの暗号化
- removedブロック
- 独自の組み込み関数

Stateファイルの暗号化は1.7の大きな目玉として注目されており、すでに公式ドキュメントも充実しています。

https://1-7-0-alpha1.opentofu.pages.dev/docs/language/state/encryption/

新規に作られるStateファイル以外にも作成済みの暗号化されていないStateファイルも暗号化されたStateファイルに移行できる手順も用意されています。
まずは新規に暗号化されたStateファイルを作成してみます。

# Stateファイル暗号化

暗号化に使用する鍵は現時点では2つの手段があります。
自身でパスフレーズを設定して、暗号鍵を生成するPBKDF2[^3]か、AWSの鍵管理マネージドサービスであるAWS KMS[^4]です。
また2024年3月時点でサポートされている暗号化方式はAWS-GCM[^5]のみになります。

ではStateファイルを暗号化するコードを書いていきます。

OpenTofuの階層構造はシンプルにしており、Amazon S3を作成するだけの構造にしています。

```terminal
$ tree
├── encrypt.tf　→ 今回の肝
├── init.tf
└── s3.tf
```

```hcl:init.tf
#AWSプロバイダーインストール
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
provider "aws" {
  region = "ap-northeast-1"
}
```

```hcl:s3.tf
#S3作成ファイル
resource "aws_s3_bucket" "example" {
  bucket = "yuta-opentofu-test-bucket"
}
```

### PBKDF2の場合

PBKDF2の場合、パスフレーズをHCLコード内に記載します。

```hcl:encrypt.tf
terraform {
  encryption {
    key_provider "pbkdf2" "passphrase" {
      passphrase = "correct-horse-battery-staple" #最低でも16文字以上
    }
    method "aes_gcm" "my_method" {
      keys = key_provider.pbkdf2.passphrase
    }
    state {
      method = method.aes_gcm.my_method
    }
  }
}
```

`tofu apply`を実行し生成された`terraform.tfstate`を見ると中身が暗号化されております。

```json
{
  "meta":{
    "key_provider.pbkdf2.basic":"~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~"
  },
  "encrypted_data":"~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~",
  "encryption_version":"v0"
}
```

`tofu state show ～`でリソースの中身は確認できます。

```terminal
$ tofu state show aws_s3_bucket.example
#見やすさ優先で一部省略しています
# aws_s3_bucket.example:
resource "aws_s3_bucket" "example" {
    arn                         = "arn:aws:s3:::yuta-opentofu-test-bucket"
    bucket                      = "yuta-opentofu-test-bucket"
    bucket_domain_name          = "yuta-opentofu-test-bucket.s3.amazonaws.com"
    bucket_regional_domain_name = "yuta-opentofu-test-bucket.s3.ap-northeast-1.amazonaws.com"
    force_destroy               = false
    id                          = "yuta-opentofu-test-bucket"
    object_lock_enabled         = false
    region                      = "ap-northeast-1"
    request_payer               = "BucketOwner"
    tags_all                    = {}

    server_side_encryption_configuration {
        rule {
            bucket_key_enabled = false

            apply_server_side_encryption_by_default {
                sse_algorithm = "AES256"
            }
        }
    }
}
```

### AWS KMSの場合

AWS KMSを使った暗号化について紹介します。KMSを使う場合カスタマー管理型のキーを作成します。
![](/images/opentofu-unique-features/image1.png)
*カスタマーキー作成*


`key_provider`識別子と中のプロパティが異なりますので注意です。

```hcl:encrypt.tf
terraform {
  encryption {
    key_provider "aws_kms" "basic" {
      kms_key_id = "<your-own-customer-managed-key-id>"
      region     = "ap-northeast-1"
      key_spec   = "AES_256"
    }
    method "aes_gcm" "my_method" {
      keys = key_provider.aws_kms.basic
    }
    state {
      method = method.aes_gcm.my_method
    }
  }
}
```

```json
{
  "meta":{
    "key_provider.aws_kms.basic":"~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~"
  },
  "encrypted_data":"~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~",
  "encryption_version":"v0"
}
```

# 既存Stateファイルの暗号化

既に平文で保存されているStateファイルを暗号化する方法もOpenTofuは提供しています。
先ほどの`encrypt.tf`の`state`ブロックに以下を追加します。

```hcl:encrypt.tf
terraform {
  encryption {
    key_provider "pbkdf2" "passphrase" {
      passphrase = "correct-horse-battery-staple" #最低でも16文字以上
    }
    method "aes_gcm" "my_method" {
      keys = key_provider.pbkdf2.passphrase
    }
    state {
      method = method.aes_gcm.my_method
      #fallbackを追加
      fallback {}
    }
  }
}
```

追加後に`tofu apply`を実行すると暗号化されます。
また`fallback`ブロックの中にmethodプロパティを入れると暗号化されたStateファイルを復号化できます。

```hcl:encrypt.tf
terraform {
  encryption {
    key_provider "pbkdf2" "passphrase" {
      passphrase = "correct-horse-battery-staple" #最低でも16文字以上
    }
    method "aes_gcm" "my_method" {
      keys = key_provider.pbkdf2.passphrase
    }
    state {
      fallback {
        #fallbackで囲む
        method = method.aes_gcm.my_method
      }
    }
  }
}
```

[^3]: https://ja.wikipedia.org/wiki/PBKDF2
[^4]: https://aws.amazon.com/jp/kms/
[^5]: https://www.mbsd.jp/research/20200901/aes-gcm/

# 参考文献
https://zenn.dev/yukionodera/articles/opentofu-ga-v1-6