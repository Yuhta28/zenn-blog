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
