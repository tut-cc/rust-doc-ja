% Cargoの導入

[Cargo][cratesio] はRustaceansがRustのプロジェクトを管理するために使用するツールです。

Cargoはプレリリースの段階で、現在も開発中です。しかしすでに実用段階にあり、Rustのプロジェクトは当初からCargoを使用することを想定しています。

[cratesio]: http://doc.crates.io

Cargoはコードのビルド、依存関係の解決とビルドを行います。

まずコードが依存関係を持たない場合、最初の機能のみ使用されます。
やがてそのコードは依存関係を持つようになるでしょう。しかしCargoを使用するとそれが容易になります。

もしあなたがRustを公式のインストーラでインストールしたならば、Cargoも同時にインストールされています。
もし他の方法でRustをインストールしたならば、[the Cargo README][cargoreadme] を参照してインストールしてください。

[cargoreadme]: https://github.com/rust-lang/cargo#installing-cargo-from-nightlies


##Cargoの適用


ではCargoをHello Worldに使用してみましょう。

Cargoをプロジェクトに適用するためには、2つのことを行う必要があります。
まず `Cargo.toml` という設定ファイルを作成し、次にソースファイルを適切な位置に保存する必要があります。

次のようにしてみましょう。

```bash
$ mkdir src
$ mv main.rs src/main.rs
```

ここで注意すべき点は、実際に実行するソースファイルは名前を `main.rs` とし、ライブラリを使用する場合はライブラリのファイル名を `lib.rs`とする点です。もし他のディレクトリ名を使用したい場合は、[こちら(`[[lib]]` or `[[bin]]`)][crates-custom]を参照してTOMLファイルを設定してください。

[crates-custom]: http://doc.crates.io/manifest.html#configuring-a-target

Cargoはソースファイルが`src`ディレクトリ下に置かれるものと想定しています。READMEファイルやライセンスに関するファイルなどの他のファイルは最上位のディレクトリに置き、ソースファイルと無関係なものとします。

Cargoはプロジェクトを綺麗で整然なままにしてくれます。
つまり、何物にも相応の場所があり、何物もその場所にあるべしということです。

次は設定ファイルを作成します。

```bash
$ editor Cargo.toml
```
ファイル名の頭文字には、大文字の`C`を使用することに注意します。

ファイル中には以下のように記述します。

```toml
[package]

name = "hello_world"
version = "0.0.1"
authors = [ "Your name <you@example.com>" ]
```

このファイルは [TOML][toml] 形式のファイルです。では、TOMLとは何か説明しましょう。
> TOMLは簡単な設定ファイルのためのフォーマットで、意味を容易に理解することができます。TOMLは、ハッシュテーブルに対して明確にマッピングを行う為に設計されました。
>TOMLはデータ構造を様々な言語で容易に解釈されるようにされるべきです。

TOMLは大変INIに似ていますが、いくつかの利点があります。

[toml]: https://github.com/toml-lang/toml

このファイルを作成したら準備完了です!以下のことを試してみてください。

```bash
$ cargo build
   Compiling hello_world v0.0.1 (file:///home/yourname/projects/hello_world)
$ ./target/debug/hello_world
Hello, world!
```

わぁ!`cargo build`と端末上で実行するだけでプロジェクトのビルドに成功しました。また、`./target/debug/hello_world` とすることで実行されます。このとき、`cargo run`としても実行することができます。

```bash
$ cargo run
     Running `target/debug/hello_world`
Hello, world!
```

いずれ、改めてビルドする必要がないことに気付くでしょう。Cargoはソースファイルに変更が無ければバイナリをそのまま実行し、変更があればビルドと実行の両方を以下のように実行してくれます。

```bash
$ cargo run
   Compiling hello_world v0.0.1 (file:///home/yourname/projects/hello_world)
     Running `target/debug/hello_world`
Hello, world!
```

`hello_world`では単純に`rustc`と実行することで問題はありませんが、将来的にプロジェクトが複雑になったとき、プロジェクト中の全てのソースに対してコンパイルを行う必要があります。Cargoを使用すれば、プロジェクトが複雑になっても`cargo build`と実行するだけで事足ります。

最終的にプロジェクトがリリースできる状態になったとき、`cargo build --release`としてコンパイルすることで最適化されてコンパイルされます。

また、Cargoによって `Cargo.lock` という新しいファイルが作られていることにも気付くでしょう。

```toml
[root]
name = "hello_world"
version = "0.0.1"
```

このファイルはCargoがあなたのアプリケーションの依存関係を記録するために使用されます。つまり、あなたがこのファイルのことを考える必要は全くありません。あなた自身がこのファイルを触れることはなく、Cargoのみがこのファイルの管理を行います。

とにかく!`hello_world`をCargoによってビルドすることに成功しました。単純なプログラムであるにも関わらず、多くの実用的なツールを使用しているので、Rustのよい経験になったでしょう。これらのことを用いてあなたは事実上あらゆるRustプロジェクトを始めることができるようになったと言えます。

```bash
$ git clone someurl.com/foo
$ cd foo
$ cargo build
```

## A New Project

新しいプロジェクトを始めたいのならば、いつもこれらの過程を通る必要はありません!
Cargoはあなたが作りたいプロジェクトの根幹となるディレクトリをすぐに作る機能があります。

Cargoを用いて新しくプロジェクトを始めるには、`cargo new`と実行します。

```bash
$ cargo new hello_world --bin
```

ここで`--bin`として実行しているのは、バイナリのプログラムを作ろうとしているためです。もしライブラリを作る場合は、これを省略します。

ではCargoがどのようにプロジェクトを生成したのか確認してみましょう。

```bash
$ cd hello_world
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

もし`tree`コマンドを使用できないならば、あなたのディストリビューションのパッケージマネージャから取得しましょう。必要不可欠というわけではありませんが、確実に役立つコマンドです。

プロジェクトの開始に必要なファイルが揃っています。まず、`Cargo.toml`を確認してみましょう。

```toml
[package]

name = "hello_world"
version = "0.0.1"
authors = ["Your Name <you@example.com>"]
```

Cargoはあなたが与える引数に応じた適切なTOMLファイルの作成と`git`の設定を行います。つまり、Cargoは`hello_world`ディレクトリを`git`のリポジトリとして初期化するのです。

`src/main.rs`の中身は以下のようになっています。

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargoは"Hello World!"のプログラムを生成してくれました。これでコーディングの準備が完了です!
Cargoの特徴その他詳細の説明は[guide][guide]に記載されています。

[guide]: http://doc.crates.io/guide.html

これでCargoについて習得することができました。では、Rustについて本格的に学習していきましょう。これらのことはあなたがこれからRustを扱う上で基本的なことです。

Rustを本格的に学習するために2つの方法を提案します。
実際のプロジェクトを始めるための‘[Learn Rust][learnrust]’、Rustの根幹から学習するための‘[Syntax and Semantics][syntax]’です。

熟練されたプログラマーには`Learn Rust`をおすすめします。Rustの動向に興味があるのならば`Syntax and Semantics`をおすすめします。

好みは人それぞれです。あなたが興味のある方を選択してください。

[learnrust]: learn-rust.html
[syntax]: syntax-and-semantics.html
