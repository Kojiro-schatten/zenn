---
title: "Tour of Rust 第6章(テキスト)まとめ"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: true
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

## utf-8 とは

・utf-8 は 1~4 バイトの可変バイト長で導入され、文字の範囲が広くなった
・可変サイズ文字の利点は、テキストが非常に一般的な ASCII のための不要なバイトを持たなかったこと
・可変サイズ文字の欠点は、例えば my_text[3] における文字の検索が O(1) 定数時間で素早くできなくなること
・前の文字が、可変幅を持つことで、4 番目の文字がバイト列のどこから実際に始まるか、が変わってしまう可能性がある
・Unicode 文字が実際にどこから始まるのか知るために、utf-8 バイトのシーケンスを繰り返さなければならない（線形時間 O(n)）

## エスケープ文字

・特定の文字の表現に、エスケープ コード を使用して、その場所に記号を挿入することができる
・\n - 改行
・\r - キャリッジ・リターン
・\t - タブ
・\\ - バックスラッシュ
・\0 - null
・\' - シングルクォート

```
fn main() {
  let a: &'static str = "Ferris は言う: \t\" こんにちは\"";
  println!("{}", a);
}
=> Ferris は言う:  こんにちは
```

## 複数行の文字列リテラル

・文字列はデフォルトで複数行に対応している
・改行したくない場合、行の末尾に \

```
fn main() {
  let boinn: &'static str = "
  あ
  い
  う
  え
  お";
  println!("{}", boinn);
  println!("こんにちは \
  世界");
}
=> あ
   い
   う
   え
   お
   こんにちは 世界
```

## 生文字列リテラル

・ str = r#" ... "# とすることで、文字列が全部表示される

```
fn main() {
  let a: &'static str = r#"
    <div class="advice">
      生文字列は役立つ
    </div>
    "#;
    println!("{}", a);
}
=>  <div class="advice">
      生文字列は役立つ
    </div>
```

## ファイルから文字列リテラルを読み込む

・大きいテキストは、include_str! マクロを使ってローカルのファイルのテキストをプログラムの中で include する

```
let hello_html = include_str!("hello.html");
```

## 文字列スライス

・常に有効な utf-8 でなければならないメモリないのバイト列への参照
・文字列のスライス（サブスライス）である str のスライスも有効な utf-8 でなければならない。
・&str の一般的なメソッド: len, starts_with, ends_with, is_empty, find

```
fn main() {
  let a = "hi 🦀";
  println!("{}", a.len());
  let first_word = &a[0..2];
  let second_word = &a[3..7];
  println!("{} {}", first_word, second_word);
}
=> 7
hi 🦀
```

## Chars

・Unicode での作業が非常に困難なため、utf-8 バイトのシーケンスを char 型の文字のベクトルとして取得する方法が提供されている
・char` は常に 4 バイトの長さ

```
fn main() {
  // 文字をcharのベクトルとして集める
  let chars = "hi 🦀.chars().collect::<Vec<char>>();
  println!("{}", chars.len()); //should be 4
  // chars は ４バイトなのでu32に変換可能
  println!("{}", chars[3] as u32);
}
=> 4
129408
```

## String

・String はヒープに utf-8 バイト列をもつ構造体
・そのメモリはヒープ上にあるため、文字列リテラルではできないような、拡張/修正などが可能
・共通メソッド: push_str, replace, to_lowercase / to_uppercase, trim
・String がドロップされると、そのヒープ内のメモリもドロップされる

```
fn main() {
  let mut helloworld = String::from("hello");
  helloworld.push_str(" world");
  helloworld = helloworld + "!";
  println!("{}", helloworld);
}
=> hello world!
```

## 関数パラメータとしてのテキスト

・文字列リテラル、文字列は、一般的に文字列スライスとして関数に渡される
・それにより、所有権を渡す必要がないほとんどの場合で柔軟性が増す

```
fn say_it_loud(msg:&str) {
  println!("{}!!!", msg.to_string().to_uppercase());
}
fn main() {
  // say_it_loud は &'static str を &str として借用することができる
  say_it_loud("hello");
  say_it_loud(&String::from("goodbye"));
}
=> HELLO!!!
GOODBYE!!!
```

## 文字列の構築

・concat と join が使える

```
fn main() {
  let helloworld = ["hello", " ", "world!", "!"].concat();
  let abc = ["a", "b", "c"].join(",");
  println!("{}", helloworld);
  println!("{}", abc);
}
=> hello world!!
a,b,c
```

## 文字列のフォーマット

・format! マクロを使うと、パラメータ化された文字列を定義して文字列を作成することができる
・値をどこにどのように配置すべきかはプレースホルダ({})で指定する
・format! は println! と同じパラメータ付きの文字列を使用する

```
fn main() {
  let a = 42;
  let f = format!("secret to life: {}", a);
  println!("{}", f);
}
=> secret to life: 42
```

## 文字列変換

・to_string を使うと、文字列変換することができる
・ジェネリック関数 parse を用いることで文字列や、文字列リテラルを型付きの値に変換できる。失敗したら Result が返る

```
fn main() -> Result<(), std::num::ParseIntError> {
  let a = 42;
  let a_string = a.to_string();
  let b = a_string.parse::<i32>()?;
  println!("{} {}", a, b);
  Ok(())
}
```
