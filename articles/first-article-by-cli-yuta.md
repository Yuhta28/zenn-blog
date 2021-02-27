---
title: "zenn-cliで記事作成してみた"
emoji: "🐭"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [初心者,Zenn]
published: false
---
# 概要
先日行われたDevelopers Summit 2021でZennの開発者である、catnose氏がZennの開発秘話を話されました。
https://event.shoeisha.jp/devsumi/20210218/session/3082/
Zennを個人で開発されており、個人開発の苦労話やZennの機能紹介など大変おもしろい内容でした。
その中で、GitHubリポジトリとZennの連携とzenn-cliを使うことでローカルエディターでZennの記事作成ができるという話がありましたので、早速一個お試しで作成してみました。

# GitHubリポジトリとの連携
具体的な方法については開発者様の記事をご覧になればわかると思いますので詳しい手順については割愛します。
https://zenn.dev/zenn/articles/connect-to-github
こんな感じで私のGitHubリポジトリとZennを連携することで記事の管理ができるようになります。
![](https://storage.googleapis.com/zenn-user-upload/no78w5u2tf85f0vjed3i1183uf7z)

# zenn-cliの導入
同じく具体的な手順については開発者様の記事を参照
https://zenn.dev/zenn/articles/zenn-cli-guide

` npx zenn preview`コマンドを実行することで`http://localhost:8000`でプレビュー画面が立ち上がり、プレビューしながらローカルエディターから執筆することができます。

### 下書き画面の様子
![](https://storage.googleapis.com/zenn-user-upload/vvj51qf1nkj3vbcq6z08cuz6x2ji)

```yaml
---
title: "zenn-cliで記事作成してみた" #title
emoji: "🐁" #icon
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [初心者,Zenn]
published: false
---
```

zenn-cliの使い方記事で紹介されている通り、`npx zenn new:article`で、ファイル内に上記のヘッダー設定が記述された記事作成用のMarkdownファイルが作成されます。
ここでタイトルや記事に含まれているトピックの設定、Zennの記事カテゴリである、Tech系かIdea系の分類を決めることができます。
**published**の部分ですが、ここを`true`にすることで、実際に記事を全体に公開することができます。
