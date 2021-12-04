---
title: "PythonではなくRcloneでGoogle共有ドライブにファイルをアップロードしてみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googledrive","rclone"]
published: false
---

# 概要
会社からの要望でEC2内で生成されたファイルを、会社のGoogle共有ドライブに格納してほしいという依頼がありました。
PythonでGoogleDriveAPIを操作してファイルアップロードに挑戦しましたが、うまくいかなかったのでRcloneを使ってファイルアップロードしましたのでその方法を紹介します。

# Pythonで失敗
先にPythonでうまくいかなかったことについて共有します。
元々PythonにはPyDriveというGoogleドライブとの連携を簡単に実現するライブラリがありました。
https://github.com/googlearchive/PyDrive
ただ上記のGitHubにも記載があるとおり、開発が終了し`Deprecated`になっていますので、できれば他の方法がないか探していました。
そこで直接GoogleDriveのAPIをPythonを使って操作して、ファイルアップロードを実現しようと考えました。

## 実施手順
具体的な実装方法はGoogle公式ドキュメントにもサンプルソースがありますし、手順をわかりやすく説明してくれるブログもあります。
https://developers.google.com/drive/api/v3/quickstart/python
https://zenn.dev/wtkn25/articles/python-googledriveapi-auth
