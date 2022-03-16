---
description: integers
---

# 整数

[**この章のすべてのコードはここにあります**](https://github.com/andmorefine/learn-go-with-tests/tree/master/integers)

整数は期待どおりに機能します。物事を試すために `Add` 関数を書いてみましょう。 `adder_test.go` というテストファイルを作成し、このコードを記述します。

**注:** Goソースファイルは、ディレクトリごとに1つの `package` のみを持つことができます。ファイルが個別に編成されていることを確認してください。[説明](https://dave.cheney.net/2014/12/01/five-suggestions-for-setting-up-a-go-project)

## 最初にテストを書く

```go
package integers

import "testing"

func TestAdder(t *testing.T) {
    sum := Add(2, 2)
    expected := 4

    if sum != expected {
        t.Errorf("expected '%d' but got '%d'", expected, sum)
    }
}
```

`%q` ではなく、`%d`をフォーマット文字列として使用していることに気付くでしょう。これは、文字列ではなく整数を出力するためです。

また、メインパッケージを使用していないことに注意してください。代わりに、`integers`という名前のパッケージを定義しました。これは、名前がこれにより、`Add`などの整数を操作するための関数をグループ化することを示唆しているためです。

## テストを試して実行する

`go test`を実行します

コンパイルエラーを検査する

`./adder_test.go:6:9: undefined: Add`

## テストを実行するための最小限のコードを記述し、失敗したテスト出力を確認します

コンパイラを満足させるのに十分なコードを記述してください。これですべてです。 テストが正しい理由で失敗したことを確認したいことに注意してください。

```go
package integers

func Add(x, y int) int {
    return 0
}
```

`(x int, y int)` ではなく、同じタイプの`（この場合は2つの整数）`の引数が複数ある場合は、 `(x, y int)`に短縮できます。

ここでテストを実行すると、テストが何が悪いのかを正しく報告していることに満足するはずです。

`adder_test.go:10: expected '4' but got '0'`

結果の意味がコンテキストから明確でない場合に一般的に使用する必要があります。この場合、`Add`関数がパラメーターを追加することはかなり明確です。 詳細については、[wiki](https://github.com/golang/go/wiki/CodeReviewComments#named-result-parameters) を参照してください。

## 成功させるのに十分なコードを書く

TDDの最も厳密な意味では、テストをパスするための最小限のコードを書く必要があります。 知識のあるプログラマはこれを行うかもしれません。

```go
func Add(x, y int) int {
    return 4
}
```

ああ！再び失敗した！TDDは意味あるんですか？

別のテストを作成して、いくつかの異なる数字を使用してそのテストを強制的に失敗させることもできますが、 [猫とマウスのゲーム](https://en.m.wikipedia.org/wiki/Cat_and_mouse)のように感じられます。

Goの構文に慣れてきたら、「プロパティベーステスト」`Property Based Testing`と呼ばれる手法を紹介します。 これは、開発者を煩わせずにバグを見つけるのに役立ちます。

とりあえず、きちんと直しましょう。

```go
func Add(x, y int) int {
    return x + y
}
```

テストを再実行すると、合格するはずです。

## リファクタリング♪

ここで実際に改善できるコードは多くありません。

戻り引数に名前を付けることにより、ドキュメントだけでなくほとんどの開発者のテキストエディターにもどのように表示されるかについては、以前に説明しました。

あなたが書いているコードの使いやすさを助けるので、これは素晴らしいです。 型シグネチャとドキュメントを見るだけで、ユーザーがコードの使用法を理解できることが望ましいです。

コメント付きの関数にドキュメントを追加できます。コメントは、標準ライブラリのドキュメントと同じようにGo Docに表示されます。

```go
// Add takes two integers and returns the sum of them.
func Add(x, y int) int {
    return x + y
}
```

### 例

一歩進んだら [例](https://blog.golang.org/examples)にしてください。標準ライブラリのドキュメントに多くの例があります。

多くの場合、READMEファイルなど、コードベースの外部にあるコード例は、チェックされないため、実際のコードと比較して古くなり、正しくありません。

Goの例はテストと同じように実行されるため、コードが実際に行っていることを確実に例に反映できます。

例は、パッケージのテストスイートの一部としてコンパイル（およびオプションで実行）されています。

典型的なテストと同様に、例はパッケージの `_test.go`ファイルにある関数です。次の`ExampleAdd`関数を`adder_test.go`ファイルに追加します。

```go
func ExampleAdd() {
    sum := Add(1, 5)
    fmt.Println(sum)
    // Output: 6
}
```

\(エディターがパッケージを自動的にインポートしない場合、 `adder_test.go`に`import "fmt"`がないため、コンパイルが失敗します。これらの種類のエラーの発生方法を調査することを強くお勧めします。本来なら使用しているエディターで自動的に修正されます。\)

サンプルが無効になるようにコードが変更された場合、ビルドは失敗します。

パッケージのテストスイートを実行すると、サンプル関数がそれ以上の調整なしで実行されていることがわかります。

```bash
$ go test -v
=== RUN   TestAdder
--- PASS: TestAdder (0.00s)
=== RUN   ExampleAdd
--- PASS: ExampleAdd (0.00s)
```

コメント `//Output: 6`を削除すると、サンプル関数は実行されないことに注意してください。 関数はコンパイルされますが、実行されません。

このコードを追加することにより、サンプルは `godoc`内のドキュメントに表示され、コードにさらにアクセスしやすくなります。

これを試すには、 `godoc -http=:6060` を実行して `http://localhost:6060/pkg/`にアクセスします

ここには、 `$GOPATH`内のすべてのパッケージのリストが表示されるので、このコードを`$GOPATH/src/github.com/{your_id}`のような場所に記述したとする場合のドキュメントの例です。

例を含むコードをパブリックURLに公開する場合は、 [pkg.go.dev](https://pkg.go.dev/)でコードのドキュメントを共有できます。たとえば、[ここ](https://pkg.go.dev/github.com/quii/learn-go-with-tests/integers/v2) は、この章の最終的なAPIです。このWebインターフェイスを使用すると、標準ライブラリパッケージおよびサードパーティパッケージのドキュメントを検索できます。

## まとめ

ここで学んだこと

* テスト駆動開発（TDD）ワークフローのさらなる実践
* 整数、加算
* より良いドキュメントを作成して、コードのユーザーがその使用法をすばやく理解できるようにする
* テストの一環としてチェックされるコードの使用例
