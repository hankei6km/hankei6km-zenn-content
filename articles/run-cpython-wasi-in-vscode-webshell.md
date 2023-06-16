---
id: run-cpython-wasi-in-vscode-webshell
title: VSCode の Web Shell 用拡張機能を作って Python を動かしてみた
emoji: 🐚
type: tech
topics:
  - vscode
  - wasm
  - wasi
  - shell
  - python
published: true
---

VSCode で WASI の実行が実験的にサポートされ、Web 版 VSCode でもシェル(Web Shell)が利用できるようなりました。

@[card](https://code.visualstudio.com/blogs/2023/06/05/vscode-wasm-wasi)

この記事では関連する内容として、下記のついて記述してあります。

*   Web Shell を試してみる
*   Web Shell 上で CPython を動かしてみる
*   WASI と Web Shell 対応の拡張機能の作り方

## VSCode の Web Shell とは

詳細は上記のブログ記事を見ていただく方が良いので、ここでは簡単な概要など。

### WASI 対応

まず、シェルの前提となる実行環境について。

VSCode の Web 版で Python などの実行をサポートするために、WASI が実験的にサポートされることになりました。

これまでも `.wasm` 自体を動かすことはできていたのですが、Workspace のファイルを扱うときに統一的な方法がなかったので拡張機能側で工夫する必要がありました。今回の WASI 採用によりこの部分が改善されるため、拡張機能の開発者にとっては大きな利点になるかと思います。

### Web Shell

`.wasm` が扱えるようになると、Python などの実行環境の他に CLI ツールなども動かしたくなります。(その辺を考慮したのかはわかりませんが)拡張機能をコマンドにように実行できる Web Shell もあわせて実験的に提供されました。

Web Shell に対応した拡張機能では `/usr/bin` の下などに仮想的なファイルをマウントできます。このファイルは Web Shell のターミナルから通常のコマンドのように実行できるので、拡張機能の `.wasm` をターミナルから利用可能となります。

### 試してみる

シェルとして試すだけならとくにコードを記述する必要ありません。[insiders.vscode.dev](https://insiders.vscode.dev/) をブラウザーで開いて下記の拡張機能をインストールするだけです。

@[card](https://marketplace.visualstudio.com/items?itemName=ms-vscode.wasm-wasi-core)
@[card](https://marketplace.visualstudio.com/items?itemName=ms-vscode.webshell)

コマンドパレットから `Terminal: Create New Web Shell` を実行するとターミナルが開いて Web Shell を利用できるようになります。リモートリポジトリやローカルのフォルダーなどを開いている場合、Web Shell では `/workspace` を通してファイルへアクセスできます。

**図 1-1 Web Shell のターミナルを開きファイルへアクセス**

![insider.vscode.dev で Web Shell のターミナルを開き、cat コマンドで Workspace のファイルを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6c99e09aa96c498bb93945f26cb6c2f1/run-cpython-wasi-in-vscode-webshell-basic.png?w=1440\&h=860\&auto=compress%2Cformat)

ただし、この記事を書いている時点ではまだ不安定であり `touch` もエラーになりました。また、パイプやリダイレクトなどの機能も未実装な状態でした。

## Web Shell で CPython を動かしてみる

冒頭の記事でも動かしていたのですが、Web 版で実際に試せるコードが見当たらなかったので拡張機能を作ってみました。

@[card](https://github.com/hankei6km/test-vscode-wasm-wasi-ext-python)

Python 本体には冒頭の記事でリンクされていた WASI 用の CPython を利用しています。

@[card](https://github.com/brettcannon/cpython-wasi-build)

リポジトリをクローンして下記のように実行します。

    ```shell-session
    $ npm ci
    $ npm run build
    $ npm run setup:python
    $ npm run serve:http
    ```

::: message

リポジトリには Dev Container もあるので Codespaces でも利用できます。

ただし、後述する理由により、`npm run serve:http` を動かす VSCode はデスクトップ版を使ってください。

:::

<!-- textlint-disable -->

[insiders.vscode.dev](https://insiders.vscode.dev/) 側でコマンドパレットから `Developer: Install Extension From Location` を実行し `http://localhost:5000` を指定します。少し待つと拡張機能がインストールされます。

<!-- textlint-enable -->

**図 2-1 拡張機能がインストールされる**

![拡張機能の管理パネルでインストールされた拡張機能を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e136b06307ba4db09929e0395a17cb9d/run-cpython-wasi-in-vscode-webshell-inst-from-python.png?w=391\&h=501\&auto=compress%2Cformat)

インストールできたら Web Shell を開き `python` を実行するとインタープリターを利用できます。Workspace 上の `.py` の実行も可能です。

**図 2-2 Web Shell で Python を実行**

![Web Shell のターミナルを開き、Python のインタープリターとスクリプトを実行しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/84fde4f49e4e43378f701eabcc7e4cef/run-cpython-wasi-in-vscode-webshell-run-python.png?w=1440\&h=860\&auto=compress%2Cformat)

::: message

ターミナルから `python` を実行すると、その時点で `python.wasm` とライブラリーのファイルをサーバー(`npm run serve:http`) へフェッチしにいきます。

ブラウザーとサーバーのネットワーク的な位置が遠いとわりと時間がかかるので、実行後に反応がない場合も気長に待ってみてください。

:::

::: message

拡張機能をインストールするとき、インストール元の場所が `localhost` でない場合は CSP で接続が拒否されました。

クラウドサービスなどでサーバー(`npm run serve:http`) を動かした場合、ポート転送ではなくサービスのプロキシ(URL)を利用する場合があります。 Codespaces や CodeSandbox などを使う場合はデスクトップ版の VSCode を利用してください。

:::

## `.wasm` で Hello World する拡張機能を作ってみる

ここからは、種類別に Web 版の VSCode で `.wasm` を利用する拡張機能の作り方についてなど。

まずは、Web Shell を利用しない簡単な拡張機能を作ってみます。この手順では下記のような拡張機能を作れます。

@[card](https://github.com/hankei6km/test-vscode-wasm-wasi-ext-hello)

なお、今回は publish などは行わない簡易的なものなので、 `yo` は使わずに手動でファイルを追加しています。

### `package.json`

下記のような `package.json` を用意します。

https://github.com/hankei6km/test-vscode-wasm-wasi-ext-hello/blob/main/package.json

基本的にはコマンドを登録するタイプの Web 拡張機能とほぼ一緒です。ここでは `.wasm` を扱うための部分について少し解説。

*   `extensionDependencies` は依存する拡張機能を指定する記述です

    *   上記では wasm-wasi のコア機能を提供する拡張機能(`ms-vscode.wasm-wasi-core`)を指定しています
    *   ただし、現状では pre release 拡張機能はインストールされないので手動インストールが必要です

*   `dependencies` の `@vscode/wasm-wasi` がコア機能にアクセスためのパッケージです

    *   `npm install @vscode/wasm-wasi` とやると古いバージョンがインストールさるので注意してください(`@next` を指定すると拡張機能とバージョンが合うようです)

*   外部パッケージは esbuild でバンドルしますが `.wasm` ファイルは含めません

    *   `.wasm` は静的ファイルとして扱います

### `extension.ts`

拡張機能のソースは下記のような `extension.ts` となります。

https://github.com/hankei6km/test-vscode-wasm-wasi-ext-hello/blob/main/src/extension.ts

こちらも、基本は一般的な Hello World なサンプル拡張機能と同じ作りです。

ただし、`.wasm` の読み込みについては少し注意点があります。サンプルによっては Node.js の API を使っている場合もありますが、これは環境に依存してしまいます(Web 版では基本的に使えない)。

下記のように拡張機能のコンテキストからリソースの URI を決定して読み込むと Web 版などでも読み込めるようになります。

**リスト 3-1 `hello.wasm` は VSCode の API と URI で読み込む**

```ts
      const filename = Uri.joinPath(
        context.extensionUri,
        'wasm',
        'bin',
        'hello.wasm'
      )
      const bits = await workspace.fs.readFile(filename)
```

### `hello.wasm`

通常は C 言語や Rust などで `.wasm` をビルドしたりしますが、上記リポジトリでは下記のような `.rs` を変換したものを `wasm/bin/hello.wasm` へ配置してあります。今回はこれをそのまま利用します。

**リスト 3-2 `hello.wasm` のソース**

```rust
fn main() {
    println!("Hello, world!");
}
```

**図 3-1 静的ファイルとしてプロジェクト内に配置**

![エクスプローラーで hello.wasm がプロジェクトに含まれている状態のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/30098fbed2134c63ac5d0b5ca078ba95/run-cpython-wasi-in-vscode-webshell-wasm-as-static.png?w=325\&h=565\&auto=compress%2Cformat)

### その他のファイル

`tsconfig.json` などを適宜配置します。

### 実行してみる

Web 版の拡張機能を試しに実行する方法はいくつかりますが、今回はなるべく実環境に近い方がよいので下記の方法を npm スクリプトにしました[^http]。

[^http]: ドキュメントでは `https` で接続するようになっていますが、 `http` 接続でも問題なさそうだったのでそのような npm スクリプトを登録してあります。気になる場合はドキュメントの方に合わせてください。

@[card](https://code.visualstudio.com/api/extension-guides/web-extensions#test-your-web-extension-in-vscode.dev)

下記にように実行するとサーバーが開始されます。

**図 3-2 拡張機能をビルドしサーバーを開始**

```shell-session
$ npm ci
$ npm run build
$ npm run serve:http
```

<!-- textlint-disable -->

[insiders.vscode.dev](https://insiders.vscode.dev/) 側でコマンドパレットから `Developer: Install Extension From Location` を実行し `http://localhost:5000` を指定します。少し待つと拡張機能がインストールされます。

<!-- textlint-enable -->

**図 3-3 拡張機能がインストールされる**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5432e17100254c3f901417ca7c74391e/run-cpython-wasi-in-vscode-webshell-inst-from-helllo.png?auto=compress%2Cformat)

インストールされない場合は `Developer: Show Window Log` などを見るとエラーが表示されていることがあります[^log]。

**図 3-4 アドレスを間違えている場合のログ**

![ログのパネルで fetch が失敗しているメッセージが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/554e1b68a45c48bc8b21f5766aee3b55/run-cpython-wasi-in-vscode-webshell-show-log.png?w=559\&h=162\&auto=compress%2Cformat)

[^log]: その他にはログの「Extension Host(Worker)」とブラウザー側のデベロッパーツールにも情報が表示されることもあります。

インストールできたら `wasm: Run hello` を実行するとターミナルが開いて Hello World が表示されます。

**図 3-5 Hello Workd が表示される**

![VSCode でターミナルが開き Hello World が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e5ccf05691c24c5886013c0f95adb2ec/run-cpython-wasi-in-vscode-webshell-run-hello.png?w=797\&h=401\&auto=compress%2Cformat)

今回の `.wasm` は Hello World するだけですが、リポジトリの Dev Container には Rust で `.wasm` をビルドできる環境も入っています。いろいろ試してみると面白いかと思います。

::: message

上記ではコマンドから実行しているだけですが、エディターのメニューにしたりもできます。この辺は下記のテストコードが参考になるかと思います。

@[card](https://github.com/microsoft/vscode-wasm/tree/main/testbeds/python)

:::

## Web Shell から実行できる拡張機能を作ってみる

Web Shell のターミナルで通常のコマンドのように実行できる拡張機能についてです。ここでは、主に上記の拡張機能との違いについて記述します。

@[card](https://github.com/hankei6km/test-vscode-wasm-wasi-ext-webshell)

まずは、依存する拡張機能として Web Shell を追加します。

**リスト 4-1 依存関係の追加**

```json
  "extensionDependencies": [
    "ms-vscode.wasm-wasi-core",
    "ms-vscode.webshell"
  ],
```

下記のように `contributes` を定義すると `/usr/bin/hello` に仮想的なファイルがマントされます。Web Sehll でこのファイルを実行すると拡張機能(`extension.ts`)側の `wasm-wasi-hello.webshell` が実行されるようになります。

**リスト 4-2 コマンドのファイルをマウント**

```json
  "contributes": {
    "webShellMountPoints": [
      {
        "mountPoint": "/usr/bin/hello",
        "command": "wasm-wasi-hello.webshell"
      }
    ]
  },
```

拡張機能側では `commands.registerCommand` した関数に引数などの情報が渡されるので 、それを元にプロセスを作成します。

**リスト 4-3 マウントされたファイルに対応するコマンドを登録**

```ts
  commands.registerCommand(
    'wasm-wasi-hello.webshell',
    async (
      _command: string,
      args: string[],
      _cwd: string,
      stdio: Stdio,
      rootFileSystem: RootFileSystem
    ): Promise<number> => {

  // ここにプロセスを作成するコードを記述
```

あとは、さきほどと同じように拡張機能をインストールすれば Web Shell のターミナルから `hello` コマンドを利用できます。

**図 4-1 `hello` をコマンドとして実行(引数も渡せる)**

![Web Shell のターミナルを開き、hello コマンドを実行しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8133172081df48b38f7cf948eb481270/run-cpython-wasi-in-vscode-webshell-webshell-hello.png?w=772\&h=554\&auto=compress%2Cformat)

だいたいは一般的な CLI ツールのようにできそうですが、カレントディレクトリの扱いが異なるようなのでファイルは絶対パスにする必要がありました。

@[card](https://github.com/WebAssembly/wasi-filesystem/issues/24)

## Python を実行できる拡張機能を作ってみる

さきほど動かしてみた Python の拡張機能についてです。基本は Web Shell 用にファイルマウントするだけですが、Python の場合は外部に標準ライブラリーも必要になるので、その辺についてなど。

@[card](https://github.com/hankei6km/test-vscode-wasm-wasi-ext-python)

### Python 本体

上の方でも少し書きましたが、WASI 対応の CPython については下記で公開されている `.zip` ファイルを展開してそのまま利用します(パッチを適用してビルドなどは必要ありませんでした)。

@[card](https://github.com/brettcannon/cpython-wasi-build)
@[card](https://github.com/brettcannon/cpython-wasi-build/releases/tag/v3.12.0b2)

### 拡張機能側の処理

基本のファイルの部分は Web Shell 用拡張機能の `hello.wasm` を `python.wasm` を置き換えることになりますが、先ほども触れたカレントディレクトリの問題があります。[サンプル](https://github.com/microsoft/vscode-wasm/blob/0987bce511099db7ceaa3e4d75fd8e5717df9f86/testbeds/python/extension.ts#L55)には回避処理が入っていたので、そのまま利用しています。

**リスト 5-1 カレントディレクトリ関連の回避処理**

```ts
      // WASI doesn't support the concept of an initial working directory.
      // So we need to make file paths absolute.
      // See <https://github.com/WebAssembly/wasi-filesystem/issues/24>
      const optionsWithArgs = new Set([
        '-c',
        '-m',
        '-W',
        '-X',
        '--check-hash-based-pycs'
      ])
      for (let i = 0; i < args.length; i++) {
        const arg = args[i]
        if (optionsWithArgs.has(arg)) {
          const next = args[i + 1]
          if (next !== undefined && !next.startsWith('-')) {
            i++
            continue
          }
        } else if (arg.startsWith('-')) {
          continue
        } else if (!arg.startsWith('/')) {
          args[i] = `${cwd}/${arg}`
        }
      }
```

### 標準ライブラリー

Python では標準ライブラリーもマウントする必要があります。これは必要なファイルを `wasm/lib` の下へコピーし下記のように定義を加えることで実現できます。

**リスト 5-2 標準ライブラリーマウントの設定**

```json
  "contributes": {
    "webShellMountPoints": [
      {
        "mountPoint": "/usr/local/lib/python3.12",
        "path": "wasm/lib"
      },
      {
        "mountPoint": "/usr/bin/python",
        "command": "wasm-wasi-python.webshell"
      }
    ]
  },
```

**図 5-1 ライブラリーのファイルを静的ファイルとして追加**

![エクスプローラーでライブラリーのファイルがプロジェクトに含まれている状態のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f1a928425d214610b7d0d171fc939683/run-cpython-wasi-in-vscode-webshell-lib-as-asset.png?w=339\&h=616\&auto=compress%2Cformat)

ただし、Web 環境で利用する場合は、ファイルツリーを扱うために `.dir.json` を生成しておく必要があります。こちらは `@vscode/wasm-wasi` に付属している `dir-dump` で生成できます。

**図 5-2 `dir-dump` でディレクトリの情報を生成**

```shell-session
$ npx dir-dump --out wasm/lib.dir.json wasm/lib

$ ls -d wasm/lib*
wasm/lib
wasm/lib.dir.json
```

### セットアップ用のスクリプト

ファイルを展開したり `.dir.json` を生成するのはやはり少し手間がかかります。この辺は Python 以外のコマンドを拡張機能化するときにも流用できそうなので、スクリプトにしておくと良いかと思います。

https://github.com/hankei6km/test-vscode-wasm-wasi-ext-python/blob/main/scripts/setup-python.sh

## おわりに

VSCode の Web Sell 用拡張機能を作り CPython を動かしてみました。

CodeSpaces などを利用しないブラウザー上の VSCode でもここまで動くのはやはり面白いです。[テスト用のフォルダー](https://github.com/microsoft/vscode-wasm/tree/main/testbeds)には Python 以外の実行環境もあったので、今後の展開がとても楽しみです。

また、`extensionDependencies` で拡張機能の依存関係を指定できるので、「Python のスクリプトを拡張機能として配布する」といったこともやりやすくなるのかなと思います。
