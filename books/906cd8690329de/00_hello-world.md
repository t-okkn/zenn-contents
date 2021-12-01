---
title: "【1日目】Hello World"
---
# Rustについて

そもそもRustとはどういった言語でしょうか？
もともとMozilla（FireFoxを開発している）が開発しているプログラミング言語で、Mozilla Researchは以下のように述べています。

> Rustはスピード、メモリーの安全性、並行性に焦点を当てたオープンソースのシステムプログラミング言語である。
> （原文：Rust is an open-source systems programming language that focuses on speed, memory safety and parallelism.）
> 
> 出典：[Rust language](https://research.mozilla.org/rust/)

流行りの言語特性としてのメモリーの安全性、並行性に焦点を当てつつもC言語並みの速度を有しています。
それがいろいろな企業やプログラマにウケているということで、気になって勉強を始めた次第です。

# Rustをやってみようと思った動機

学習難易度が高めであるとのことで、好奇心からやってみようと思った次第です。
それ以外にもC言語並みの速さが欲しい場合に書けるようになっておきたいなと。
C言語はあまり好きではないもので・・・。

# 実行環境準備

RustにはPlaygroundが存在しているので実は実行環境を整えなくてもお手軽に実行できるので良いですね。
実行環境の整備等は公式ドキュメントに委ねようと思います。

https://doc.rust-jp.rs/book/second-edition/ch01-01-installation.html

MacかLinuxなら環境を準備することにそれほど手間取らないと思います。
Windowsの場合は上記手順かWSLを利用するのが良いと思います。


Rustに付属しているCargoというビルドシステム兼、パッケージマネージャが卑怯なほど使い勝手が良いのも魅力の一つですね。
すべての言語にCargoの亜種が搭載されないかなと思うほどです。
標準でgitの準備までしてくれる「お・も・て・な・し」にはしびれます。

# -Hello World-

1日目ということでいろいろ書きましたが、今日の本題です。
今日は環境を準備した上でお約束のHello Wordをやってみましょう。

```rust:HelloWorld.rs
fn main() {
   println!("Hello, world!");
}
```

https://play.rust-lang.org/

# 雑感

```rust
fn main() {
```

関数定義は `fn` ですね。
短くてよいのですが、最近kotlinとGolangのせいで割とごちゃごちゃになっていたのに、更に混乱してきました。
（Kotlin： `fun` 、Golang： `func` ）

-----

```rust
println!("Hello, world!");
```

`println!` はマクロですね。
C言語っぽいですが、Rustのマクロはどんな感じなのでしょうか？
おいおい見ていきましょう。

-----

あと気になったのは文末の **`;`** （セミコロン）ですかね。
Rustではどうやら省略することも可能なようです。
Golangはつけなくてよい（言語として文末のセミコロンの省略が推奨されている）のですが、サラッと見た感じRustは省略しないほうが良さそうです。
省略不可の例外が多そうなので。
複数言語をやってきた勘ですが、多分そのほうがバグを生みにくいと思います。
なので、以後セミコロンは省略せずに書いていきます。

# 最後に

こんな感じでゆる～く学習を続けていく予定です。
最終的にはWebAPIか何かを作ろうかなと思っております。

## 参照文献

https://doc.rust-jp.rs/book-ja/ch01-02-hello-world.html
