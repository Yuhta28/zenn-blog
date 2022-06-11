---
title: "dev.toの記事もGitHubで管理してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["devto","GitHub","GitHubActions","vscode"]
published: true
---

# 概要

私はZenn以外にも[dev.to](https://dev.to/yuta28)でもブログを書いています。
基本的にZennの記事を英訳してdev.toに投稿しており、Zennのようにdev.toの記事もGitHubで管理して、可能ならGitHub Actionsを使って自動で投稿できるようにしたいと考えていました。
調べると、dev.toでまさに私がやりたかったdev.toの記事をGitHubで管理する方法を紹介している人がいました。

https://dev.to/maxime1992/manage-your-dev-to-blog-posts-from-a-git-repo-and-use-continuous-deployment-to-auto-publish-update-them-143j?signin=true

またdev.toは[API](https://developers.forem.com/api)が公開されていて、記事の投稿もAPIを使うことで、GitHub Actionsで自動投稿できるみたいです。
https://dev.to/beeman/automate-your-dev-posts-using-github-actions-4hp3

私も記事作成者の[GitHubリポジトリ](https://github.com/maxime1992/dev.to)をコピーして、それを基にdev.toの記事管理専用のリポジトリを作成しました。
https://github.com/Yuhta28/dev-to-blog-template

記事の手順を基に自分なりにアレンジしましたので私のdev.toの記事管理の方法について紹介いたします。
# 実装

## 1 テンプレートコピー
GitHubリポジトリテンプレートをコピーします。
https://github.com/Yuhta28/dev-to-blog-template ![](/images/dev-github-vscode/image1.png)

コピーできましたら、自分のGitHubにも同じリポジトリが生成されます。
念のためテンプレートにある`package.json`のYarnパッケージを、`yarn upgrade --latest`で更新します。
## 2 DEV Community APIキー作成

dev.toの設定画面からAPIキーを作成します。 ![](/images/dev-github-vscode/image2.png)

## 3 APIキーをGitHubに登録
元記事ではTravisを使って、CIを実装していましたが私はGitHub Actionsを使ってCI/CDを実装しました。
リポジトリの設定ページからGitHub Actionsで使用するシークレットに環境変数を利用できるRepository Secretを登録します。 ![](/images/dev-github-vscode/image3.png)

## 4 GitHub Actions ワークフロー設定
下記記事では1ジョブで文書校正とdev.toへの記事投稿を同時に行なっていますが、私はジョブを複数に分けてPR時に文書校正させ、マージ時にdev.toへの記事投稿を行なうようにしました。
https://dev.to/beeman/automate-your-dev-posts-using-github-actions-4hp3

元の記事ではPrettierを使ったコードフォーマットのみでしたが、ZennブログのGitHubで管理しているときに使用している、textlintとreviewdogも採用しました。詳細はこちらの記事をご覧ください。
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

## 5 GitHubとdevの連携

直下の`package.json`内にdev.toの記事を管理するリポジトリのURLを記載します。

```json: package.json
  "name": "dev.to",
  "repository": {
    "type": "git",
    "url": "https://github.com/Yuhta28/dev-to-blog.git"
  }
```

## 6 新規記事作成
記事IDを取得するために、先にdev.toで新規記事の作成を行なう必要があります。
新規記事の作成ですが、3パターンあります。

- ブラウザから登録する
- APIから登録する
- VScodeのプラグインから登録する

それぞれの方法を紹介しますので、好みの方法で作成してください。

### 6-1 ブラウザから登録する

これは単純にdev.toのページから記事作成ボタンを押すだけです。![](/images/dev-github-vscode/image4.png)
### 6-2 APIから登録する

dev.toのAPIには記事の新規作成APIがありますので、curlで叩いて記事を作成します。
(`API_KEY`に先ほど生成したAPIキーを入れてください)

```bash: Create a new article
curl -X POST -H "Content-Type: application/json" \
  -H "api-key: API_KEY" \
  -d '{"article":{"title":"Title","body_markdown":"Body","published":false,"tags":["discuss", "javascript"]}}' \
  https://dev.to/api/articles
```

毎回curlコマンドを叩くのが面倒でしたらGoで同じようなことができるソースコードを作成しましたので、`DEVAPIKEY`という環境変数に先ほど取得したAPIキーをセットして実行してみてください。
https://github.com/Yuhta28/dev-to-blog-template/blob/main/create-post.go

### 6-3 VScodeのプラグインから登録する

VS Codeのプラグインから記事を新規作成します。
https://marketplace.visualstudio.com/items?itemName=sneezry.vscode-devto

+ボタンから新規記事を作成できます。この方法ですと`front matter`とよばれるメタデータのテンプレートも作成してくれます。 ![](/images/dev-github-vscode/image5.png)


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

## 7 記事IDを取得
記事を作成しましたらIDを取得します。元の記事ではブラウザの開発モードで取得していますが、APIでも取得できますので、APIを使って取得する方法を紹介します。

```bash: User's unpublished articles
$ curl -H "api-key: API_KEY" https://dev.to/api/articles/me/unpublished | jq '.[].id'
1064058
```

こちらも同じようにGoでIDを取得できるソースコードを作成しました。
curlコマンドを書くのが面倒でしたらお試しください。
https://github.com/Yuhta28/dev-to-blog-template/blob/main/get-blog-id.go

## 8 記事フォルダの作成

`blog-posts`ディレクトリ直下が記事フォルダとなります。
テンプレートからコピーしたら`template-posts`ディレクトリがすでにありますので、これを同じ階層に新規コピーします。
私のディレクトリではこのようになっています。

```powershell
$ tree blog-posts/
blog-posts/
├── be-attention-to-sshkey
│   ├── assets
│   ├── be-attention-to-replace-a-key-pair-for-ec2-4e3j.md
│   └── code
├── cw-oss-cloudwatch
│   ├── assets
│   ├── code
│   └── i-use-cw-which-is-oss-to-tail-aws-cloudwatch-logs-2e9g.md
├── diffrence-ec2-risp-and-rds-ri
│   ├── assets
│   ├── code
│   └── difference-between-ec2-ri-sp-and-rds-ri-1ad.md
├── gha-with-oidc-terraform
│   ├── assets
│   ├── code
│   └── github-actions-cicd-without-aws-credential-information-4no6.md
├── keep-in-mind-coding-existing-infra
│   ├── assets
│   ├── code
│   └── keep-in-mind-when-coding-existing-infrastructure-15b3.md
├── repography-make-readme-rich
│   ├── assets
│   │   ├── image1.png
│   │   └── image2.png
│   ├── code
│   └── repography-makes-github-repository-beautiful-3dn3.md
└── template-posts
    ├── assets
    ├── code
    └── template-blog.md

21 directories, 9 files
```
::: message
###### assests/codeディレクトリ
記事内に画像やソースコードを挿入したいときはこの中にファイルを置いて相対リンクで記事に埋め込みます。
:::

Goでテンプレートフォルダを作成するソースコードはこちらです。`go run make-template.go`で実行し、任意の名称で記事フォルダを作成できます。
https://github.com/Yuhta28/dev-to-blog-template/blob/main/make-template.go

## 9 記事リンクの連携

記事フォルダの作成と記事IDを取得できましたら、2つを紐づける作業があります。
`dev-to-git.json`にIDと記事フォルダにある記事ファイルのパスを記載します。

```json: dev-to-git.json
[
  {
    "id": 773216,
    "relativePathToArticle": "./blog-posts/cw-oss-cloudwatch/i-use-cw-which-is-oss-to-tail-aws-cloudwatch-logs-2e9g.md"
  },
  {
    "id": 784222,
    "relativePathToArticle": "./blog-posts/be-attention-to-sshkey/be-attention-to-replace-a-key-pair-for-ec2-4e3j.md"
  },
  {
    "id": 806920,
    "relativePathToArticle": "./blog-posts/diffrence-ec2-risp-and-rds-ri/difference-between-ec2-ri-sp-and-rds-ri-1ad.md"
  },
  {
    "id": 890855,
    "relativePathToArticle": "./blog-posts/gha-with-oidc-terraform/github-actions-cicd-without-aws-credential-information-4no6.md"
  },
  {
    "id": 964471,
    "relativePathToArticle": "./blog-posts/keep-in-mind-coding-existing-infra/keep-in-mind-when-coding-existing-infrastructure-15b3.md"
  },
  {
    "id": 1056501,
    "relativePathToArticle": "./blog-posts/repography-make-readme-rich/repography-makes-github-repository-beautiful-3dn3.md"
  }
]
```
## 10 記事投稿
これでGitHubとdev.toの記事連携は完了です。
ブランチを切って、`git push`してmainブランチにマージすれば文書校正のCIが走ります。 ![](/images/dev-github-vscode/image6.png)
textlintかPretterで警告が出たらジョブが失敗しますので、修正とCIを繰り返すことで文章の校正が進みます。
CIが問題なく完了しましたらマージしてデプロイを走らせます。 ![](/images/dev-github-vscode/image7.png)

先ほどのメタデータで`published`が`true`になっている記事はこの段階でdev.toに公開されうようになります。

# 所感

dev.toの記事をGitHubで管理できるようにしました。
記事をGitHubで管理できるとバージョン管理が楽になりますし、ローカルの好みのエディターを使って記事を書くことができます。
私のようにLinterツールを使えば文章のケアレスミスも減らせますので、皆さんもdev.toで記事を書くときはぜひこの方法で試してみてください。

# LT資料

LTのスライド資料です。
@[speakerdeck](a008006a77104f1e9a5f9d975418b945)

# 英語記事

英訳しました。
https://dev.to/yuta28/i-manage-my-devto-blog-in-github-repository-5880

# 参考文献

https://developers.forem.com/api
https://zenn.dev/hiroga/articles/manage-dev-to-by-git