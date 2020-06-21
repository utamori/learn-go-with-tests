---
description: Install Go
---

# Goをインストールする

Goの公式インストール手順は、[こちら](https://golang.org/doc/install)から入手できます。

このガイドでは、パッケージマネージャーを使用していることを前提としています。 [Homebrew](https://brew.sh), [Chocolatey](https://chocolatey.org), [Apt](https://help.ubuntu.com/community/AptGet/Howto) or [yum](https://access.redhat.com/solutions/9934).

デモのために、Homebrewを使用したOSXのインストール手順を紹介します。

## インストール

インストールの手順はとても簡単です。

まず、やるべきことは、このコマンドを実行してhomebrewをインストールすることです。これはXcodeに依存しているので、最初にこれがインストールされていることを確認してください。

```bash
xcode-select --install
```

そして、以下を実行してhomebrewをインストールします。

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

この時点でGoをインストールすることができます。

```bash
brew install go
```

※ パッケージマネージャが推奨する指示に従ってください。**注意** これらはホストOS固有のものかもしれません。

インストールを確認することができます。

```bash
$ go version
go version go1.14 darwin/amd64
```

## Go 環境

### $GOPATH

Goは意見があります。

慣例では、すべてのGoコードは1つのワークスペース (フォルダ) に格納されています。

このワークスペースはマシンのどこにでもあります。指定しない場合、Go はデフォルトのワークスペースとして`$HOME/go`を想定します。ワークスペースは、環境変数[GOPATH](https://golang.org/cmd/go/#hdr-GOPATH_environment_variable)によって識別され(変更され)ます。

環境変数を設定しておくと、後でスクリプトやシェルなどで使えるようになります。

`.bash_profile`を以下のエクスポートを含むように更新してください。

```bash
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

*注意* これらの環境変数を拾うためには、新しいシェルを開く必要があります。

Goは、ワークスペースに特定のディレクトリ構造が含まれていることを前提としています。

Goはファイルを3つのディレクトリに配置します。すべてのソースコードは`src`に、パッケージオブジェクトは`pkg`に、コンパイルされたプログラムは`bin`に置かれます。

これらのディレクトリは、次のように作成できます。

```bash
mkdir -p $GOPATH/src $GOPATH/pkg $GOPATH/bin
```

この時点で`go get`ができ、`src/package/bin`が適切な`$GOPATH/xxx`ディレクトリに正しくインストールされます。

### Go モジュール

Go1.11では [モジュール](https://github.com/golang/go/wiki/Modules)が導入され、代替のワークフローが可能になりました。
この新しいアプローチは徐々に [デフォルト](https://blog.golang.org/modules2019)モードになり、`GOPATH`の使用は廃止されます。

モジュールは、依存関係管理、バージョン選択、再現性のあるビルドに関連する問題を解決することを目的としており、ユーザは `GOPATH` の外で Go コードを実行することもできます。

モジュールの使用方法は非常に簡単です。`GOPATH`外の任意のディレクトリをプロジェクトのルートとして選択し、`go mod init`コマンドで新しいモジュールを作成します。

ファイル `go.mod`が生成され、モジュールのパス、Go のバージョン、依存関係の要件が含まれています。

`<modulepath>`が指定されていない場合、`go mod init`はディレクトリ構造からモジュールのパスを推測しようとしますが、引数を与えることでそれを上書きすることもできます。

```bash
mkdir my-project
cd my-project
go mod init <modulepath>
```

`go.mod`ファイルは次のようになります。

```
module cmd

go 1.14

require (
        github.com/google/pprof v0.0.0-20190515194954-54271f7e092f
        golang.org/x/arch v0.0.0-20190815191158-8a70ba74b3a1
        golang.org/x/tools v0.0.0-20190611154301-25a4f137592f
)
```

組み込みのドキュメントには、利用可能なすべての`go mod`コマンドの概要が記載されています。

```bash
go help mod
go help mod init
```

## Go エディター

エディタの好みは非常に個人的なものなので、Goをサポートしているエディタをすでにお持ちかもしれません。もしそうでない場合は、[Visual Studio Code](https://code.visualstudio.com)のような、例外的にGoをサポートしているエディタを検討する必要があります。

以下のコマンドでインストールできます。

```bash
brew cask install visual-studio-code
```

あなたは、あなたのシェルで次のように実行することができます`VS Code`が正しくインストールされていることを確認することができます。

```bash
code .
```

VS Code はほとんどのソフトウェアが有効化された状態で出荷されています。Goのサポートを追加するには、拡張機能をインストールする必要があります。VS Codeには様々な種類のものがありますが、例外的なものとして[Luke Hoban's package](https://github.com/golang/vscode-go)があります。これは以下のようにインストールすることができます。

```bash
code --install-extension ms-vscode.go
```

VS Codeで初めてGOファイルを開くと、解析ツールがないことが表示されるので、ボタンをクリックしてインストールする必要があります。VS Codeでインストールされる(使用される)ツールのリストは[こちら](https://github.com/golang/vscode-go/blob/master/docs/tools.md)を参照してください。

## Go デバッガー

Goのデバッグ（VS Codeと統合されている）に良いオプションは`Delve`です。
これは以下のようにインストールすることができます。

```bash
go get -u github.com/go-delve/delve/cmd/dlv
```

VSCodeでの Goデバッガーの設定と実行に関する追加のヘルプは、[VS Code デバッギング・ドキュメント](https://github.com/golang/vscode-go/blob/master/docs/debugging.md) を参照してください。

## Go リンター

デフォルトのリンターよりも改良されたものは、[GolangCI-Lint](https://golangci-lint.run)を使用して設定することができます。

以下のようにインストールすることができます。

```bash
brew install golangci/tap/golangci-lint
```

## リファクタリングとあなたのツーリング

このサイトでは、リファクタリングの重要性を重視しています。

あなたのツールは、より大きなリファクタリングを自信を持って行うのに役立ちます。

エディタを使いこなせば、簡単なキーの組み合わせで以下のことができるようになるはずです。

* **変数を抽出する/インライン変数**。魔法の値を取得して名前を付けることができるので、コードを素早くシンプルにすることができます。
* **メソッド/関数を抽出する**。コードの一部を取り出して、関数/メソッドを抽出できることが重要です。
* **リネーム**。ファイル間でシンボルの名前を自信を持って変更できるようになるはずです。
* **Go fmt**. Goには、`go fmt`と呼ばれるオピニオン付きフォーマッタがあります。エディタはファイルを保存するたびにこれを実行しなければなりません。
* **テストの実行**。上記のいずれかを実行して、リファクタリングが何も壊れていないことを確認するために、すぐにテストを再実行できるようにすべきであることは言うまでもありません。

さらに、あなたのコードで作業するのを助けるために、あなたができるようにする必要があります。

* **関数のシグネチャを表示** - Goの関数の呼び出し方がわからなくなってはいけません。IDEは、関数のドキュメント、パラメータ、戻り値を記述しなければなりません。
* **関数の定義を表示** - 関数が何をするのかわからない場合は、ソースコードを参照して自分で考えてみてください。
* **シンボルの使用法を見る** - 呼び出された関数のコンテキストを見ることができると、リファクタリングを行う際の意思決定プロセスに役立ちます。

ツールを使いこなせば、コードに集中でき、コンテキストの切り替えを減らすことができます。

## まとめ

この時点で、Goがインストールされていて、エディタが利用可能で、基本的なツールがいくつか用意されているはずです。Goには、サードパーティ製品の非常に大きなエコシステムがあります。ここでは、便利なコンポーネントをいくつか紹介します。

[https://awesome-go.com](https://awesome-go.com)を参照してください。
