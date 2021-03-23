---
title: "zenn-cliで記事作成してみた"
emoji: "🐭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [初心者,Zenn,GitHub]
published: true
---
# 概要
先日行われたDevelopers Summit 2021でZennの開発者である、catnose氏がZennの開発秘話を話されました。
https://event.shoeisha.jp/devsumi/20210218/session/3082/
Zennを個人で開発されており、個人開発の苦労話やZennの機能紹介など大変おもしろい内容でした。
その中で、GitHubリポジトリとZennの連携、zenn-cliを使うことでローカルエディターでZennの記事作成ができるという話がありましたので、早速一個お試しで作成してみました。

# GitHubリポジトリとの連携
具体的な方法については開発者様の記事をご覧になればわかると思いますので詳しい手順については割愛します。
https://zenn.dev/zenn/articles/connect-to-github
こんな感じで私のGitHubリポジトリとZennを連携することで記事の管理ができるようになります。
![](https://storage.googleapis.com/zenn-user-upload/no78w5u2tf85f0vjed3i1183uf7z)

# zenn-cliの導入
同じく具体的な手順については開発者様の記事を参照
https://zenn.dev/zenn/articles/zenn-cli-guide

` npx zenn preview`コマンドを実行することで`http://localhost:8000`でプレビュー画面が立ち上がり、プレビューしながらローカルエディターで執筆することができます。

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

# デプロイ
デプロイできるブランチは一つだけですので、Zennの設定画面からデプロイ実行するブランチを決めて、該当のブランチで`git push`すれば自動デプロイされます。
![](https://storage.googleapis.com/zenn-user-upload/vc06ql2xnwo5n9tcgi2qbk4a9i1e)

# 使った感想
この記事がzenn-cliで書いた記事になります。
私はエディターでVisual Studio Codeを使用していますが、オンラインエディターと異なりプラグインを導入することで記事作成がしやすいエディターを使うことができて便利でした。
自分がタイピングした内容をリアルタイムでブラウザから確認することもでき、画像の挿入もZenn側でアップローダーがあるおかげで、自分でimgurから画像URLを作成する必要がなくまさに痒いところに手が届くいいサービスだと思いました。
2021年2月現在は既に記事で公開しているものをGitHubリポジトリで管理することはできず、記事内容をコピペしてローカルエディターに貼り付けるというやり方になるそうです。
その時に設置する記事のファイル名はURLから見れるこれです。
![](https://storage.googleapis.com/zenn-user-upload/bbgts9c54qos0cehtvipvlwelpyl)
ちなみに記事のMarkdownの名前ですが、Zenn内で一意である必要があります。
(Slugというそうです。詳しくはこちら⇓)
https://zenn.dev/zenn/articles/what-is-slug
既に公開している記事のSlugの変更はできないので、GitHubリポジトリで管理したい場合は、

1. `articles/books`ディレクトリ内にファイル名が公開済み記事のSlugであるMarkdownファイルを作成する。
2. 作成したMarkdownファイルに記事の内容をコピペする。
3. **published**を`true`にして`git push`でデプロイする。

これで完了です。今後はこちら中心に記事を書いていこうと思います。
