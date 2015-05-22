% Rust プログラミング言語

ようこそ!
この本は、 [Rust プログラミング言語][rust] について説明します。
Rust は、 3 つの目的 (安全性、速度、そして並列性)
に焦点を当てたシステムプログラミング言語です。
この言語はガーベジコレクタを持つことなくそれらの目的を維持します。
そして、他の言語が得意としないいくつかのユースケース
(他言語への埋め込み、具体的な空間・時間条件を要求するプログラム、
デバイスドライバやオペレーティングシステムのような低水準コードの記述)
に役立つ言語になっています。
実行時オーバヘッドのない幾つかのコンパイル時安全検査があることで、
そのスペースを目標としている現在の言語をより良いものにします。
またその一方で、全てのデータ競合を取り除きます。
たとえこれらの抽象化の一部が高級言語のもののように感じられるとしても、
Rust は「ゼロコストの抽象化」を成し遂げることを目指しています。
それでもなお、 Rust は低水準言語がするような正確な制御を可能としています。

[rust]: http://rust-lang.org

“Rust プログラミング言語”は 7 つの節に分かれています。
本序論がその 1 つ目です。 その後は、次のようになっています。

* [はじめに][gs] - Rust 開発のためのコンピュータのセットアップ
* [Rust の学習][lr] - 小規模プロジェクトを通じた Rust プログラミング
* [Effective Rust][er] - 優れた Rust コードを書くための高度な概念
* [構文と意味論][ss] - Rust を小さな塊へ分解
* [Nightly Rust][nr] - 安定版ビルドにはまだない最先端の機能
* [用語集][gl] - 本で使われる用語のリファレンス
* [学術研究][ar] - Rust に影響を与えた文献

[gs]: getting-started.html
[lr]: learn-rust.html
[er]: effective-rust.html
[ss]: syntax-and-semantics.html
[nr]: nightly-rust.html
[gl]: glossary.html
[ar]: academic-research.html

本序論を読んだ後、
あなたは好みに従って「Rust の学習」または「構文と意味論」
のどちらかに飛び込みたくなるでしょう。
もしプロジェクトに飛び込みたいなら「Rust の学習」を、
小さくはじめるのを好むなら「構文と意味論」を読み、
次へ進む前に完全に一つの概念を学びましょう。
おびただしいクロスリンキングが一緒にこれらのパートをつなぎます。

### 貢献

この本のソースファイルは、以下の Github で見つかります。
[github.com/tut-cc/rust-doc-ja/tree/stable/src/doc/trpl](https://github.com/tut-cc/rust-doc-ja/tree/stable/src/doc/trpl)

## A brief introduction to Rust

Is Rust a language you might be interested in? Let’s examine a few small code
samples to show off a few of its strengths.

The main concept that makes Rust unique is called ‘ownership’. Consider this
small example:

```rust
fn main() {
    let mut x = vec!["Hello", "world"];
}
```

This program makes a [variable binding][var] named `x`. The value of this
binding is a `Vec<T>`, a ‘vector’, that we create through a [macro][macro]
defined in the standard library. This macro is called `vec`, and we invoke
macros with a `!`. This follows a general principle of Rust: make things
explicit. Macros can do significantly more complicated things than function
calls, and so they’re visually distinct. The `!` also helps with parsing,
making tooling easier to write, which is also important.

We used `mut` to make `x` mutable: bindings are immutable by default in Rust.
We’ll be mutating this vector later in the example.

It’s also worth noting that we didn’t need a type annotation here: while Rust
is statically typed, we didn’t need to explicitly annotate the type. Rust has
type inference to balance out the power of static typing with the verbosity of
annotating types.

Rust prefers stack allocation to heap allocation: `x` is placed directly on the
stack. However, the `Vec<T>` type allocates space for the elements of the
vector on the heap. If you’re not familiar with this distinction, you can
ignore it for now, or check out [‘The Stack and the Heap’][heap]. As a systems
programming language, Rust gives you the ability to control how your memory is
allocated, but when we’re getting started, it’s less of a big deal.

[var]: variable-bindings.html
[macro]: macros.html
[heap]: the-stack-and-the-heap.html

Earlier, we mentioned that ‘ownership’ is the key new concept in Rust. In Rust
parlance, `x` is said to ‘own’ the vector. This means that when `x` goes out of
scope, the vector’s memory will be de-allocated. This is done deterministically
by the Rust compiler, rather than through a mechanism such as a garbage
collector. In other words, in Rust, you don’t call functions like `malloc` and
`free` yourself: the compiler statically determines when you need to allocate
or deallocate memory, and inserts those calls itself. To err is to be human,
but compilers never forget.

Let’s add another line to our example:

```rust
fn main() {
    let mut x = vec!["Hello", "world"];

    let y = &x[0];
}
```

We’ve introduced another binding, `y`. In this case, `y` is a ‘reference’ to
the first element of the vector. Rust’s references are similar to pointers in
other languages, but with additional compile-time safety checks. References
interact with the ownership system by [‘borrowing’][borrowing] what they point
to, rather than owning it. The difference is, when the reference goes out of
scope, it will not deallocate the underlying memory. If it did, we’d
de-allocate twice, which is bad!

[borrowing]: references-and-borrowing.html

Let’s add a third line. It looks innocent enough, but causes a compiler error:

```rust,ignore
fn main() {
    let mut x = vec!["Hello", "world"];

    let y = &x[0];

    x.push("foo");
}
```

`push` is a method on vectors that appends another element to the end of the
vector. When we try to compile this program, we get an error:

```text
error: cannot borrow `x` as mutable because it is also borrowed as immutable
    x.push("foo");
    ^
note: previous borrow of `x` occurs here; the immutable borrow prevents
subsequent moves or mutable borrows of `x` until the borrow ends
    let y = &x[0];
             ^
note: previous borrow ends here
fn main() {

}
^
```

Whew! The Rust compiler gives quite detailed errors at times, and this is one
of those times. As the error explains, while we made our binding mutable, we
still cannot call `push`. This is because we already have a reference to an
element of the vector, `y`. Mutating something while another reference exists
is dangerous, because we may invalidate the reference. In this specific case,
when we create the vector, we may have only allocated space for three elements.
Adding a fourth would mean allocating a new chunk of memory for all those elements,
copying the old values over, and updating the internal pointer to that memory.
That all works just fine. The problem is that `y` wouldn’t get updated, and so
we’d have a ‘dangling pointer’. That’s bad. Any use of `y` would be an error in
this case, and so the compiler has caught this for us.

So how do we solve this problem? There are two approaches we can take. The first
is making a copy rather than using a reference:

```rust
fn main() {
    let mut x = vec!["Hello", "world"];

    let y = x[0].clone();

    x.push("foo");
}
```

Rust has [move semantics][move] by default, so if we want to make a copy of some
data, we call the `clone()` method. In this example, `y` is no longer a reference
to the vector stored in `x`, but a copy of its first element, `"Hello"`. Now
that we don’t have a reference, our `push()` works just fine.

[move]: move-semantics.html

If we truly want a reference, we need the other option: ensure that our reference
goes out of scope before we try to do the mutation. That looks like this:

```rust
fn main() {
    let mut x = vec!["Hello", "world"];

    {
        let y = &x[0];
    }

    x.push("foo");
}
```

We created an inner scope with an additional set of curly braces. `y` will go out of
scope before we call `push()`, and so we’re all good.

This concept of ownership isn’t just good for preventing dangling pointers, but an
entire set of related problems, like iterator invalidation, concurrency, and more.
