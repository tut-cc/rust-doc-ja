# トレイト

あなたは[method syntax][methodsyntax]で関数を呼び出すために用いた`impl`キーワードを覚えていますか？

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

[methodsyntax]: https://doc.rust-lang.org/stable/book/method-syntax.html

トレイトはそれに似ていますが、定義するときにはシグネチャだけを記述し、その後構造体毎に実装します。

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

trait HasArea {
    fn area(&self) -> f64;
}

impl HasArea for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

このように、`traits`の記述は`impl`とかなり似ているように見えますが、関数の実装は記述せず、シグネチャだけを定義します。トレイトを`実装`するときは、ただ`impl 実装の対象`とするのではなく、`impl トレイト for 実装の対象`とします。

ジェネリクスに制約を持たせるためにトレイトを使用できます。以下の関数について考えてみましょう。

```rust,ignore
fn print_area<T>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}
```

これはコンパイルできませんが、このようなエラーが発生するはずです。

```text
error: type `T` does not implement any method in scope named `area`
```

`T`はありとあらゆる型であるため、Rustは`.area()`メソッドが`T`について実装されているか確信を持てません。しかしジェネリック型`T`に対して'トレイトによる制約'を加えることで、`.area()`メソッドが実装されていることを保証できます。

```rust
trait HasArea {
    fn area(&self) -> f64;
}
fn print_area<T: HasArea>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}
```

`<T: HasArea>`構文は`HasAreaトレイトを実装するあらゆる型`を意味します。トレイトは関数のシグネチャを定義しているため、`HasArea`を実装するあらゆる型が`.area()`メソッドを持っていることを確認できます。
上記のコードを拡張した例がこちらです。

```rust
trait HasArea {
    fn area(&self) -> f64;
}

struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl HasArea for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}

struct Square {
    x: f64,
    y: f64,
    side: f64,
}

impl HasArea for Square {
    fn area(&self) -> f64 {
        self.side * self.side
    }
}

fn print_area<T: HasArea>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}

fn main() {
    let c = Circle {
        x: 0.0f64,
        y: 0.0f64,
        radius: 1.0f64,
    };

    let s = Square {
        x: 0.0f64,
        y: 0.0f64,
        side: 1.0f64,
    };

    print_area(c);
    print_area(s);
}
```

このプログラムの出力は、

```text
This shape has an area of 3.141593
This shape has an area of 1
```

上記の通り、`print_area`はジェネリックですが、正しい型が渡されることを保証します。もし不正な型が渡されるとすると、

```rust,ignore
print_area(5);
```

コンパイルエラーになります。

```text
error: failed to find an implementation of trait main::HasArea for int
```

これまでは構造体にのみトレイトの実装を追加していましたが、あなたはあらゆる型に対してトレイトを実装することができます。技術的には、`i32`のための`HasArea`を実装することも可能です。

```rust
trait HasArea {
    fn area(&self) -> f64;
}

impl HasArea for i32 {
    fn area(&self) -> f64 {
        println!("this is silly");

        *self as f64
    }
}

5.area();
```

しかし例え可能であったとしても、プリミティブ型が持つメソッドを実装する方法としては望ましくないと考えられています。

ここまでくると何でもありな様に見えますが、手が負えなくなるのを防ぐためにトレイトの実装周りには2つの制限が設けられています。第1に、あなたのスコープ内で定義されていないトレイトは適用されません。例えば、標準ライブラリは`File`にI/O機能を追加するための[`write`][write]トレイトを提供します。デフォルトでは、`File`はそのメソッドを持っていません。

[write]: https://doc.rust-lang.org/std/io/trait.Write.html

```rust,ignore
let mut f = std::fs::File::open("foo.txt").ok().expect("Couldn’t open foo.txt");
let result = f.write("whatever".as_bytes());
result.unwrap(); // ignore the error
```

エラーは以下のとおりです。

```text
error: type `std::fs::File` does not implement any method in scope named `write`

let result = f.write(b"whatever");
               ^~~~~~~~~~~~~~~~~~
```

`Write`トレイトのために、初めに`use`が必要となります。

```rust,ignore
use std::io::Write;

let mut f = std::fs::File::open("foo.txt").ok().expect("Couldn’t open foo.txt");
let result = f.write("whatever".as_bytes());
result.unwrap(); // ignore the error
```

これはエラー無しでコンパイルされます。

これは、例え誰かが`int`へメソッドを追加するような望ましくない何かを行ったとしても、あなたがトレイトの`use`を行わない限り、影響はないことを意味します。

トレイトの実装についての制限はもう1つあります。トレイト、またはあなたが書いている`impl`の対象となる型は、あなた自身によって実装されなければなりません。`HasArea`は私たちが記述したコードであるため、`i32`型のための`HasArea`を実装することができます。しかし、`i32`のためにRustによって提供されている`ToString`トレイトを実装したいとしても、トレイトと型が共に私たちの記述したコードでないため、それはできません。

最後に1つ。トレイトによって束縛されたジェネリック関数は`monomorphization`(mono:単一の、morph:形式)されるため、静的ディスパッチが行われます。一体どういう意味でしょうか？詳細については、[トレイトオブジェクト][to]の章をチェックしてください。

[to]: https://doc.rust-lang.org/stable/book/trait-objects.html

# 複数のトレイト束縛

ここまで、トレイトによってジェネリックな型パラメータが束縛できることを見てきました。

```rust
fn foo<T: Clone>(x: T) {
    x.clone();
}
```

もし2つ以上の束縛が必要なのであれば、`+`を使うことができます。

```rust
use std::fmt::Debug;

fn foo<T: Clone + Debug>(x: T) {
    x.clone();
    println!("{:?}", x);
}
```

`T`は今`Clone`と`Debug`、両方の実装が必要です。

# Where 節

ジェネリック型とトレイト拘束が少ないうちは良いのですが、数が増えるといよいよこの構文では不便になってきます。

``` rust
use std::fmt::Debug;

fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
}
```

関数名は左端にあり、引数リストは右端にあります。トレイト拘束を記述する部分が邪魔になっているのです。

そこでRustは'`where`節'と呼ばれる解決策を用意しています。

``` rust
use std::fmt::Debug;

fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
}

fn bar<T, K>(x: T, y: K) where T: Clone, K: Clone + Debug {
    x.clone();
    y.clone();
    println!("{:?}", y);
}

fn main() {
    foo("Hello", "world");
    bar("Hello", "workd");
}
```

`foo()`は先程見せたままの構文で、`bar()`は`where`節を用いています。あなたは型パラメータの定義時ではなく、引数リストの後ろに`where`を追加することでトレイト束縛を記述できます。長いリストであれば、空白を加えることもできます。

``` rust
use std::fmt::Debug;

fn bar<T, K>(x: T, y: K)
    where T: Clone,
          K: Clone + Debug {

    x.clone();
    y.clone();
    println!("{:?}", y);
}
```

この柔軟な構文により、複雑な状況であっても可読性を保つことができます。
また、`where`は基本的な構文よりも強力です。例えば、
(訳注: [std::convert::From](https://doc.rust-lang.org/std/convert/trait.From.html))
``` rust
trait ConvertTo<Output> {
    fn convert(&self) -> Output;
}

impl ConvertTo<i64> for i32 {
    fn convert(&self) -> i64 { *self as i64 }
}

// T == i32の時呼び出せる
fn normal<T: ConvertTo<i64>>(x: &T) -> i64 {
    x.convert()
}

// T == i64のとき呼び出せる
fn inverse<T>() -> T
        // これは"ConvertFrom<i32>"であるかのようにConvertToを用いている
        where i32: ConvertTo<T> {
    42.convert()
}
```

ここでは`where`節の追加機能を披露しています。この節は型パラメータではない任意の型(`T`ではなく、今回は`i32`のような型)に対してトレイト束縛が可能です。

## デフォルトメソッド

私たちがカバーしておくべきトレイトの最後の機能がデフォルトメソッドです。これの例を示すのは最も簡単です。

```rust
trait Foo {
    fn bar(&self);

    fn baz(&self) { println!("We called baz."); }
}
```

`Foo`トレイトの実装者は`bar()`を実装する必要がありますが、`baz()`を実装する必要はありません。`baz()`にはデフォルトの動作が与えられているからです。実装者の選択次第ではこのデフォルトの動作をオーバーライドすることも可能です。

```rust
trait Foo {
    fn bar(&self);
    fn baz(&self) { println!("We called baz."); }
}
struct UseDefault;

impl Foo for UseDefault {
    fn bar(&self) { println!("We called bar."); }
}

struct OverrideDefault;

impl Foo for OverrideDefault {
    fn bar(&self) { println!("We called bar."); }

    fn baz(&self) { println!("Override baz!"); }
}

let default = UseDefault;
default.baz(); // prints "We called baz."

let over = OverrideDefault;
over.baz(); // prints "Override baz!"
```

# 継承

時々、トレイトを実装するのに他のトレイトの実装が必要なこともあります。

```rust
trait Foo {
    fn foo(&self);
}

trait FooBar : Foo {
    fn foobar(&self);
}
```

`FooBar`の実装者は`Foo`もまた実装しなければなりません。以下のようになります。

```rust
trait Foo {
    fn foo(&self);
}
trait FooBar : Foo {
    fn foobar(&self);
}
struct Baz;

impl Foo for Baz {
    fn foo(&self) { println!("foo"); }
}

impl FooBar for Baz {
    fn foobar(&self) { println!("foobar"); }
}
```

もし`Foo`の実装を忘れると、Rustは以下のように知らせます。

```text
error: the trait `main::Foo` is not implemented for the type `main::Baz` [E0277]
```
