---
title: "Tour of Rust 第4章(ジェネリック型)まとめ"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: false
---

ジェネリック型は、null 許容な値の表現、エラー処理、コレクションなどに使用される

ここでのまとめ

```
ジェネリック型とは？
値がないことの表現
Option
for
match
loop から値を返す
ブロック式から値を返す
```

## ジェネリック型とは？

・ジェネリック型は struct や enum を部分的に定義することを可能にする
・ジェネリック型は、コンパイル時に型が生成される
・Rust は、インスタンスを生成するコードから型を推論できる
・しかし、::<T> 演算子(turbofish)を使って明示的に指定もできる

```
// <T> コンパイル時に型が生成される
struct BagOfHolding<T> {
  item: T,
}
fn main() {
  // turbofishによる型の指定
  let i32_bag = BagOfHolding::<i32> { item: 42 };
  let bool_bag = BagOfHolding::<bool> { item: true };
  // ジェネリック型での型推論
  let float_bag = BagOfHolding { item: 3.14 };
  let bag_in_bag = BagOfHolding {
    item: BagOfHolding { item: "Boom!"},
  };
  println!(
    "{} {} {} {}",
    i32_bag.item, bool_bag.item, float_bag.item, bag_in_bag.item.item
  )
}
=> 42 true 3.14 Boom!
```

### 値がないことの表現

・Rust に null は無い
・代わりに、1 つ以上の値を None によって代替する
・

```
enum Item {
  Inventory(String),
  None, // 項目が無いことを表す
}
struct BagOfHolding {
  item: Item,
}
=>
   Compiling playground v0.0.1 (/playground)
warning: enum is never used: `Item`
 --> src/lib.rs:1:6
  |
1 | enum Item {
  |      ^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: struct is never constructed: `BagOfHolding`
 --> src/lib.rs:7:8
  |
7 | struct BagOfHolding {
  |        ^^^^^^^^^^^^

warning: `playground` (lib) generated 2 warnings
    Finished dev [unoptimized + debuginfo] target(s) in 4.20s
Warnings
No main function was detected, so your code was compiled
but not run. If you’d like to execute your code, please
add a main function.
```

エラーは吐かず、warning になる。

### Option

・Option とはジェネリックな列挙型が組み込まれている
・null を使わず null 許容な値を表現できる
・Some と None を使うとどこでもインスタンスを生成できる

```
fn main() {
  let mut x = 0;
  while x != 32 {
    x += 1;
  }
  println!("{}", x);
}
=> 32
```

## for

・イテレータとして表 k される式を反復処理する
・イテレータとは項目がなくなるまで「次の項目は何か」と質問することができるオブジェクト
・Rust では整数のシーケンスを生成するイテレータを簡単に生成できる
・ .. 演算子は、開始番号から終了番号-1 までの数値を生成するイテレータを作成する
・ ..= 演算子は、開始番号から終了番号までの数値を生成するイテレータを作成する

```
fn main() {
  for x in 0..5 {
    println!("{}", x);
  }
  for x in 0..=5 {
    println!("{}", x);
  }
}

=> 0 1 2 3 4
=> 0 1 2 3 4 5
```

### match

・value が取り得る条件全てにマッチさせて、true なら true 時の処理を走らせられる

```
fn main() {
  let x = 42;
  match x {
    0 => {
      println!("zero");
    }
    1 | 2 => {
      println!("found 1 or 2");
    }
    //3 ~ 9
    3..=9 => {
      println!("found a number 3 to 9 inclusively");
    }
    matched_num @ 10..=100 => {
      println!("found {} number between 10 to 100!", matched_num);
    }
    // default処理
    _ => {
      println!("found something else");
    }
  }
}
=> found 42 number between 10 to 100!
```

## loop から値を返す

・loop は break で抜けて値を返すことができる

```
fn main() {
  let mut x = 0;
  let v = loop {
    x += 1;
    if x == 13 {
      break "13 を発見";
    }
  };
  println!("loop の戻り値: {}", v);
}
=> loop の戻り値: 13 を発見
```

## ブロック式から値を返す

・if, match, 関数, ブロックは単一の方法で値を返せる
・if, match, 関数, ブロックの最後が; のない式なら、戻り値として使用される
・if 文は三項演算子のように使用できる

```
fn example() -> i32 {
  let x = 42;
  let v = if x > 42 { -1 } else { 1 }; // 三項演算子
  println!("if より: {}", v);
  let food = "ハンバーガー";
  let result = match food {
    "ホットドッグ" => "ホットドッグです",
    _ => "ホットドッグではありません",
  };
  println!("食品の識別: {}", result);
  let v = {
    let a = 1;
    let b = 2;
    a + b
  };
  println!("ブロックより: {}", v);
  // rust　で関数の最後から値を返す寛容的な方法
  v + 4
}
fn main() {
  println!("関数より: {}", example());
}
=>
if より: 1
食品の識別: ホットドッグではありません
ブロックより: 3
関数より: 7
```
