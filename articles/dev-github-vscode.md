---
title: "dev.toの記事もGitHubで管理してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["devto","GitHub","GitHubActions","vscode"]
published: false
---

# 概要

私はZenn以外にも[dev.to](https://dev.to/yuta28)でもブログを書いています。
基本的にZennの記事を英訳してdev.toに投稿しており、Zennのようにdev.toの記事もGitHubで管理して、可能ならGitHub Actionsを使って自動で投稿できるようにしたいと考えていました。
調べると、dev.toでまさに私がやりたかったdev.toの記事をGitHubで管理する方法を紹介している人がいました。

https://dev.to/maxime1992/manage-your-dev-to-blog-posts-from-a-git-repo-and-use-continuous-deployment-to-auto-publish-update-them-143j?signin=true

またdev.toは[API](https://developers.forem.com/api)が公開されていて、記事の投稿もAPIを使うことで、GitHub Actionsで自動投稿できるみたいです。
https://dev.to/beeman/automate-your-dev-posts-using-github-actions-4hp3

これらの記事を参考にしてdev.toの記事もGitHubで管理してみました。

# 実装

基本的に上の記事の手順を訳して進めますが、一部アレンジした部分もありますのでそこを詳しく説明します。
## 1. テンプレートコピー
記事作成者のGitHubリポジトリテンプレートをコピーします。
https://github.com/maxime1992/dev.to

![](/images/dev-github-vscode/image1.png)

コピーできましたら、自分のGitHubにも同じリポジトリが生成されます。
ただテンプレートにある`package.json`のYarnパッケージが古いので、`yarn upgrade --latest`でパッケージを更新します。
## 2. dev.to APIキー作成

dev.toの設定画面からAPIキーを作成します。 ![](/images/dev-github-vscode/image2.png)

## 3. APIキーをGitHubに登録
記事ではTravisを使って、CIを実装していましたが私はGitHub Actionsを使ってCI/CD実装しました。
リポジトリの設定ページからGitHub Actionsで使用するシークレットに環境変数を利用できるRepository Secretを登録します。 ![](/images/dev-github-vscode/image3.png)

## 4. GitHub Actions ワークフロー設定
下記記事では1ジョブで文書構成とdev.toへの記事投稿を同時に行なっていますが、私はジョブを複数に分けてPR時に文書校正させるジョブを実行し、マージ時にdev.toへの記事投稿を行なうジョブを実行させるようにしました。
https://dev.to/beeman/automate-your-dev-posts-using-github-actions-4hp3

元の記事ではPrettierを使ったコードフォーマットのみでしたが、ZennブログのGitHubで管理しているときに使用している、textlintとreviewdogも組み入れました。詳細はこちらの記事をご覧ください。
https://zenn.dev/yuta28/articles/blog-lint-ci-reviewdog

```yml: build.yml
name: Build and Deploy

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    name: Build
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repo
        uses: actions/checkout@master

      - name: Install reviewdog
        uses: reviewdog/action-setup@v1
        with:
          reviewdog_version: latest

      - name: Install Dependencies
        uses: bahmutov/npm-install@v1

        #Make sure that cache is retrieved even if steps fail
      - name: cache-node-modules
        uses: pat-s/always-upload-cache@v3
        env:
          cache-name: cache-node-modules
        with:
          path: ~/.npm
          key: node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            node-

      - name: Install textlint
        run: 'npm install --save-dev textlint textlint-rule-common-misspellings textlint-rule-spellchecker'
      
      - name: Run textlint
        run: npx textlint -f checkstyle "blog-posts/**/*.md" >> .textlint.log

      - name: Run reviewdog
        if: failure()
        env:
          REVIEWDOG_GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: cat .textlint.log | reviewdog -f=checkstyle -name="textlint" -reporter="github-pr-review"

      - name: Run Prettier
        run: yarn run prettier:check

      - name: Run Embedme
        run: yarn run embedme:check

  deploy:
    name: Deploy
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repo
        uses: actions/checkout@master

      - name: Install Dependencies
        uses: bahmutov/npm-install@v1

      - name: Deploy to dev.to
        run: DEV_TO_GIT_TOKEN=${{ secrets.DEV_TO_GIT_TOKEN }} yarn run dev-to-git
```

## 5. 新規記事作成
新規記事の作成ですが、3パターンあります。

- ブラウザから登録する
- APIから登録する
- VScodeのプラグインから登録する

それぞれの方法を紹介しますので、好みの方法で作成してください。

### 5-1. ブラウザから登録する

これは単純にdev.toのページから記事作成ボタンを押すだけです。![](/images/dev-github-vscode/image4.png)
### 5-2. APIから登録する

dev.toのAPIには記事の新規作成APIがありますので、curlで叩いて記事を作成します。
(`API_KEY`に先ほど生成したAPIキーを入れてください)

```bash: Create a new article
curl -X POST -H "Content-Type: application/json" \
  -H "api-key: API_KEY" \
  -d '{"article":{"title":"Title","body_markdown":"Body","published":false,"tags":["discuss", "javascript"]}}' \
  https://dev.to/api/articles
```
### 5-3. VScodeのプラグインから登録する

VS Codeのプラグインから記事を新規作成します。
https://marketplace.visualstudio.com/items?itemName=sneezry.vscode-devto

+ボタンから新規記事を作成できます。この方法ですと`front matter`とよばれるメタデータのテンプレートも作成してくれます。 ![](/images/dev-github-vscode/image5.png)

個人的にはメタデータを作成してくれるプラグインからの新規作成がおすすめです。

::: message
APIでもできるはずなんですが、私の環境だとエラーになりました。
```bash
$ curl -X POST -H "Content-Type: application/json" \
-H "api-key: API_KEY" \
-d '{"article":{"title":"Test-Tile","body_markdown":"---\ntitle:A sample article about...\npublished:false\n---\n...","published":false,"tags":["discuss", "javascript"]}}' https://dev.to/api/articles

{"error":"(\u003cunknown\u003e): could not find expected ':' while scanning a simple key at line 3 column 1","status":422}
```
メタデータを明示するハイフン`---`が悪さをしているみたいで、ハイフンを消すと作成できます。
冒頭のハイフンはエスケープすることで回避できましたが、末尾のハイフンはエスケープができませんでした。
私の方では原因がわからずお手上げです。
:::

## 6. 記事IDを取得
後々使う記事の固有IDを取得します。元の記事ではブラウザの開発モードで取得していますが、APIでも取得できますので、APIを使って取得する方法を紹介します。

```bash: User's unpublished articles
$ curl -H "api-key: API_KEY" https://dev.to/api/articles/me/unpublished | jq '.[].id'
1064058
```

# 参考文献

https://developers.forem.com/api
https://zenn.dev/hiroga/articles/manage-dev-to-by-git