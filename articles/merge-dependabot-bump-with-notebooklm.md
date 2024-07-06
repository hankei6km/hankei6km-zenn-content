---
id: merge-dependabot-bump-with-notebooklm
title: NotebookLM と考える Dependabot の Bump 対応
emoji: 🤔
type: idea
topics:
  - notebooklm
  - dependabot
published: true
---

[`esbuild`](https://github.com/evanw/esbuild) [v0.22.0](https://github.com/evanw/esbuild/releases/tag/v0.22.0) の bump 対応で NotebookLM を使ってみたところ、ちょっと便利だったので、その辺についてなど。

## 想定している読者

*   [Dependabot](https://docs.github.com/ja/code-security/dependabot/working-with-dependabot) の Bump を手動でゆるく対応している人
*   英語のリリースノートを読むのは少し苦手という人
*   NotebookLM で簡単な効率化を試してみたい人

## NotebookLM と考えるとは

Bump の PR を見ていたら「対応が必要そう」なリリースノートがちらっと目に入りました。

**図 2-1 backwards-incompatible changesと書かれたリリースノート**

![GitHub Mobile でプルリクエストのリリースノートを見ているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8cfa40781207406687ce151575908af5/merge-dependabot-bump-with-notebooklm-rel-note.png?w=864\&h=1920\&auto=compress%2Cformat)

いつもなら、ブラウザーでリリースノートを開いて右クリックから翻訳をするところですが、「リリースノートをノートブックのソースにすればよいのでは」と思いつきました。

やってみると、esbuild は「リリースに詳細が記述されている」「[API ドキュメン](https://esbuild.github.io/api/)トが単一なページになっている」と条件が整っていたので、対応法方法をノートブックで確認できてしまいました。という内容の記事です。

## 破壊的な変更点を質問してみる

まず、リリースノートをソースとして指定したノートブックを作成します。

**図 3-1 ノートブック作成時にリリースノートをソースとして指定**

![ソース指定のダイアログボックスでリリースノートの URL を入力しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b05a9152473845d98db96f2054cd6cc5/merge-dependabot-bump-with-notebooklm-rel-note-as-source.png?w=648\&h=279\&auto=compress%2Cformat)

ノートブックが作成できたら、プロンプトで「esbuild 0.22.0 の破壊的変更についておしえてください」のように質問すると以下のような回答になる。

> ## esbuild 0.22.0 の破壊的変更点
>
> esbuild バージョン 0.22.0 では、後方互換性のない変更が意図的に導入されています。 自動的にこのようなリリースに更新されないように、`package.json` ファイルで esbuild の正確なバージョンを指定するか (推奨)、`^0.21.0` や `~0.21.0` などのパッチアップグレードのみを受け入れるバージョン範囲構文を使用する必要があります。
>
> ### Node.js をターゲットにした場合のパッケージのバンドル除外
>
> **破壊的変更:** `--platform=node` を指定して Node.js 向けにコードをバンドルする場合、esbuild はデフォルトでパッケージをバンドルから除外するようになりました。 これは、一部のパッケージがバンドラー向けに設計されておらず、バンドラーと互換性のない Node.js 固有の機能を使用している可能性があるためです。
>
> 以前は `--packages=external` を使用することでこの問題を回避できましたが、このリリースではこの動作がデフォルトになりました。 この動作を変更する場合は、`--packages=bundle` を指定することでパッケージのバンドルを許可できます。 ただし、`--packages=bundle` を指定してもすべてのパッケージがバンドルされるわけではなく、バンドルが許可されるようになるだけです。 `--packages=bundle` を指定している場合でも、`--external:` を使用して個々のパッケージをバンドルから除外できます。
>
> **依存関係のファイルシステム上での存在:** パッケージがバンドルに含まれなくなったため、バンドルを実行する際に依存関係がファイルシステム上に存在する必要があります。
>
> ### サポート対象外となったプラットフォーム
>
> **サポート終了:** esbuild 0.22.0 は、以下のオペレーティングシステムのサポートを終了しました。
>
> *   Windows 7
> *   Windows 8
> *   Windows Server 2008
> *   Windows Server 2012
>
> これは、Go 言語がバージョン 1.21 でこれらのオペレーティングシステムのサポートを終了したためです。 esbuild のこのリリースでは、Go 1.20 から Go 1.22 に更新されています。
>
> **影響範囲:** この変更は、`esbuild` npm パッケージから提供されるバイナリ実行ファイルのみに影響します。 これらの古いオペレーティングシステム用に esbuild のソースコードをコンパイルすることは可能です。 必要であれば、Go コンパイラの古いバージョン (Go 1.21 より前) を使用して、自分で esbuild をコンパイルできます。
>
> **Node.js の最小バージョンの変更:** esbuild の JavaScript API で必要となる Node.js の最小バージョンが、Node.js 12 から Node.js 18 に引き上げられました。 これは、Go コンパイラが `esbuild-wasm` パッケージ用に生成する JavaScript と、Node.js 17.4 より前のバージョンとの間に非互換性があるためです (具体的には `crypto.getRandomValues` 関数)。
>
> ### その他の破壊的変更
>
> **`await using` の挙動変更:** TypeScript 5.5 で `await using` の挙動が微妙に変更されました。 このリリースでは、esbuild が TypeScript のこれらの変更に合わせて更新されました。

ちょっと長いのですが、前半のデフォルトの挙動が変更される部分で「これは対応せねば」と判断できました。また、今回は自分のリポジトリでは影響なかったのですが、 `await using` のところは普通に見ていたら「見落としていたかも」という感じもします。

そのような感じで、日本保で要点が解説されるだけでもかなりありがたいです。

::: message

ちなみに、esbuild [v0.23.0](https://github.com/evanw/esbuild/releases/tag/v0.23.0) でデフォルトの挙動がやっぱり元に戻るのですが、それはまた別の話ということでここではとくに触れません。

:::

## 対応方法を質問してみる

せっかくなので対応方法も質問してみます。

私が esbuild でビルドするときは `esbuild.config.mfs` などでスクリプト(API)を使うので、API リファレンスなど関係しそうなページをソースとして追加します。

@[card](https://esbuild.github.io/api/)
@[card](https://esbuild.github.io/getting-started/)
@[card](https://esbuild.github.io/)

今回は何個かのリポジトリで [`esbuild.config.mjs`](https://github.com/hankei6km/vscode-ext-serve-run-wasm/blob/4cb8a8e4480e660f2f68bd49980d939036aa4281/esbuild.config.mjs) が影響を受けそうなので、以下のようなテキストをソースとして追加します。(プロンプトから複数行のテキストを入力する方法がわからなかったのでソースにしています)

**リスト 4-1 ****`esbuild.config.mjs`**** をソースとして指定するためのテキスト(実際には ****`---`**** より下はコードブロックにしてあります)**

```md
# 開発中のプロジェクトで使っている `esbuild.config.mjs`

以下のような `esbuild.config.mjs` を使っています。

---
import * as esbuild from 'esbuild'

await esbuild.build({
entryPoints: ['src/extension/extension.ts'],
outfile: 'out/extension.js',
bundle: true,
external: ['vscode'],
sourcemap: true,
platform: 'node',
format: 'cjs',
logLevel: 'info'
})
```

**図 4-1 最終的なソースの一覧**

![ソースの一覧が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/13bbd30a96794880931b362a3ef28aea/merge-dependabot-bump-with-notebooklm-sources.png?w=255\&h=394\&auto=compress%2Cformat)

プロンプトに「開発中のプロジェクトで使っている `esbuild.config.mjs` は 0.22.0 の破壊的変更の影響を受けると思われます。どのように対応すればよいですか？」を入力します。

以下のような返答が表示されます。(長いのでまとめの部分を抜粋)

> まとめ
> 提供された情報源に基づくと、開発中のプロジェクトで使っている `esbuild.config.mjs` は esbuild 0.22.0 の **Node.js をターゲットにした場合のパッケージのバンドル除外** の影響を受ける可能性があります。対応策として、esbuild のバージョンを 0.22.0 未満に固定するか、packages: 'bundle' を設定して依存パッケージをバンドルに含めるようにしてください。

```js
import * as esbuild from 'esbuild'

await esbuild.build({
  entryPoints: ['src/extension/extension.ts'],
  outfile: 'out/extension.js',
  bundle: true,
  // 依存パッケージをバンドルに含める
  packages: 'bundle',
  external: ['vscode'],
  sourcemap: true,
  platform: 'node',
  format: 'cjs',
  logLevel: 'info',
})
```

## 万能というわけではない

テストに使う [`esbuild.test-electron.config.mjs`](https://github.com/hankei6km/vscode-ext-serve-run-wasm/blob/4cb8a8e4480e660f2f68bd49980d939036aa4281/esbuild.test-electron.config.mjs) もあるので対応方法を確認したかったのですが、こちらはちょっと独特なこともありバッチリな回答は得られませんでした。

https://github.com/hankei6km/vscode-ext-serve-run-wasm/blob/4cb8a8e4480e660f2f68bd49980d939036aa4281/esbuild.test-electron.config.mjs

それでも、「`build` API の `bundle: false` について調べてください」など質問しながら対応できたので、いつもより楽に感じました。

## 考慮点

`esbuild.config.mjs` を手動でソースに指定したことは少し手間だと感じました。また、他のファイルが関連することもあるので、できれはデフォルトブランチのツリーをまるっとソースにする対応をしたいところです。

## おわりに

Dependabot の Bump 対応するときに、リリースノートをノートブックのソースにしながら対応方法を考えてみました。

生成された内容を全面的に信用するのも危ないですが、「この辺が引っ掛かりそうかな？」というときに考え方を補強してくれるのは便利でした。

世は大生成 AI 時代なので専用ツールでもっと便利にできそうですが、「ブラウザーで右クリックからの翻訳」くらいのノリでできたので、ズボラな自分にはあっていそうです。
