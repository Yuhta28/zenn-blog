---
title: "CloudWatch Logsをいい感じにtailしてくれるcwを使ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","cloudwatch","cw"]
published: true
---
# 概要
CloudWatch Logsのログをいい感じに追いかけるのに役に立つOSS`cw`の紹介になります。

# 前提知識
AWS CLIにはCloudWatch Logsに収集されているログをtailしてくれるコマンドがあります。
` aws logs tail <ロググループ名>`

```bash
$ aws logs tail access_log
2021-07-22T07:45:55.422000+00:00 wordpress1 18.207.253.146 - - [22/Jul/2021:07:45:54 +0000] "GET /.env HTTP/1.1" 404 196 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36"
2021-07-22T07:45:59.986000+00:00 wordpress1 18.207.253.146 - - [22/Jul/2021:07:45:55 +0000] "POST / HTTP/1.1" 404 196 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36"
2021-07-22T07:49:31.689000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:31 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:49:31.689000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:31 +0000] "GET /icons/apache_pb2.gif HTTP/1.1" 200 4234 "http://13.231.136.114/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:49:31.940000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:31 +0000] "GET /icons/poweredby.png HTTP/1.1" 200 3412 "http://13.231.136.114/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:49:33.195000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:31 +0000] "GET /favicon.ico HTTP/1.1" 404 196 "http://13.231.136.114/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:49:34.198000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:33 +0000] "GET / HTTP/1.1" 403 4890 "https://www.google.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36"
2021-07-22T07:49:34.198000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:34 +0000] "GET /icons/poweredby.png HTTP/1.1" 200 3412 "http://13.231.136.114/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36"
```

`tail -f`のようにリアルタイムでログの記録を見たいときは以下のオプションをつけます。

`aws logs tail --follow <ロググループ名>`
```bash
$ aws logs tail --follow access_log
2021-07-22T07:45:55.422000+00:00 wordpress1 18.207.253.146 - - [22/Jul/2021:07:45:54 +0000] "GET /.env HTTP/1.1" 404 196 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36"
2021-07-22T07:45:59.986000+00:00 wordpress1 18.207.253.146 - - [22/Jul/2021:07:45:55 +0000] "POST / HTTP/1.1" 404 196 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36"
2021-07-22T07:49:31.689000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:31 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:49:31.689000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:31 +0000] "GET /icons/apache_pb2.gif HTTP/1.1" 200 4234 "http://13.231.136.114/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:49:31.940000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:31 +0000] "GET /icons/poweredby.png HTTP/1.1" 200 3412 "http://13.231.136.114/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:49:33.195000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:31 +0000] "GET /favicon.ico HTTP/1.1" 404 196 "http://13.231.136.114/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:49:34.198000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:33 +0000] "GET / HTTP/1.1" 403 4890 "https://www.google.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36"
2021-07-22T07:49:34.198000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:34 +0000] "GET /icons/poweredby.png HTTP/1.1" 200 3412 "http://13.231.136.114/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36"
2021-07-22T07:49:38.949000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:49:34 +0000] "GET /icons/apache_pb2.gif HTTP/1.1" 200 4234 "http://13.231.136.114/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36"
2021-07-22T07:54:52.911000+00:00 wordpress1 27.121.46.196 - - [22/Jul/2021:07:54:52 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:54:52.911000+00:00 wordpress1 27.121.46.196 - - [22/Jul/2021:07:54:52 +0000] "GET /icons/apache_pb2.gif HTTP/1.1" 200 4234 "http://54.168.233.33/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:54:52.911000+00:00 wordpress1 27.121.46.196 - - [22/Jul/2021:07:54:52 +0000] "GET /icons/poweredby.png HTTP/1.1" 200 3412 "http://54.168.233.33/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:54:56.985000+00:00 wordpress1 27.121.46.196 - - [22/Jul/2021:07:54:52 +0000] "GET /favicon.ico HTTP/1.1" 404 196 "http://54.168.233.33/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:55:05.949000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:55:01 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:55:15.949000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:55:11 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:55:19.470000+00:00 wordpress1 27.121.46.196 - - [22/Jul/2021:07:55:15 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:55:23.985000+00:00 wordpress1 27.121.46.196 - - [22/Jul/2021:07:55:19 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
2021-07-22T07:55:29.949000+00:00 wordpress2 27.121.46.196 - - [22/Jul/2021:07:55:25 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"

....
```

出力結果の見方は以下の通りです。
`タイムスタンプ+タイムゾーン ログストリーム名 ログメッセージ`

上記の図を見ても分かるとおり、`wordpress1`、`wordpress`と記載されている部分がログストリーム名になります。

![](/images/cloudwatch-fast-tail/image1.png)

`tail -f`でログを参照できるのは便利なのですが、ご覧のとおりログストリーム名ごとに指定ができず、複数のサーバーのアクセスログが混在している状態です。

今回はそんな悩みを解決してくれるOSSであるcwを紹介して使ってみた感想を記していきます。

# cwとは
cwはターミナル画面からCloudWatch Logsのログを**高速**で取得できるgoで作成されたOSSです。
https://github.com/lucagrulla/cw

珍しく`pip`や`npm`などの外部パッケージを必要とせず、OS依存となる実行アプリになります。
CloudWatch Logsの補助ツールで有名な`awslogs`[^1]と比較すると実行が速く、複数のロググループを指定することが大きな利点となります。
今回はcwを使ってログストリームごとに`tail -f`を実行して、ログ参照してみたいと思います。
[^1]: https://github.com/jorgebastida/awslogs

# インストール方法
GitHubの紹介ページにあるとおり、brewコマンドで簡単にインストールできます。[^2]
```bash:インストール方法
brew tap lucagrulla/tap
brew install cw
```
[^2]: Windowsもscoopというものを使えばできるそうです。https://scoop.sh/

# ハンズオン
インストールできましたので、早速使っていきます。
ローカルにAWS CLIをインストールしている場合、`aws configure`で認証情報をローカルマシンに登録していると思いますが、もしIAMロールを付与したEC2インスタンス内でAWS CLIを利用していた場合は、`~/.aws/config`ファイルにデフォルトのリージョン情報を設定する必要があります。
```bash:config
[default]
region = ap-northeast-1
```
まずは`cw -h`と入力してみます。

```bash
$ cw -h
Usage: cw <command>

The best way to tail AWS Cloudwatch Logs from your terminal.

Flags:
  -h, --help               Show context-sensitive help.
      --endpoint=URL       The target AWS endpoint url. By default cw will use the default aws endpoints. NOTE: v4.0.0
                           dropped the flag short version.
      --profile=PROFILE    The target AWS profile. By default cw will use the default profile defined in the
                           .aws/credentials file. NOTE: v4.0.0 dropped the flag short version.
      --region=REGION      The target AWS region. By default cw will use the default region defined in the
                           .aws/credentials file. NOTE: v4.0.0 dropped the flag short version.
      --no-color           Disable coloured output.NOTE: v4.0.0 dropped the flag short version.
      --version            Print version information and quit

Commands:
  ls groups
    Show all groups.

  ls streams <group>
    Show all streams in a given log group.

  tail <groupName[:logStreamPrefix]> ...
    Tail log groups/streams.

Run "cw <command> --help" for more information on a command.
```

cwの基本的な使い方やサブコマンドの紹介があります。
次に`cw ls groups`でCloudWatch Logsに収集されているロググループ一覧を見てみます。

```bash
$ cw ls groups
/aws/lambda/Yuta-rds-auto-stop
/ecs/first-run-task-definition
RDSOSMetrics
access_log
```

CloudWatch Logsのログを`tail -f` してみます。
```bash
$ cw tail access_log --follow --stream-name
wordpress2 - 27.121.46.196 - - [22/Jul/2021:09:19:33 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress2 - 27.121.46.196 - - [22/Jul/2021:09:20:24 +0000] "-" 408 - "-" "-"
wordpress2 - 27.121.46.196 - - [22/Jul/2021:09:20:29 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress2 - 27.121.46.196 - - [22/Jul/2021:09:20:29 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:20:48 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:20:49 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:20:49 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"

....

```

`tail -f`と同じことができました。
ここで本題となるログストリームごとのログ参照を実施します。
ロググループ名の後ろに`:ログストリーム名`をつけることで簡単にフィルタリングできます。

```bash
 cw tail access_log:wordpress1 --follow --stream-name
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:53 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:53 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:54 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:54 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:55 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:55 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:55 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:55 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:55 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:55 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:55 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:56 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:56 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:56 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:56 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:24:56 +0000] "GET / HTTP/1.1" 403 4890 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"

....

```

ご覧のとおりwordpress1のログだけが表示されるようになりました。

# 使っていて気になったこと
便利なcwですが、一個だけわからないエラーが発生しました。
しばらく`tail -f`でログを吐き出し続けていましたが、突然以下のエラーが出てコマンド実行が終了する結果になりました。

```bash
....

wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:29:45 +0000] "GET /wordpress/wp-admin/plugin-install.php?tab=popular HTTP/1.1" 200 126481 "http://54.168.233.33/wordpress/wp-admin/plugin-install.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
wordpress1 - 27.121.46.196 - - [22/Jul/2021:09:29:47 +0000] "GET /wordpress/wp-admin/plugin-install.php?tab=recommended HTTP/1.1" 200 127412 "http://54.168.233.33/wordpress/wp-admin/plugin-install.php?tab=popular" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
operation error CloudWatch Logs: FilterLogEvents, exceeded maximum number of attempts, 3, https response error StatusCode: 400, RequestID: 69b981bb-0bcf-4263-951e-73aabf9ab379, api error ThrottlingException: Rate exceeded
```

最大試行回数が超過したというAPIエラーですが、詳細が不明です。
情報が全然ないのでissueを立てて直接開発者に問い合わせてみようと思います。
##### 2021/7/23追記
問い合わせました。
https://github.com/lucagrulla/cw/issues/214
# 所感
cwを使ってログストリームごとに`tail -f`でログ参照してみました。
CloudWatch LogsのOSSで有名なのは`awslogs`ですが、少し動作がもっさりしていてかつPython製なので使っているホストのバージョン依存で動作に不安な面がありました。
cwはパッケージ不要でbrewで簡単にインストールできましたが、最後に出てきたエラーだけ気になったのでここを調査して問題なければ会社にも導入しようと思います。

# LT資料
LTのスライド資料です。
@[speakerdeck](2c749ceb40034cbea63d55621ed7944c)

# 英語記事
英訳しました。
https://dev.to/yuta28/i-use-cw-which-is-oss-to-tail-aws-cloudwatch-logs-2e9g

# 参考文献
https://blog.mmmcorp.co.jp/blog/2021/02/14/best-5ways-searching-cloudwatch-logs/
https://awscli.amazonaws.com/v2/documentation/api/latest/reference/logs/tail.html