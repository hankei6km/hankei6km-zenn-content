---
id: react-streaming-ssr-in-wasmedge-quickjs
title: WasmEdge と QuickJS で Rect Streaming SSR のコンテナを作ったら 5MB に収まってしまった
emoji: 🌤️
type: tech
topics:
  - wasm
  - wasi
  - quickjs
  - container
  - react
published: true
---

JavaScript で作ったツールを実行ファイルにする方法を考えていたところ「Wasm(WASI) で JavaScript を使えたらいけるのでは」と思いました。

そこで少し調べてみたところ、世の中は **React の SSR を Wasm でやってやろうぜ** というところまで進んでいました。今回はその辺についてなど。 (WASI で JavaScript を実行ファイルにするのは別途記事にしたいと思っています)

## WasmEdge と QuickJS

日本語の情報が少なかったので WasmEdge と QuickJS、およびその組み合わについて少し説明など。

### WasmEdge(Wasm と WASI)

WasmEdge は Wasm(WASI) の実装の 1 つで、ざっくり言うと Wasm をブラウザー以外で動作させるための WASI ランタイムとなります。

@[card](https://wasmedge.org/)

ここでいきなり WASI というワードが出てきましたが、「Wasm と何が違うの？」的な話は下記のスクラップが参考になります。

@[card](https://zenn.dev/newgyu/scraps/ffbce244b960e6)

WASI の実行環境としては [Wasmer](https://wasmer.io/) や [Wasmtime](https://github.com/bytecodealliance/wasmtime) などが有名ですが、WasmEdge は Cloud(Edge) で動かすことも意識しているそうです。この辺は CNCF 関連でニュースになっていました[^sandbox][^docker]。

[^sandbox]: サンドボックスプロジェクトとはなんぞは「[CNCF連載始めます | フューチャー技術ブログ](https://future-architect.github.io/articles/20200928/)」が参考になります。

[^docker]: 1 年前の記事だからなのか、少し実際の内容とは食い違っています([現状では Docker からコンテナとしての実行は難しい](https://wasmedge.org/book/en/kubernetes/docker.html)など)。

@[card](https://www.publickey1.jp/blog/21/ociwebassemblywasmedgecncf.html)

### QuickJS

QuickJS は組み込み環境などでも動作する JavaScript(ECMAScript)の実行環境です。ES2020 にも対応しているので、いまどきなコードを(おそらくは最初の印象よりも)利用できます。

@[card](https://bellard.org/quickjs/)
@[card](https://qiita.com/poruruba/items/e159464305e5bab6e06e)

また、QuickJS を使うと自分で開発しているアプリケーションへ JavaScript 実行環境を組み込めます。

@[card](https://qiita.com/taskie/items/16cdbc69d4fd4a3dccbf)

### wasmedge-quickjs

WasmEdge では [アプリケーション開発用の言語に JavaScript も含まれています](https://wasmedge.org/book/en/dev/js.html)が、Rust などとは少し対応が異なります。

構成としては `.js` のコードを Wasm 化するのではなく、QuickJS を Wasm 化しています。その上で `.js` コードが実行されます。

@[card](https://github.com/second-state/wasmedge-quickjs)

このような構成は [Wasmer CLI の例](https://github.com/wasmerio/wasmer#quickstart)としても出てきますが、wasmedge-quickjs は特徴の 1 つとして Node.js との互換性が挙げられます。

@[card](https://wasmedge.org/book/en/dev/js/nodejs.html)

たとえば、以下のように `process` を import しているコードも実行できます。

**リスト 1-1 `process` を import するコード**

```js
import * as os from 'os';
import * as std from 'std';
import * as process from 'process'

args = args.slice(1);
print('Hello', ...args);
setTimeout(() => {
    print('timeout 2s');
}, 2000);

let env = process.env
for(var k in env){
    print(k,'=',env[k])
}
```

**図 1-1 エラーなしで実行可能**

```shell-session
$ wasmedge --dir .:. --env VAR1=val1 wasmedge_quickjs.wasm example_js/hello.js 
Hello
VAR1 = val1
timeout 2s
```

ただし、実装されていない API も多いので「ある程度の互換はあるかな」くらいに考えておくのがよさそうです。

## React Streaming SSR とは

こちら説明がなくて良さそうですが、下記の記事がわかりやすかったです。

@[card](https://zenn.dev/yusukebe/articles/41becfd057c416)

## wasmedge-quickjs で React Streaming SSR を試してみる

[WasmEdge のドキュメントに React SSR のページ](https://wasmedge.org/book/en/dev/js/ssr.html)がありますが少し長いです。今回はサンプルの react\_ssr\_stream をとりあえず動かしてみたいと思います。

@[card](https://github.com/second-state/wasmedge-quickjs/tree/main/example_js/react_ssr_stream)

実際に動かすには以下のような手順になります。

*   WasmEdge の実行環境(CLI)をインストール
*   wasmedge-quickjs の`wasmedge_quickjs.wasm` を用意
*   サンプル(react\_ssr\_stream)をビルド
*   WasmEdge の CLI で `wasmedge_quickjs.wasm` からビルドされたサンプルを実行

各種環境を整えるのが少し手間なので、今回は devcontainer を用意しました。Codespace を作成するか、VSCode の Remote - Containers などで開くと Terminal から利用できます。

@[card](https://github.com/hankei6km/test-wasmedge-quickjs-react-ssr-stream)

### WasmEdge の実行環境をインストール

今回は devcontainer 内にインストールしてあるので、Terminal を開けば `wamsedge` を実行できます。

手動でインストールする場合は下記に手順があります。

@[card](https://wasmedge.org/book/en/start/install.html)

### `wasmedge_quickjs.wasm` を用意

devcontainer 内の `~/tmp/wasmedge-quickjs` にリポジトリを clone してあるので、ここでビルドします。

**図 3-1 `.wasm` をビルド**

```shell-session
$ cd ~/tmp/wasmedge-quickjs/
$ cargo build --release --target wasm32-wasi
    Updating crates.io index                
  Downloaded encoding v0.2.33
  Downloaded encoding-index-singlebyte v1.20141219.5
  Downloaded encoding-index-tradchinese v1.20141219.5
<snip...>
```

これで `target/wasm32-wasi/release/wasmedge_quickjs.wasm` にビルドされたファイルが保存されています。

なお、[リリースページからダウンロードもできますが](https://github.com/second-state/wasmedge-quickjs/releases)、この場合はサイズが増えます。

### react\_ssr\_stream をビルド

`example_js/react_ssr_stream` にサンプルのプロジェクトがあるので、ここで依存パッケージをインストールしビルドします。

**図 3-2 パッケージのインストールとビルド**

```shell-session
$ npm install
<snip...>

$ npm run buiuld

> build
> rollup -c rollup.config.js

./main.mjs → dist/main.mjs...
babelHelpers: 'bundled' option was used by default. It is recommended to configure this option explicitly, read more here: https://github.com/rollup/plugins/tree/master/packages/babel#babelhelpers
(!) Unresolved dependencies
https://rollupjs.org/guide/en/#warning-treating-module-as-external-dependency
http (imported by main.mjs)
stream (imported by stream?commonjs-external)
util (imported by util?commonjs-external)
(!) Plugin replace: @rollup/plugin-replace: 'preventAssignment' currently defaults to false. It is recommended to set this option to `true`, as the next major version will default this option to `true`.
created dist/main.mjs in 4.1s
```

これで `dist/main.mjs` にビルドされたファイルが保存されました。なお、警告メッセージにあるように `stream` などの依存関係は解決されていませんが、これは実行時に import されます。

### 実行してみる

`example_js/react_ssr_stream` ディレクトリーで以下のように実行します。

```shell-session
$ wasmedge --dir .:. --dir ./modules:../../modules ../../target/wasm32-wasi/release/wasmedge_quickjs.wasm dist/main.mjs
```

各パラメーターなどは以下のようになっています。

*   `--dir .:.` - ランタイムにカレントディレクトリのアクセスを許可する(`dist/main.mjs` の読み込みに利用)
*   `--dir ./modules:../../modules` - `../../modules` を `./modules` としてアクセスすることを許可する(`main.mjs` からの import に利用)
*   `../../target/wasm32-wasi/release/wasmedge_quickjs.wasm` - コンパイルされた QuickJS の `.wasm` ファイル
*   `dist/main.mjs` - 実行される `.js` の指定

実行すると Port `8001` で listen するのでブラウザーから開いてみます。

**図 3-3 ブラウザーで開くとページが表示される**

![localhost:8001 をブラウザーで開いているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5fdfc859c39f42ffa54929297202e8b0/react-streaming-ssr-in-wasmedge-qiuickjs-wasi.png?w=929\&h=542\&auto=compress%2Cformat)

**図 3-4 Stream により時間差で更新される**
![ブラウザーをリロードし時間差で更新されることを確認しているスクリーンショット](/images/react-streaming-ssr-in-wasmedge-quickjs/react-streaming-ssr-in-wasmedge-quickjs-reload.gif)

**図 3-5 DevTools で確認すると 1 ファイルとして受信されている**

![ブラウザーで DevTools を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/36f92de61c354018b82fa98a21a1e1b0/react-streaming-ssr-in-wasmedge-qiuickjs-devtools.png?w=793\&h=477\&auto=compress%2Cformat)

また curl で開くと時間差でソースが受信されています。

**図 3-6 時間差で受信される**
![curl で受信時の時間差を確認しているスクリーンショット](/images/react-streaming-ssr-in-wasmedge-quickjs/react-streaming-ssr-in-wasmedge-quickjs-curl.gif)

この辺は[前述の記事にあるように動作している](https://zenn.dev/yusukebe/articles/41becfd057c416#%E8%A9%A6%E3%81%99)ので問題はないと思います。

## コンテナで実行してみる

WasmEdge はコンテナとして実行できるというので試してみます。

コンテナというと Docker が有名ですが、現在の Docker はコンテナのランタイムを操作する実装の 1 つとなります。この辺の話は長くなるので下記の記事などを参照してみてください。

@[card](https://zenn.dev/akb428/articles/49e51d4db36896)
@[card](https://zenn.dev/utam0k/articles/74d08c9f556534)

上記を読んで改めて「Wasm をコンテナとして実行できる」の意味を考えると、 **コンテナの低レベルランタイムが `.wasm` を (Linux の実行ファイルのように)実行できる** となります。

よって、Wasm をコンテナとして実行する場合は「対応している低レベルランタイムとそれを利用できるツールセット」を用意することになります。

現時点では [WaamEdge のドキュメントを見ると crun が対応している](https://wasmedge.org/book/en/kubernetes.html)ようなので[^crun-wasi][^youki]、これを元に「イメージのビルドは [buildah ](https://github.com/containers/buildah)、全体の管理は [podman](https://podman.io/)」という構成にしてみます。

buildah と podman については以下が参考になります。

@[card](https://rheb.hatenablog.com/entry/2020/07/16/podman_buidah_for_docker_users)

[^crun-wasi]: crun の [configure](https://github.com/containers/crun/blob/main/configure.ac) を見ると Wasm(WASI)ランタイムとして Wasmer や Wasmtime も指定できそうです。

[^youki]: [Youki](https://containers.github.io/youki/youki.html) は [Wasmer に対応していました](https://containers.github.io/youki/user/webassembly.html)([Wasmtime や WasmEdge もサポートされそうな感じです](https://github.com/containers/youki/pull/548))。

### 環境の構築

:::message
コンテナの Wasm 対応は experimental な部分も多く、環境の構築も「これが定番」というのもなさそうです。

よって、以下の方法は現時点での**どうにかこうにか WasmEdge が動くレベルの環境を構築する方法**となります。
:::

今回は WasmEdge のドキュメントを参考に、さきほど試した devcontainer 上で構築してみます。

@[card](https://wasmedge.org/book/en/kubernetes.html)

crun は `--with-wasmedge` 付きでビルドされたものが必要になるので、以下の方法でインストールします。

**図 4-1 WasmEdge 対応の crun をインストール**

```shell-session
$ wget -qO- https://raw.githubusercontent.com/second-state/wasmedge-containers-examples/main/crio/install.sh | bash

<snip...>

$ crun --version
crun version 1.6.0.0.0.7-5159
commit: 515912848d18bdc29bc87a9d6164fd0fdad6f2a6
spec: 1.0.0
+SYSTEMD +SELINUX +APPARMOR +CAP +SECCOMP +EBPF +CRIU +WASM:wasmedge +YAJL
```

通常は上記方法だけで動きますが、今回は Container 内で動かすのでそのままでは動作しません。対応のために Storage Driver を変更します。 `/etc/containers/storage.con` の `driver` 指定を以下にように変更します。

**リスト 4-1 Storage Driver を VFS へ変更**

    driver = "vfs"

buildah のインストールは[ドキュメントの方法だとエラーになるので](https://wasmedge.org/book/en/kubernetes/demo/wasi.html#build-and-install-the-latest-buildah-on-ubuntu)少し変更して実行します(長いのでファイルにコピーして実行してください)。

**リスト 4-2 buildah のインストール**

```bash
sudo apt-get -y install software-properties-common

export OS="xUbuntu_20.04"
sudo bash -c "echo \"deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /\" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"
sudo bash -c "curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -"

sudo add-apt-repository -y ppa:alexlarsson/flatpak
sudo apt-get -y -qq update
sudo apt-get -y install bats git libapparmor-dev libdevmapper-dev libglib2.0-dev libgpgme-dev libseccomp-dev libselinux1-dev skopeo-containers go-md2man containers-common
sudo apt-get -y install golang-1.16 make

mkdir -p ~/buildah
cd ~/buildah
export GOPATH=`pwd`
git clone https://github.com/containers/buildah ./src/github.com/containers/buildah
cd ./src/github.com/containers/buildah
PATH=/usr/lib/go-1.16/bin:$PATH make bin/buildah
sudo cp bin/buildah /usr/bin/buildah
buildah --help
```

最後に podman をインストールします。

**図 4-2 Podman をインストール**

```shell-session
$ sudo apt-get install podman
```

### イメージをビルド

環境ができたので Rect Streaming SSR を実行できるコンテナのイメージをビルドしてみます(前節で各種ファイルをビルドしている前提で進めます)。

まずはディレクトを作成し必要なファイルをコピーしておきます。

**図 4-3 各種ファイルをコピー**

```shell-session
$ mkdir -p ~/tmp/container
$ cd ~/tmp/container
$ cp ../wasmedge-quickjs/example_js/react_ssr_stream/dist/main.mjs .
$ cp -r ../wasmedge-quickjs/modules .
$ cp ../wasmedge-quickjs/target/wasm32-wasi/release/wasmedge_quickjs.wasm .
```

同じディレクトリーに `Dockerfile` を作成します。ここでは[サンプルのあわあせて](https://wasmedge.org/book/en/kubernetes/demo/wasi.html#create-dockerfile) `CMD` で `.wasm` を指定しています[^entrypoiont]。

[^entrypoiont]: Youki の[ドキュメントでは](https://containers.github.io/youki/user/webassembly.html#build-a-container-image-with-the-module) `ENTRYPOINT`を使っています。おそらくは Linux の実行ファイルのように使い分ければよいのかなと。

**リスト 4-3 Dockerfile**

```docker
FROM scratch
ADD main.mjs modules wasmedge_quickjs.wasm /
CMD ["/wasmedge_quickjs.wasm", "/main.mjs"]
```

準備ができたのでビルドしてみます。ここで `--annotation` を指定することにより Wasm 用コンテナのイメージだと明示されます。

**図 4-4 `--annotation` 付きでビルド**

```shell-session
$ buildah build --annotation "module.wasm.image/variant=compat-smart" -t react_ssr_stream .
STEP 1/3: FROM scratch
STEP 2/3: ADD main.mjs modules wasmedge_quickjs.wasm /
STEP 3/3: CMD ["/wasmedge_quickjs.wasm", "/main.mjs"]
COMMIT react_ssr_stream
Getting image source signatures
Copying blob 0e23d946294b done  
Copying config 5cb21751e8 done  
Writing manifest to image destination
Storing signatures
--> 5cb21751e85
Successfully tagged localhost/react_ssr_stream:latest
5cb21751e85f61c9a94bb7bf41c36113d2c4f3697a1b2d9d8310b13fcb856154
```

とくにエラーもなくイメージが作成できました。サイズを確認すると 4.52MB となっています[^modules]。

[^modules]: `modules` ディレクトリーはそのままコピーしただけなので、ここはもう少し削れるかもしれません。

**図 4-5 作成されたイメージを確認**

```shell-session
$ podman image ls
REPOSITORY                       TAG         IMAGE ID      CREATED             SIZE
localhost/react_ssr_stream       latest      5cb21751e85f  About a minute ago  4.52 MB
```

## コンテナとして実行

イメージができたので podman で `run` できますが、podman のデフォルト設定では標準の crun が使われてしまいます。

**図 5-1 デフォルト設定で run するとエラー**

```shell-session
$ podman run --rm -it --publish 8001:8001 localhost/react_ssr_stream:latest
{"msg":"exec container process `/wasmedge_quickjs.wasm`: Exec format error","level":"error","time":"2022-09-13T05:04:43.000371086Z"}
```

今回は `run` のオプションでラインタイムを切り替えることで対応しています。

**図 5-2 Wasm 対応 crun を指定して実行**

```shell-session
$ podman run --runtime /usr/local/bin/crun --rm -it --publish 8001:8001 localhost/react_ssr_stream:latest
listen 8001...
```

ブラウザーで開くと先ほどと同じように表示されます。

**図 5-3 ブラウザーで開くとページが表示される**

![localhost:8001 をブラウザーで開いているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ae97fd836dae480aa42ed718be9d5aaa/react-streaming-ssr-in-wasmedge-qiuickjs-container.png?w=899\&h=562\&auto=compress%2Cformat)

停止は podman から `container stop` で行います。

**図 5-4 コンテナを停止**

```shell-session
$ podman container stop strange_chebyshev
```

という感じでコンテナとしても動作させることができました。

## 考慮点

### パッケージング(ビルド)がちょっと面倒

「各種ランタイムをどのように用意するか」「`.js` はどの程度の規模になるか」などで状況は変わってきますが、定型的なビルドのフローがないのでその辺が少し面倒かなという感じです。

### WASI ランタイムの互換性

原理的にはコンパイルされた wasmedge-quickjs の `.wasm` ファイルは他の WASI ランタイムでも動作しそうですが、実際に試すとエラーになります。

**図 6-1 WasmEdge 以外だとエラーになる**

```shell-session
$ wasmtime --dir . wasmedge_quickjs.wasm async_demo.js 
Error: failed to run main module `wasmedge_quickjs.wasm`

Caused by:
    0: failed to instantiate "wasmedge_quickjs.wasm"
    1: unknown import: `wasi_snapshot_preview1::sock_open` has not been defined
```

実行環境を自前でインストールできる場合はよいのですが、コンテナランタイムで WasmEdge 以外を有効化されていると困ることになりそうです。

また、バインディングで WASI ランタイムをアプリケーション内に組み込む場合、WasmEdge のビルドが面倒だったりします(とくに Windows 用ビルド)。できればビルドしやすいランタイムでも使いたいのですが、ちょっと難しそうかなと[^windows]。

[^windows]: Wasmtime + [Javy](https://github.com/Shopify/javy) だと特殊な設定をしなくとも Windows 用にビルドできたので WasmEdge も簡単にビルドできるとよいのですが…。

### パフォーマンス

ここまで触れませんでしたが、実行速度などは遅いようです。

@[card](https://wasmedge.org/book/en/dev/js.html#a-note-on-v8)

> Now, the choice of QuickJS as our JavaScript engine might raise the question of performance. Isn't QuickJS [a lot slower](https://bellard.org/quickjs/bench.html) than v8 due to a lack of JIT support? Yes, but ...

リンク先を読んだ感じでは、今回の React SSR のような場合だと**サーバーレスファンクション的に使うならロード時間などでも判断してね**ということなのかなと。

なお `.wasm` 部分は `wasmegec` で[AOT コンパイルができる](https://wasmedge.org/book/en/start/cli.html#wasmedgec)ので、これを利用すると改善できることもあるようです(それでも Node.js よりは遅いようです)。

@[card](https://github.com/second-state/wasmedge-quickjs/issues/57)

## おわりに

wasmedge-quickjs を利用して React Streaming SSR のコンテナを作ってみました。

実際に試してみて「5 MB くらいのサイズで React の SSR が動いている」というのは面白かったです。また、コンテナとして WASI 実行環境が扱えるようになってきているのも興味深いところです。

そして、少し本題から外れますが、下記の記事にクラウドな WASI の面白い利用例があります。

@[card](https://blog.cloudflare.com/announcing-wasi-on-workers/)

今後、マネージドなコンテナなどでも WASI が普通にサポートされたら、React の SSR にも対応できる軽量な JavaScript 環境は面白い存在になるのかもしれません。
