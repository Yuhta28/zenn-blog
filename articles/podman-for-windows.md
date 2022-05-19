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
4系の新機能にWindowsホストでPodmanが利用できるようになったとアナウンスがありました。
> The podman machine init command now supports a new VM type, wsl, available only on Windows; this uses WSL as a backend for podman machine, instead of creating a separate VM and managing it via QEMU

WSLでLinuxディストリビューションを起動しなくてもWindowsでもPodmanが使えるとのことですので、今回はWindowsでPodmanを動かしてみました。おまけで、podman-composeもWindowsで動かしてみました。


# Podmanについておさらい
https://podman.io/
以前書いた記事と重複しますが、改めてPodmanについて説明いたします。Podmanとはdockerコマンドと互換性のあるコンテナエンジンです。デーモンレスで動かすことができるため、リソース消費を抑えられるというメリットがあります。



# 参考文献
https://crunchtools.com/should-i-use-docker-compose-or-podman-compose-with-podman/
https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/index