% Vectors

A ‘vector’ is a dynamic or ‘growable’ array, implemented as the standard
library type [`Vec<T>`][vec].
 ‘vector’ は動的で ‘growable’ な配列です。標準ライブラリの  [`Vec<T>`][vec] という方を満たします。
The `T` means that we can have vectors
of any type (see the chapter on [generics][generic] for more).
 `T` というのはベクターはどんな型でも保持出来るという事を意味しています(詳しくは [generics][generic] の章を読んで下さい)。
Vectors always allocate their data on the heap.
ベクターはいつでもそれらのデータをヒープ領域に保持します。
 `vec!` マクロでベクターを作成する事が出来ます。

```rust
let v = vec![1, 2, 3, 4, 5]; // v: Vec<i32>
```

(Notice that unlike the `println!` macro we’ve used in the past, we use square
brackets `[]` with `vec!` macro. Rust allows you to use either in either situation,
this is just convention.)

There’s an alternate form of `vec!` for repeating an initial value:

```
let v = vec![0; 10]; // ten zeroes
```

## Accessing elements

To get the value at a particular index in the vector, we use `[]`s:

```rust
let v = vec![1, 2, 3, 4, 5];

println!("The third element of v is {}", v[2]);
```

The indices count from `0`, so the third element is `v[2]`.

## Iterating

Once you have a vector, you can iterate through its elements with `for`. There
are three versions:

```rust
let mut v = vec![1, 2, 3, 4, 5];

for i in &v {
    println!("A reference to {}", i);
}

for i in &mut v {
    println!("A mutable reference to {}", i);
}

for i in v {
    println!("Take ownership of the vector and its element {}", i);
}
```

Vectors have many more useful methods, which you can read about in [their
API documentation][vec].

[vec]: ../std/vec/index.html
[generic]: generics.html
