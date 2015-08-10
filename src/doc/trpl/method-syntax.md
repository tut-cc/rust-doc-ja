% メソッドシンタックス

関数は素晴らしいのですが、データに対し複数の関数をまとめて適用したいときは困ったことになります。

このコードについて考えてみましょう。

```rust,ignore
baz(bar(foo)));
```

私たちはこれを左から右へ'baz bar foo'と読みますが、関数が呼び出される順番は異なり、内側から外へ'foo bar baz'となります。もし代わりにこう書けたらいいとは思いませんか？

```rust,ignore
foo.bar().baz();
```

もう分かっているかもしれませんが、あなたは幸いにもこう書けるのです！Rustは`impl`キーワードでこの「メソッド呼び出し構文」を提供しています。

# メソッド呼び出し

どんな風に書けるかがこちらになります。

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

fn main() {
    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
    println!("{}", c.area());
}
```

これは`12.566371`と出力します。


私たちは円を表す構造体を作りました。加えて`impl`ブロックを書き、その中に`area`というメソッドを定義しました。

メソッドは特別に、`self`、`&self`、`&mut self`の3種類の内1つを第1引数に取ります。`foo.bar()`ならメソッドの第1引数は`foo`であると考えて下さい。3種類の引数はそれぞれ、`self`がスタック上の値である場合、`&self`が参照である場合、`&mut self`がミュータブル参照である場合に対応しています。`area`は`&self`で受け取っていますから、いつもの引数と同じように扱うことができます。`Circle`も内容が分かっていますから、`radius`へアクセスするのに普通の構造体との違いはありません。

あなたは所有権を得るよりも借用を好んで使うべきですし、ミュータブルな値や参照よりもイミュータブルな参照を使うべきですから、`&self`を常用すべきです。以下が3種類全ての例です。

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn reference(&self) {
       println!("selfは参照で受け取ります！");
    }

    fn mutable_reference(&mut self) {
       println!("selfはミュータブル参照で受け取ります！");
    }

    fn takes_ownership(self) {
       println!("selfの所有権を受け取ります！");
    }
}
```

# メソッド呼び出しの連鎖

ここまでで、`foo.bar()`というメソッドの呼び出し方は分かりましたね。ですが最初の例の`foo.bar().baz()`についてはどうでしょう？これは`メソッドチェーン`と呼ばれており、`self`自体を返り値とすることで実現できます。

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

    fn grow(&self, increment: f64) -> Circle {
        Circle { x: self.x, y: self.y, radius: self.radius + increment }
    }
}

fn main() {
    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
    println!("{}", c.area());

    let d = c.grow(2.0).area();
    println!("{}", d);
}
```

返り値の型を確認して下さい。

```
# struct Circle;
# impl Circle {
fn grow(&self) -> Circle {
# Circle } }
```

単に`Circle`を返しているだけです。この関数により、私たちは任意のサイズの円を作成できるようになりました。

# 関連付けられた関数

また、あなたは`self`を引数に取らずとも`Circle`に関連付けられた関数を定義することができます。
以下はRustのコードではごくありふれたパターンです。

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn new(x: f64, y: f64, radius: f64) -> Circle {
        Circle {
            x: x,
            y: y,
            radius: radius,
        }
    }
}

fn main() {
    let c = Circle::new(0.0, 0.0, 2.0);
}
```

この`関連付けられた関数`は新たに`Circle`を生成します。この関数は`ref.method()`ではなく、`Struct::function()`という構文で呼び出されることに気をつけて下さい。幾つかの言語では、関連付けられた関数を`静的メソッド`と呼んでいます。

# Builderパターン

ユーザーが円を作成できる機能を追加したいものの、ユーザーに許可できるのは`Circle`の初期値を設定することだけだと仮定しましょう。なお、指定が無ければ`x`と`y`の値を`0.0`に、`radius`を`1.0`に設定するものとします。Rustはメソッドのオーバーロードや名前付き引数、可変個引数といった機能がない代わりにBuilderパターンを採用しており、それは以下のようになります。

```
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

struct CircleBuilder {
    x: f64,
    y: f64,
    radius: f64,
}

impl CircleBuilder {
    fn new() -> CircleBuilder {
        CircleBuilder { x: 0.0, y: 0.0, radius: 1.0, }
    }

    fn x(&mut self, coordinate: f64) -> &mut CircleBuilder {
        self.x = coordinate;
        self
    }

    fn y(&mut self, coordinate: f64) -> &mut CircleBuilder {
        self.y = coordinate;
        self
    }

    fn radius(&mut self, radius: f64) -> &mut CircleBuilder {
        self.radius = radius;
        self
    }

    fn finalize(&self) -> Circle {
        Circle { x: self.x, y: self.y, radius: self.radius }
    }
}

fn main() {
    let c = CircleBuilder::new()
                .x(1.0)
                .y(2.0)
                .radius(2.0)
                .finalize();

    println!("area: {}", c.area());
    println!("x: {}", c.x);
    println!("y: {}", c.y);
}
```

私たちはここでもう1つの構造体である`CircleBuilder`を作成し、Builderメソッドを定義しました。また、`Circle`に`area`メソッドを定義し、加えて`CircleBuilder`に`finalize()`というメソッドを作りました。このメソッドはBuilderメソッドで設定した値を基に、最終的な`Circle`を生成します。今、私たちは最初の仮定をユーザーに強制させるために型システムを利用しました。この`CircleBuilder`の手法を使うことで、用意した方法から選んで値(今回は`Circle`)を生成するという制約が実現できます。
