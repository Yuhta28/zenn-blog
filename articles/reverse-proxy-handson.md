---
title: "リバースプロキシを使って、別Webサイトのコンテンツを表示させてみた"
emoji: "🐀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nginx","WordPress","プロキシ"]
published: true
---
# 概要
最近仕事でリバースプロキシを扱いましたが、今まで実際に触ったことがなく本で読んだ知識レベルでしかなかったせいで、GW前最後の仕事でかなり手こずりました。
なのでもう一度リバースプロキシの知識を見直すために実際にプロキシサーバーを立てて理解を深めてみようと思います。

# プロキシの基本
基本から立ち返るためにリバースプロキシの前に通常のプロキシ(フォーワードプロキシ)の役割についてもう一度調べてみます。
## プロキシについて
プロキシとは日本語で**代理**を意味し、インターネット接続ができない社内ネットワークにあるPCの代理として、インターネット接続を担います。
プロキシの役割を持つサーバーのことをプロキシサーバーと呼びます。
https://www.infraexpert.com/study/security23.html

リンク先の図に記載されているとおり、PCからはプロキシサーバーはWebサイトを表示させるサーバーとしての役割を持ち、Webサーバーからはコンテンツを返すためのクライアントの役割を持ちます。
PCからはWebサーバーの存在を直接感知できないことからさまざまなメリットがあります。

#### プロキシのメリット
- キャッシュによるレスポンス向上
- ユーザー認証
- ユーザー管理
- 有害サイトのフィルタリング
- 匿名性の確保

リンク先から紹介しましたが、特に大きいのはキャッシュ機能でしょうか。
毎回PCからのリクエストに対してWebサーバーがレスポンスを返すと負荷がかかり多人数に同時に行なうと処理が遅くなり、ユーザーの利便性も損なわれてしまいます。
一度プロキシサーバー経由でアクセスし、次回以降はプロキシサーバー内のキャッシュを活用することでWeb通信の高速化を実現させます。
## リバースプロキシについて
それではリバースプロキシとはどのような役割を持つのでしょうか。
リバースプロキシはインターネットから社内サーバーへアクセスされる通信を中継する仕組みです。
社内からインターネットへ接続することの反対で、インターネットから内部ネットワーク接続するように中継することからリバースプロキシと呼びます。
リバースプロキシも通常のプロキシと同じく自身をクライアントとしてWebサーバーへリクエストを転送し、Webサーバーからをレスポンスを受け取ったら今度はサーバーとしてレスポンス結果をPCに返します。
https://www.atmarkit.co.jp/ait/articles/1608/25/news034.html
リバースプロキシのメリットも通常のプロキシと重複している部分もありますが、以下のメリットがあります。
#### リバースプロキシのメリット
- 負荷分散によるスループット向上
- キャッシュによるレスポンス向上
- 異なるアプリやフレームワーク、プラットフォームの統合
- セキュリティ強化
- SSLの集中管理

各項目の詳細については上記のリンク先を見ていただきたいですが、通常のプロキシと大きく違うのは負荷分散によるスループット向上でしょうか。
PCからは１台のサーバーに見えても、実はリバースプロキシを使うことで複数の同じ機能を持つWebサーバーにリクエストを分散させることでアクセス負荷に耐えられることができるロードバランサーのような役割を持つことができます。

# ハンズオン
では実際にプロキシサーバーを構築していきます。
今回の構築目的ですが、Webサーバーで表示されているWebコンテンツをプロキシサーバーから見れるかどうか検証しキャッシュできるか確認していきたいと思います。
## ネットワーク構成図
簡単な図になりますが、今回のハンズオンの構成図は以下のとおりです。
![](https://storage.googleapis.com/zenn-user-upload/u5j7x3z68wrc2yh01k3j2xloy6p5)
PCからリバースプロキシへアクセスし、プロキシサーバーはPCから受け取ったリクエストをWebサーバーへ転送します。
リクエストを受け取ったWebサーバーはプロキシサーバーへWebコンテンツをレスポンスし、コンテンツを受け取ったプロキシサーバーはPCへレスポンスを転送します。
そしてPCからはプロキシサーバーからコンテンツを受け取ったようにみえるということです。
プロキシサーバーはnginxで動かし、WebサーバーはApacheが入ったWordPressを使います。
#### プロキシサーバー情報
- Amazon Linux2
- nginx 1.18
#### Webサーバー情報
- Ubuntu 20.04
- WordPress 5.7
- Apache 2.4
- MySQL 8.0
- PHP 7.4

Webサーバーの構築ですが、こちらのスクラップをご覧ください。
https://zenn.dev/yuta28/scraps/1e14a65a26626b

## プロキシサーバー構築
Amazon Linux2でnginxをインストールする場合、AWSが提供する`amazon-linux-extras`パッケージからインストールします。
```
$ sudo amazon-linux-extras install nginx1
Installing nginx
読み込んだプラグイン:extras_suggestions, langpacks, priorities, update-motd
リポジトリーを清掃しています: amzn2-core amzn2extra-docker amzn2extra-nginx1
12 個の metadata ファイルを削除しました
4 個の sqlite ファイルを削除しました
0 個の metadata ファイルを削除しました
読み込んだプラグイン:extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                                                       | 3.7 kB  00:00:00
amzn2extra-docker                                                                                | 3.0 kB  00:00:00
amzn2extra-nginx1                                                                                | 3.0 kB  00:00:00

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

完了しました!
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 38  nginx1=latest            enabled      [ =stable ]
```

せっかくなので無料ドメインを取得してプロキシサーバーにドメインを付与します。
ドメイン名：`proxy-yuta.ddns.net`

nginxを起動して`http://proxy-yuta.ddns.net` でnginxのテストページが表示できることを確認します。
![](https://storage.googleapis.com/zenn-user-upload/4b54z4oy4oh5jj9df79czv0t6zgv)



nginxの設定ファイル, `/etc/nginx/nginx.conf`のserverセクション内を以下のように修正します。

```nginx 
server {
        listen       80;
        listen       [::]:80;
        server_name  http://proxy-yuta.ddns.net/;
        root         /usr/share/nginx/html;


        location / {
            proxy_pass https://www.yuta-aws.name/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Host $host;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
            }

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

修正後にnginxを再起動して設定を読み込ませます。
もう一度`http://proxy-yuta.ddns.net`を開きますとWordPressのページが表示されます。
![](https://storage.googleapis.com/zenn-user-upload/olzykwjmhl2v3ii2l3quxbzju8es)

# 解決できない問題点
proxyドメインでWebサイトコンテンツを表示させることができましたが、解決できずに困っていることがあります。
proxyドメインのページから他のページに遷移すると元のWebサーバーのドメインに変更されてしまいます。
![](https://storage.googleapis.com/zenn-user-upload/k3viv6y862k6ynx8oyrazya9qxwd)
やりたいこととしてはプロキシサーバーのドメインでWebサイトのコンテンツを表示し続けたいのですが、nginxの公式ドキュメントを調べてもわかりませんでした。
`ngx_http_proxy_module`モジュールのドキュメントを調べましたが、それ以外で調べるポイントなどご存じのかたがいましたらアドバイス宜しくお願いいたします。

# 所感
リバースプロキシの知識を深めるためにプロキシサーバーを構築し、プロキシサーバーでWebサイトのコンテンツを表示させることができました。
ですがトップページから別サイトに移動するとドメインが戻ってしまうのでこれを何とかしたいと考えています。
Stack Overflowやteratailを使って質問をしてみようと思います。

# 参考文献
https://eset-info.canon-its.jp/malware_info/special/detail/201021.html
http://nginx.org/en/docs/http/ngx_http_proxy_module.html


