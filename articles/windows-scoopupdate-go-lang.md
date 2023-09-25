---
title: "Scoopのアップデートからキャッシュ削除までまとめて実行プログラム"
emoji: "🥄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows","scoop","go","chatgpt"]
published: false
---

# 序論

[以前の記事](https://zenn.dev/yuta28/articles/windows-development-env)でも書きましたがWindowsでソフトウェアをインストールする際、Scoopを利用したコマンドラインベースのインストールを利用しています。

https://scoop.sh/

コマンドラインベースでツールをインストールするメリットとしてアップデートが簡単にできることがあります。GUIでツール提供元ページからインストールすると、最新バージョンへの更新のたびに提供元ページへアクセスして、ソフトウェアを再インストールする必要があります。
Scoop経由でインストールすればターミナル上でコマンド実行することで複数ソフトウェアを一括でインストールできますので抜け漏れがなくなります。

Macを利用されている人でしたらHomebrewをつかってソフトウェアのバージョンアップデートをしているかと思います。

```terminal:homebrew更新
$ brew update && brew upgrade
```

ScoopもHomebrewと似たようなコマンドでソフトウェアのアップデートができます。

```powershell:scoop更新
scoop update && scoop update *
```

ただ更新後も古いパッケージがキャッシュで残っているのでクリーンアップして削除します。

```powershell:scoopクリーンアップ
scoop cleanup * && scoop cache rm *
```

毎回アップデートしてクリーンアップコマンドをターミナルから実行するのも面倒だと感じましたので、更新からパッケージのクリーンアップまでを一括で実行したいなと思いました。
自分で実装できればいいのですが、プログラミングが苦手でしたので思い切ってChatGPTに全部作ってもらって動くかどうか検証してみました。

# 対象読者

- Scoopインストーラーを使っている
- パッケージのアップデートを楽にしたい

# 実行プログラム
実行するプログラミング言語は~~得意な言語がなかったので~~なんでもよかったのですが、バージョン互換やランタイムの有無を気にする必要がないGoで作ってみようと思いました。

最初にChatGPTに聞いて作られたプログラムがこちらです。

```go:scoop-package-to-cleanup.go
package main

import (
	"bytes"
	"fmt"
	"os"
	"os/exec"
)

func runCommand(name string, args ...string) error {
	cmd := exec.Command(name, args...)

	var stdout, stderr bytes.Buffer
	cmd.Stdout = &stdout
	cmd.Stderr = &stderr

	err := cmd.Run()

	if err != nil {
		return fmt.Errorf("Error executing %s: %s\n%s", name, err, stderr.String())
	}

	fmt.Printf("Command %s output:\n%s\n", name, stdout.String())
	return nil
}

func main() {
	if err := runCommand("scoop", "update"); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	if err := runCommand("scoop", "update", "*"); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	if err := runCommand("scoop", "cleanup", "*"); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	if err := runCommand("scoop", "cache", "rm", "*"); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	fmt.Println("All commands executed successfully.")
}
```

`os/exec`パッケージで外部コマンドを呼び出します。PowerShellコマンドを呼び出し、ScoopのコマンドをGoで実行できるようにします。
単純にScoopコマンドを4回呼び出して次々に実行するだけのシンプルなプログラムです。これをビルドしてバイナリファイルとして実行できるようにしましたが、プログラム実行中の間ターミナル上に何も変化がなく実行完了したら直ぐにターミナルが閉じてしまうため正しく動いているのかわかりにくいものでした。

ですのプログレスバーを実装して処理がどれくらいまで進んでいるのか把握できるようにしました。
Goでプログレスバーを実装する方法は以下の記事を参考にして同様にChatGPTにプログラムを作成してもらいました。
https://qiita.com/Akazawa_Naoki/items/a63193e3ac4c8cd4f19a

そして出来上がったプログラムがこちらです。

https://github.com/Yuhta28/Scoop-update-to-cleanup/blob/main/scoop-update-to-cleanup.go

# 参考文献
https://pkg.go.dev/os/exec