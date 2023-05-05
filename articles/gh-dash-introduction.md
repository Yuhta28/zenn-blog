---
title: "GitHubの内容をターミナル上で表示させるgh dashを使ってみる"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github","githubcli"]
published: false
---

# 概要
GitHub社は2020年にGitHubをコマンドライン上から操作できるGitHub CLIをリリースしました。
https://github.blog/2020-02-12-supercharge-your-command-line-experience-github-cli-is-now-in-beta/

その後2021年にGitHub CLI 2.0をリリースしました。
https://github.blog/2021-08-24-github-cli-2-0-includes-extensions/

2.0では拡張機能をサポートしており、誰でもカスタムコマンドを作成できます。 `gh extension browse`もしくは`gh extension search`で他の人が作成し公開した拡張機能を表示させられます。

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

このリストの並びはデフォルトではスターの数が多い順に表示されているのですが、トップの`dlvhdr/gh-dash`は2位と比べて10倍近い差をつけてスターを獲得しています。
- gh-dash
![](/images/gh-dash-introduction/image2.png)

- gh-poi
![](/images/gh-dash-introduction/image3.png)

このgh-dashは一体どういう機能なのかインストールして試してみます。



# 参考文献
https://zenn.dev/aiiro/articles/8cf5d1ef6443ce