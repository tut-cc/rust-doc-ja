% Hello, world!

Rust のインストールが終わったら、初めの Rust プログラムを書いてみましょう。
様々なプログラミング言語で初めのプログラムは伝統的に、
画面に “Hello, world!” と表示するものです。
このシンプルなプログラムから始めることは、
あなたのコンパイラーがインストールされただけでなく、
おそらく正しく動作することを確認するよい方法です。
また、画面に情報を表示することはよくあることです。

初めに私たちがすべきことはコードを入れておくファイルを作ることです。
私は `projects` ディレクトリをホームディレクトリに作って、
すべてのプロジェクトをそこに保存しています。
Rust はどこにあなたのコードがあるか管理してくれません。

This actually leads to one other concern we should address: this guide will
assume that you have basic familiarity with the command line.
Rust 自体は特定のエディターやコードの場所を要求しません。
もしあなたがIDEをコマンドラインより好むのであれば、
[SolidOak][solidoak] を参照するか、
あなたのお気に入りのIDEのプラグインを探すとよいでしょう。
コミュニティによって開発された様々なクオリティの拡張があります。
また、Rust チームは [plugins for various editors][plugins] で案内しています。
特に、エディタやIDEの設定はこのチュートリアルの範囲外なので、
ドキュメントを確認して設定してください。

[solidoak]: https://github.com/oakes/SolidOak
[plugins]: https://github.com/rust-lang/rust/blob/master/src/etc/CONFIGS.md

というわけで、プロジェクトのディレクトリの中にディレクトリを作りましょう。

```bash
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

もし Windows で、PowerShell を使っていなければ、
おそらく `~` はうまく動作しません。
シェルのドキュメントを調べて下さい。

次に、新しいソースファイルを作りましょう。
`main.rs` のようにします。
Rust ファイルは常に `.rs` という拡張子で終わります。
もし、2つ以上の単語をファイル名にするときは、
`helloworld.rs` とするよりは、アンダーバーを使って `hello_world.rs` としてください。

ファイルを開いたら、次のように入力してください。

```rust
fn main() {
    println!("Hello, world!");
}
```

ファイルを保存して、次のようにターミナルに入力してください。

```bash
$ rustc main.rs
$ ./main # or main.exe on Windows
Hello, world!
```

成功です! 何が起こったのか詳しく見てみましょう。

```rust
fn main() {

}
```

この行は Rust において *関数* を定義します。
`main` 関数は特別で、
すべての Rust プログラムは `main` 関数から始まります。
1行目は
「これは `main` という名前の関数で、引数を取らず、何も返しません。」
という意味です。
もし引数があれば括弧の中(`(`と`)`)に書きます。
そしてこの関数は何も返さないので、
返す型を省略出来ます。
これについては後で触れます。

You’ll also note that the function is wrapped in curly braces (`{` and `}`).
Rust requires these around all function bodies. It is also considered good
style to put the opening curly brace on the same line as the function
declaration, with one space in between.

Next up is this line:

```rust
    println!("Hello, world!");
```

This line does all of the work in our little program. There are a number of
details that are important here. The first is that it’s indented with four
spaces, not tabs. Please configure your editor of choice to insert four spaces
with the tab key. We provide some [sample configurations for various
editors][configs].

[configs]: https://github.com/rust-lang/rust/tree/master/src/etc/CONFIGS.md

The second point is the `println!()` part. This is calling a Rust [macro][macro],
which is how metaprogramming is done in Rust. If it were a function instead, it
would look like this: `println()`. For our purposes, we don’t need to worry
about this difference. Just know that sometimes, you’ll see a `!`, and that
means that you’re calling a macro instead of a normal function. Rust implements
`println!` as a macro rather than a function for good reasons, but that's an
advanced topic. One last thing to mention: Rust’s macros are significantly
different from C macros, if you’ve used those. Don’t be scared of using macros.
We’ll get to the details eventually, you’ll just have to trust us for now.

[macro]: macros.html

Next, `"Hello, world!"` is a ‘string’. Strings are a surprisingly complicated
topic in a systems programming language, and this is a ‘statically allocated’
string. If you want to read further about allocation, check out
[the stack and the heap][allocation], but you don’t need to right now if you
don’t want to. We pass this string as an argument to `println!`, which prints the
string to the screen. Easy enough!

[allocation]: the-stack-and-the-heap.html

Finally, the line ends with a semicolon (`;`). Rust is an ‘expression oriented’
language, which means that most things are expressions, rather than statements.
The `;` is used to indicate that this expression is over, and the next one is
ready to begin. Most lines of Rust code end with a `;`.

Finally, actually compiling and running our program. We can compile with our
compiler, `rustc`, by passing it the name of our source file:

```bash
$ rustc main.rs
```

This is similar to `gcc` or `clang`, if you come from a C or C++ background. Rust
will output a binary executable. You can see it with `ls`:

```bash
$ ls
main  main.rs
```

Or on Windows:

```bash
$ dir
main.exe  main.rs
```

There are now two files: our source code, with the `.rs` extension, and the
executable (`main.exe` on Windows, `main` everywhere else)

```bash
$ ./main  # or main.exe on Windows
```

This prints out our `Hello, world!` text to our terminal.

If you come from a dynamic language like Ruby, Python, or JavaScript,
you may not be used to these two steps being separate. Rust is an
‘ahead-of-time compiled language’, which means that you can compile a program,
give it to someone else, and they don't need to have Rust installed. If you
give someone a `.rb` or `.py` or `.js` file, they need to have a
Ruby/Python/JavaScript implementation installed, but you just need one command
to both compile and run your program. Everything is a tradeoff in language
design, and Rust has made its choice.

Congratulations! You have officially written a Rust program. That makes you a
Rust programmer! Welcome. 🎊🎉👍

Next, I'd like to introduce you to another tool, Cargo, which is used to write
real-world Rust programs. Just using `rustc` is nice for simple things, but as
your project grows, you'll want something to help you manage all of the options
that it has, and to make it easy to share your code with other people and
projects.
