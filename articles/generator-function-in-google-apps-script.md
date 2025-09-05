---
id: generator-function-in-google-apps-script
title: Google Apps Script で Generator 関数を使うことは普通ではない？
emoji: 🏗️
type: tech
topics:
  - googleappsscript
  - zennfes2025free
published: true
---

「GAS で Generator 関数を使った例ってあまり見かけないなぁ」と思ったので表題のことを GitHub Copilot に質問してみた。

<!-- textlint-disable -->

::: details GitHub Copilot からの「はい、その通りです」な返答

> はい、その通りです。
>
> Google Apps Script（GAS）でgenerator関数（function）を使うことは一般的ではありません。\*
>
> ### 理由
>
> *   **GASはES5ベースが長く続いていたため、ES6のgenerator関数がサポートされたのは比較的最近です。**
> *   公式ドキュメントやサンプルでもgenerator関数の利用例はほとんどありません。
> *   GASはサーバーサイド・バッチ処理の用途が多く、非同期処理や逐次処理をgeneratorで制御するケースが少ないです。
> *   for...ofやspread構文など、generatorと相性の良い最新構文もGASの環境によっては動作しない場合があります。
>
> ### 技術的には使える？
>
> *   **現在（2025年）ではV8ランタイムのGASであればgenerator関数自体はサポートされています。**
> *   ただし、TypeScriptで書いた場合や古いプロジェクトでは注意が必要です。
>
> ### まとめ
>
> *   **使えるが、普通は使われない。**
> *   コードの可読性や移植性を考えると、GASでは通常の関数やforEach、mapなどを使うことが多いです。
>
> **どうしてもgeneratorを使いたい理由があれば、技術的な注意点を説明できますのでご相談ください。**

:::

<!-- textlint-enable -->

こちらとしては「そんなことないですよ」という返答を期待していたのですが、あっさり肯定されてしまいました。

しかし、よく読んでみると「V8 ランタイム利用は通常ではないみたいな空気だしてるけど、[前の実行環境はもうすぐ利用できなくなるよね](https://developers.google.com/apps-script/guides/v8-runtime/migration?hl=ja)」とも思います。

そこで、「GAS でも Generator 関数を使うと便利ですよ」という記事を書いてみます。

## そもそも Generator 関数とは

詳しいことは MDN の記事に丸投げします。

@[card](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/function*)

この記事では主に、以下のような「[`for...of`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Iterators_and_generators)[ 用に Iterator を簡単に作成する](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Iterators_and_generators)」利用方法について注目していきます。

**リスト 1-1 Generator 関数で簡単なループの例**

```js
function basic() {
  function* Gen() {
    let c = 0;
    yield `あいうえお: ${c++}`
    yield `かきくけこ: ${c++}`
    yield `さしすせそ: ${c++}`
  }

  const i = Gen()
  for (const w of i) {
    console.log(w)
  }
}
```

    19:34:04	お知らせ	実行開始
    19:34:04	情報	あいうえお: 0
    19:34:04	情報	かきくけこ: 1
    19:34:04	情報	さしすせそ: 2
    19:34:05	お知らせ	実行完了

## GAS では使えるのか？

冒頭の Copilot からの返答に「使えない構文あるのでは」みたいに書かれていたので、軽く確認してみます。

### スクリプトエディターでは Generator 関数(とGenerator オブジェクト)を認識している

まず、上記例は GAS で試した結果を記述していますが、想定した通りに動作しています。

また、スクリプトエディターでは型の推論も行われ、保存時にもとくにエラーなどは表示されませんでした。

**図 2-1 エディター上では Generator 関数も認識されている(****`Generetor`**** ジェネリック型も定義されている)**

![スクリプトエディターで Generator 関数を呼び出す記述をしているとき、関数が返す Generator 型についてホバー表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3253d34aed274436add3a7a6f2e6abbb/generator-function-in-google-apps-sciript-generator-type.png?w=717\&h=172\&auto=compress%2Cformat)

エディター上での挙動だけで実際の動作が保証されるわけでもないのですが、少なくともコードの記述は普通に行えそうです。

### `for…of` などでの利用

まずは、Copilot の返答でも表示されていた、代表的な利用方法である `for...of` と Spread 構文を試してみます。

**リスト 2-1 配列を 2 要素毎に生成するサンプル**

```js
function basic1() {
  function* Gen(arr, count) {
    let s = []
    for (const i of arr) {
      s.push(i)
      if (s.length == count) {
        yield s
        s = []
      }
    }
    if (s.length > 0) {
      yield s
    }

  }

  const count = 2
  const arr = ['あ', 'か', 'さ', 'た', 'な']

  // for...of
  for (const w of Gen(arr, count)) {
    console.log(w)
  }

  // spread
  const s =[...Gen(arr,count)]
  console.log(s)
}
```

    22:28:07	お知らせ	実行開始
    22:28:06	情報	[ 'あ', 'か' ]
    22:28:06	情報	[ 'さ', 'た' ]
    22:28:06	情報	[ 'な' ]
    22:28:06	情報	[ [ 'あ', 'か' ], [ 'さ', 'た' ], [ 'な' ] ]
    22:28:07	お知らせ	実行完了

とくに問題なく利用できました。

### Iterator の高度な機能

Generator 関数が作成する Generator オブジェクトは Iterator として[外部から影響を与える機能もあります](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Iterators_and_generators#%E9%AB%98%E5%BA%A6%E3%81%AA%E3%82%B8%E3%82%A7%E3%83%8D%E3%83%AC%E3%83%BC%E3%82%BF%E3%83%BC)。今回は簡単に `return()` を試してみましたが、これもとくに問題なく使えるようです。

**リスト 2-2 ****`return()`**** で外部から停止**

```js
function return_func() {
  function* Gen() {
    try {
      console.log('----1')
      yield 'あいうえお'
      console.log('----2')
      yield 'かきくけこ'
      console.log('----3')
      yield 'さしすせそ'
      console.log('----4')
      yield 'たちつてと'
      console.log('----5')
    } catch (e) {
      console.error(e)
    } finally {
      console.log('done:')
    }
  }

  const i1 = Gen()
  for (const w of i1) {
    console.log(w)
  }

  const i2 = Gen()
  for (const w of i2) {
    console.log(w)
    if (w === 'かきくけこ') {
      i2.return()
    }
  }
}
```

    19:59:38	お知らせ	実行開始
    19:59:38	情報	----1
    19:59:38	情報	あいうえお
    19:59:38	情報	----2
    19:59:38	情報	かきくけこ
    19:59:38	情報	----3
    19:59:38	情報	さしすせそ
    19:59:38	情報	----4
    19:59:38	情報	たちつてと
    19:59:38	情報	----5
    19:59:38	情報	done:
    19:59:38	情報	----1
    19:59:38	情報	あいうえお
    19:59:38	情報	----2
    19:59:38	情報	かきくけこ
    19:59:38	情報	done:
    19:59:38	お知らせ	実行完了

このような感じなので、少し突っ込んで使っても大きな問題はなさそうに思われます。

## GAS だと何が便利になるのか？

GAS でも使えそうだとわかったところで、GAS ならではの使い所について少し書いてみます。

### ファイル一覧のループを簡素化

Drive API サービスを使った場合で、普通にループを記述するとファイル一覧のページ処理を必要とするため少し面倒です。

これに対して Generator 関数を使うと以下のようになります。

**リスト 3-1 ファイル一覧取得を ****`for...of`**** で扱う**

```js
function* fileGenerator_(folderId) {
  let pageToken = undefined
  do {
    const f = Drive.Files.list({
      q: `'${folderId}' in parents and trashed=false`,
      pageToken
    })
    for (const file of f.files) {
      yield file.name
    }
    pageToken = f.nextPageToken
  } while (pageToken != undefined)
}

function myFunction() {
  const i = fileGenerator_('folder id');
  for (const fileName of i) {
    console.log(fileName);
  }
}
```

`pageToken` を使った `do while` ループが残っているので簡素化された実感も少ないですが、本体のループ処理を `for...of` で記述できるので心理的には楽になると思います。

### `UrlFetchApp.fetchAll` をレコード受信処理のバッファーとして利用

GAS でのフェッチは同期処理なので平行処理化は難しいのですが、複数の URL を扱う場合は [`UrlFetchApp.fetchAll`](https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app?hl=ja#fetchAll\(Object\)) を利用するとまとめて処理できます。

これを利用すると、クラウドサービスから複数のレコードを受信するとき、バッファー付きストリームのような処理を比較的容易に記述できます。

**リスト 3-2 ****`fetchAll`**** をバッファー付きのストリーム的な処理にする**

```js
function* buffered_(urlsArray, count) {
  let s = []
  for (const i of urlsArray) {
    s.push(i)
    if (s.length == count) {
      yield s
      s = []
    }
  }
  if (s.length > 0) {
    yield s
  }
}

function* streamed_(urlsArray, count) {
  for (const urls of buffered_(urlsArray, count)) {
    const resAll = UrlFetchApp.fetchAll(urls)
    for (const res of resAll) {
      const arr = JSON.parse(res.getContentText()).args.arr
      for (const v of JSON.parse(arr)) {
        yield v
      }
    }
  }
}

function myFunction() {
  const count = 3
  const i = streamed_([
    'https://httpbin.org/get?arr=[10,20,30,40]',
    'https://httpbin.org/get?arr=[15,25]',
    'https://httpbin.org/get?arr=[50,60,70,80,90]',
    'https://httpbin.org/get?arr=[55,65,75]',
    'https://httpbin.org/get?arr=[40]',
    'https://httpbin.org/get?arr=[45,55,85,95]',
    'https://httpbin.org/get?arr=[10,60]'
  ], count)
  for (const v of i) {
    console.log(v)
  }
}
```

    12:46:04	お知らせ	実行開始
    12:46:06	情報	10
    12:46:06	情報	20
    12:46:06	情報	30
    12:46:06	情報	40
    12:46:06	情報	15
    12:46:06	情報	25
    12:46:06	情報	50
    12:46:06	情報	60
    12:46:06	情報	70
    12:46:06	情報	80
    12:46:06	情報	90
    12:46:07	情報	55
    12:46:07	情報	65
    12:46:07	情報	75
    12:46:07	情報	40
    12:46:07	情報	45
    12:46:07	情報	55
    12:46:07	情報	85
    12:46:07	情報	95
    12:46:07	情報	10
    12:46:07	情報	60
    12:46:06	お知らせ	実行完了

不特定多数のフェッチ処理を n 件毎にまとめることで、(サーバーの負荷状況などにもよりますが)一般的には受信待ちの時間が短縮されます。このような処理はループ構造が多段になりがちですが、Generator 関数で記述することにより「最終的なレコードを処理するループ」は簡素化していけると思います。

## 非同期 Generator 関数

Generator 関数は上記のようになりましたが、非同期の Generator 関数というものもあり、これを利用できると「GAS 以外の環境で使うコードを共有しやすくなる」という利点があります。

そこで、非同期 Generator 関数についても簡単に調べてみます。

@[card](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/async_function*)

### GAS で非同期処理コードの記述

GAS は基本的に同期処理なのですが、試した限りでは `Promise` などを使ってもエラーとはならないです。

また、非同期の用の構文などもスクリプトエディターでは認識されるようです。

**図 4-1 エディター上では非同期な構文も認識される**

![スクリプトエディターで async 関数を呼び出す記述をしているとき、関数が返す Promise 型についてホバー表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1824467204ee447db7caa0e50e7cc59c/generator-function-in-google-apps-sciript-async-annotate.png?w=340\&h=130\&auto=compress%2Cformat)

そして、スクリプトエディターでの Run や Google スプレッドシートのカスタム関数として非同期な関数を指定すると await した状態になりました。

**リスト 4-1 非同期な動作となるカスタム関数**

```js
function RETURN_PROMISE(){
  return Promise.resolve('あいうえお')
}

async function ASYNC_FUNC(){
  return 'かきくけこ'
}
```

**図 4-2 非同期なカスタム関数の結果表示**

![Google スプレッドシート上で上記リストの関数を利用しているスクリーンショット、結果は await された値が表示されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1ead9c35c268469db8f23472e35223b5/generator-function-in-google-apps-sciript-async-custom-func-in-sheets.png?w=548\&h=170\&auto=compress%2Cformat)

ただし、エラーにはならなというだけで、ドキュメントを軽く確認した限りでは非同期な関数の扱いについての記述はなさそうでした。

また、非同期処理による平行化などの利点はおそらくないです。 (利点があるとすれば、前述のように他環境とのコードを共有しやすいという点です)

### 非同期 Generator 関数を普通に記述するとエラーだがプロパティとしては利用できた

では、非同期 Generator 関数が使えるのかというと、少し微妙な結果となります。

まず、これを書いている時点(2025/09)では非同期 Generator 関数を `async function*` で記述すると、スクリプトエディターでの保存時にエラーとなります。

**図 4-3 エラーが表示されて保存できない**

![スクリプトエディター上で構文エラーのポップアップが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/aa7146a33a8a42768e6dbc6691551442/generator-function-in-google-apps-sciript-error-async-generator-function.png?w=854\&h=277\&auto=compress%2Cformat)

しかし、理由は不明ですが、オブジェクトプロパティとして記述するとエラー表示はされず利用できます。
(`async *[Symbol.asyncIterator]()` でもエラーにならなかったです)

**リスト 4-2 オブジェクトプロパティとしての例**

```js
async function myFunction() {
  const obj = {
    async *gen() {
      let c = 0;
      yield Promise.resolve(`あいうえお: ${c++}`)
      yield Promise.resolve(`かきくけこ: ${c++}`)
      yield Promise.resolve(`さしすせそ: ${c++}`)
    }
  }

  for await (const w of obj.gen()) {
    console.log(w)
  }
}
```

    21:42:31	お知らせ	実行開始
    21:42:31	情報	あいうえお
    21:42:31	情報	かきくけこ
    21:42:31	情報	さしすせそ
    21:42:32	お知らせ	実行完了

しかし、実行はできるけれども、スクリプトエディターでは非同期 Generator 関数として認識されないようです。

**図 4-4 エディターでは正しく認識されない**

![スクリプトエディターで非同期 Generator 関数を呼び出す記述をしているとき、正しくない戻り値がホバー表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/851c3bc4399648408d3a6ec03ced4f56/generator-function-in-google-apps-sciript-not-annotate.png?w=536\&h=195\&auto=compress%2Cformat)

このような感じなので、「たまたま使えるだけ」というくらいに考えていた方がよさそうです。

### 他環境とコードを共有しやすくなるとは

たとえば、前述の `fetchAll` を利用しているコード(リスト 3-2)を Node.js 環境でも利用したいとします。

その場合、まず以下のように非同期 Generator 関数を使った処理へ変更します。 (本来なら必要のない Promise なオブジェクトを何個も作ることになるので効率は悪くなると思います)

**リスト 4-3 ストリーム的な処理を非同期 Generator 関数で作成**

```js
function* buffered_(urlsArray, count) {
  let s = []
  for (const i of urlsArray) {
    s.push(i)
    if (s.length == count) {
      yield s
      s = []
    }
  }
  if (s.length > 0) {
    yield s
  }
}

const streamed_ = {
  async* gen(urlsArray, count) {
    for (const urls of buffered_(urlsArray, count)) {
      const resAll = UrlFetchApp.fetchAll(urls)
      for (const res of resAll) {
        const arr = JSON.parse(res.getContentText()).args.arr
        for (const v of JSON.parse(arr)) {
          yield v
        }
      }
    }
  }
}

async function myFunction() {
  const count = 3
  const i = streamed_.gen([
    'https://httpbin.org/get?arr=[10,20,30,40]',
    'https://httpbin.org/get?arr=[15,25]',
    'https://httpbin.org/get?arr=[50,60,70,80,90]',
    'https://httpbin.org/get?arr=[55,65,75]',
    'https://httpbin.org/get?arr=[40]',
    'https://httpbin.org/get?arr=[45,55,85,95]',
    'https://httpbin.org/get?arr=[10,60]'
  ], count)
  for await (const v of i) {
    console.log(v)
  }
}
```

そして、非同期 Generator 関数である `streamed_.gen` を以下のように変更します。これで Node.js でも動作します。

**リスト 4-4 非同期 Generator 関数を Node.js 用に変更**

```js
const streamed_ = {
  async* gen(urlsArray, count) {
    for (const urls of buffered_(urlsArray, count)) {
      const resAll = await Promise.all(urls.map(url => fetch(url)));
      for (const res of resAll) {
        const arr = JSON.parse(await res.text()).args.arr
        for (const v of JSON.parse(arr)) {
          yield v
        }
      }
    }
  }
}
```

上記の Node.js 用コード(リスト 4-4)は `fetchAll` との比較用に `Promise.all` を使っていますが、もう少し待ち時間を減らせるコードにもできます。

いずれにしても、読み出し側から見た非同期 Generator の挙動が同じであればコードを共有しやすくなります。

### トランスパイラで `async function*` を利用

上記のようにオブジェクトプロパティでの記述で非同期 Generator 関数も利用できますが、「自分が記述しているコード」に限定されます。

たとえば、バンドル用ツールを使って NPM をパッケージを利用している場合、パッケージ内で `async function*` が記述されているとエラーになります。

この場合の対応として、ターゲットに非同期 Generator 関数が存在しない古い環境を指定することが考えられます。 (ただし、GAS ネイティブで利用できる構文も変換されるので効率は悪くなるかと思います)

以下、TypeScript でターゲットに ES2017 を指定した例です。

**図 4-5 TypeScript を ES2017 の ****`.js`**** へトランスパイル**

```shell
tsc --target es2017 src/foo.ts
```

**リスト 4-5 単純な ****`async function*`**** のコード**

```ts
async function* Gen(): AsyncGenerator<number, void, unknown> {
    yield 1;
    yield 2;
    yield 3;
}
```

**リスト 4-6 変換されたコード**

```js
var __await = (this && this.__await) || function (v) { return this instanceof __await ? (this.v = v, this) : new __await(v); }
var __asyncGenerator = (this && this.__asyncGenerator) || function (thisArg, _arguments, generator) {
    if (!Symbol.asyncIterator) throw new TypeError("Symbol.asyncIterator is not defined.");
    var g = generator.apply(thisArg, _arguments || []), i, q = [];
    return i = Object.create((typeof AsyncIterator === "function" ? AsyncIterator : Object).prototype), verb("next"), verb("throw"), verb("return", awaitReturn), i[Symbol.asyncIterator] = function () { return this; }, i;
    function awaitReturn(f) { return function (v) { return Promise.resolve(v).then(f, reject); }; }
    function verb(n, f) { if (g[n]) { i[n] = function (v) { return new Promise(function (a, b) { q.push([n, v, a, b]) > 1 || resume(n, v); }); }; if (f) i[n] = f(i[n]); } }
    function resume(n, v) { try { step(g[n](v)); } catch (e) { settle(q[0][3], e); } }
    function step(r) { r.value instanceof __await ? Promise.resolve(r.value.v).then(fulfill, reject) : settle(q[0][2], r); }
    function fulfill(value) { resume("next", value); }
    function reject(value) { resume("throw", value); }
    function settle(f, v) { if (f(v), q.shift(), q.length) resume(q[0][0], q[0][1]); }
};
function Gen() {
    return __asyncGenerator(this, arguments, function* Gen_1() {
        yield yield __await(1);
        yield yield __await(2);
        yield yield __await(3);
    });
}
```

利用しているバンドルツールがパッケージ内のコードのターゲットなども変換できるなら、このようなターゲットの指定で対応できるかと思います。

以下は実際に「[esbuild](https://esbuild.github.io/) で依存パッケージ内の `async function*` を変換し利用している」GAS ライブラリーのリポジトリです。

@[card](https://github.com/hankei6km/gas-notion2content)

ただし、注意点もあります。上記例(リスト 4-6)の場合では `Symbol.asyncIterator` に依存したコードとなるため、GAS 環境への依存は完全には排除されません。よって、GAS 側の変更で動作しなくなる可能性はあります。 (GAS 環境では `Symbol.asyncIterator` を利用はできるが、スクリプトエディターでは型定義がない状態です)

::: message

[Rollup](https://rollupjs.org/) では(しばらく前に試したときは)設定だけでは対応できず、babel プラグインが必要となりました。その辺もあって、普段使っている GAS 用のスターターは Rollup から esbuild へ移行しました。

@[card](https://github.com/hankei6km/my-starter-gas-lib-ts)

:::

## おわりに

Google Apps Script で Generator 関数を試してみました。

GAS で正式にサポートされているかは微妙なところですが(とくに非同期関連)、それなりに動作することはわかりました。

GitHub Copilot からの返答では「普通ではない」と言われてしまいますが、GAS でも利用していこうと思います。
