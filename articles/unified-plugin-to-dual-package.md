---
id: unified-plugin-to-dual-package
title: unified プラグイン(TypeScript で記述)を ESM と CJS のデュアルパッケージにする
emoji: "\U0001F54C"
type: tech
topics:
  - npm
  - unified
  - typescript
  - esm
  - cjs
published: true
---
現在、[nuxt-content](https://content.nuxtjs.org/ja/)(の[Docs テーマ](https://content.nuxtjs.org/ja/themes-docs)) で利用したい [unified](https://unifiedjs.com/) プラグインを作っています。

この場合「unified は ESM 化を進めている」「nuxt-content は CJS としてインポート(require)しようとする」という状況になります。

これについて少し調べてみたところ、現状では以下のような対応がとれるようです。

*   古いバージョンのプラグイン(パッケージ)を使う[^pin]
*   `nuxt.config.js` の `build.transpile` で関連するパッケージを全てトランスパイルする[^transpile]

なのですが、今回は自作のモジュールだったので(少し興味もあったので) ESM と CJS のデュアルパッケージ化を行ってみました。

なお、上記以外に「実行時にスクリプトで CJS 化する方法」も試してあるので、その辺はいずれ。

[^pin]: <https://github.com/nuxt/content/issues/941#issuecomment-894467510>

[^transpile]: <https://github.com/nuxt/content/issues/921#issuecomment-877698146>

## 方針

1 つのパッケージで ESM と CJS 対応する場合、以下に注意点とそれをできるだけ回避しながらの記述方法があります。

@[card](https://nodejs.org/api/packages.html#dual-commonjses-module-packages)

今回は、以下のことから [Approach #2: Isolate state](https://nodejs.org/api/packages.html#approach-2-isolate-state) で記述することにしました。

*   過渡期の対応なので、将来 CJS 部分を簡単に切り離したい

*   自作モジュールはステートレスになっている(ように振る舞う)

    *   ステートの分離などを考えなくてもよい

*   ESM と CJS でプラグインのインスタンスが異なることは割り切る[^instance]

[^instance]: インスタンスが異なると [`processor.use()`](https://github.com/unifiedjs/unified#processoruseplugin-options) の挙動に影響しますが、プラグインを異なる方法でインポートするのはまれだと思われるので。

    > If the processor is already using this plugin, the previous plugin configuration is changed based on the options that are passed in. The plugin is not added a second time.

## コードの記述

ステートレスとしておくことに注意しておけば、普通に TypeScript で記述するだけです。

今回固有の設定としてあえて挙げるとすれば、プラグイン関数のエクスポート方法です。

nuxt-content では default export を期待する動作となっているので、プラグインの関数はデフォルトでエクスポートすることになります。

```typescript:src/index.ts
import { remarkQRCode } from './qrcode.js'
export default remarkQRCode
```

## CJS 用にもトランスパイルする

今回は Isolate で対応するので ESM とは別に CJS 用にもトランスパイルする必要があります。

トランスパイル自体は `tsc` に `module=commonjs` を指定すればよさそうですが、ESM パッケージ内から CJS としてインポートされるにはいろいろハードルがありました。

## 変換後の各ファイルを `.cjs` にする(だけでは上手く行かない)

`package.json` で `type: module` を指定している場合、モジュールを CJS としてインポートしてもらうためには拡張子を `.cjs` とする必要があります。

最初は「module=commonjs を指定して `.cjs` に書き出せばよいかな」くらいに考えてしましたが、「各ファイルを `.cjs` で書き出す」ことは tsc だけだとむずかしそう[^cts]。

ならば「1 ファイルにバンドル後、`mv` してしまう」と思ったのですが、tsc では module=commonjs を 1 ファイルにまとめる方法がないようでした。

また、自身のモジュールのファイルだけ `.cjs` にしても、「外部パッケージで ESM + `.js`」のものがあればやはりエラーとなってしまいます。

unified のプラグインの場合、[unist のユーティリティー](https://github.com/syntax-tree/unist#utilities)(ESM 化が完了しているものが多い)等を使うことが多いので、ここで引っかかります。

[^cts]: TypeScript 4.5 では拡張子に `.cts` を使う方法もあるようです([TypeScript 4.5 以降で ESM 対応はどうなるのか？](https://zenn.dev/teppeis/articles/2021-10-typescript-45-esm))。

## バンドルツールの利用

過渡期の対応のために外部ツールを利用するのもどうかと思っていましたが、あまり時間をかけるのもなにかと思ったので [rollup.js](https://rollupjs.org/guide/en/) でバンドルすることにしました。

なお、今回は「ESM 用の `.js` は tsc で書き出し、そのあとに rollup.js で `index.cjs` を追加する」という流れにしています。

ディレクトリーを `dist/cjs` のように分けたい等の場合は、`index.cjs` 書き出し時にあわせて `.d.ts` を作成する必要があります。

```shell-session
$ npm install --save-dev rollup @rollup/plugin-commonjs @rollup/plugin-node-resolve @rollup/plugin-typescript tslib
```

```javascript:rollup.config.js
import typescript from '@rollup/plugin-typescript'
import commonjs from '@rollup/plugin-commonjs'
import { nodeResolve } from '@rollup/plugin-node-resolve'

export default {
  input: './src/index.ts',
  output: {
    file: './dist/index.cjs',
    format: 'cjs',
    exports: 'default'
    },
  plugins: [
    typescript({
      //module: 'commonjs'
    }),
    nodeResolve(),
    commonjs({
      // extensions: ['.js', '.ts']
    })
  ]
}
```

```json:package.json
{
  "scripts": {
    "build:esm": "tsc && rimraf dist/test && mv dist/src/* dist/ && rimraf dist/src",
    "build:cjs": "rollup -c rollup.config.js",
    "build": "npm run clean && npm run build:esm && npm run build:cjs",
  },
}
```

## エクスポート

各ファイルの準備ができたので、あとはパッケージとしてエクスポートすれば利用可能となります。

今回は、`import` と `require` それぞれの要求に対して、`index.js` と `index.cjs` をエクスポートする設定となります。

```json:package.json
{
  "main": "dist/index.cjs",
  "exports": {
    "import": "./dist/index.js",
    "require": "./dist/index.cjs"
  },
}
```

## 作ってみた

実際に作ってみたものが以下になります。とくに捻ったことをしなくても nuxt-content(の Docs Theme)と自作の native ESM ツールで利用できています。

*   [`remark-qrcode`](https://github.com/hankei6km/remark-qrcode) - Markdown 内の Image 等を QR Code へ変換するプラグイン
*   [`rehype-image-salt`](https://hankei6km.github.io/rehype-image-salt-doc/) - `<img>` を `<nuxt-img>` 等へ再構築するプラグイン

なお、[`rehype-image-salt`](https://hankei6km.github.io/rehype-image-salt-doc/) の「[nuxt-image へ再構築する](https://hankei6km.github.io/rehype-image-salt-doc/nuxt-image)」等は実際に nuxt-content とプラグインを組み合わせて作成しています。

## おわりに

unified が native ESM 化していってる中で逆行するようなことになりますが、しばらくはこのような状況もあると思われるので、対応方法をストックしておくのもありなのかなと。
