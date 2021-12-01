---
title: "【2日目】FizzBuzz問題"
---
# FizzBuzz問題とは？

それでは2日目です。
言語の基礎力を見るには最適なFizzBuzz問題。
それをテーマに見ていきましょう。

そもそもFizzBuzz問題とはなにか？
知らない人はいないと思いますが、一応説明をしておきます。
1以上の自然数を順に標準出力に表示し（縛りによっては標準出力以外のことも）、3の倍数のときには **"Fizz"** を、5の倍数のときには **"Buzz"** を、15の倍数（3の倍数且つ5の倍数）のときには **"FizzBuzz"** を出力するプログラムのことです。
もともとは[FizzBuzzゲーム](https://ja.wikipedia.org/wiki/Fizz_Buzz)から来ているようです。

言語の基礎力を見るのに最適と言われるゆえんは以下にあると思っております。

1. 文字列の出力方法の理解
2. if文（言語によっては式）の理解
3. for文の理解
4. 剰余算とその記法の理解

とりあえずベーシックな実装をすると、少なくともこれくらいはわかっていないと書けないと思います。
縛りプレイを始めだすとまた別の知識が必要になるので、ここではその話はしません。

因みに、縛りプレイとして有名なものはワンライナー系FizzBuzz問題がそれに当たるでしょうか？
Pythonで試しに書いてみます。

```python:oneliner.py
for i in range(1, 21): print(i if i%3>0 and i%5>0 else "Fizz"*(i%3<1)+"Buzz"*(i%5<1))
```

lambdaを使ってもいいのですが長くなるので、文字列の掛け算というPython独特の機能を使って表現しました。
今回はこういうことはしません。

# -FizzBuzz問題-

では、Rustで書いてみましょう。
1～20までのFizzBuzz問題として記載しています。

```rust:fizzbuzz.rs
fn main() {
    let limit = 20;
    
    for number in 1..limit+1 {
        if number % 3 == 0 && number % 5 == 0 {
            println!("FizzBuzz({})", number);
            
        } else if number % 3 == 0 {
            println!("Fizz({})", number);
            
        } else if number % 5 == 0 {
            println!("Buzz({})", number);
            
        } else {
            println!("{}", number);
        }
    }
}
```

https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=25ba28357973a1fdec9b62d0421d8580

# 雑感

```rust
let limit = 20;
```

**let文**で変数の定義ができます。
しかし、Rustの変数の本質はとても難しい上に、理解するまでに一番苦労するところなので、今回は詳細を省略します。
この `let limit = 20` は **"limit"** という変数を **"20"** に設定していると捉えておけば現状大丈夫です。
Rustは強い静的型付け言語でありながらも型推論が可能ですので、型の省略が可能です。
省略せずに書くならば、以下のようになります。

```rust
let limit: i32 = 20;
```

データ型については別途見ていきます。

-----

```rust
for number in 1..limit+1 {
```

Rustの**for文**は最近流行りのイテレータタイプ（ `for i in (iter等)` ）ですね。
手動設定タイプ（ `for (i = 1; i <= 20; i++)` ）ではないです。
`1..limit+1` についてはRange型を生成しています。
Range型はイテレータです。
記法的にはRubyやGo言語と同じですね。
イテレータが空になるまで、一つ一つの要素についてfor文内の処理を実施します。

-----

```rust
if number % 3 == 0 && number % 5 == 0 {
```

Rustの `if` は**文**ではなく**式**です。
**if式**なので以下のような記載も可能です。

```rust:expression.rs
fn main() {
    let number = if true { 100 } else { 0 };
    
    println!("{}", number);
}
```

その代わり、C言語やJavaなどにあるような三項演算子（？：の書式）は存在しません。
演算子は多くの他言語と同様に表されます。

|処理|記号|
|:--:|:--:|
|剰余算|`%`|
|論理AND|`&&`|
|等価比較|`==`|

このあたりは問題ないと思われます。
因みに、論理演算子のANDは当然短絡評価が働きます。
短絡評価をしたくなければ、ビット演算子のAND `&` を使いましょう。

-----

```rust
println!("{}", number);
```

前回登場した、 `println!` ですが、変数を文字列として標準出力をすることが可能です。
`{}` がプレースホルダとなっており、そのプレースホルダに対応する変数の値を文字列化して出力してくれます。
詳細は以下のリンクから。

https://qiita.com/YusukeHosonuma/items/13142ab1518ccab425f4

# 最後に

今回はこれで終了です。
主にfor文とif式について見ました。
特に変わったものではなく普通でしたね。

## 参照文献

https://doc.rust-jp.rs/book-ja/ch03-05-control-flow.html
https://doc.rust-jp.rs/book-ja/appendix-02-operators.html
