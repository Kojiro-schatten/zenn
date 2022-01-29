---
title: "Tour of Rust 第3章(基本的なデータ構造体)まとめ"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

ここでのまとめ

```
構造体
メソッドの定義
while
for
match
loop から値を返す
ブロック式から値を返す
```

## 構造体

・一つの struct はフィールドの集合
・フィールドとは「データ構造」と「キーワードを紐づける」値
・その値は、プリミティブ値かデータ構造を指定できる

```
struct SeaCreature {
  animal_type: String,
  name: String,
  arms: i32,
  legs: i32,
  weapon: String,
}
```

### メソッドの定義

・関数と違い、メソッドは特定のデータ型と紐づく関数
・static: ある型そのものに紐づく。演算子 :: で呼び出せる
・instance: ある型のインスタンスに紐づく。演算子 . で呼び出せる

```
fn main() {
  let s = String::from("hello world");
  println!("{} is {} characters long.", s, s.len);
}
```

### while

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
