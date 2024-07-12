---
id: inline-chat-for-text-processing-tasks
title: 少し込み入ったテキストの加工に GitHub Copilot のインラインチャットを使う
emoji: 📝
type: idea
topics:
  - githubcopilot
published: true
---

編集中のテキストに少し込み入った加工(置換や整形)をしたくなることがあります。

このようなとき、正規表現や Vim(または [VSCodeVim](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim)) の[レコーディング](https://vimhelp.org/repeat.txt.html#complex-repeat) (q マクロ)を使うのですが、GitHub Copilot のインラインチャットを使うことも増えました。

インラインチャットでも工夫するとなんとなく意図した通りの加工ができるようになってきたので、その辺についてなど。

## 想定している読者

*   正規表現やレコーディングを使う方法も好きだけど、たまには違うことも試したいという人
*   GitHub Copilot のインラインチャットでテキストの整形をしたい人

## 加工してみる

基本的にはインラインチャットで指示するだけですが、試してみた限りでは以下のようにすると意図した結果を得やすかったです。

*   プロンプトに例を含める
*   加工するためのコードが生成されたときは、そのコードをチャットで実行する

以下、実際に使ってみた例です。

### 特定の場所の装飾などを変更する

少し説明が難しいのですが、以下のような場合です。

> 以下のような Markdown をファイル編集していて、あとから「見出しのオプション名はコードで装飾するはずだった」と気がついた。

**リスト 2-1これを**

```md
#### Options

##### inOpts.target

`props` と `content` のどちらを変換するか配列で指定(同時にどちらも指定可能)

##### inOpts.workersNum

`content` の変換(child ブロックのフェッチ)を並行で行う数。
```

**リスト 2-2 こうしたい**

```md
#### Options

##### `inOpts.target`

`props` と `content` のどちらを変換するか配列で指定(同時にどちらも指定可能)

##### `inOpts.workersNum`

`content` の変換(child ブロックのフェッチ)を並行で行う数。
```

説明が難しいということはプロンプトで指示するのも難しいのですが、今回は以下のようなプロンプトを使いました。

**リスト 2-3 プロンプト**

    選択範囲の見出しを例のように変更してください。例: "##### inOpts.target"を"##### `inOpts.target`"とする

**図 2-1 加工結果**

![指定した通りにテキストが加工されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e138c67bbe8b409a961f95da5a5e9967/few-shot-prompting-for-text-processing-tasks-decorate-res.png?w=785\&h=641\&auto=compress%2Cformat)

他には以下のようなコードで色名だけを大文字にする場合などにも使えます。

**リスト 2-4 変換元のリスト**

```ts
    expect(c.props("gray_background")).toEqual({
      style: "background-color:#EBECED",
    });
    expect(c.props("brown_background")).toEqual({
      style: "background-color:#E9E5E3",
    });
    expect(c.props("orange_background")).toEqual({
      style: "background-color:#FAEBDD",
    });
    expect(c.props("yellow_background")).toEqual({
      style: "background-color:#FBF3DB",
    });
```

**リスト 2-5 プロンプト**

    例を参考に変換。例 "gray_background" は "GRAY_background"

**図 2-2 加工結果**

![指定した通りにテキストが加工されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4098875fa5ea4cadaccf554a816e20d1/few-shot-prompting-for-text-processing-tasks-to-upper-res-dot.png?w=785\&h=641\&auto=compress%2Cformat)

### 法則のありそうなテキストを変換する

特定の法則に沿っているぽいテキストを別の形式に変更したくなることもあります。たとえば、以下のコードの引数を抜き出して対応表を作りたい場合などです。

**リスト 2-6元のコード**

```ts
    expect(c.props("gray")).toEqual({ style: "color:#9b9a97" });
    expect(c.props("brown")).toEqual({ style: "color:#64473a" });
    expect(c.props("orange")).toEqual({ style: "color:#d9730d" });
    expect(c.props("yellow")).toEqual({ style: "color:#dfab01" });
    expect(c.props("green")).toEqual({ style: "color:#0f7b6c" });
    expect(c.props("blue")).toEqual({ style: "color:#0b6e99" });
    expect(c.props("purple")).toEqual({ style: "color:#6940a5" });
    expect(c.props("pink")).toEqual({ style: "color:#ad1a72" });
    expect(c.props("red")).toEqual({ style: "color:#e03e3e" });
```

**表 2-1 作りたいテーブル**

| color    | expect                       |
| -------- | ---------------------------- |
| `gray`   | `{ style: "color:#9b9a97" }` |
| `brown`  | `{ style: "color:#64473a" }` |
| `orange` | `{ style: "color:#d9730d" }` |
| `yellow` | `{ style: "color:#dfab01" }` |
| `green`  | `{ style: "color:#0f7b6c" }` |
| `blue`   | `{ style: "color:#0b6e99" }` |
| `purple` | `{ style: "color:#6940a5" }` |
| `pink`   | `{ style: "color:#ad1a72" }` |
| `red`    | `{ style: "color:#e03e3e" }` |

この場合は以下のようなプロンプトにします。

**リスト 2-7 プロンプト**

    例を参考にし、引数を抜き出してテーブルを作成して。

    例:

    入力
    `expect(c.props("gray")).toEqual({ style: "color:#9b9a97" });`
    出力
    | color | expect |
    | -- | -- |
    | `gray` | `{ style: "color:#9b9a97" }`|

前述のコードを `.md` のエディタータブに貼り付けて指示すると以下のようになります。

**図 2-3 加工結果**

![Markdown のコードブロックとしてテーブルが生成されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7f2865bea0194084bba88e90c206cee4/few-shot-prompting-for-text-processing-tasks-to-tb-res.png?w=712\&h=512\&auto=compress%2Cformat)

コードブロックとして生成されているので(先頭に空白がある)少し調整することになりますが、テーブルが完成しました。また、逆にテーブルを他の形式に変換する応用もできます。

### オブジェクトの配列にフィールドを追加定義する

テストデータのオブジェクトにフィールドを追加したいときがわりとあります。これもレコーディングで対応することが多かったのですが、ダミーデータを付加するのは少し面倒だったります。

このようなとき、ある程度法則性があるのならば、例を数個定義しておくとよい感じで補完してくれます。

**リスト 2-8 プロンプト**

    nameフィールドを定義してください。例 [{name:'C-3PO',index:0},{name:'R2-D2',index:1},...]

**図 2-4 加工結果**

![指定した通りにテキストが加工されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/eeea56a479704a6d9b878f7ecd39cfa9/few-shot-prompting-for-text-processing-tasks-fld-res.png?w=766\&h=326\&auto=compress%2Cformat)

オブジェクトのフィールドが増えると例を記述するのが大変ですが、`...` などで省略しても通るようです。

    各要素にnameフィールドを追加してください。
    例:
    [{name:'alice',...},{name:'bob,...},...]

**図 2-5 加工結果**

![指定した通りにテキストが加工されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e2f9a0a8f96547649f2f9cf7c01524b1/few-shot-prompting-for-text-processing-tasks-fld-res-dot.png?w=720\&h=323\&auto=compress%2Cformat)

他には、少し成功率が低い印象ですが、テキスト側に例を含める方法もあります。たとえば、以下のようにいくつか `name` を定義しておきます。

**リスト 2-9 参考になる定義を追加しておく**

```js
const mock = [
  { name: 'C-3PO', index: 0 },
  { name: 'R2-D2', index: 1 },
  { index: 2 },
  { index: 3 },
  { index: 4 },
];
```

プロンプトは以下のように 2 回にわけて指示します。

**リスト 2-10 要素を Copilot に説明してもらう(会話の中に例を登場させる)**

    0と1の要素について説明して

**リスト 2-11 説明された要素を参考に定義してもらう**

    0と1を参考に他の要素にもnameを定義して

**図 2-6 加工結果**

![プロンプトを 2 回に分割して実行した結果のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/97262b24980841f992aa81679d06991b/few-shot-prompting-for-text-processing-tasks-fld-res-text.png?w=768\&h=534\&auto=compress%2Cformat)

これはエディター上で例を記述できるのでやりやすいのですが、前述のように少し成功率が低いので、どちらを使うかは悩ましいところです。

::: message

例に挙げたプロンプトは次の 2 つに相当しそうな気もしますが、少し違う気もします。こういうのを見分けるのは苦手なので断言はしないでおきます。

*   [Few-shotプロンプティング](https://www.promptingguide.ai/jp/techniques/fewshot)
*   [Prompt Chaining](https://www.promptingguide.ai/jp/techniques/prompt_chaining)

:::

### 生成されたコードの出力結果を利用する

データへロジック的な変更を適用するとき、加工するためのコードが生成されることも多いです。たとえば、`age フィールドを乱数にしてください。例: [{age:34,...},{age:48,...},...]` のようにすると、以下のような結果になることもあります。

**図 2-7 コードでランダムを実現するように加工される**

```ts
const mock_age = [
  { index: 0, age: getRandomAge(), name: "alice" },
  { index: 1, age: getRandomAge(), name: "bob" },
  { index: 2, age: getRandomAge(), name: "charlie" },
  { index: 3, age: getRandomAge(), name: "david" },
  { index: 4, age: getRandomAge(), name: "eve" },
];

function getRandomAge() {
  return Math.floor(Math.random() * 100) + 1;
}
```

この場合、インラインチャットではなく通常のチャットで `#selection console.log(mock_age)の出力は？` の結果を表示し、それで置き換えるという方法もあります。

**図 2-8 チャットでコードを評価(実行)**

![チャットの返答にランダムな値で置き換えられたオブジェクトが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b8446b84706e4b1699fe6e46c44d213a/few-shot-prompting-for-text-processing-tasks-fld-res-eval.png?w=757\&h=679\&auto=compress%2Cformat)

また、JSON ファイルで実施すると、データ内にコードを上書きしてくるときもありました。このようなときは、いったん accept してコードをコピペ、undo してからチャットで実行するという方法もあります。

**リスト 2-12 プロンプト**

    "age"フィールドの値を"添字+1*10"に置き換えてください。例: [{"age":10,...},{"age":20,...},...]

**リスト 2-13 JSONファイル内に生成されてしまったコード(****`/path/to`**** は実際のパスが記述されている)**

```python
import json

# JSONファイルのパス
file_path = '/path/to/test.json'

# JSONファイルを読み込む
with open(file_path, 'r') as file:
  data = json.load(file)

# "age"フィールドの値を置き換える
for i, item in enumerate(data):
  item['age'] = (i + 1) * 10

# 結果をJSONファイルに書き込む
with open(file_path, 'w') as file:
  json.dump(data, file, indent=2)
```

上記コードをチャットにコピペし、「これはどのような出力になりますか？」を追加してから実行する。

**図 2-9 チャットでコードを評価(実行)**

![チャットの返答に意図した JSON 文字列が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5c18ca43cd9f4313b7fbd2f8a1d4b1cd/few-shot-prompting-for-text-processing-tasks-json-res-eval.png?w=754\&h=707\&auto=compress%2Cformat)

うまくいかないこともありますが、手間的にはテキスト加工用のコマンド(`sed` など)を利用する場合と同じくらいのノリでいけるかとおもいます。

## おわりに

GitHub Copilot のインラインチャットでテキストを加工するとき、工夫していることを記述してみました。

例を考える(入力する)などは少し手間ですが、従来の方法だと対応が難しいときに利用すると便利かなと思っています。
