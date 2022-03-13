---
description: Mocking
---

# モック

[**この章のすべてのコードはここにあります**](https://github.com/andmorefine/learn-go-with-tests/tree/main/mocking)

3からカウントダウンするプログラムを作成するように求められました。各数値を新しい行に表示します（1秒の間隔を置いて）、ゼロに達すると「Go！」と表示します。そして終了します。

```text
3
2
1
Go!
```

これに取り組むには、`Countdown`という関数を作成します。この関数を`main`プログラム内に配置して、次のようにします。

```go
package main

func main() {
    Countdown()
}
```

これはかなり簡単なプログラムですが、完全にテストするには、いつものように反復的、テストドリブンのアプローチを取る必要があります。

反復とはどういう意味ですか？私たちは、有用なソフトウェアを入手するために、できる限り小さなステップを踏んでいることを確認します。

丁寧に解析すれば理論的に機能するコードに長い時間を費やしたくありません。開発者がうさぎの穴に陥ることが多いからです。 **要件をできる限り小さくスライスして、** _**動作するソフトウェア**_ **を作るのは重要なスキルです。**

作業を分割して反復する方法は次のとおりです。

* 3 を表示できるようにします
* 3、2、1 と Go！ を表示できるようにします
* 各行の間で1秒待ちます

## 最初にテストを書く

私たちのソフトウェアは標準出力に出力する必要があり、DIを使用してこれをテストする方法をDIセクションで確認しました。

```go
func TestCountdown(t *testing.T) {
    buffer := &bytes.Buffer{}

    Countdown(buffer)

    got := buffer.String()
    want := "3"

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```

`buffer`のようなものに不慣れな場合は、[前のセクション](dependency-injection.md)をもう一度お読みください。

`Countdown`関数でどこかにデータを書き込む必要があることはわかっています。`io.Writer`は、Goのインターフェースとしてデータをキャプチャするための一般的な方法です。

* `main`で`os.Stdout`に送信して、ターミナルに出力されたカウントダウンをユーザーに表示します。
* テストでは、`bytes.Buffer`に送信して、生成されるデータをテストでキャプチャできるようにします。

## テストを試して実行する

`./countdown_test.go:11:2: undefined: Countdown`

## テストを実行するための最小限のコードを記述し、失敗したテスト出力を確認します

`Countdown`を定義します

```go
func Countdown() {}
```

再試行

```go
./countdown_test.go:11:11: too many arguments in call to Countdown
    have (*bytes.Buffer)
    want ()
```

コンパイラーは、関数のシグニチャーが何であるかを通知しているので、更新してください。

```go
func Countdown(out *bytes.Buffer) {}
```

`countdown_test.go:17: got '' want '3'`

パーフェクト！

## 成功させるのに十分なコードを書く

```go
func Countdown(out *bytes.Buffer) {
    fmt.Fprint(out, "3")
}
```

`fm.Fprint`を使用しています。これは、`io.Writer`（`* bytes.Buffer`など）を受け取り、それに`string`を送信します。テストは成功するはずです。

## リファクタリング♪

`*bytes.Buffer`は機能しますが、代わりに汎用インターフェースを使用する方がよいことはわかっています。

```go
func Countdown(out io.Writer) {
    fmt.Fprint(out, "3")
}
```

テストを再実行すると、テストに合格するはずです。

問題を完了するために、関数を`main`に結び付けましょう。そうすれば、作業を進めていることを確信できる実用的なソフトウェアができます。

```go
package main

import (
    "fmt"
    "io"
    "os"
)

func Countdown(out io.Writer) {
    fmt.Fprint(out, "3")
}

func main() {
    Countdown(os.Stdout)
}
```

プログラムを試して実行すると、あなたの便利な作業に驚かされます。

はい、これはささいなことのようですが、このアプローチはどのプロジェクトにもお勧めします。
**機能のごく一部を取得し、テストに裏打ちされた`end-to-end`で機能するようにします。**

次に、2,1を表示してから、「Go！」を表示します。

## 最初にテストを書く

配管全体を正しく機能させるために投資することで、ソリューションを安全かつ簡単に反復できます。
すべてのロジックがテストされるので、プログラムが機能していることを確認するために、プログラムを停止して再実行する必要がなくなります。

```go
func TestCountdown(t *testing.T) {
    buffer := &bytes.Buffer{}

    Countdown(buffer)

    got := buffer.String()
    want := `3
2
1
Go!`

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```

バックティック構文は`string`を作成するもう1つの方法ですが、テストに最適な改行などを配置できます。

## テストを試して実行する

```text
countdown_test.go:21: got '3' want '3
        2
        1
        Go!'
```

## 成功させるのに十分なコードを書く

```go
func Countdown(out io.Writer) {
    for i := 3; i > 0; i-- {
        fmt.Fprintln(out, i)
    }
    fmt.Fprint(out, "Go!")
}
```
`i--`で逆にカウントする`for`ループを使用し、`fmt.Fprintln`を使用して、番号と改行文字を続けて`out`に出力します。最後に`fmt.Fprint`を使用して終わった後「Go！」を送信します。

## リファクタリング♪

いくつかのマジックナンバーを名前付き定数にリファクタリングする以外に、リファクタリングすることは多くありません。

```go
const finalWord = "Go!"
const countdownStart = 3

func Countdown(out io.Writer) {
    for i := countdownStart; i > 0; i-- {
        fmt.Fprintln(out, i)
    }
    fmt.Fprint(out, finalWord)
}
```

ここでプログラムを実行すると、目的の出力が得られるはずですが、1秒の一時停止による劇的なカウントダウンとしてはありません。

Goでは、`time.Sleep`でこれを実現できます。コードに追加してみてください。

```go
func Countdown(out io.Writer) {
    for i := countdownStart; i > 0; i-- {
        time.Sleep(1 * time.Second)
        fmt.Fprintln(out, i)
    }

    time.Sleep(1 * time.Second)
    fmt.Fprint(out, finalWord)
}
```

プログラムを実行すると、期待どおりに機能します。

## モック

テストは引き続き成功し、ソフトウェアは意図したとおりに機能しますが、いくつかの問題があります。

* テストの実行には4秒かかります。
  * ソフトウェア開発に関する先進的な記事はすべて、迅速なフィードバックループの重要性を強調しています。
  * **遅いテストは開発者の生産性を台無しにします**。
  * 要件がさらに洗練され、より多くのテストが必要になると想像してください。 `Countdown`のすべての新しいテストのテスト実行に4が追加されたことに満足していますか？
* 関数の重要なプロパティはテストしていません。

`Sleep`への依存性を抽出し、テストでそれを制御できるようにする必要があります。

`time.Sleep`をモックできる場合は、「実際の」`time.Sleep`の代わりに**DI** _dependency injection_ を使用して、**呼び出しをスパイ**してアサーションを作成できます。

## 最初にテストを書く

依存関係をインターフェースとして定義しましょう。これにより、`main`で「実際の」Sleeperを使用し、テストで _spy sleeper_ を使用できるようになります。インターフェースを使用することにより、`Countdown`関数はこれを意識せず、呼び出し側に柔軟性を追加します。

```go
type Sleeper interface {
    Sleep()
}
```

私の`Countdown`関数はスリープ時間の長さに責任を負わないと設計上の決定をしました。
これにより、少なくとも今のところコードが少し単純化され、関数のユーザーがそのスリープ時間を好きなように設定できるようになります。

次に、テストで使用するために**モック**を作成する必要があります。

```go
type SpySleeper struct {
    Calls int
}

func (s *SpySleeper) Sleep() {
    s.Calls++
}
```

_スパイ_ はモックの一種で、依存関係がどのように使用されるかを記録することができます。送られてきた引数や、何回呼び出されたのかなどを記録することができます。この例では、`Sleep()`が何回呼ばれたかを記録して、テストでそれを確認できるようにしています。

テストを更新してSpyへの依存関係を注入し、Sleepが4回呼び出されたことをアサートします。

```go
func TestCountdown(t *testing.T) {
    buffer := &bytes.Buffer{}
    spySleeper := &SpySleeper{}

    Countdown(buffer, spySleeper)

    got := buffer.String()
    want := `3
2
1
Go!`

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }

    if spySleeper.Calls != 4 {
        t.Errorf("not enough calls to sleeper, want 4 got %d", spySleeper.Calls)
    }
}
```

## テストを試して実行する

```text
too many arguments in call to Countdown
    have (*bytes.Buffer, *SpySleeper)
    want (io.Writer)
```

## テストを実行するための最小限のコードを記述し、失敗したテスト出力を確認します

`Sleeper`を受け入れるには`Countdown`を更新する必要があります

```go
func Countdown(out io.Writer, sleeper Sleeper) {
    for i := countdownStart; i > 0; i-- {
        time.Sleep(1 * time.Second)
        fmt.Fprintln(out, i)
    }

    time.Sleep(1 * time.Second)
    fmt.Fprint(out, finalWord)
}
```

もう一度試すと、同じ理由で`main`はコンパイルできなくなります

```text
./main.go:26:11: not enough arguments in call to Countdown
    have (*os.File)
    want (io.Writer, Sleeper)
```

必要なインターフェースを実装する「実際の」スリーパーを作成しましょう

```go
type DefaultSleeper struct {}

func (d *DefaultSleeper) Sleep() {
    time.Sleep(1 * time.Second)
}
```

それを実際のアプリケーションで使用することができます

```go
func main() {
    sleeper := &DefaultSleeper{}
    Countdown(os.Stdout, sleeper)
}
```

## 成功させるのに十分なコードを書く

テストはコンパイルされていますが、依存関係が挿入されているのではなく、`time.Sleep`を呼び出しているため、パスしていません。修正しましょう。

```go
func Countdown(out io.Writer, sleeper Sleeper) {
    for i := countdownStart; i > 0; i-- {
        sleeper.Sleep()
        fmt.Fprintln(out, i)
    }

    sleeper.Sleep()
    fmt.Fprint(out, finalWord)
}
```

テストに合格し、4秒はかかりません。

### まだいくつかの問題

テストしていない重要なプロパティがまだあります。

`Countdown` は各表示の前にスリープする必要があります。

例：

* `Sleep`
* `Print N`
* `Sleep`
* `Print N-1`
* `Sleep`
* `Print Go!`
* etc

私たちの最新の変更は、それが4回寝たとアサートしているだけですが、これらのスリープは順番に発生しない可能性があります。

テストを書くとき、もしそのテストが十分な信頼性を与えていないと確信したら、それを壊せばいいのです! (ただし、変更をソース管理にコミットしていることを確認してください)。

コードを次のように変更してください。


```go
func Countdown(out io.Writer, sleeper Sleeper) {
    for i := countdownStart; i > 0; i-- {
        sleeper.Sleep()
    }

    for i := countdownStart; i > 0; i-- {
        fmt.Fprintln(out, i)
    }

    sleeper.Sleep()
    fmt.Fprint(out, finalWord)
}
```

テストを実行すると、実装が間違っていてもテストは成功するはずです。

新しいテストで再度スパイを使用して、操作の順序が正しいことを確認してみましょう。

2つの異なる依存関係があり、それらのすべての操作を1つのリストに記録したいと考えています。
ですから、両方のスパイを1つ作成します。

```go
type SpyCountdownOperations struct {
    Calls []string
}

func (s *SpyCountdownOperations) Sleep() {
    s.Calls = append(s.Calls, sleep)
}

func (s *SpyCountdownOperations) Write(p []byte) (n int, err error) {
    s.Calls = append(s.Calls, write)
    return
}

const write = "write"
const sleep = "sleep"
```

`SpyCountdownOperations`は`io.Writer`と`Sleeper`の両方を実装し、すべての呼び出しを1つのスライスに記録します。このテストでは操作の順序のみを考慮しているため、名前付き操作のリストとして記録するだけで十分です。

テストスイートにサブテストを追加して、スリープと表示が希望する順序で動作することを確認します。

```go
t.Run("sleep before every print", func(t *testing.T) {
    spySleepPrinter := &SpyCountdownOperations{}
    Countdown(spySleepPrinter, spySleepPrinter)

    want := []string{
        sleep,
        write,
        sleep,
        write,
        sleep,
        write,
        sleep,
        write,
    }

    if !reflect.DeepEqual(want, spySleepPrinter.Calls) {
        t.Errorf("wanted calls %v got %v", want, spySleepPrinter.Calls)
    }
})
```

このテストは失敗するはずです。`Countdown`をテストの修正前の状態に戻します。

`Sleeper`をスパイする2つのテストができたので、テストをリファクタリングして、1つは表示されているものをテストし、もう1つは表示の合間に確実にスリープするようにします。
最後に、最初のスパイは使用されなくなったので削除できます。

```go
func TestCountdown(t *testing.T) {

    t.Run("prints 3 to Go!", func(t *testing.T) {
        buffer := &bytes.Buffer{}
        Countdown(buffer, &SpyCountdownOperations{})

        got := buffer.String()
        want := `3
2
1
Go!`

        if got != want {
            t.Errorf("got %q want %q", got, want)
        }
    })

    t.Run("sleep before every print", func(t *testing.T) {
        spySleepPrinter := &SpyCountdownOperations{}
        Countdown(spySleepPrinter, spySleepPrinter)

        want := []string{
            sleep,
            write,
            sleep,
            write,
            sleep,
            write,
            sleep,
            write,
        }

        if !reflect.DeepEqual(want, spySleepPrinter.Calls) {
            t.Errorf("wanted calls %v got %v", want, spySleepPrinter.Calls)
        }
    })
}
```

これで、機能とその2つの重要なプロパティが適切にテストされました。

## Sleeperを拡張して設定できるようにする

`Sleeper`が構成可能であることは素晴らしい機能です。これは、メインプログラムでスリープ時間を調整できることを意味します。
### 最初にテストを書く

まず、設定とテストに必要なものを受け入れる新しいタイプ`ConfigurableSleeper`を作成しましょう。

```go
type ConfigurableSleeper struct {
    duration time.Duration
    sleep    func(time.Duration)
}
```

スリープ時間を設定するために`duration`を使用し、スリープ関数を渡す方法として`sleep`を使用しています。 `sleep`のシグネチャは`time.Sleep`のシグネチャと同じであり、実際の実装で`time.Sleep`を使用して、テストで次のスパイを使用できます。

```go
type SpyTime struct {
    durationSlept time.Duration
}

func (s *SpyTime) Sleep(duration time.Duration) {
    s.durationSlept = duration
}
```

スパイを配置したら、構成可能なスリーパーの新しいテストを作成できます。

```go
func TestConfigurableSleeper(t *testing.T) {
    sleepTime := 5 * time.Second

    spyTime := &SpyTime{}
    sleeper := ConfigurableSleeper{sleepTime, spyTime.Sleep}
    sleeper.Sleep()

    if spyTime.durationSlept != sleepTime {
        t.Errorf("should have slept for %v but slept for %v", sleepTime, spyTime.durationSlept)
    }
}
```

このテストには何も新しいものはなく、以前の模擬テストと非常によく似た設定になっています。

### テストを試して実行する

```text
sleeper.Sleep undefined (type ConfigurableSleeper has no field or method Sleep, but does have sleep)
```

`ConfigurableSleeper`で作成された`Sleep`メソッドがないことを示す非常に明確なエラーメッセージが表示されます。

### テストを実行し、失敗したテスト出力を確認するための最小限のコードを記述します

```go
func (c *ConfigurableSleeper) Sleep() {
}
```

新しい`Sleep`関数が実装されたため、テストに失敗しました。

```text
countdown_test.go:56: should have slept for 5s but slept for 0s
```

### 成功させるのに十分なコードを書く

ここで行う必要があるのは、`ConfigurableSleeper`の`Sleep`関数を実装することだけです。

```go
func (c *ConfigurableSleeper) Sleep() {
    c.sleep(c.duration)
}
```

この変更により、すべてのテストが再び成功し、メインプログラムがまったく変更されなかったので、なぜすべての面倒な作業に不思議に思うかもしれません。
うまくいけば、次のセクションで明らかになります。

### クリーンアップとリファクタリング

最後に行う必要があるのは、メイン関数で実際に `ConfigurableSleeper`を使用することです。

```go
func main() {
    sleeper := &ConfigurableSleeper{1 * time.Second, time.Sleep}
    Countdown(os.Stdout, sleeper)
}
```

テストとプログラムを手動で実行すると、すべての動作が同じであることがわかります。

`ConfigurableSleeper`を使用しているので、`DefaultSleeper`実装を削除しても安全です。
私たちのプログラムをまとめ、より[一般化された](https://stackoverflow.com/questions/19291776/whats-the-difference-between-abstraction-and-generalization)任意の長いカウントダウンを備えたSleeperのプログラムが完成しました。
## しかし、モックを行うのは悪ではないのか？

モックは悪だと聞いたことがあるかもしれません。ソフトウェア開発のあらゆる手法は、[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)でさえも悪く利用できます。

人々は通常、テストに耳を貸さず、リファクタリングの段階を尊重していないときに悪い状態に陥ります。

モックコードが複雑になったり、何かをテストするためにたくさんのものをモック化したりする必要がある場合は、その悪い予感に耳を傾け、コードについて考える必要があります。それは次のようなサインです。

* テスト対象の責務が多すぎる（モックにするには依存関係が多すぎる）
  * モジュールを分解して、より少ない処理で済むようにする
* 依存関係が細かすぎる
  * これらの依存関係のいくつかを1つの意味のあるモジュールに統合する方法を考えてください
* テストが実装の詳細を気にしすぎている
  * 実装ではなく期待される動作のテストを優先する

通常、モックが多すぎることは、コードにおける_悪い抽象化_ 示します。

**これはTDDの弱点と思われがちですが、実は強みでもあります。**。多くの場合、貧弱なテストコードは、設計が悪い結果であるか、より適切に設計されたコードをテストすることは簡単です。テストコードが貧弱であることは、設計が悪いことが原因であることが多いです。よく設計されたコードは、テストしやすいからです。
### でも、モックやテストはまだ私を苦しめます！

この状況に遭遇したことがありますか？

* リファクタリングを行いたい
* そのために、たくさんのテストを変更する
* TDDに疑問を持ち、Mediumに "モックは有害だと思います"と題した投稿をする

これは通常、実装の詳細をテストしすぎていることを示しています。
システムにとって実装が本当に重要でない限り、テストは有用な振る舞いをテストするようにしてください。

どのレベルでテストをすればいいのかを正確に知るのは難しいですが、私が心がけている思考プロセスとルールをいくつか紹介します

* **リファクタリングの定義は、コードは変わっても、動作は同じというものです**。理論的にには、リファクタリングを行うことに決めたら、テストを変更せずにコミットできるはずです。ですから、テストを書くときには次のことを自分に問いかけてみてください。
  * 私がテストしているのは、欲しい振る舞いなのか、それとも実装の詳細なのか？
  * もしこのコードをリファクタリングするとしたら、テストに多くの変更を加えなければならないだろうか？
* Go ではプライベート関数をテストすることができますが、プライベート関数はパブリックな振る舞いをサポートするための実装の詳細であるため、私はそれをあまりテストしません。パブリックな振る舞いをテストしてください。Sandi Metzは、プライベート関数は「安定性に欠ける」と表現しており、テストをそれに付き合わせたくはないでしょう。
* テストが**3つ以上のモックで動作している場合、それは危険信号**です。設計を再検討する時です。
* スパイは注意して使用してください。スパイを使用すると、作成中のアルゴリズムの内部を確認できます。これは非常に便利ですが、テストコードと実装の間の結合がより緊密になることを意味します。 **これらをスパイする場合は、これらの詳細について実際に気にしていることを確認してください**

#### モッキングフレームワークを使うのはダメですか？

モックは魔法を必要とせず、比較的シンプルです。フレームワークを使うと、モックが実際よりも複雑に見えてしまうことがあります。この章ではモックの自動化を使わないので、次のようなことメリットがあります。

- どのようにモックを行うかについてのよりよい理解
- インターフェースの実装の練習

共同プロジェクトでは、モックを自動生成することに価値があります。チームにおいて、モック生成ツールはテストダブル周りの一貫性を成文化します。これにより、テストダブルの記述に矛盾が生じ、それがテストの記述の矛盾につながることを防げます。
モック生成ツールは、インターフェイスに対するテストダブルを生成するものだけを使用するようにしましょう。テストの書き方を過度に指定したり、"魔法"を多用したりするようなツールは邪魔になります。

## まとめ

### TDDアプローチの詳細

* ささいな例に直面した場合は、問題を「薄いスライス」に分解してください。ウサギの穴に入り込み、「ビッグバン」アプローチをとらないように、できるだけ早くテストに裏打ちされた動作するソフトウェアを使用できるようにしてください。
* 動作するソフトウェアを入手したら、必要なソフトウェアにたどり着くまで**小さなステップで繰り返す**のが簡単です。

> 「反復開発を使用する場合？反復開発は、成功させたいプロジェクトでのみ使用する必要があります。」

マーティン・ファウラー（Martin Fowler）。

### モック

* **モックを使わないと、コードの重要な部分がテストされないことになります。**。私たちのケースでは、各表示の間にコードが一時停止することをテストすることはできませんが、他にも数え切れないほどの例があります。失敗する可能性のあるサービスを呼び出していますか？特定の状態でシステムをテストしたいですか？モックなしでこれらのシナリオをテストすることは非常に困難です。
* モックがないと、単純なビジネスルールをテストするためだけに、データベースや他のサードパーティの設定が必要になる場合があります。テストが遅くなり、**フィードバックループが遅くなる**可能性があります。
* 何かをテストするためにデータベースまたはWebサービスをスピンアップする必要があるため、そのようなサービスの信頼性が低いために、**壊れやすいテスト**を受ける可能性があります。

開発者がモックについて学ぶと、システムの _何をするか_ ではなく _どのように動作するか_ という観点から、システムのあらゆる側面を過剰にテストすることが非常に簡単になります。 **テストの価値**と、それらが将来のリファクタリングに与える影響について常に注意してください。

モックに関するこの投稿では、モックの一種である**スパイ**のみを取り上げました。 モックの「正しい」用語は「テストダブル」です。
 
 > [テストダブルは、テスト用に本番オブジェクトを置き換える方法の総称です。](https://martinfowler.com/bliki/TestDouble.html)。

テストダブルには、スタブ、スパイ、モックなど様々な種類があります。詳しくは[Martin Fowlerの投稿](https://martinfowler.com/bliki/TestDouble.html)をご覧ください。 
