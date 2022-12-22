---
title: "WindowsでPodmanを触ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["podman","Windows","WSL","podmancompose"]
published: true
---

# 概要
以前、Podmanに関する記事を書いたことがあります。
https://zenn.dev/yuta28/articles/23d303bb097f69095073

その後、Podmanのバージョンも上がり今年の2月にバージョン4系がリリースされました。
https://github.com/containers/podman/releases/tag/v4.0.0
4系の新機能としてWindowsホストでPodmanが利用できるようになったとアナウンスがありました。
> The podman machine init command now supports a new VM type, wsl, available only on Windows; this uses WSL as a backend for podman machine, instead of creating a separate VM and managing it via QEMU

WSLでLinuxディストリビューションを起動しなくてもWindowsでもPodmanが使えるとのことですので、今回はWindowsでPodmanを動かしてみて挙動を確かめてみました。

# Podmanについておさらい
https://podman.io/
以前書いた記事と重複しますが、改めてPodmanについて説明いたします。Podmanとはdockerコマンドと互換性のあるコンテナエンジンです。デーモンレスで動かすことができるため、リソース消費を抑えられるというメリットがあります。
Red Hat Enterprise Linuxではdockerに替わりPodmanを使用することが推奨されています。

# インストール方法
RHEL8、もしくはCentOS8以降のOSでしたらPodmanやBuildahを含んだ`container-tools`モジュールをインストールできます。
```bash
yum(dnf) module install -y container-tools
```

Windowsでしたら[こちらのREADME](https://github.com/containers/podman/blob/main/docs/tutorials/podman-for-windows.md)に紹介されているとおり、wingetでも入手できますし私が普段使っている[Scoop](https://scoop.sh/)でもインストールできます。

```powershell
scoop install podman
scoop list podman
Installed apps matching 'podman':

Name   Version Source Updated             Info
----   ------- ------ -------             ----
podman 4.1.0   main   2022-05-14 02:09:32
```

# 起動方法
先ほどのREADMEの説明によりますとWindowsでのPodmanはWSLのディストリビューションの1つとして仮想化されたPodman Machineを起動して管理されていると記載されています。
> This guest is referred to as a Podman machine and is managed with the podman machine command. On Windows, each Podman machine is backed by a virtualized Windows System for Linux (WSLv2) distribution.

WindowsにPodmanをインストールしましたら、WindowsホストにPodman Machineを作成いたします。

```powershell: Podman Machine作成
PS C:\Users\User> podman machine init
Extracting compressed file
Importing operating system into WSL (this may take 5+ minutes on a new WSL install)...
Installing packages (this will take a while)...
Complete!
Configuring system...
Generating public/private ed25519 key pair.
Your identification has been saved in podman-machine-default
Your public key has been saved in podman-machine-default.pub
The key fingerprint is:
SHA256:RGTGg2Q/LX7ijN+mzu8+BzcS3cEWP6Hir6pYllJtceA root@WINPC
Machine init complete
To start your machine run:

        podman machine start
```

最後に表示されたコマンドに従って、Podman Machineを起動させます。

```powershell: Podman Machine起動
PS C:\Users\User> podman machine start

Starting machine "podman-machine-default"

This machine is currently configured in rootless mode. If your containers
require root permissions (e.g. ports < 1024), or if you run into compatibility
issues with non-podman clients, you can switch using the following command:

        podman machine set --rootful

API forwarding listening on: npipe:////./pipe/docker_engine

Docker API clients default to this address. You do not need to set DOCKER_HOST.
Machine "podman-machine-default" started successfully

wsl -l -v
  NAME                      STATE           VERSION
* Ubuntu                    Running         2
  podman-machine-default    Running         2      # Podman Machineが起動されている
  docker-desktop-data       Running         2
  docker-desktop            Running         2
```

# コンテナ起動
Podman Machineを起動しましたのでPodmanのコンテナを起動してみます。

```powershell:Hello-Podman!
podman run --rm quay.io/podman/hello #quay.ioはRed Hat社が管理しているコンテナレジストリ
Trying to pull quay.io/podman/hello:latest...
Getting image source signatures
Copying blob sha256:f11656b0bc64345fd33f98397564064ad321c6aed1cf356003f7a4c0558a83b2
Copying blob sha256:f11656b0bc64345fd33f98397564064ad321c6aed1cf356003f7a4c0558a83b2
Copying config sha256:1b33cca41a84e34a955aae5c1d5691ef220403213dc66d272b6e659c9ebd8b8a
Writing manifest to image destination
Storing signatures
!... Hello Podman World ...!

         .--"--.
       / -     - \
      / (O)   (O) \
   ~~~| -=(,Y,)=- |
    .---. /`  \   |~~
 ~/  o  o \~~~~.----. ~~
  | =(X)= |~  / (O (O) \
   ~~~~~~~  ~| =(Y_)=-  |
  ~~~~    ~~~|   U      |~~

Project:   https://github.com/containers/podman
Website:   https://podman.io
Documents: https://docs.podman.io
Twitter:   @Podman_io
```

## ポートフォワーディング

```powershell: ポートフォワーディング確認
 podman run --rm -d -p 8080:80 --name httpd docker.io/library/httpd #Apacheコンテナを起動
 Trying to pull docker.io/library/httpd:latest...
Getting image source signatures
Copying blob sha256:dc25474c7f97bb2bfe901e77ab63a2e725a82c11cd0e36eb4e8c27c8a980f658
Copying blob sha256:dc25474c7f97bb2bfe901e77ab63a2e725a82c11cd0e36eb4e8c27c8a980f658
Copying blob sha256:214ca5fb90323fe769c63a12af092f2572bf1c6b300263e09883909fc865d260
Copying blob sha256:7cf31a2eeec6ac0953b123fc95d3f54bd0b08038aa69791279d186f736e699da
Copying blob sha256:bf666e57b9f28b78cde9f890d0af95d0a75c0ed4d95c5a3fb186da00ada008eb
Copying blob sha256:214ca5fb90323fe769c63a12af092f2572bf1c6b300263e09883909fc865d260
Copying blob sha256:bf666e57b9f28b78cde9f890d0af95d0a75c0ed4d95c5a3fb186da00ada008eb
Copying blob sha256:c15a4e94ae6b799e14422b659d15e9dc4d84de6fad9257e4d871a47141163672
Copying blob sha256:7cf31a2eeec6ac0953b123fc95d3f54bd0b08038aa69791279d186f736e699da
Copying blob sha256:c15a4e94ae6b799e14422b659d15e9dc4d84de6fad9257e4d871a47141163672
Copying config sha256:c58ef9bfbb5789a9882cee610ba778b1368d21b513d6caf32e3075542e13fe81
Writing manifest to image destination
Storing signatures
7cb081a256f59919b1e63bb6abde4b81a9ca3986e77b50705720d20c3d48bc62

curl -i http://localhost:8080/ #Apacheが起動している
HTTP/1.1 200 OK
Date: Sat, 21 May 2022 05:22:56 GMT
Server: Apache/2.4.53 (Unix)
Last-Modified: Mon, 11 Jun 2007 18:53:14 GMT
ETag: "2d-432a5e4a73a80"
Accept-Ranges: bytes
Content-Length: 45
Content-Type: text/html

<html><body><h1>It works!</h1></body></html>

podman container ls #コンテナ一覧を表示
CONTAINER ID  IMAGE                           COMMAND           CREATED         STATUS             PORTS                 NAMES
7cb081a256f5  docker.io/library/httpd:latest  httpd-foreground  21 minutes ago  Up 21 minutes ago  0.0.0.0:8080->80/tcp  httpd
```

ポートフォワーディングしてバックグラウンドでApacheのコンテナが起動されていることが確認できます。
Dockerでコンテナ起動していないのでDockerデスクトップや`docker container ls`にはApacheのコンテナは表示されていません。

```powershell
docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

![](/images/podman-for-windows/image1.png)

## ホスト-コンテナ間のボリュームマウント
Podmanのバージョン4.1以上の機能になりますが、WindowsホストのパスをLinuxコンテナへボリュームマウントできるようになりました。

```powershell: Cドライブにあるフォルダをコンテナへマウント
podman run --rm -v C:\Users\yuta_\ghq\github.com\Yuhta28\dev-to-blog\blog-posts:/blog-posts ubi8-micro ls -l blog-posts
total 0
drwxrwxrwx 1 nobody nobody 512 Apr 18 11:25 be-attention-to-sshkey
drwxrwxrwx 1 nobody nobody 512 May  4 06:40 cw-oss-cloudwatch
drwxrwxrwx 1 nobody nobody 512 May  7 06:05 dev-to-github
drwxrwxrwx 1 nobody nobody 512 Apr 18 03:43 diffrence-ec2-risp-and-rds-ri
drwxrwxrwx 1 nobody nobody 512 Apr 18 04:02 gha-with-oidc-terraform
drwxrwxrwx 1 nobody nobody 512 Apr 18 07:20 keep-in-mind-coding-existing-infra
drwxrwxrwx 1 nobody nobody 512 Apr 20 14:25 repography-make-readme-rich
```
## Podman環境へのログイン
Podman MachineはWSLディストリビューションの1つなのでSSHやWindows TerminalでPodman Machineへ直接ログインできます。

```powershell: Podman Machineへのログイン
podman machine ssh                                                                           pwsh   100  17:06:23 
Connecting to vm podman-machine-default. To close connection, use `~.` or `exit`
Warning: Permanently added '[localhost]:64958' (ECDSA) to the list of known hosts.
[user@Yuta-PC ~]$
[user@Yuta-PC ~]$ cat /etc/os-release
NAME="Fedora Linux"
VERSION="35 (Container Image)"
ID=fedora
VERSION_ID=35
VERSION_CODENAME=""
PLATFORM_ID="platform:f35"
PRETTY_NAME="Fedora Linux 35 (Container Image)"
ANSI_COLOR="0;38;2;60;110;180"
LOGO=fedora-logo-icon
CPE_NAME="cpe:/o:fedoraproject:fedora:35"
HOME_URL="https://fedoraproject.org/"
DOCUMENTATION_URL="https://docs.fedoraproject.org/en-US/fedora/f35/system-administrators-guide/"
SUPPORT_URL="https://ask.fedoraproject.org/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_BUGZILLA_PRODUCT="Fedora"
REDHAT_BUGZILLA_PRODUCT_VERSION=35
REDHAT_SUPPORT_PRODUCT="Fedora"
REDHAT_SUPPORT_PRODUCT_VERSION=35
PRIVACY_POLICY_URL="https://fedoraproject.org/wiki/Legal:PrivacyPolicy"
VARIANT="Container Image"
VARIANT_ID=container
```

![](/images/podman-for-windows/image2.png)

OS情報を確認するとFedora LinuxなのでFedoraのアップデート情報がありましたらこれでアップデートできます。

その他は基本的にDockerと同じように操作できそうです。

# Podman-compose
Podmanにもpodman-composeというdocker-composeみたいなものがあります。
https://github.com/containers/podman-compose

ルートレスでデーモン起動不要というPodmanの特徴を持ち、[Compose Spec](https://compose-spec.io/)を満たした仕様となっています。pipパッケージで配布されていますので、OS問わずインストールできます。

```powershell
PS> pip install podman-compose
PS> pip list
Package            Version
------------------ ---------
certifi            2021.10.8
charset-normalizer 2.0.12
idna               3.3
pip                22.1.1
podman-compose     1.0.3
python-dotenv      0.20.0
PyYAML             6.0
requests           2.27.1
urllib3            1.26.9
```

:::message
Fedora系OSならdnfパッケージでインストールできます。
`sudo dnf install podman-compose`
そのほかにもcurlで直接インストールする方法も紹介されています。
https://github.com/containers/podman-compose#installation
:::

コマンド体系などはdocker-composeと遜色はないと思います。

```powershell: podman-composeのコマンド集
PS> podman-compose --help
usage: podman-compose [-h] [-v] [-f file] [-p PROJECT_NAME] [--podman-path PODMAN_PATH] [--podman-args args] [--podman-pull-args args] [--podman-push-args args]
                      [--podman-build-args args] [--podman-inspect-args args] [--podman-run-args args] [--podman-start-args args] [--podman-stop-args args]
                      [--podman-rm-args args] [--podman-volume-args args] [--no-ansi] [--no-cleanup] [--dry-run]
                      {help,version,pull,push,build,up,down,ps,run,exec,start,stop,restart,logs} ...

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show version
  -f file, --file file  Specify an alternate compose file (default: docker-compose.yml)
  -p PROJECT_NAME, --project-name PROJECT_NAME
                        Specify an alternate project name (default: directory name)
  --podman-path PODMAN_PATH
                        Specify an alternate path to podman (default: use location in $PATH variable)
  --podman-args args    custom global arguments to be passed to `podman`
  --podman-pull-args args
                        custom arguments to be passed to `podman pull`
  --podman-push-args args
                        custom arguments to be passed to `podman push`
  --podman-build-args args
                        custom arguments to be passed to `podman build`
  --podman-inspect-args args
                        custom arguments to be passed to `podman inspect`
  --podman-run-args args
                        custom arguments to be passed to `podman run`
  --podman-start-args args
                        custom arguments to be passed to `podman start`
  --podman-stop-args args
                        custom arguments to be passed to `podman stop`
  --podman-rm-args args
                        custom arguments to be passed to `podman rm`
  --podman-volume-args args
                        custom arguments to be passed to `podman volume`
  --no-ansi             Do not print ANSI control characters
  --no-cleanup          Do not stop and remove existing pod & containers
  --dry-run             No action; perform a simulation of commands

command:
  {help,version,pull,push,build,up,down,ps,run,exec,start,stop,restart,logs}
    help                show help
    version             show version
    pull                pull stack images
    push                push stack images
    build               build stack images
    up                  Create and start the entire stack or some of its services
    down                tear down entire stack
    ps                  show status of containers
    run                 create a container similar to a service to run a one-off command
    exec                execute a command in a running container
    start               start specific services
    stop                stop specific services
    restart             restart specific services
    logs                show logs from services
```

# チュートリアル
READMEに記載されているとおり、`examples`ディレクトリにサンプルソースコードがありますのでいくつか試してみます。

## hello-appコンテナ起動
GCRのhello-appコンテナを起動してみます。

https://github.com/containers/podman-compose/blob/devel/examples/hello-app/docker-compose.yaml

```powershell:Podmanコンテナ起動
PS> podman-compose up
['podman', '--version', '']
using podman version: 4.1.0
** excluding:  set()
['podman', 'network', 'exists', 'hello-app_default']
['podman', 'network', 'create', '--label', 'io.podman.compose.project=hello-app', '--label', 'com.docker.compose.project=hello-app', 'hello-app_default']
['podman', 'network', 'exists', 'hello-app_default']
podman create --name=hello-app_web_1 --label io.podman.compose.config-hash=123 --label io.podman.compose.project=hello-app --label io.podman.compose.version=0.0.1 --label com.docker.compose.project=hello-app --label com.docker.compose.project.working_dir=C:\Users\yuta_\ghq\github.com\containers\podman-compose\examples\hello-app --label com.docker.compose.project.config_files=docker-compose.yaml --label com.docker.compose.container-number=1 --label com.docker.compose.service=web --net hello-app_default --network-alias web -p 8080:8080 gcr.io/google-samples/hello-app:1.0
Trying to pull gcr.io/google-samples/hello-app:1.0...
Getting image source signatures
Copying blob sha256:da75187f77d85161dc2631b60db75b1b99522cfde60b3deacd26f3208b341778
Copying blob sha256:da75187f77d85161dc2631b60db75b1b99522cfde60b3deacd26f3208b341778
Copying blob sha256:59bf1c3509f33515622619af21ed55bbe26d24913cedbca106468a5fb37a50c3
Copying blob sha256:59bf1c3509f33515622619af21ed55bbe26d24913cedbca106468a5fb37a50c3
Copying config sha256:f4d42e081a02b89d8bab78682f5879e6b593efe76690872c71a7a04910ee474a
Writing manifest to image destination
Storing signatures
63de71063a0ca424f90905932fde72257e668e528f2b63fa5b02702b18bd2a5b
exit code: 0
podman start -a hello-app_web_1
2022/05/21 15:17:30 Server listening on port 8080
2022/05/21 15:18:09 Serving request: /
2022/05/21 15:18:09 Serving request: /favicon.ico
```

![](/images/podman-for-windows/image3.png)
*ブラウザ表示確認*

## Hello-Pythonコンテナ起動
Pythonで作られたシンプルなアプリコンテナを起動します。

https://github.com/containers/podman-compose/blob/devel/examples/hello-python/App/web.py
https://github.com/containers/podman-compose/blob/devel/examples/hello-python/docker-compose.yaml

:::details (長いのでたたんでいます)
```powershell:Podmanコンテナ起動
PS> podman-compose up -d
['podman', '--version', '']
using podman version: 4.1.0
** excluding:  set()
['podman', 'inspect', '-t', 'image', '-f', '{{.Id}}', 'hello-py-aioweb']
Error: inspecting object: unable to inspect "hello-py-aioweb": failed to find image hello-py-aioweb: hello-py-aioweb: image not known
podman build -t hello-py-aioweb -f .\Dockerfile .
STEP 1/7: FROM python:3.9-alpine
Resolved "python" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/python:3.9-alpine...
Getting image source signatures
Copying blob sha256:c1cbc3d14cc64f2b6731207dd2a11295d96cc5ed53bed2546e89e223145ca001
Copying blob sha256:c1cbc3d14cc64f2b6731207dd2a11295d96cc5ed53bed2546e89e223145ca001
Copying blob sha256:a1ef3e6b7a02c0ce1d7d3ee96f273c2c5074b3f127082b4cd3f629f82097e1a8
Copying blob sha256:a1ef3e6b7a02c0ce1d7d3ee96f273c2c5074b3f127082b4cd3f629f82097e1a8
Copying blob sha256:79b7769af0709a58ec8e18c527dbf5a740a9392da6f195569a3ae94f65263f3b
Copying blob sha256:df9b9388f04ad6279a7410b85cedfdcb2208c0a003da7ab5613af71079148139
Copying blob sha256:79b7769af0709a58ec8e18c527dbf5a740a9392da6f195569a3ae94f65263f3b
Copying blob sha256:df9b9388f04ad6279a7410b85cedfdcb2208c0a003da7ab5613af71079148139
Copying blob sha256:cdeefedcfa76158549020ac809422d85bbf044313d198b47d2f938afbc7615ea
Copying blob sha256:cdeefedcfa76158549020ac809422d85bbf044313d198b47d2f938afbc7615ea
Copying config sha256:f51e54f4d536b75b3b74896692c706083ce96d42e84e2bbacb3604b80ec7bf1c
Writing manifest to image destination
Storing signatures
STEP 2/7: WORKDIR /usr/src/app
--> 7d5c1875033
STEP 3/7: COPY requirements.txt ./
--> db5e3e0103f
STEP 4/7: RUN pip install --no-cache-dir -r requirements.txt
Collecting aiohttp
  Downloading aiohttp-3.8.1-cp39-cp39-musllinux_1_1_x86_64.whl (1.2 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.2/1.2 MB 15.8 MB/s eta 0:00:00
Collecting aioredis
  Downloading aioredis-2.0.1-py3-none-any.whl (71 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 71.2/71.2 KB 84.1 MB/s eta 0:00:00
Collecting charset-normalizer<3.0,>=2.0
  Downloading charset_normalizer-2.0.12-py3-none-any.whl (39 kB)
Collecting attrs>=17.3.0
  Downloading attrs-21.4.0-py2.py3-none-any.whl (60 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 60.6/60.6 KB 45.1 MB/s eta 0:00:00
Collecting aiosignal>=1.1.2
  Downloading aiosignal-1.2.0-py3-none-any.whl (8.2 kB)
Collecting async-timeout<5.0,>=4.0.0a3
  Downloading async_timeout-4.0.2-py3-none-any.whl (5.8 kB)
Collecting frozenlist>=1.1.1
  Downloading frozenlist-1.3.0-cp39-cp39-musllinux_1_1_x86_64.whl (158 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 158.2/158.2 KB 56.0 MB/s eta 0:00:00
Collecting yarl<2.0,>=1.0
  Downloading yarl-1.7.2-cp39-cp39-musllinux_1_1_x86_64.whl (312 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 312.5/312.5 KB 34.7 MB/s eta 0:00:00
Collecting multidict<7.0,>=4.5
  Downloading multidict-6.0.2-cp39-cp39-musllinux_1_1_x86_64.whl (134 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 134.5/134.5 KB 40.2 MB/s eta 0:00:00
Collecting typing-extensions
  Downloading typing_extensions-4.2.0-py3-none-any.whl (24 kB)
Collecting idna>=2.0
  Downloading idna-3.3-py3-none-any.whl (61 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 61.2/61.2 KB 57.6 MB/s eta 0:00:00
Installing collected packages: typing-extensions, multidict, idna, frozenlist, charset-normalizer, attrs, async-timeout, yarl, aiosignal, aioredis, aiohttp
Successfully installed aiohttp-3.8.1 aioredis-2.0.1 aiosignal-1.2.0 async-timeout-4.0.2 attrs-21.4.0 charset-normalizer-2.0.12 frozenlist-1.3.0 idna-3.3 multidict-6.0.2 typing-extensions-4.2.0 yarl-1.7.2
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 22.0.4; however, version 22.1.1 is available.
You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.
--> a528d725a09
STEP 5/7: COPY . .
--> b74a8092180
STEP 6/7: CMD [ "python", "-m", "App.web" ]
--> 2b576773b77
STEP 7/7: EXPOSE 8080
COMMIT hello-py-aioweb
--> 87f7140d82e
Successfully tagged localhost/hello-py-aioweb:latest
87f7140d82e0f46f8a5dcd9cc31c44ccf96e9cfcba1159bffd5629a6cefa4c41
exit code: 0
podman volume inspect hello-python_redis || podman volume create hello-python_redis
['podman', 'volume', 'inspect', 'hello-python_redis']
Error: inspecting object: no such volume "hello-python_redis"
['podman', 'volume', 'create', '--label', 'io.podman.compose.project=hello-python', '--label', 'com.docker.compose.project=hello-python', 'hello-python_redis']
['podman', 'volume', 'inspect', 'hello-python_redis']
['podman', 'network', 'exists', 'hello-python_default']
['podman', 'network', 'create', '--label', 'io.podman.compose.project=hello-python', '--label', 'com.docker.compose.project=hello-python', 'hello-python_default']
['podman', 'network', 'exists', 'hello-python_default']
podman run --name=hello-python_redis_1 -d --read-only --label io.podman.compose.config-hash=123 --label io.podman.compose.project=hello-python --label io.podman.compose.version=0.0.1 --label com.docker.compose.project=hello-python --label com.docker.compose.project.working_dir=C:\Users\yuta_\ghq\github.com\containers\podman-compose\examples\hello-python --label com.docker.compose.project.config_files=docker-compose.yaml --label com.docker.compose.container-number=1 --label com.docker.compose.service=redis -v hello-python_redis:/data --net hello-python_default --network-alias redis docker.io/redis:alpine redis-server --appendonly yes --notify-keyspace-events Ex
Trying to pull docker.io/library/redis:alpine...
Getting image source signatures
Copying blob sha256:192e0352348275fe7c4099cf478959878b50bda7afe561ad71f1f239daa17d31
Copying blob sha256:683d62ead94f696f45b48f9c73021355a3301d47dfd281405295edf906b4bf77
Copying blob sha256:192e0352348275fe7c4099cf478959878b50bda7afe561ad71f1f239daa17d31
Copying blob sha256:b4ca937b9a43c2fcbc10314f983d1c591909de7f607b289402700cd28d8f8472
Copying blob sha256:683d62ead94f696f45b48f9c73021355a3301d47dfd281405295edf906b4bf77
Copying blob sha256:b4bb2d8d1296bf5f01ab9f9cf194b7ce56b992601e610fc007c00a64f220b00c
Copying blob sha256:b4ca937b9a43c2fcbc10314f983d1c591909de7f607b289402700cd28d8f8472
Copying blob sha256:7151bccd2756efc52d97e3ec15c8053c4313a5bee00319382ba922c7a1cdf2aa
Copying blob sha256:b4bb2d8d1296bf5f01ab9f9cf194b7ce56b992601e610fc007c00a64f220b00c
Copying blob sha256:7151bccd2756efc52d97e3ec15c8053c4313a5bee00319382ba922c7a1cdf2aa
Copying blob sha256:df9b9388f04ad6279a7410b85cedfdcb2208c0a003da7ab5613af71079148139
Copying config sha256:8a0942cae737a6aa63f41546bbe718ed759bcdf469785d1869496c9e92f48539
Writing manifest to image destination
Storing signatures
58a49577919a40fa32eafb3365f0e0b75a3b905b4d2d481aefa277b6aa3a5a5a
exit code: 0
['podman', 'network', 'exists', 'hello-python_default']
podman run --name=hello-python_web_1 -d --read-only --label io.podman.compose.config-hash=123 --label io.podman.compose.project=hello-python --label io.podman.compose.version=0.0.1 --label com.docker.compose.project=hello-python --label com.docker.compose.project.working_dir=C:\Users\yuta_\ghq\github.com\containers\podman-compose\examples\hello-python --label com.docker.compose.project.config_files=docker-compose.yaml --label com.docker.compose.container-number=1 --label com.docker.compose.service=web -e REDIS_HOST=redis --net hello-python_default --network-alias web -p 8080:8080 hello-py-aioweb
af792ffc2707dbb63fcffde8dae5a09040835b0c491c3532392f94d8fc8afed1
exit code: 0

PS> curl localhost:8080/
counter=1
PS> curl localhost:8080/hello.json
{"counter": 2}
```
:::

## WordPressコンテナ起動
WordPressコンテナも起動してみます。
https://github.com/containers/podman-compose/blob/devel/examples/wordpress/docker-compose.yaml

::: details (長いのでたたんでいます)
```powershell:Podmanコンテナ起動
PS> podman-compose up -d
['podman', '--version', '']
using podman version: 4.1.0
** excluding:  set()
['podman', 'network', 'exists', 'wordpress_default']
['podman', 'network', 'create', '--label', 'io.podman.compose.project=wordpress', '--label', 'com.docker.compose.project=wordpress', 'wordpress_default']
['podman', 'network', 'exists', 'wordpress_default']
podman run --name=wordpress_wordpress_1 -d --label io.podman.compose.config-hash=123 --label io.podman.compose.project=wordpress --label io.podman.compose.version=0.0.1 --label com.docker.compose.project=wordpress --label com.docker.compose.project.working_dir=C:\Users\yuta_\ghq\github.com\containers\podman-compose\examples\wordpress --label com.docker.compose.project.config_files=docker-compose.yaml --label com.docker.compose.container-number=1 --label com.docker.compose.service=wordpress -e WORDPRESS_DB_HOST=db -e WORDPRESS_DB_USER=wordpress -e WORDPRESS_DB_PASSWORD=password -e WORDPRESS_DB_NAME=wordpress --net wordpress_default --network-alias wordpress -p 8080:80 docker.io/library/wordpress:latest
Trying to pull docker.io/library/wordpress:latest...
Getting image source signatures
Copying blob sha256:5f145e355bc44a3e4977431603bb5670c5a18d105dd8fbfda36c9e1d8500cfe8
Copying blob sha256:cd813a1b2cb8a76f5b4f268f422edd8f2cdea4c0354dd4762ea7b1d1e1c766de
Copying blob sha256:5f145e355bc44a3e4977431603bb5670c5a18d105dd8fbfda36c9e1d8500cfe8
Copying blob sha256:cd813a1b2cb8a76f5b4f268f422edd8f2cdea4c0354dd4762ea7b1d1e1c766de
Copying blob sha256:54c27146d16e29ce547bb7328aca3930153a051520bbeb17e85b2548c164d71b
Copying blob sha256:63cf7574573d0f63818130302247fc08e99f40a8c2b9b7be197cef5fe5006264
Copying blob sha256:54c27146d16e29ce547bb7328aca3930153a051520bbeb17e85b2548c164d71b
Copying blob sha256:214ca5fb90323fe769c63a12af092f2572bf1c6b300263e09883909fc865d260
Copying blob sha256:63cf7574573d0f63818130302247fc08e99f40a8c2b9b7be197cef5fe5006264
Copying blob sha256:214ca5fb90323fe769c63a12af092f2572bf1c6b300263e09883909fc865d260
Copying blob sha256:078f4450f949cf8479a92a7318d0596aa9a42aaf4a0b6770a30dee9d557430c3
Copying blob sha256:078f4450f949cf8479a92a7318d0596aa9a42aaf4a0b6770a30dee9d557430c3
Copying blob sha256:fdc797cb9eeaeb3f771cd61108fec7f9846b4d2e12059cc4fb098774c4484a3c
Copying blob sha256:fdc797cb9eeaeb3f771cd61108fec7f9846b4d2e12059cc4fb098774c4484a3c
Copying blob sha256:0e1a880fdf1905b43e7bd1d2b7c67b1c60d6480662554b24f3080bfa3ce8a98c
Copying blob sha256:0e1a880fdf1905b43e7bd1d2b7c67b1c60d6480662554b24f3080bfa3ce8a98c
Copying blob sha256:b09a58fafe61c11f0804321a25198e0f1ddad5b90d81b5a38c760c543a89271e
Copying blob sha256:b09a58fafe61c11f0804321a25198e0f1ddad5b90d81b5a38c760c543a89271e
Copying blob sha256:99a91c7abd4b7305a2ed49c76b1adfe0459f96fe1b18e97a570ec95dcc2a1ad9
Copying blob sha256:99a91c7abd4b7305a2ed49c76b1adfe0459f96fe1b18e97a570ec95dcc2a1ad9
Copying blob sha256:a561dfd0a6e49e41bb1ca5a4dd48de8e3e44842625e9d6829f3889a6123fc6c6
Copying blob sha256:a561dfd0a6e49e41bb1ca5a4dd48de8e3e44842625e9d6829f3889a6123fc6c6
Copying blob sha256:9bbca95a37a878e208e1d4f1c3cabaf0bb2c1002ca31d151eaeef718e99b4689
Copying blob sha256:9bbca95a37a878e208e1d4f1c3cabaf0bb2c1002ca31d151eaeef718e99b4689
Copying blob sha256:3b08a964eacd3a799f7ffb0509d512c6b545f1e370d018a5b918ac13012bb1eb
Copying blob sha256:3b08a964eacd3a799f7ffb0509d512c6b545f1e370d018a5b918ac13012bb1eb
Copying blob sha256:d678d31a4e0dc33d6fbcace972e908daa4682de759eac2de674a09a6f8baa22d
Copying blob sha256:d678d31a4e0dc33d6fbcace972e908daa4682de759eac2de674a09a6f8baa22d
Copying blob sha256:a56558133e814848310a3cc14947c30d7cace0e38edb5dc001a4ddfe4cface7d
Copying blob sha256:a56558133e814848310a3cc14947c30d7cace0e38edb5dc001a4ddfe4cface7d
Copying blob sha256:407f56ae93918963b670ec7c254f12217b93f7e2ecc007309614681fb8ee0cb9
Copying blob sha256:407f56ae93918963b670ec7c254f12217b93f7e2ecc007309614681fb8ee0cb9
Copying blob sha256:5169a2dd75c2970b379776171c6b7b5ecf1e2df73fa9c30789f8ab0547688829
Copying blob sha256:5169a2dd75c2970b379776171c6b7b5ecf1e2df73fa9c30789f8ab0547688829
Copying blob sha256:5a4b786a36b3fa97e392dd900530e1fd8d264406f04e5e13d2ed47924fa298d2
Copying blob sha256:5a4b786a36b3fa97e392dd900530e1fd8d264406f04e5e13d2ed47924fa298d2
Copying blob sha256:7f028687a4a0c0edc576d7955b61fbe26721dcb25f155a762827df2b08a100e9
Copying blob sha256:7f028687a4a0c0edc576d7955b61fbe26721dcb25f155a762827df2b08a100e9
Copying blob sha256:5d5119ba463f3babaccca5165fda780967b50bc0598ab7597166a36349ee1bb1
Copying blob sha256:5d5119ba463f3babaccca5165fda780967b50bc0598ab7597166a36349ee1bb1
Copying blob sha256:55ceb3f8793b36d7d880daafb6ce705129ab8466a09c6766ba9f5a00a58a2118
Copying blob sha256:55ceb3f8793b36d7d880daafb6ce705129ab8466a09c6766ba9f5a00a58a2118
Copying config sha256:140ebc8d8136c8322afee8c75ed7cd3cd28c1ca519272cd5bdf773d3592ee062
Writing manifest to image destination
Storing signatures
4fbb290947820aa9aec6fdeef320eb3e6036311b3ff0f09df2e99d25d6a5845b
exit code: 0
podman volume inspect wordpress_db_data || podman volume create wordpress_db_data
['podman', 'volume', 'inspect', 'wordpress_db_data']
Error: inspecting object: no such volume "wordpress_db_data"
['podman', 'volume', 'create', '--label', 'io.podman.compose.project=wordpress', '--label', 'com.docker.compose.project=wordpress', 'wordpress_db_data']
['podman', 'volume', 'inspect', 'wordpress_db_data']
['podman', 'network', 'exists', 'wordpress_default']
podman run --name=wordpress_db_1 -d --label io.podman.compose.config-hash=123 --label io.podman.compose.project=wordpress --label io.podman.compose.version=0.0.1 --label com.docker.compose.project=wordpress --label com.docker.compose.project.working_dir=C:\Users\yuta_\ghq\github.com\containers\podman-compose\examples\wordpress --label com.docker.compose.project.config_files=docker-compose.yaml --label com.docker.compose.container-number=1 --label com.docker.compose.service=db -e MYSQL_ROOT_PASSWORD=somewordpress -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=password -v wordpress_db_data:/var/lib/mysql --net wordpress_default --network-alias db docker.io/library/mariadb:10.6.4-focal --default-authentication-plugin=mysql_native_password
Trying to pull docker.io/library/mariadb:10.6.4-focal...
Getting image source signatures
Copying blob sha256:dbcda06785eb5474250fd32695909199c0de3375a06cf66401d277da518f3da9
Copying blob sha256:dbcda06785eb5474250fd32695909199c0de3375a06cf66401d277da518f3da9
Copying blob sha256:034655750c88e076cbd516354371b6176d01179bf595d928444e663ba1fa6845
Copying blob sha256:7b1a6ab2e44dbac178598dabe7cff59bd67233dba0b27e4fbd1f9d4b3c877a54
Copying blob sha256:034655750c88e076cbd516354371b6176d01179bf595d928444e663ba1fa6845
Copying blob sha256:5c37daf8b6b55859dc1f92dbe017858ae03bd5bf503a3cdbf317b0b4387b1383
Copying blob sha256:7b1a6ab2e44dbac178598dabe7cff59bd67233dba0b27e4fbd1f9d4b3c877a54
Copying blob sha256:5c37daf8b6b55859dc1f92dbe017858ae03bd5bf503a3cdbf317b0b4387b1383
Copying blob sha256:f0b757a2a0f02848b1254cbd8b54e1d3cd4651228ede9a8c75e127f91cb415c4
Copying blob sha256:b4cd9409b0f69a0ebe270f43c3a217f05b4c326e5d7c9a4ffb0cb7852ecddc81
Copying blob sha256:f0b757a2a0f02848b1254cbd8b54e1d3cd4651228ede9a8c75e127f91cb415c4
Copying blob sha256:b4cd9409b0f69a0ebe270f43c3a217f05b4c326e5d7c9a4ffb0cb7852ecddc81
Copying blob sha256:a34cd90f184c404cd4ecd88ebe612e6f180f76836efd3c24a329a3b812c5c345
Copying blob sha256:a34cd90f184c404cd4ecd88ebe612e6f180f76836efd3c24a329a3b812c5c345
Copying blob sha256:fd6cef4ce4896ed34f82cdec0226170c9ec12e780095f39a268e4f4cd87be523
Copying blob sha256:fd6cef4ce4896ed34f82cdec0226170c9ec12e780095f39a268e4f4cd87be523
Copying blob sha256:3cb89a1550ea4efead895dec48626ae277912ca9eeffae1a3c1b8c3f0f9ed02d
Copying blob sha256:3cb89a1550ea4efead895dec48626ae277912ca9eeffae1a3c1b8c3f0f9ed02d
Copying blob sha256:df9f153bd93044efcf8d9cd1ea5266b32bf28df2ad72e5ce195428e8c81b917b
Copying blob sha256:df9f153bd93044efcf8d9cd1ea5266b32bf28df2ad72e5ce195428e8c81b917b
Copying config sha256:12e05d5da3c5223e9877e8eb90d68560ff66cedcb955131061d60d093a908f0c
Writing manifest to image destination
Storing signatures
4963164d86873fa3720fba90cf317b0557c650ed14ef702c05b881e6b35ce2fe
exit code: 0
```
:::

![](/images/podman-for-windows/image4.png)
*WPのインストール画面*

```powershell:コンテナの起動確認
PS> podman-compose ps
using podman version: 4.1.0
podman ps -a --filter label=io.podman.compose.project=wordpress
CONTAINER ID  IMAGE                                   COMMAND               CREATED         STATUS             PORTS                 NAMES
4fbb29094782  docker.io/library/wordpress:latest      apache2-foregroun...  31 seconds ago  Up 32 seconds ago  0.0.0.0:8080->80/tcp  wordpress_wordpress_1
4963164d8687  docker.io/library/mariadb:10.6.4-focal  --default-authent...  17 seconds ago  Up 18 seconds ago                        wordpress_db_1
exit code: 0
```

# docker-composeとpodman-composeどっちを使うべきか
実はPodmanでもdocker-composeは使えます。バージョン3系からdocker-composeが正式にサポートされることになりました。
https://www.redhat.com/sysadmin/podman-docker-compose

:::message alert
docker-composeバージョン1系のみ。docker-composeバージョン2系は未対応(2022/5現在)
:::
ただし、サービスの有効化が必要なためWSLディストリビューションでは利用できません。
```bash
$ systemctl status podman.socket
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```
Windowsホストでも環境変数でDockerホストをセットし、Podman Machineに渡すことでPodmanをDocker APIクライアントとして使えます。

https://github.com/containers/podman/blob/main/docs/tutorials/podman-for-windows.md#using-api-forwarding


一方、podman-composeはサードパーティー製のOSSです。どちらを使うほうがいいのでしょうか🤔
このような記事を見つけました。
https://crunchtools.com/should-i-use-docker-compose-or-podman-compose-with-podman/

Red Hat社の人が表題の質問への問いに回答を示しています。
>1. Red Hat generally recommends Kubernetes YAML instead of Compose, and we’re working towards a roadmap to deliver more and more Compose-like functionality with creating and consuming Kubernetes YAML with the podman-play/generate-kube functionality (image builds, application tear down, intiContainers, and expanded support for Kubernetes primitives in general)


::: message
##### 意訳
Red HatはComposeの代わりにKubernetes YAMLを推奨しています。PodmanはKubernetes YAMLを作成・利用することでComposeライクな機能を提供しています。
:::
Red Hatの見解としてCompose機能を使うのではなく、Kubernetesに代表されるPod機能を使うことを推奨しているようです。Compose機能を使う場合、Red Hatはdocker-composeにもpodman-composeにも肩入れはせず中立の立場をとると述べています。
> 2. If you still want to use Compose, Red Hat/RHEL is neutral between podman-compose and docker-compose

Podについては[以前試したこと](https://zenn.dev/yuta28/articles/23d303bb097f69095073#kube-pod-yaml-%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AE%E4%BD%9C%E6%88%90)がありますが、もう少し詳しく調べてみて記事にしたいと思います。

# 所感
WindowsにPodmanを入れてホスト側で動かせるようにしてみました。Windowsホストでもコンテナ技術関連でやれることが増えてきて、便利だなと感じました。昨今のDocker Desktop有料化の流れもあり、今後Podmanを使う機会が増えそうなのでより理解を深めてみようと思います。

# 参考文献
https://zenn.dev/dozo/articles/0ced3feae9ac63
https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/index
https://rheb.hatenablog.com/entry/2020/07/16/podman_buidah_for_docker_users
https://rheb.hatenablog.com/entry/podman3-rootless-docker-compose
https://rheb.hatenablog.com/entry/rhel84beta_podman3
https://rheb.hatenablog.com/entry/podman3_docker_compose