---
id: writing-article-with-headless-cms
title: Zenn  の記事を Headless CMS(今回は microCMS を利用) で作成する
emoji: "\U0001F516"
type: idea
topics:
  - zenn
  - markdown
  - cms
  - microcms
published: true
---
Zenn を使うにあたり、記事の作成を HeadlessCMS(今回は [microCMS](https://microcms.io/) を利用) + 自作のツール([remote-cms-content](https://github.com/hankei6km/remote-cms-content))で行うことにしました。

この記事はその検証も兼ねて作成したものです。

## 構成

まだテスト段階で手動による実行部分も多いので、簡単な構成のみ記述します。

### 編集とプレビュー

以下のように記事(article)用 API を作成しリッチエディタ記述、ある程度まとまったら保存用スクリプトを実行しローカルにダウンロードします(手動で実行)。

保存用スクリプトで利用している `rcc` コマンドはマッピング設定を用意することで、受信データを FrontMatter + Markdown へ変換する機能を持っています。

変換機能により Zenn 形式で保存されたファイルは [zenn-cli](https://zenn.dev/zenn/articles/install-zenn-cli) と [textlint](https://github.com/textlint/textlint) でプレビューとチェックが行えます(チェック結果の確認はスクリプトの出力を目視で行う)。

```mermaid
flowchart LR
  subgraph "microCMS"
    api[/article API/]
  end
  api -- HTML --> rcc[["$ rcc save"]]
  subgraph "local repo(preview)"
    rcc -- Markdown --> md[("articles/slug.md")]
    md --> zenn[["$ zenn --preview"]]
    md --> textlint[["$ textlint"]]
  end
```

:::details スクリーンショットとマッピング設定。

![article API のスキーマを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/89f72d63469045e5b5edcb4aff713da1/writing-article-with-headless-cms-article-api.png?w=600\&h=356\&auto=compress%2Cformat)*article API のスキーマ*

![リッチエディタでコンテンツを編集中しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e8ccb5dc22ee479aa13e3753cca499e1/writing-article-with-headless-cms-edit.png?w=600\&h=368\&auto=compress%2Cformat)*リッチエディタでの編集*

![保存スクリプトを実行して textlint のエラーが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e97d1a9db8c54c43a5e7cb5a9552b9c8/writing-article-with-headless-cms-save-script.png?w=600\&h=482\&auto=compress%2Cformat)*保存用スクリプトの実行例*

```yaml:マッピング設定
disableBaseFlds: true
passthruUnmapped: false
selectFldsToFetch: true
position:
  disable: true

media:
  image:
    library:
      - src: https://images.microcms-assets.io/assets/
        kind: imgix

flds:
  - fetchFld: title
    query: title
    dstName: title
    fldType: string

  - fetchFld: content
    query: content
    dstName: content
    fldType: html
    convert: markdown
    toMarkdownOpts:
      imageSalt:
        command: rebuild
        baseURL: https://images.microcms-assets.io/assets/
        rebuild:
          keepBaseURL: true
          baseAttrs: 'data-salt-qm="auto=compress,format"'
      unusualSpaceChars: throw

  - fetchFld: emoji
    query: emoji
    dstName: emoji
    fldType: string

  - fetchFld: type
    query: type
    dstName: type
    fldType: string

  - fetchFld: topics
    query: '[topics.topic]'
    dstName: topics
    fldType: object

  - fetchFld: published
    query: published
    dstName: published
    fldType: boolean
```

:::

### Zenn へのデプロイ

保存用スクリプトは「`published` が `true` でなおかつ textlint をパスチェックしたファイル」を GitHub へプッシュするためのリポジトリへコピーします。

コピーされたファイルが問題なさそうであれば [GitHub CLI](https://cli.github.com/) で GitHub のリポジトリを更新します(ここも目視で確認、手動で実行)。

```mermaid
flowchart LR
  subgraph "local repo(preview)"
    mdp[("articles/slug1.md")]
  end
  subgraph "local repo(push)"
    mdl[("articles/slug1.md")]
    mdl2[("articles/slug2.md")]
    mdl3[("articles/slug3.md")]
  end
  subgraph "GitHub repo"
    mdr[("articles/slug1.md")]
    mdr2[("articles/slug2.md")]
    mdr3[("articles/slug3.md")]
  end
  mdp -- "copy(published)" --> mdl
  mdl -- $ git push --> mdr
```

### メモ機能

記事とは別にメモ用の API を作成することで、参考にしたページなどをメモしておくことができます。

また、メモは参照フィールドを使うことで記事からリンクさせることもできます(項目を自由に増やしたりリンクできるのは Headless CMS ならではかと)。

```mermaid
flowchart LR
  subgraph "article API"
    a1[("article1")]
    a2[("article2")]
    a3[("article3")]
  end
  subgraph "memo API"
    m1[("memo1")]
    m2[("memo2")]
    m3[("memo3")]
    m4[("memo4")]
    m5[("memo5")]
  end
  a1 --> m1
  a2 --> m2
  a2 --> m3
  a3 --> m4
```

:::details スクリーンショット。

![memo API のスキーマを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/91180ccf27e34e3eb5641572081e7843/writing-article-with-headless-cms-memo-api.png?w=600\&h=349\&auto=compress%2Cformat)*memo API のスキーマ*

![記事からメモへリンクしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/62e6b0fcf769459c93fb59da17607a43/writing-article-with-headless-cms-article-link-memo.png?w=600\&h=352\&auto=compress%2Cformat)*記事からメモへのリンク*

:::

## 実際に記述してみたもの

「Zenn の Markdown 記法」を上記構成で記述したサンプルです。

基本的にはリッチエディタの HTML 出力を [`rehype-remark`](https://github.com/rehypejs/rehype-remark) で変換し、リッチエディタで表現できない部分はコードブロックを独自記法で拡張すること等により対応しています。

@[card](https://hankei6km.github.io/mardock/deck/using-rich-editor-as-markdown-editor)

### 画像

リッチエディタの通常の UI 操作で挿入し、サイズ変更や `alt` とリンクを設定できます。

Markdown に比べると煩雑なように思えますが、エスケープなどはエディター側が処理してくれるので(気分的には)楽に感じます。

![画像の設定をしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7d2dd849ff49405ab1ca2215a2c3ea0f/writing-article-with-headless-cms-image.png?w=600\&h=398\&auto=compress%2Cformat)

![曇り空な河川敷](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/769f93b0d4b1485183948b9d92d3d7cd/cloudy-picture.jpg?w=400\&h=225\&auto=compress%2Cformat)

### 画像(imgix)

microCMS の画像はバックエンドが [imgix](https://imgix.com/) なので[クエリーパラメーター](https://docs.imgix.com/apis/rendering)で加工もできます。

ただし、通常の UI では行えないので、[rehype-image-salt](https://hankei6km.github.io/rehype-image-salt-doc/) の[記法で設定](https://hankei6km.github.io/rehype-image-salt-doc/writing)します(この辺は remote-cms-content による Markdown 変換時に処理されます)。

![rehype-image-salt の記法でクエリーパラメーターを設定しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/018ac13d16014755a80408437d393dfa/writing-article-with-headless-cms-image-imgix.png?w=600\&h=274\&auto=compress%2Cformat)

![曇り空な河川敷](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/769f93b0d4b1485183948b9d92d3d7cd/cloudy-picture.jpg?w=400\&h=225\&auto=compress%2Cformat\&blur=100)

:::message

各画像には以下のパラメーターがデフォルトで付与されています。

*   CMS によるサイズ関連のパラメーター(上記 UI による指定)
*   rehype-image-salt による [`auto=compress,fromat`](https://docs.imgix.com/apis/rendering/auto/auto)

:::

### 画像(Zenn)

Zenn 独自の記法ではキャプション風の表示は以下の方法で対応できます。

*   画像の直後にイタリックでテキスト入力する
*   rehype-image-salt の [`data-salt-zenn-cap`](https://hankei6km.github.io/rehype-image-salt-doc/writing#data-salt-zenn-cap) を利用する

ただし、厳密には[正規の指定](https://zenn.dev/zenn/articles/markdown-guide#%E3%82%AD%E3%83%A3%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E3%81%A4%E3%81%91%E3%82%8B)と若干異なる表記になるので(本来は画像の下に記述すべき)、将来的に使えなくなる可能性はあります。

> 画像のすぐ下の行に`*`で挟んだテキストを配置すると、キャプションのような見た目で表示されます。

また、横幅の指定は対応できていません。

![イタリックにしたテキストによるZennのキャプション風表示を指定しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/fa172fdbbe204ac2b224df7dd9c8af38/writing-article-with-headless-cms-image-zenn-caption.png?w=600\&h=271\&auto=compress%2Cformat)

![曇り空な河川敷](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/769f93b0d4b1485183948b9d92d3d7cd/cloudy-picture.jpg?w=400\&h=225\&auto=compress%2Cformat)*Zenn の Caption 風表示*

### コードブロック

コードブロックで言語やファイル名を指定する場合は拡張されたコードブロック内でコードブロックを記述する必要があります(ややこしいですが、慣れるとあまり気になりません)。

なお、リッチエディタでは書式なし(いわゆるプレーンテキスト)で連続する空白文字をコードブロックへペーストすると No-Break Space になってしまいます。

テキストエディターからペーストするとき気になる場合は対策が必要です(remote-cms-content では「警告を出す」「変換する」ことで対応しています)。

また、(コードブロックに限った話ではないですが) `&` のエスケープ処理が独特なので `＆gt;` などを記述することが難しいです(ここではいわゆる全角で `&` を記述することで回避しています)。

参考: [micoCMS のメモとサンプル - & のエスケープ](https://hankei6km.github.io/test-collage-cms-content/microcms-plain#h0722d3ceb0)

![拡張されたコードブロック内に記述しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d3448a1e2d94411298b983111a0b83fd/writing-article-with-headless-cms-codeblock.png?w=600\&h=270\&auto=compress%2Cformat)

```js:fooBar.js
const great = () => {
  console.log("Awesome")
}
```

### 引用

リッチエディタの通常の UI で指定できます。ただしペーストするときの書式の有無、改行や空白の扱いなどに注意は必要です。

![リッチエディタのUIで引用を指定しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e0e17b2516a84bbaaeae230f760decf0/writing-article-with-headless-cms-blockquote.png?w=600\&h=253\&auto=compress%2Cformat)

> 引用文。 引用文。

### 注釈

注釈は`[]` がエスケープされてしまうので、下線を利用する機能を追加しました。

1.  `^` 付きの文字を下線ありにすると [footnoteReference](https://github.com/syntax-tree/mdast#footnotedefinition) へ変換(`[]` は記述しない)

2.  それ以外の下線ありの文字列は [footnote](https://github.com/syntax-tree/mdast#footnote)(インライン注釈)へ変換(`^[]` は記述しない)

    *   インライン注釈内では Markdown で記述可能(リンクなどを設定できる)。

![リッチエディタの UI で下線を指定し脚注を記述しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b7c7af6ce0554bff88d74887951f9ce0/writing-article-with-headless-cms-footnote.png?w=600\&h=254\&auto=compress%2Cformat)

脚注の例[^1]です。インライン^[脚注の内容その 2]で書くこともできます。

[^1]: 脚注の内容その 1。

### テーブル

拡張されたコードブロック内で記述できます。

![拡張されたコードブロック内にテーブルを記述しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/298abe2b27a2421497467b4f47c9614d/writing-article-with-headless-cms-table.png?w=600\&h=241\&auto=compress%2Cformat)

| Head | Head | Head |
| ---- | ---- | ---- |
| Text | Text | Text |
| Text | Text | Text |

### メッセージとアコーディオン(トグル)

`:::` はそのまま記述できるので、Markdown へ変換後に改行されるようにすれば記述できます。

![通常の段落に ":::" を記述してメッセージとしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5dd3150bd64345deb146222bd095dfc3/writing-article-with-headless-cms-message.png?w=600\&h=278\&auto=compress%2Cformat)

:::message

メッセージをここに。**装飾**なども指定可能。

:::

:::details タイトル。

表示したい内容。**装飾**なども指定可能。

:::

### 数式

これも `$$` をそのまま記述できるので、とくに変わったことはしなくても記述できます。

ただし、(KaTex の記法にうといので影響度は不明ですが)Markdown 的な記法があるとリッチエディタの機能で変換される可能性はあります。

![通常の段落に "$$" を記述して数式としているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/875ede9c1bd0480980dc6addcaaf2ffd/writing-article-with-headless-cms-math.png?w=600\&h=294\&auto=compress%2Cformat)

$$

e^{i\theta} = \cos\theta + i\sin\theta

$$

インラインの場合 $a\ne0$ も記述できます。

### コンテンツの埋め込み

[`remark-gfm`](https://github.com/remarkjs/remark-gfm) の autolink とリッチエディタの変換が効いてしまうので、拡張されたコードブロック内で `@[card](URL)` 等の記法を使う必要があります。

![拡張されたコードブロック内にカードのリンクを記述しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8926ac79146f4d7496ad7fe8f24cc5ea/writing-article-with-headless-cms-cardlink.png?w=600\&h=159\&auto=compress%2Cformat)

@[card](https://zenn.dev/zenn/articles/markdown-guide)

### ダイアグラム

拡張されたコードブロック内で記述できます。

![拡張されたコードブロック内にダイアグラムを記述しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/47a533db71044e7f9c721bf3de876685/writing-article-with-headless-cms-diagram.png?w=600\&h=237\&auto=compress%2Cformat)

```mermaid
graph LR
  cms[/"Hedless CMS(RichText)"/] --> local[("local repo")] --> github[("repo on GitHub")] --> zenn[/"Zenn"/]
```

## モバイル対応

記事はデスクトップでないと厳しい面も多いのですが、「メモは思いついたときにちゃちゃっと入力したい」というのがあったのでわりと重視した項目です。

以下のスクリーンショットはジョギング中に思いついたこをメモしたものです。

:::details スクリーンショット。

![スマートフォンでメモを作成したときのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/cfae375dd70b450da9c042cae4a6c31a/writing-article-with-headless-cms-mobile.png?w=540\&h=968\&auto=compress%2Cformat)

:::

ジョギング中にいつもリッチエディタでメモをとっているので慣れもありますが、簡単なメモならそれなりに入力できます。

ちなみに、各種 Headless CMS サービスの中から microCMS を選択した理由の 1 つがこれです。

「レスポンシブ対応」「スマートフォンでも RichText 系フィールドへの日本語入力が安定している」サービスとなると、現状では選択肢が限られます。

## 各種 Headless CMS 対応

今回は microCMS を利用してますが、remote-cms-content が対応しているサービスであれば同じように構成できます。

たとえば、[GraphCMS](https://graphcms.com/) では以下のような GraphQL のクエリーとマッピング設定を用意することで対応できます(実は当初 GraphCMS で試していた)。

:::details 参考設定(内容的にはちょっと古いです)。

```graphql:クエリー
query GetContent($stage:Stage!, $endCursor: String, $pageSize: Int) {
  articlesConnection(stage:$stage, after:$endCursor, first:$pageSize){
    edges{
      node{
        id
        slug
        title
        content {
          html
        }
        emoji
        type
        topics
        published
      }
    }
    pageInfo{
      hasNextPage
      endCursor
    }
  }
}
```

```yaml:マッピング設定
disableBaseFlds: true
passthruUnmapped: false
position:
  disable: true

transform: data.articlesConnection.{"items":[edges.node],"pageInfo":pageInfo}

flds:
  - query: slug
    dstName: id
    fldType: id

  - query: title
    dstName: title
    fldType: string

  - query: content.html
    dstName: content
    fldType: html
    convert: markdown
    toMarkdownOpts:
      imageSalt:
        command: "rebuild"
        baseURL: https://media.graphcms.com/
        rebuild:
          keepBaseURL: true

  - query: emoji
    dstName: emoji
    fldType: string

  - query: type
    dstName: type
    fldType: string

  - query: topics
    dstName: topics
    fldType: object

  - query: published
    dstName: published
    fldType: boolean
```

:::

各サービスの機能は異なるので全く同じにはできませんが、逆にいえばサービス別の設定を用意しておき「記事の特性にあわせて利用するサービスを切り替える」ということもできます。

## Markdown 系フィールドは？

サービスによっては Markdown 入力に特化したフィールドが用意されています。

Contentful であれば「(デスクトップ環境で)日本語入力が普通に利用できる」「フルスクリーンにできる」「コードブロックも記述できる」など利点も多いです。

また、今回の利用方法であれば「GitHub の変更も取り込みやすい」という点も見逃せません。

しかし、([個人的な好み](https://hankei6km.github.io/mardock/deck/using-rich-editor-as-markdown-editor)もありますが)以下の点で RichText 系フィールドを利用しています。

*   Markdown フィールドへアセット(画像) を挿入すると参照関係にならない

「記事のエントリー」と「アセット(画像)」が参照関係にならないことには 2 つのデメリットがあります。

*   アセット(画像)の変更が Markdown フィールドに反映されない
*   アセット(画像)を利用しているエントリーを把握できない

写真などを主に扱う場合は影響が少ないのですが(画像 API で加工できることが多い)、スクリーンショットなどを扱う種類のテキストでは画像を入れ替えることも多いので操作性へ影響してきます。

以上のことから現状では Markdown 系フィールドの利用を見送っています。

## 本の作成

記事の作成とは直接の関係はないのでさわりだけ。

「設定」と「チャプター」のモデル(API) を作成し参照関係を構成することで本の作成([CLI 管理形式での保存](https://zenn.dev/zenn/articles/zenn-cli-guide#cli-%E3%81%A7%E6%9C%AC%EF%BC%88book%EF%BC%89%E3%82%92%E7%AE%A1%E7%90%86%E3%81%99%E3%82%8B))もできます。

:::details スクリーンショット。

![本の設定からチャプターの参照している状況のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0d23b79a68324311a8fb8b5f5b609005/writing-article-with-headless-cms-book-config.png?w=600\&h=372\&auto=compress%2Cformat)*本の設定からチャプターを参照*

![本のプレビューを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2bac4c0f511549c0ad6950f59bee44c4/writing-article-with-headless-cms-book-preview.png?w=600\&h=481\&auto=compress%2Cformat)*本のプレビュー*

:::

## 検討事項

いくつか記事を作成してみたところ良い感じではあるのですが、やはり検討すべき項目もあります。

### プレビューとチェック

現在の構成では「リッチエディタ」「ターミナル」「プレビュー」用の画面を行ったり来たりするのでどうにかしたいところです。

一番よさそうなのは、[draftlint](https://github.com/hankei6km/draftlint) のときのように「リッチエディタの画面プレビューボタンをクリックするとチェック結果付きのプレビューが表示される」なのですが、ちょっと大変そうかなと。

@[card](https://github.com/hankei6km/draftlint)

### publish の自動化

これもローカル側での手動部分が多いので、CMS 側から直接 GitHub Actions を起動して自動化する方がよいのかなと。

ただ、感覚的な問題なのですが、自動化(ブラックボックス化)しすぎてしまうのも味気ないかなというのもあって少し考え中です。

### CMS と Zenn の publish

現在はスキーマ内に Zenn 用の publish フィールドを定義しています。

本来なら CMS の publish にあわせてるべきですが、各 CMS でプレビューの扱いが異なるのでこれも対応方法を考え中です。

### メモ

#### プレビューアプリ

メモの中で mermaid によるダイアグラムを書きたくなることがあり、そうなるとメモにもプレビュー的な表示が欲しくなってきます。

#### Scraps との使い分け

[Zenn Scraps](https://zenn.dev/zenn/articles/about-zenn-scraps) を少し使ってみたとろ「メモはこれでもいいかな」と少しぐらついてしまっている。

#### 情報が分散してしまう

普段は howm([qfix-howm](https://sites.google.com/site/fudist/Home/qfixhowm))でメモをとっているので情報が分散してしまいます。

これについては、howm の中に `zenn` ディレクトリーを作ってダウンロードしてしまおうかと考えています。

どちらも Markdown なので、1 箇所にまとめてしまえば検索などは違和感なく使えるかなと。

### プルリクエスト

以下の記事を参考にしてリポジトリの名前を変更しているときに「現状の構成だとプルリクエストなどをいただいても Headless CMS 側へ反映させる方法がない」ということに気が付きました。

@[card](https://zenn.dev/j5c8k6m8/articles/zenn-github-repository-name)

### 編集履歴

各 CMS の無料プランでは履歴管理はだいたいが対象外なのですが、今回はローカル側のリポジトリにダウンロードするのでそちら対応できないか考え中です。

### Vim キー配列

mermaid の記述をするときなどにテキストオブジェクトが使えないのはやはりきびしいなと。

ブラウザー拡張などで解決しそうですが、拡張はあまり入れたくないのでおとなしく普通に編集しています。

### emoji

これが意外とめんどうで「ランダムな emoji をどうやって取得する？」「バリデーションで emoji を 1 文字ってどう指定する？」等々。

現状は「zenn-cli で作成した article のファイルからコピペ」することで対応しています。

## おわりに

以上のような感じで「記事と関連メモを手軽に編集できる環境」ができあがりました。

いくつか検討事項もありますが、しばらくはこの構成で記事を作成していこうかと。
