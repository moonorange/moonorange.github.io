---
title: 'Golang Basic(Golang基礎)'
date: '2021-12-11'
categories: ["Programming Language", "Go"]
tags: ["Go", "Japanese Article"]
---

# 概要

Golang についての基礎的な情報をまとめる資料である。

## Go とは？

Google が開発したプログラミング言語であり、2009 年に最初のバージョンがオープンソースでリリースされた。

以下のような特徴を持つ。

- 強力でシンプルな言語設計と文法
- 並行プログラミング
- 豊富な標準ライブラリ群
- 周辺ツールの充実
- シングルバイナリ、クロスコンパイル

### 強力でシンプルな言語設計と文法

スクリプト言語、LL のような書きやすさがあり冗長な記述が必要ない(型推論などもある)。

型のある言語の厳密さ。e.g. 整数と浮動小数点の演算などをしようとするとコンパイルエラーとなる。

機能を増やすことで言語を拡張していくことはない。言語拡張が他言語に比べ慎重でありシンプルに保たれるよう意識されている。

### 並行プログラミング

軽量なスレッドのようなものであるゴールーチンという機構を持つ。

go キーワードをつけ関数を呼び出すだけで並行処理を実現できる。

例：https://go.dev/play/p/yLva1AwfP06.go

```go
// 関数fを別のgoroutineで呼び出し
go f()
```

マルチスレッド間のデータのやり取りは難しいが、Go ではチャネルという機能を使って実現している。

ゴールーチン間のデータのやり取りを安全に行えるよう設計されている。

### 標準ライブラリ一覧

- fmt: 書式に関する処理
- net/http: HTTP サーバーなど
- archive, compress: zip や gzip など
- encoding: json, xml, csv など
- html/template: HTML テンプレート
- os, path/filepath: ファイル操作など

### 周辺ツールの充実

go tool として標準/準標準で様々なツールが提供されている。

- go build: コンパイルして実行可能ファイル（バイナリ）を生成
- go run: コンパイルから実行までを行う
- go test: xxx_test.go に書かれたテストの実行
- godoc: ドキュメント生成
- gofmt goimports: コードフォーマッター
- go vet: コードチェッカー、リンター
- gopls: Language Server Protocol の提供

### シングルバイナリ、クロスコンパイル

クロスコンパイルできる。開発環境とは違う OS やアーキテクチャ向けのバイナリを作成できる。

```bash
# Linux(64ビット)向けにコンパイル
$ GOOS=linux GOARCH=amd64 go build
```

シングルバイナリになるので動作環境を用意しなくて良い

e.g. スクリプト言語で必要になるような処理系をインストールせずに非エンジニアの pc で動く

## Variable definition

```go
var n int = 0

var n int
n = 0

var n = 100

// Omit var(only in function)
n := 0

var (
    n = 0
    m = 1
)
```

## Builtin types

- Built-in string type: string.
- Built-in boolean type: bool.
- Built-in numeric types:
  int8, uint8 (byte), int16, uint16, int32 (rune), uint32, int64, uint64, int, uint, uintptr.
  float32, float64.
  complex64, complex128.

Note, byte is a built-in alias of uint8, and rune is a built-in alias of int32

## Constants

```go
const n int = 0

// 型のない定数
const n = 0

const (
    x = 0
    y = 1
)
```

シフト演算 `1 << 2` 4

シフト演算とはビット列を左または右にずらすこと。

上記の例だと 8 ビットだとして 00000001 を右に２つずらす(溢れたビットは捨てる)と 00000100 となり 10 進数で 4 となる。

## コンポジット型

複数のデータ型が集まって一つの型になっているもの

- 構造体　　型の異なるデータ型を集めたデータ型
- 配列　　　同じ型のデータを集めて並べたデータ型
- スライス　配列の一部を切り出したデータ型
- マップ　　キーと値をマッピングさせたデータ型

### スライスと配列の違い

スライスとは配列の一部を取り出したデータ構造

- 要素の型はすべて同じ
- ポインタ、len, cap を持つ

## ポインタ

Go 言語ではポインタが使える。基本的には C と同じように使える。

```go
func f(xp *int) {
    *xp = 100 // put 100 to the xp pointer using *
}

func main() {
    var x int
    f(&x) // Get the x pointer by &
    println(x)
}
```

内部でポインタが用いられているデータ型

- スライス
- マップ
- チャネル

## メソッド

レシーバと紐づけられた関数

```go
type Receiver int
func (x Receiver) String() string {
    return fmt.Sprintf("%x", int(x))
}

var receiver Receiver = 1
fmt.Println(receiver.String())
```

## レシーバ

メソッドに関連付けられた変数

メソッド呼び出し時にはコピーが発生する

レシーバに変更を与えたい場合はポインタにする

### レシーバにできる型

- type で定義した型 ユーザ定義の型
- ポインタ型 レシーバに変更を与えたい場合に使う
- 内部にポインタを持つ型 マップやスライスなど

## パッケージ

関数や変数、定数、型を意味のある単位でまとめたもの

Go のプラグラムはパッケージを組み合わせることで実現される

先頭を大文字にした識別子が公開される

```go
var Hoge string // exported
var hoge string // not exported
```

### GOPATH

- Go のソースコードやビルドされたファイルが入るパスを設定できる
- インポートされるパッケージもここから検索される

環境変数として設定されておりデフォルトの値がある。

複数設定ができ、`go env GOPATH`で参照できる。

### Package management

コマンド

- go mod init (go mod init モジュール名)

指定したモジュール名で go.mod ファイルを作成する

モジュール名を省略すると GOPATH から推測する

- go mod tidy
  使用していないパッケージの go.mod からの削除

必要なパッケージのダウンロードと go.mod への追加

- go mod why
  指定したパッケージがなぜ必要になったか表示

- go mod vendor
  依存するパッケージを vendor 以下にコピーする

## GoDoc

ソースコード上に書かれたコメントをドキュメントとして生成する。

https://pkg.go.dev/golang.org/x/tools/cmd/godoc

ドキュメント生成

```console
godoc -http=:6060
```

## テスト

go test で単体テストを実行。

\_test.go というサフィックスの付いたファイルを
対象にしてテストを実行

### testing package

テストを行う機能を提供するパッケージ

使い方例

```go
type Hex int
func (h Hex) String() string {
    return fmt.Sprintf("%x", int(h))
}
```

```go
package mypkg_test
import "testing"
func TestHex_String(t *testing.T) {
    expect := "a"
    actual := mypkg.Hex(10).String()
    if actual != expect {
        t.Errorf(`expect="%s" actual="%s"`, expect, actual)
    }
}
```

## 参考

[Gopher 自習室](https://gopherdojo.org/)

[Go 101(a knowledge base for Go programming)](https://go101.org/)

[シフト演算を分かりやすく解説](https://medium-company.com/%E3%82%B7%E3%83%95%E3%83%88%E6%BC%94%E7%AE%97/)
