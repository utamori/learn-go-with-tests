---
description: Intro to property based tests
---

# プロパティベースのテスト概要

[**この章のすべてのコードはここにあります**](https://github.com/andmorefine/learn-go-with-tests/tree/master/roman-numerals)

一部の企業は、インタビュープロセスの一環として、[ローマ数字のカタ](http://codingdojo.org/kata/RomanNumerals/)を実行するように求めます。この章では、TDDでこれに取り組む方法を示します。

[アラビア数字](https://en.wikipedia.org/wiki/Arabic_numerals)（数字0〜9）をローマ数字に変換する関数を記述します。

[ローマ数字](https://en.wikipedia.org/wiki/Roman_numerals)について聞いたことがない場合は、ローマ人が数字を書き留めた方法です。

あなたはシンボルを一緒に貼り付けることによってそれらを構築し、それらのシンボルは数字を表します

つまり`I`は「1」です。`III`は「3」です。

簡単に見えますが、いくつか興味深いルールがあります。
`V`は「5」を意味しますが、`IV`は「4」です（ `IIII`ではありません）。

`MCMLXXXIV`は「1984」です。それは複雑に見え、これを最初から理解するためのコードをどのように書くことができるか想像することは困難です。

この本で強調しているように、ソフトウェア開発者にとって重要なスキルは、「有用な」機能の「薄い垂直スライス」を特定して特定し、**繰り返す**ことです。 TDDワークフローは、反復的な開発を容易にするのに役立ちます。

したがって、「1984」ではなく、「1」から始めましょう。

## 最初にテストを書く

```go
func TestRomanNumerals(t *testing.T) {
    got := ConvertToRoman(1)
    want := "I"

    if got != want {
        t.Errorf("got %q, want %q", got, want)
    }
}
```

あなたがこの本でこれまでに持っているならば、これはうまくいけばあなたにとって非常に退屈で日常的な感じです。それは良いことです。

## テストを実行してみます

`./numeral_test.go:6:9: undefined: ConvertToRoman`

コンパイラーに道を案内する

## テストを実行するための最小限のコードを記述し、失敗したテスト出力を確認します

関数を作成しますが、まだテストに合格しないでください。常に、期待どおりにテストが失敗することを確認してください。

```go
func ConvertToRoman(arabic int) string {
    return ""
}
```

今すぐ実行されます。

```go
=== RUN   TestRomanNumerals
--- FAIL: TestRomanNumerals (0.00s)
    numeral_test.go:10: got '', want 'I'
FAIL
```

## 成功させるのに十分なコードを書く

```go
func ConvertToRoman(arabic int) string {
    return "I"
}
```

## リファクタリング

まだリファクタリングする必要はありません。

結果をハードコーディングするだけでは変だと感じますが、TDDではできるだけ長く「赤字」を避けたいと考えています。あまり達成していないように感じるかもしれませんが、APIを定義し、ルールの1つをキャプチャするテストを取得しました。「実際の」コードがかなりばかげていても。

ここで、その不安な気持ちを使って新しいテストを記述し、少しだけ馬鹿なコードを書くように強制します。

## 最初にテストを書く

サブテストを使用してテストを適切にグループ化できます

```go
func TestRomanNumerals(t *testing.T) {
    t.Run("1 gets converted to I", func(t *testing.T) {
        got := ConvertToRoman(1)
        want := "I"

        if got != want {
            t.Errorf("got %q, want %q", got, want)
        }
    })

    t.Run("2 gets converted to II", func(t *testing.T) {
        got := ConvertToRoman(2)
        want := "II"

        if got != want {
            t.Errorf("got %q, want %q", got, want)
        }
    })
}
```

## テストを実行してみます

```text
=== RUN   TestRomanNumerals/2_gets_converted_to_II
    --- FAIL: TestRomanNumerals/2_gets_converted_to_II (0.00s)
        numeral_test.go:20: got 'I', want 'II'
```

それほど驚きはありません

## 成功させるのに十分なコードを書く

```go
func ConvertToRoman(arabic int) string {
    if arabic == 2 {
        return "II"
    }
    return "I"
}
```

ええ、まだ問題に取り組んでいないようです。したがって、前進させるために、さらに多くのテストを作成する必要があります。

## リファクタリング

テストでいくつかの繰り返しがあります。 「与えられた入力X、Yを期待する」の問題のように感じる何かをテストしているときは、おそらくテーブルベースのテストを使用する必要があります。

```go
func TestRomanNumerals(t *testing.T) {
    cases := []struct {
        Description string
        Arabic      int
        Want        string
    }{
        {"1 gets converted to I", 1, "I"},
        {"2 gets converted to II", 2, "II"},
    }

    for _, test := range cases {
        t.Run(test.Description, func(t *testing.T) {
            got := ConvertToRoman(test.Arabic)
            if got != test.Want {
                t.Errorf("got %q, want %q", got, test.Want)
            }
        })
    }
}
```

これ以上テストのボイラプレートを書かなくても、簡単にケースを追加できるようになりました。

頑張って「3」に行きましょう

## 最初にテストを書く

以下をケースに追加してください

```go
{"3 gets converted to III", 3, "III"},
```

## テストを実行してみます

```text
=== RUN   TestRomanNumerals/3_gets_converted_to_III
    --- FAIL: TestRomanNumerals/3_gets_converted_to_III (0.00s)
        numeral_test.go:20: got 'I', want 'III'
```

## 成功させるのに十分なコードを書く

```go
func ConvertToRoman(arabic int) string {
    if arabic == 3 {
        return "III"
    }
    if arabic == 2 {
        return "II"
    }
    return "I"
}
```

## リファクタリング

わかりましたので、これらのifステートメントを楽しんでいないようになり、コードを十分に見てみると、`arabic`のサイズに基づいて`I`の文字列を構築していることがわかります。

より複雑な数値については、ある種の算術および文字列連結を行うことを "知っています"。

これらの考えを念頭に置いてリファクタリングを試してみましょう。
それは最終的なソリューションには適さないかもしれませんが、それは問題ありません。
私たちはいつでもコードを捨てて、私たちをガイドする必要のあるテストからやり直すことができます。

```go
func ConvertToRoman(arabic int) string {

    var result strings.Builder

    for i:=0; i<arabic; i++ {
        result.WriteString("I")
    }

    return result.String()
}
```

これまでに[`strings.Builder`](https://golang.org/pkg/strings/#Builder)を使用したことがない可能性があります

> Builderは、Writeメソッドを使用して文字列を効率的に構築するために使用されます。メモリのコピーを最小限に抑えます。

通常、実際のパフォーマンスの問題が発生するまで、このような最適化に悩まされることはありませんが、コードの量は文字列に追加する「手動」よりも大きくないため、より高速なアプローチを使用することもできます。

コードは私にはよく見え、ドメインを _私たちが今知っているように_ 記述しています。

### ローマ人もDRYに夢中になりました...

物事は今より複雑になり始めています。ローマ人はその知恵の中で、繰り返し登場する人物は読みにくく、数え難くなるだろうと考えていました。したがって、ローマ数字のルールでは、同じ文字を3回以上続けて繰り返すことはできません。

代わりに、次に高いシンボルを取り、その左側にシンボルを置くことで「減算」します。すべてのシンボルを減算器として使用できるわけではありません。I（1）、X（10）、C（100）のみ。

たとえば、ローマ数字の「5」は`V`です。 「4」を作成するには、 `IIII`ではなく、` IV`を実行します。

## 最初にテストを書く

```go
{"4 gets converted to IV (can't repeat more than 3 times)", 4, "IV"},
```

## テストを実行してみます

```text
=== RUN   TestRomanNumerals/4_gets_converted_to_IV_(cant_repeat_more_than_3_times)
    --- FAIL: TestRomanNumerals/4_gets_converted_to_IV_(cant_repeat_more_than_3_times) (0.00s)
        numeral_test.go:24: got 'IIII', want 'IV'
```

## 成功させるのに十分なコードを書く

```go
func ConvertToRoman(arabic int) string {

    if arabic == 4 {
        return "IV"
    }

    var result strings.Builder

    for i:=0; i<arabic; i++ {
        result.WriteString("I")
    }

    return result.String()
}
```

## リファクタリング

文字列の構築パターンを壊したことが「好き」ではないので、それを続けたいと思います。

```go
func ConvertToRoman(arabic int) string {

    var result strings.Builder

    for i := arabic; i > 0; i-- {
        if i == 4 {
            result.WriteString("IV")
            break
        }
        result.WriteString("I")
    }

    return result.String()
}
```

「4」を現在の考えに「合わせる」ために、アラビア数字からカウントダウンし、進行中に文字列に記号を追加します。これが長期的に機能するかどうかはわかりませんが、見てみましょう！

「５」を作成しましょう

## 最初にテストを書く

```go
{"5 gets converted to V", 5, "V"},
```

## テストを実行してみます

```text
=== RUN   TestRomanNumerals/5_gets_converted_to_V
    --- FAIL: TestRomanNumerals/5_gets_converted_to_V (0.00s)
        numeral_test.go:25: got 'IIV', want 'V'
```

## 成功させるのに十分なコードを書く

「4」で行ったアプローチをコピーするだけです

```go
func ConvertToRoman(arabic int) string {

    var result strings.Builder

    for i := arabic; i > 0; i-- {
        if i == 5 {
            result.WriteString("V")
            break
        }
        if i == 4 {
            result.WriteString("IV")
            break
        }
        result.WriteString("I")
    }

    return result.String()
}
```

## リファクタリング

このようなループでの繰り返しは、通常、呼び出されるのを待っている抽象化の兆候です。ループを短絡することは読みやすさのための効果的なツールであるかもしれませんが、それはまたあなたに何か他のものを伝えているかもしれません。

アラビア数字をループしていて、特定の記号にぶつかった場合は`break`と呼びますが、実際に行っているのは、`i`を手間をかけて減算することです。

```go
func ConvertToRoman(arabic int) string {

    var result strings.Builder

    for arabic > 0 {
        switch {
        case arabic > 4:
            result.WriteString("V")
            arabic -= 5
        case arabic > 3:
            result.WriteString("IV")
            arabic -= 4
        default:
            result.WriteString("I")
            arabic--
        }
    }

    return result.String()
}
```

* いくつかの非常に基本的なシナリオのテストから得られたコードから読み取っている信号を考えると、ローマ数字を作成するには、シンボルを適用するときに「アラビア語」から減算する必要があることがわかります
* `for`ループはもはや`i`に依存せず、代わりに、`arabic`から十分な数のシンボルを減算するまで文字列を構築し続けます。

このアプローチが「6」（VI）、「7」（VII）、「8」（VIII）にも有効であると確信しています。
それでも、ケースをテストスイートに追加し、（簡潔にするためにコードは含めません。不明な場合はgithubのサンプルを確認してください）を確認してください。

「9」は、次の数の表現から「I」を減算する必要があるという点で、「4」と同じルールに従います。 「10」はローマ数字で「X」で表されます。したがって、「9」は「IX」になります。

## 最初にテストを書く

```go
{"9 gets converted to IX", 9, "IX"}
```

## テストを実行してみます

```text
=== RUN   TestRomanNumerals/9_gets_converted_to_IX
    --- FAIL: TestRomanNumerals/9_gets_converted_to_IX (0.00s)
        numeral_test.go:29: got 'VIV', want 'IX'
```

## 成功させるのに十分なコードを書く

以前と同じアプローチを採用できるはずです

```go
case arabic > 8:
    result.WriteString("IX")
    arabic -= 9
```

## リファクタリング

それはコードがまだどこかにリファクタリングがあることを私たちに伝えているように感じますが、私には完全に明白ではないので、続けましょう。

これもコードはスキップしますが、テストケースに「10」のテストを追加します。「10」は「X」である必要があり、先に進む前に合格にします。

「39」までのコードが機能すると確信しているので、ここに追加したいくつかのテストがあります。

```go
{"10 gets converted to X", 10, "X"},
{"14 gets converted to XIV", 14, "XIV"},
{"18 gets converted to XVIII", 18, "XVIII"},
{"20 gets converted to XX", 20, "XX"},
{"39 gets converted to XXXIX", 39, "XXXIX"},
```

これまでにオブジェクト指向プログラミングを行ったことがある場合は、少し疑惑を抱いて`switch`ステートメントを表示する必要があることがわかります。通常、実際には代わりにクラス構造でキャプチャできる場合でも、一部の命令コード内でコンセプトまたはデータをキャプチャしています。

Goは厳密にはオブジェクト指向ではありませんが、オブジェクト指向が提供するレッスンを完全に無視することを意味するわけではありません（あなたが伝えたいだけのことです）。

私たちの切り替えステートメントは、動作とともにローマ数字に関するいくつかの真実を説明しています。

データを動作から分離することで、これをリファクタリングできます。

```go
type RomanNumeral struct {
    Value  int
    Symbol string
}

var allRomanNumerals = []RomanNumeral {
    {10, "X"},
    {9, "IX"},
    {5, "V"},
    {4, "IV"},
    {1, "I"},
}

func ConvertToRoman(arabic int) string {

    var result strings.Builder

    for _, numeral := range allRomanNumerals {
        for arabic >= numeral.Value {
            result.WriteString(numeral.Symbol)
            arabic -= numeral.Value
        }
    }

    return result.String()
}
```

これはずっと気分が良いです。数値に関連するいくつかのルールをアルゴリズムで非表示にするのではなくデータとして宣言しました。アラビア数字を処理して、適合する場合は結果に記号を追加する方法を確認できます。

この抽象化はより大きな数で機能しますか？ 「50」のローマ数字（`L`）で機能するようにテストスイートを拡張します。

ここにいくつかのテストケースがあります。

```go
{"40 gets converted to XL", 40, "XL"},
{"47 gets converted to XLVII", 47, "XLVII"},
{"49 gets converted to XLIX", 49, "XLIX"},
{"50 gets converted to L", 50, "L"},
```

助けが必要？

追加するシンボルは[この要点](https://gist.github.com/pamelafox/6c7b948213ba55332d86efd0f0b037de)で確認できます。

## そして残りの部分！

残りの記号は次のとおりです

| Arabic | Roman |
| :--- | :---: |
| 100 | C |
| 500 | D |
| 1000 | M |

残りのシンボルについても同じ方法を使用します。テストとシンボルの配列の両方にデータを追加するだけです。

あなたのコードは`1984`:`MCMLXXXIV`で動作しますか？

これが私の最後のテストスイートです

```go
func TestRomanNumerals(t *testing.T) {
    cases := []struct {
        Arabic int
        Roman  string
    }{
        {Arabic: 1, Roman: "I"},
        {Arabic: 2, Roman: "II"},
        {Arabic: 3, Roman: "III"},
        {Arabic: 4, Roman: "IV"},
        {Arabic: 5, Roman: "V"},
        {Arabic: 6, Roman: "VI"},
        {Arabic: 7, Roman: "VII"},
        {Arabic: 8, Roman: "VIII"},
        {Arabic: 9, Roman: "IX"},
        {Arabic: 10, Roman: "X"},
        {Arabic: 14, Roman: "XIV"},
        {Arabic: 18, Roman: "XVIII"},
        {Arabic: 20, Roman: "XX"},
        {Arabic: 39, Roman: "XXXIX"},
        {Arabic: 40, Roman: "XL"},
        {Arabic: 47, Roman: "XLVII"},
        {Arabic: 49, Roman: "XLIX"},
        {Arabic: 50, Roman: "L"},
        {Arabic: 100, Roman: "C"},
        {Arabic: 90, Roman: "XC"},
        {Arabic: 400, Roman: "CD"},
        {Arabic: 500, Roman: "D"},
        {Arabic: 900, Roman: "CM"},
        {Arabic: 1000, Roman: "M"},
        {Arabic: 1984, Roman: "MCMLXXXIV"},
        {Arabic: 3999, Roman: "MMMCMXCIX"},
        {Arabic: 2014, Roman: "MMXIV"},
        {Arabic: 1006, Roman: "MVI"},
        {Arabic: 798, Roman: "DCCXCVIII"},
    }
    for _, test := range cases {
        t.Run(fmt.Sprintf("%d gets converted to %q", test.Arabic, test.Roman), func(t *testing.T) {
            got := ConvertToRoman(test.Arabic)
            if got != test.Roman {
                t.Errorf("got %q, want %q", got, test.Roman)
            }
        })
    }
}
```

* データに十分な情報が記載されていると感じたため、「コメント」を削除しました。
* もう少し自信を与えるために見つけた他のエッジケースをいくつか追加しました。テーブルベースのテストでは、これは非常に安価です。

アルゴリズムは変更しませんでした。`allRomanNumerals`配列を更新するだけで済みました。

```go
var allRomanNumerals = []RomanNumeral{
    {1000, "M"},
    {900, "CM"},
    {500, "D"},
    {400, "CD"},
    {100, "C"},
    {90, "XC"},
    {50, "L"},
    {40, "XL"},
    {10, "X"},
    {9, "IX"},
    {5, "V"},
    {4, "IV"},
    {1, "I"},
}
```

## ローマ数字の解析

まだ終わっていません。次に、ローマ数字の _from_ を`int`に変換する関数を書きます

## 最初にテストを書く

テストケースを少しリファクタリングして再利用できます

`case`変数を`var`ブロックのパッケージ変数としてテストの外に移動します。

```go
func TestConvertingToArabic(t *testing.T) {
    for _, test := range cases[:1] {
        t.Run(fmt.Sprintf("%q gets converted to %d", test.Roman, test.Arabic), func(t *testing.T) {
            got := ConvertToArabic(test.Roman)
            if got != test.Arabic {
                t.Errorf("got %d, want %d", got, test.Arabic)
            }
        })
    }
}
```

スライス機能を使用して、今のところテストの1つだけを実行していることに注意してください。（`cases [:1]`）これらのテストを一度にすべて合格にしようとすると、飛躍的に大きくなります。

## テストを実行してみます

```text
./numeral_test.go:60:11: undefined: ConvertToArabic
```

## テストを実行するための最小限のコードを記述し、失敗したテスト出力を確認します

新しい関数定義を追加します

```go
func ConvertToArabic(roman string) int {
    return 0
}
```

テストが実行され、失敗するはずです

```text
--- FAIL: TestConvertingToArabic (0.00s)
    --- FAIL: TestConvertingToArabic/'I'_gets_converted_to_1 (0.00s)
        numeral_test.go:62: got 0, want 1
```

## 成功させるのに十分なコードを書く

あなたは何をするべきか知っています

```go
func ConvertToArabic(roman string) int {
    return 1
}
```

次に、テストのスライスインデックスを変更して、次のテストケースに移動します（例:`cases [:2]`）。
3つ目のケースについても、考えられる最もおかしなコードを理解し、（最高の本で間違いありませんか？）これが私のばかげたコードです。

```go
func ConvertToArabic(roman string) int {
    if roman == "III" {
        return 3
    }
    if roman == "II" {
        return 2
    }
    return 1
}
```

機能する実際のコードの愚かさを通して、以前のようなパターンを見ることができます。入力を反復処理して _something_ を構築する必要があります。この場合は合計です。

```go
func ConvertToArabic(roman string) int {
    total := 0
    for range roman {
        total++
    }
    return total
}
```

## 最初にテストを書く

次に、`cases[:4]`（`IV`）に移動します。これは、文字列の長さである2を返すため、失敗します。

## 成功させるのに十分なコードを書く

```go
// earlier..
type RomanNumerals []RomanNumeral

func (r RomanNumerals) ValueOf(symbol string) int {
    for _, s := range r {
        if s.Symbol == symbol {
            return s.Value
        }
    }

    return 0
}

// later..
func ConvertToArabic(roman string) int {
    total := 0

    for i := 0; i < len(roman); i++ {
        symbol := roman[i]

        // look ahead to next symbol if we can and, the current symbol is base 10 (only valid subtractors)
        if i+1 < len(roman) && symbol == 'I' {
            nextSymbol := roman[i+1]

            // build the two character string
            potentialNumber := string([]byte{symbol, nextSymbol})

            // get the value of the two character string
            value := allRomanNumerals.ValueOf(potentialNumber)

            if value != 0 {
                total += value
                i++ // move past this character too for the next loop
            } else {
                total++
            }
        } else {
            total++
        }
    }
    return total
}
```

これは恐ろしいことですが、機能します。コメントを追加する必要があると感じたのは残念です。

* 与えられたローマ数字の整数値を検索できるようにしたいので、`RomanNumeral`の配列から型を作成し、それにメソッド`ValueOf`を追加しました
* 次にループの中では、文字列が十分に大きくて、現在の記号が有効な減算器であるかどうか、先を見る必要があります。現時点では、 `I`（1）ですが、`X`（10）または `C`（100）にすることもできます。
  * これらの両方の条件を満たす場合、値を検索して合計に追加する必要があります _if_ 特別な減算器の1つです。それ以外の場合は無視します
  * 次に、このシンボルを2回カウントしないように、`i`をさらにインクリメントする必要があります。

## リファクタリング

私はこれが長期的なアプローチになると完全に確信しているわけではなく、私たちができる可能性のある興味深いリファクタリングが潜在的にあるかもしれませんが、私たちのアプローチが完全に間違っている場合に備えて私はそれに抵抗します。私はむしろいくつかのテストに最初に合格して見てもらいたいです。その間、私は最初の`if`ステートメントを少し恐ろしくなくしました。

```go
func ConvertToArabic(roman string) int {
    total := 0

    for i := 0; i < len(roman); i++ {
        symbol := roman[i]

        if couldBeSubtractive(i, symbol, roman) {
            nextSymbol := roman[i+1]

            // build the two character string
            potentialNumber := string([]byte{symbol, nextSymbol})

            // get the value of the two character string
            value := allRomanNumerals.ValueOf(potentialNumber)

            if value != 0 {
                total += value
                i++ // move past this character too for the next loop
            } else {
                total++
            }
        } else {
            total++
        }
    }
    return total
}

func couldBeSubtractive(index int, currentSymbol uint8, roman string) bool {
    return index+1 < len(roman) && currentSymbol == 'I'
}
```

## 最初にテストを書く

`cases[:5]`に移りましょう

```text
=== RUN   TestConvertingToArabic/'V'_gets_converted_to_5
    --- FAIL: TestConvertingToArabic/'V'_gets_converted_to_5 (0.00s)
        numeral_test.go:62: got 1, want 5
```

## 成功させるのに十分なコードを書く

それが減算である場合を除いて、コードはすべての文字が`I`であると想定しているため、値は「1」です。これを修正するには、`ValueOf`メソッドを再利用できるはずです。

```go
func ConvertToArabic(roman string) int {
    total := 0

    for i := 0; i < len(roman); i++ {
        symbol := roman[i]

        // look ahead to next symbol if we can and, the current symbol is base 10 (only valid subtractors)
        if couldBeSubtractive(i, symbol, roman) {
            nextSymbol := roman[i+1]

            // build the two character string
            potentialNumber := string([]byte{symbol, nextSymbol})

            if value := allRomanNumerals.ValueOf(potentialNumber); value != 0 {
                total += value
                i++ // move past this character too for the next loop
            } else {
                total++ // this is fishy...
            }
        } else {
            total+=allRomanNumerals.ValueOf(string([]byte{symbol}))
        }
    }
    return total
}
```

## リファクタリング

Goで文字列にインデックスを付けると、`byte`を取得します。これが、文字列を再度構築するときに、`string([]byte{symbol})`のようなことをしなければならない理由です。
数回繰り返されますが、その機能を移動して、`ValueOf`が代わりに数バイトを取るようにします。

```go
func (r RomanNumerals) ValueOf(symbols ...byte) int {
    symbol := string(symbols)
    for _, s := range r {
        if s.Symbol == symbol {
            return s.Value
        }
    }

    return 0
}
```

次に、バイトをそのまま関数に渡すことができます

```go
func ConvertToArabic(roman string) int {
    total := 0

    for i := 0; i < len(roman); i++ {
        symbol := roman[i]

        if couldBeSubtractive(i, symbol, roman) {
            if value := allRomanNumerals.ValueOf(symbol, roman[i+1]); value != 0 {
                total += value
                i++ // move past this character too for the next loop
            } else {
                total++ // this is fishy...
            }
        } else {
            total+=allRomanNumerals.ValueOf(symbol)
        }
    }
    return total
}
```

それはまだかなり厄介ですが、そこに到達しています。

`cases[:xx]`の数値を移動し始めると、かなりの数が現在通過していることがわかります。スライスオペレーターを完全に削除して、失敗するオペレーターを確認します。これが私のスイートの例です。

```text
=== RUN   TestConvertingToArabic/'XL'_gets_converted_to_40
    --- FAIL: TestConvertingToArabic/'XL'_gets_converted_to_40 (0.00s)
        numeral_test.go:62: got 60, want 40
=== RUN   TestConvertingToArabic/'XLVII'_gets_converted_to_47
    --- FAIL: TestConvertingToArabic/'XLVII'_gets_converted_to_47 (0.00s)
        numeral_test.go:62: got 67, want 47
=== RUN   TestConvertingToArabic/'XLIX'_gets_converted_to_49
    --- FAIL: TestConvertingToArabic/'XLIX'_gets_converted_to_49 (0.00s)
        numeral_test.go:62: got 69, want 49
```

私たちが見逃しているのは、`couldBeSubtractive`の更新だけなので、他の種類の減算記号を考慮に入れていると思います

```go
func couldBeSubtractive(index int, currentSymbol uint8, roman string) bool {
    isSubtractiveSymbol := currentSymbol == 'I' || currentSymbol == 'X' || currentSymbol =='C'
    return index+1 < len(roman) && isSubtractiveSymbol
}
```

もう一度試してください。まだ失敗します。しかし、私たちは以前にコメントを残しました...

```go
total++ // this is fishy...
```

すべての記号が`I`であることを意味するので、`total`をインクリメントするだけではいけません。それを次のものに置き換えます。

```go
total += allRomanNumerals.ValueOf(symbol)
```

そして、すべてのテストに合格しました！

これで完全に機能するソフトウェアができたので、自信を持ってリファクタリングを行うことができます。

## リファクタリング

ここに私が仕上げたすべてのコードがあります。私はいくつかの失敗した試みをしましたが、強調し続けているように、それは問題ありません、そしてテストは私が自由にコードをいじるのを助けます。

```go
import "strings"

func ConvertToArabic(roman string) (total int) {
    for _, symbols := range windowedRoman(roman).Symbols() {
        total += allRomanNumerals.ValueOf(symbols...)
    }
    return
}

func ConvertToRoman(arabic int) string {
    var result strings.Builder

    for _, numeral := range allRomanNumerals {
        for arabic >= numeral.Value {
            result.WriteString(numeral.Symbol)
            arabic -= numeral.Value
        }
    }

    return result.String()
}

type romanNumeral struct {
    Value  int
    Symbol string
}

type romanNumerals []romanNumeral

func (r romanNumerals) ValueOf(symbols ...byte) int {
    symbol := string(symbols)
    for _, s := range r {
        if s.Symbol == symbol {
            return s.Value
        }
    }

    return 0
}

func (r romanNumerals) Exists(symbols ...byte) bool {
    symbol := string(symbols)
    for _, s := range r {
        if s.Symbol == symbol {
            return true
        }
    }
    return false
}

var allRomanNumerals = romanNumerals{
    {1000, "M"},
    {900, "CM"},
    {500, "D"},
    {400, "CD"},
    {100, "C"},
    {90, "XC"},
    {50, "L"},
    {40, "XL"},
    {10, "X"},
    {9, "IX"},
    {5, "V"},
    {4, "IV"},
    {1, "I"},
}

type windowedRoman string

func (w windowedRoman) Symbols() (symbols [][]byte) {
    for i := 0; i < len(w); i++ {
        symbol := w[i]
        notAtEnd := i+1 < len(w)

        if notAtEnd && isSubtractive(symbol) && allRomanNumerals.Exists(symbol, w[i+1]) {
            symbols = append(symbols, []byte{byte(symbol), byte(w[i+1])})
            i++
        } else {
            symbols = append(symbols, []byte{byte(symbol)})
        }
    }
    return
}

func isSubtractive(symbol uint8) bool {
    return symbol == 'I' || symbol == 'X' || symbol == 'C'
}
```

以前のコードの私の主な問題は、以前のリファクタリングに似ています。一緒に結合された懸念が多すぎました。文字列からローマ数字を抽出し、それらの値を検索するアルゴリズムを作成しました。

そこで、数値の抽出を処理する新しいタイプの`windowedRoman`を作成し、それらをスライスとして取得するための`Symbols`メソッドを提供しました。これは、`ConvertToArabic`関数が単にシンボルを反復処理してそれらを合計できることを意味しました。

いくつかの関数を抽出することでコードを少し壊しました。特に、現在処理している記号が2文字の減算記号であるかどうかを判断するために、不安定なifステートメントを中心にしています。

おそらくもっとエレガントな方法があるでしょうが、私はそれを気にするつもりはありません。コードはそこにあり、動作し、テストされています。私（または他の誰か）が安全に変更できるより良い方法を見つけた場合-大変な作業は完了です。

## プロパティベースのテストの概要

この章で使用したローマ数字のドメインにはいくつかのルールがあります

* 3つ以上の連続したシンボルは使用できません
* `I` （1）、 `X` （10） 、 `C` （100） のみが「減算器」になります
* `ConvertToRoman(N)`の結果を取得して`ConvertToArabic`に渡すと、`N`が返されます

これまでに作成したテストは、「サンプル」ベースのテストとして説明できます。このテストでは、コード周辺のいくつかの例を検証するためのツールを提供します。

ドメインについて知っているこれらのルールを採用し、コードに対して何らかの方法でそれらを実行できるとしたらどうでしょうか。

プロパティベースのテストは、コードにランダムデータを投げ、記述したルールが常に正しいことを確認することで、これを行うのに役立ちます。多くの人は、プロパティベースのテストは主にランダムデータについてであると考えていますが、それは間違いです。プロパティベースのテストに関する本当の課題は、ドメインをよく理解して、これらのプロパティを記述できるようにすることです。

十分な言葉、いくつかのコードを見てみましょう

```go
func TestPropertiesOfConversion(t *testing.T) {
    assertion := func(arabic int) bool {
        roman := ConvertToRoman(arabic)
        fromRoman := ConvertToArabic(roman)
        return fromRoman == arabic
    }

    if err := quick.Check(assertion, nil); err != nil {
        t.Error("failed checks", err)
    }
}
```

### 財産の根拠

最初のテストでは、数値をローマ数字に変換する場合に、他の関数を使用して、最初に取得した数値に変換するときにチェックします。

* 与えられた乱数（例えば`4`）。
* 乱数で`ConvertToRoman`を呼び出します（`4`の場合は`IV`を返す必要があります）。
* 上記の結果を受け取り、それを`ConvertToArabic`に渡します。
* 上記により、元の入力（`4`）が得られます。

これは、どちらかにバグがあると壊れるので、自信をつける良いテストのように感じます。合格できる唯一の方法は、彼らに同じ種類のバグがあるかどうかです。これは不可能ではありませんが、ありそうにありません。

### 技術的な説明

標準ライブラリの[testing/quick](https://golang.org/pkg/testing/quick/)パッケージを使用しています

下から読んで、`quick.Check`関数をいくつかのランダムな入力に対して実行する関数を提供します。関数が`false`を返す場合、チェックに失敗したと見なされます。

上記の`assertion`関数は乱数を受け取り、関数を実行してプロパティをテストします。

### テストを実行する

実行してみてください。

お使いのコンピュータがしばらくハングアップする可能性があるので、退屈したらそれを殺してください。

どうしたの？
以下をアサーションコードに追加してみてください。

```go
assertion := func(arabic int) bool {
    if arabic <0 || arabic > 3999 {
        log.Println(arabic)
        return true
    }
    roman := ConvertToRoman(arabic)
    fromRoman := ConvertToArabic(roman)
    return fromRoman == arabic
}
```

次のようなものが表示されます。

```text
=== RUN   TestPropertiesOfConversion
2019/07/09 14:41:27 6849766357708982977
2019/07/09 14:41:27 -7028152357875163913
2019/07/09 14:41:27 -6752532134903680693
2019/07/09 14:41:27 4051793897228170080
2019/07/09 14:41:27 -1111868396280600429
2019/07/09 14:41:27 8851967058300421387
2019/07/09 14:41:27 562755830018219185
```

この非常に単純なプロパティを実行するだけで、実装に欠陥が明らかになりました。入力として`int`を使用しましたが、

* ローマ数字では負の数を実行できません
* 最大3つの連続する記号という規則を考慮すると、「3999」を超える値を表すことはできません（[まあ、ちょっと](https://www.quora.com/Which-is-the-maximum-number-in -Roman-numerals)）および `int`の最大値は「3999」よりはるかに大きくなっています。

これは素晴らしい！
プロパティベースのテストの真の強みであるドメインについて、より深く考えることを余儀なくされました。

明らかに`int`は素晴らしいタイプではありません。もう少し適切なものを試した場合はどうなりますか？

### [`uint16`](https://golang.org/pkg/builtin/#uint16)

Goには _unsigned integers_ の型があります。つまり、負の値にはできません。
これにより、コード内の1つのクラスのバグがすぐに除外されます。
16を追加することは、最大「65535」を格納できる16ビット整数であることを意味します。
これはまだ大きすぎますが、必要なものに近づきます。

コードを更新して`int`ではなく`uint16`を使うようにしてみてください。テストの`assertion`を更新して、もう少し見やすくしました。

```go
assertion := func(arabic uint16) bool {
    if arabic > 3999 {
        return true
    }
    t.Log("testing", arabic)
    roman := ConvertToRoman(arabic)
    fromRoman := ConvertToArabic(roman)
    return fromRoman == arabic
}
```

テストを実行すると、実際に実行され、何がテストされているかを見ることができます。複数回実行して、私たちのコードが様々な値にうまく対応していることを確認することができます！これで、私たちのコードが思い通りに動作していることを確信することができます。これは、私たちのコードが私たちの望むように動作していることに大きな自信を与えてくれます。

デフォルトでは`quick.Check`の実行回数は100回ですが、設定で変更することができます。

```go
if err := quick.Check(assertion, &quick.Config{
    MaxCount:1000,
}); err != nil {
    t.Error("failed checks", err)
}
```

### さらなる作業

* 私たちが説明した他のプロパティをチェックするプロパティテストを書くことができますか？
* 誰かが「3999」以上の番号で私たちのコードを呼び出すことができないようにする方法を考えられますか？
  * エラーを返すことができます。
  * または、「3999」を表現できない新しい型を作成します。
    * あなたは何が一番いいと思いますか？

## まとめ

### 反復開発でTDDの練習を増やす

「1984」を`MCMLXXXIV`に変換するコードを書くことは、最初は怖く感じましたか？私は長い間ソフトウェアを書いてきました。

いつものことですが、コツは、**簡単なことから始めて** **小さなステップ**を踏むことです。

このプロセスでは、大きな飛躍をしたり、大きなリファクタリングをしたり、混乱に陥ったりすることはありませんでした。

誰かが「これはただの型だ」と皮肉を言っているのが聞こえてきます。これに異論はありませんが、私は今でも自分が取り組むすべてのプロジェクトで同じアプローチをとっています。私は最初のステップで大きな分散システムを出荷することはありません。チームが出荷できる最も単純なものを見つけて、「Hello world」というウェブサイトを出荷し、その後、管理可能な小さな断片の機能を反復していくのです。

スキルは作業を分割する方法を知っていることです。

### プロパティベースのテスト

* 標準ライブラリに組み込まれている
* ドメインルールをコードで記述する方法を考えることができれば、自信をつけるための優れたツールになります。
* ドメインについて深く考えさせられる
* テストスイートを補完するものになる可能性があります。
