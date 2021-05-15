---
title: "Linuxのホスト名について調査してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux","hostname","RFC"]
published: false
---

# 概要
マシンを扱う上で大切なホスト名の命名規則について調べてみました。

# 背景
先日Ubuntuのホストネームを変更しようとしたら_(アンスコ)がホスト名につかない問題に遭遇しました。

```bash
$ cat /etc/os-release
NAME="Ubuntu"
VERSION="20.04.2 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.2 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
$ sudo hostnamectl set-hostname yuta_ubuntu
$ hostname
yutaubuntu
```

ご覧の通り、ホスト名にアンスコが入ると`hostname`で確認しても省略されてしまいます。

調べてみるとRFCではホスト名にアンスコを設定することは、命名規則に違反していることが分かりました。
:::message
RFCとはインターネット上にまつわるさまざまな技術的使用群やルールをまとめたドキュメント[^1]のことです。
[^1]: https://www.nic.ad.jp/ja/rfc-jp/WhatisRFC.html
:::

インターネットホストテーブルの仕様についてまとめたRFC952には以下の内容が書かれています。

> 1. A "name" (Net, Host, Gateway, or Domain name) is a text string up
   to 24 characters drawn from the alphabet (A-Z), digits (0-9), minus
   sign (-), and period (.).  Note that periods are only allowed when
   they serve to delimit components of "domain style names". (See
   RFC-921, "Domain Name System Implementation Schedule", for
   background).  No blank or space characters are permitted as part of a
   name. No distinction is made between upper and lower case.  The first
   character must be an alpha character.  The last character must not be
   a minus sign or period.  A host which serves as a GATEWAY should have
   "-GATEWAY" or "-GW" as part of its name.  Hosts which do not serve as
   Internet gateways should not use "-GATEWAY" and "-GW" as part of
   their names. A host which is a TAC should have "-TAC" as the last
   part of its host name, if it is a DoD host.  Single character names
   or nicknames are not allowed.

:::message
訳：ホスト名は24文字以下で英字、数字、ハイフン(-)、ピリオド(.)が使用できます。ピリオドは、ドメイン名の区切りに限ってのみ使用できます。
ブランクやスペース文字はホスト名に使うことはできません。英字の大文字小文字の区別はありません。
最初の文字は英字でなければなりません。最後の文字にハイフンやピリオドを使用することはできません。

ゲートウェイの機能を持つ機器には"-GATEWAY"や"-GW"の文字を入れるべきです。
逆にゲートウェイの機能を持たないのに、"-GATEWAY"や"-GW"の文字は入れるべきではありません。
1文字だけのホスト名は使用できません。
:::

UbuntuはこのRFCのルールを準拠しているため、ホスト名にアンスコを入れても無視されるというわけです。
ただこれらのルール全てを厳密に守っているわけではなく、例えばホスト名が数字のみでも問題ありませんでした。

```bash
$ sudo hostnamectl set-hostname 11111
$ hostname
11111
```
他にも24文字以上でもホスト名を設定することができました。

```bash
$ sudo hostnamectl set-hostname abcdefghijklmnopqrstuvwxyz
$ hostname
abcdefghijklmnopqrstuvwxyz
```

どこまでいけるのか試してみたところ64文字まで設定することが可能でした。
これはDNSのラベルの文字数制限[^2]と同じです。
[^2]:https://jprs.jp/glossary/index.php?ID=0149

そこでRFC952で定めている命名規則のルールでどれが厳格に守られていて、どれが許容されているのか色んなOSを使って検証してみることにしました。



```
$ cat /etc/os-release
NAME="Amazon Linux"
VERSION="2"
ID="amzn"
ID_LIKE="centos rhel fedora"
VERSION_ID="2"
PRETTY_NAME="Amazon Linux 2"
ANSI_COLOR="0;33"
CPE_NAME="cpe:2.3:o:amazon:amazon_linux:2"
HOME_URL="https://amazonlinux.com/"

$ sudo hostnamectl set-hostname yuta_ec2
$ hostname
yuta_ec2
```

```
$ cat /etc/os-release
NAME="AlmaLinux"
VERSION="8.3 (Purple Manul)"
ID="almalinux"
ID_LIKE="rhel centos fedora"
VERSION_ID="8.3"
PLATFORM_ID="platform:el8"
PRETTY_NAME="AlmaLinux 8.3 (Purple Manul)"
ANSI_COLOR="0;34"
CPE_NAME="cpe:/o:almalinux:almalinux:8.3:GA"
HOME_URL="https://almalinux.org/"
BUG_REPORT_URL="https://bugs.almalinux.org/"

ALMALINUX_MANTISBT_PROJECT="AlmaLinux-8"

$ sudo hostnamectl set-hostname yuta_alma-linux
$ hostname
yuta_alma-linux
```

```
$ cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

$ sudo hostnamectl set-hostname yuta_centos
$ hostname
yuta_centos
```


```
$ cat /etc/os-release
NAME="Red Hat Enterprise Linux"
VERSION="8.3 (Ootpa)"
ID="rhel"
ID_LIKE="fedora"
VERSION_ID="8.3"
PLATFORM_ID="platform:el8"
PRETTY_NAME="Red Hat Enterprise Linux 8.3 (Ootpa)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:redhat:enterprise_linux:8.3:GA"
HOME_URL="https://www.redhat.com/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"

REDHAT_BUGZILLA_PRODUCT="Red Hat Enterprise Linux 8"
REDHAT_BUGZILLA_PRODUCT_VERSION=8.3
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="8.3"
$ sudo hostnamectl set-hostname yuta_redhat
$ hostname
yuta_redhat
```


```
> cat /etc/os-release
NAME="SLES"
VERSION="15-SP2"
VERSION_ID="15.2"
PRETTY_NAME="SUSE Linux Enterprise Server 15 SP2"
ID="sles"
ID_LIKE="suse"
ANSI_COLOR="0;32"
CPE_NAME="cpe:/o:suse:sles:15:sp2"

> sudo hostnamectl set-hostname yuta_suse
> hostname
yuta_suse
```


# 参考文献
https://www.nic.ad.jp/ja/rfc-jp/WhatisRFC.html
https://jprs.jp/glossary/index.php?ID=0149
