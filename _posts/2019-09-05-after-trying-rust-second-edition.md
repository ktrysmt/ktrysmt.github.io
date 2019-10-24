---
layout: post
title: "Rust Second Editionで勉強になったことまとめ"
date: 2019-09-06 01:00:00 +0900
comments: true
categories: ""
published: true
use_toc: false
description: ""
---

最近仲間内でRust熱が高まっておりSecondEditionがすごく良いみたいだったのでやってみました。


時間はかかりましたが、プログラミング言語の様々なパラダイムやRust流のデザインパターン、プログラミングにまつわる諸問題と、Rustではそれにどう取り組むか、、などなど、とても学びが多かったです。
翻訳チームの方々には頭が下がるばかりです。ありがとうございます、とても勉強になりました。せっかくなので、個人的に学びがあったところをまとめました。

### let if は気軽に使えて便利
リッチな三項演算子といった感じ。
ただし網羅性はmatchに劣るので複雑なパターンが多数想定されるときは避けたほうがいい。
```rs
let number = if condition {
    5 
} else {
    6 
}; // 戻り値の型は全条件で一致させる
```

### 配列の添字アクセスが配列長より大きいとrustはパニックする
しかしコンパイルは通る（！）

### crateのディレクトリ名はkebab-case, extern crateするときはsnake_case
一瞬わかりにくくて焦る。
ディレクトリ名まではさすがにlsp等でlintできないので、そういうもんだと覚えるしかない。

### 文字列操作が他の言語と比べて結構めんどくさい
`String::from("hello")`とか、`String.push_str("world")`などの書き心地の悪さに加え、`&str`のlifetime制約ややこしさを助長してると思う。
structフィールドに`&str`をそのままでは入れられないのははじめは面食らうかも。所有権とlifetimeを理解してから使うこと。とりあえずStringにしておくべき。

あと、`&str`の&は「バイナリに埋め込まれた文字列リテラルを借用している」と理解する(後述)。

文字列操作について
* 文字列結合は `format!` を使うだけでだいぶラクできる。
* slicingテクニックについては[こちらを参照](https://wduquette.github.io/parsing-strings-into-slices/)。
* 文字列パースにはunicodeスカラ値（≒コードポイント）で操作するほうがいい、スライスでの切り出しはめんどくさい。
* ダブルクオートは`&str`型、シングルクォートは`char`型。クォートには注意すること。

### 不変借用ポインタと可変借用ポインタはひとつの値に対して同時に指定できない
考えてみれば当然だけど書くときは気をつけたほうがいい、OOPとかStateを持ち回す書き方はやはり向いてない気がする

### メソッドシグニチャ第一引数の&selfについて
```rs
impl Struct {
  fn method(&self)
}
```
型はStruct自身であり&がついているのでこれはStruct自身の所有権は不要で、値のみ読み出ししたいという意味。
ここをselfとしてしまうと、当該fnを抜けたら同時に所有権をリリースし、呼び出し元側で当該Structが解放されてしまう。
一方で&mut selfとすれば値の読み書き両方ができるようになる。


### structをdeepcopyするには#[derive(Clone)]をつける
ジェネリックな型なら `T: std::clone::Clone` とかでOK



### Option<T>は積極的に使うべき
```rs
let v = vec![1, 2, 3];
let data: Option<&i32> = v.get(2);
match data {
    Some(x) => println!("{}", x),
    None => panic!("no data"),
}
```
値がない場合はNoneが自動で入ってくれる、可変データを安全に扱えるようになる。

### HashMapはランダムアクセスなのに注意
必要であればindexmap::IndexMapを検討すること。
preloadはされていないので使うときはuseすること。
HashMapに値を渡す場合はmoveに注意。なるべくmoveさせてしまうほうがあとあと楽だと思う。

### RUST_BACKTRACE=1は非常に便利
とにかく便利の一言。デバッガたちあげるまでもない、ささっと見る程度でいいならこれ。

### デバッガも便利
docsには言及がなかったけどデバッガにも慣れておくと相当便利、rust-lldbを使う。rustupで勝手に入ってくれる。

### エディタのlsp設定はかなり重要
もはや必須。補完やlint、定義ジャンプは大変便利。
これないと生きていけない。
私はVimmerだが、lspが入ってればrust.vimは別になくてもなんとかなる。

一度入れて慣れてしまえば他の言語でも同じインターフェイスで使えるようになるので本当におすすめ。

### ?演算子は安全な状況なら積極的に使いたい
Result型の組み合わせは戻りのエラー型をある程度自動判定してくれるので、非常に使いやすい。
安全かつ気軽に、LL言語っぽいメソッドチェーンを実現できる。

### ライフタイム三原則
1. 参照引数の数だけライフタイムは存在する
2. 入力ライフタイムが一つで出力ライフタイムも一つなら注釈は省略する
3. &selfはカウントされない

### ジェネリクスとトレイトの組み合わせ
引数のジェネリックな型についてトレイトを指定することで制約が生まれ、
当該トレイトにより利用したいメソッドの存在が保証される。トレイト境界という。
ジェネリックな型に対するトレイト境界についてはwhere句を使うと可読性があがっておすすめ。

### panicを安全にテストするなら
panic用のメッセージを埋め込んでおくとデバッグ効率が上がる。
```rs
#[should_panic(expected = "It is a panic message.")]
```

### テストの走らせ方いろいろ
1. cargo test                     ... 標準
2. cargo test -- --nocapture      ... printlnなどstdoutもそのまま出す、通常は出さない
3. cargo test -- --test-threads=1 ... 並列化しない、fsに書き込みなどする場合はこれが良い

2.3.組み合わせるのもあり。

特定の結合テストを流すには以下のように行う
```sh
ls ./tests/another_integration_test.rs
cargo test --test integration_test
```
拡張子が要らない、 -- はいらない、この二点に注意。

インスタンスを新規生成する際、文字列を使うなら以下のようにまとめて参照を渡してcloneすると所有権まわりを回避しやすくなる
```rs
fn new(args: &[String]) -> Self {
  let id = args[0].clone();
  let name = args[0].clone();
  Struct {
    id, name
  }
}
```

### Option<T>と&str
`return Err("error message")` としたい場合、
戻りの型は`Result<_, &'static str>`と'staticを添える点に注意。あまり直感的ではない。

### 複数mod構成にするときの依存関係
依存し合うmodについては Cargo.toml に

```
[dependencies]
add-one = { path = "../add-one" }
```

とローカル相対パスが使えるのでこれを使うと効率良くできる。
またそれを踏まえ複数のmodで構成されるcrateについては top root に workspace を設定しておくといい。

```
[workspace]

members = [
  "mod-a",
  "mod-b",
]
```

この Cargo.toml があるパスで

```sh
cargo new --lib mod-a 
cargo new --lib mod-b
cargo build 
```
などができる。各モジュールごとにtestやrunももちろんできる。
```sh
cargo test -p mod-a
cargo run -p mod-b
```

### RefCellを使うとMockを書きやすくなる
特に戻り値のないメソッドのUnitTestに使える。

受け取った値をよしなに処理する際、RefCellでwrapしておくと値を積んでおけるのであとから評価できるようになる。

### Rustの並列処理の標準はネイティブスレッド
1:1である点に注意。
なお標準とは別に、並列処理を便利にしてくれるcrateは多くあり、tokio,rayon,crossbeamなどが有名っぽい。

### 構造体とmatch
構造体の各フィールドとマッチするかまでかける
```rs
let p = Point { x: 100, y: 222 };
match p {
    Point { x, y: 222 } => println!("{}", x), // y = 222でxは何でも良い場合
    Point { x: 0, y } => println!("{}", y),   // x = 0でyは何でもいい場合
    Point { x, y } => println!("Default, {}, {}", x, y), // デフォルトパターンの意味
}
```

### matchやlet ifの条件に使ったらもうその変数はmoveされてしまう
一応回避する術もあるので比較的単純な型なら回避策を使ってもいい。

matchでmoveを回避するなら
```rs
let robot_name_2 = Some(String::from("bob"));
match robot_name_2 {
    Some(ref name) => println!("{}", name),
    None => (),
}
println!("{:?}", robot_name_2); // これならmoveが起こらず、コンパイル通る
```
mutも同時に使える
```rs
let mut robot_name_3 = Some(String::from("alice")); // ここと let mut して、
match robot_name_3 {
    Some(ref mut name) => {
        // さらにここで ref mut して、
        *name = String::from("ken"); // ここで Deref すれば
        println!("{}", name); // 値を変更しつつmoveもしない感じに使える
    }
    None => (),
}
println!("{:?}", robot_name_3); // ここは Some("ken") と出る
```
ただcollection系やstructなら素直にCopyトレイトを実装したほうが面倒がないと思う。

### マッチガード式
match arm の中で if を追加的に扱える、これめっちゃ便利
```rs
let x = 4;
let f = false;
match x {
    4 | 5 | 6 if f => println!("4 or 5 or 6 and f is true"),
    n if n == 9 => println!(
        "なんらかの中身をnとして受け取り、かつ n == 9 ならここを通る"
    ),
    _ => (),
}
```

### 空のtupleとResultを使った慣用表現
空のtupleは無意味な、単にOkかErrかを返すときに慣用的に使われるものと捉えておくといいらしい
中に値を入れるならその型をいれないと、もちろんコンパイルエラーになる。
```rs
fn hoge() -> Result<(), ()> {
    if 1 > 0 {
        Ok(())
    } else {
        Err(())
    }
}
```

### ニュータイプパターンにはDerefをimplしておくとラク
wrapper structにいちいちフィールドの値操作に必要なメソッドを実装していくときりがないので、
Derefだけとりあえず実装しておけば薄いコードで必要な処理のコールが許されるので、助かる。


### なぜstrではなく&strなのか？
以下はコンパイルできない。
```rs
let s1: str = "hello there!";
let s2: str = "how is it going?";
```
なぜなのか？

Rustでは、ある型に必要なストレージサイズは１つに定まらないといけないのである。
`i32`なら`int`の32bitだし、 `enum Hoge {}` ならその列挙子の数によって決まるだろう。
とにかくある型に対するストレージサイズというものは、型に対して１：１で定まらないとならんのだ。
そこで `str` は上記のようにDynamicに使いたい。てか書き心地的にDynamicでないとありえない、
あまりにも書きづらすぎるから。

だから `&str` として、あたまにポインタをつけることで、これは動的にサイズが決定する型ですよ、と、
いうことを明らかにしたのである。

同じ理屈で trait objectを使うには `&Trait` or `Box<Trait>` とする必要があり、
常にポインタの後ろにTraitを書かねばならない。

### Unsafeの話（やさしめ）
基本的なやつ。
```rs
// 可変で静的な変数へのアクセスはunsafe
static mut COUNTER: u32 = 0; // global領域に可変。。
fn add(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}
add(3); // 呼び出し自体はunsafeでなくてもよい
unsafe {
    println!("{}", COUNTER); // ここもダングリングの可能性があるのでunsafe
}
```
FFIとUnsafe
```rs
// FFIをするときは常にUnsafeである
extern "C" {
    // C言語の関数を呼び出す書き方
    fn abs(input: i32) -> i32;
}
unsafe {
    println!("abs: {}", abs(-3)); // unsafeでないとabs()は呼び出せない。
}
```

以上です。

次はnomiconか、また別の課題に取り組んでみようと思ってます。
