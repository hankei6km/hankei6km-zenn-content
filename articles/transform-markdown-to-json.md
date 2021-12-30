---
id: transform-markdown-to-json
title: Markdown を JSON にしてしまえ
emoji: "\U0001F35B"
type: idea
topics:
  - markdown
  - remark
  - json
  - jq
  - jsonata
published: true
---
[remark-cli](https://github.com/remarkjs/remark/tree/main/packages/remark-cli) を試しているとき「[入出力を JSON(AST) で扱える](https://github.com/unifiedjs/unified-args#--tree)」のを見て「これを利用すれば Markdown を [`jq`](https://stedolan.github.io/jq/) などで変形できるのでは」ということで試してみました。

## remark-cli の設定

まずは remark-cli を実行できるように設定していきます。

### インストール

remark のプラグインなどもインストールするので npm のパッケージを作ってその中にインストール(設定)していきます。

```shell-session
$ mkdir test-remark-cli
$ cd test-remark-cli/
$ npm init -y
$ npm install remark-cli
$ npx remark --version
remark: 14.0.2, remark-cli: 10.0.1
```

### 試す

`$ npx remark --quiet < test.md` のようにすると処理された Markdown が出力されます。

:::details (テストデータ)

```markdown:test.md
# テストのファイル

remark-cli のテスト用 Maarkdonw ファイル。

## 整形される様子を確認

段落の区切りを多めにとってみる。



2 つめの段落。

見出しの記述方法
--------

見出しを `---` で記述してみる。
```

:::

```shell-session
$ npx remark --quiet < test.md
# テストのファイル

remark-cli のテスト用 Maarkdonw ファイル。

## 整形される様子を確認

段落の区切りを多めにとってみる。

2 つめの段落。

## 見出しの記述方法

見出しを `---` で記述してみる。
```

表記のゆれなどが除去されたことを確認できます。

### プラグイン

remark-cli だけでは処理できない表記方法もあるので適宜プラグインを利用することになります[^need-plugin]。

[^need-plugin]: わかりやすいところでは FrontMatter 用のプラグインを利用しないと配列がリストになったりします。 <https://zenn.dev/link/comments/de42a25a320a6e>

設定方法についてはいくつかありますが[^variant-use-plugin]、プラグインの数が増えても対応しやすいように今回は設定ファイル([`.remarkrc.mjs`](https://github.com/remarkjs/remark/tree/main/packages/remark-cli#example-config-files-json-yaml-js))を使うことにします。

[^variant-use-plugin]: コマンドラインから [`--use`](https://github.com/unifiedjs/unified-args#--use-plugin) オプションを使った場合。<https://zenn.dev/link/comments/dadab7065c958e>

```js:.remarkrc.mjs
import emoji from 'remark-emoji'

const remarkConfig = {
  plugins: [
    [emoji, {emoticon: true}]
  ]
}

export default remarkConfig
```

```shell-session
$ npm install remark-emoji

$ cat test.md
# テストのファイル

プラグインのテスト :dog: :+1: :-)

$ npx remark --quiet < test.md
# テストのファイル

プラグインのテスト 🐶 👍 😃
```

:::message

`.remarkrc.js` (拡張子が `.js` )を使う場合は `package.json` に `"type": "module"` が必要です。

:::

####

### Zenn

Zenn 記事用の Markdown を扱う場合、試したかぎりでは以下のプラグインを利用することになります[^plugins-for-zenn]。

*   [`remark-frontmatter`](https://github.com/remarkjs/remark-frontmatter) - FrontMatter 処理
*   [`remark-gfm`](https://github.com/remarkjs/remark-gfm) - テーブル、注釈など

```js:.remarkrc.mjs
import remarkFrontmatter from 'remark-frontmatter';
import remarkGfm from 'remark-gfm'

const remarkConfig = {
  plugins: [
    remarkFrontmatter,
    remarkGfm
  ]
}

export default remarkConfig
```

[^plugins-for-zenn]: その他に [`remark-directive`](https://github.com/remarkjs/remark-directive) も利用できるかと思ったのですが少し書式があわないようです。 <https://zenn.dev/link/comments/bdd54d1d8bcc9f>

## JSON にする

設定ができていればとくに難しいことはなく、`--tree-out` を指定するだけです。実行結果の JSON はプラグイン処理などが適用されています。

```markdown:test.md
# テストのファイル

プラグインのテスト :dog: :+1: :-)
```

```shell-session
$ npx remark-cli --quiet --tree-out < test.md > output.json
```

:::details (実行結果)

```json:output.json
{
  "type": "root",
  "children": [
    {
      "type": "heading",
      "depth": 1,
      "children": [
        {
          "type": "text",
          "value": "テストのファイル",
          "position": {
            "start": {
              "line": 1,
              "column": 3,
              "offset": 2
            },
            "end": {
              "line": 1,
              "column": 11,
              "offset": 10
            }
          }
        }
      ],
      "position": {
        "start": {
          "line": 1,
          "column": 1,
          "offset": 0
        },
        "end": {
          "line": 1,
          "column": 11,
          "offset": 10
        }
      }
    },
    {
      "type": "paragraph",
      "children": [
        {
          "type": "text",
          "value": "プラグインのテスト 🐶 👍 😃",
          "position": {
            "start": {
              "line": 3,
              "column": 1,
              "offset": 12
            },
            "end": {
              "line": 3,
              "column": 19,
              "offset": 30
            }
          }
        }
      ],
      "position": {
        "start": {
          "line": 3,
          "column": 1,
          "offset": 12
        },
        "end": {
          "line": 3,
          "column": 19,
          "offset": 30
        }
      }
    }
  ],
  "position": {
    "start": {
      "line": 1,
      "column": 1,
      "offset": 0
    },
    "end": {
      "line": 4,
      "column": 1,
      "offset": 31
    }
  }
}
```

:::

JSON から戻すときは `--tree-in` を指定します。

```shell-session
$ npx remark-cli --quiet --tree-out < test.md | npx remark-cli --quiet --tree-in
# テストのファイル

プラグインのテスト 🐶 👍 😃
```

## JSON を変形させる

JSON 用のツールを利用して変形させてみます。

### `jq` の場合

[`jq`](https://stedolan.github.io/jq/) で「深さが 2 と 3 の見出し」を抜き出してみます。

見出しは `#` や `---` での記述やコードブロック内の Markdown など考慮することが多いのですが、JSON(AST) であれば node の `type` で判別できます。

:::details (少し意地悪なテストデータ)

    ---
    ## テストのファイル
    id: preview-zenn-article-toc-in-terminal
    title: Zenn 記事の目次をターミナル内でプレビュー
    emoji: "\U0001F30A"
    type: idea
    topics:
      - zenn
      - markdown
      - cli
      - terminal
    published: true
    ---
    # テストのファイル

    ## 見出し 2-1

    段落

    ### 見出し 3-1

    段落

    #### 見出し 3-1

    段落

    ### 見出し 3-2

    ```markdown
    ## コードブロック内の見出し 2-1

    段落

    ### コードブロック内の見出し 3-1
    ```

    見出し 2-2
    ----------

    段落

    ## おわりに

    段落

:::

jq のフィルターは以下のようになります。

    jq '.children |=  map(select(.type == "heading" and .depth >=2 and .depth <=3))'

実行してみます。

```shell-session
$ npx remark --quiet --tree-out < test.md | jq '.children |=  map(select(.type == "heading" and .depth >=2 and .depth <=3))' | npx remark --quiet --tree-in
## 見出し 2-1

### 見出し 3-1

### 見出し 3-2

## 見出し 2-2

## おわりに
```

### `jsonata` の場合

[jsonata](http://docs.jsonata.org/overview.html) で「見出しの末尾に `!` を追加」してみます。

jsonata の式は以下のようになります。

    $ ~> | children[type="heading"] | {"children": $append(children,{"type":"text", "value": "!"}) }|

jsonata は CLI ツールではないので `.mjs` でスクリプトを作成します。

```javascript:test.mjs
import jsonata from 'jsonata'

const t = jsonata('$ ~> | children[type="heading"] | {"children": $append(children,{"type":"text", "value": "!"}) }|')

let data=''

process.stdin.on('data', (v)=>{data = data + v.toString()})
process.stdin.on('close', ()=>{
  process.stdout.write(JSON.stringify((t.evaluate(JSON.parse(data)))))
})
```

実行してみます。

```shell-session
$ npx remark --quiet --tree-out < test.md | node test.mjs | npx remark --quiet --tree-in
```

:::details (実行結果)

    ---
    ## テストのファイル
    id: preview-zenn-article-toc-in-terminal
    title: Zenn 記事の目次をターミナル内でプレビュー
    emoji: "\U0001F30A"
    type: idea
    topics:
      - zenn
      - markdown
      - cli
      - terminal
    published: true
    ---

    # テストのファイル!

    ## 見出し 2-1!

    段落

    ### 見出し 3-1!

    段落

    #### 見出し 3-1!

    段落

    ### 見出し 3-2!

    ```markdown
    ## コードブロック内の見出し 2-1

    段落

    ### コードブロック内の見出し 3-1
    ```

    ## 見出し 2-2!

    段落

    ## おわりに!

    段落

:::

### パイプで連鎖させる

上記の変形処理は JSON のフィルター的に動作しているので、プラグインのように連鎖させることもできます。

```shell-session
$ npx remark-cli --quiet --tree-out < test.md | jq '.children |=  map(select(.type == "heading" and .depth >=2 and .depth <=3))' | node test.mjs | npx remark-cli --quiet --tree-in
## テストのファイル!

## 見出し 2-1!

### 見出し 3-1!

### 見出し 3-2!

## 見出し 2-2!

## おわりに!
```

## おわりに

Markdown を JSON(AST) へ変換し簡単な変形を試してみました。

remark-cli の設定が少し手間ですが「Markdown を変形させたいけどプラグインを作るほどでもないかな」という場合に使えそうです。

## スクラップ

この記事は以下のスクラップから作成しました。

@[card](https://zenn.dev/hankei6km/scraps/7fae1422b8ad4c)
