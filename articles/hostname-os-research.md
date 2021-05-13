---
title: "ホスト名にアンダースコアが使えないLinuxと使えるLinuxを調査してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux","hostname"]
published: false
---


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
