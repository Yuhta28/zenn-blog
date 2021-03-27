---
title: "GitHub Actionsでzennブログの校正を自動化してみた"
emoji: "🐀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [GitHubActions,reviewdog,Zenn,CI]
published: false
---
# 概要
前回の記事でも紹介したとおり、ZennはGitHubリポジトリと連携すると、自分の好きなエディタで記事を書くことができます。
https://zenn.dev/yuta28/articles/first-article-by-cli-yuta
ほかにもGitHubを連携することでCIツールを導入することが可能となり、記事作成の助けにもなります。
そこで今回は、GitHub Actionsとtextlintとreviewdogを使うことで、PRする度に自動ドキュメント校正してくれるCI環境の構築してみました。
# ドキュメント校正ツール
今回GitHub Actionsに組み込む校正ツールは以下の２つを使用します。

- textlint
- reviewdog

それぞれのツールについて簡単にはなりますが、紹介致します。
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

## reviewdog
`reviewdog`はtextlintなどLinterツールの結果を、GitHubのPRにレビューコメントを投稿してくれるGoで記述されたOSSツールです。
https://github.com/reviewdog/reviewdog

紹介した２つのツールを使うことで、ローカルで書く→`git push`→PRする→GtiHub Actionsが動くというCIが構築され、自動でドキュメントの校正が始まります。

それではGitHub Actionsのワークフローファイルの中身を紹介していきます。

### textlint.yml

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
        run:  'npm install --save-dev textlint textlint-rule-preset-ja-technical-writing textlint-rule-prh'
      
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
### ワークフローファイル解説
大まかなステップの流れとしては次の通りです。

- reviewdog,nodeのアクションを起動
- textlintのインストール
- textlintを記事に対して実行
- 何かしらの記載ミスが有った場合のみtextlintの出力結果をreviewdogに渡してGitHubにレビューコメントを投稿

