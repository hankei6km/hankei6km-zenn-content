---
id: swap-variables-xor-js-destructuring-assignment
title: JavaScript で値の入れ替え(XOR 交換、分割代入)
emoji: 🔀
type: tech
topics:
  - javascript
  - algorithm
published: true
---

下記の記事を見て、スタックが小さいデバイスで値を入れ替えるのにあれこれ捻くりまわしていた記憶がよみがえってきたので。

@[card](https://zenn.dev/qnighy/articles/a62e5c2a6ba8ef)

あとは、いまどきの JavaScript では入れ替えに使える構文があるのですが、いつも忘れてしまうのでその辺のメモとして。

## 数値を一時変数無しで入れ替える(XOR 交換アルゴリズム)

とくに JavaScript に特化した方法はありませんが、冒頭で書いた「スタックが小さいデバイス」用に使おうかと思った方法です(当時は C 言語で使おうかと思ってました)。

@[card](https://ja.wikipedia.org/wiki/XOR%E4%BA%A4%E6%8F%9B%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0)

JavaScript だと下記の記事の [3. Addition and difference](https://dmitripavlutin.com/swap-variables-javascript/#3-addition-and-difference) と [4. Bitwise XOR operator](https://dmitripavlutin.com/swap-variables-javascript/#4-bitwise-xor-operator) になります。

@[card](https://dmitripavlutin.com/swap-variables-javascript/)

### 試してみる

XOR 交換アルゴリズムは難しそうな名前ですが、コード自体はシンプルなので少し試してみます。

**リスト 1-1 JavaScript の場合**

```js
let a = -10;
let b = 3;

console.log(`${a}, ${b}`);

a = a ^ b;
b = a ^ b;
a = a ^ b;

console.log(`${a}, ${b}`);

// -10, 3
// 3, -10
```

### 他の言語では

もちろんいろいろな言語で使えますよということで。

**リスト 1-2 Rust の場合**

```rust
fn main() {
    let mut a = -10;
    let mut b = 3;

    println!("{}, {}", a, b);

    a = a ^ b;
    b = a ^ b;
    a = a ^ b;

    println!("{}, {}", a, b);

    // -10, 3
    // 3, -10
}
```

### ハマりどころ

上記のようにシンプルですが、注意点として[前述の記事](https://dmitripavlutin.com/swap-variables-javascript/)にあるように XOR 交換では**型が整数で表現されるもの**に制限されます。

その他、わりとハマりやすそうなのは**同じ変数同士で実施すると 0 になる**ことです。

> 一方でそのように掩蔽すると、2個の引数が同じ場所を表す式であった場合に、両方とも0になってしまう、…

@[card](https://ja.wikipedia.org/wiki/XOR%E4%BA%A4%E6%8F%9B%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0#%E5%AE%9F%E9%9A%9B%E3%81%AE%E4%BD%BF%E7%94%A8%E6%B3%95)

「いやいや、同じ変数を入れ替えるとかやらんだろ」となりそうですが、以下のようなこともあるかと(あとは配列の部分入れ替えで同じ添え字を使ってしまうなど)。

**リスト 1-3 異なるオブジェクトのフィールド同士**

```js
const o1 = {p: -10};
const o2 = {p: 3};

console.log(o1, o2);

o1.p = o1.p ^ o2.p;
o2.p = o1.p ^ o2.p;
o1.p = o1.p ^ o2.p;

console.log(o1, o2);

// { p: -10 } { p: 3 }
// { p: 3 } { p: -10 }
```

**リスト 1-4 実は同一オブジェクトのフィールド同士**

```js
const o1 = {p: -10};
const o2 = o1;

console.log(o1, o2);

o1.p = o1.p ^ o2.p;
o2.p = o1.p ^ o2.p;
o1.p = o1.p ^ o2.p;

console.log(o1, o2);

// { p: -10 } { p: -10 }
// { p: 0 } { p: 0 }
```

## 分割代入を使う

これも[前述の記事](https://dmitripavlutin.com/swap-variables-javascript/)にでていますが、分割代入を使って入れ替える方法です。便利なのですが慣れてないと「どうやるんだっけ」となるのが悩ましいところ。

@[card](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

**リスト 2-1 変数の入れ替え**

```js
let a = 'one'
let b = 'two';

console.log(`${a}, ${b}`);

[b, a] = [a, b];

console.log(`${a}, ${b}`);

// one, two
// two, one
```

**リスト 2-2 配列の一部を入れ替え**

```js
const a = [1, 2, 3];

console.log(a);

[a[2], a[0]] = [a[0], a[2]];

console.log(a);

// [ 1, 2, 3 ]
// [ 3, 2, 1 ]
```

**リスト 2-3 オブジェクトのフィールドを一部入れ替え**

```js
let o = {a: 1, b: 2, c: 3}

console.log(o);

[o.b, o.a] = [o.a, o.b];

console.log(o);

// { a: 1, b: 2, c: 3 }
// { a: 2, b: 1, c: 3 }
```

**リスト 2-4 同一オブジェクトのフィールドを入れ替えても 0 にはならない**

```js
const o1 = {p: -10};
const o2 = o1;

console.log(o1, o2);

[o1.p, o2.p] = [o2.p, o1.p];

console.log(o1, o2);

// { p: -10 } { p: -10 }
// { p: -10 } { p: -10 }
```

## おわりに

JavaScript で値を入れ替える方法を試してみました。

XOR 交換は「特殊な事情がなければ利用することは少ないかな[^embedded]」という感じですが、分割代入は「値の入れ替え」の他にも便利そうなので積極的に使っていきたいなと考えています。

[^embedded]: 組み込み用などで分割代入に対応してない処理系では使うかな？と思ったのですが…。いまどきは対応してそうですね([QuickJS](https://bellard.org/quickjs/) は対応してました)。
