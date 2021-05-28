---
title: "Linuxのホスト名について調査してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux","hostname","RFC"]
published: true
---

# 概要
マシンを扱ううえで大切なホスト名の命名規則について調べてみました。

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

インターネットホストテーブルの仕様についてまとめたRFC 952には以下の内容が書かれています。

>  A "name" (Net, Host, Gateway, or Domain name) is a text string up
>   to 24 characters drawn from the alphabet (A-Z), digits (0-9), minus
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
最初の文字は英字でなければなりません。最後の文字にハイフンやピリオドは使用できません。

ゲートウェイの機能を持つ機器には"-GATEWAY"や"-GW"の文字を入れるべきです。
逆にゲートウェイの機能を持たないのに、"-GATEWAY"や"-GW"の文字は入れるべきではありません。
1文字だけのホスト名は使用できません。
:::

UbuntuはこのRFCのルールを準拠しているため、ホスト名にアンスコを入れても無視されるというわけです。
ただこれらのルールすべてを厳密に守っているわけではなく、例えばホスト名が数字のみでも問題ありませんでした。

```bash
$ sudo hostnamectl set-hostname 11111
$ hostname
11111
```
また24文字以上でもホスト名を設定できることが分かりました。

```bash
$ sudo hostnamectl set-hostname abcdefghijklmnopqrstuvwxyz
$ hostname
abcdefghijklmnopqrstuvwxyz
```

どこまでいけるのか試してみたところ64文字まで設定できました。
これはDNSのラベルの文字数制限[^2]と同じです。
[^2]:https://jprs.jp/glossary/index.php?ID=0149

気になったのが、以前別のOSを使ったときはアンスコがホスト名に設定できましたので、OSごとに遵守されているルールが違うのではないかと思い、色んなOSを使って検証してみることにしました。

# 調査方法
調査方法としてRFC 952で定められているルールが遵守されているか確認するため、以下の違反行為が許容されるかどうか検証していきます。
### ルール違反行為一覧

1. ホスト名にアンスコ(_)、アットマーク(@)、アスタリスク(*)を使えるか
2. ホスト名に日本語は使えるか
3. ホスト名の先頭に英字以外の文字が使えるか
4. ホスト名の最後にハイフン(-)、ピリオド(.)が使えるか
5. ホスト名の文字数に24文字以上設定できるか
6. 空白文字が使えるか

### 検証対象OS

- Ubuntu 20.04
- Amazon Linux 2
- Red Hat Enterprise Linux 8.3
- openSUSE 15.2
- Debian GNU/Linux 10

OSはAWSのマーケットプレイス上にあるOSを適当にチョイスしました。
では始めていきます。

# 検証

## Ubuntu 20.04


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

$ hostname #初期ネーム
Yuta-Ubuntu

$ sudo hostnamectl set-hostname Yuta_Ubuntu@PC*AWS #ルール①検証
$ hostname
YutaUbuntuPCAWS

$ sudo hostnamectl set-hostname ユータ #ルール②検証
$ hostname
localhost

$ sudo hostnamectl set-hostname 12345 #ルール③検証
$ hostname
12345

$ sudo hostnamectl set-hostname Yuta- #ルール④検証
$ hostname
Yuta

$ sudo hostnamectl set-hostname yuta* #ルール④検証
$ hostname
yuta

$ sudo hostnamectl set-hostname 1223334444555556666667777777888888889999999990000000000abcdefghijklmn #ルール⑤検証
$ hostname
1223334444555556666667777777888888889999999990000000000abcdefghi

$ sudo hostnamectl set-hostname "Yuta Ubuntu" #ルール⑥検証
$ hostname
YutaUbuntu
```

## Amazon Linux2

```bash
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

$ hostname #初期ネーム
yuta-azln2

$ sudo hostnamectl set-hostname Yuta_Amazon@Linux2*AWS #ルール①検証
$ hostname
yuta_amazonlinux2aws

$ sudo hostnamectl set-hostname ユータ #ルール②検証
$ hostname
localhost

$ sudo hostnamectl set-hostname 12345 #ルール③検証
$ hostname
12345

$ sudo hostnamectl set-hostname Yuta- #ルール④検証
$ hostname
yuta-

$ sudo hostnamectl set-hostname Yuta* #ルール④検証
$ hostname
yuta

$ sudo hostnamectl set-hostname 1223334444555556666667777777888888889999999990000000000abcdefghijklmn #ルール⑤検証
$ hostname
1223334444555556666667777777888888889999999990000000000abcdefghi

$ sudo hostnamectl set-hostname "Yuta amzn2" #ルール⑥検証
$ hostname
yutaamzn2

```

## Red Hat Enterprise Linux 8.3

```bash
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

$ hostname #初期ネーム
yuta-redhat

$ sudo hostnamectl set-hostname yuta_redhat@Enterprize*linux　#ルール①検証
$ hostname
yuta_redhatEnterprizelinux

$ sudo hostnamectl set-hostname ユータ #ルール②検証
$ hostname
ip-10-0-2-31

$ sudo hostnamectl set-hostname 12345 #ルール③検証
$ hostname
12345

$ sudo hostnamectl set-hostname Yuta- #ルール④検証
$ hostname
Yuta-

$ sudo hostnamectl set-hostname yuta* #ルール④検証
$ hostname
yuta

sudo hostnamectl set-hostname 1223334444555556666667777777888888889999999990000000000abcdefghijklmn #ルール⑤検証
$ hostname
1223334444555556666667777777888888889999999990000000000abcdefghi

$ sudo hostnamectl set-hostname "Yuta Redhat" #ルール⑥検証
$ hostname
YutaRedhat
```

## openSUSE 15.2

```bash
> cat /etc/os-release
NAME="SLES"
VERSION="15-SP2"
VERSION_ID="15.2"
PRETTY_NAME="SUSE Linux Enterprise Server 15 SP2"
ID="sles"
ID_LIKE="suse"
ANSI_COLOR="0;32"
CPE_NAME="cpe:/o:suse:sles:15:sp2"

> hostname #初期ネーム
yuta-opensuse

> sudo hostnamectl set-hostname yuta_open@su*se #ルール①検証
> hostname
yuta_opensuse

> sudo hostnamectl set-hostname ユータ #ルール②検証
> hostname
localhost

> sudo hostnamectl set-hostname 12345 #ルール③検証
> hostname
12345

> sudo hostnamectl set-hostname Yuta- #ルール④検証
> hostname
Yuta-

> sudo hostnamectl set-hostname yuta* #ルール④検証
> hostname
yuta

> sudo hostnamectl set-hostname 1223334444555556666667777777888888889999999990000000000abcdefghijklmn #ルール⑤検証
> hostname
1223334444555556666667777777888888889999999990000000000abcdefghi

> sudo hostnamectl set-hostname "Yuta openSUSE" #ルール⑥検証
> hostname
YutaopenSUSE
```

## Debian GNU/Linux 10

```bash
$ cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 10 (buster)"
NAME="Debian GNU/Linux"
VERSION_ID="10"
VERSION="10 (buster)"
VERSION_CODENAME=buster
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"

$ hostname #初期ネーム
yuta-debian

$ sudo hostnamectl set-hostname yuta_de*bi@an #ルール①検証
$ hostname
yutadebian

$ sudo hostnamectl set-hostname ユータ #ルール②検証
$ hostname
localhost

$ sudo hostnamectl set-hostname 12345 #ルール③検証
$ hostname
12345

$ sudo hostnamectl set-hostname Yuta- #ルール④検証
$ hostname
Yuta

$ sudo hostnamectl set-hostname yuta* #ルール④検証
$ hostname
yuta

$ sudo hostnamectl set-hostname 1223334444555556666667777777888888889999999990000000000abcdefghijklmn #ルール⑤検証
$ hostname
1223334444555556666667777777888888889999999990000000000abcdefghi

$ sudo hostnamectl set-hostname "Yuta debian" #ルール⑥検証
$ hostname
Yutadebian
```
# 調査結果
複数のOSに対して検証した結果以下の調査結果になりました。

| ルール | Ubuntu | Amazon Linux | RHEL | SUSE | Debian |
| ----   | :----: | :----:           | :----:  | :----: | :----:   |
| ルール検証① | ☓ | △ | △ | △ | ☓ |
| ルール検証② | ☓ | ☓ | ☓ | ☓ | ☓ |
| ルール検証③ | ○ | ○ | ○ | ○ | ○ |
| ルール検証④ | ☓ | △ | △ | △ | ☓ |
| ルール検証⑤ | ○ | ○ | ○ | ○ | ○ |
| ルール検証⑥ | ☓ | ☓ | ☓ | ☓ | ☓ |

- ○：ルールを無視できた
- △：一部だけ無視できなかった
- ☓ ：ルールを無視できなかった

こうして確認しますとDebian系の2つは少しルールに厳しく、Fedora系はある程度ルールに寛容ということがわかります。
OpenSUSEもFedora系と同じですね。

面白かったのが、Amazon Linux2の挙動です。
Amazon Linux2では大文字小文字の区別がつかないのか大文字が小文字に変換されました。
これはAWSのドキュメントにも記載が見つからずどういう根拠でこのような挙動なのか気になりました。
ほかにもRed Hat Enterprise Linuxではホスト名に日本語を設定したらDNS名に変更されたこともユニークな動きでした。
ほかのOSでは`localhost`になりましたが、ここだけ違う動きをしていたのが興味深いです。

# 所感
普段あまり意識しないホスト名の命名規則について調査してみました。
どの会社でもホスト名の命名規則を定めるかと思いますが、RFCに則った命名規則にしないと思わぬトラブルに巻き込まれることもあり得ますので、皆さまも一度調べてみると面白いかもしれません。

# 参考文献
https://saku-jp.hatenablog.com/entry/20130612/p1
