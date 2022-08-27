---
title: "Ansible経由でUbuntuにDatadogをインストールするときの注意点"
emoji: "🐶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["datadog","ubuntu","ansible"]
published: false
---

# 概要
会社で新しくDatadogを導入しようとしたときにちょっとしたトラップに遭遇しましたので詳細と回避策について紹介します。

# Datadog
https://www.datadoghq.com/ja/
詳しい説明はドキュメントなり他の人が書いた記事などがありますので、割愛しますが監視SaaSとして人気のあるサービスです。
モニタリング以外にもログ分析やトレーシングなどがありまして弊社でもメトリクス監視以外の機能も活用しようとして導入を進めている最中です。
いわゆるエージェント方式で監視対象ホストにエージェントを導入することで、Datadogの管理画面にメトリクスやログ、プロセス情報を収集できます。公式でインストール方法が細かく紹介されていまして、直接ホストにインストールする方法以外にもAnsible経由やDockerでのインストール方法も紹介されています。
https://docs.datadoghq.com/ja/agent/

# インストール
弊社では複数のメディアサーバーに共通のミドルウェアを導入するときにAnsibleの入ったDockerコンテナからインストールするようにしています。
![](/images/datadog-install-on-ubuntu-by-ansible/image1.png)

Datadogをインストールするロールそのものも公式から提供されているのでそれをダウンロードしてplaybookに追加すれば簡単にできると考えました。

## Ansible用インストール手順
https://docs.datadoghq.com/ja/agent/basic_agent_usage/ansible/
Ansibleを実行するホストでAnsible Galaxy上に存在する公式のDatadogロールをダウンロードします。
ちなみにこのロールはどこに格納されるかというとデフォルトではホームディレクトリ配下にある`.ansible`ディレクトリ配下に保存されます。
```console
% ansible-galaxy install datadog.datadog
Starting galaxy role install process
- downloading role 'datadog', owned by datadog
- downloading role from https://github.com/DataDog/ansible-datadog/archive/4.16.0.tar.gz
- extracting datadog.datadog to /home/yuta0128/.ansible/roles/datadog.datadog
- datadog.datadog (4.16.0) was installed successfully
```

その後、Playbookにロール追加とDatadogセットアップ時に発行されるAPIキーを追加します。

```yml: Playbook
- hosts: servers
  roles:
    - { role: datadog.datadog, become: yes }
  vars:
    datadog_api_key: "<DD_API_キー>"
```

準備はこれだけです。このあとにECSコンテナをデプロイし直せばメディアサーバーにDatadogがインストールされるはずでした。

# 失敗
途中でエラーが生じて、Ansibleの処理が止まってしまいました。

```console
TASK [datadog.datadog : Include Gather Ansible Facts task on Ansible >= 2.10] ***********************************************************
skipping: [172.27.13.181]

TASK [datadog.datadog : Include Gather Ansible Facts task on Ansible < 2.10] ************************************************************
included: /home/ubuntu/.ansible/roles/datadog.datadog/tasks/facts-ansible9.yml for 172.27.13.181

TASK [datadog.datadog : Gather Ansible Facts] *******************************************************************************************
ok: [172.27.13.181]

TASK [datadog.datadog : Check if OS is supported] ***************************************************************************************
included: /home/ubuntu/.ansible/roles/datadog.datadog/tasks/os-check.yml for 172.27.13.181

省略
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
省略

TASK [datadog.datadog : Linux Configuration Tasks (Agent 5)] ****************************************************************************
skipping: [172.27.13.181]

TASK [datadog.datadog : Linux Configuration Tasks] **************************************************************************************
included: /home/ubuntu/.ansible/roles/datadog.datadog/tasks/agent-linux.yml for 172.27.13.181

TASK [datadog.datadog : Populate service facts] *****************************************************************************************
fatal: [172.27.13.181]: FAILED! => {"changed": false, "msg": "Malformed output discovered from systemd list-unit-files: accounts-daemon.service                        enabled         enabled      "}

PLAY RECAP ******************************************************************************************************************************
172.27.13.181              : ok=20   changed=0    unreachable=0    failed=1    skipped=22   rescued=0    ignored=0
```

Datadogのロールを確認したところ以下のタスクの処理でエラーが起きていました。

```yml: aget-linux.yml
- name: Populate service facts
  service_facts:
```

https://docs.ansible.com/ansible/2.9_ja/modules/service_facts_module.html

service_factsモジュールはサービス情報を収集するモジュールですが、`accounts-daemon`というサービスで不正な出力が発見されたことから`failed`エラーとなって処理が止まったように思われます。
実はこの挙動はバグで、AnsibleのGitHubにもissueとして挙がっていました。
https://github.com/ansible/ansible/issues/68536

バグ自体はver2.9.8で修正済みですが、ECSコンテナで動かしているLinuxディストリビューションのUbuntu20.04ではAPTパッケージからver2.9.6までしか取得できませんでした。

```console:Ubuntu 20.04
NAME="Ubuntu"
VERSION="20.04.3 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.3 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal

$ ansible --version
ansible 2.9.6
  config file = /home/ubuntu/infra/ansible/ansible.cfg
  configured module search path = ['/home/ubuntu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Jun 22 2022, 20:18:18) [GCC 9.4.0]

$ apt search ansible
Sorting... Done
Full Text Search... Done
ansible/focal,now 2.9.6+dfsg-1 all [installed]
  Configuration management, deployment, and task execution system
```

# 参考文献
https://github.com/ansible/ansible/blob/stable-2.9/changelogs/CHANGELOG-v2.9.rst#v2-9-8