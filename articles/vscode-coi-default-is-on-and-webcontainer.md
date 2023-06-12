---
id: vscode-coi-default-is-on-and-webcontainer
title: '?vscode-coi=on を使わないで WebContainer を利用できる拡張機能を作ったけど、デフォルトが変わりそうで少し涙目な話'
emoji: 🦅
type: tech
topics:
  - vscode
  - webcontainer
  - nodebox
  - nodejs
  - wasi
published: true
---

[vscode.dev] でもターミナルを使いたくなったので、[WebContainer] を利用する拡張機能を作ろうと思ったのが 4 月の末頃。

**図 1 こういう感じにしたかった**

![vscode.dev で拡張機能を利用し、vite サーバーを動かしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/21dae5f467334eefa038d128711368c2/vscode-coi-default-is-on-and-webcontainer-intro.png?w=1440\&h=860\&auto=compress%2Cformat)

[WebView](https://github.com/microsoft/vscode-webview-ui-toolkit-samples) を利用すれば対応できるかと考えていたが、[WebContainer] は Cross Origin isolation(coi) を必要とするのが難点。

*   [vscode.dev] のデフォルトでは coi が有効になっていない
*   クエリーパラメーターに `?vscode-coi=on` を指定すると coi になるが、多くの場合で外部の画像などが表示できなくなる

外部の画像が表示されないのでは困りそうなので試行錯誤してみたところ、回避する方法がわかってきたので拡張機能として実装してみた。

<!-- textlint-disable -->

そして、当初の予定ではマーケットプレイスに publish した後、「いやー、苦労したっすよ（ドヤ顔）」って記事を書くつもりでいたら、[Run WebAssemblies in VS Code for the Web](https://code.visualstudio.com/blogs/2023/06/05/vscode-wasm-wasi) によるとデフォルトが `?vscode-coi=on` 変わるのかもという話。

<!-- textlint-enable -->

## `?vscode-coi=on` を使わないで WebContainer を利用する方法

今回は [Nodebox] を利用することにより coi を有効化した外部タブを作りました。

[Nodebox] もブラウザー上で Node.js 的なランタイムを提供するものですが、こちらは coi を必要としません[^nodebox]。

[^nodebox]: だったら Nodebox 使えばいいんじゃね？となりそうですが、Nodebox は Node.js との互換性が低いことと、シェル的なものがないので今回の用途には微妙に合致しませんでした。

1.  WebView で Nodebox のランタイムを動かす
2.  Nodebox のランタイムで [Vite] と [WebSocket] のサーバーを動かす
    *   Vite サーバーは coi を有効にしておき WebContainer ランタイムを動かすページを配信する
3.  外部タブを開き Vite サーバーへ接続し WebContainer のランタイムを動かす
4.  外部タブと WebView は WebSocket と Nodebox の API(stdio)経由で通信する

このようにして coi が有効化された外部タブを作ることにより、coi 無しの [vscode.dev] と [WebContainer] が通信できる状態になります。 (実際には postMessage とのすり合わせとかいろいろあるのですがその辺は割愛します)

## 使ってみる

上記を実装したのが Start WebContaner 拡張機能となります。

@[card](https://marketplace.visualstudio.com/items?itemName=hankei6km.vscode-ext-start-webcontainer)

リポジトリを開いてコマンドパレットから `Start WebContainer: Start` を実行すると外部タブが開き WebContainer が開始されます。

Workspace のファイルは WebContaner 側へロードされているので、あとは、Jsh 用のターミナルで普通に `npm ci` してからファイルを編集するような感じです。

**図 2-1 WebContaine を開始し `npm ci` を実行**

![Start WebContainer 拡張機能でコンテナを開始し、Jsh 用ターミナルで npm ci したスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/da5a8a4be67a4f69bd83ac2b272bd00f/vscode-coi-default-is-on-and-webcontainer-start.png?w=1440\&h=860\&auto=compress%2Cformat)

**図 2-2 エディターでの編集は WebContainer 側へ反映される**

![エディターで編集した内容が、WebContainer が動かしている開発サーバーへ反映されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ea08cccfb5b74428b8e493ae3f1dbcf0/vscode-coi-default-is-on-and-webcontainer-edit.png?w=1440\&h=860\&auto=compress%2Cformat)

リポジトリを用意するのは面倒という場合、ローカルのフォルダーも利用できます。空のフォルダーを開いた後に `Start WebContainer: Start` でターミナルを開き、`npm create next-app@latest` などでプロジェクトを作成するといった感じです。

**図 2-3 空のフォルダーを開き、`npm create` を実行**

![空のフォルダーを開き、\`nom create\` を実行しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4486d0c84df24a348051648d45eff5ef/vscode-coi-default-is-on-and-webcontainer-local.png?w=1440\&h=860\&auto=compress%2Cformat)

作ったプロジェクトを編集する場合は、コマンドパレットから `Start WebContainer: Pick up all files from a container` を実行します。これで各ファイルが Workspace に取り込まれます。

**図 2-4 作成したファイルを取り込み、エディターで開く**

![WebContainer 上で作成したプロジェクトを取り込み、ファイルをエディターで開いているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/49b3f93e35ad4724a1290c1fe0ee0f28/vscode-coi-default-is-on-and-webcontainer-pick-all-files.png?w=1440\&h=860\&auto=compress%2Cformat)

と、ここまでは良い感じですが、いくつかやりにくいところもあります。

*   [Nodebox] と [WebContainer] のランタイムを展開するので重い

*   バックグラウンドでのファイル同期がない

    *   [WebContianer の FileSystemAPI](https://webcontainers.io/api#filesystemapi) に watch 的なものがないのと同期の処理は大変なので実装していません

*   NPM スクリプトやタスクを UI から利用できない

*   Platform 別のバインディングが必要な場合は動かないこともある[^binding]

*   `node_modules` が同期されてないなどから定義の参照などができない

*   時々ターミナルが反応しなくなってしてしまう

こんな感じで少し扱いにくいところもありますが、気楽にファイルを編集しながらターミナルで `npm` も使えるのでわりと気に入っています。

[^binding]: swc は binding がないのでエラーになりました。Next.js では swc を使っているようですが、こちらは専用(?)の [wasm バインディング](https://www.npmjs.com/package/@next/swc-wasm-nodejs)があります。この辺は [StackBlitz] の WebContainer による IDE 環境でも同じようになりそうですが、何かノウハウ的なものがあるのでしょうか？

## Insiders 版のデフォルトは `on` になっている

今回作った拡張機能はわりと気に入ったので、未実装な機能の補完や安定化を進めようと思っていました。ですが README を記述するために `vscode-coi` 関連のことを検索していると [vscode.dev] で coi がデフォルトになりそうな記事がヒットしました。

<!-- textlint-disable -->

[Run WebAssemblies in VS Code for the Web](https://code.visualstudio.com/blogs/2023/06/05/vscode-wasm-wasi) によると [insiders.vscode.dev] では既にデフォルトが `?vscode-coi=on` に変更されたようです[^defaultt-on]\(wasm-wasi のために変更したのか不明ですが、他に記述が見つからないのでここから引用しています)。

[^defaultt-on]: 最初に試したときは Insiders 版でも off だったと思うのですが、きちんとメモしていなかったので少しあやふやです。4 月末より前から on だったかもしれません。

<!-- textlint-enable -->

> [VS Code's WASI implementation](https://code.visualstudio.com/blogs/2023/06/05/vscode-wasm-wasi#_vs-codes-wasi-implementation)
>
> The only difficulty with this approach is that `SharedArrayBuffer` and `Atomics` require the site to be [cross-origin isolated](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer#security_requirements), which, because CORS is very viral, can be an endeavor by itself. This is why it is currently only enabled by default on the Insiders version [insiders.vscode.dev](https://insiders.vscode.dev/) and must be enabled using the query parameter `?vscode-coi=on` on [vscode.dev](https://vscode.dev/)

実際に試してみてもそのような挙動になっています。

下記の検証は [Cross-Browser support with Cross-Origin isolation](https://blog.stackblitz.com/posts/cross-browser-with-coop-coep/) で利用されている画像を用いて行っています。

**図 3-1 `https://insiders.vscode.dev/` では外部リソースに cors corp などの対応が必要**

![insider.vscode.dev では通常の外部画像が表示されない状態のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/086e0612a0b04c318d8bac70c51e283c/vscode-coi-default-is-on-and-webcontainer-coi-on.png?w=1440\&h=860\&auto=compress%2Cformat)

**図 3-2 `https://insiders.vscode.dev/?vscode-coi=off` で off にすると普通に表示される**

![\`?vscode-coi=off\` を指定して開いた insider.vscode.dev では画像が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a8d19a32f07040f18dbb7aebbcdaca81/vscode-coi-default-is-on-and-webcontainer-coi-off.png?w=1440\&h=860\&auto=compress%2Cformat)

そして、Start WebContainer 拡張機能では従来との互換性を優先して [Nodebox] を利用していますが、[Nodebox] はランタイムを `iframe` 内に展開しています。このとき `iframe` は `Cross-Origin-Embedder-Policy` が設定されていないリソースを指しているので、coi が有効になっていると逆に動作しません。

**図 3-3 Nodebox のリソースは coi では読み込めない**

![デベロッパーツールで Cross-Origin-Embedder-Policy に関する警告が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f60775e949c34122bc5a3fed1f869a9c/vscode-coi-default-is-on-and-webcontainer-coep.png?w=1087\&h=202\&auto=compress%2Cformat)

実際のところ、[vscode.dev] でも on になるのかはわかりませんが、on になれば Start WebContainer 拡張機能はおしまいといった感じになります [^nodebxo-coep]。

[^nodebxo-coep]: [Nodebox] が coi 対応するかもしれません。しかし、coi がデフォルトになったら WebView で直接 [WebContainer] を扱えるのはずなので、どちらにしても [Nodebox] なしで作り直した方がシンプルでよいのかなと。

::: message

文章が恨み節ぽくなっているかもしれないので念のため。

Web 版の拡張機能における WASI サポートについては、「VSCode でも [WASIX] がサポートされて BusyBox とか動かないかな」とも妄想していたので、これ自体は超歓迎だったりします。

:::

## その他

coi には直接関係ないのですが、今回の拡張機能を作るときに作った [Rollup] のプラグインについても少し。

[Nodebox] 内にファイルを配置するには、ファイルツリーをオブジェクトとして記述し [`nodebox.fs.init`](https://github.com/codesandbox/nodebox-runtime/blob/main/packages/nodebox/api.md#nodeboxfsinitfiles) API へわたすことになります。

**リスト 4-1 `nodebox.fs.init` のサンプル(ドキュメントより引用)**

```js
await nodebox.fs.init({
  'index.js': `
import { greet } from './greet'
greet('Hello world')
  `,

  'greet.js': `
export function greet(message) {
  console.log(message)
}
  `,
});
```

これは面倒そうだったので [Rollup] のプラグインを作って外部のファイルツリーを差し込めるようしました。

@[card](https://github.com/hankei6km/rollup-plugin-nodebox-fs-files)

テキストファイルだけでなく画像などのバイナリーファイルも扱えたりします。[Nodebox] でちょっと込み入ったコードを扱うとき、よければ試してみてください。

## おわりに

デフォルトの設定を尊重して拡張機能を作ったら、デフォルト設定が変わって動かなくなるというピタゴラ的展開で少し涙目になったので、次に作るときは coi が on のことも想定していこうかと思っています。とほほ。

とか言いながらも、[vscode.dev] でも [WASIX] は気にしているぽいので、内心では wasm-wasi 用拡張機能を試してみたくてウキウキなんですけどね。

> [What comes next?](https://code.visualstudio.com/blogs/2023/06/05/vscode-wasm-wasi)
>
> *   There is also the [WASIX](https://wasix.org/) effort that extends WASI with additional [operating system-like features](https://wasix.org/docs/api-reference) such as process or futex. We will continue to watch this work.

[WebContainer]: https://webcontainers.io/

[Nodebox]: https://sandpack.codesandbox.io/docs/advanced-usage/nodebox

[Vite]: https://ja.vitejs.dev/

[WebSocket]: https://github.com/websockets/ws

[StackBlitz]: https://stackblitz.com/

[ms-vscode.wasm-wasi-core]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.wasm-wasi-core

[Rollup]: https://rollupjs.org/

[vscode.dev]: https://vscode.dev/

[insiders.vscode.dev]: https://insiders.vscode.dev/

[WASIX]: https://wasix.org/
