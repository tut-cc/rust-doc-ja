% イテレータ

ループの話をしましょう。

Rustの`for`ループを覚えていますか？
こちらが例です:
```rust
for x in 0..10 {
    println!("{}", x);
}
```
今やあなたはRustに詳しいですから、私達はどのようにこの文が動いているのか詳しく話せます。
Ranges(ここでは`0..10`)は'イテレータ'です。イテレータは繰り返し`.next()`メソッドを呼び出すことができ、データの連なりを与えてくれます。

こんな風に書いてみます:

```rust
let mut range = 0..10;

loop {
    match range.next() {
        Some(x) => {
            println!("{}", x);
        },
        None => { break }
    }
}
```

rangeにイテレータをmutable束縛しています。`loop`の中の`match`は、イテレータである`range.next()`を呼び出すことで返ってくる次の値への参照を使用しています。`next`には`Option<i32>`が返ってきます。このケースでは、イテレータの次の値が返ってくればその値は`Some(i32)`であり、イテレータが尽きれば`None`が返ってきます。もし`Some(i32)`であればそれを表示し、`None`であれば`break`によりループから脱出しています。

このコードは、基本的に`for`ループバージョンと同じ動作です。`for`ループはこの`loop`/`match`/`break`で構成された処理を手軽に書ける方法というわけです。

しかしながら、`for`ループはイテレータのみに使うものではありません。`Iterator`トレイトを実装し組み込むことで、あなた自作のイテレータを書くこともできます。このガイドの範囲外ですが、Rustは多様な反復処理を実現するために便利なイテレータを幾つか提供しています。ただ、それらについて話す前に、Rustのアンチパターンについて話しておかなければなりません。しかもアンチパターンというのは、先ほど書いたようにrangesを使うことなのです。

ええ、確かに私たちはたった今rangesが如何にクールなのか話しました。しかしrangesは古典的過ぎます。例えば、もしあなたがvectorの中身を繰り返し処理したいとき、こんな風に書きたくなるかもしれません:

```rust
let nums = vec![1, 2, 3];

for i in 0..nums.len() {
    println!("{}", nums[i]);
}
```

これは実際のイテレータの使い方からすれば全く正しくありません。あなたはvectorを直接反復処理できるのですから、こう書くべきです:

```rust
let nums = vec![1, 2, 3];

for num in &nums {
    println!("{}", num);
}
```

これには2つの理由があります。第一に、このほうが書き手の意味するところがはっきり表現できます。私たちはvectorのインデックスを作成してからその要素を繰り返し参照したいのではなく、vector自体を反復処理したいのです。第二に、このバージョンのほうがより効率的です: 1つ目の例では`num[i]`というようにインデックスを使っているため、余計な境界チェックが発生します。しかし、イテレータが順番にvectorの各要素の参照を生成していくため、2つ目の例では境界チェックが発生しません。これはイテレータにとってごく一般的な性質です: 私たちは不要な境界チェックを無視してもなお、安全であることを知っています。

ここには`println!`の動作という、100%はっきりしていないもう1つの詳細があります。`num`は実際には`&i32`型です。これは`i32`の参照であり、`i32`それ自体ではありません。`println!`は私たちがそれを理解していなくとも、参照からうまく値を取り出してくれます。このコードも正しく動作します:

```rust
let nums = vec![1, 2, 3];

for num in &nums {
    println!("{}", *num);
}
```

今、私たちは明示的に`num`から参照を取り出しました。なぜ`&nums`は私たちに参照を渡すのでしょうか？第一に、`&`を用いて私たちが明示的に要求したからです。第二に、もしデータそれ自体を私たちに渡す場合、私たちはデータの所有者でなければならないため、データの複製と、それを私たちに渡す操作が伴います。参照を使えば、データの参照を借用し、参照を通すだけで、ムーブを行う必要がなくなります。

そういうわけで、今やrangesはあまりあなたが欲しいものではなくなりました。それではあなたが代わりに欲しいと思うものについて話しましょう。

それは大きく分けて3つあり、これらに関係あるものです: イテレータ、*イテレータアダプタ*、そして*コンシューマ*。定義はこちら:

* *イテレータ*はあなたに値の列を渡します。
* *イテレータアダプタ*はイテレータ上で動作し、出力の異なるイテレータを生成します。
* *コンシューマ*はイテレータ上で動作し、幾つかの最終的な値の組を返します。

あなたは既にイテレータとrangesを見てきたのですから、最初にコンシューマについて話しましょう。

## コンシューマ

A *consumer* operates on an iterator, returning some kind of value or values.
The most common consumer is `collect()`. This code doesn't quite compile,
but it shows the intention:

```{rust,ignore}
let one_to_one_hundred = (1..101).collect();
```

As you can see, we call `collect()` on our iterator. `collect()` takes
as many values as the iterator will give it, and returns a collection
of the results. So why won't this compile? Rust can't determine what
type of things you want to collect, and so you need to let it know.
Here's the version that does compile:

```rust
let one_to_one_hundred = (1..101).collect::<Vec<i32>>();
```

If you remember, the `::<>` syntax allows us to give a type hint,
and so we tell it that we want a vector of integers. You don't always
need to use the whole type, though. Using a `_` will let you provide
a partial hint:

```rust
let one_to_one_hundred = (1..101).collect::<Vec<_>>();
```

This says "Collect into a `Vec<T>`, please, but infer what the `T` is for me."
`_` is sometimes called a "type placeholder" for this reason.

`collect()` is the most common consumer, but there are others too. `find()`
is one:

```rust
let greater_than_forty_two = (0..100)
                             .find(|x| *x > 42);

match greater_than_forty_two {
    Some(_) => println!("We got some numbers!"),
    None => println!("No numbers found :("),
}
```

`find` takes a closure, and works on a reference to each element of an
iterator. This closure returns `true` if the element is the element we're
looking for, and `false` otherwise. Because we might not find a matching
element, `find` returns an `Option` rather than the element itself.

Another important consumer is `fold`. Here's what it looks like:

```rust
let sum = (1..4).fold(0, |sum, x| sum + x);
```

`fold()` is a consumer that looks like this:
`fold(base, |accumulator, element| ...)`. It takes two arguments: the first
is an element called the *base*. The second is a closure that itself takes two
arguments: the first is called the *accumulator*, and the second is an
*element*. Upon each iteration, the closure is called, and the result is the
value of the accumulator on the next iteration. On the first iteration, the
base is the value of the accumulator.

Okay, that's a bit confusing. Let's examine the values of all of these things
in this iterator:

| base | accumulator | element | closure result |
|------|-------------|---------|----------------|
| 0    | 0           | 1       | 1              |
| 0    | 1           | 2       | 3              |
| 0    | 3           | 3       | 6              |

We called `fold()` with these arguments:

```rust
# (1..4)
.fold(0, |sum, x| sum + x);
```

So, `0` is our base, `sum` is our accumulator, and `x` is our element.  On the
first iteration, we set `sum` to `0`, and `x` is the first element of `nums`,
`1`. We then add `sum` and `x`, which gives us `0 + 1 = 1`. On the second
iteration, that value becomes our accumulator, `sum`, and the element is
the second element of the array, `2`. `1 + 2 = 3`, and so that becomes
the value of the accumulator for the last iteration. On that iteration,
`x` is the last element, `3`, and `3 + 3 = 6`, which is our final
result for our sum. `1 + 2 + 3 = 6`, and that's the result we got.

Whew. `fold` can be a bit strange the first few times you see it, but once it
clicks, you can use it all over the place. Any time you have a list of things,
and you want a single result, `fold` is appropriate.

Consumers are important due to one additional property of iterators we haven't
talked about yet: laziness. Let's talk some more about iterators, and you'll
see why consumers matter.

## Iterators

As we've said before, an iterator is something that we can call the
`.next()` method on repeatedly, and it gives us a sequence of things.
Because you need to call the method, this means that iterators
can be *lazy* and not generate all of the values upfront. This code,
for example, does not actually generate the numbers `1-100`, instead
creating a value that merely represents the sequence:

```rust
let nums = 1..100;
```

Since we didn't do anything with the range, it didn't generate the sequence.
Let's add the consumer:

```rust
let nums = (1..100).collect::<Vec<i32>>();
```

Now, `collect()` will require that the range gives it some numbers, and so
it will do the work of generating the sequence.

Ranges are one of two basic iterators that you'll see. The other is `iter()`.
`iter()` can turn a vector into a simple iterator that gives you each element
in turn:

```rust
let nums = vec![1, 2, 3];

for num in nums.iter() {
   println!("{}", num);
}
```

These two basic iterators should serve you well. There are some more
advanced iterators, including ones that are infinite.

That's enough about iterators. Iterator adapters are the last concept
we need to talk about with regards to iterators. Let's get to it!

## Iterator adapters

*Iterator adapters* take an iterator and modify it somehow, producing
a new iterator. The simplest one is called `map`:

```{rust,ignore}
(1..100).map(|x| x + 1);
```

`map` is called upon another iterator, and produces a new iterator where each
element reference has the closure it's been given as an argument called on it.
So this would give us the numbers from `2-100`. Well, almost! If you
compile the example, you'll get a warning:

```text
warning: unused result which must be used: iterator adaptors are lazy and
         do nothing unless consumed, #[warn(unused_must_use)] on by default
(1..100).map(|x| x + 1);
 ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Laziness strikes again! That closure will never execute. This example
doesn't print any numbers:

```{rust,ignore}
(1..100).map(|x| println!("{}", x));
```

If you are trying to execute a closure on an iterator for its side effects,
just use `for` instead.

There are tons of interesting iterator adapters. `take(n)` will return an
iterator over the next `n` elements of the original iterator. Note that this
has no side effect on the original iterator. Let's try it out with our infinite
iterator from before:

```rust
# #![feature(step_by)]
for i in (1..).step_by(5).take(5) {
    println!("{}", i);
}
```

This will print

```text
1
6
11
16
21
```

`filter()` is an adapter that takes a closure as an argument. This closure
returns `true` or `false`. The new iterator `filter()` produces
only the elements that that closure returns `true` for:

```rust
for i in (1..100).filter(|&x| x % 2 == 0) {
    println!("{}", i);
}
```

This will print all of the even numbers between one and a hundred.
(Note that because `filter` doesn't consume the elements that are
being iterated over, it is passed a reference to each element, and
thus the filter predicate uses the `&x` pattern to extract the integer
itself.)

You can chain all three things together: start with an iterator, adapt it
a few times, and then consume the result. Check it out:

```rust
(1..1000)
    .filter(|&x| x % 2 == 0)
    .filter(|&x| x % 3 == 0)
    .take(5)
    .collect::<Vec<i32>>();
```

This will give you a vector containing `6`, `12`, `18`, `24`, and `30`.

This is just a small taste of what iterators, iterator adapters, and consumers
can help you with. There are a number of really useful iterators, and you can
write your own as well. Iterators provide a safe, efficient way to manipulate
all kinds of lists. They're a little unusual at first, but if you play with
them, you'll get hooked. For a full list of the different iterators and
consumers, check out the [iterator module documentation](../std/iter/index.html).
