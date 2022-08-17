---
id: tokio-tasks-with-buffered-stream-in-rust
title: 【Rust】Tokio の Task を複数実行するとき、バッファリングした Stream でコンパクトにまとめる
emoji: 🏞️
type: tech
topics:
  - rust
  - stream
  - tokio
published: true
---

Rust で実行数を制限しながら複数のファイルを同時にダウンロードしたくなりました。

最初は「非同期処理と Channel でどうにかなるかな」と軽く考えていましたが、`.await` を複数配置することになったりと複雑になりがちです。

そこで試行錯誤してみたところ [Stream](https://rust-lang.github.io/async-book/05_streams/01_chapter.html)(のアダプター)を使うとコンパクトにまとめることができたので、今回はその辺についての記事です。

:::message
この記事で使っている「並行」「並列」などのワードについては Tokio のチュートリアル日本語訳を参考にしています。

@[card](https://zenn.dev/magurotuna/books/tokio-tutorial-ja)

**「Chapter 04 Spawning / 並行性」より引用**

> 並行（concurrency）と並列（parallelism）は同じものではありません。もしあなたが、2つのタスクを交互に行うのなら、それはそれらのタスクを「並行して」こなしているということになりますが、「並列」ではありません。これを「並列」にしたいのであれば、2人の人間が必要で、それぞれのタスクにそれぞれの人間を専任させることになるでしょう。
:::

## ランタイムと各種ユーティリティー

Rust では非同期処理のランタイムや関連するユーティリティーは各種 Crate を組み合わせる必要があります。以下は今回利用している Crate についてです。

ランタイムは表題にあるように [Tokio](https://tokio.rs/) を利用します。また、非同期処理の記述も基本的には Tokio の [Task](https://docs.rs/tokio/0.2.4/tokio/task/index.html) を使うようにしています。

ただし、Stream のアダプター(コンビネーター)は [tokio-stream](https://docs.rs/tokio-stream/latest/tokio_stream/) ではなく [futures-rs](https://rust-lang.github.io/futures-rs/) の [StreamExt](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html) と [TryStreamExt](https://docs.rs/futures/latest/futures/stream/trait.TryStreamExt.html) を利用します。これは利用したいアダプターが [futures-rs](https://rust-lang.github.io/futures-rs/) で定義されているためです。

あとは直接の関連ではないのですが、エラー処理の簡易化のために [anyhow](https://github.com/dtolnay/anyhow) も利用しています。

@[card](https://zenn.dev/yukinarit/articles/b39cd42820f29e)

## どのような処理(パターン)にするか

複数の非同期処理を同時に実行させるパターンはいくつかありますが、今回は冒頭で書いたように「複数ファイルをダウンロードし処理する」を想定しています。

これは「一連のパラメーターをもとに非同期リクエストを送信」「受け取った結果を(同期的に)処理」「最終的に全体をまとめる」パターンになるかと思います。

今回は他の用途にも転用しやすくなるよう、処理を簡素化して考えてみます。

1.  パラメーターとして 1 から 8 の数字を利用

2.  数字をわたされたら Task として以下を実行

    1.  非同期リクエストの代わりに以下を実行

        *   数字が 4 で割り切れるときは 600ms スリープ
        *   それ以外では 200 ms スリープ

    2.  レスポンスを(同期的に)処理する代わりに以下を実行

        *   (`.await` させないで)100 ms スリープ
        *   数字を 10 倍した後に文字列へ変換して返す

3.  返された文字列を結合して保持する(まとめる)

4.  すべてのリクエスト終了後に結合した文字列を表示する

また、上記の確認ができたら最後に [`reqwest`](https://github.com/seanmonstar/reqwest) を使った少し現実的な処理も試してみます。

## シンプルに `join` ではダメなの？

本題の前に少しだけ。

JavaScript の Promise に慣れた方なら「[`Promise.all`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) みたいなのは無いの？」となるかもしれません。

Rust の場合でも(利用する Crate にもよりますが)、`join` `join_all` 的な名前の関数やマクロがあります。パラメーターの個数が「同時に実行しても許容される範囲に収まる」のであればこちらを利用できます。

@[card](https://zenn.dev/nojima/articles/30bef27473a6fd)

しかし、今回はパラメーターの個数を制限できない状況でも扱いたいので、残念ながら少し要件から外れてしまいます。

## ループで順次処理

ここからは実際にコードを試しながら、Stream で Tokio の Task を扱っていきます。

:::message
各コードは [Rust Playground](https://play.rust-lang.org/) で試せるようになっています。
:::

まず、(Stream ではなく)通常のループの中で Task を `.await` することで順次処理してみます。同期的な処理と同じような感覚で記述できるので、同時実行数が 1 で問題ない場合はお手軽かなと思います。

しかし、「リクエスト関連の処理」と「結果をまとめる処理(文字列の連結)」が同じループなのはちょっと面白くない感じもします。

**リスト 4-1 ループで順次処理([Playground で試す](https://play.rust-lang.org/?version=stable\&mode=debug\&edition=2021\&gist=303f2df1f9aa6fa3e6df0f0443e2d53b))**

```rust
use anyhow::Result;
use std::time::{Duration, Instant};

async fn f() -> Result<String> {
    let mut res = String::from("");

    for v in 1..=8 {
        let x = tokio::spawn(async move {
            println!("-start req: {}: {:?}", v, std::thread::current().id());
            tokio::time::sleep(Duration::from_millis(if v % 4 == 0 { 600 } else { 200 })).await;
            let res = v * 10;
            println!("-end req: {}: {:?}", v, std::thread::current().id());

            println!("-start conv: {}: {:?}", v, std::thread::current().id());
            std::thread::sleep(Duration::from_millis(100));
            let res = format!("{}", res);
            println!("-end conv: {}: {:?}", v, std::thread::current().id());
            
            Result::<String>::Ok(res)
        })
        .await;
        
        let s = x??;
        println!("-res: {}", s);
        res = format!("{}:{}", res, s);
    }

    Ok(res)
}

#[tokio::main]
async fn main() {
    let start = Instant::now();

    match f().await {
        Ok(res) => println!("done: {}", res),
        Err(err) => eprintln!("err {}", err),
    }

    let duration = start.elapsed();
    println!("elapsed time: {:?}", duration);
}
```

:::details (実行結果)

**図 4-1 実行結果**

    -start req: 1: ThreadId(2)
    -end req: 1: ThreadId(2)
    -start conv: 1: ThreadId(2)
    -end conv: 1: ThreadId(2)
    -res: 10
    -start req: 2: ThreadId(2)
    -end req: 2: ThreadId(2)
    -start conv: 2: ThreadId(2)
    -end conv: 2: ThreadId(2)
    -res: 20
    -start req: 3: ThreadId(3)
    -end req: 3: ThreadId(3)
    -start conv: 3: ThreadId(3)
    -end conv: 3: ThreadId(3)
    -res: 30
    -start req: 4: ThreadId(3)
    -end req: 4: ThreadId(3)
    -start conv: 4: ThreadId(3)
    -end conv: 4: ThreadId(3)
    -res: 40
    -start req: 5: ThreadId(3)
    -end req: 5: ThreadId(3)
    -start conv: 5: ThreadId(3)
    -end conv: 5: ThreadId(3)
    -res: 50
    -start req: 6: ThreadId(2)
    -end req: 6: ThreadId(2)
    -start conv: 6: ThreadId(2)
    -end conv: 6: ThreadId(2)
    -res: 60
    -start req: 7: ThreadId(2)
    -end req: 7: ThreadId(2)
    -start conv: 7: ThreadId(2)
    -end conv: 7: ThreadId(2)
    -res: 70
    -start req: 8: ThreadId(2)
    -end req: 8: ThreadId(2)
    -start conv: 8: ThreadId(2)
    -end conv: 8: ThreadId(2)
    -res: 80
    done: :10:20:30:40:50:60:70:80
    elapsed time: 3.213847805s

:::

## Stream で順次処理

続いて Stream を使ってみます。

以下は Stream 化したパラメーターを [`map`](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html#method.map) で Task 化することにより順次処理しています。Task の結果も Stream で扱えるのでループを分離できています。

**リスト 5-1 Stream で順次処理([Playground で試す](https://play.rust-lang.org/?version=stable\&mode=debug\&edition=2021\&gist=effa33a4dbc7228c26f89b4c73125b2a))**

```rust
use anyhow::Result;
use std::time::{Duration, Instant};

async fn f() -> Result<String> {
    use futures::StreamExt as _;

    // === リクエスト関連の処理 ===
    let st = futures::stream::iter(1..=8)
        .map(|v| {
            tokio::spawn(async move {
                println!("-start req: {}: {:?}", v, std::thread::current().id());
                tokio::time::sleep(Duration::from_millis(if v % 4 == 0 { 600 } else { 200 })).await;
                let res = v * 10;
                println!("-end req: {}: {:?}", v, std::thread::current().id());

                println!("-start conv: {}: {:?}", v, std::thread::current().id());
                std::thread::sleep(Duration::from_millis(100));
                let res = format!("{}", res);
                println!("-end conv: {}: {:?}", v, std::thread::current().id());

                Result::<String>::Ok(res)
            })
        })
        .then(|x| async move { x.await? });

    // === 結果をまとめる処理 ===
    let mut res = String::from("");
    tokio::pin!(st);
    while let Some(x) = st.next().await {
        let s = x?;
        println!("-res: {}", s);
        res = format!("{}:{}", res, s);
    }
    Ok(res)
}

#[tokio::main]
async fn main() {
    let start = Instant::now();

    match f().await {
        Ok(res) => println!("done: {}", res),
        Err(err) => eprintln!("err {}", err),
    }

    let duration = start.elapsed();
    println!("elapsed time: {:?}", duration);
}
```

:::details (実行結果)

**図 5-1 実行結果**

    -start req: 1: ThreadId(2)
    -end req: 1: ThreadId(2)
    -start conv: 1: ThreadId(2)
    -end conv: 1: ThreadId(2)
    -res: 10
    -start req: 2: ThreadId(2)
    -end req: 2: ThreadId(3)
    -start conv: 2: ThreadId(3)
    -end conv: 2: ThreadId(3)
    -res: 20
    -start req: 3: ThreadId(3)
    -end req: 3: ThreadId(2)
    -start conv: 3: ThreadId(2)
    -end conv: 3: ThreadId(2)
    -res: 30
    -start req: 4: ThreadId(2)
    -end req: 4: ThreadId(2)
    -start conv: 4: ThreadId(2)
    -end conv: 4: ThreadId(2)
    -res: 40
    -start req: 5: ThreadId(2)
    -end req: 5: ThreadId(2)
    -start conv: 5: ThreadId(2)
    -end conv: 5: ThreadId(2)
    -res: 50
    -start req: 6: ThreadId(2)
    -end req: 6: ThreadId(2)
    -start conv: 6: ThreadId(2)
    -end conv: 6: ThreadId(2)
    -res: 60
    -start req: 7: ThreadId(2)
    -end req: 7: ThreadId(2)
    -start conv: 7: ThreadId(2)
    -end conv: 7: ThreadId(2)
    -res: 70
    -start req: 8: ThreadId(3)
    -end req: 8: ThreadId(3)
    -start conv: 8: ThreadId(3)
    -end conv: 8: ThreadId(3)
    -res: 80
    done: :10:20:30:40:50:60:70:80
    elapsed time: 3.224600355s

:::

また [futures-rs](https://rust-lang.github.io/futures-rs/) の [StreamExt](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html) と [TryStreamExt](https://docs.rs/futures/latest/futures/stream/trait.TryStreamExt.html) には便利なアダプターが用意されているので、処理によっては全体を Stream にまとめることもできます。今回の場合だと [`try_fold`](https://docs.rs/futures/latest/futures/stream/trait.TryStreamExt.html#method.try_fold) を使うと以下のように Future を返す関数へ変形できます(`f()` が `async fn` ではなく `fn` になっています)。

**リスト 5-2 Stream とアダプターで順次処理([Playground で試す](https://play.rust-lang.org/?version=stable\&mode=debug\&edition=2021\&gist=4f6731eda16beeb07c44f8743d51e062))**

```rust
use anyhow::Result;
use std::time::{Duration, Instant};

// === async が外れて Future を返している
fn f() -> impl futures::Future<Output = Result<String>> {
    use futures::{StreamExt as _, TryStreamExt as _};

    // === リクエスト関連の処理 ===
    futures::stream::iter(1..=8)
        .map(|v| {
            tokio::spawn(async move {
                println!("-start req: {}: {:?}", v, std::thread::current().id());
                tokio::time::sleep(Duration::from_millis(if v % 4 == 0 { 600 } else { 200 })).await;
                let res = v * 10;
                println!("-end req: {}: {:?}", v, std::thread::current().id());

                println!("-start conv: {}: {:?}", v, std::thread::current().id());
                std::thread::sleep(Duration::from_millis(100));
                let res = format!("{}", res);
                println!("-end conv: {}: {:?}", v, std::thread::current().id());

                Ok(res)
            })
        })
        .then(|x| async move { x.await? })
        .try_fold(String::new(), |acc, x| async move { // === アダプターで結果をまとめる ===
            println!("-res: {}", x);
            Ok(format!("{}:{}", acc, x))
        })
}

#[tokio::main]
async fn main() {
    let start = Instant::now();

    match f().await {
        Ok(res) => println!("res: {}", res),
        Err(err) => eprintln!("err {}", err),
    }

    let duration = start.elapsed();
    println!("elapsed time: {:?}", duration);
}
```

:::details (実行結果)

**図 5-2 実行結果**

    -start req: 1: ThreadId(2)
    -end req: 1: ThreadId(3)
    -start conv: 1: ThreadId(3)
    -end conv: 1: ThreadId(3)
    -res: 10
    -start req: 2: ThreadId(2)
    -end req: 2: ThreadId(2)
    -start conv: 2: ThreadId(2)
    -end conv: 2: ThreadId(2)
    -res: 20
    -start req: 3: ThreadId(2)
    -end req: 3: ThreadId(2)
    -start conv: 3: ThreadId(2)
    -end conv: 3: ThreadId(2)
    -res: 30
    -start req: 4: ThreadId(3)
    -end req: 4: ThreadId(3)
    -start conv: 4: ThreadId(3)
    -end conv: 4: ThreadId(3)
    -res: 40
    -start req: 5: ThreadId(3)
    -end req: 5: ThreadId(2)
    -start conv: 5: ThreadId(2)
    -end conv: 5: ThreadId(2)
    -res: 50
    -start req: 6: ThreadId(3)
    -end req: 6: ThreadId(3)
    -start conv: 6: ThreadId(3)
    -end conv: 6: ThreadId(3)
    -res: 60
    -start req: 7: ThreadId(2)
    -end req: 7: ThreadId(2)
    -start conv: 7: ThreadId(2)
    -end conv: 7: ThreadId(2)
    -res: 70
    -start req: 8: ThreadId(3)
    -end req: 8: ThreadId(3)
    -start conv: 8: ThreadId(3)
    -end conv: 8: ThreadId(3)
    -res: 80
    res: :10:20:30:40:50:60:70:80
    elapsed time: 3.216072554s

:::

なお、今回は手を抜いてしまいましたが、 [`try_fold`](https://docs.rs/futures/latest/futures/stream/trait.TryStreamExt.html#method.try_fold) は Future を返すことになっているので Tokio の Task にもできます。

## `buffered` で並行処理にする

Task を Stream で扱えるようになったので、アダプターの [`buffered`](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html#method.buffered) を利用して並行処理 (同時実行)を試してみます。

[`buffered`](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html#method.buffered) はアダプター内で `Future` をバッファリングしながら `.await` します。このとき「バッファーの容量だけ非同期処理が実行される」ことになるので Task 生成に組み合わせると同時実行数を制限した並行処理にできます。

**リスト 6-1 バッファー容量を 3 にした場合([Playground で試す](https://play.rust-lang.org/?version=stable\&mode=debug\&edition=2021\&gist=da16a6fbe795f74f95601980eb0527bf))**

```rust
use anyhow::Result;
use std::time::{Duration, Instant};

fn f() -> impl futures::Future<Output = Result<String>> {
    use futures::{StreamExt as _, TryStreamExt as _};

    futures::stream::iter(1..=8)
        .map(|v| {
            tokio::spawn(async move {
                println!("-start req: {}: {:?}", v, std::thread::current().id());
                tokio::time::sleep(Duration::from_millis(if v % 4 == 0 { 600 } else { 200 })).await;
                let res = v * 10;
                println!("-end req: {}: {:?}", v, std::thread::current().id());

                println!("-start conv: {}: {:?}", v, std::thread::current().id());
                std::thread::sleep(Duration::from_millis(100));
                let res = format!("{}", res);
                println!("-end conv: {}: {:?}", v, std::thread::current().id());

                Ok(res)
            })
        })
        // === バッファリングすることで複数の Task を並行化 ===
        .buffered(3)
        .map(|x| x?)
        .try_fold(String::new(), |acc, x| async move {
            println!("-res: {}", x);
            Ok(format!("{}:{}", acc, x))
        })
}

#[tokio::main]
async fn main() {
    let start = Instant::now();

    match f().await {
        Ok(res) => println!("res: {}", res),
        Err(err) => eprintln!("err {}", err),
    }

    let duration = start.elapsed();
    println!("elapsed time: {:?}", duration);
}
```

::: message

`buffered` を通すと `then` したのと同じような扱いになるので、`try_fold` の前の `then` が `map` に代わっています。

:::

結果は以下のとおりで、これまでの例に比べて 1 から 3 の Task が(ほぼ)同時に開始されています。その後もバッファーの空き状況により順次開始されているため、経過時間を短縮できています(図 5-2 の約 3.2 秒から訳 1.7 秒へ短縮)。

:::details (実行結果)

**図 6-1 実行結果**

    -start req: 1: ThreadId(2)
    -start req: 2: ThreadId(2)
    -start req: 3: ThreadId(2)
    -end req: 3: ThreadId(2)
    -start conv: 3: ThreadId(2)
    -end req: 1: ThreadId(3)
    -start conv: 1: ThreadId(3)
    -end conv: 3: ThreadId(2)
    -end req: 2: ThreadId(2)
    -start conv: 2: ThreadId(2)
    -end conv: 1: ThreadId(3)
    -res: 10
    -start req: 4: ThreadId(3)
    -end conv: 2: ThreadId(2)
    -res: 20
    -res: 30
    -start req: 5: ThreadId(2)
    -start req: 6: ThreadId(2)
    -end req: 5: ThreadId(2)
    -start conv: 5: ThreadId(2)
    -end req: 6: ThreadId(3)
    -start conv: 6: ThreadId(3)
    -end conv: 5: ThreadId(2)
    -end conv: 6: ThreadId(3)
    -end req: 4: ThreadId(2)
    -start conv: 4: ThreadId(2)
    -end conv: 4: ThreadId(2)
    -res: 40
    -res: 50
    -res: 60
    -start req: 7: ThreadId(2)
    -start req: 8: ThreadId(2)
    -end req: 7: ThreadId(2)
    -start conv: 7: ThreadId(2)
    -end conv: 7: ThreadId(2)
    -res: 70
    -end req: 8: ThreadId(2)
    -start conv: 8: ThreadId(2)
    -end conv: 8: ThreadId(2)
    -res: 80
    res: :10:20:30:40:50:60:70:80
    elapsed time: 1.705036034s

:::

また、[`buffered`](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html#method.buffered) は Stream の順番を保証しますが、保証しない [`buffer_unordered`](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html#method.buffer_unordered) もあります。「時間のかかる Task を追い抜く」ことができるので処理時間が短縮される傾向になります([Playground で `buffer_unordered` を試す](https://play.rust-lang.org/?version=stable\&mode=debug\&edition=2021\&gist=c7b6bec35593ae553fdf206ce143b488))。

## Task を並行処理と並列処理に分割する

`buffered` を使うことにより複数の Task が並行に実行され、処理時間が短縮されました。

しかし、Task の中をよく見ると「I/O バウンドな処理(外部へのリクエスト)」と「CPU バウンドな処理(文字列への変換)」が混在しています[^bound]。今回は軽い処理ですが重い計算などになれば、それぞれを「並行」と「並列」にしたくなりそうです。

[^bound]: 今回は意図的にやりましたが、実際の処理でも同じようなパターンはそれなりにあると予想しています。たとえば静的サイトの生成で複数の Markdown を取得し HTML へ変換するなど。

そこで、文字列への変換部分を [Blocking thread](https://docs.rs/tokio/1.20.1/tokio/task/fn.spawn_blocking.html) として分割してみます。

**リスト 7-1 文字列変換を Blocking thread へ分割([Playground で試す](https://play.rust-lang.org/?version=stable\&mode=debug\&edition=2021\&gist=8098dc1e91f72dd572a73b3f055c5972))**

```rust
use anyhow::Result;
use std::time::{Duration, Instant};

fn f() -> impl futures::Future<Output = Result<String>> {
    use futures::{StreamExt as _, TryStreamExt as _};

    futures::stream::iter(1..=8)
        .map(|v| {
            // === 非同期なリクエストは通常の Task ===
            tokio::spawn(async move {
                println!("-start req: {}: {:?}", v, std::thread::current().id());
                tokio::time::sleep(Duration::from_millis(if v % 4 == 0 { 600 } else { 200 })).await;
                let res = v * 10;
                println!("-end req: {}: {:?}", v, std::thread::current().id());

                Result::<i32>::Ok(res)
            })
        })
        .buffered(3)
        .map(|x| x?)
        .map(|v| {
            // === 文字列変換は Blocking thread ===
            tokio::task::spawn_blocking(move || {
                let s = v?;
                println!("-start conv: {}: {:?}", s, std::thread::current().id());
                std::thread::sleep(Duration::from_millis(100));
                let s = format!("{}", s);
                println!("-end conv: {}: {:?}", s, std::thread::current().id());

                Result::<String>::Ok(s)
            })
        })
        .buffered(3)
        .map(|x| x?)
        .try_fold(String::new(), |acc, x| async move {
            println!("-res: {}", x);
            Ok(format!("{}:{}", acc, x))
        })
}

#[tokio::main]
async fn main() {
    let start = Instant::now();

    match f().await {
        Ok(res) => println!("done: {}", res),
        Err(err) => eprintln!("err {}", err),
    }

    let duration = start.elapsed();
    println!("elapsed time: {:?}", duration);
}
```

結果は以下のとおりで、文字列への変換(`conv`)は異なる Thread に割り振られているので(開始時点で確実に)並列実行となっています。その効果などから若干ですが経過時間が短縮されています(図 6-1 の約 1.7 秒から訳 1.5 秒へ短縮)。

:::details (実行結果)

**図 7-1 実行結果**

    -start req: 1: ThreadId(2)
    -start req: 2: ThreadId(3)
    -start req: 3: ThreadId(2)
    -end req: 3: ThreadId(3)
    -end req: 2: ThreadId(3)
    -end req: 1: ThreadId(2)
    -start req: 4: ThreadId(2)
    -start conv: 10: ThreadId(4)
    -start req: 5: ThreadId(3)
    -start conv: 30: ThreadId(5)
    -start conv: 20: ThreadId(6)
    -end conv: 20: ThreadId(6)
    -end conv: 10: ThreadId(4)
    -end conv: 30: ThreadId(5)
    -res: 10
    -res: 20
    -res: 30
    -start req: 6: ThreadId(3)
    -end req: 5: ThreadId(2)
    -end req: 6: ThreadId(2)
    -end req: 4: ThreadId(2)
    -start conv: 40: ThreadId(4)
    -start req: 7: ThreadId(2)
    -start req: 8: ThreadId(2)
    -start conv: 50: ThreadId(5)
    -start conv: 60: ThreadId(6)
    -end conv: 40: ThreadId(4)
    -res: 40
    -end conv: 60: ThreadId(6)
    -end conv: 50: ThreadId(5)
    -res: 50
    -res: 60
    -end req: 7: ThreadId(2)
    -start conv: 70: ThreadId(4)
    -end conv: 70: ThreadId(4)
    -res: 70
    -end req: 8: ThreadId(2)
    -start conv: 80: ThreadId(6)
    -end conv: 80: ThreadId(6)
    -res: 80
    done: :10:20:30:40:50:60:70:80
    elapsed time: 1.505321287s

:::

ただし、Blocking thread の大量生成はよろしくはさそうなのと、通常の Task でもある程度 Thread が分かれるので使いどころを考える必要はありそうです[^yield]。

[^yield]: また、(あまり良くはないですが)通常の Task でビジーループを使う場合、[`yield_now`](https://docs.rs/tokio/latest/tokio/task/index.html#yield_now) などを使うとスケジューラーに制御を移すこともできます。 (Windows3.1 + VB でループさせるおまじないを思い出して胃がキリキリします)

## エラー処理

`try_fold` を使っているので Task 側でエラーになれば停止するようにはなっています。しかし、同時に実行されていた非同期処理は処理を継続しています。全体的に処理をキャンセルさせるには別途枠組みが必要です。

現状では `.map(|x| x?)` で無造作に [`JoinHandle`](https://docs.rs/tokio/latest/tokio/task/struct.JoinHandle.html) の Result を外していますが、ここでリクエストのエラーもチェックしシグナル的なものを導入するなどが考えられます。この辺は今後の課題といったところでしょうか。

@[card](https://tokio.rs/tokio/topics/shutdown)

**リスト 8-1 エラーをわざと発生させる([Playground で試す](https://play.rust-lang.org/?version=stable\&mode=debug\&edition=2021\&gist=c5eed2238af7766bb63df4b70ecb0c34))**

```rust
use anyhow::Result;
use std::time::{Duration, Instant};

fn f() -> impl futures::Future<Output = Result<String>> {
    use futures::{StreamExt as _, TryStreamExt as _};

    futures::stream::iter(1..=8)
        .map(|v| {
            tokio::spawn(async move {
                println!("-start req: {}: {:?}", v, std::thread::current().id());
                tokio::time::sleep(Duration::from_millis(if v % 4 == 0 { 600 } else { 200 })).await;
                // === 値が 5 のときにエラーを発生させる ===
                if v == 5 {
                    println!("-!err: {}: {:?}", v, std::thread::current().id());
                    anyhow::bail!(format!("Error at {}", v));
                }
                let res = v * 10;
                println!("-end req: {}: {:?}", v, std::thread::current().id());
                Result::<i32>::Ok(res)
            })
        })
        .buffered(3)
        .map(|x| x?)
        .map(|v| {
            tokio::task::spawn_blocking(move || {
                let s = v?;
                println!("-start conv: {}: {:?}", s, std::thread::current().id());
                std::thread::sleep(Duration::from_millis(100));
                let s = format!("{}", s);
                println!("-end conv: {}: {:?}", s, std::thread::current().id());

                Result::<String>::Ok(s)
            })
        })
        .buffered(3)
        .map(|x| x?)
        .try_fold(String::new(), |acc, x| async move {
            println!("-res: {}", x);
            Ok(format!("{}:{}", acc, x))
        })
}

#[tokio::main]
async fn main() {
    let start = Instant::now();

    match f().await {
        Ok(res) => println!("done: {}", res),
        Err(err) => eprintln!("err {}", err),
    }

    let duration = start.elapsed();
    println!("elapsed time: {:?}", duration);
}
```

:::details (実行結果)

**図 8-1 実行結果**

    # stdout
    -start req: 1: ThreadId(3)
    -start req: 2: ThreadId(3)
    -start req: 3: ThreadId(3)
    -end req: 3: ThreadId(3)
    -end req: 1: ThreadId(2)
    -end req: 2: ThreadId(3)
    -start req: 4: ThreadId(2)
    -start req: 5: ThreadId(3)
    -start conv: 10: ThreadId(4)
    -start conv: 20: ThreadId(6)
    -start conv: 30: ThreadId(5)
    -end conv: 10: ThreadId(4)
    -res: 10
    -end conv: 20: ThreadId(6)
    -res: 20
    -end conv: 30: ThreadId(5)
    -res: 30
    -start req: 6: ThreadId(2)
    -!err: 5: ThreadId(3)
    -end req: 6: ThreadId(3)
    -end req: 4: ThreadId(3)
    -start conv: 40: ThreadId(4)
    -start conv: 60: ThreadId(6)
    -start req: 7: ThreadId(2)
    -start req: 8: ThreadId(3)
    -end conv: 40: ThreadId(4)
    -res: 40
    elapsed time: 904.512451ms
    -end conv: 60: ThreadId(6)

    #stderr
    err Error at 5

:::

## 少し現実的な処理で試してみる

ここまでは簡単なスリープで試してみましたが、実際には各種 Crate のオブジェクトを利用することになるので move などでハマる可能性もあります。

@[card](https://caddi.tech/archives/1997#i-2)

そこで、下記の記事を参考に [`reqwest`](https://github.com/seanmonstar/reqwest) を使った少し現実的な処理で試してみます。

@[card](https://qiita.com/Kumassy/items/fec47952d70b5073b1b7)

今回は以下のような処理にしてみます。

*   パラメーターとして GitHub のリポジトリ一覧を渡す

*   各リポジトリで以下の非同期処理を実行

    *   GitHub の API でリポジトリの情報を取得
    *   リポジトリの情報を取得できたら JSON から topics を取りだす

*   topics を一覧として Vec にまとめる

*   処理完了後に Vec を表示

また「リポジトリの情報を取得(リクエスト)」と「topics を取りだす(JSON 操作)」を別 Task に分割してあります。

なお、Playground ではネットワーク処理を利用できないので、サンプルのリポジトリを作成しました。

@[card](https://github.com/hankei6km/test-rust-tokio-tasks-stream)

結果は以下のとおりです。[`reqwest`](https://github.com/seanmonstar/reqwest) の [`Client`](https://docs.rs/reqwest/latest/reqwest/struct.Client.html)を move + clone することでとくに問題なく実行できています。

:::details (実行結果)

**図 9-1 実行結果**

    $ cargo run
         Finished dev [unoptimized + debuginfo] target(s) in 0.07s
         Running `target/debug/test-rust-tokio-tasks-stream`
    -start fetch json: rust-lang-nursery/failure: ThreadId(4)
    -start fetch json: rust-lang-nursery/lazy-static.rs: ThreadId(5)
    -start fetch json: rust-lang/libc: ThreadId(3)
    -end fetch json: rust-lang-nursery/lazy-static.rs: ThreadId(3)
    -start get "topics" from json: rust-lang-nursery/lazy-static.rs: ThreadId(8)
    -end get "topics" from json: rust-lang-nursery/lazy-static.rs: ThreadId(8)
    -start fetch json: bitflags/bitflags: ThreadId(3)
    -end fetch json: rust-lang/libc: ThreadId(5)
    -start get "topics" from json: rust-lang/libc: ThreadId(7)
    -start fetch json: rust-lang/log: ThreadId(5)
    -end get "topics" from json: rust-lang/libc: ThreadId(7)
    -end fetch json: rust-lang-nursery/failure: ThreadId(4)
    -start get "topics" from json: rust-lang-nursery/failure: ThreadId(6)
    -res: rust-lang-nursery/failure: error-handling
    -res: rust-lang-nursery/failure: rust
    -end get "topics" from json: rust-lang-nursery/failure: ThreadId(6)
    -end fetch json: rust-lang/log: ThreadId(4)
    -start get "topics" from json: rust-lang/log: ThreadId(8)
    -res: rust-lang/log: logging
    -res: rust-lang/log: rust-library
    -end get "topics" from json: rust-lang/log: ThreadId(8)
    -end fetch json: bitflags/bitflags: ThreadId(4)
    -start get "topics" from json: bitflags/bitflags: ThreadId(7)
    -res: bitflags/bitflags: bitflags
    -res: bitflags/bitflags: macros
    -res: bitflags/bitflags: structures
    -end get "topics" from json: bitflags/bitflags: ThreadId(7)
    done: [
        "error-handling",
        "rust",
        "logging",
        "rust-library",
        "bitflags",
        "macros",
        "structures",
    ]

:::

## おわりに

複数の Tokio の Task を同時実行するときに Stream を使ってみました。

途中でいろいろ回り道もしたのですが[^trap]、最終的には Stream で [`buffered`](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html#method.buffered) を使うことによりコンパクトになったかな思います。

Task と Stream は他にも便利に使えそうなので、いろいろな事例を調べてみると面白いかもしれません。

[^trap]: [Async Book](https://rust-lang.github.io/async-book/) の Stream のところで[`for_each_concurrent` の解説](https://rust-lang.github.io/async-book/05_streams/02_iteration_and_concurrency.html)を見たときは「これだ」と思ったのでいろいろ試したのですが…。
