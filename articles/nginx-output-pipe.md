---
title: "nginxコマンドの出力内容をパイプしてみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux","Nginx"]
published: false
---
# 概要
会社で使っているWebサーバがApacheなのかNginxなのか棚卸しする機会があり、バージョン情報と一緒に情報を収集するスクリプトを作っていました。
それぞれのバージョン情報はCLIコマンドがありますので簡単に取得できます。

```shell
$ httpd -v
Server version: Apache/2.2.34 (Unix)
Server built:   Nov  1 2017 18:47:16

$ nginx -v
nginx version: nginx/1.18.0
```

