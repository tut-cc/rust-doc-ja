% for Loops

以前は， `for` ループは特定の回数の繰り返しに使われていました．
しかし Rust の `for` ループは，他のシステム言語とは少し違った動きをします．
Rust の `for` ループは以下のような「 C スタイル」の `for` ループではないのです．

```c
for (x = 0; x < 10; x++) {
    printf( "%d\n", x );
}
```

代わりに，このようにします．

```rust
for x in 0..10 {
    println!("{}", x); // x: i32
}
```

より抽象的な語で表すと次のようになります．

```ignore
for 変数 in 式 {
    コード
}
```

The expression is an [iterator][iterator]. The iterator gives back a series of
elements. Each element is one iteration of the loop. That value is then bound
to the name `var`, which is valid for the loop body. Once the body is over, the
next value is fetched from the iterator, and we loop another time. When there
are no more values, the `for` loop is over.

[iterator]: iterators.html

In our example, `0..10` is an expression that takes a start and an end position,
and gives an iterator over those values. The upper bound is exclusive, though,
so our loop will print `0` through `9`, not `10`.

Rust does not have the “C-style” `for` loop on purpose. Manually controlling
each element of the loop is complicated and error prone, even for experienced C
developers.
