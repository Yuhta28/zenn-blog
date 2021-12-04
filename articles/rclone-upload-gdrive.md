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

色々と調べてみましたが、原因を特定できませんでした。
(もしかして共有ドライブの権限で許可されていないのかもしれません)

# RcloneでGoogleDriveへアップロード
どうしたものかと悩んでいましたが、OSSでRcloneというものの存在を見つけました。
https://rclone.org/
RcloneはGo製のOSSでGoogleDrive以外にもOneDrive、Dropboxなど40以上のクラウドストレージサービスと連携し、ローカル端末とクラウドストレージ間のファイル管理を実現できます。
ダウンロードページを確認するとMac,Linux以外にもWindows用のインストーラーも用意されていて色々とよさそうだったのでこちらを試してみることにしました。
![](/images/rclone-upload-gdrive/image3.png)
*https://rclone.org/downloads/*

## セットアップ
Rcloneをインストールしましたら初期設定として`rclone config`を実行します。

```powershell
 rclone config
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n #新規リモート作成を選択
name> googledrive #リモート名を入力(好きな名前でよい)
Option Storage.
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value.
 1 / 1Fichier
   \ "fichier"
 2 / Alias for an existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Providers including AWS, Alibaba, Ceph, Digital Ocean, Dreamhost, IBM COS, Minio, SeaweedFS, and Tencent COS
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Better checksums for other remotes
   \ "hasher"
 7 / Box
   \ "box"
 8 / Cache a remote
   \ "cache"
 9 / Citrix Sharefile
   \ "sharefile"
10 / Compress a remote
   \ "compress"
11 / Dropbox
   \ "dropbox"
12 / Encrypt/Decrypt a remote
   \ "crypt"
13 / Enterprise File Fabric
   \ "filefabric"
14 / FTP Connection
   \ "ftp"
15 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
16 / Google Drive
   \ "drive"
17 / Google Photos
   \ "google photos"
18 / Hadoop distributed file system
   \ "hdfs"
19 / Hubic
   \ "hubic"
20 / In memory object storage system.
   \ "memory"
21 / Jottacloud
   \ "jottacloud"
22 / Koofr
   \ "koofr"
23 / Local Disk
   \ "local"
24 / Mail.ru Cloud
   \ "mailru"
25 / Mega
   \ "mega"
26 / Microsoft Azure Blob Storage
   \ "azureblob"
27 / Microsoft OneDrive
   \ "onedrive"
28 / OpenDrive
   \ "opendrive"
29 / OpenStack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
30 / Pcloud
   \ "pcloud"
31 / Put.io
   \ "putio"
32 / QingCloud Object Storage
   \ "qingstor"
33 / SSH/SFTP Connection
   \ "sftp"
34 / Sia Decentralized Cloud
   \ "sia"
35 / Sugarsync
   \ "sugarsync"
36 / Tardigrade Decentralized Cloud Storage
   \ "tardigrade"
37 / Transparently chunk/split large files
   \ "chunker"
38 / Union merges the contents of several upstream fs
   \ "union"
39 / Uptobox
   \ "uptobox"
40 / Webdav
   \ "webdav"
41 / Yandex Disk
   \ "yandex"
42 / Zoho
   \ "zoho"
43 / http Connection
   \ "http"
44 / premiumize.me
   \ "premiumizeme"
45 / seafile
   \ "seafile"
Storage> drive #接続するドライブ先を選択
Option client_id.
Google Application Client Id
Setting your own is recommended.
See https://rclone.org/drive/#making-your-own-client-id for how to create your own.
If you leave this blank, it will use an internal key which is low performance.
Enter a string value. Press Enter for the default ("").
client_id> #OAuth認証用のクライアントIDを入力
Option client_secret.
OAuth Client Secret.
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret> #OAuth認証用のクライアントシークレットを入力
Option scope.
Scope that rclone should use when requesting access from drive.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value.
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 1 #ドライブへの権限を選択(1がフルアクセス権限)
Option root_folder_id.
ID of the root folder.
Leave blank normally.
Fill in to access "Computers" folders (see docs), or for rclone to use
a non root folder as its starting point.
Enter a string value. Press Enter for the default ("").
root_folder_id> #ルートフォルダーにしたいfolder IDを入力(ルートフォルダなら空白でOK)
Option service_account_file.
Service Account Credentials JSON file path.
Leave blank normally.
Needed only if you want use SA instead of interactive login.
Leading `~` will be expanded in the file name as will environment variables such as `${RCLONE_CONFIG_DIR}`.
Enter a string value. Press Enter for the default ("").
service_account_file> #認証情報の保存先を指定(空白でOK)
Edit advanced config?
y) Yes
n) No (default)
y/n> n #詳細設定を編集するかどうか選択
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine

y) Yes (default)
n) No
y/n> n #アクセストークンの取得方法を選択(yを選択するとターミナル上で認証を行います)
Option config_verification_code.
Verification code
Go to this URL, authenticate then paste the code here.
https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=<client_id>&redirect_uri=urn:ietf:wg:oauth:2.0:oob&response_type=code&scope=https://www.googleapis.com/auth/drive
Enter a string value. Press Enter for the default ("").
config_verification_code> #上記のURLをブラウザで開いて表示されたバリデーションコードを入力
2021/12/04 14:59:52 NOTICE: Make sure your Redirect URL is set to "urn:ietf:wg:oauth:2.0:oob" in your custom config.
Configure this as a Shared Drive (Team Drive)?

y) Yes
n) No (default)
y/n> y #共有ドライブを設定するかどうか選択
--------------------
[googledrive]
type = drive
client_id = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
client_secret = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
scope = drive
root_folder_id = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
token = {} #トークン情報
team_drive =
--------------------
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y #設定を保存
Current remotes:

Name                 Type
====                 ====
googledrive          drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q #設定を終了
```

`rclone config`で対話形式の初期設定し、GoogleDriveとローカルの連携を行ないます。
保存した設定ファイルの格納場所は指定しない場合デフォルトの場所に保存されますが、場所を確認する場合は`rclone config file`を実行してください。

```powershell
rclone config file
Configuration file is stored at:
C:\Users\yuta_\scoop\apps\rclone\current\rclone.conf

# 接続先一覧の表示
rclone listremotes
googledrive:
```
初期設定が完了しましたら、ファイルをアップロードしてみます。

## ファイルアップロード

```powershell
ls .\sample.txt

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          2021/12/04    15:54             20 sample.txt

cat .\sample.txt
file Upload Success!
rclone copy .\sample.txt googledrive:
```

![](/images/rclone-upload-gdrive/image4.png)
![](/images/rclone-upload-gdrive/image5.png)
ローカルからアップロードしたファイルは、GoogleDriveに格納されました。
ファイルの中身もちゃんと含まれています。

# Rcloneでできること
Rcloneはファイルのアップロード/ダウンロード以外にもいくつか機能を持っています。
ファイル転送時に暗号化も可能ですので、セキュアな状態でクラウドストレージにファイルを転送できます。
暗号化してファイル転送をしたい場合は、`rclone config`でストレージ先を選択したときにGoogleDriveを選択しましたが、ここで`crypt`を選択し暗号化設定を行なっていきます。
## 暗号化設定

```powershell
rclone config
Current remotes:

Name                 Type
====                 ====
googledrive          drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> n #新規にリモートを追加
name> encrypt-drive #任意のリモート名を設定
Option Storage.
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value.
 1 / 1Fichier
   \ "fichier"
 2 / Alias for an existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Providers including AWS, Alibaba, Ceph, Digital Ocean, Dreamhost, IBM COS, Minio, SeaweedFS, and Tencent COS
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Better checksums for other remotes
   \ "hasher"
 7 / Box
   \ "box"
 8 / Cache a remote
   \ "cache"
 9 / Citrix Sharefile
   \ "sharefile"
10 / Compress a remote
   \ "compress"
11 / Dropbox
   \ "dropbox"
12 / Encrypt/Decrypt a remote
   \ "crypt"
13 / Enterprise File Fabric
   \ "filefabric"
14 / FTP Connection
   \ "ftp"
15 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
16 / Google Drive
   \ "drive"
17 / Google Photos
   \ "google photos"
18 / Hadoop distributed file system
   \ "hdfs"
19 / Hubic
   \ "hubic"
20 / In memory object storage system.
   \ "memory"
21 / Jottacloud
   \ "jottacloud"
22 / Koofr
   \ "koofr"
23 / Local Disk
   \ "local"
24 / Mail.ru Cloud
   \ "mailru"
25 / Mega
   \ "mega"
26 / Microsoft Azure Blob Storage
   \ "azureblob"
27 / Microsoft OneDrive
   \ "onedrive"
28 / OpenDrive
   \ "opendrive"
29 / OpenStack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
30 / Pcloud
   \ "pcloud"
31 / Put.io
   \ "putio"
32 / QingCloud Object Storage
   \ "qingstor"
33 / SSH/SFTP Connection
   \ "sftp"
34 / Sia Decentralized Cloud
   \ "sia"
35 / Sugarsync
   \ "sugarsync"
36 / Tardigrade Decentralized Cloud Storage
   \ "tardigrade"
37 / Transparently chunk/split large files
   \ "chunker"
38 / Union merges the contents of several upstream fs
   \ "union"
39 / Uptobox
   \ "uptobox"
40 / Webdav
   \ "webdav"
41 / Yandex Disk
   \ "yandex"
42 / Zoho
   \ "zoho"
43 / http Connection
   \ "http"
44 / premiumize.me
   \ "premiumizeme"
45 / seafile
   \ "seafile"
Storage> crypt #ここで暗号化設定のcryptを選択します
Option remote.
Remote to encrypt/decrypt.
Normally should contain a ':' and a path, e.g. "myremote:path/to/dir",
"myremote:bucket" or maybe "myremote:" (not recommended).
Enter a string value. Press Enter for the default ("").
remote> googledrive:encrypt/ #暗号化先を決定、既存ドライブを選択する場合前述で決めたリモート名を書くことで暗号化が可能
Option filename_encryption.
How to encrypt the filenames.
Enter a string value. Press Enter for the default ("standard").
Choose a number from below, or type in your own value.
   / Encrypt the filenames.
 1 | See the docs for the details.
   \ "standard"
 2 / Very simple filename obfuscation.
   \ "obfuscate"
   / Don't encrypt the file names.
 3 | Adds a ".bin" extension only.
   \ "off"
filename_encryption> standard #ファイル名の暗号化方法
Option directory_name_encryption.
Option to either encrypt directory names or leave them intact.
NB If filename_encryption is "off" then this option will do nothing.
Enter a boolean value (true or false). Press Enter for the default ("true").
Choose a number from below, or type in your own value.
 1 / Encrypt directory names.
   \ "true"
 2 / Don't encrypt directory names, leave them intact.
   \ "false" 
directory_name_encryption> true #ディレクトリ名の暗号化可否
Option password.
Password or pass phrase for encryption.
Choose an alternative below.
y) Yes type in my own password
g) Generate random password
y/g> g #gを選択することでランダムなパスワードを生成
Password strength in bits.
64 is just about memorable
128 is secure
1024 is the maximum
Bits> 128 
Your password is: XXXXXXXXXXXXXXXXXXXXXX
Use this password? Please note that an obscured version of this
password (and not the password itself) will be stored under your
configuration file, so keep this generated password in a safe place.
y) Yes (default)
n) No
y/n> y
Option password2.
Password or pass phrase for salt.
Optional but recommended.
Should be different to the previous password.
Choose an alternative below. Press Enter for the default (n).
y) Yes type in my own password
g) Generate random password
n) No leave this optional password blank (default)
y/g/n> g #ソルトを自動生成
Password strength in bits.
64 is just about memorable
128 is secure
1024 is the maximum
Bits> 128
Your password is: XXXXXXXXXXXXXXXXXXXXXX
Use this password? Please note that an obscured version of this
password (and not the password itself) will be stored under your
configuration file, so keep this generated password in a safe place.
y) Yes (default)
n) No
y/n> y
Edit advanced config?
y) Yes
n) No (default)
y/n> n
--------------------
[encrypt-drive]
type = crypt
remote = googledrive:encrypt/
password = *** ENCRYPTED ***
password2 = *** ENCRYPTED ***
--------------------
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y
Current remotes:

Name                 Type
====                 ====
encrypt-drive        crypt
googledrive          drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q

```

ファイルを転送して暗号化されるか確かめてみます。

```powershellt
rclone copy .\sample.txt encrypt-drive:crypted/
```

![](/images/rclone-upload-gdrive/image6.png)
![](/images/rclone-upload-gdrive/image7.png)
ファイル名が暗号化され、中身も見れません。
ちなみにRcloneでファイルを確認するとき、`crypt`で設定したプレフィックス`encrypt-drive`で`ls`コマンドをたたくとファイル情報が見れます。

```powershell
rclone listremotes

encrypt-drive: #ストレージ設定がGoogleDrive
googledrive: #ストレージ設定がcrypt

rclone ls googledrive:encrypt
       68 2qhniiup8u2s3ag18o9h1bmevo/kb6l5jrsndq1qduer20588545c

rclone ls encrypt-drive:
       20 crypted/sample.txt
```

## GUIダッシュボード
これはプレビュー的な機能みたいですが、RcloneにはGUIダッシュボードがあるみたいです。
https://rclone.org/gui/

```powershell
rclone rcd --rc-web-gui #GUIダッシュボードを起動
2021/12/04 18:59:25 ERROR : Error reading tag file at C:\Users\yuta_\AppData\Local\rclone\webgui\tag
2021/12/04 18:59:25 NOTICE: A new release for gui (v2.0.5) is present at https://github.com/rclone/rclone-webui-react/releases/download/v2.0.5/currentbuild.zip
2021/12/04 18:59:25 NOTICE: Downloading webgui binary. Please wait. [Size: 4763452, Path :  C:\Users\yuta_\AppData\Local\rclone\webgui\v2.0.5.zip]
2021/12/04 18:59:25 NOTICE: Unzipping webgui binary
2021/12/04 18:59:26 NOTICE: Serving Web GUI
2021/12/04 18:59:26 NOTICE: Serving remote control on http://localhost:5572/
```
![](/images/rclone-upload-gdrive/image8.png)
左のサイドバーを参照すると新規作成や既存のリモート先クラウドストレージの中身を確認できます。
![](/images/rclone-upload-gdrive/image9.png)

GUIダッシュボードはこちらのGitHubで開発されているみたいですので、興味がある人はOSSコントリビューションしてみてもいいかもしれません。
https://github.com/rclone/rclone-webui-react

# 所感
Rcloneを使って共有ドライブへファイルアップロードする方法について紹介しました。
個人的にはエンジニアっぽくPythonでAPIを操作して、ファイルアップロードしたかったのですがスキル不足もあり結局はOSSに逃げてしまった感じがします。
しばらく会社の運用はRcloneに任せますが、何かしらプログラミングを使って改善していけるようにしていきたいです。。。
# 参考文献
https://laboratory.kazuuu.net/upload-files-to-google-drive-with-python/
https://github.com/rclone/rclone
https://www.nextdoorwith.info/wp/se/how-to-control-google-drive-from-linux-rclone/
https://zenn.dev/milly/books/rclone-crypt-gdrive
