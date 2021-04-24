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
そこで今回は、GitHub Actionsとtextlintとreviewdogを使うことで、PRするたびに自動ドキュメント校正してくれるCI環境の構築してみました。
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

```text:.textlintrc
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

```text:.textlintrc
{
    "rules": {
        "preset-smarthr":{
            "sentence-length": false
        },
        "prh": {
            "rulePaths": [
                "dict/smarthr-prh-basic.yml",
                "dict/smarthr-prh-tech-word.yml"
            ]
        }
    }
}
```

`textlint-rule-prh`の独自辞書の中身も公開してくれましたのでこちらも使ってみました。
https://github.com/kufu/textlint-rule-preset-smarthr/tree/main/dict
みなさんが使う場合は、ローカルのPCにSmart HRさんの辞書を格納しておいてください。
(私はdictディレクトリを作成してその中に置きました)

## reviewdog
`reviewdog`はtextlintなどLinterツールの結果を、GitHubのPRにレビューコメントを投稿してくれるGoで記述されたOSSツールです。
https://github.com/reviewdog/reviewdog

紹介した２つのツールを使うことで、ローカルで書く→`git push`→PRする→GtiHub Actionsが動くというCIが構築され、自動でドキュメントの校正が始まります。

それではGitHub Actionsのワークフローファイルの中身を紹介していきます。

# textlint.yml

```yml:textlint.yml
# This is a basic workflow to help you get started with Actions

name: reviewdog

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  pull_request:
    branches: [ main ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  reviewdog-github-check:
    name: reviewdog (github-check)
    runs-on: ubuntu-latest

    steps:
        #reviewdogのアクション
      - uses: reviewdog/action-setup@v1
        with:
          reviewdog_version: latest

        #textlintを動かすためのnodeアクション
      - uses: actions/setup-node@v2

      - uses: actions/checkout@v2

      - name: cache-node-modules
        #stepsが失敗(文章の乱れ)した場合でもcacheを取得するようにする
        uses: pat-s/always-upload-cache@v2.1.3
        env:
          cache-name: cache-node-modules
        with:
          path: ~/.npm
          key: node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            node-

      - name: Install textlint
        run:  'npm install --save-dev textlint textlint-rule-preset-smarthr textlint-rule-prh'
      
      - name: Install dependent module
        run: npm install
      
      - name: Execute textlint for ブログ記事
        run: |
          npx textlint -f checkstyle "articles/*.md" >> .textlint.log

      - name: Run reviewdog
        # textlintで文章上のミスがあった場合のみ、reviewdogを実行させるようにする
        if: failure()
        env:
          REVIEWDOG_GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          cat .textlint.log | reviewdog -f=checkstyle -name="textlint" -reporter="github-pr-review"
```

中身について解説します。
# ワークフローファイル解説
大まかなステップの流れとしては次の通りです。

- reviewdog,nodeのアクションを起動
- textlintのインストール
- textlintを記事に対して実行
- 何かしらの記載ミスが有った場合のみtextlintの出力結果をreviewdogに渡してGitHubにレビューコメントを投稿

工夫した点としてcacheアクションを使うことで、ワークフローの高速化を図りました。
ところがGitHub Actionsが提供している[actions/cache](https://github.com/actions/cache)ですが、stepsが失敗しているとキャッシュを取得しないという仕様があります。
https://github.com/actions/cache/blob/main/action.yml#L24
この仕様のおかげでドキュメントを校正し、textlintで記載ミスがあるとキャッシュを取得してくれず、毎回textlintのインストールを最初からインストールし直します。
![](https://storage.googleapis.com/zenn-user-upload/umcoopmh5z5ix9b55qcy22hp6j29)
調べてみると有志で`actions/cache`をforkしてstepsが失敗してもキャッシュを取得するようにしたアクションを公開してくれる人がいましたので、それを使うことにしました。
https://github.com/pat-s/always-upload-cache
こちらを使うことで、stepsが失敗してもキャッシュを取得してくれてワークフローの処理を速めることができました。

#### キャッシュ取得前
![](https://storage.googleapis.com/zenn-user-upload/7waih5q0yowk6bjm1ztmplh0u6b9)
#### キャッシュ取得後
![](https://storage.googleapis.com/zenn-user-upload/0eph9zjbzvfnylyh7jxid7rfeirv)

またtextlintで特に問題が見つからない場合、reviewdogを実行するのは無駄なstepsになりますので、`if: failure()`で前述のtextlintで異常が見つかった場合のみ実行するようにしました。

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
この文章は敬体です。この文章は常態である。
https://github.com/Yuhta28/zenn-blog

# 参考文献
https://gihyo.jp/book/2020/978-4-297-11411-4
https://swet.dena.com/entry/2018/09/18/142413
https://dev.classmethod.jp/articles/shuntaka-github-actions-reviewdog/
https://www.npmjs.com/package/textlint-rule-preset-smarthr

# LT資料
LTのスライド資料です。
@[speakerdeck](d37f3955e3a143e1a20fb26da3831b31)
