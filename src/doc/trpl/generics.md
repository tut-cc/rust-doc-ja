# ジェネリクス

度々、関数やデータ型を書いていると、引数が複数の型に対応したものが欲しくなることがあります。Rustでは、ジェネリクスを用いてそれを実現しています。ジェネリクスは型理論において'パラメトリック多相(parametric polymorphism)'と呼ばれており、与えられたパラメータにより('parametric')型や関数が多数の様相('poly'は「多様」、'morph'は「様相」を意味します)(訳注: ここで「様相」は型を指します)を持つことをそう呼びます。

さて、型理論はこれで十分です。続いてジェネリックなコードについてチェックしてみましょう。Rustが標準ライブラリで提供している型`Option<T>`はジェネリックです。

```rust
enum Option<T> {
    Some(T),
    None,
}
```

`<T>`の部分は、前に何度か見たことがあると思いますが、ジェネリックなデータ型であることを示しています。enumの宣言内にある`T`は同じ型に置き換えられます。型注釈を用いた`Option<T>`の使用例が以下になります。

```rust
let x: Option<i32> = Some(5);
```

型の宣言は`Option<i32>`とします。`Option<T>`とどこが違って見えますか？そう、この`Option`では`T`は`i32`の値を持つのです。このlet束縛の右辺の`Some(T)`では`T`は`5`となります。`i32`であれば両辺の型が一致するため、正しく型推論されます。型が不一致であれば以下のようなエラーが発生します。

```rust,ignore
let x: Option<f64> = Some(5);
// error: mismatched types: expected `core::option::Option<f64>`,
// found `core::option::Option<_>` (expected f64 but found integral variable)
```

`f64`で特殊化した`Option<T>`が作れないという意味ではありませんからね！リテラルの型も正しく合わせなければなりません。

```rust
let x: Option<i32> = Some(5);
let y: Option<f64> = Some(5.0f64);
```

これだけで結構です。1つ定義すれば様々な型で用いることができます。

ジェネリクスは1つの型だけに対応しているではありません。Rustの標準ライブラリに入っている`Result<T, E>`について考えてみましょう。

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

この型では`T`と`E`の_2つ_がジェネリックです。ちなみに、大文字の部分はあなたの好きにしてもらって構いません。もしあなたが望むなら`Result<T, E>`を、

```rust
enum Result<A, Z> {
    Ok(A),
    Err(Z),
}
```

のように定義しても良いです。"Type"から第1ジェネリックパラメータは`T`とすべきですし、"Error"から`E`とするのが慣例なのですが、Rustは気にしません。

`Result<T, E>`は計算の結果を返すのが目的の型であり、失敗した場合にエラーを示す値を返す機能を持っています。

## ジェネリック関数

私たちは似た構文でジェネリックな型をとる関数を記述することができます。

```rust
fn takes_anything<T>(x: T) {
    // do something with x
}
```

構文は2つのパーツから成ります。`<T>`は"この関数は`T`というパラメータを持つジェネリック関数である"ということであり、`x: T`は"xは`T`型である"という意味です。

複数の引数が同じジェネリック型を持つこともできます。

```rust
fn takes_two_of_the_same_things<T>(x: T, y: T) {
    // ...
}
```

複数の型を取るバージョンを記述することも可能です。

```rust
fn takes_two_things<T, U>(x: T, y: U) {
    // ...
}
```

ジェネリック関数は'トレイト束縛'と一緒に使えば最高に便利になります。これについては[トレイトの章][traits]で解説しています.

[traits]: https://doc.rust-lang.org/stable/book/traits.html

## ジェネリック構造体

あなたにもジェネリック型を保存する`struct`を書くことができます。

``` rust
struct Point<T> {
    x: T,
    y: T,
}

let int_origin = Point { x: 0, y: 0 };
let float_origin = Point { x: 0.0, y: 0.0 };
```

関数と同様に、`<T>`でジェネリックパラメータを宣言し、型を宣言するときは`x: T`とします。
