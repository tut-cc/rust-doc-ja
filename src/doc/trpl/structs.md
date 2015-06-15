% 構造体

構造体はより複雑なデータ型を作る方法の1つです。例えば、もし私たちが2次元空間の座標に関する計算を行っているとして、`x`と`y`、両方の値が必要になるでしょう。

```rust
let origin_x = 0;
let origin_y = 0;
```

構造体はこれら2つを1つに結びつけ、データ型を定義することができます。

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let origin = Point { x: 0, y: 0 }; // origin: Point

    println!("The origin is at ({}, {})", origin.x, origin.y);
}
```
ここでは多くのことが起きていますから、分解して見て行きましょう。私たちは`struct`キーワードを使って構造体とその名前を宣言しています。慣習により、構造体は初めが大文字のキャメルケース、いわゆる`UpperCaml`で記述しています: `PointInSpace`であり、`Point_In_Space`ではありません。

いつものように、`let`で構造体のインスタンスを作ることができますが、私たちは`key: value`スタイルの構文でそれぞれのフィールドに値をセットしています。順序は元の宣言と同じである必要はありません。

最終的にフィールドは名前を持つため、ドット表記でアクセスできます: `origin.x`

構造体の値はRustの他の束縛のようにイミュータブルが基本です。`mut`を使うと値をミュータブルにできます。

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let mut point = Point { x: 0, y: 0 };

    point.x = 5;

    println!("The point is at ({}, {})", point.x, point.y);
}
```

これは`The point is at (5, 0)`と出力されます。Rustは言語レベルでフィールドのミュータブル化に対応していないため、このように書くことはできません。

```rust,ignore
struct Point {
    mut x: i32,
    y: i32,
}
```

ミュータブルは束縛に付与できる属性であり、構造体自体に付与できる属性ではありません。もしあなたがフィールドレベルのミュータブルを使うのであれば、最初はこれは奇妙に見えるかもしれませんが、非常に簡単に実現できる方法があります。以下の方法で少しの間だけミュータブルな構造体を作ることができます。

```rust,ignore
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let mut point = Point { x: 0, y: 0 };

    point.x = 5;

    let point = point; // この新しい束縛で変更不可にします

    point.y = 6; // これが原因でエラーが起きます
}
```

# アップデートシンタックス

構造体の初期化には、値の一部について、他の構造体からのコピーを使用したいことを示す`..`を含めることができます。例えばこのようになります。

```rust
struct Point3d {
    x: i32,
    y: i32,
    z: i32,
}

let mut point = Point3d { x: 0, y: 0, z: 0 };
point = Point3d { y: 1, .. point };
```

ここでは`point`に新しい`y`を与えていますが、`x`と`z`は古い値を維持します。どちらかと同じ構造をする必要はなく、あなたはこの構文を新しい構造体を作る際に使うことができ、明示することなく維持したい値のコピーが行われます。

```rust

# struct Point3d {
#     x: i32,
#     y: i32,
#     z: i32,
# }
let origin = Point3d { x: 0, y: 0, z: 0 };
let point = Point3d { z: 1, x: 2, .. origin };
```

# タプル構造体

Rustには構造体と[タプル][tuple]のハイブリットである'タプル構造体'と呼ばれるものがあります。タプル構造体は名前を持ちますがそのフィールドには名前がありません。

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
```

[tuple]: primitive-types.html#tuples

これら2つは同じ値同士であったとしても等しくありません。

```rust
# struct Color(i32, i32, i32);
# struct Point(i32, i32, i32);
let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

ほとんどの場合タプル構造体よりも構造体を使ったほうが良いです。`Color`や`Point`はこうに書けます。

```rust
struct Color {
    red: i32,
    blue: i32,
    green: i32,
}

struct Point {
    x: i32,
    y: i32,
    z: i32,
}
```

今、私たちはフィールドの位置ではなくフィールド自体の名前を持っています。良い名前は重要で、構造体を使うということは、私たちは名前を持っているということです。

タプル構造体が非常に便利な場合も_あります_が、1要素で使う場合だけです。タプル構造体の中に入っている値や式と、それ自身のセマンティクスを明確に区別できるような新しい型を作成できることから、私たちはこれを'newtype'パターンと呼んでいます:

```rust
struct Inches(i32);

let length = Inches(10);

let Inches(integer_length) = length;
println!("length is {} inches", integer_length);
```

ここで見て取れるように、あなたは`let`を使った分解により、標準のタプルと同じように内部の整数型を取り出すことができます。
このケースでは`let Inches(integer_length)`が`integer_length`へ`10`を束縛します。

# Unit-like 構造体

あなたは全くメンバーを持たない構造体を定義できます。

```rust
struct Electron;
```

空のタプルである`()`は時々`unit`と呼ばれ、それに似ていることからこのような構造体は`unit-like`と呼ばれています。タプル構造体のように、これは新しい型を定義します。

これは単体でも極稀に役立ちます(もっとも、時々型をマーク代わりとして役立てる程度です)が、他の機能と組み合わせることで便利になります。例えば、ライブラリはあなたにイベントを必ず処理できる[トレイト][trait]が実装されたデータ構造の作成を要求するかもしれません。もしそのデータ構造の中に保存すべき値が何もなければ、あなたはただunit-like構造体を作ることができます。
