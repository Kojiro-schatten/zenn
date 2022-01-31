---
title: "Tour of Rust 第2章(基本制御フロー)まとめ"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: true
---

ここでのまとめ

```
if/else if/else
loop
while
for
match
loop から値を返す
ブロック式から値を返す
```

## if/else if/else

・論理演算子が使える ==, !=, <, >, <=, >=, !, ||, &&

```
fn main() {
  let x = 42;
  if x < 42 {
    println!("42より小さい");
  } else if x == 42 {
    println!("42")
  } else {
    println!("42より大きい");
  }
}
=> 42
```

## loop

・break によってループから抜けれる

```
fn main() {
  let mut x = 0;
  loop {
    x += 1;
    if x == 42 {
      break;
    }
  }
  println!("{}", x);
}

=> 42
```

## while

・条件が false となればループは終了となる

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

## match

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
