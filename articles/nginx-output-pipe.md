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

# ファイルディスクリプタについて
あれこれ調べていたら気になる記事を見つけました。

> Check for gunzip_module compiled or not
nginx -V 2>&1 | grep --color -o 'http_gunzip_module'[^1]

[^1]: https://www.cyberciti.biz/faq/unix-linux-bsd-display-nginx-version/

これを見た時nginxの出力形式は標準エラー出力ということに気が付きました。

注目するポイントは `2>&1`です。
これはファイルディスクリプタと言いまして、主にUNIX系OSで使われるファイル操作を汎用的に行なうために使用する整数値です。

一般的には以下の３点が有名です。
- 0:stdin(標準入力)
- 1:sdout(標準出力)
- 2:stderr(標準エラー出力)

1の標準出力ですが、これは普段出力されている結果で、先ほどのApacheのバージョン情報出力は標準出力となります。一方で、nginxのバージョン情報出力は2の標準エラー出力になります。(標準入力は私たちがコマンドを叩いてコンピュータに文字を渡していることを意味しています)

# パイプについて
コマンドの結果を渡すパイプ`|`ですが、これは**標準出力を次のコマンドの標準入力に引き渡す**という意味になります。
そのため、`nginx -v`のように標準エラー出力で表示されたものを単純にパイプするだけでは後続の`cut`コマンドに引き渡されないことになっているのです。
最初のコマンドの実行結果を次のコマンドに引き渡すためには、標準出力エラーと標準出力をまとめて引き渡すという設定が必要になります。

先ほどの記事の内容に戻りますが、`2>&1`は**標準エラー出力を標準出力にマージさせる**という意味になります。
実際にこのコマンドの実行結果を比較すればわかりやすいかと思います。

### nginxコマンドの実行比較

```shell
$ nginx -V  | grep --color -o 'http_gunzip_module' #標準出力の場合
nginx version: nginx/1.18.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --with-compat --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-stream_ssl_preread_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_geoip_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-http_auth_request_module --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic' --with-ld-opt=' -Wl,-E'
```

```shell
$ nginx -V 2>&1 | grep --color -o 'http_gunzip_module' #標準エラー出力の場合
http_gunzip_module
```

ご覧の通り、`nginx -v`の後ろに `2>&1`をつけることで出力結果が標準エラー出力と標準出力をまとめて次のコマンドに引き渡している事が分かります。

これにならって、最初のnginxのバージョン情報出力を`cut`で加工するためには以下の設定を行えば抽出することが出来ます。

```shell
$ nginx -v 2>&1 | cut -f 3 -d " " #標準エラー出力と標準出力をcutコマンドに引き渡し
nginx/1.18.0

$ nginx -v 2>&1 | head -n 1 | awk '{print $3}' #awsコマンドでもOK
nginx/1.18.0
```

# 所感
nginxの出力内容をパイプして次のコマンドに引き渡すことができました。
なぜnginxの出力が標準エラー出力になっているのかわからないのですが、どなたかご存知でしょうか🤔

もしかすると他にも実は標準エラー出力にて表示されるCLIコマンドがあるかもしれないので、もしパイプできないと思った時は標準エラー出力の存在を疑ってみるのもいいのかもしれません。

# 参考文献
https://www.cyberciti.biz/faq/unix-linux-bsd-display-nginx-version/
https://codezine.jp/article/detail/4836
https://docs.microsoft.com/ja-jp/cpp/c-runtime-library/stdin-stdout-stderr?view=msvc-160
https://qiita.com/ritukiii/items/b3d91e97b71ecd41d4ea
https://www.infraeye.com/study/linuxz15.html
