---
title: "nginxコマンドの出力内容をパイプしてみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux","Nginx"]
published: false
---
# 背景
会社で使っているWebサーバーがApacheなのかnginxなのか棚卸しする機会があり、バージョン情報と一緒に情報を収集するスクリプトを作っていました。
それぞれのバージョン情報はCLIコマンドがありますので簡単に取得できます。

```shell
$ httpd -v
Server version: Apache/2.2.34 (Unix)
Server built:   Nov  1 2017 18:47:16

$ nginx -v
nginx version: nginx/1.18.0
```

出力した情報をファイルに書き込んでKintoneにデータを集約するつもりでしたが、他の余分なものは削ぎ落したいと思いましたので、パイプで出力結果を渡して`cut`コマンドで加工して必要な部分だけ情報を抽出しました。

```shell
$ httpd -v | head -n 1 | cut -f 3 -d " "
Apache/2.2.34
```

Apacheは上記のようにバージョン情報だけを表示できましたので、nginxも同じように加工して抽出しようとしました。

```shell
$ nginx -v  | head -n 1 | cut -f 3 -d " "
nginx version: nginx/1.18.0
```

ところがパイプでnginxのバージョン情報を`cut`で加工しようとしても上手くいきません。
試しに`awk`コマンドでも試しましたが結果は変わりません。

```shell
$ httpd -v | head -n 1 | awk '{print $3}' #Apacheではちゃんと上手くいく
Apache/2.2.34

$ nginx -v | head -n 1 | awk '{print $3}' 
nginx version: nginx/1.18.0
```
今回はそんな感じでnginxのパイプの渡し方について調査してみました。

# nginxのCLI出力について
あれこれ調べていたら気になる記事を見つけました。

> Check for gunzip_module compiled or not
nginx -V 2>&1 | grep --color -o 'http_gunzip_module'[^1]


[^1]: https://www.cyberciti.biz/faq/unix-linux-bsd-display-nginx-version/


ここで注目するポイントは `2>&1`です。
これはファイルディスクリプタと言いまして、主にUNIX系OSで使われるファイル操作を汎用的に行うために使用する整数値です。





# 参考文献
https://www.cyberciti.biz/faq/unix-linux-bsd-display-nginx-version/
https://codezine.jp/article/detail/4836

