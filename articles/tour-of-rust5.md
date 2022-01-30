---
title: "Tour of Rust 第5章(データの所有権と借用)まとめ"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: false
---

ここでのまとめ

```
所有権
スコープベースのリソース管理
ドロップは階層的
所有権の移動
所有権を返す
参照による所有権の借用
参照による所有権の可変な借用
参照外し
借用したデータの受け渡し
参照の参照
明示的なライフタイム
複数のライフタイム
スタティックライフタイム
データ型のライフタイム

```

## 所有権

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

### スコープベースのリソース管理

・関数と違い、メソッドは特定のデータ型と紐づく関数
・static: ある型そのものに紐づく。演算子 :: で呼び出せる
・instance: ある型のインスタンスに紐づく。演算子 . で呼び出せる

```
fn main() {
  let s = String::from("hello world");
  println!("{} is {} characters long.", s, s.len);
}
```

### ドロップは階層的

３つのメモリ空間がある

・データメモリ：固定長もしくは static なデータ。プログラム内の文字列("hello world")など、
文字列は読み取りにしか使えないため、データメモリ領域に入る。コンパイルはこういったデータに対して
チューニングし、メモリ上の位置は既に知られていてかつ固定であるため、非常に早く使える

・スタックメモリ：関数内で宣言された変数。関数が呼び出されている間は、メモリ上の位置は変更されることがないため、
コンパイラからチューニングが出来る。結果、スタックメモリも非常に早くデータにアクセスできる

・ヒープメモリ：プログラムの実行時に、作られるデータ。このメモリにあるデータは追加、移動、削除、サイズなどの
操作が許される。動的だが、メモリの使い方に柔軟性を生み出せる。データをヒープメモリに入れることを「allocation」
といい、データをヒープメモリから削除することを「deallocation」と言う。

## 所有権の移動

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

### 所有権を返す

```
struct Location(i32, i32);
fn main() {
  let loc = Location(42, 32);
  println!("{}, {}", loc.0, loc.1);
}
```

## 参照による所有権の借用

・フィールドを持たない構造体を宣言できる
・unit は空ユニット() の別名称
・あまり使われない

```
struct Marker;
fn main() {
  let _m = Marker;
}
```

## 参照による所有権の可変な借用

・キーワード enum で新しい（いくつかのタグ付けされた値を持てる）型が生成できる
・match は保有する全ての列挙値を処理する手助けができる。

```

```

## 参照外し

・enum は一個もしくは複数な型のデータを持つことができ、C 言語の union のような表現ができる

・複数の型を組み合わせて新しい型を作ることができる（Rust が algebraic types(代数型) を持つと言われ理由）

```

```

## 借用したデータの受け渡し

・enum は一個もしくは複数な型のデータを持つことができ、C 言語の union のような表現ができる

・複数の型を組み合わせて新しい型を作ることができる（Rust が algebraic types(代数型) を持つと言われ理由）

```

```

## 参照の参照

・enum は一個もしくは複数な型のデータを持つことができ、C 言語の union のような表現ができる

・複数の型を組み合わせて新しい型を作ることができる（Rust が algebraic types(代数型) を持つと言われ理由）

```

```

## 明示的なライフタイム

・enum は一個もしくは複数な型のデータを持つことができ、C 言語の union のような表現ができる

・複数の型を組み合わせて新しい型を作ることができる（Rust が algebraic types(代数型) を持つと言われ理由）

```

```

## 複数のライフタイム

・enum は一個もしくは複数な型のデータを持つことができ、C 言語の union のような表現ができる

・複数の型を組み合わせて新しい型を作ることができる（Rust が algebraic types(代数型) を持つと言われ理由）

```

```

## スタティックライフタイム

・enum は一個もしくは複数な型のデータを持つことができ、C 言語の union のような表現ができる

・複数の型を組み合わせて新しい型を作ることができる（Rust が algebraic types(代数型) を持つと言われ理由）

```

```

## データ型のライフタイム

・enum は一個もしくは複数な型のデータを持つことができ、C 言語の union のような表現ができる

・複数の型を組み合わせて新しい型を作ることができる（Rust が algebraic types(代数型) を持つと言われ理由）

```

```
