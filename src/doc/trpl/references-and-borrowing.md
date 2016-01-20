# 参照と借用

このガイドはRustの所有権システムに含まれる3要素の内の1つについて示したものです。これはRustの開発者が熟達すべきであり、かつRustの最もユニークで魅力的な機能の1つです。所有権はRustの最大の目的であるメモリ安全性を実現する方法です。以下の章でそれぞれ別個の概念について説明しています。


* キーコンセプトとなるのが[所有権][ownership]です
* 今あなたが読んでいるのが借用です
* 借用を進めた概念が[lifetimes][lifetimes]です

これら3つの章はその順序にも関係があります。所有権システムの完全な理解のためには3つ全ての読解が必要となるでしょう。

[ownership]: https://doc.rust-lang.org/book/ownership.html
[lifetimes]: https://doc.rust-lang.org/book/lifetimes.html

# メタ

詳細について見て行く前に、所有権システムについて2つ重要な注意事項があります。

Rustは安全性と速度に焦点を当てています。それら目的は、Rustにおいて動作のための抽象化コストを可能な限り抑えることを意味する'ゼロコストの抽象化'を多く行うことで達成しています。所有権システムはゼロコストの抽象化の典型的な例です。私たちがこのガイドで話す詳細の全ては_コンパイル時に行われます_。あなたはこれらの機能のために実行時コストを払うことはありません。

しかしながら、このシステムは確実にコストを要し、その影響は学習曲線に現れます。多くのRustの新規ユーザは自身が有効だと考えているプログラムのコンパイルをコンパイラに拒否されるという、所謂'ボローチェッカーとの戦い'を経験しています。これがしばしば起きるのはプログラマが頭の中で考える所有権がこのように動作するはずだというモデルとRustに実装された実際のルールが一致しないためです。あなたも恐らく初めは同じようなことを経験するでしょう。しかしながら良い知らせもあります。経験豊富なRustの開発者曰く、一度でもある程度の期間所有権システムのルールの下で作業すれば、借用チェッカーとの戦いは次第に減っていくとのことです。

このことを念頭において、所有権について学んでいきましょう。

# 借用

At the end of the [ownership][ownership] section, we had a nasty function that looked
like this:

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

This is not idiomatic Rust, however, as it doesn’t take advantage of borrowing. Here’s
the first step:

```rust
fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
    // do stuff with v1 and v2

    // return the answer
    42
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let answer = foo(&v1, &v2);

// we can use v1 and v2 here!
```

Instead of taking `Vec<i32>`s as our arguments, we take a reference:
`&Vec<i32>`. And instead of passing `v1` and `v2` directly, we pass `&v1` and
`&v2`. We call the `&T` type a ‘reference’, and rather than owning the resource,
it borrows ownership. A binding that borrows something does not deallocate the
resource when it goes out of scope. This means that after the call to `foo()`,
we can use our original bindings again.

References are immutable, just like bindings. This means that inside of `foo()`,
the vectors can’t be changed at all:

```rust,ignore
fn foo(v: &Vec<i32>) {
     v.push(5);
}

let v = vec![];

foo(&v);
```

errors with:

```text
error: cannot borrow immutable borrowed content `*v` as mutable
v.push(5);
^
```

Pushing a value mutates the vector, and so we aren’t allowed to do it.

# &mut references

There’s a second kind of reference: `&mut T`. A ‘mutable reference’ allows you
to mutate the resource you’re borrowing. For example:

```rust
let mut x = 5;
{
    let y = &mut x;
    *y += 1;
}
println!("{}", x);
```

This will print `6`. We make `y` a mutable reference to `x`, then add one to
the thing `y` points at. You’ll notice that `x` had to be marked `mut` as well,
if it wasn’t, we couldn’t take a mutable borrow to an immutable value.

Otherwise, `&mut` references are just like references. There _is_ a large
difference between the two, and how they interact, though. You can tell
something is fishy in the above example, because we need that extra scope, with
the `{` and `}`. If we remove them, we get an error:

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
note: previous borrow of `x` occurs here; the mutable borrow prevents
subsequent moves, borrows, or modification of `x` until the borrow ends
        let y = &mut x;
                     ^
note: previous borrow ends here
fn main() {

}
^
```

As it turns out, there are rules.

# The Rules

Here’s the rules about borrowing in Rust:

First, any borrow must last for a smaller scope than the owner. Second, you may
have one or the other of these two kinds of borrows, but not both at the same
time:

* 0 to N references (`&T`) to a resource.
* exactly one mutable reference (`&mut T`)


You may notice that this is very similar, though not exactly the same as,
to the definition of a data race:

> There is a ‘data race’ when two or more pointers access the same memory
> location at the same time, where at least one of them is writing, and the
> operations are not synchronized.

With references, you may have as many as you’d like, since none of them are
writing. If you are writing, you need two or more pointers to the same memory,
and you can only have one `&mut` at a time. This is how Rust prevents data
races at compile time: we’ll get errors if we break the rules.

With this in mind, let’s consider our example again.

## Thinking in scopes

Here’s the code:

```rust,ignore
let mut x = 5;
let y = &mut x;

*y += 1;

println!("{}", x);
```

This code gives us this error:

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
```

This is because we’ve violated the rules: we have a `&mut T` pointing to `x`,
and so we aren’t allowed to create any `&T`s. One or the other. The note
hints at how to think about this problem:

```text
note: previous borrow ends here
fn main() {

}
^
```

In other words, the mutable borow is held through the rest of our example. What
we want is for the mutable borrow to end _before_ we try to call `println!` and
make an immutable borrow. In Rust, borrowing is tied to the scope that the
borrow is valid for. And our scopes look like this:

```rust,ignore
let mut x = 5;

let y = &mut x;    // -+ &mut borrow of x starts here
                   //  |
*y += 1;           //  |
                   //  |
println!("{}", x); // -+ - try to borrow x here
                   // -+ &mut borrow of x ends here
```

The scopes conflict: we can’t make an `&x` while `y` is in scope.

So when we add the curly braces:

```rust
let mut x = 5;

{
    let y = &mut x; // -+ &mut borrow starts here
    *y += 1;        //  |
}                   // -+ ... and ends here

println!("{}", x);  // <- try to borrow x here
```

There’s no problem. Our mutable borrow goes out of scope before we create an
immutable one. But scope is the key to seeing how long a borrow lasts for.

## Issues borrowing prevents

Why have these restrictive rules? Well, as we noted, these rules prevent data
races. What kinds of issues do data races cause? Here’s a few.

### Iterator invalidation

One example is ‘iterator invalidation’, which happens when you try to mutate a
collection that you’re iterating over. Rust’s borrow checker prevents this from
happening:

```rust
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
}
```

This prints out one through three. As we iterate through the vectors, we’re
only given references to the elements. And `v` is itself borrowed as immutable,
which means we can’t change it while we’re iterating:

```rust,ignore
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
    v.push(34);
}
```

Here’s the error:

```text
error: cannot borrow `v` as mutable because it is also borrowed as immutable
    v.push(34);
    ^
note: previous borrow of `v` occurs here; the immutable borrow prevents
subsequent moves or mutable borrows of `v` until the borrow ends
for i in &v {
          ^
note: previous borrow ends here
for i in &v {
    println!(“{}”, i);
    v.push(34);
}
^
```

We can’t modify `v` because it’s borrowed by the loop.

### use after free

References must live as long as the resource they refer to. Rust will check the
scopes of your references to ensure that this is true.

If Rust didn’t check that this property, we could accidentally use a reference
which was invalid. For example:

```rust,ignore
let y: &i32;
{
    let x = 5;
    y = &x;
}

println!("{}", y);
```

We get this error:

```text
error: `x` does not live long enough
    y = &x;
         ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
let y: &i32;
{
    let x = 5;
    y = &x;
}

note: ...but borrowed value is only valid for the block suffix following
statement 0 at 4:18
    let x = 5;
    y = &x;
}
```

In other words, `y` is only valid for the scope where `x` exists. As soon as
`x` goes away, it becomes invalid to refer to it. As such, the error says that
the borrow ‘doesn’t live long enough’ because it’s not valid for the right
amount of time.

The same problem occurs when the reference is declared _before_ the variable it refers to:

```rust,ignore
let y: &i32;
let x = 5;
y = &x;

println!("{}", y);
```

We get this error:

```text
error: `x` does not live long enough
y = &x;
     ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
    let y: &i32;
    let x = 5;
    y = &x;

    println!("{}", y);
}

note: ...but borrowed value is only valid for the block suffix following
statement 1 at 3:14
    let x = 5;
    y = &x;

    println!("{}", y);
}
```
