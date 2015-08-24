% トレイトオブジェクト

コードがポリモーフィズムを伴う場合、実際に実行される対象を特定するためのメカニズムが必要です。
これはディスパッチと呼ばれます。ディスパッチには静的ディスパッチと動的ディスパッチという2つの形式があります。Rustは静的ディスパッチを支持していますが、'トレイトオブジェクト'と呼ばれるメカニズムを用いることで動的ディスパッチについても対応しています。

## 背景

後に必要となるので、ここでトレイトと幾つかの実装を示しておきます。ここでは単純に`Foo`としましょう。これは`String`型の値を返す関数を1つ持っています。

```rust
trait Foo {
    fn method(&self) -> String;
}
```

このトレイトを`u8`と`String`のために実装します。

```rust
# trait Foo { fn method(&self) -> String; }
impl Foo for u8 {
    fn method(&self) -> String { format!("u8: {}", *self) }
}

impl Foo for String {
    fn method(&self) -> String { format!("string: {}", *self) }
}
```


## 静的ディスパッチ
トレイト束縛による静的ディスパッチを実現するために、このトレイトを使うことができます。

```rust
# trait Foo { fn method(&self) -> String; }
# impl Foo for u8 { fn method(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn method(&self) -> String { format!("string: {}", *self) } }
fn do_something<T: Foo>(x: T) {
    x.method();
}

fn main() {
    let x = 5u8;
    let y = "Hello".to_string();

    do_something(x);
    do_something(y);
}
```

Rustはここで静的ディスパッチを実現するために、'モノモーフィゼーション(monomorphization)'を用います。これはRustが`u8`専用、`String`専用の`do_something()`をそれぞれ作成し、その専用の関数を宛がうように呼び出しの部分を書き換えるという意味です。言い換えれば、Rustはこのようなコードを生成します。

```rust
# trait Foo { fn method(&self) -> String; }
# impl Foo for u8 { fn method(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn method(&self) -> String { format!("string: {}", *self) } }
fn do_something_u8(x: u8) {
    x.method();
}

fn do_something_string(x: String) {
    x.method();
}

fn main() {
    let x = 5u8;
    let y = "Hello".to_string();

    do_something_u8(x);
    do_something_string(y);
}
```

これは素晴らしい利点です。呼び出し先はコンパイル時に分かっているため、静的ディスパッチは関数呼び出しをインライン化することができます。インライン化は優れた最適化の鍵です。静的ディスパッチは高速ですが、トレードオフとしてそれぞれの型ごとに同じ関数を多くコピーするため、バイナリサイズが膨張してしまいます。これは'code bloat'[^code_bloat]と呼ばれています。

その上、コンパイラは完璧ではなく、"最適化"したコードが遅くなってしまうこともあります。
例えば、あまりにも熱心な関数のインライン化は、命令キャッシュ(キャッシュはマシンの環境全てを支配しています)を膨張させてしまいます。それが`#[inline]`や`#[inline(always)]`を慎重に使うべきである理由の1つであり、動的ディスパッチが度々静的ディスパッチよりも効率的である理由の1つなのです。

しかしながら、一般的なケースでは静的ディスパッチを使用する方が効率的であり、また、動的ディスパッチを行う薄い静的ディスパッチラッパー関数を持つことは常に可能ですが、その逆はできません。これは静的ディスパッチの方が柔軟性に富むことを示唆しています。標準ライブラリは上記の理由から可能であれば静的ディスパッチを試みます。

## 動的ディスパッチ

Rustは'トレイトオブジェクト'と呼ばれる機能によって動的ディスパッチを提供しています。トレイトオブジェクトは`&Foo`か`Box<Foo>`[^Box]の様に記述され、実行時に型が判明する特定のトレイトを実装している*あらゆる*型の値を格納します。

トレイトオブジェクトはトレイトを実装した具体的な型のポインタから*キャスト*する(e.g. `&x as &Foo`)か、*型変換*する(e.g. `&x`を関数の引数で`&Foo`として受け取る)ことで取得できます。

これらトレイトオブジェクトの型変換とキャストは`&mut T`から`&mut Foo`へ、`Box<T>`から`Box<Foo>`へ、というようにどちらもポインタに対する操作ですが、今はこれがトレイトオブジェクトの全てです。なお型変換とキャストは実質同一です。

この操作がまるで特定のポインタの型を消去しているように見えることから、トレイトオブジェクトは時に'type erasure(型消去)'とも呼ばれます。

上記の例に戻ってみると、私たちはトレイトオブジェクトを用いた動的ディスパッチ実現のために、キャストによって同一のトレイトを使用することができます。

```rust
# trait Foo { fn method(&self) -> String; }
# impl Foo for u8 { fn method(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn method(&self) -> String { format!("string: {}", *self) } }

fn do_something(x: &Foo) {
    x.method();
}

fn main() {
    let x = 5u8;
    do_something(&x as &Foo);
}
```

型変換を用いると、

```rust
# trait Foo { fn method(&self) -> String; }
# impl Foo for u8 { fn method(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn method(&self) -> String { format!("string: {}", *self) } }

fn do_something(x: &Foo) {
    x.method();
}

fn main() {
    let x = "Hello".to_string();
    do_something(&x);
}
```
トレイトオブジェクトを受け取った関数が`Foo`を実装した特定の型毎に特殊化されることはありません。関数は1つだけ生成され、多くの場合(とはいえ常にではありませんが)code bloatは少なく済みます。しかしながら、この操作は低速な仮想関数の呼び出しを必要とし、更にインライン化と関連する最適化の機会を阻害するといったコストが伴います。

### なぜポインタなのか？

(訳注: 恐らく第1パラグラフで静的ディスパッチが有効な場合について話し、続いて動的ディスパッチの場合の話をしています)
Rustはガーベジコレクタによって管理される多くの言語とは異なり、デフォルトではポインタの参照先に値を配置するようなことはしませんが、それぞれの型の値が異なるサイズを持つことができます。コンパイル時に値のサイズを知っていることは、関数へ引数として渡されるような値を保持するためにスタックに移したり、ヒープ上にメモリをアロケートしたり(または開放したり)するために重要です。

`Foo`のために、私たちは`String`(24 bytes)、`u8`(1 byte)、もしくは依存しているクレイト(訳注:クレイトは他の言語では'ライブラリ'や'パッケージ'とも呼ばれるものです)のうち(全く中身の無い)`Foo`を実装しているいずれかの型の値を格納する必要があります。ポインタ無しで値を格納した場合、その直後の動作が正しいかどうかを保証する方法がありません。型によって値のサイズがそれぞれ異なるからです。

トレイトオブジェクトを渡したとき、渡されるのはポインタのサイズのみです。ポインタの参照先に値を配置するということは、変数が値自体のサイズに依存しなくなるということであり、これによって1つの変数に異なる型の値を格納できるようになります。

### トレイトオブジェクトの再現

トレイトの関数は、伝統的に'vtable'(これはコンパイラによって作成、管理されます)と呼ばれる特別な関数ポインタのレコードを介してトレイトオブジェクトから呼び出すことができます。

トレイトオブジェクトは単純ですが難解です。核となる表現とレイアウトは非常に単純ですが、複雑なエラーメッセージを吐いたり、予期せぬ振る舞いが見つかったりします。

単純な例として、トレイトオブジェクトの実行時の再現から始めましょう。`std::raw`モジュールは複雑なビルドインの型と同じレイアウトの構造体を格納しており、[トレイトオブジェクトも含まれています][stdraw]。

```rust
# mod foo {
pub struct TraitObject {
    pub data: *mut (),
    pub vtable: *mut (),
}
# }
```

[stdraw]: ../std/raw/struct.TraitObject.html

つまり、`&Foo`というトレイトオブジェクトは'data'ポインタと'vtable'ポインタから成るわけです。

dataポインタはトレイトオブジェクトに格納されている(幾つか考えられる不明瞭な型`T`の)データにアドレッシングされており、vtableポインタは`T`のための`Foo`の実装に対応しているvtable('virtual method table')を指しています。

vtableは本質的には関数ポインタの構造体で、各ポインタはそれぞれのメソッドの実装の機械語を指しています。`trait_object.method()`のようなメソッド呼び出しを行うと、vtableの中から適切なポインタを取り出し、動的に呼び出しを行います。例えば、

```rust,ignore
struct FooVtable {
    destructor: fn(*mut ()),
    size: usize,
    align: usize,
    method: fn(*const ()) -> String,
}

// u8:

fn call_method_on_u8(x: *const ()) -> String {
    // `x`がu8を指しているとき、コンパイラはこの関数だけが呼ばれることを保証します
    let byte: &u8 = unsafe { &*(x as *const u8) };

    byte.method()
}

static Foo_for_u8_vtable: FooVtable = FooVtable {
    destructor: /* コンパイラマジック */,
    size: 1,
    align: 1,

    // 関数ポインタへキャスト
    method: call_method_on_u8 as fn(*const ()) -> String,
};


// String:

fn call_method_on_String(x: *const ()) -> String {
    // `x`がStringを指しているとき、コンパイラはこの関数だけが呼ばれることを保証します
    let string: &String = unsafe { &*(x as *const String) };

    string.method()
}

static Foo_for_String_vtable: FooVtable = FooVtable {
    destructor: /* コンパイラマジック */,
    // 64-bitコンピュータのための値を32-bitコンピュータのために半分にしておきます
    size: 24,
    align: 8,

    method: call_method_on_String as fn(*const ()) -> String,
};
```

関数を指している各vtableに存在する`destructor`フィールドは、それぞれのvtableの型に応じてリソースの解放を行います。ここでは`u8`は単純な型なので何もしませんが、`String`はメモリの解放を行います。このフィールドはスコープから出た時の値や、`Box<Foo>`のように`Box`によってアロケートした自作トレイトオブジェクトのリソースを解放するのに必要です。`size`及び`align`フィールドは消去された型のサイズとアライメント要件です。この2つの情報はデストラクタに組み込まれているため、現時点では基本的に使われていませんが、将来的にトレイトオブジェクトがより柔軟になれば使われるようになるでしょう。

仮に私たちが`Foo`を実装した値を幾つか持っているとしましょう。以下の明示的な形式で値を生成する方法と、これまでに紹介した`Foo`のトレイトオブジェクトを使う方法は少しだけ似ているかもしれません。(とにかく全てポインタにすることで型の不一致を無視しています)

```rust,ignore
let a: String = "foo".to_string();
let x: u8 = 1;

// let b: &Foo = &a;
let b = TraitObject {
    // データを保存
    data: &a,
    // メソッドを保存
    vtable: &Foo_for_String_vtable
};

// let y: &Foo = x;
let y = TraitObject {
    // データを保存
    data: &x,
    // メソッドを保存
    vtable: &Foo_for_u8_vtable
};

// b.method();
(b.vtable.method)(b.data);

// y.method();
(y.vtable.method)(y.data);
```

---

[^code_bloat] 訳注: [Wikipediaの記事](https://en.wikipedia.org/wiki/Code_bloat)もあります

[^Box] 訳注: Boxはヒープを指すポインタの型です、[モジュールはこちら](https://doc.rust-lang.org/std/boxed/index.html)
