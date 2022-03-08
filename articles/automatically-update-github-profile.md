---
id: automatically-update-github-profile
title: GitHub の Profile ページを GitHub Actions と zx で更新し、Zenn 記事一覧や小倉百人一首などを埋め込む
emoji: ⏰
type: idea
topics:
  - zenn
  - github
  - githubactions
  - zx
published: true
---

こちらの記事を見かけて「面白そう」「[zx] での並列的な処理の練習になりそう」ということで自動更新を試してみました。

@[card](https://zenn.dev/kawarimidoll/articles/283179cffd2ef6)

## 全体像

現状では以下のような感じになっています。よく見かける構成ですが、Zenn 記事の一覧にアイキャッチ画像(emoji)を表示させています。

![Profile ページ全体のスクリーンショット](/images/automatically-update-github-profile/automatically-update-github-profile-overview.png)
*全体の表示*

:::details (クリックで GitHub Mobile でのサンプルを表示)

![Profile ページを GitHub Mobile で表示したスクリーンショット](/images/automatically-update-github-profile/automatically-update-github-profile-mobile.png)
*GitHub Mobile 上での表示*

:::

## リポジトリ

@[card](https://github.com/hankei6km/hankei6km)

## テンプレート

スクリプトで作成した要素を埋め込めるように以下のようなテンプレートを作成しました。書式については [unified.js] などで処理するか少し悩んだのですが、単純に「キーワードを文字列置換で置き換えて埋め込む」ような書式にしました[^directive]。

:::details (クリックで表示)

▼ *リスト 3-1 テンプレート*

```md:templates/README-template.md
<p align="center">

![()=>hankei6km](:replace{#header})

</p>

<h2>
<img width="24" height="24" style="height:1em;width:1em;margin:0 0.05em 0 0.1em;vertical-align:-0.1em;"
 src="assets/images/github.svg" />
hankei6km's Github Stats
</h2>

<p align="center">
 <a href="https://github.com/anuraghazra/github-readme-stats">
  <img width="457" alt="hankei6km's GitHub stats" src="https://github-readme-stats.vercel.app/api?username=hankei6km&show_icons=true">
 </a>
 <a href="https://github.com/anuraghazra/github-readme-stats">
  <img width="382" src="https://github-readme-stats.vercel.app/api/top-langs/?username=hankei6km&layout=compact">
 </a>
</p>

<h2>
<img width="24" height="24" style="width:1em; height:1em; margin: 0 .05em 0 .1em; vertical-align: -0.1em;" src="assets/images/zenn.svg">
Recent posts from Zenn
</h2>

:replace{#zenn-articles}

<h2>
<img width="24" height="24" style="width:1em; height:1em; margin: 0 .05em 0 .1em; vertical-align: -0.1em;" src="https://twemoji.maxcdn.com/v/13.1.0/72x72/1f5bc.png">
Recent deck from mardock
</h2>

<p align="center">
:replace{#mardock-cards}
</p>

<h2>
<img width="24" height="24" style="width:1em; height:1em; margin: 0 .05em 0 .1em; vertical-align: -0.1em;" src="https://twemoji.maxcdn.com/v/13.1.0/72x72/1f38e.png">
Today's Ogura Hyakunin Isshu
</h2>

:replace{#ogura-shuffle}

<details>
<summary>credit</summary>

- Title: 小倉百人一首かるたデータ
- Author: [Nanako Takahashi](http://linkdata.org/user/tnanako)
- Source: http://linkdata.org/work/rdf1s6834i
- License: http://creativecommons.org/licenses/by/3.0/deed.ja

</details>
```

:::

[^directive]: キーワードは [directive](https://github.com/micromark/micromark-extension-directive#syntax) のような感じになっていますが、これは見た目だけです。

## zx の利用

セクションの内容の生成はシェルスクリプトでもいけそうな感じでしたが、以下の理由で [zx] を利用しています。

*   HTML の組み立てに [`hastscript`] を使いたい
*   [zx] と自作モジュール [`chanpuru`] での並列的な処理を練習したかった

少しオーバースペックですが、実行数を制限しながら並列的に API を利用したり、ファンアウト - ファンイン的な処理を実現しています[^paralle]。

[^paralle]: この辺は普通に [GNU Parallel](https://www.gnu.org/software/parallel/) などを使った方が幸せになれそうですが、それはそれということで。

## 各セクションの生成

今回は以下のようにセクションの内容を生成しています。

### ヘッダー画像

以下の画像のファイル名をランダムに埋め込んでいます。とくに非同期で実行する必要はないのでテンプレート読み込み後にそのまま処理しています。

![ヘッダー画像 1](https://github.com/hankei6km/hankei6km/blob/main/assets/images/header1.jpg?raw=true)
![ヘッダー画像 2](https://github.com/hankei6km/hankei6km/blob/main/assets/images/header2.jpg?raw=true)
![ヘッダー画像 3](https://github.com/hankei6km/hankei6km/blob/main/assets/images/header3.jpg?raw=true)

▼ *リスト 5-1 ヘッダー画像埋め込み*

```ts:src/gen-readme.ts
let md = (await readFile('templates/README-template.md')).toString('utf-8')

const header = Math.floor(Math.random() * (4 - 1) + 1)
md = md.replace(
  `:replace{#${'header'}}`,
  `assets/images/header${header}.jpg`
)
```

### GitHub Stats

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b641c687f59647839dc0cba5a5de261a/automatically-update-github-profile-overview.png?auto=compress%2Cformat\&border64=MSw1NTAwMDAwMA\&rect64=NDQwLDQ0MCw4ODAsMzEw)

ここは自前での生成ではなく以下の記事などを参考に定番サービスを利用しています。ただし、カラーテーマ対応などで少しアレンジしました。

@[card](https://zenn.dev/yutakatay/articles/kirakira-github-profile)
@[card](https://github.com/anuraghazra/github-readme-stats)

### カラーテーマの切り替え

![ダークモードではカラーテーマ対応が切り替わっている状態のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e3b571fdd57048228af4a571657dbd1c/automatically-update-github-profile-overview-dark.png?auto=compress%2Cformat\&border64=MSw1NTAwMDAwMA\&rect64=NDQwLDQ0MCw4ODAsMzEw)*▲ 図 5-1 ダークモードでの表示*

GitHub のカラーテーマには「[Specifying the theme an image is shown to](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#specifying-the-theme-an-image-is-shown-to)」の方法で対応できます。しかし、「外部の画像」「リンク付きの画像」では機能しませんでした。

*   外部の画像 - 外部の画像は[匿名化プロキシ](https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/about-anonymized-urls)でファイル名の変更でフラグメント識別子が消える
*   リンク - おそらくはリンクにもフラグメント識別子が必要(GitHub が自動的に追加するリンクでは機能する)

どちらもセレクターに合致しなくなることが原因と思われます。

よって、いったん `.svg` ファイルをダウンロードしてローカルの画像として表示させています[^proxy]。また、[GitHub Readme Stats](https://github.com/anuraghazra/github-readme-stats) へのリンクは画像とは別に追加しました。

▼ *リスト 5-2 カラーテーマ対応の記述*

```md
<p align="center">

<img width="457" alt="hankei6km's GitHub stats" src="assets/images/stats-dark.svg#gh-dark-mode-only">
<img width="457" alt="hankei6km's GitHub stats" src="assets/images/stats-light.svg#gh-light-mode-only">
<img width="382" alt="Top Langs" src="assets/images/top-langs-dark.svg#gh-dark-mode-only">
<img width="382" alt="Top Langs" src="assets/images/top-langs-light.svg#gh-light-mode-only">

</p>

Generated by [GitHub Readme Stats](https://github.com/anuraghazra/github-readme-stats)
```

[^proxy]: 最初は「リアルタイムな更新にならない」と思って見送っていたのですが、よく考えたら「プロキシが入るなら似たような感じかな」ということで対応しました。

### 画像サイズ

(他の画像にも当てはまりますが)`<img>` で `width` と `height` 属性を指定すると、画像が縮小されるモバイル環境では縦横比や SVG の余白などに影響がありました。

![画像の上下に余白ができてしまっているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b5d17a79af8f42a1ab842c57975f80be/automatically-update-github-profile-size.png?auto=compress%2Cformat\&border64=MSw1NTAwMDAwMA)*▲ 図 5-2 GitHub Mobile での表示(上下に余白ができる)*

これは `style` あたりで回避出来ればよいのですが書式に制限がるのと [GitHub Mobile] で少し挙動が違うので良い回避策は思いつきませんでした。とりあえずは `width` 属性のみを指定すると回避できたのですが、この方法はレイアウトシフトが発生します。

そこで「みなさんどうやって解決しているのだろう？」と「[Awesome GitHub Profile README](https://github.com/abhisheknaiidu/awesome-github-profile-readme)」からざっと確認してみたところ、とくに定番的な手法はなさそうな感じです。

よって、今回は「各環境で見た目が不自然にならない」ことを優先して `width` 属性のみの指定になっています。

### Zenn 記事の一覧

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b641c687f59647839dc0cba5a5de261a/automatically-update-github-profile-overview.png?auto=compress%2Cformat\&border64=MSw1NTAwMDAwMA\&rect64=NDQwLDczNiw4ODAsMjI1)

Zenn 記事の一覧自体は冒頭の記事にあるように RSS フィードで取得できます。しかし、RSS フィードにはアイキャッチ(絵文字)情報が含まれていなったので以下のようなコマンドを作成しました。

1.  記事のファイル名(slug)を使って GitHub 上の記事ソースから絵文字を取得
2.  「[GitHubのREADMEでもTwemojiを使いたいよ〜！](https://zenn.dev/kawarimidoll/articles/4183e072266140)」記事を参考に API を使って絵文字から [Twemoji] の画像 URL を取得
3.  [`hastscript`] で HTML へ変換

出力される HTML のフォーマットは固定ですが、使い方はシンプルです。

以下の環境変数を設定してコマンドを実行すれば結果の HTML が出力されます。

*   `GITHUB_TOKEN` - GitHub の PAT
*   `ACCOUNT` - Zenn のユーザー名
*   `OWNER` - GitHub のユーザー名
*   `REPO` - Zenn と連携しているリポジトリ名

また、コマンドのフラグとしては以下が指定できます。

*   `--worker-num` - GitHub から記事を取得するときの API 同時実行数
*   `--limit` - 出力する記事タイトルの件数

▼ *図 5-3 Zenn 記事タイトル一覧の生成*

```shell-session
$ zx dist/zenn-articles.js --limit 5
<ul><li><a href="https://zenn.dev/hankei6km/articles/promise-or-abort-controller"><img style="width:1.1em; height:1.1em; margin: 0 .5em 0 .1em; vertical-align: -0.1em;" width="18" height="18" alt="🎬" src="https://twemoji.maxcdn.com/v/13.1.0/72x72/1f3ac.png"> 順次実行される非同期処理(コンテキスト)を停止させるには Promise か AbortContoller か？</a></li><li><a href="https://zenn.dev/hankei6km/articles/promise-memo"><img style="width:1.1em; height:1.1em; margin: 0 .5em 0 .1em; vertical-align: -0.1em;" width="18" height="18" alt="📝" src="https://twemoji.maxcdn.com/v/13.1.0/72x72/1f4dd.png"> ふわっとした理解で Promise(と Async Generator) を使っていたらいろいろハマってしまったのでメモ</a></li><li><a href="https://zenn.dev/hankei6km/articles/github-release-note-generator-to-various-purposes"><img style="width:1.1em; height:1.1em; margin: 0 .5em 0 .1em; vertical-align: -0.1em;" width="18" height="18" alt="⚗️" src="https://twemoji.maxcdn.com/v/13.1.0/72x72/2697.png"> GitHiub のリリースノート自動生成機能をリリースノート以外にも使ってみる</a></li><li><a href="https://zenn.dev/hankei6km/articles/completion-label-names-in-github-cli"><img style="width:1.1em; height:1.1em; margin: 0 .5em 0 .1em; vertical-align: -0.1em;" width="18" height="18" alt="🏷️ " src="https://twemoji.maxcdn.com/v/13.1.0/72x72/1f3f7.png"> GitHub CLI でラベル名の入力補完をできるようにした</a></li><li><a href="https://zenn.dev/hankei6km/articles/credentials-contained-files-on-github-actions"><img style="width:1.1em; height:1.1em; margin: 0 .5em 0 .1em; vertical-align: -0.1em;" width="18" height="18" alt="🔏" src="https://twemoji.maxcdn.com/v/13.1.0/72x72/1f50f.png"> GitHub Actions でクレデンシャルなどが含まれるファイルを扱うときのアレコレ</a></li></ul>
```

なお、[Twemoji] の表示サイズ指定は [GitHub Mobile] の挙動にあわせて `width` `height` 属性で行っています。

### mardock スライドのサムネイル

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b641c687f59647839dc0cba5a5de261a/automatically-update-github-profile-overview.png?auto=compress%2Cformat\&border64=MSw1NTAwMDAwMA\&rect64=NDQwLDkzNiw4ODAsMjUw)

個人的に作った Web アプリの RSS フィードを取得しているだけですが、これも結果を HTML で出力するのでコマンドを作成しました。

なお、ここでも画像サイズについての問題が出てきますが、「モバイル環境でも縮小されないだろう値」を指定することで回避しています。

▼ *図 5-4 mardockの スライドサムネイルの生成*

```shell-session
$ zx dist/mardock-card.js 
<a href="https://hankei6km.github.io/mardock/deck/2022-03-in-outdoor-150"><img alt="ジョグメモ 150" src="https://hankei6km.github.io/mardock/assets/deck/2022-03-in-outdoor-150/2022-03-in-outdoor-150.png" width="270" height="152"></a>
<a href="https://hankei6km.github.io/mardock/deck/2022-02-in-outdoor-149"><img alt="ジョグメモ 149" src="https://hankei6km.github.io/mardock/assets/deck/2022-02-in-outdoor-149/2022-02-in-outdoor-149.png" width="270" height="152"></a>
```

### 小倉百人一首をランダムに取得

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b641c687f59647839dc0cba5a5de261a/automatically-update-github-profile-overview.png?auto=compress%2Cformat\&border64=MSw1NTAwMDAwMA\&rect64=NDQwLDExNjQsODgwLDIwMg)

[小倉百人一首かるたデータ]の内容をランダムに返す API を [SSSAPI] と [Google Apps Script]\(スプレッドシート)で作成し、さらに HTML を出力するコマンドを作成しました。

なお、元のデータは LOD(Linked Open Data)なので、表示の方法はいろいろ工夫できそうですが今回は単純に `<a>` で表示しています[^lod]。

▼ *図 5-5 ランダムな小倉百人一首*

```shell-session
$ zx dist/ogura-shuffle.js 
<h3>侘びぬれば 今はた同じ 難波なる</h3>
<p><details><summary>下の句と情報</summary><p>身をつくしても 逢はむとぞ思ふ</p><p>(わびぬれば いまはたおなじ なにわなる　みをつくしても あわんとぞおもう)</p><ul><li>歌人 - <a href="http://linkdata.org/resource/rdf1s6833i#kajin_020">http://linkdata.org/resource/rdf1s6833i#kajin_020</a></li><li>読札 - <a href="https://commons.wikimedia.org/wiki/File:Hyakuninisshu_020.jpg">https://commons.wikimedia.org/wiki/File:Hyakuninisshu_020.jpg</a></li><li>異なる記録形式 - <a href="http://linkdata.org/resource/rdf1s8931i#audio_nhk_020">http://linkdata.org/resource/rdf1s8931i#audio_nhk_020</a></li></ul></details></p>
```

[^lod]: LOD は <https://www.nic.ad.jp/ja/materials/iw/2014/proceedings/s15/s15-takeda.pdf> などを読んでみると面白そうなので、いずれなにか挑戦したいところです。

## README の生成

各セクションの内容を生成するコマンドを用意できたのでテンプレートに埋め込みます。

これも [zx] で以下のようなコマンドを作成しています。

1.  ヘッダーと GitHub Stats は順次実行
2.  HTML を生成する核コマンドは [zx] の `$` で同時に実行(Promise を作成)する
3.  Promise から結果を受け取ったらテンプレートへ埋め込む

このような場合 `Promise.all` を使うのが定番ですが、今回は [`Chan`] と [`select()`] を利用することで「結果が確定したものは即座に変換処理を実行」しています。

▼ *リスト 6-1 select() による fan-in 的処理*

```ts:src/gen-readme.ts
const s = {
  'zenn-articles': zennArticles(errCh.send),
  'mardock-cards': mardockCards(errCh.send),
  'ogura-shuffle': oguraShuffle(errCh.send)
}

for await (const [key, i] of select(s)) {
  if (!i.done) {
    md = md.replace(`:replace{#${key}}`, i.value.stdout)
  }
}
```

## ワークフロー

`README.md` の生成ができるようになったのであとは「GitHub 上で定期的に実行」「生成された `README.md` をコミット」するワークフローを作成したら完成です。

今回は、冒頭の記事を参考にして以下のようにしました。

### 定期的な実行

以下のようにしています。

*   `workflow_dispatch` - 手動実行するために指定
*   `schedule` - 毎日 23:55(日本時間)に実行

とくにキッチリと動かしたいわけでもないので、毎時の開始時点からズラしています(それでも少し待ってから実行されています)。

@[card](https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows#schedule)

> ノート: scheduleイベントは、GitHub Actionsのワークフローの実行による高負荷の間、遅延させられることがあります。 高負荷の時間帯には、毎時の開始時点が含まれます。 遅延の可能性を減らすために、Ⅰ時間の中の別の時間帯に実行されるようワークフローをスケジューリングしてください。

▼ *リスト 7-1 定期的な実行の定義*

```yaml:
on:
  workflow_dispatch:
  schedule:
    - cron: '55 14 * * *'
```

▼ *図 7-1 実際に開始された時刻*

    2022-03-07T15:09:56.1781180Z Requested labels: ubuntu-latest
    2022-03-07T15:09:56.1781240Z Job defined at: hankei6km/hankei6km/.github/workflows/gen-readme.yaml@refs/heads/main

### コミット

冒頭記事のステップをほぼ流用させていただいたのですが、オプションは少し変えました(`41898282`については [GitHub Actions bot email address?](https://github.community/t/github-actions-bot-email-address/17204) あたりなど)。

▼ *リスト 7-2 コミット処理の定義*

```yaml
- name: Git commit
  run: |
    git config user.name github-actions[bot]
    git config user.email 41898282+github-actions[bot]@users.noreply.github.com
    if [ -n "$(git status --porcelain)" ]
    then git commit -am 'Generate README.md' && git push origin
    else echo 'nothing to commit, working tree clean'
    fi
```

## その他

自動更新とは少し離れますが、レイアウトなどを調整するときに少しハマったところをメモしておきます。

### 見出しのアイコン

見出しの GitHub と Zenn のアイコンは [Simple Icons] を利用しています[^zenn-icon]。

画像の `style` 属性で色をつけるのが難しかったので、以下のように `.svg` へ直接 `style` を追加しています[^svg]。

```svg:assets/images/zenn.svg
<!-- https://simpleicons.org/ からダウンロードしたファイルにスタイルを追加 -->
<svg style="fill:#3EA8FF;" role="img" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><title>Zenn</title><path d="M.264 23.771h4.984c.264 0 .498-.147.645-.352L19.614.874c.176-.293-.029-.645-.381-.645h-4.72c-.235 0-.44.117-.557.323L.03 23.361c-.088.176.029.41.234.41zM17.445 23.419l6.479-10.408c.205-.323-.029-.733-.41-.733h-4.691c-.176 0-.352.088-.44.235l-6.655 10.643c-.176.264.029.616.352.616h4.779c.234-.001.468-.118.586-.353z"/></svg>
```

GitHub のアイコンはダークとライト用を用意し、「[Specifying the theme an image is shown to](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#specifying-the-theme-an-image-is-shown-to)」の方法でカラーテーマに対応していてます。

```html
<h2>
<img width="24" height="24" style="height:1em;width:1em;margin:0 0.05em 0 0.1em;vertical-align:-0.1em;"
 src="assets/images/github-dark.svg#gh-dark-mode-only" />
<img width="24" height="24" style="height:1em;width:1em;margin:0 0.05em 0 0.1em;vertical-align:-0.1em;"
 src="assets/images/github-light.svg#gh-light-mode-only" />
hankei6km's Github Stats
</h2>
```

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c63cf44087cb4fdbb993c28de9a6197c/automatically-update-github-profile-heade-icon-light.png?auto=compress%2Cformat)

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/24f349043bef461fa0120bf8f8aef0ba/automatically-update-github-profile-header-icon-darrk.png?auto=compress%2Cformat)

[^zenn-icon]: Zenn アイコンは <https://zenn.dev/nekocodex/articles/ab915845d300c6> で登録していただいたようです。

[^svg]: <https://zenn.dev/qsf/articles/a4c1b527e77bf6> の方法を利用したかったのですが、どうにもうまくいかなかったので直接編集しました。(あとは、ここでも [GitHub Mobile] の名前が出ていたので…)

### 少し縮小される

Profile ページでの表示は「リポジトリ上で `REAME.md` を表示した場合にくらべて少し縮小される」ようです(フォントサイズの指定などが数 px 小さい)。

### Markdown レンダリングの挙動は GitHub 全体で統一されてるわけではない

気が付いた限りでは以下の場合に「ブラウザーでリポジトリの `README.md` を表示させたとき」と挙動が異なっていました(`<img>` の `style` 属性で指定した `width` などが適用されない)。

*   gist のプレビュー表示
*   [GitHub Mobile] での表示(Android 版で確認しています)

今回の `README.md` を作成するときに最初は gist で確認していたのですが、「プレビューのときだけ」挙動が異なるので少し混乱してしまいました。細かく表示を確認する場合は `draft.md` などを作成してリポジトリにアップロードするのが無難かと思われます。

![gist のプレビューで画像が大きく表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1ced9d84db2f4974a43f793757a89d77/automatically-update-github-profile-gitst-preview.png?w=600\&h=292\&auto=compress%2Cformat)*▲ 図 8-1 gist のプレビューでは画像の style が適用されていない*

## おわりに

`README.md` を定期的に更新し、あわせて外部の情報を埋め込むためにくつか外部の API を利用してみました。

API を実行すること自体は「GitHub Actions で静的ページを生成する」のと同じ要領なので固有のハマりところは無かった印象です。組み合わせる API やワークフロー実行のトリガーを工夫することにより、いろいろな面白い Profile ページが作れると思われます[^coffee]。

[^coffee]: 「[懐かしのコーヒーポットの画像配信](https://ja.wikipedia.org/wiki/%E3%83%88%E3%83%AD%E3%82%A4%E3%81%AE%E9%83%A8%E5%B1%8B%E3%81%AE%E3%82%B3%E3%83%BC%E3%83%92%E3%83%BC%E3%83%9D%E3%83%83%E3%83%88)」「定期的に更新される[ライフゲーム](https://ja.wikipedia.org/wiki/%E3%83%A9%E3%82%A4%E3%83%95%E3%82%B2%E3%83%BC%E3%83%A0)」みたいなのも楽しそうです。

[zx]: https://github.com/google/zx 

[unified.js]: https://unifiedjs.com/

[GitHub Mobile]: https://github.com/mobile

[Twemoji]: https://twemoji.twitter.com/

[`hastscript`]: https://github.com/syntax-tree/hastscript

[`chanpuru`]: https://github.com/hankei6km/chanpuru 

[SSSAPI]: https://sssapi.app/

[Google Apps Script]: https://workspace.google.co.jp/intl/ja/products/apps-script/

[小倉百人一首かるたデータ]: http://linkdata.org/work/rdf1s6834i

[`Chan`]: https://github.com/hankei6km/chanpuru/blob/main/docs/classes/Chan.md

[`select()`]: https://github.com/hankei6km/chanpuru/blob/main/docs/modules.md#select

[Simple Icons]: https://simpleicons.org/
