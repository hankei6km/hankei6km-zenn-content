---
id: native-esm-with-typescript-jest
title: TypeScript + Jest で native ESM
emoji: "\U0001F363"
type: tech
topics:
  - nodejs
  - npm
  - esm
  - typescript
  - jest
published: true
---
[unified](https://www.npmjs.com/package/unified) まわりが ESM になっていっているので、自作のプラグイン等も ESM 対応していくようにしました。

> This package is [ESM only](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c): Node 12+ is needed to use it and it must be `import`ed instead of `require`d

そして、予想通りにいろいろハマったのですが、TypeScript + Jest 関連で悩み所が多かったのでその辺のメモなど。

なお、TypeScript 4.4.x と Jest 27.3.x の頃からのメモなので内容的に古いこともあります。

## モジュールの native ESM 対応

[my-starter-ts-npm-cli-and-lib](https://github.com/hankei6km/my-starter-ts-npm-cli-and-lib) をベースにした自作モジュールでは、[Pure ESM package](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c) を参考にすることで、実行コード部分は比較的容易に対応できました。

ただし、上記の内容は少し攻めた設定のようなので注意点もあります。

*   `main:` を `exports:` で置き換えてしまうとインポートされなくなるときがある
*   `target` に `es2020` が推奨されていますが、`esnext`(または `es2022`)が必要になるときもある(今回は後述の top-level await 関連で必要となりました)

## ts-jest の ESM 対応設定

上記の設定で `import` に拡張子が必要となりますが、jest(ts-jest) 側で認識させるには設定が必要でした。

この辺は ts-jest のドキュメントに推奨の設定があるのでそれを適用すれば解決されます。

@[card](https://kulshekhar.github.io/ts-jest/docs/guides/esm-support)

外部モジュールに ESM 必須のものがなれけば、ここまでの設定で普通にテストも動いてしまいます(`.ts` のテストコードをトランスパイルする関係で CJS として動作するもよう) 。

しかし、`require` も通ってしまうことと、 unified 等を import したいので jest 用の設定も native ESM 対応することにしました。

## Jest の native ESM 対応設定

これも Jest のドキュメンに設定方法があります。

@[card](https://jestjs.io/ja/docs/ecmascript-modules)

また、以下も参考になります。

@[card](https://github.com/facebook/jest/issues/9430#issuecomment-616232029)

前述の ts-jest の設定を適用している場合は、以下の設定で対応できました(ESM 必須のモジュールをインポートしたテストが実行できる状態になる)。

*   `jest.config.js` を ESM のデフォルトエクスポートに書き換える
*   `transform` は `{}` にする
*   Jest を走らせる Node.js には `--experimental-vm-modules` を渡す(後述しますが少し注意点があります)
*   テストコードで `jest` オブジェクトを使う場合は `@jest/globals` からインポートする

```javascript:jest.config.js
export default {
  roots: ['<rootDir>'],
  testMatch: [
    '**/__tests__/**/*.+(ts|tsx|js)',
    '**/?(*.)+(spec|test).+(ts|tsx|js)'
  ],
  transform: {},
  testEnvironment: 'jest-environment-node',
  // https://kulshekhar.github.io/ts-jest/docs/next/guides/esm-support/
  preset: 'ts-jest/presets/default-esm',
  extensionsToTreatAsEsm: ['.ts'],
  globals: {
    'ts-jest': {
      useESM: true
    }
  },
  moduleNameMapper: {
    '^(\\.{1,2}/.*)\\.js$': '$1'
  }
}
```

```json:package.json

  "scripts": {
    "test": "node --experimental-vm-modules node_modules/.bin/jest",

```

なお、 `@jest/globals` からインポートした `jest` で `mockImplementation` をいままでのように使うと型のチェックで弾かれます。

とりあえずは `jest.fn()` で関数を渡してしまえばよいのですが、この辺の対応は調査中。

また、node へ `--experimental-vm-modules` を渡す方法に少し注意点があります(後述の VSCode 関連)。

## native ESM での jest モックモジュール

ESM では `require` を使わなないので従来とはモックの作り方が異なるようです。

@[card](https://github.com/facebook/jest/issues/10025)

試した限りでは以下のようにして動かすことができました。

1.  現時点では `jest.unstable_MockModule` を使う

2.  モックにしたいモジュールは[動的インポート](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import#dynamic_import)で読み込む

3.  モックしたモジュールを利用するモジュールも後から動的インポートする

    1.  [モジュールの読み込み順の自動調整](https://jestjs.io/ja/docs/manual-mocks#es-module-import%E3%82%92%E5%88%A9%E7%94%A8%E3%81%99%E3%82%8B)は**実施されない**
    2.  よって、top-level await が必要となる

ここで、コードの記述とは別に top-level await 対応がでてきます。TypeScript で top-level await を使うには `module` の設定で `esnext` (あるいは `es2022`) が必要になります。

```json:tsconfig.json

  "compilerOptions": {
    "module": "esnext",

```

なお、native ESM 時での `__mocks__` の利用方法は不明です(自分ではあまり使わないので詳しくは調べていません)。

## VSCode 関連

以下は VSCode 依存ということでもなさそうですが、VSCode でしか試していないのでこのようなくくりにしています。

### 構文チェック

VSCode でテストコードを記述している場合、構文チェックに `esnext` を適用させるにはトランスパイルの対象にテストコードを含める必要がありました。

ただし、これを実施するとビルド時にテストコードも含まれてしまうので、この辺も別途対応が必要になってきます。

```json:tsconfig.json

  "include": [
    "src/**/*.ts",
    "test/**/*.ts",

```

### デバッグ

Jest 実行時に `--experimental-vm-modules` を利用する場合、ブレイクポイントを有効にするには以下のように Jest を実行する必要がありました。

```json:package.json

  "scripts": {
    "test": "node --experimental-vm-modules node_modules/.bin/jest",

```

以下のようなシェル変数経由での指定では有効になりませんでした(理由は不明)。

```json:package.json

  "scripts": {
    "test": "NODE_OPTIONS=--experimental-vm-modules jest",

```

or

```json:package.json

  "scripts": {
    "test": "NODE_OPTIONS=--experimental-vm-modules node node_modules/.bin/jest",

```

### コード補完と動的インポート

コード補完で関数等の `import` を実施すると常に静的なインポートとして追加されてしまいます。

例えば、以下のようにモックを利用する関数が動的インポートされている場合に、`transformContent` 関数をコード補完で追加インポートしたとします。

```typescript:foo.spec.ts
jest.unstable_mockModule('fs/promises', async () => {
  // ...
})

const { saveContentFile, saveRemoteContent } = await import(
  '../../src/lib/content.js'
)
```

これは以下のようになり、`content.js` がモックの前にインポートされてしまいます。

```typescript:foo.spec.ts
import { transformContent } from '../../src/lib/content.js'

jest.unstable_mockModule('fs/promises', async () => {
  // ...
})

const { saveContentFile, saveRemoteContent } = await import(
  '../../src/lib/content.js'
)
```

この状態では `saveContentFile` などもモックが適用されない状態になります。

現時点で対応方法は不明なので、手動で対応することになります。

## とりあえず

Jest のモック関連で`jest.unstable_MockModule` を使う必要があったり等の暫定的な記述がありますが、native ESM としてテスト等が実行できるようになりました。
