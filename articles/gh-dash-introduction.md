---
title: "PRやissuesの一覧をターミナル上でイイ感じにまとめるgh dashを使ってみた"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github","githubcli","ghdash"]
published: true
---

# 概要
GitHub社は2020年にGitHubをコマンドライン上から操作できるGitHub CLIをリリースしました。
https://github.blog/2020-02-12-supercharge-your-command-line-experience-github-cli-is-now-in-beta/

その後2021年にGitHub CLI 2.0をリリースしました。
https://github.blog/2021-08-24-github-cli-2-0-includes-extensions/

2.0では拡張機能をサポートしており、誰でもカスタムコマンドを作成できます。 `gh extension browse`もしくは`gh extension search`で他の人が作成し公開した拡張機能を表示させます。

- gh extension brose

![](/images/gh-dash-introduction/image1.png)

- gh extension search

```terminal:
Showing 30 of 336 extensions

   REPO                          DESCRIPTION
✓  dlvhdr/gh-dash                A beautiful CLI dashboard for GitHub 🚀
   seachicken/gh-poi             ✨ Safely clean up your local branches
   jrnxf/gh-eco                  🦎 gh cli extension to explore the ecosystem
   github/gh-actions-importer    GitHub Actions Importer helps you plan and automate the migration of Azure DevOps, C...
   yusukebe/gh-markdown-preview  GitHub CLI extension to preview Markdown looks like GitHub.
   gennaro-tedesco/gh-s          🔎 search github repositories interactively
   github/gh-gei                 Migration CLI for GitHub to GitHub migrations
   actions/gh-actions-cache      A GitHub (gh) CLI extension to manage the GitHub Actions caches being used in a GitH...
   github/gh-net                 A network bridge between a Codespace and a local machine.
   mislav/gh-branch              GitHub CLI extension for fuzzy finding, quickly switching between and deleting branc...
   gennaro-tedesco/gh-f          🔎 the ultimate compact fzf gh extension
   Link-/gh-token                Create an installation access token for a GitHub app from your terminal 💻
   k1LoW/gh-grep                 :octocat: Print lines matching a pattern in repositories using GitHub API
   davidraviv/gh-clean-branches  Safely deletes local branches with no upstream and no un-pushed commits
   vilmibm/gh-screensaver        full terminal animations
   redraw/gh-install             install GitHub release binaries from the CLI interactively
   meiji163/gh-notify            GitHub CLI extension to display GitHub notifications
   KOBA789/gh-sql                Query GitHub Projects with SQL
   advanced-security/gh-sbom     Generate SBOMs with gh CLI
   kawarimidoll/gh-graph         GitHub contribution graph in your terminal
   korosuke613/gh-user-stars     Displays an interactive and searchable list of your GitHub starred repositories
   chelnak/gh-changelog          A gh cli extension that will make your changelogs ✨
   matt-bartel/gh-clone-org      A github-cli extension script to clone all repositories in an organization, optional...
   rnorth/gh-combine-prs         A `gh` extension for combining multiple PRs (e.g. Dependabot PRs) into one.
   mislav/gh-cp                  GitHub CLI extension to copy a file from a GitHub repository locally without cloning...
   gabe565/gh-profile            :octocat: Use multiple GitHub accounts with the gh cli.
   github/gh-projects            gh-projects
   vilmibm/gh-user-status        set and get github user statuses
```

このリストの並びはデフォルトでスターの数が多い順に表示されているのですが、トップの`dlvhdr/gh-dash`は2位と比べて10倍近い差をつけてスターを獲得しています。
- gh-dash
![](/images/gh-dash-introduction/image2.png)

- gh-poi
![](/images/gh-dash-introduction/image3.png)

このgh-dashとはどんな機能なのかインストールして試してみます。

# gh-dash
https://github.com/dlvhdr/gh-dash

`gh-dash`はGitHubのPRやissuesの一覧をターミナル上にダッシュボードで表示させてくれます。
![](/images/gh-dash-introduction/demo1.gif)
*READMEより[^1]*

[^1]: https://github.com/dlvhdr/gh-dash/blob/main/README.md

## インストール方法
以下のコマンドでインストールできます。

```terminal
gh extension install dlvhdr/gh-dash
```
ただデモ動画のようにアイコンをターミナル上に表示させる場合、Nerd Fontsをインストールする必要があります。
https://www.nerdfonts.com/

:::message
Windowsユーザーでしたら以前私が書いたこの記事も参考にインストールしてみてください。
https://zenn.dev/yuta28/articles/windows-git-dev
:::

## 使い方
Nerd Fontsのインストールまで完了しましたら、`gh dash`コマンドを実行することで自身のGitHubのPRやissuesのリストをダッシュボード上に表示できます。

![](/images/gh-dash-introduction/image4.png)
*？ボタンでコマンドヘルプ表示*

## Configuring
`gh-dash`の特徴として自分でダッシュボードを自由にカスタマイズできます。インストール時に`~/.config/gh-dash/config.yml`が作成され、引数なしで実行した場合このファイルを自動的に読み込みます。

```yml:config.yml
prSections:
- title: My Pull Requests
  filters: is:open author:@me
- title: Needs My Review
  filters: is:open review-requested:@me
- title: Involved
  filters: is:open involves:@me -author:@me
issuesSections:
- title: My Issues
  filters: is:open author:@me
- title: Assigned
  filters: is:open assignee:@me
- title: Involved
  filters: is:open involves:@me -author:@me
defaults:
  preview:
    open: true
    width: 50
  prsLimit: 20
  issuesLimit: 20
  view: prs
  layout:
    prs:
      updatedAt:
        width: 7
      repo:
        width: 15
      author:
        width: 15
      assignees:
        width: 20
        hidden: true
      base:
        width: 15
        hidden: true
      lines:
        width: 16
    issues:
      updatedAt:
        width: 7
      repo:
        width: 15
      creator:
        width: 10
      assignees:
        width: 20
        hidden: true
  refetchIntervalMinutes: 30
keybindings:
  issues: []
  prs: []
repoPaths: {}
pager:
  diff: ""
```

`prSections`がPRで`issuesSections`がissuesのダッシュボード構成になります。`title`セクションにダッシュボード上の項目名、`filters`セクションにどのリポジトリか、起案中かクローズ済みか、特定作者のみを表示させるか除外させるかなどルール付けを設定できます。
またconfig.ymlは明示的に指定でき、config.ymlを複数作成することで仕事用と個人用で複数のダッシュボードを用意できます。


```yml:config.yml
prSections:
- title: My Pull Requests
  filters: is:open author:@me
- title: Needs My Review
  filters: is:open review-requested:@me
- title: Involved
  filters: is:open involves:@me -author:@me
- title: Subscribed #自分以外のGitHub CLIやgh-dashのPRを表示
  filters: is:open -author:@me repo:cli/cli repo:dlvhdr/gh-dash
  limit: 50 # optional limit of rows fetched for this section
issuesSections:
- title: My Issues
  filters: is:open author:@me
- title: Assigned
  filters: is:open assignee:@me
- title: Involved
  filters: is:open involves:@me -author:@me
- title: Subscribed
  filters: is:closed -author:@me repo:cli/cli #クローズ済みのGitHub CLIのissuesを表示
```

```terminal:特定ダッシュボードを表示させたい場合
gh dash -c <path/to/config.yml>
```

- GitHub CLIリポジトリのクローズされてるissuesのみを表示
![](/images/gh-dash-introduction/image5.png)

## テーマカスタマイズ
`theme`セクションで自由にダッシュボードの文字色や背景色をカスタマイズできます。

```yml:カスタム例
theme:
  colors:
    text:
      primary: "ff7f00"
      secondary: "#666CA6"
      inverted: "#242347"
      faint: "#3E4057"
      warning: "#F23D5C"
      success: "#3DF294"
    background:
      selected: "#39386B"
    border:
      primary: "#383B5B"
      secondary: "#39386B"
      faint: "#2B2B40"
```

- デフォルトテーマ
![](/images/gh-dash-introduction/image6.png)

- カスタマイズテーマ
![](/images/gh-dash-introduction/image7.png)

前述のように仕事用と個人用のリポジトリで分けてカスタマイズすればダッシュボードの取り違いを減らせると思います。

# 所感
GitHub CLIの拡張機能で最もスターが多いgh-dashについて紹介しました。
本当はキーバインド機能も紹介したかったのですが、やり方が悪かったのかうまく試せませんでした。
GitHub CLIの登場以降どんどんGitHub上の操作がコマンドライン上で完結できるようになったと感じます。ブラウザとターミナル間を何回も行き来するのは地味にストレスでしたので開発体験も向上すると思います。
GitHub CLIの拡張機能は他にも面白いものがありますので気になりましたら`gh extension browse`で探してみてください。

# 参考文献
https://zenn.dev/aiiro/articles/8cf5d1ef6443ce
https://cli.github.com/manual/gh_extension_search
https://cli.github.com/manual/gh_extension_browse