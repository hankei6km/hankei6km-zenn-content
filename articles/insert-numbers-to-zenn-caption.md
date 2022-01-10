---
id: insert-numbers-to-zenn-caption
title: Zenn のキャプション表示などに連番を振る remark プラグイン
emoji: 🔢
type: idea
topics:
  - zenn
  - markdown
  - unified
  - remark
published: true
---

記事の画像に Zenn のキャプション表示を使うようにしていたら連番を付けたくなりました。

少し考えた結果「キャプションに限らず Markdown 内で連番を割り振る remark プラグイン」を作ってみました。

今回はこのプラグインをコマンドとしてインストールし 、Zenn 記事の作成に利用してみます。

@[card](https://github.com/hankei6km/remark-numbers)

## 概要

[`remark-numbers`](https://github.com/hankei6km/remark-numbers) の主な機能は「見出しに反応するカウンター」と「番号が割り当てられる変数」です。

この 2 つを利用して以下のような連番(写真 1-1、写真 1-2)を割り振っていきます。

![夕暮れの河川で中央に何かの設備が写っている写真](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c0dbed84c5a14217bd43fabfe884a720/insert-numbers-to-zenn-caption-twilight-river.jpg?w=400\&h=200\&auto=compress%2Cformat)*写真 1-1 夕暮れの河川*

![夕焼け空の手前に車道の通っている橋が写っている写真](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6f2b485fffa94f25a35fd378be98d5e9/insert-numbers-to-zenn-caption-sunset-bridge.jpg?w=400\&h=200\&auto=compress%2Cformat)*写真 1-2 夕焼けと橋*

## インストール

`remark-numbers` をコマンドとして利用するには `remark-cli` が必要となります。Zenn 用に各種プラグインもインストールするので npm のパッケージを作ってその中にインストール(設定)していきます(図 2-1)。

▼ *図 2-1 インストール*

```shell-session
$ mkdir numbers
$ cd numbers
$ npm init -y
$ npm install remark-cli remark-directive remark-frontmatter remark-gfm @hankei6km/remark-numbers
```

## 設定

`remark-cli` の設定ファイル(`.remarkrc.mjs` )を以下のように作成します(リスト 3-1)。

今回は連番表示を「図 3-n」のようにする書式を設定しています。

▼ *リスト 3-1 設定ファイル*

```js:.remarkrc.mjs
import remarkDirective from 'remark-directive'
import remarkFrontmatter from 'remark-frontmatter'
import remarkGfm from 'remark-gfm'
import remarkNumbers from '@hankei6km/remark-numbers'

const remarkConfig = {
  plugins: [
    remarkDirective,
    remarkFrontmatter,
    remarkGfm,
    [
      remarkNumbers,
      {
        template: [
          `
:::num{reset assign}
## :num
:::
:::num{format assign}
:num[図 :num]{series=fig}
:num[図 :num[sec]-:num]{series=fig}
:num[写真 :num]{series=photo}
:num[写真 :num[sec]-:num]{series=photo}
:num[グラフ :num]{series=chart}
:num[グラフ :num[sec]-:num]{series=chart}
:num[グラフ :num]{series=graph}
:num[グラフ :num[sec]-:num]{series=graph}
:num[図式 :num]{series=diagram}
:num[図式 :num[sec]-:num]{series=diagram}
:num[フロー :num]{series=flow}
:num[フロー :num[sec]-:num]{series=flow}
:num[表 :num]{series=tbl}
:num[表 :num[sec]-:num]{series=tbl}
:num[リスト :num]{series=list}
:num[リスト :num[sec]-:num]{series=list}
:::

:::num{reset assign name=simple}
## :num{delete}
:::
:::num{format assign name=simple}
:num[図 :num]{series=fig}
:num[写真 :num]{series=photo}
:num[グラフ :num]{series=chart}
:num[グラフ :num]{series=graph}
:num[図式 :num]{series=diagram}
:num[フロー :num]{series=flow}
:num[表 :num]{series=tbl}
:num[リスト :num]{series=list}
:::
      `
        ],
        keepDefaultTemplate: true
      }
    ]
  ]
}

export default remarkConfig
```

## 実行

`remark-cli` をインストールしたディレクトリーで以下のようにコマンドを実行します(図 4-1)。[^sed]。今回の例ではソースは `source-numbers.md` ファイルとして入力し、変換結果は `transformed_numbers.md` ファイルへ保存されます。

▼ *図 4-1 実行例*

```shell-session
$ npx remark --quiet < source-numbers.md | sed -e 's/^\\:::/:::/' > transformed-numbers.md
```

保存されたファイルを zenn-cli のプレビュー用ディレクトリーへコピーすれば連番が割り振られた状態で表示されます。

[^sed]: `sed` は微調整のために利用しています。 <https://zenn.dev/link/comments/41643c6a036269>

## 記述

Markdown ファイル内で連番を割り振るための記述方法です。

### 基本

`remark-numbers` は [micromark-extension-directive](https://github.com/micromark/micromark-extension-directive) の[構文](https://github.com/micromark/micromark-extension-directive#syntax)を利用しているので、基本的な連番の割り振りは以下のように記述します。

*   `:num{#定義する変数}` - `id` 属性として変数を定義する
*   `:num[参照する変数]` - 参照したい変数の `id` をラベルとして記述

### 変数の定義

Markdown 内で番号を割り振りたい場所に `:num{#fig-secret-set}` のように記述します。これは `id` が `fig-secret-set` の変数を定義し連番を割り振ります。

たとえば画像のキャプション部分では以下のようにします(図 5-1)。

▼ *図 5-1 定義の記述*

```markdown
![GitHub リポジトリに secret が設定されたスクリーンショット](/images/screenshot-set.png)
*:num{#fig-secret-set} 設定された secret*
```

これは以下のように変換されます。今回は書式を設定しているのでラベルやセクション番号が付与されています(図 5-2)。

▼ *図 5-2 定義の変換結果*

```markdown
![GitHub リポジトリに secret が設定されたスクリーンショット](/images/screenshot-set.png)
*図 5-1 設定された secret*
```

### 変数を参照

参照する場合は `:num[fig-secret-set]` のように `[]` 内に `id` を記述します(図 5-3)。

▼ *図 5-3 参照の記述*

```markdown
コマンドを実行することで secret が設定されました(:num[fig-secret-set])。
```

これは以下のように変換されます。参照時にも書式が適用されています(図 5-4)。

▼ *図 5-4 参照の変換結果*

```markdown
コマンドを実行することで secret が設定されました(図 5-1)。
```

### セクション 0

深さ 2 の見出し(`##` ) が現れる前のセクション用カウンターは `0` になっています。この場合は「図 0-n」ではなく「図 n」のように表示されます(図 5-5、図 5-6)。

▼ *図 5-5 セクション 0 位置での記述*

```markdown
# test

今季の傾向です(:num[chart-overview])

![Overview](/images/chart-overview.png)
*:num{#chart-overview}*
```

▼ *図 5-6 セクション 0 位置での変換結果*

```markdown
# test

今季の傾向です(グラフ 1)

![Overview](/images/chart-overview.png)
*グラフ 1*
```

### 複数系列の利用

変数の `-` より前の部分は系列名として扱われます。

以下は `chart` 系列と `photo` 系列を利用しているサンプルです(図 5-7)。系列を複数利用している場合、書式や連番は系列別に処理されます(図 5-8)。

▼ *図 5-7 複数系列の記述*

:::details (入力)

```markdown
## Fruits

資料(:num[chart-fruits]) によると Orange(:num[photo-orange]) が好調です。

![Fruits](/images/chart-fruits.png)
*:num{#chart-fruits}*

![Apple](/images/apple.jpg)
*:num{#photo-apple}*

![Banana](/images/banana.jpg)
*:num{#photo-banana}*

![Orange](/images/orange.jpg)
*:num{#photo-orange}*

## Drink

資料(:num[chart-drink]) によると Tea(:num[photo-tea]) は Apple(:num[photo-apple]) とのセットメニューが好調です。

![Drink](/images/chart-drink.png)
*:num{#chart-drink}*

![Coffee](/images/coffee.jpg)
*:num{#photo-coffee}*

![Tea](/images/tea.jpg)
*:num{#photo-tea}*
```

:::

▼ *図 5-8 複数系列の変換結果*

:::details (結果)

```markdown
## Fruits

資料(グラフ 1-1) によると Orange(写真 1-3) が好調です。

![Fruits](/images/chart-fruits.png)
*グラフ 1-1*

![Apple](/images/apple.jpg)
*写真 1-1*

![Banana](/images/banana.jpg)
*写真 1-2*

![Orange](/images/orange.jpg)
*写真 1-3*

## Drink

資料(グラフ 2-1) によると Tea(写真 2-2) は Apple(写真 1-1) とのセットメニューが好調です。

![Drink](/images/chart-drink.png)
*グラフ 2-1*

![Coffee](/images/coffee.jpg)
*写真 2-1*

![Tea](/images/tea.jpg)
*写真 2-2*
```

:::

### 書式の切り替え

セクション番号が不要の場合、今回の設定では Front Matter に `numGroupName: simple` を指定するとセクション番号無しの書式に切り替わります(図 5-9)。

▼ *図 5-9 書式切り替えのサンプル*

```markdown
---
title: Zenn のキャプション表示などに連番を振る remark プラグイン
emoji: 🔢
type: idea
topics:
  - zenn
  - markdown
  - unified
  - remark
published: false
numGroupName: simple
---
# test

```

## サンプル表示

### 図と写真のキャプション

:::message
以下はキャプション内で利用したサンプルです(内容にとくに意味はありせん)。
:::

:::details (サンプルソース)

```markdown
コマンドを実行することで secret が設定されました(:num[fig-secret-set])。また、削除もできます(:num[fig-secret-remove])。

![GitHub リポジトリに secret が設定されたスクリーンショット](/images/screenshot-set.png)
*:num{#fig-secret-set} 設定された secret*

![GitHub リポジトリから secret が削除されたスクリーンショット](/images/screenshot-remove.png)
*:num{#fig-secret-remove} 削除された secret*
```

:::

コマンドを実行することで secret が設定されました(図 6-1)。また、削除もできます(図 6-2)。

![GitHub リポジトリに secret が設定されたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8dd3354448074d07a13d840d27d7077e/insert-numbers-to-zenn-caption-screenshot-set.png?w=800\&h=169\&auto=compress%2Cformat)*図 6-1 設定された secret*

![GitHub リポジトリから secret が削除されたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7ecad8af133a4cf8a259ea0b9530ac8a/insert-numbers-to-zenn-caption-screenshot-remove.png?w=799\&h=238\&auto=compress%2Cformat)*図 6-2 削除された secret*

### 各種系列のサンプル

:::message
以下はキャプション以外で利用したサンプルです(内容にとくに意味はありせん)[^try]。
:::

[^try]: 表などのラベル的な表示はまだ試行錯誤しているところです。

:::details (サンプルソース)

    関数の実装方法がいつくか出てきたのでベンチ用のテストコードを作成しました(:num[list-bench])。

    ▼ *:num{#list-bench} 関数のベンチマーク*

    ```typescript:bench.ts
    import { unified } from 'unified'
    import remarkParse from 'remark-parse'
    import remarkStringify from 'remark-stringify'
    import remarkDirective from 'remark-directive'
    import remarkFrontMatter from 'remark-frontmatter'
    import { remarkNumbers, RemarkNumbersOptions } from '../../src/lib/numbers.js'

    // ここにベンチマークのコードが記述されているとする。
    ```

    以下はベンチをとってみた結果です(:num[tbl-bench-elapsed]、:num[tbl-bench-mem])、`FUNC3` は回数が増えると実行時間は短縮される特性がわかりました(:num[graph-bench-elapsed])。

    ▼ *:num{#tbl-bench-elapsed} ベンチマーク(実行時間)*

    | 回数  | FUNC1 | FUNC2 | FUNC3 |
    | --- | ----- | ----- | ----- |
    | 1   | 300   | 400   | 600   |
    | 10  | 2000  | 4500  | 4000  |
    | 50  | 9000  | 25000 | 15000 |
    | 100 | 30000 | 50000 | 20000 |

    ▼ *:num{#tbl-bench-mem} ベンチマーク(メモリー)*

    | 回数  | FUNC1 | FUNC2 | FUNC3 |
    | --- | ----- | ----- | ----- |
    | 1   | 300   | 400   | 600   |
    | 10  | 2000  | 4500  | 1000  |
    | 100 | 18000 | 50000 | 9000  |

    ![ベンチマーク(実行時間)の折れ線グラフ、縦軸に実行時間、横軸に実行回数](/images/graph-bench-elapsed.png)
    *:num{#graph-bench-elapsed} ベンチマーク(実行時間)*

    また、`FUNC3` では実行後の各要素は直線的な関係になります(:num[diagram-cap])

    ```mermaid
    graph LR
      A[要素 A] --> B[要素 B]
      B --> C[要素 C]
    ```

    ▲ *:num{#diagram-cap} 要素同士の関係*

:::

関数の実装方法がいつくか出てきたのでベンチ用のテストコードを作成しました(リスト 6-1)。

▼ *リスト 6-1 関数のベンチマーク*

```typescript:bench.ts
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkStringify from 'remark-stringify'
import remarkDirective from 'remark-directive'
import remarkFrontMatter from 'remark-frontmatter'
import { remarkNumbers, RemarkNumbersOptions } from '../../src/lib/numbers.js'

// ここにベンチマークのコードが記述されているとする。
```

以下はベンチをとってみた結果です(表 6-1、表 6-2)、`FUNC3` は回数が増えると実行時間は短縮される特性がわかりました(グラフ 6-1)。

▼ *表 6-1 ベンチマーク(実行時間)*

| 回数  | FUNC1 | FUNC2 | FUNC3 |
| --- | ----- | ----- | ----- |
| 1   | 300   | 400   | 600   |
| 10  | 2000  | 4500  | 4000  |
| 50  | 9000  | 25000 | 15000 |
| 100 | 30000 | 50000 | 20000 |

▼ *表 6-2 ベンチマーク(メモリー)*

| 回数  | FUNC1 | FUNC2 | FUNC3 |
| --- | ----- | ----- | ----- |
| 1   | 300   | 400   | 600   |
| 10  | 2000  | 4500  | 1000  |
| 100 | 18000 | 50000 | 9000  |

![ベンチマーク(実行時間)の折れ線グラフ、縦軸に実行時間、横軸に実行回数](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/674a636344be4afa8eb6c2861b9e9e76/insert-numbers-to-zenn-caption-bench-graph.png?w=600\&h=371\&auto=compress%2Cformat)*グラフ 6-1 ベンチマーク(実行時間)*

また、`FUNC3` では実行後の各要素は直線的な関係になります(図式 6-1)

```mermaid
graph LR
  A[要素 A] --> B[要素 B]
  B --> C[要素 C]
```

▲ *図式 6-1 要素同士の関係*

## Zenn 記事で利用する場合の注意点

「何らかのツールで Markdown を変形させてから GitHub へ push した場合、プルリクエストをいただいてもマージできない」という問題が出てきます[^pr]。

[^pr]: ソースではなくビルドしたものにプルリクエストをいただくのと同じような状況になる。

対応方法としては「リポジトリの `src/articles/*.md` などに変換前のファイルを含め、ツールの実行手順なども開示しておく」といったことが考えられます[^src]。

[^src]: なのですが、私の場合は「記事を Headless CMS で作成」しているため、この辺の対応は棚上げになってる状態です。

## おわりに

長々と書いてしまいましたが、(書式の設定などをいじらずに)連番を振っていくだけならリンクの定義や注釈を記述するような感覚でいけるのではと思っています。

また、個人的にはわりと気に入っているプラグインなので、ゆるゆると機能追加していきたいなとも考えています。

## スクラップ

`remark-numbers` 用に作成したスクラップです。

@[card](https://zenn.dev/hankei6km/scraps/12ff6acbcbcde4)

@[card](https://zenn.dev/hankei6km/scraps/110914459760d5)

@[card](https://zenn.dev/hankei6km/scraps/b8ea620cf513d0)
