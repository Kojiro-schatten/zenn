---
title: "Tour of Rust 第3章(基本的なデータ構造体)まとめ"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: true
---

ここでのまとめ

```
構造体
メソッドの定義
メモリ
メモリの中でデータを作成する
タプルライクな構造体
列挙型
データを持つ列挙型
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

## メソッドの定義

・関数と違い、メソッドは特定のデータ型と紐づく関数
・static: ある型そのものに紐づく。演算子 :: で呼び出せる
・instance: ある型のインスタンスに紐づく。演算子 . で呼び出せる

```
fn main() {
  let s = String::from("hello world");
  println!("{} is {} characters long.", s, s.len);
}
```

## メモリ

３つのメモリ空間がある

・データメモリ：固定長もしくは static なデータ。プログラム内の文字列("hello world")など、
文字列は読み取りにしか使えないため、データメモリ領域に入る。コンパイルはこういったデータに対して
チューニングし、メモリ上の位置は既に知られていてかつ固定であるため、非常に早く使える

・スタックメモリ：関数内で宣言された変数。関数が呼び出されている間は、メモリ上の位置は変更されることがないため、
コンパイラからチューニングが出来る。結果、スタックメモリも非常に早くデータにアクセスできる

・ヒープメモリ：プログラムの実行時に、作られるデータ。このメモリにあるデータは追加、移動、削除、サイズなどの
操作が許される。動的だが、メモリの使い方に柔軟性を生み出せる。データをヒープメモリに入れることを「allocation」
といい、データをヒープメモリから削除することを「deallocation」と言う。

## メモリの中でデータを作成する

・コードの中で構造体をインスタンス化する際、プログラムはフィールドデータをメモリ上で隣り合うように作成する
・全フィールドの値を指定して、インスタンス化する場合
構造体{...}.
というように演算子 . で取り出せる
・

```
structure SeaCreature {
  animal_type: String,
  name: String,
  arms: i32,
  legs: i32,
  weapon: String,
}

fn main() {
  // SeaCreature のデータはスタックに入る
  let ferris = SeaCreature {
    // String構造体もスタックに入りますが、ヒープに入るデータの参照アドレスが１つ
    // ダブルクォーとで囲まれているテキスト＝読み取り専用データ＝データメモリに入る
    // String::from と  SeaCreature のフィールドを隣り合う形でスタックに入れる
    // フィールドの値は変更可能で、メモリ上では下記のように変更される
    // 1: ヒープに変更可能なメモリを作る、テキストを入れる
    // 2: 1で作成した参照アドレスをヒープに保存、それを String に保存
    animal_type: String::from("crab"),
    name: String::from("Ferris"),
    arms: 2,
    legs: 4,
    weapon: String::from("claw"),
  };
  let sarah = SeaCreature {
    animal_type: String::from("octopus"),
    name: String::from("Sarah"),
    arms: 8,
    legs: 0,
    weapon: String::from("none"),
  }
  println!(
    "{} is a {}. They have {} arms, {} legs and {} weapon",
    ferris.name, ferris.animal_type, ferris.arms, ferris.legs, ferris.weapon
  );
  println!(
    "{} is a {}. They have {} arms, and {} legs. They have no weapon..",
    sarah.name, sarah.animal_type, sarah.arms, sarah.legs, sarah.weapon
  )
}
```

## タプルライクな構造体

```
struct Location(i32, i32);
fn main() {
  let loc = Location(42, 32);
  println!("{}, {}", loc.0, loc.1);
}
```

## ユニットライクな構造体

・フィールドを持たない構造体を宣言できる
・unit は空ユニット() の別名称
・あまり使われない

```
struct Marker;
fn main() {
  let _m = Marker;
}
```

## 列挙型

・キーワード enum で新しい（いくつかのタグ付けされた値を持てる）型が生成できる
・match は保有する全ての列挙値を処理する手助けができる。

```
#![allow(dead_code)] // この行でコンパイラのwaringsメッセージを止めます。
enum Species {
  Crab,
  Octopus,
  Fish,
  Clam
}
struct SeaCreature {
  species: Species,
  name: String,
  arms: i32,
  legs: i32,
  weapong: String,
}
fn main() {
  let ferris = SeaCreature {
    species: Species::Crab,
    name: String::from("Ferris"),
    arms: 2,
    legs: 4,
    weapong: String::from("claw"),
  };
  match ferris.species {
    Species::Crab => println!("{} is a crab", ferris.name),
    Species::Octopus => println!("{} is a octopus", ferris.name),
    Species::Fish => println!("{} is a fish", ferris.name),
    Species::Clam => println!("{} is a clam", ferris.name),
  }
}
=> Ferris is a crab
```

## データを持つ列挙型

・enum は一個もしくは複数な型のデータを持つことができ、C 言語の union のような表現ができる
・match を用いて列挙値に対するパターンマッチングを行う際、かくデータを変数名に束縛することができる
・列挙のメモリ事情
・・列挙型のメモリサイズはそれが持つ最大要素のサイズと等しい
・・・そのため、代替加入な同じサイズのメモリ空間を利用することができる
・・要素の型以外に、各要素には数字値がついている
・複数の型を組み合わせて新しい型を作ることができる（Rust が algebraic types(代数型) を持つと言われ理由）

```
#![allow(dead_code)] // この行でコンパイラのwaringsメッセージを止めます。
enum Species { Crab, Octopus, Fish, Clam }
enum PoisonType { Acidic, Painful ,Lethal }
enum Size { Big, Small }
enum Weapon {
  Claw(i32, Size),
  Poison(PoisonType),
  None
}

struct SeaCreature {
  species: Species,
  name: String,
  arms: i32,
  legs: i32,
  weapon: Weapon,
}

fn main() {
  let ferris = SeaCreature {
    species: Species::Crab,
    name: String::from("Ferris"),
    arms: 2,
    legs: 4,
    weapon: Weapon::Claw(2, Size::Small),
  };
  match ferris.species {
    Species::Crab => {
      match ferris.weapon {
        Weapon::Claw(num_claws, size) => {
          let size_description = match size {
            Size::Big => "big",
            Size::Small => "small"
          };
          println!("ferris is a crab with {} {} claws", num_claws, size_description)
        },
        _ => println!("ferris is a crab with some other weapong")
      }
    },
    _ => println!("feris is some other animal"),
  }
}

=> ferris is a crab with 2 small claws
```
