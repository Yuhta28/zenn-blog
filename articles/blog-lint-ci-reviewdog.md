---
title: "GitHub ActionsでZennブログの校正を自動化してみた"
emoji: "🐀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [GitHubActions,reviewdog,Zenn,GitHub,textlint]
published: true
---
# 概要
前回の記事でも紹介したとおり、ZennはGitHubリポジトリと連携すると、自分の好きなエディタで記事を書くことができます。
https://zenn.dev/yuta28/articles/first-article-by-cli-yuta
ほかにもGitHubを連携することでCIツールを導入することが可能となり、記事作成の助けにもなります。
そこで今回はGitHub Actionsとtextlintとreviewdogを使うことで、PRするたびに自動でドキュメント校正をしてくれるCI環境の構築をしてみました。
# ドキュメント校正ツール
今回GitHub Actionsに組み込む校正ツールは以下の２つを使用します。

- textlint
- reviewdog

それぞれのツールについて簡単にはなりますが、紹介いたします。
## textlint
`textlint`はMarkdown形式やプレーンテキストのファイルを校正してくれるJavaScriptで記述されたOSSツールです。
https://textlint.github.io/
textlintはデフォルトルールというものが存在せず、自分でルールの設定とプラグインが必要になります。
ルールの設定箇所は`.textlintrc`ファイル内に記述します。

```json:.textlintrc
{
    "rules": {
        "preset-ja-technical-writing": {
            "sentence-length": false,
            "ja-no-weak-phrase": false
        },
        "prh": {
            "rulePaths": [
                "node_modules/prh/prh-rules/media/WEB+DB_PRESS.yml"
            ]
        }
    }
}
```

ここでは2つのルールをセットしてあります。

- textlint-rule-preset-ja-technical-writing
- textlint-rule-prh

`textlint-rule-preset-ja-technical-writing`は技術文書向けのルールプリセットです。
ルールプリセットは複数のルールを1つにまとめたルールで、`textlint-rule-preset-ja-technical-writing`は以下のルールが含まれています。
https://efcl.info/2016/07/13/textlint-rule-preset-ja-technical-writing/

- 「ですます調」、「である調」を統一します
- 同じ助詞を連続して使用しない
- 一文の長さは90文字以下とする

ただ、文字数の縛りはURLを文中に貼るとURLの文字もカウントしてしまい、PRの承認がうまくいかないときもありましたので`false`にしました。
ほかにも、「〜だと思います」という弱い日本語表現は技術書でしたら使わないようにしますが、ブログですのでそこまで厳しくしなくても良いと判断して、同じく`false`にしています。

`textlint-rule-prh`は表記ゆれをチェックしてくれる校正補助ツールです。
https://github.com/textlint-rule/textlint-rule-prh

たとえば、`りんごが箱の中に有った`という文でしたら、有ったという文をあったに変換してくれます。
#### 2021/4/10追記
Smart HRさんが新しくルールプリセットを作成して、外部に公開してくれたようです。
参考記事⇓
https://shanaiho.smarthr.co.jp/n/n881866630eda
GitHub⇓
https://github.com/kufu/textlint-rule-preset-smarthr
私の校正ルールに合いそうだと思いましたので、`.textlintrc`の中身を変更しました。

https://github.com/Yuhta28/zenn-blog/blob/main/.textlintrc

`textlint-rule-prh`の独自辞書の中身も公開してくれましたのでこちらも使ってみました。
https://github.com/kufu/textlint-rule-preset-smarthr/tree/main/dict
みなさんが使う場合は、ローカルのPCにSmart HRさんの辞書を格納しておいてください。
(私はdictディレクトリを作成してその中に置きました)

#### 2023/2追記
Smart HRさんのルールプリセットですが頻繁にルールが更新され、新しいルールが投稿済みの記事の校正を行ない大量にルール検知するということがありましたので一旦取りやめました。
2023/2現在で私が利用しているtextlintのルールは以下のものを使用しています。

- `textlint-rule-preset-ja-technical-writing`
    - https://github.com/textlint-ja/textlint-rule-preset-ja-technical-writing
- `textlint-rule-prh`
    - https://github.com/textlint-rule/textlint-rule-prh
- `textlint-filter-rule-allowlist`
    - https://github.com/textlint/textlint-filter-rule-allowlist
- `textlint-rule-aws-spellcheck`
    - https://github.com/37108/textlint-rule-aws-spellcheck

## reviewdog
`reviewdog`はtextlintなどLinterツールの結果を、GitHubのPRにレビューコメントを投稿してくれるGoで記述されたOSSツールです。
https://github.com/reviewdog/reviewdog

紹介した２つのツールを使うことで、ローカルで書く→`git push`→PRする→GtiHub Actionsが動くというCIが構築され、自動でドキュメントの校正が始まります。

それではGitHub Actionsのワークフローファイルの中身を紹介していきます。
https://github.com/Yuhta28/zenn-blog/blob/main/.github/workflows/textlint.yml

中身について解説します。
# ワークフローファイル解説
大まかなステップの流れとしては次の通りです。

- Nodeセットアップ
- キャッシュ取得
- textlintのインストール
- デバッグできるようにGitHub Actions内にSSH接続するジョブ(通常時はコメントアウト)
- textlintの実行
- reviewdogでPRコメントに通知

毎回のPRでNPMパッケージのインストールが走ると時間がかかります。数秒程度の差ですがキャッシュを使うことで短縮できます。

#### キャッシュ取得前
![](https://storage.googleapis.com/zenn-user-upload/7waih5q0yowk6bjm1ztmplh0u6b9)
#### キャッシュ取得後
![](https://storage.googleapis.com/zenn-user-upload/0eph9zjbzvfnylyh7jxid7rfeirv)

## 2023/2更新
今まで純正のreviewdogアクションを利用していましたが、textlintとセットになったアクションを提供している人がいらっしゃいましたのでこちらを利用することにしました。

https://github.com/tsuyoshicho/action-textlint

こちらのアクションはvimのLinterツールである[Vint](https://github.com/Vimjas/vint)が含まれる[reviewdogアクション](https://github.com/reviewdog/action-vint)をベースにしたアクションで、PRへのレビュー体験が向上したものになります。
まれに純正のreviewdogアクションでtextlintの実行結果がPRコメントに通知されないことがありましたので、こちらをしばらく試してみます。
# デモ実行
実際に`git push`して`develop`ブランチから`main`ブランチにPRし、何かしらの表現ミスがありましたら、reviewdogがこのようにレビューコメントを出してくれます。
![](https://storage.googleapis.com/zenn-user-upload/ybiyeuxyl6nxr41xt8eizy7qf2h7)
こんな感じで、PRして`git push`するたびにGitHub Actionsが動いてドキュメント校正してくれます。
1個前の記事からリポジトリで記事を管理しましたので、そちらもtextlintをかけてみましたが、結構指摘箇所がありました。
![](https://storage.googleapis.com/zenn-user-upload/i5lwtmyrkls9t3kf0hswh4wvgwj0)
https://zenn.dev/yuta28/articles/first-article-by-cli-yuta

# 所感
もともとCircleCI実践入門本でつまずいた部分でしたが、知り合いの力を借りて何とか解決できたところです。
(詳細はこちらのスクラップ参照)
https://zenn.dev/yuta28/scraps/221c80e7c07172
身近でCIを試せるものはないかと検討して、ブログ記事の自動校正化にチャレンジしてみようと思いました。
まだ記事のリポジトリ移管が進んでいませんが、移管後にドキュメント校正をして、ついでに文章の見直しをしようと思います。

https://github.com/Yuhta28/zenn-blog

# LT資料
LTのスライド資料です。
@[speakerdeck](d37f3955e3a143e1a20fb26da3831b31)

# 参考文献
https://gihyo.jp/book/2020/978-4-297-11411-4
https://swet.dena.com/entry/2018/09/18/142413
https://dev.classmethod.jp/articles/shuntaka-github-actions-reviewdog/
https://www.npmjs.com/package/textlint-rule-preset-smarthr
https://zenn.dev/tsuyoshicho/articles/2021-04-20-action-textv3
