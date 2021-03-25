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
校正のためのルールを扱うプラグインが多数開発されており、自分自身で自由に校正のルールを決めて使用します。
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
    },
    "plugins": [
        "review"
    ]
}
```