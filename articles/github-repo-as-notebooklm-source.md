---
id: github-repo-as-notebooklm-source
title: GitHub のリポジトリを 1 ファイルにまとめるとノートブックにしやすい
emoji: 📖
type: idea
topics:
  - GitHub
  - NotebookLM
published: true
---

以前にも少し書きましたが、[NotebookLM を使って GitHub 上のリポジトリについて調べると便利だった](https://zenn.dev/hankei6km/articles/merge-dependabot-bump-with-notebooklm)ので、その辺についてなど。

## 想定している読者

*   GitHub のリポジトリの内容からノートブックを作りたい人
*   外部のドキュメントなどを組み合わせてリポジトリの分析や編集に利用したい人
*   リポジトリに関連する諸々(Issues の検索など)を AI チャットボットに手助けしてほしい人

なお、この記事は「[GitHub Mobile でリポジトリを開くだけの簡単なナレッジベースです](https://zenn.dev/hankei6km/articles/github-mobile-copilot-knowledge-base)」の続きのような内容になっています。そちらもお読みいただくと、AI チャットボットによるリポジトリ活用のヒントが増えるかもしれません。 (この記事だけでもお読みいただける内容にはなっています)

## GitHub のリポジトリをノートブック化する

GitHub 上のリポジトリの情報(履歴やアクションのログなど)をまとめてノートブック化できるのが最善と思われます。しかし、その場合では複数の API を組み合わせてそれなりのデータ量をダウンロードすることになります。よって、現実的な落とし所としては以下のような感じになるかと思います。

*   特定の Git 参照(ブランチ名など)をチェックアウトした状態を 1 つのファイルに書き出す
*   上記ファイルをノートブックのソースにする
*   アクションのログなど必要な情報は別のソースとして追加する

ここで問題になるのが「チェックアウトした状態を 1 つのファイルに書き出す」部分かと思います。これについて、方法はいくつかありそうですが、ちょっと試してみようかなという場合は以下のシェルスクリプトで簡易的に対応できます。

https://github.com/hankei6km/shell-script-gh-repo-files/blob/main/scripts/to-json/repo-files.sh

使い方としては、以下のように `onwer` `repo` `ref`(ブランチ名など)を指定して、 `.txt` ファイルとして書き出します。

```sh
repo-files.sh hankei6km gas-gh-repo-files main > 'hankei6km gas-gh-repo-files main.txt'
```

::: message

認証などは必要としないのですが、その代わりにプライベートリポジトリなど権限が必要なリポジトリには使えません。 また、デフォルトブランチを判別できないので手動で `main` か `master` を指定することになります。

:::

あとは、ノートブック作成時に `.txt` をテキストファイルとして追加すると利用できます。

**図 2-1 ノートブックを作成しコードについて質問**

![リポジトリ内のコードに記述してあるクラスについて質問し、返答が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8aa97164910846479d5ca50a9056ea2a/github-repo-as-notebooklm-source-sample.png?w=1396\&h=484\&auto=compress%2Cformat)

小規模なライブラリーなどのリポジトリであれば、このように内容についの質問を行えます。

ただし、上記のスクリプトはテキストファイルとして JSON をベタ書きしているだけなので、大きめのリポジトリでは正しく認識されない傾向にあります。おそらく JSON ファイル単体は正式にサポートされていないことの影響かと思います。 (画像なども base64 で書き込んでいる影響もありそうです)。

対応として、Google ドキュメントとして書き出す手順も記述しておきます。少し手順が増えてしまいますが、大きめのリポジトリや画像を含むリポジトリを試す場合はこちらの方が適しています。 (次セクション以降の利用例でもこちらのスクリプトを利用しています)

::: details Google ドキュメントとして書き出す手順。

1.  下記の HTML で書き出すスクリプトを `repo-files owner repo ref > 'owner repo ref.html'` のように実行
2.  `.html` を Google ドライブへアップロード
3.  アップロードしたファイルを右クリックして「アプリで開く / Google ドキュメント」を選択

これで Google ドキュメントのファイルが作成されます。 少し面倒ですが、Google ドキュメント形式にしておくと画像を扱えるという利点もあります。

なお、HTML を Google ドキュメントとしてアップロードする CLI ツールもあるので、そちらを使うともう少し楽になるかと思います。

**HTML で書き出すスクリプト**

https://github.com/hankei6km/shell-script-gh-repo-files/blob/main/scripts/to-html/repo-files.sh

:::

## リポジトリをノートブック化したものを使ってみる

いくつか利用例を記述してみます。

### ドキュメントのリポジトリをノートブック化する

ライブラリーのドキュメントなどのソースが GitHub リポジトリとして公開されている場合、それをノートブック化すると簡単なナレッジベースができあがります。

以下は、Zenn のドキュメントをノートブック化した例です。

@[card](https://github.com/zenn-dev/zenn-docs)

**図 3-1 スクリプトで書き出したファイルから作成したノートブック**

![作成したノートブックで Zenn の Markdown 記法について質問し、返答が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b9df0de3543a453f861d1a4ff91c4a7f/github-repo-as-notebooklm-zenn.png?w=1396\&h=484\&auto=compress%2Cformat)

以上のように、ドキュメントのリポジトリをノートブック化すると「Zenn の Markdown 記法について教えてください」のようなプロンプトを使えるようになります。また、この後の利用例でも触れますが、ドキュメント単体ではなく他のリポジトリと組み合わせて以下のようにも利用できます。

*   コンテンツがドキュメントに沿っているかのチェック
*   コンテンツをドキュメントにあわせて変更(追加)する

### ドキュメントを元にコードの変更箇所を考えてもらう

[このライブラリー](https://github.com/hankei6km/gas-gh-repo-files)を作っているとき、JSDoc の型注釈付きコメントを追加したのですが、これはノートブックのソースに [JSDoc リファレンス](https://www.typescriptlang.org/ja/docs/handbook/jsdoc-supported-types.html) を追加し NotebookLM に考えてもらいました。

一発で生成してくれないときもありますが、以下のようにプロンプトを分割するとうまく生成してくれる印象です。

*   "GhRepoFiles " namespace について教えてください
*   "GhRepoFiles" namespace の各定義用の型注釈付き jsdoc コメントを考えてください

**図 3-2 考えてもらったコメントをメモ化したもの(実際に使ったノートブックは削除していたので内容は若干違います)**

![JSDoc コメントが追加されたコードを保存したメモを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0e673372a4d24e018d98eb811be224c8/github-repo-as-notebooklm-source-jsdoc.png?w=1172\&h=501\&auto=compress%2Cformat)

この他にも、依存パッケージの更新でマイグレーションガイドが出ている場合は、「ガイドをソースにして修正が必要な箇所をピックアップしてもらう」といった使い方もあります。

難点は(Copilot のインラインチャットなどに比べると)、考えてもらったコードを使うにはコピペする手間があるというところです。対策として「git コマンドで適用できる patch ファイルとして表示してください」などとやったのですが、微妙に正しくない内容で適用はできませんでした。この辺は今後に期待といったところでしょうか。

::: message

**「考えてください」とお願いする**

NotebookLM ではソースに記載されていない内容を質問すると、「返答に必要な情報が提供されていません」のような返答になります。

よって、上記の例の場合であれば、「どのようなコメントになりますか？」ではなく「最適なコメントを考えてください」とすることで受け付けされる率が上がるようです。ただし、NotebookLM の「ソースに書かれていなことは知らないと返答する」という利点は失われるので注意は必要です。

:::

### 図解付き修正指示をソースとして利用する

実際には 1 度使っただけですが、こういうこともできますという例として挙げておきます。

最初に、UI のスクリーンショットに修正箇所を追記した Google ドキュメントのファイルを作成します。 (NotebookLM で画像を扱うには Google ドキュメントかスライドが必要なため)

**図 3-3 スクリーンショットなどを用いて修正箇所を指定する**

![スクリーンショットで変更したい箇所を赤枠で囲み(ボタンを囲んでいる)、その色を「青系に変更」という修正内容が指示されている Google ドキュメントのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/914a9b7ca3174604baa64ff4e81e1c5b/github-repo-as-notebooklm-source-img-gdoc.png?w=1146\&h=900\&auto=compress%2Cformat)

修正の指示を記述した Google ドキュメントを作成したら、ソースコードのリポジトリとあわせてノートブックを作成します。

**図 3-4: ソースコードと修正指示のノートブックを作成**

![作成されたノートブックのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c346af1abbe84c2d8d822786786c2842/github-repo-as-notebooklm-source-code-img.png?w=1387\&h=649\&auto=compress%2Cformat)

作成したノートブックに以下のようなプロンプトを入力し、コードの修正方法を考えてもらいます。

*   UI の修正が必要な箇所を教えてください

::: details ノートブックからの返答。

> ### UIの修正が必要な箇所
>
> ソース\[1]によると、赤枠で囲んだボタンが目立たないので青系の色へ修正する必要があります。 \[1]
>
> **改善点**:
>
> *   現在のボタンの色は背景色と似ており、目立ちません。
> *   青系の色に変更することで、ボタンをより目立たせることができます。
>
> **その他**:
>
> *   ソース\[2-44]は、UIの修正について言及しておらず、今回の質問に関係ない情報です。
>
> **UI修正の参考例**:
>
> *   「Run」ボタンと同じ青系の色にすることで、UIに統一感が生まれます。
> *   ボタンの色を青系にするだけでなく、影をつけたり、枠線を濃くしたりすることで、さらに目立たせることができます。

:::

*   そのボタンを青系の色へ変更するために、リポジトリの "index.html" をどのように編集すべきか考えてください

::: details ノートブックからの返答。

> ### "index.html" の編集方法
>
> ソース\[1]にある赤枠で囲まれたボタンを青系に変更するには、ソース\[2, 3]の pages/index.html 内にあるボタンのHTMLを編集する必要があります。
>
> 具体的には、下記の3つのボタンの class 属性を編集し、青系の色を指定します。
>
> *   id="append-template-git-diff" のボタン
> *   id="append-template-actions-log" のボタン
> *   id="append-template-clear" のボタン
>
> これらのボタンには現在、bg-gray-300 と hover:bg-gray-400 のクラスが適用され、それぞれ灰色と濃い灰色を表現しています。
>
> これらのクラスを、Tailwind CSSのカラーパレットを用いて青系の色に変更します。
>
> **変更例**:
>
> *   bg-blue-500: 青色
> *   hover\:bg-blue-700: 濃い青色
>
> 上記は一例であり、Tailwind CSSのカラーパレットには様々な青系の色があります。<https://tailwindcss.com/docs/customizing-colors> を参考に、好みの色を選択してください。
>
> **編集後のHTML例**:
>
> <button                 type="button"                 class="px-4 py-1 bg-blue-500 text-white rounded hover:bg-blue-700 disabled:opacity-50 mr-2"                 id="append-template-git-diff"                 onclick="handleTemplate('git-diff')"               >                 git diff               </button>
>
> 変更後、npm run build を実行して変更を反映します。
>
> **注意点**:
>
> *   ソース\[2, 3]はTailwind CSSを使用しており、bg-gray-300のようなクラス名で色が指定されています。
> *   Tailwind CSSは、あらかじめ定義されたクラス名を使うことで、CSSを直接記述することなくスタイルを適用できるライブラリです。

:::

**図 3-5 修正案が表示される**

![プロンプトへの返答してコードの修正案が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2cbd966b5a014433955f7e43d1b42dfd/github-repo-as-notebooklm-source-suggest.png?w=1387\&h=518\&auto=compress%2Cformat)

修正案に沿って変更したものが以下になります。

**リスト 3-1 修正した内容(git diff)**

::: details 長いの折りたたんでいます。

```diff
diff --git a/pages/index.html b/pages/index.html
index e66345e..6086e8c 100644
--- a/pages/index.html
+++ b/pages/index.html
@@ -31,7 +31,7 @@
             <div class="flex mb-2">
               <button
                 type="button"
-                class="px-4 py-1 bg-gray-300 rounded hover:bg-gray-400 disabled:opacity-50 mr-2"
+                class="px-4 py-1 bg-blue-500 text-white rounded hover:bg-blue-700 disabled:opacity-50 mr-2"
                 id="append-template-git-diff"
                 onclick="handleTemplate('git-diff')"
               >
@@ -39,7 +39,7 @@
               </button>
               <button
                 type="button"
-                class="px-4 py-1 bg-gray-300 rounded hover:bg-gray-400 disabled:opacity-50 mr-2"
+                class="px-4 py-1 bg-blue-500 text-white rounded hover:bg-blue-700 disabled:opacity-50 mr-2"
                 id="append-template-actions-log"
                 onclick="handleTemplate('actions-log')"
               >
@@ -47,7 +47,7 @@
               </button>
               <button
                 type="button"
-                class="px-4 py-1 bg-gray-300 rounded hover:bg-gray-400 disabled:opacity-50 mr-2"
+                class="px-4 py-1 bg-blue-500 text-white rounded hover:bg-blue-700 disabled:opacity-50 mr-2"
                 id="append-template-clear"
                 onclick="handleTemplate('clear')"
               >
```

:::

**図 3-6 ボタンが青系になる(今度は目立ちすぎるので実際には取り込みませんでした)**

![修正指示で指摘されていたボタンが青系の色に変更されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ca5f9bc0da564ca6abf3f77a550e1cb6/github-repo-as-notebooklm-source-buttion-color.png?w=687\&h=564\&auto=compress%2Cformat)

こちらも専用のツールなどに比べると使いどころは難しそうですが、スクリーンショットと Google ドキュメントだけで実現できるのでいざというときに使えるのかなと思っています。 (思っているだけになりそうですが)

### その他

他にも利用方法はありますが、以下はわりと利用している例です。

*   `git diff` の内容を使ってブランチ名などを考えてもらう(Copilot チャットでは diff の扱いが意外とめんどうだった)
*   GitHub Action のワークフローが落ちたときにログをコピペして原因を調べてもらう
*   パッケージ(ライブラリー)を探しているとき、ヒットしたリポジトリについて調べてもらう

## Issues をノートブック化してみる

リポジトリに関連する諸々でいうと、Actions のログなどについては、ウェブ UI から落ちたジョブの生ログをコピペすれば大体は目的を達成できました。

しかし、Issues の場合は必要な情報を全体から探す必要があります。そういうツール(サービス)もありそうですが、ここでは GitHub CLI を使ってノートブック用のソースを作ってみます。

今回は、ちょっと前に「[clasp](https://github.com/google/clasp) の push コマンドの終了コードがおかしいのだけど issue 立ってるかな」という検索で使ってみた手順を例として記載してみます。

まず、GitHub CLI で「push」を含む issues を JSON として `.txt` に書き出します。

```sh
gh issue list --repo google/clasp --limit 2000 --state all --json title,number,body,comments --search push > issue_push.txt
```

これをソースにノートブックを作成します。

次に「push コマンドが失敗したときの終了コードについて議論している issue は？」とは**質問しません**。issues の大きさにもよりますが、前述のように大きい JSON への返答は精度があまりよくないです。代わりに「push コマンドの終了コードについて議論している issue の一覧を issue 番号付きで作成してください」とします。 (探してくださいより要約してください的な指示の方が良い返答を得られる印象です)

**図 4-1要約された一覧が作成される**

![issue 番号付きの項目が 2 つ表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/bc1f9d3148fb483faf7c33b1644c78f2/github-repo-as-notebooklm-source-summarized-issues.png?w=1390\&h=493\&auto=compress%2Cformat)

これも正しくないことが多いのですが、おおざっぱな傾向はつかめます。あとは、質問を続けるか、傾向をもとに GitHub のウェブ UI で普通に検索を続けて目的の情報を見つけます。

なお、同じ要領で PR もいきたいところですが、PR は単純な方法でコミットを含める方法がわかりませんでした。よって、今回は見送っています。

## 考慮点

NotebookLM わりと更新されているようなので(これを書いているときもソース追加の UI が変更されました)、現時点(2024 年 8 月)時点での「少し困ったな」的な項目を挙げておきます。

### 大きなリポジトリは扱いにくい

ファイルツリーなどのコピーを作成しソースとしてアップロードする必要があるため、いくつかサイズ的な制限があります。

*   HTML から Google ドキュメントへ変換可能なサイズ
*   ノートブックで扱えるワード数

たとえば、Tailwind CSS のドキュメントページのリポジトリを HTML へ書き出すと 4M バイトくらいですが、Google ドキュメントとして開くとエラーになりました。

@[card](https://github.com/tailwindlabs/tailwindcss.com)

### ソースを確認しにくい

ノートブックからの返答には参照されたソースへのリンクが提示されます。しかし、JSON 形式でソースを作っていると各ファイルが 1 フィールド(行)に詰め込まれているので目視では判断しにくい状況です。

また、Google ドキュメントで作成していても `<code>` `<pre>` によるコードブロック表現は改行がいわゆる豆腐になるため、こちらも少し内容を把握しにくい状況です。

ソースを作るときには、「ノートブック側が理解できればそれでよい」だけでなく「人間が参照しやすい書式にする」ことも考慮した方がよさそうです。

### ノートブックのライフサイクル

おそらくですが、NotebookLM は「厳選されたノートブックを適切に管理する」ような利用を想定しているように思われます。

しかし、ここまでの例に挙げたような利用の場合は、「なにか課題が持ち上がったらすぐノートブックを作ってしまう」ようになります。

そのため、(利用が終わったノートブックは削除しているのですが)ノートブックの数が増えていき管理が大変になりがちです。NotebookLM にはノートブックを管理する機能は少ないので、このまま使い続ける場合は、管理方法を別途考える必要がありそうです。

## おまけ(リポジトリを Google ドキュメントにする GAS のライブラリーなど)

シェルスクリプトを動かすにはそれなりに環境を用意することになるため、「スマートフォンでちょっと書き出したい」は難しいです。

そこで、GAS のライブラリーと個人的に使う用のウェブアプリを作りました。 (ウェブアプリはデプロイしたものを公開していません)

*   ライブラリー - <https://github.com/hankei6km/gas-gh-repo-files>
*   ウェブアプリ - <https://github.com/hankei6km/gas-webapp-gh-repo-files>

![ウェブアプリをスマーフォンのブラウザーで表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7e62c5c827b9404da6c6dab818c3e619/github-repo-as-notebooklm-source-webapp.png?w=864\&h=1920\&auto=compress%2Cformat)

アーカイブをオンメモリーで展開するので大きいリポジトリは扱えないのですが、スケジュール実行で書き出しておいたり、スマートフォンでファイルに書き出したりしています。また、すでに書き出してある Google ドキュメントを更新できるので、ノートブックのソースを(手動ですが)ブランチに追従させたりともできます。

## おわりに

GitHub リポジトリからノートブックを作成し、ソースコードの編集支援に利用したり Issues の検索(要約)などを試してみました。

[GitHub Mobile と GitHub Copilot の組み合わせの手軽さ](https://zenn.dev/hankei6km/articles/github-mobile-copilot-knowledge-base)に比べると、NotebookLM ではノートブックを作成する手間がかかります。しかし、ソースの組み合わせにより、さらに込み入った用途での利用が可能となっています。

また、ノートブックは開発に関連しないようなプロンプトも受け付けてくれるので、いろいろなカテゴリーのリポジトリをノートブックにしても面白いかと思います。
