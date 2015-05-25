% Rust のインストール

ここではまず最初のステップとして Rust のインストールを行います!
 Rust をインストールするには幾つかの方法が存在しますが、ここでは最も簡単な `rustip` というスクリプトを使用します。
もし Linux か Mac の環境ならば、次のことをすれば良いだけです($を入力しないように注意して下さい．これはコマンドの開始を示しています)。

```bash
$ curl -sf -L https://static.rust-lang.org/rustup.sh | sh
```

もしこのコマンドを実行することが不安なら [potential insecurity][insecurity]の `curl
| sh` を読んでみて下さい。そして以下の私達の免責事項を見て下さい。
そして気楽に次の二段階のバージョンのインストールを使用して，インストールスクリプトを調べてみて下さい。

```bash
$ curl -f -L https://static.rust-lang.org/rustup.sh -O
$ sh rustup.sh
```

[insecurity]: http://curlpipesh.tumblr.com

もし貴方が Windows なら、[32-bit installer][win32] か [64-bit installer][win64] をダウンロードして実行して下さい。

[win32]: https://static.rust-lang.org/dist/rust-1.0.0-beta-i686-pc-windows-gnu.msi
[win64]: https://static.rust-lang.org/dist/rust-1.0.0-beta-x86_64-pc-windows-gnu.msi

## アンインストール

もし Rust を必要ないと決定した場合は、私達は少し悲しいです、しかしそれもいいでしょう。
必ずしも全てのプログラミング言語が皆にとって素晴らしい物ではないですしね。
アンインストールには次のスクリプトを実行するだけです。

```bash
$ sudo /usr/local/lib/rustlib/uninstall.sh
```

もし Windows インストーラーを使用した場合は、 `*.msi` をもう一度実行して下さい、そしてアンインストールオプションを指定して下さい。

一部の人々は当たり前のように私達が `curl | sh` するように言うと文句を言うことがあります。
基本的に、貴方がこれを実行する時、 Rust をメンテナンスしているいい人が自分のコンピュータを Hack して悪さをしようと思っていない事を信じています。
それは良い直感だと思います!
もし貴方が文句を言う人の一人なら、ドキュメントの[ソースコードからの Rust のビルド][from source]か[公式のバイナリダウンロードページ][install page]から入手して使って下さい。

[from source]: https://github.com/rust-lang/rust#building-from-source
[install page]: http://www.rust-lang.org/install.html

おっと、私達が公式でサポートしているプラットフォームを言うのを忘れてました。

* Windows (7, 8, Server 2008 R2)
* Linux (2.6.18 or later, various distributions), x86 and x86-64
* OSX 10.7 (Lion) or greater, x86 and x86-64

これらのプラットフォームについては広範囲のテストを行っています。そしてその他の少しの Android のような環境でも多少のテストを行っています。
しかし、これらの公式にサポートするプラットフォームが最も高い可用性を持っていてほとんどのテストを通過しています。

最後に， Windows についてです。 Rust はリリースする時に Windows というプラットフォームをファーストクラスに考えています。しかし、正直に言うと、私達の Windows の経験が Linux/OS X の経験のように集まっていません 。
私達は Windows の環境に取り組んでます!!
もし何かしらのバグで動かなかったりしたら、そのバグを私達に知らせてください。
それぞれ全てのコミットは他のプラットフォームと同じように Windows でもテストされます。

もし Rust をインストール出来たならば、シェルを立ち上げて、次のコマンドを入力して下さい。

```bash
$ rustc --version
```

バージョン番号、そのコミットのハッシュ，ビルドを行った日が表示されます。

```bash
rustc 1.0.0-beta (9854143cb 2015-04-02) (built 2015-04-02)
```

もし出来たなら、 Rust のインストールは成功しています! おめでとう!

このインストーラーはインストールするとドキュメントをローカルにコピーします。そしてそれをオフラインで読むことが出来ます。
 UNIX システムなら `/usr/local/share/doc/rust` という場所に、
 Windows なら貴方がRustをインストールしたディレクトリの中の `share/doc` にあります。

もしインストールが完了出来てない場合は、貴方を助ける事の出来る場所が幾つか存在します。
最も簡単な方法は[#rustという Mozilla 公式の IRC チャンネルがあります][irc]、ここには [Mibbit][mibbit] を使用してアクセスする事が出来ます。
そのリンクをクリックすると Rustaceans (私達が呼んでいる Rust ユーザの愛称)とチャットを行う事ができ、そこで私達が貴方を助けられるでしょう。
その他の偉大なリソースとしては[ユーザーズフォーラム][users]や [Stack Overflow][stackoverflow] などがあります。


[irc]: irc://irc.mozilla.org/#rust
[mibbit]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[users]: http://users.rust-lang.org/ 
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust
