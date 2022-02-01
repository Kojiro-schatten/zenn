---
title: "Tour of Rust 第4章(ジェネリック型)まとめ"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: true
---

ジェネリック型は、null 許容な値の表現、エラー処理、コレクションなどに使用される

ここでのまとめ

```
ジェネリック型とは？
値がないことの表現
Option
Result
失敗するかもしれないmain
ベクタ型
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

## 値がないことの表現

・Rust に null は無いが、１つ以上の値を None によって代替できる

enum Item {
Inventory(String),
// None は項目が無いことを表す
None,
}

struct BagOfHolding {
item: Item,
}

## Option

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

## Result

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

## 失敗するかもしれない main

・main は Result を返せる

```
fn do_something_that_might_fail(i: i32) -> Result<f32, String> {
  if i == 42 {
    Ok(13.0)
  } else {
    Err(String::from("正しい値では無い"))
  }
}
fn main() -> Result<(), String> {
  let result = do_something_that_might_fail(12);
  match result {
    Ok(v) => println!("発見 {}", v),
    Err(_e) => {
      //エラーをうまく処理
      //何が起きたのかを説明する新しい Err を main から返す
        return Err(String::from("main で何か問題が起きました!"));
    }
  }
  // Result の Ok の中にある unit 値によって
  // 全てが正常であることを表現していることに注意
  Ok(())
}
=> main で何か問題が起きました!
```

## 簡潔なエラー処理

・Result はとてもよく使うため、Rust にはそれを扱うための ? 演算子が用意されている

```
fn do_something_that_might_fail(i: i32) -> Result<f32, String> {
  if i == 42 {
    Ok(13.0)
  } else {
    Err(String::from("正しい値では無い"))
  }
}
fn main() -> Result<(), String> {
  let v = do_something_that_might_fail(42)?;
  println!("発見 {}", v);
  Ok(())
}
=> 発見 13
```

## やっつけな Option/Result 処理

・Option, Result の両方には unwrap と呼ばれる関数がある。
・unwrap とは、
１：Option / Result 内の値を取得する
２：列挙型が None / Err の場合、panic! する

```
fn do_something_that_might_fail(i: i32) -> Result<f32, String> {
  if i == 42 {
    Ok(13.0)
  } else {
    Err(String::from("正しい値ではない"))
  }
}
fn main() -> Result<(), String> {
  let v = do_something_that_might_fail(42).unwrap();
  println!("発見 {}", v);
  let v = do_something_that_might_fail(1).unwrap();
  println!("発見 {}", v);
  Ok(())
}

```

良い Rust 使いであるためには、可能な限り適切に match を使用すること

## ベクタ型

・最も重要なジェネリック型のいくつかはコレクション型である
・ベクタは構造体 Vec で表される可変サイズのリストである
・マクロ vec! を使うと簡単にベクタを生成できる
・Vec には iter() メソッドがあり、簡単に for ループに入れられる

メモリに関する詳細
・Vec は構造体だが、内部的にはヒープ上の固定リストへの参照を含む
・ベクタはデフォルトの容量で始まる。容量よりも多くの項目が追加された時、ヒープ上により大きな容量の固定リストを生成して、データを再割り当てする

```
fn main() {
  let mut i32_vec = Vec::<i32>::new();
  i32_vec.push(1);
  i32_vec.push(2);
  i32_vec.push(3);

  // もっと賢く、型を自動的に推論
  let mut float_vec = Vec::new();
  float_vec.push(1.3);
  float_vec.push(2.3);
  float_vec.push(2.4);

  let string_vec = vec![String::from("Hello"), String::from("World")];
  for word in string_vec.iter() {
    println!("{}", word);
  }
}
```
