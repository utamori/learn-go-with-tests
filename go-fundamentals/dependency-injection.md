---
description: Dependency Injection
---

# 依存性の注入

[**この章のすべてのコードはここにあります**](https://github.com/andmorefine/learn-go-with-tests/tree/main/di)

これにはインターフェースの理解が必要になるため、構造体のセクションをすでに読んでいることが前提です。

プログラミングコミュニティには、依存性の注入に関する誤解がたくさんあります。

このガイドでは、次のことを説明します

* DIにフレームワークは必要ありません
* DIは設計を複雑にしすぎることはありません
* DIはテストを容易にします
* DIを使えば、素晴らしい汎用的な関数が書けるようになります

`hello-world`の章で行ったように、誰かに挨拶する関数を書きたいのですが、今回は _実際の出力_ をテストします。

要約すると、その関数は次のようになります。

```go
func Greet(name string) {
    fmt.Printf("Hello, %s", name)
}
```

しかし、これをどのようにテストできますか？
`fmt.Printf`を呼び出すと _stdout_ に出力されますが、テストフレームワークを使用してキャプチャするのはかなり困難です。

私たちがする必要があるのは、Printの依存関係を**注入**できるようにすることです(これは特別な意味を持った言葉です)。
私たちの関数は、Printが _**どこで**_ _**どのように**_ 行われるかを**気にする必要はない**ので、**具象型ではなく** **インターフェイス** を受け入れる必要があります。
そうすれば、私たちがコントロールするものにPrintするように実装を変更することができ、テストすることができます
現実の世界では、stdoutに書き込むようなものを注入することになるでしょう。

[`fmt.Printf`](https://pkg.go.dev/fmt#Printf)のソースコードを見ると、フックする方法がわかります。

```go
// It returns the number of bytes written and any write error encountered.
func Printf(format string, a ...interface{}) (n int, err error) {
    return Fprintf(os.Stdout, format, a...)
}
```

面白いですね！
内部では、 `Printf`は`os.Stdout`を渡して `Fprintf`を呼び出しているだけです。

`os.Stdout`とは正確に何ですか？
`Fprintf`は第1引数として何が渡されることを期待していますか？

```go
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {
    p := newPrinter()
    p.doPrintf(format, a)
    n, err = w.Write(p.buf)
    p.free()
    return
}
```

`io.Writer` です

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

このことから、 `os.Stdout` は `io.Writer` を実装していると推測できます。`Printf` は `io.Writer` を期待している`Fprintf` に`os.Stdout` を渡しています。

このインターフェースは「このデータをどこかに置いておく」ための汎用的なインターフェースなので、Goのコードを書いているとよく出てくるようになります。

つまり、私たちは最終的に `Writer`を使用して挨拶をどこかに送信していることを知っています。この既存の抽象化を使用して、コードをテスト可能にし、再利用しやすくしてみましょう。

## 最初にテストを書く

```go
func TestGreet(t *testing.T) {
    buffer := bytes.Buffer{}
    Greet(&buffer, "Chris")

    got := buffer.String()
    want := "Hello, Chris"

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```

`bytes`パッケージの`Buffer`型は`Write(p []byte) (n int, err error)`メソッドを持っているので、 `Writer`インターフェースを実装しています。

テストでこれを使用して`Writer`として送信し、`Greet`を呼び出した後に何が書き込まれたかを確認できます。

## テストを試して実行する

テストはコンパイルされません

```text
./di_test.go:10:7: too many arguments in call to Greet
    have (*bytes.Buffer, string)
    want (string)
```

## テストを実行するための最小限のコードを記述し、失敗したテスト出力を確認します

コンパイラを読んで、問題を修正してください。

```go
func Greet(writer *bytes.Buffer, name string) {
    fmt.Printf("Hello, %s", name)
}
```

`Hello, Chris di_test.go:16: got '' want 'Hello, Chris'`

テストは失敗します。名前は出力されますが、標準出力になることに注意してください。

## 成功させるのに十分なコードを書く

このテストでは、バッファに挨拶を送信するためにwriterを使用します。`fmt.Fprintf`は`fmt.Printf`と似ていますが、`fmt.Printf`がstdoutがデフォルトなのに対し、代わりに文字列を送信する`Writer`を取ることを覚えておいてください。

```go
func Greet(writer *bytes.Buffer, name string) {
    fmt.Fprintf(writer, "Hello, %s", name)
}
```

テストに合格しました。

## リファクタリング♪

以前のコンパイラーは、`bytes.Buffer`へのポインターを渡すように指示しました。これは技術的には正しいですが、あまり役に立ちません。

これを実証するために、`Greet`関数を標準出力に出力するGoアプリケーションに接続してみてください。

```go
func main() {
    Greet(os.Stdout, "Elodie")
}
```

`./di.go:14:7: cannot use os.Stdout (type *os.File) as type *bytes.Buffer in argument to Greet`

先に説明したように、`fmt.Fprintf` は `os.Stdout` と `bytes.Buffer` が実装している `io.Writer` を渡すことができます。

より汎用的なインターフェースを使用するようにコードを変更すると、テストとアプリケーションの両方で使用できるようになります。

```go
package main

import (
    "fmt"
    "io"
    "os"
)

func Greet(writer io.Writer, name string) {
    fmt.Fprintf(writer, "Hello, %s", name)
}

func main() {
    Greet(os.Stdout, "Elodie")
}
```

## io.Writerの詳細

`io.Writer`を使用して他にどのような場所にデータを書き込むことができますか？
`Greet`関数はどのくらい汎用的ですか？

### インターネット

以下を実行します

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
)

func Greet(writer io.Writer, name string) {
    fmt.Fprintf(writer, "Hello, %s", name)
}

func MyGreeterHandler(w http.ResponseWriter, r *http.Request) {
    Greet(w, "world")
}

func main() {
	log.Fatal(http.ListenAndServe(":5000", http.HandlerFunc(MyGreeterHandler)))
}
```

プログラムを実行し、[http://localhost:5000](http://localhost:5000)に移動します。Greet関数が使用されているのがわかります。

HTTPサーバーについては後の章で説明しますので、詳細についてはあまり気にしないでください。

HTTPハンドラーを作成すると、 `http.ResponseWriter`と、リクエストの作成に使用された` http.Request`が与えられます。サーバーを実装するときは、Writer を使用してレスポンスを _書きこみ_ ます。

`http.ResponseWriter`も`io.Writer`を実装しているので、ハンドラー内で `Greet`関数を再利用できます。

## まとめ

最初のコードは、制御できない場所にデータを書き込んだため、簡単にテストできませんでした。

テストによって動機付けされたコードをリファクタリングして、制御できるようにしました。
テスト駆動により、私たちはコードをリファクタリングし、データの書き込み先を制御できるように**依存関係を注入**しました。これにより、私たちは以下のことができるようになりました。

_Motivated by our tests_ we refactored the code so we could control _where_ the data was written by **injecting a dependency** which allowed us to:

* **コードをテストする**関数を簡単にテストできない場合は、たいていは依存関係が関数またはグローバルな状態に組み込まれていることが原因です。たとえば、ある種のサービス層で使用されているグローバルなデータベース接続プールがある場合、テストが困難になる可能性が高く、実行が遅くなります。DIは、（インターフェイスを介して）データベースの依存関係を注入し、テストで制御できるものでモック化しやすくします。
* **関心の分離**を行い、「データの行き先」と「生成方法」を分離します。メソッド/関数の責務が多すぎると感じた場合は、（データの生成、およびデータベースへの書き込み、HTTPリクエストの処理、およびドメインレベルのロジックの実行）おそらくDIが必要になるでしょう。
* **コードをさまざまなコンテキストで再利用できるようにする** コードを使用できる最初の「新しい」コンテキストは、テスト内です。しかし、さらに誰かがあなたの関数で何か新しいことを試したい場合、彼らは彼ら自身の依存関係を注入できます。

### モックってどうなの？ DIにも必要だそうですが、それも悪だそうです。

モックについては後で詳しく説明します（そしてそれは悪ではありません）。
モックを使用して、実際に注入するものを、テストで制御および検査できる偽バージョンに置き換えます。
私たちの場合でも、標準ライブラリには、使用する準備ができています。

### Go標準ライブラリは本当に良いです。時間をかけて勉強してください。

このように`io.Writer`インターフェースにある程度慣れていることで、テストで`bytes.Buffer`を `Writer`として使うことができ、標準ライブラリの他の`Writer`を使ってコマンドラインアプリやウェブサーバで関数を使うことができます。

標準ライブラリに慣れるほど、これらの汎用インターフェイスを目にすることが多くなり、自分のコードで再利用することでソフトウェアの再利用が可能になります。

この例は、[プログラミング言語Go](https://www.amazon.co.jp/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E8%A8%80%E8%AA%9EGo-ADDISON-WESLEY-PROFESSIONAL-COMPUTING-Donovan/dp/4621300253/ref=sr_1_6?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&dchild=1&keywords=Go+Programming+Language&qid=1592323254&sr=8-6), の章に大きく影響されているため、これを楽しんだ場合、是非買ってみてください！
