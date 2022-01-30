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

### Result

・ジェネリックな列挙型、失敗する可能性のある値を返す
・カンマで区切られた複数のパラメータ化された型を持つ

```
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

・Ok と Err を使えばどこでもインスタンスを生成

```
fn do_something_that_might_fail(i:i32) -> Result<f32, String> {
  if i == 42 {
    Ok(13.0)
  } else {
    Err(String::from("正しい値ではない"))
  }
}
fn main() {
  let result = do_something_that_might_fail(12);
  match result {
    Ok(v) => println!("発見 {}", v),
    Err(e) => println!("Error {}", e),
  }
}
```

### Option

・Option はジェネリックな列挙型が組み込まれている
・null を使わず null 許容な値を表現できる
・Some と None を使うとどこでもインスタンスを生成できる

```
struct BagOfHolding<T> {
  item: Option<T>,
}
fn main() {
  let i32_bag = BagOfHolding::<i32> { item: None };
  if i32_bag.item.is_none() {
    println!("バッグには何もない")
  } else {
    println!("バッグには何かある")
  }
  let i32_bag = BagOfHolding::<i32> { item: Some(42) };
  if i32_bag.item.is_some() {
    println!("何かある")
  } else {
    println!("何もない")
  }
  match i32_bag.item {
    Some(v) => println!("バッグに {} を発見", v),
    None => println!("何もなかった")
  }
}

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
