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

WindowsでもHomebrew[^1]みたいにCLIベースでソフトウェア管理できるので、コマンド一発で一括更新できて大変便利です。とは言え毎回手動で更新コマンドを実行して、古いパッケージのクリーンアップコマンドを実行するのは面倒でしたので、更新からキャッシュ削除までをまとめて実行するプログラムをChatGPT君に作ってもらいました。

[^1]: https://brew.sh/

# 対象読者

- Scoopインストーラーを使っている
- アップデート、クリーンアップを効率的にしたい

# 実行プログラム
プログラミング言語は~~得意な言語がなかったので~~なんでもよかったのですが、バージョン互換やランタイムの有無を気にする必要がないGoにしました。

最初にChatGPTが作ったプログラムがこちらです。

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

`os/exec`パッケージで外部コマンドを呼び出し、GoでScoopのコマンドを実行できるようにします。
`go build`でビルドしてバイナリファイルにしましたが、プログラムの実行中何も変化がなく実行完了したら直ぐにターミナルが閉じてしまうため正しく動いているのかわかりにくいものでした。

ですのでプログレスバーを実装して処理がどれくらいまで進んでいるのか把握できるようにしました。
Goでプログレスバーを実装する方法は以下の記事を参考にして同様にChatGPTにプログラムを作成してもらいました。
https://qiita.com/Akazawa_Naoki/items/a63193e3ac4c8cd4f19a

そして出来上がったプログラムがこちらです。

```go:scoop-update-to-cleanup.go
package main

import (
	"fmt"
	"os"
	"os/exec"
	"strings"

	"github.com/schollz/progressbar/v3"
)

func main() {
	// コマンドを作成
	cmd := exec.Command("cmd.exe", "/C", "scoop update && scoop update *")

	// 標準出力をキャプチャ
	stdoutPipe, _ := cmd.StdoutPipe()

	// コマンドを実行
	err := cmd.Start()
	if err != nil {
		fmt.Println("コマンドの実行中にエラーが発生しました:", err)
		return
	}

	// プログレスバーの設定
	bar := progressbar.NewOptions(-1,
		progressbar.OptionSetDescription("Scoopアップデート中"),
		progressbar.OptionSpinnerType(14),
		progressbar.OptionShowCount(),
		progressbar.OptionSetWidth(15),
		progressbar.OptionSetTheme(progressbar.Theme{
			Saucer:        "[green]=[reset]",
			SaucerHead:    "[green]>[reset]",
			SaucerPadding: " ",
			BarStart:      "[",
			BarEnd:        "]",
		}))

	// 標準出力からデータを読み取り、プログレスバーに反映
	buffer := make([]byte, 1024)
	for {
		n, err := stdoutPipe.Read(buffer)
		if err != nil {
			break
		}
		output := string(buffer[:n])
		// Scoopの出力に進捗情報が含まれているか確認
		if strings.Contains(output, "Updating") {
			bar.Add(1)
		}
		fmt.Print(output)
	}

	// コマンドの終了を待機
	err = cmd.Wait()
	if err != nil {
		fmt.Println("コマンドの実行中にエラーが発生しました:", err)
		return
	}

	fmt.Println("Scoopパッケージのアップデートが完了しました。")

	// クリーンアップとキャッシュ削除コマンドを作成
	cleanupCmd := exec.Command("cmd.exe", "/C", "scoop cleanup * && scoop cache rm *")
	
	// 標準出力をキャプチャ
	cleanupCmd.Stdout = os.Stdout
	cleanupCmd.Stderr = os.Stderr

	// クリーンアップとキャッシュ削除コマンドを実行
	err = cleanupCmd.Run()
	if err != nil {
		fmt.Println("クリーンアップコマンドの実行中にエラーが発生しました:", err)
		return
	}

	fmt.Println("古いパッケージのクリーンアップとキャッシュの削除が完了しました。")

	// プログラムが終了しないように待機
	fmt.Println("Enterキーを押してプログラムを終了してください...")
	var input string
	fmt.Scanln(&input)
}
```

# 参考文献
https://pkg.go.dev/os/exec