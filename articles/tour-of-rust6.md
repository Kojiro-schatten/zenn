---
title: "Tour of Rust 第6章(テキスト)まとめ"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: false
---

ここでのまとめ

```
文字列リテラル
utf-8とは
エスケープ文字
複数行の文字列リテラル
生文字列リテラル
ファイルから文字列リテラルを読み込む
文字列スライス
Chars
String
関数パラメータとしてのテキスト
文字列の構築
文字列のフォーマット
文字列変換
```

## 文字列リテラル

・文字列リテラルは常に Unicode
・型は&'static str
・& はメモリ中の場所を参照していることを意味する
・&mut を欠いているのはコンパイラがそれを修正することを認めていないということ
・'static は文字列データがプログラムの終了まで有効であるということ
・str は常に有効な utf-8 バイト列を指している

```
fn main() {
  let a: &'static str = "こんにちは";
  println!("{} {}", a, a.len());
}
=> こんにちは 15
```

15 という len は、つまり日本語 1 文字に対し 3byte であることを意味する
英語の場合("asdf")なら、len は 4
アイコンの場合("🔔") は、4

### utf-8 とは

・

```

```

### エスケープ文字

・

```

```

## 複数行の文字列リテラル

・

```

```

### 生文字列リテラル

・

```

```

## ファイルから文字列リテラルを読み込む

・

```

```

## 文字列スライス

・

```

```

## Chars

・

```

```

## String

・

```

```

## 関数パラメータとしてのテキスト

・

```

```

## 文字列の構築

・

```

```

## 文字列のフォーマット

・

```

```

## 文字列変換

・

```

```