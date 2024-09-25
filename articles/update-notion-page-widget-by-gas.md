---
id: update-notion-page-widget-by-gas
title: Notion のページウィジェットの表示を GAS で更新
emoji: 🧩
type: idea
topics:
  - notion
  - googleappsscript
published: true
---

@[card](https://k-tai.watch.impress.co.jp/docs/column/minna/1625346.html)

プロの人の参考になるような上手な使い方はしていませんが、Notion のページウィジェットの表示を GAS で更新する方法はいずれ記事を作りたいと思っていたので便乗してみます。

## Notion のページウィジェットとは？

正式な名称は知りませんが、Notion のウィジェット一覧で「ページ」と名前の付いているウィジェットを指しています。これをホーム画面に配置すると指定したページ(データベース)のカバー画像とタイトルおよびページアイコンが表示されます。

**図 1-1 Notion のウィジェット**

![Notion で利用できるウィジェットの一覧に「ページ 2x2」というタイトルのタイル状のウィジェットが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/389a0d048c4346239308bce1c86a9142/update-notion-page-widget-by-gas-page.png?w=1080\&h=2289\&auto=compress%2Cformat)

## ページウィジェットを配置してみる

**図 2-1 ホーム画面**

![ホーム画面に Notion のページウィジェットを配置してあるスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7648375ac7ef49e2a3e3fcf8fd5e9e5a/update-notion-page-widget-by-gas-home.png?w=1080\&h=1913\&auto=compress%2Cformat)

今使っているスマホのホーム画面、Notion 関連(一部 Google Keep)はこのような感じになっています。

2x2 のタイル状に並んでいるのが Notion のページウィジェットです。左上は天気の情報をタイトルと絵文字へ設定しています(カバー画像も更新していたのですが、Unsplash Source の機能が停止したのでいまは放置しています)。右上は[フィードリーダー](https://github.com/hankei6km/gas-feed2notion)更新時に、更新された記事からランダムにタイトルとカバー画像を設定しています。

左下はメモリストのデータベースで、カバー画像などは固定表示です(今回の仕組みは適用していません)。また、今回の話から少し外れますが、右下の「＋」はクイックメモウィジェットでここからメモを新規作成できます。

## ページウィジェットの表示を GAS で更新する方法

とくに捻りもなく、ウィジェットに指定してあるページ(またはデータベース)を API で更新しているだけです。Notion のサーバー側のデータが更新されると、ページウィジェットの表示も(数時間くらい遅れることもありますが)自動的に更新されるのでこれを利用しています。

少し工夫した点としては、API を直接使うとページとデータベースで設定方法が微妙に異なっていたりとめんどうだったので、GAS のライブラリーを作りました。コードエディターからライブラリーを追加することで、カバー画像などを 10 行程度のコードで更新できるようになります。

https://github.com/hankei6km/gas-notion-update-header

## お天気ウィジェット風にする

先のスクリーンショットで配置していたページウィジェットに使っているスクリプトです。

https://github.com/hankei6km/test-gas-weather-to-header/blob/main/code.js

@[card](https://github.com/hankei6km/test-gas-weather-to-header)

フィードリーダーの方は割愛しますが、[Transformer 関数](https://github.com/hankei6km/gas-feed2notion#transformer)で利用する記事を決定しておいて、先ほどのライブラリーを使うことで実現できます。

## 実用面では？

実を言うとあまり無くて、「Notion の特定のページを大きなボタンから開くことができる」くらいです。しかも、Android 版の Notion は動かしたままだとページ切り替えが怪しくなってくるので、場合によっては目的のページが開かないこともあります。 (しかたないので、時々アプリを手動で終了させています)

私の場合は、Notion でよく使うページはウィジェットで指定しているページと新規のメモ作成なのでそこそこ便利ですが、使い込んでいる人には物足りないかもしれません。

ちなみに、天気情報を表示しているページウィジェットは、リマインド設定しているメモの一覧やカレンダーなどを表示するページになっています(ブラウザー版のホームに似た感じです)。

**図 5-1 期日が過ぎた or 迫っているメモが表示される**

![タイトルとページアイコンが天気情報でリマンドが設定されているメモリストが表示されているページのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4e7402a7c146431ab1b8fc86ae1ece3e/update-notion-page-widget-by-gas-howm.png?w=1080\&h=1780\&auto=compress%2Cformat)

## デスクトップ環境のブラウザー版の場合

Android のウィジェットは関係ない話ですが、サイドバーに小さく天気が表示されるので、メモを書いているときにちらっと天気などを確認できるようになります。

**図 6-1 サイドバーに天気やフィードの情報が表示される**

![ブラウザー版 Notion のサイドバーに表示されているタイトル一覧の中に天気やニュースが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0094d6f9e31f4848ae33879f6707dd55/update-notion-page-widget-by-gas-sidebar.png?w=264\&h=208\&auto=compress%2Cformat)

あとは、カバー画像を設定してあるとブラウザー版ホームの表示も少しにぎやかになります。

**図 6-2 最近のアクセスにカバー画像が表示される**

![ブラウザーのホームを開いているスクリーンショット、最近アクセス下ページのサムネイルとしてカバー画像が表示されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/512c88195c204196bd382ff191eb1676/update-notion-page-widget-by-gas-browser.png?w=983\&h=365\&auto=compress%2Cformat)

## おわりに

Notion のページウィジェットの表示を GAS で更新してみました。

普段使っているスマホではこれを設定するようになって 1 年半で、「すげぇ便利でもないけれど無いと寂しい」くらいの位置づけになっています。

少し不満な点としては、ページウィジェットのサイズを 4x3 とかに設定できるとうれしいのですが、これは Notion アプリに期待ということで。
