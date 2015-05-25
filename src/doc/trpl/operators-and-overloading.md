% 演算子と演算子オーバーロード

Rust では、演算子をオーバーロードできますが、すべての演算子がオーバーロードできるわけではありません。
特定のトレイトを実装することによって、各演算子をオーバーロードできます。

たとえば、 `+` 演算子は `Add` トレイトを実装することでオーバーロードできます。

```rust
use std::ops::Add;

#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point { x: self.x + other.x, y: self.y + other.y }
    }
}

fn main() {
    let p1 = Point { x: 1, y: 0 };
    let p2 = Point { x: 2, y: 3 };

    let p3 = p1 + p2;

    println!("{:?}", p3);
}
```

`Point` に `Add<Output=Point>` を実装しているので、 `main` では、二つの `Point` に対して `+` 演算子が使用できます。

この方法でオーバーロードできる演算子はいくつかあり、 [`std::ops`][stdops] に各演算子に対応するトレイトがあります。
どのような演算子がオーバーロードでき、どのようなトレイトがあるかは、ドキュメントを確認してください。

[stdops]: ../std/ops/index.html

これらのトレイトの実装方法は、あるパターンに従っています。
もう少し詳細に [`Add`][add] を確認してみましょう。

```rust
# mod foo {
pub trait Add<RHS = Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
# }
```

[add]: ../std/ops/trait.Add.html

`Add` では、合計で3つの型が関連付けされています。
トレイトを実装する `impl Add` に対して、 デフォルトで `Self` となっている `RHS` と `Output` の3つです。
たとえば`let z = x + y` という式では、 `x` が `Self` 型で、 `y` の型は `RHS`、 `z` が `Self::Output` 型となります。

```rust
# struct Point;
# use std::ops::Add;
impl Add<i32> for Point {
    type Output = f64;

    fn add(self, rhs: i32) -> f64 {
        // add an i32 to a Point and get an f64
# 1.0
    }
}
```

上記のように実装したのであれば、次のように書くことができます。

```rust,ignore
let p: Point = // ...
let x: f64 = p + 2i32;
```
