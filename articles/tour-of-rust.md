---
title: "Tour of Rust 第1章(基礎)まとめ"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: true
---

ここでのまとめ

```
変数
シャドーイングとは
変数の変更
型について
基本型の変換
定数
配列
関数
複数の戻り値
空の戻り値
```

## 変数

・let で宣言する
・値を割り当てる際、Rust は 99%のケースで変数の型を推論できる。できない場合、変数宣言に型を追加できる。
・変数名にはスネークケース(snake_case)を使用する

### シャドーイングとは

・同じ変数名に複数回割り当てられること。

```
fn main() {
  let x = 13;
  prinln!("{}", x);

  // x の型を指定
  let x: f64 = 3.14159;
  println!("{}", x);

  // 宣言の後で初期化: 基本使わない
  let x;
  x = 0;
  println!("{}", x);
}

=> 13 3.141659 0
```

### 変数の変更

Rust の変数の変更は、２種類に分類される。
・mutable: read & write
・immutable: readonly

mutable にする場合、

```
let mut x = 42;
```

と、mut キーワードを使う

## 型について

```
fn main() {
  let x = 12; // デフォルトでは i32
  let a = 12u8;
  let b = 4.3; // デフォルトでは f64
  let c = 4.3f32;
  let bv = true;
  let t = (13, false);
  let sentence = "hello world!";
  println!(
    "{} {} {} {} {} {} {} {}",
    x, a, b, c, bv, t.0, t.1, sentence
  );
}

=> 12 12 4.3 4.3 true 13 false hello world!
```

> u8, u32, u64, u128 => 符号なし整数型
> i8, i32, i64, i128 => 符号付き整数型
> f32, f64 => 浮動小数点数型
> usize, isize => ポインタサイズ整数型（メモリ内のインデックスとサイズを表す）
> 配列型（コンパイル時に長さが決まる同じ要素のコレクション）
> スライス型（実行時に長さが決まる同じ要素のコレクション）
> str(実行時に長さが決まるテキスト)

### 基本型の変換

・数値型を扱うとき、型を明示する必要がある。
・例えば、u8, u32 を混ぜるとエラーになる。
・as をつけることで、型変換ができる。

```
fn main() {
    let a = 13u8;
    let b = 7u32;
    let c = a + b; // 本来は、a か b を型変換する必要あり。
    println!("{}", c);

    let t = true;
    println!("{}", t as u8);
}
=> Compiling playground v0.0.1 (/playground)
error[E0308]: mismatched types
 --> src/main.rs:4:17
  |
4 |     let c = a + b;
  |                 ^ expected `u8`, found `u32`

error[E0277]: cannot add `u32` to `u8`
 --> src/main.rs:4:15
  |
4 |     let c = a + b;
  |               ^ no implementation for `u8 + u32`
  |
  = help: the trait `Add<u32>` is not implemented for `u8`

Some errors have detailed explanations: E0277, E0308.
For more information about an error, try `rustc --explain E0277`.
error: could not compile `playground` due to 2 previous errors
```

なので、

```
let c = a as u32 + b;
```

と型変換すると、コンパイルが通る。

## 定数

・明示的な型指定が必要
・大文字のスネークケースを使用する

```
const PI: f32 = 3.14159;
fn main() {
  println!(
    "ゼロからApple {} を作るには、まず宇宙を想像すること",
    PI
  );
}

```

## 配列

・データ要素が全て同じ型の固定長コレクション
・データ型は[T;N]となる。(T=要素の型、N=コンパイル時に決まる固定長)

```
fn main() {
  let nums: [i32; 3] = [1, 2, 3];
  println!("{:?}", nums);
  println!("{}", nums[1]);
}
=> [1,2,3] 2
```

## 関数

・0 個以上の引数がある
・関数名にはスネークケース(snake_case)を使う

```
fn add(x: i32, y:i32) -> i32 {
  return x + y;
}
fn main() {
  println!("{}", add(42, 133));
}
=> 175
```

## 複数の戻り値

・関数は値をタプルで返すことで複数の値を返せる
・タプルの要素はインデックス番号で参照可能

```
fn swap(x: i32, y:i32) -> (i32, i32) {
  return (y, x);
}
fn main() {
  let result = swap(123, 321);
  println!("{} {}", result.0, result.1);
  let (a, b) = swap(result.0, result.1);
  println!("{} {}", a, b);
}
=> 321 123
=> 123 321
```

## 空の戻り値

・関数に戻り値の型が指定されていない場合、unit と呼ばれる空のタプルを返す
・空のタプルは () と書く

```
fn make_nothing() -> () {
  return ();
}
fn make_nothing2() {
  // この関数は戻り値が指定されないため()を返す
}
fn main() {
  let a = make_nothing()
  let b = make_nothing2()
  println!("aの値: {:?}", a);
  println!("bの値: {:?}", b);
}
```
