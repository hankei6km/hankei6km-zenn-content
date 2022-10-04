---
id: try-ai-programmer
title: AI Programmer を話のネタにと試してみたら
emoji: ✍️
type: idea
topics:
  - ai
  - プログラミング
published: true
---

以下の記事をみかける。

<!-- textlint-disable -->

@[card](https://forest.watch.impress.co.jp/docs/serial/yajiuma/1444554.html)

> 日本語の文章を元にAIがソースコードを書いてくれるサービス「AI Programmer」が公開されたのだそうです。最近は自然言語で指示した通りに絵を描いてくれる[「Stable Diffusion」](https://forest.watch.impress.co.jp/docs/special/1437123.html)や[「DALL·E」](https://forest.watch.impress.co.jp/docs/news/1443392.html)などのサービスが人気で、いつかはこうしたものも出てくるのだろうとは思っていましたが、意外に早かったですね……。

## 試してみる

今回は JavaScript で試してみます。

@[card](https://aiprogrammer.hashlab.jp/)

:::message
下記の各入力に対して生成されるコードは記事を書いている時点でのものです。
:::

まずは普通にちょっとした処理を頼んでみる。

**入力:** `文字列を","で分割する。`

**出力:**

```js
var text = "文字列を、で分割する。";
var textByComma = text.split(",");
console.log(textByComma);
```

`var` を使いたくない場合は `ES6以上で` などを加えるとよいそうです。

@[card](https://zenn.dev/riya_amemiya/articles/db4fb02ecf57a6)

**入力:** `ES6以上で文字列を","で分割する。`

```js
const string = "ES6以上で文字列を、で分割する。";
const array = string.split(",");
```

とりあえずできました。  「ではコレはどうかな」と外部パッケージを使うように頼んでみます。

**入力:** `node-fetchパッケージを使って"https://aiprogrammer.hashlab.jp/"からテキストをGETする。`

**出力:**

```js
const fetch = require('node-fetch');

fetch('https://aiprogrammer.hashlab.jp/')
  .then(response => response.text())
  .then(text => console.log(text));
```

「え？できちゃうの？」

実際には `import` にして `.mjs` にしないと動かないのですが、それでも驚きです。少し試した感じではここまでうまくいくケースは少ない感じですが、 外部パッケージとかも認識するもよう。

ここまでくると「Jest も使えたりしちゃいますか？」とお願いしてみます。

**入力:** `Jestで関数をテストしてください。`

**出力:**

```js
test('function to be tested', () => {
  expect(function()).toBe();
});
```

**入力:** `Jestで非同期関数をテストしてください。`

**出力:**

```js
async function testAsync () {
  // do something async
}

test('test async function', async () => {
  await testAsync();
  // async test passes
})
```

**入力:** `Jestで"ABC"クラスの"method"をspyしてください。`

**出力:**

```js
jest.spyOn(ABC.prototype, 'method');
```

非同期関数はちょっとアレですが「あ、やっぱりイケるんですね」という感じでした。

完全に動作するコードができるわけではないですが「えーと、spy ってどうやるんだっけ」という場合などのヒントにはなりそうです。

<!-- textlint-enable -->

## API シークレットとかは？

GitHub Copilot では下記のような記事もあったので少し試してみます。

@[card](https://forest.watch.impress.co.jp/docs/serial/yajiuma/1336988.html)

**入力:** `node-fetchでAPIキーを使ったGETを行う。`

**出力:**

```js
var fetch = require("node-fetch");

fetch("https://example.com/api/v1/endpoint", {
  headers: {
    "X-API-KEY": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  }
})
.then(function(response) {
  return response.json();
})
.then(function(json) {
  console.log(json);
});
```

どういうルールでマスクしているのかは不明ですが、先行事例で問題視されている部分はクリアしてある感じでしょうか(だといいなという願望)。

## おわりに

簡単にですが AI Programmer を試してみました。

今回は JavaScript で少しお願いしてみただけですが「あれってどうやるんだっけ？」みたいなときにサポートしてくれそうな感じです。 (感覚的には「ググるとコードが表示される」といったところでしょうか)

API が公開されて[^api]、各種エディターの拡張機能が作られると面白そうです。

[^api]: 探してはみたのですが現時点(2022-10-04)では公開されていなさそうです。
