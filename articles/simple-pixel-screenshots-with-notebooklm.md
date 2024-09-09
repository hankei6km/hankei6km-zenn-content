---
id: simple-pixel-screenshots-with-notebooklm
title: Pixel Screenshots 風なことを NotebookLM でやってみたらわりと返答してくれるのだが、そっと消去されることもある
emoji: 📇
type: idea
topics:
  - notebooklm
  - googledocs
published: true
---

[Pixel Screenshots ](https://store.google.com/intl/en/ideas/articles/pixel-screenshots/)は便利そうですが「Pixel 9 シリーズが前提で日本語ではしばらく使えないらしい」とのことで、「お世話になることはなさそうかな」と思っていました。

しかし、[前回の記事](https://zenn.dev/hankei6km/articles/github-repo-as-notebooklm-source)の「リポジトリのファイルツリーを Google ドキュメントへまとめる」方法を応用したら、「NotebookLM で簡易的なものを作れそうかな」と思いつきました。

以下、思い付いた作り方。

*   スマートフォンのスクリーンショットを Google ドライブのフォルダーへ共有(アップロード)
*   フォルダー内の画像ファイルを定期的に Google ドキュメントファイルにまとめる
*   まとめた Google ドキュメントファイルを元にノートブックを作る

試してみたらわりとあっさりできたのと、少し予想外の問題があったのでその辺についてなど。

## Pixel Screenshots 風とは

まず、本家の Pixel Screenshots について。

残念ながら実際に利用したことはないので、紹介記事に丸投げします。

@[card](https://store.google.com/intl/en/ideas/articles/pixel-screenshots/)
@[card](https://www.itmedia.co.jp/mobile/articles/2408/15/news100.html)

簡単にまとめると、メールやウェブページなどのスクリーンショットを採取しておくことで、自動的に分類したり内容についての確認(質問)をできるようになるアプリです。また、確認した内容によっては、料理のレシピを提案したりカレンダーへ予定を追加するようなこともできるらしいです。

今回は、上記の特徴をすべて実現するのは難しいので、「複数のスクリーンショットについての Q & A(テキスト検索)」ぽいことを NotebookLM と目指してみます。

## スクリーンショットを Google ドライブへアップロード

まずはお試しということで、今回はスクリーンショット採取時の共有メニューからアップロードしています。

**図 2-1 スクリーンショット取得後に共有メニューからドライブへアップロード**

![Android 出スクリーンショットを採取したときの共有メニューを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/19f1bf87a9434daeace7c49940a8b9c6/simple-pixel-screenshots-with-notebooklm-upload.png?w=315\&h=407\&auto=compress%2Cformat)

## 複数のスクリーンショットを Google ドキュメントのファイルにまとめる

これは GAS を使うと簡単にできます。今回は以下のような内容のスクリプトを作成しました。

1.  指定されたフォルダーから画像ファイル一覧を取得
2.  各画像を表示するための署名付き URL を作成
3.  URL などをもとに画像一覧が表示される Markdown テキストを作成
4.  Markdown テキストを Google ドキュメントファイルとして保存

**リスト 3-1 スクリプトのソース**

https://github.com/hankei6km/test-gas-folder-images-to-gdoc/blob/main/code.js

使い方については、[リポジトリの README](https://github.com/hankei6km/test-gas-folder-images-to-gdoc/blob/main/README.md) に記述してあります。GAS でスケジュール実行の設定を経験したことがあれば、とくに難しい部分はないと思います。

**図 3-1 作成された Google ドキュメントのファイル(説明用に手動で「ページ分けなし」にしています)**

![Google ドキュメント内で画像一覧の一部を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ebe3a418ec1e4130834a1f2948c1e59c/simple-pixel-screenshots-with-notebooklm-images-in-gdoc.png?w=450\&h=720\&auto=compress%2Cformat)

## まとめたファイルを元にノートブックを作成して質問してみる

新規にノートブックを作成するときに、ソースとして `memo-images` ファイルを追加すると利用できます。また、Google ドキュメントのファイルなので、画像一覧が更新されたときは(手動ですが)内容の同期もできます。

**図 4-1 ****`memo-images`**** をソースとして追加**

!["memo-images" ファイルをソースとして追加した画面、概要と同期用ボタンなどが表示されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0676ab899007437c988a8dcd4e606be2/simple-pixel-screenshots-with-notebooklm-gdoc-to-source.png?w=702\&h=662\&auto=compress%2Cformat)

今回は、画像を 30 ファイルほど追加してあるファイルで簡単に質問してみます。

### モスの月見フォカッチャはいつから？

![モスバーガーのアプリで月見バーガーの宣伝が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/81620d2632844ebeb9d587cf5050dcad/simple-pixel-screenshots-with-notebooklm-chat-mos-ss.png?w=319\&h=644\&auto=compress%2Cformat)

![ノートブックでの会話で発売日について返答されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/cafa833c448342fb91c992fd171c90a3/simple-pixel-screenshots-with-notebooklm-chat-mos-ans.png?w=1093\&h=321\&auto=compress%2Cformat)

### A 銀行の暗証番号は？

![メモに書き込んである暗証番号のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9e3ced696090466ea9bcee0dcdf5c168/simple-pixel-screenshots-with-notebooklm-chat-pin-ss.png?w=648\&h=637\&auto=compress%2Cformat)

![ノートブックでの会話で暗証番号が返答されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4cc79d8658f64c5c90f11be2cca4b2c0/simple-pixel-screenshots-with-notebooklm-chat-pin-ans.png?w=1091\&h=254\&auto=compress%2Cformat)

### 乗り換えについての画像一覧を作成してください

![Google マップで乗り換え情報を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d555f5b915474baa8b9abbe0f3622677/simple-pixel-screenshots-with-notebooklm-chat-map-ss.png?w=308\&h=622\&auto=compress%2Cformat)

![ノートブックでの会話で上記画像を含む一覧が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9ab2d71f3f6c401786e5b27720502b55/simple-pixel-screenshots-with-notebooklm-chat-map-ans.png?w=1082\&h=416\&auto=compress%2Cformat)

### 工事のお知らせに関連する画像の一覧を作成してください

これは Google ドライブアプリのスキャン画像を利用しています。

![工事のお知らせのポスターをカメラでスキャンした画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ac48ea01b64f4311b41e2d78bd6d40a1/simple-pixel-screenshots-with-notebooklm-chat-info-scan.png?w=474\&h=626\&auto=compress%2Cformat)

![ノートブックでの会話で上記画像を含む一覧が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c04735960b514e6b9ade5e67e50491f5/simple-pixel-screenshots-with-notebooklm-chat-info-ans.png?w=1073\&h=446\&auto=compress%2Cformat)

### 新しい種類のメニューには弱そう

オーストリッチ丼はなぜかローストビーフ丼になる(別のスクリーンショットでもローストビーフ丼になった)。

![オーストリッチ丼の紹介記事を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c8424f4751c94889bd5e9a93a0879fdc/simple-pixel-screenshots-with-notebooklm-chat-donburi-ss.png?w=300\&h=607\&auto=compress%2Cformat)

![ノートブックでの会話ではローストビーフ丼と表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/cee4f3c12f9e4044936b4360798e9c2c/simple-pixel-screenshots-with-notebooklm-chat-donburi-ans.png?w=1085\&h=351\&auto=compress%2Cformat)

月見関連の商品名もわりとおかしい。

![月見ファミリーの宣伝記事を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c0f3640ca33b4d6cacdd13644b53aefd/simple-pixel-screenshots-with-notebooklm-chat-tsukimi-ss.png?w=257\&h=478\&auto=compress%2Cformat)

![ノートブックでの会話では月見ファミリーのメニューが一部間違っているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3eeded353fc74e5ea1f1b0d8b8d0489c/simple-pixel-screenshots-with-notebooklm-chat-tsukimi-ans.png?w=1082\&h=434\&auto=compress%2Cformat)

## 数日使って気がついた点

実際に使って見た様子としては上記の通りで、「期待していた返答を得られるがそれなりに間違いもある」といった感じです。

ここでは、その他に気がついた点を書き出してみます。

*   これを書いている時点でソース用のファイルはそこそこ大きいが、 ファイル後半の方の画像でも認識している

    *   ファイルサイズは約 31.5 MB
    *   画像は 50 ファイル程度
    *   おそらく、画像はソース同期時にテキスト化されているのかなと

*   スクリーンショットをアップロードするのはわりと面倒

    *   ネットワーク環境に左右されるので少しストレスになる

*   動作はやはり重い

    *   返答に時間がかかる他にソースの同期も重い
    *   その代わりに他のデバイスや PC でも質問できるのは扱いやすくて良い

*   ウェブページのスクリーンショットの情報がヒットしたときは元のページを参照したくなる

    *   Pixel Screenshots ではこれができるらしい

*   直接の質問で返答されないときは、「一覧を作ってください」から絞り込んでいく

    *   例:「A 銀行の暗証番号は？」でダメなときは「暗証番号の一覧を作ってください」とする

*   うまく認識されていた画像がソース同期後に認識されなくなることがある(その逆もある)

*   画像内の数字や日付についても(事前の予想よりかは)答えてくれる

    *   そうは言っても間違えるのでチェックは必要です
    *   記事のスクリーンショットなどを OCR 的に読み込むのは無理そう

*   新しい種類の単語には弱い

    *   さきほどの例のように吉野家のオーストリッチ丼は(画像内にテキストがあっても)商品名を間違え続けた
    *   おそらくは学習内容(って表現でいいのかな？)に含まれていない単語には弱いのだと予想

*   取り込まれない画像がある(詳細は後述します)

いまのところ大きく不便なところもないのですが、やはりローカル LLM を利用していないことに起因するデメリットも目立ちます。そういった点ではローカルの LLM で動くらしい本家 Pixel Screenshots が少し羨ましくなってきました。(ちょっと本末転倒な感じですね)

あとは、次のセクションに分けましたが、画像の内容に敏感なところは少し悩ましい部分です。

## ソースに取り込まれない画像がある

具体的なルールを見つけることはできなかったのですが、いくつか取り込まれない画像があることに気が付きました。

少し試してみたところ、画像内に人物の写真やイラストが配置されていると取り込まれないようです。また、これには漫画風にデフォルメされた人物やヒューマノイドロボット、異星人なども含まれるようです。 (○ボコップはダメで○ーミネーターの骨格を画像検索しているスクリーンショットは取り込むので判定基準はイマイチ不明ですが)

**図 6-1 Google ドキュメントでは画像検索のスクリーンショットが表示されている**

![人物を含む画像が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d257f995abc54871b21ad65273cad933/simple-pixel-screenshots-with-notebooklm-erase-gdoc.png?w=652\&h=753\&auto=compress%2Cformat)

**図 6-2 同期したソースでは消去されている**

![人物を含む画像が表示されていないスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2957df1316ae4c3abc0cdb84966ff6c3/simple-pixel-screenshots-with-notebooklm-erase-source.png?w=648\&h=480\&auto=compress%2Cformat)

取り込まれないことについてはプライバシーや各種配慮から必要だとは思いますが、とくに通知なく消去されるのは少し困ってしまいます。 (「期間限定メニューの紹介ページで人物が含まれている」みたいな場合もダメなのでわりと困ります)

他にも取り込まれない種類の画像があるかもしれないので、この辺は意識して利用するのが良いかと思います。

あとは、今回のことにあまり関係ありませんが、ノートブック本来の使い方でありそうな「伝記や偉人伝を執筆するための資料整理」みたいなとき、人物のイラストを扱えないと困るのでないかなと少し思いました。

## 改善点について

今回作った仕組みはわりと良い感じになったので個人的にはしばらくは使ってみようかと思っています。そこで、改善点について少し考えてみます。

### Google ドキュメントファイルの更新の効率化

今回は Markdown テキストを作成し Google ドキュメントへ変換していますが、GAS の [`DocumentApp`](https://developers.google.com/apps-script/reference/document/document-app?hl=ja) を使うと部分的な更新が可能になると思われます。

あとは、定期実行も効率がわるいので、スクリーンショット保存時に処理を起動できるよう対応しておくとファイルを即時更新できるかと思います。

### 従来的な OCR との併用

Google ドライブの OCR で画像ファイルをテキスト化できるので、Google ドキュメントへまとめるときに OCR テキストも含めると返答の精度がよくなるのかなと予想しています。

### スクリーンショットを Notion へ保存する

スクリーンショットを Google ドライブへアップロードするときに共有メニューを使っていますが、ここから Notion へのアップロードもできます。 (ネットワークが低速だと失敗しやすくリトライしてくれないのは難点ですが)

こちらはデータベースのページとして保存されるので、普段 Notion を使っているならコメントの追加などもやりやすいかと思います。

**Notion に保存したスクリーンショット**

![スクリーンショットを保存した複数のページを Notion でギャラリー表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e887d05c34344555bed9ccf70a520a28/simple-pixel-screenshots-with-notebooklm-notion.png?w=999\&h=454\&auto=compress%2Cformat)

Notion のメモを Google ドキュメントへ保存する処理は以前にも作ったので、将来的にはこちらに移行しようかなと少し考えています。

## おわりに

複数のスクリーンショットを Google ドキュメントのファイルとしてまとておき、ノートブックから利用することで簡易的な Pixel Screenshots 風チャットボットを作ってみました。

NotebookLM 特有のクセはありますが、思っていたよりもスクリーンショットについての質問に答えてくれる印象です。とくに、各アプリから期間限定メニューなどの情報をスクリーンショットで保存しておいて、あとから一箇所で検索できるのは良いなと思いました。

ただし、ネットワーク利用に関連するデメリットや扱えない種類の画像があるので、(ローカル LLM で動く)本家 Pixel Screenshots が少し羨ましくなるという想定外なこともありました。

以上のような感じですが個人的にはちょっと気に入ったので、しばらく今回の仕組みを利用してみようかと思っています。
