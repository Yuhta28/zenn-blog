---
title: "PythonではなくRcloneでGoogle共有ドライブにファイルをアップロードしてみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googledrive","rclone"]
published: false
---

# 概要
会社からの要望でEC2内で生成されたファイルを、会社のGoogle共有ドライブに格納してほしいという依頼がありました。
![](/images/rclone-upload-gdrive/image1.png)
PythonでGoogleDriveAPIを操作してファイルアップロードに挑戦しましたが、うまくいかなかったのでRcloneを使ってファイルアップロードしましたのでその方法を紹介します。

# Pythonで失敗
先にPythonでうまくいかなかったことについて共有します。
もともとPythonにはPyDriveというGoogleドライブとの連携を簡単に実現するライブラリがありました。
https://github.com/googlearchive/PyDrive
ただ上記のGitHubにも記載があるとおり、開発が終了し`Deprecated`になっていますので、できれば他の方法がないか探していました。
そこで直接GoogleDriveのAPIをPythonを使って操作して、ファイルアップロードを実現しようと考えました。

## 実施手順
具体的な実装方法はGoogle公式ドキュメントにもサンプルソースがありますし、手順をわかりやすく説明してくれるブログもあります。
https://developers.google.com/drive/api/v3/quickstart/python
https://zenn.dev/wtkn25/articles/python-googledriveapi-auth

まずはこの手順通りに進めて、OAuth認証を通して自分のマイドライブにファイルをアップロードできるか検証してみました。

上記ブログを参考にGCPコンソールからGoogleDriveAPIを有効化し、認証情報を取得してEC2内に格納しました。
その後、必要なライブラリをインストールして認証情報ファイルのあるディレクトリにファイルアップロードするPythonコードを書きます。

### ライブラリインストール

``` bash
 $ pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib oauth2client
 ```

### ディレクトリ構成

```bash
$ tree drive
drive
├── client_secret.json
├── file-upload.py
└── test.csv #アップロードファイル
```

### Pythonコード

```python:file-upload.py
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload
from oauth2client.service_account import ServiceAccountCredentials
import os

def uploadFileToGoogleDrive(fileName, localFilePath):
    service = getGoogleService()
        "parents": ["XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"] #マイドライブのfolder ID
    file_metadata = {"name": fileName, "mimeType": "text/csv", "parents": ["****"] }
    media = MediaFileUpload(localFilePath, mimetype="text/csv", resumable=True)
    file = service.files().create(body=file_metadata, media_body=media, fields='id').execute()

def getGoogleService():
    scope = ['https://www.googleapis.com/auth/drive.file']
    keyFile = 'client_secret.json'
    credentials = ServiceAccountCredentials.from_json_keyfile_name(keyFile, scopes=scope)
    return build("drive", "v3", credentials=credentials, cache_discovery=False)


getGoogleService()
uploadFileToGoogleDrive("test.csv", "./text.csv")
```

このPythonを実行してみて自分のドライブを確認したらファイルが正しくアップロードされることを確認しました。
次に共有ドライブの格納フォルダを作成し、そのfolder IDを指定した状態でPythonを実行したら次のようなエラーが出てファイルのアップロードに失敗しました。

```bash
googleapiclient.errors.HttpError:
<HttpError 404 when requesting https://www.googleapis.com/upload/drive/v3/files?fields=id&alt=json&uploadType=resumable
returned "File not found: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX".
Details: "[{'domain': 'global', 'reason': 'notFound', 'message': 'File not found: 1mE_qImJ7ASarUCE4XUkv9sVER4-sVJKM.', 'locationType': 'parameter', 'location': 'fileId'}]">
```

`File not found`の部分は共有ドライブのfolder IDになります。
IDをコピーしてファイルに貼り付けたのですが、どういうわけかそのフォルダが存在しないということでアップロードに失敗しました。
![](/images/rclone-upload-gdrive/image2.png)

# 参考文献
https://laboratory.kazuuu.net/upload-files-to-google-drive-with-python/