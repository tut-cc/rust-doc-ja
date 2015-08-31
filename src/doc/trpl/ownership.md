# 所有権

これはRustの所有権システムを構成する3要素のうちの1つです。これはRustの最もユニークで魅力的な機能の1つであり、Rustの開発者が熟知していなければならない機能でもあります。所有権はRustの最大の目的であるメモリ安全性を実現するための方法です。個々の概念毎に章を用意してあります。

* 所有権は、今あなたが見ている章です。
* [借用][borrowing]では、「参照」に関連する機能について説明しています。
* [生存期][lifetimes]では、借用を発展させた概念について説明しています。

これら3つの章はその順番にも関係があります。所有権システムを完全に理解するためには3つ全ての読了が不可欠です。

[borrowing]: https://doc.rust-lang.org/book/references-and-borrowing.html
[lifetimes]: https://doc.rust-lang.org/book/lifetimes.html

# メタな話

詳細を見て行く前に、所有権システムについて2つ覚えておいて欲しいことがあります。

Rustは安全性と速度を目的としています。その目標は'ゼロコストの抽象化'、つまり抽象化のコストを必要最小限することで達成しています。所有権システムはゼロコストの抽象化の最たる例です。私たちがこのガイドで話す全ての詳細は_コンパイル時に実行されます_。これら機能を用いるのに実行時コストは掛かりません。

しかしながら、このシステムは確実にコストを要します。そう、学習に要するコストです。Rustを経験した多くの新規ユーザは有効なコードだと思ってもコンパイラに弾かれる、所謂'借用チェッカーとの戦い'を経験しています。これはプログラマが頭のなかで考える所有権の動作モデルとRustに実装された実際のルールが一致していないために起きてしまいます。あなたも恐らく初めは同じようなことを経験するでしょう。しかしながら良い知らせもあります。より経験を積んだRustの開発者曰く、一度でもある程度の間所有権システムのルールの下で働けば、借用チェッカーとの戦いは次第に減っていくとのことです。

このことを念頭において、所有権について学んでいきましょう。

# 所有権

[変数束縛][bindings]はRustにおいて所有権を持つ方法です。変数は束縛した対象の'所有権を持っている'のです。これは束縛している変数がスコープ外に出ると、束縛されていたリソースが開放されることを意味しています。例えば、

```rust
fn foo() {
    let v = vec![1, 2, 3];
}
```

`v`がスコープに入ってくると、新たな[`Vec<T>`][vect]が作成されます。このケースでは、vectorは3つの要素を[ヒープ領域][heap]にアロケートします。`foo()`が終了し`v`がスコープの外へ出ると、Rustはヒープ領域にアロケートされたメモリも含めvectorに関連する全てを片付けます。これはスコープの終端に達したとき、確定的に行われます。

[vect]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[heap]: https://doc.rust-lang.org/book/the-stack-and-the-heap.html
[bindings]: https://doc.rust-lang.org/book/variable-bindings.html

# ムーブセマンティクス

少し細かい話になりますが、Rustはあらゆるリソースについて束縛が厳密に1つだけということを保証します。例えばvectorを作成したとして、もう1つ束縛をアサインできます。

```rust
let v = vec![1, 2, 3];

let v2 = v;
```

しかし、その後`v`を使おうとすると、エラーが発生します。

```rust,ignore
let v = vec![1, 2, 3];

let v2 = v;

println!("v[0] is: {}", v[0]);
```

エラーの内容は次のようになります。

```text
error: use of moved value: `v`
println!("v[0] is: {}", v[0]);
                        ^
```

私たちが所有権を取る関数を定義し、引数として渡した後に変数を使おうとする場合にも同じようなことが起きます。

```rust,ignore
fn take(v: Vec<i32>) {
    // ここで起きることは重要ではありません。
}

let v = vec![1, 2, 3];

take(v);

println!("v[0] is: {}", v[0]);
```

'use of moved value'、同じエラーですね。何かに所有権を移すとき、私たちはその値を'ムーブ'したと言います。特別なアノテーションの類は必要なく、Rustはデフォルトでこれを行います。

## ムーブの詳細

ムーブした後の値を使うことができない理由は細かいようですが重要な話です。このようなコードを書いているとき、

```rust
let v = vec![1, 2, 3];

let v2 = v;
```

1行目はvectorオブジェクト`v`と格納している要素のためにメモリをアロケートしています。vectorオブジェクトは[スタック][sh]に保存されており、[ヒープ領域][sh]に保存されている要素(`[1, 2, 3]`)のポインタを格納しています。`v`を`v2`へムーブすると`v2`のためにポインタのコピーを作成します。するとヒープ領域上のvectorの要素を指すポインタが2つになってしまうため、データの競合を引き起こしRustの安全性の保証に違反してしまいます。従って、Rustはムーブした後の`v`の使用を禁じているのです。

[sh]: https://doc.rust-lang.org/book/the-stack-and-the-heap.html

また、状況に応じて最適化がスタック上のデータのコピーを取り除く可能性にも注目してください。ですから最初に思ったほど非効率的ではないかもしれません。

## `Copy` 型

所有権が他の束縛に移されると元の束縛が使用できない所までは話してきました。しかしながら、その振る舞いを変える[トレイト][traits]があり、それは`Copy`と呼ばれています。私たちはまだトレイトについて議論することができませんから、今だけ振る舞いを追加する特殊な型のためのアノテーションと考えてください。例えば、

```rust
let v = 1;

let v2 = v;

println!("v is: {}", v);
```

In this case, `v` is an `i32`, which implements the `Copy` trait. This means
that, just like a move, when we assign `v` to `v2`, a copy of the data is made.
But, unlike a move, we can still use `v` afterward. This is because an `i32`
has no pointers to data somewhere else, copying it is a full copy.

We will discuss how to make your own types `Copy` in the [traits][traits]
section.

[traits]: traits.html

# More than ownership

Of course, if we had to hand ownership back with every function we wrote:

```rust
fn foo(v: Vec<i32>) -> Vec<i32> {
    // do stuff with v

    // hand back ownership
    v
}
```

This would get very tedious. It gets worse the more things we want to take ownership of:

```rust
fn foo(v1: Vec<i32>, v2: Vec<i32>) -> (Vec<i32>, Vec<i32>, i32) {
    // do stuff with v1 and v2

    // hand back ownership, and the result of our function
    (v1, v2, 42)
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let (v1, v2, answer) = foo(v1, v2);
```

Ugh! The return type, return line, and calling the function gets way more
complicated.

Luckily, Rust offers a feature, borrowing, which helps us solve this problem.
It’s the topic of the next section!











