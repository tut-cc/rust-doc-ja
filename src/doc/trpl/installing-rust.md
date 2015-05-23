% Rustのインストール

ここではまず最初のステップとしてRustのインストールを行います!
Rustをインストールするには幾つかの方法が存在しますが，ここでは最も簡単な`rustip`というスクリプトを使用します．
もし貴方がLinuxかMacの環境ならば，貴方は次のことをすれば良いだけです($を入力しないように注意して下さい．これはコマンドの開始を示しています.)

```bash
$ curl -sf -L https://static.rust-lang.org/rustup.sh | sh
```

もし貴方がこのコマンドを実行することが不安なら[potential insecurity][insecurity]の`curl
| sh`を読んでみて下さい．そして以下の私達の免責事項を見て下さい．
そして気楽に次の二段階のバージョンのインストールを使用して，インストールスクリプトを調べてみて下さい.

```bash
$ curl -f -L https://static.rust-lang.org/rustup.sh -O
$ sh rustup.sh
```

[insecurity]: http://curlpipesh.tumblr.com

もし貴方がWindowsなら，[32-bit installer][win32]か[64-bit installer][win64]をダウンロードして実行して下さい.

[win32]: https://static.rust-lang.org/dist/rust-1.0.0-beta-i686-pc-windows-gnu.msi
[win64]: https://static.rust-lang.org/dist/rust-1.0.0-beta-x86_64-pc-windows-gnu.msi

## Uninstalling

もし貴方がRustを必要ないと決定した場合は，私達は少し悲しいです，しかしそれもいいでしょう．
必ずしも全てのプログラミング言語が皆にとって素晴らしい物ではないですしね．
アンインストールには次のスクリプトを実行するだけです．

```bash
$ sudo /usr/local/lib/rustlib/uninstall.sh
```

もしWindowsインストーラーを使用した場合は，`*.msi`をもう一度実行して下さい，そして，アンインストールオプションを指定して下さい．

一部の人々は当たり前のように私達が`curl | sh`するように言うと文句を言うことがあります．
基本的に，貴方がこれを実行する時，貴方はRustをメンテナンスしているいい人が自分のコンピュータをHackして悪さをしようと思っていない事を信じています.
それは良い直感だと思います!
もし貴方が文句を言う人の一人なら，ドキュメントの[ソースコードからのRustのビルド][from source]か[公式のバイナリダウンロードページ][install page]から入手して使って下さい.

[from source]: https://github.com/rust-lang/rust#building-from-source
[install page]: http://www.rust-lang.org/install.html

Oh, we should also mention the officially supported platforms:
おお，私達が公式でサポートしているプラットフォームを言うのを忘れてました．

* Windows (7, 8, Server 2008 R2)
* Linux (2.6.18 or later, various distributions), x86 and x86-64
* OSX 10.7 (Lion) or greater, x86 and x86-64

私達は広範囲のテストをこれらのプラットフォームに行っています，そして，少しのAndroidのような環境にも．
しかし，これらの公式にサポートするプラットフォームが最も高い可用性を持っていてほとんどのテストを通過しています．

最後に，Windowsについてです．Rustはリリースする時にWindowsというプラットフォームをファーストクラスに考えています．しかし，正直に言うと，私達のWindowsの経験がLinux/OS Xの経験のように集まっていません ．
私達はWindowsの環境に取り組んでます!!
もし何かしらのバグで動かなかったりしたら．そのそれを私達に知らせてください.
それぞれ全てのコミットは他のプラットフォームと同じようにWindowsでもテストされます．

もし貴方がRustをインストールしたならば，シェルを立ち上げて，次のコマンドを入力して下さい:

```bash
$ rustc --version
```

貴方がバージョン番号，そのコミットのハッシュ，ビルド日時を見ることが出来ます:

```bash
rustc 1.0.0-beta (9854143cb 2015-04-02) (built 2015-04-02)
```

もし出来たなら，Rustのインストールは成功しています! おめでとう!

このインストーラーはインストールするとドキュメントをローカルにコピーします．そして貴方はそれをオフラインで読むことが出来ます.
UNIXシステムなら`/usr/local/share/doc/rust`という場所に．
Windowsなら貴方がRustをインストールしたディレクトリの中の`share/doc`にあります．

もしインストールが完了出来てない場合は，貴方は助けるをる事の出来る場所が幾つか存在します.
最も簡単な方法は[#rustというmozilla公式のIRCチャンネルがあります][irc]，ここには[Mibbit][mibbit]を使用してアクセスする事が出来ます.
そのリンクをクリックするとRustaceans(私達が読んでいるRustユーザの相性)とチャットを行う事ができ，そこで私達が貴方を助けられるでしょう．
その他の偉大なリソースとしては[ユーザーズフォーラム][users]や[Stack Overflow][stackoverflow]などがあります．


[irc]: irc://irc.mozilla.org/#rust
[mibbit]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[users]: http://users.rust-lang.org/ 
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust
