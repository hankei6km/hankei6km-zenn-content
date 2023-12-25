---
id: run-wasm-wasi-in-vscode
title: VS Code で .wasm(WASI) を実行する拡張機能を 2 つ作った
emoji: 🏃
type: tech
topics:
  - wasm
  - wasi
  - vscode
published: true
---

[WASM WASI Core Extension] により VS Code の拡張機能で `.wasm` (WASI)を扱いやすくなったのですが、実行するためには個別の拡張機能を作成する必要があります。

「それもちょっと面倒だよなぁ」ということで、[WASM WASI Core Extension] を利用しつつ任意の `.wasm` を実行できる拡張機能を作りました。

## どのような拡張機能？

[Wasmtime] ランタイムのようなコマンドライン(ターミナル)で任意の `.wasm` を実行する拡張機能です。

**図 1-1 ****`wasmtime run`**** の例**

```shell-session
$ wasmtime run /path/to/file.wasm
```

**図 1-2 今回の拡張機能の例**

```shell-session
$ rw /path/to/file.wasm
```

上記のように似たような使い方になりますが、今回の拡張機能は「インストールから実行まで手軽に行える」ことが特徴と考えています(構成によっては Web 版の VS Code だけでも実行できます)。

## Web Shell 用の拡張機能

今回作った拡張機能は 2 つあり、1 つは Web Shell 内に `rw` コマンドを配置します。これにより `rw` コマンドから `.wasm` を実行できるようになります。

### インストール

以下の拡張機能をインストールすると利用可能になります。依存する拡張機能も自動的にインストールされます。

@[card](https://marketplace.visualstudio.com/items?itemName=hankei6km.vscode-ext-run-wasm)

ただし、1 つ注意点があります。現状では Preview 版のみ公開しているので、基本的には Insiders 版の VS Code にのみインストールできます。

### 基本操作

以下は [cowsay](https://github.com/wapm-packages/cowsay) を実行しているスクリーンショットです。追加のランタイムを必要としないので Web 版 VS Code 単体でも利用可能となっています。

**図 2-1 ****`rw`**** コマンドで ****`cowsay.wasm`**** を実行**

![Web版の VS Code で Web Shell のターミナルを開き、"rw" コマンドで "cowsay.wasm" を実行しているスクリーショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/95028d0e756b481093e658407383b6f6/run-wasm-wasi-in-vscode-intro-rw.png?w=642\&h=428\&auto=compress%2Cformat)

### `.wasm` の情報

また、 `.wasm` のインポートとエクスポート状況を表示する機能もあります。

**図 2-2 ****`chk1.wasm`**** のインポートとエクスポート**

```shell-session
$ rw --print-imports-exports -- chk1.wasm
import: wasi_snapshot_preview1 args_get function
import: wasi_snapshot_preview1 args_sizes_get function
import: wasi_snapshot_preview1 fd_read function
import: wasi_snapshot_preview1 fd_readdir function
import: wasi_snapshot_preview1 fd_write function
import: wasi_snapshot_preview1 path_open function
import: wasi_snapshot_preview1 environ_get function
import: wasi_snapshot_preview1 environ_sizes_get function
import: wasi_snapshot_preview1 fd_close function
import: wasi_snapshot_preview1 fd_prestat_get function
import: wasi_snapshot_preview1 fd_prestat_dir_name function
import: wasi_snapshot_preview1 proc_exit function
export: memory memory
export: _start function
export: __main_void function
```

### マルチスレッド

[WASM WASI Core Extension] では wasi thread-spawn も利用できています。Rust の [`wasm32-wasi-preview1-threads`] ターゲットを使ってビルドしたものも少し試した限りでは動作しました。

以下、テスト用の `.wasm` を動かしているところです(`0..10` のインデックスを 100ms 間隔で表示する)。

**リスト 2-1 テスト用のソースコード**

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let mut handles = vec![];
    let workers = 2;
    for worker in 0..workers {
        handles.push(thread::spawn(move || {
            println!("start {}", worker);
            for i in 0..10 {
                println!("worker {} count {}", worker, i);
                thread::sleep(Duration::from_millis(100));
            }
            println!("end {}", worker);
        }));
    }
    for handle in handles {
        handle.join().unwrap();
    }
}
```

**図 2-3 ビルドすると thread-spwan を利用している ****`.wasm`**** になる**

```shell-session
/workspace $ rw --print-imports-exports -- workspace.wasm
import: env memory memory
import: wasi_snapshot_preview1 fd_write function
import: wasi_snapshot_preview1 poll_oneoff function
import: wasi_snapshot_preview1 environ_get function
import: wasi_snapshot_preview1 environ_sizes_get function
import: wasi_snapshot_preview1 clock_time_get function
import: wasi_snapshot_preview1 proc_exit function
import: wasi_snapshot_preview1 sched_yield function
import: wasi thread-spawn function
export: memory memory
export: _start function
export: __main_void function
export: wasi_thread_start function
```

**図 2-4 実行し経過時間を表示**

![Web版の VS Code で Web Shell のターミナルを開き、"rw" コマンドでマルチスレッド検証用の \`.wasm\` を実行しているスクリーショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/491059fbc55b47ddac1e4d1dc43cb868/run-wasm-wasi-in-vscode-intro-rw-run-thread-spwan%7D.png?w=962\&h=736\&auto=compress%2Cformat)

スレッドを 2 つ動かしたところ、実行時間は 1382ms でした。いろいろオーバーヘッドがあるので若干遅いですが、2000ms を超えていないのでおそらくは並列に動作していると考えられます。

:::message
**普通にマルチスレッドを試してみたい**

上記の `.wasm` は [CodeSandbox] 上に作った [Devbox] [^csb-devbox] でビルドしてあります。

今回の拡張機能とは関係なく普通に [Wasmtime] でマルチスレッドを試す場合はこちらが利用できます。

@[card](https://codesandbox.io/p/sandbox/test-wasm-thread-spawn-crxtzr)

Fork 後にターミナルから利用できますが、パブリックな [Devbox] は思っているよりも外部から参照できてしまうので気をつけてください。

**図 2-5 ビルド**

```shell-session
➜  workspace git:(master) cargo build --target wasm32-wasi-preview1-threads
➜  workspace git:(master) ll target/wasm32-wasi-preview1-threads/debug/workspace.wasm
-rwxr-xr-x 2 root root 2.1M Dec 23 17:28 target/wasm32-wasi-preview1-threads/debug/workspace.wasm
```

**図 2-6 ****`wasmtime`**** による実行(環境設定は Bash 用にしてあるので Bash から実行)**

```shell-session
➜  workspace git:(master) bash
root@crxtzr:/workspaces/workspace# wasmtime run -S threads=y target/wasm32-wasi-preview1-threads/debug/workspace.wasm
start 1
worker 1 count 0
start 0
worker 0 count 0
worker 1 count 1

< snip ... >

worker 1 count 9
worker 0 count 9
end 1
end 0
root@crxtzr:/workspaces/workspace#
```

[^csb-devbox]: devbox という名前で開発環境を構築するツールはいくつかありますが、それらとは別のツールです(たぶん)。
:::

### 考慮点

Web Shell は実験的な実装であり一般的な機能が実装されていないので(パイプなどが使えない)、あまり凝ったことはできません。

また、どのような `.wasm` ファイルを実行できるわけでもありません。基本的には [WASM WASI Core Extension] が対応しているものに限られます。

たとえば、[Wasmer] のレジストリーの登録されているパッケージは、試した限りでは動作しない方が多い印象でした。

**図 2-7 動作しないパッケージの例(****`wasi_unstable`**** をインポートしている)**

!["rw" コマンドでインポートしている項目の一覧を表示しているスクリーショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b6df266607b84850ae0ea2eb035e19cf/run-wasm-wasi-in-vscode-intro-rw-wasmer.png?w=527\&h=513\&auto=compress%2Cformat)

## 通常ターミナル(シェル)用の拡張機能

もう 1 つは通常のターミナルから利用できる拡張機能です。こちらは、拡張機能が `.wasm` を実行するサーバーとして動作します。

@[card](https://marketplace.visualstudio.com/items?itemName=hankei6km.vscode-ext-serve-run-wasm)

別途、[クライアントツール(](https://github.com/hankei6km/vscode-ext-serve-run-wasm#preparing-the-client-tool)[`crw`](<unsafe:\[object Object]>)[ コマンド)をインストールする](https://github.com/hankei6km/vscode-ext-serve-run-wasm#preparing-the-client-tool)と通常のターミナルで`.wasm` を実行できるようになります。

Web Shell 用の拡張機能と比べるとパイプなども利用できるので扱いやすくはなっています。

**図 3-1 ****`crw`**** コマンドで ****`cowsay.wasm`**** を実行**

![VS Code で通常ののターミナルを開き、"crw" コマンドで "cowsay.wasm" を実行している。また実行結果をパイプで他コマンドへ渡しているスクリーショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f6bd7f0645594980a09f28819a7c4fa5/run-wasm-wasi-in-vscode-intro-crw.png?w=1053\&h=558\&auto=compress%2Cformat)

しかし、下記のような問題点を含んでいます。

*   `.wasm` プロセスの stdio が安定しない

    *   現状、`wasm` プロセスの stdin を閉じる方法がなさそう

*   遅い

*   クライアント - サーバー間の通信はデータを `number[]` に変換している

*   pty を考慮していない

*   クライアントツールは Linux 用しか用意していない

実行ファイルを 1 つ追加するだけで通常のターミナル内で `.wasm` を実行できるところは気に入っているのですが、現状ではちょっと厳しいかなというところです。

## おわりに

任意の `wasm` を VS Code で実行する拡張機能を作ってみました。

正直なところ頻繁に使うものではないですが「自作ツールを `.wasm` としてちょっとだけ動かしてみたい」という場合、「気軽に試す選択肢になるかな」と考えています。

[WASM WASI Core Extension]: https://github.com/microsoft/vscode-wasm

[Wasmtime]: https://wasmtime.dev/

[Wasmer]: https://wasmer.io/

[`wasm32-wasi-preview1-threads`]: https://doc.rust-lang.org/nightly/rustc/platform-support/wasm32-wasi-preview1-threads.html

[WASIX]: https://wasix.org/

[CodeSandbox]: https://codesandbox.io/

[Devbox]: https://codesandbox.io/docs/learn/devboxes/overview
