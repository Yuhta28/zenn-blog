---
title: "WindowsでPodmanを触ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["podman","Windows","WSL","podmancompose"]
published: false
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
以前書いた記事と重複しますが、あらためてPodmanについて説明いたします。Podmanとはdockerコマンドと互換性のあるコンテナエンジンです。デーモンレスで動かすことができるため、リソース消費を抑えられるというメリットがあります。
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
podman run --rm quay.io/podman/hello #quay.ioはRedHat社が管理しているコンテナレジストリ
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


# 参考文献
https://crunchtools.com/should-i-use-docker-compose-or-podman-compose-with-podman/
https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/index
https://rheb.hatenablog.com/entry/2020/07/16/podman_buidah_for_docker_users