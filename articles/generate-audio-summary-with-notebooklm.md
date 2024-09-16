---
id: generate-audio-summary-with-notebooklm
title: NotebookLM が小粋な会話風の解説音声(英語)を作ってくれるらしい
emoji: 🎧
type: idea
topics:
  - notebooklm
published: true
---

NotebookLM に表題のような機能が追加されたらしい。

@[card](https://blog.google/technology/ai/notebooklm-audio-overviews/)

機能の内容については上記ブログを見ていただくのがよいのですが、私は最初にこの機能を知ったとき「アメリカンな感じのゆっくり解説音声(英語)ですね」と思いました。(共通点は「二人で会話している」くらいですが)

@[card](https://dic.nicovideo.jp/a/%E3%82%86%E3%81%A3%E3%81%8F%E3%82%8A%E8%A7%A3%E8%AA%AC)

面白そうなので少し試してみます。

## 作ってみる

NotebookLM をすでに使っている前提で書きます。

1.  NotebookLM でノートブックを作成または用意します
2.  ソースを追加するか、右下の「ノートブックガイド」をクリックします
3.  表示された画面の右上に「音声の概要」が表示されます
    *   新規に作成したノートブックでは表示されないことがあり、リロードで表示されました
4.  「生成」ボタンをクリックすると会話風の解説音声の生成が始まります

**図 1-1 生成用のボタンの配置など**

![ノートブックガイドを開くためのボタンと、表示されている生成ボタンに赤い枠を付けたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3ca5030f8ab945e68c14d156bf003182/generate-audio-summary-with-notebooklm-generate.png?w=1280\&h=685\&auto=compress%2Cformat)

今回はソースとして GitHub で利用できる各種 URL のメモ(Google ドキュメント 9 ページ程度)で試しましたが、生成には最低でも 3 分くらいはかかる感じでした。 (サーバー側で生成してからブラウザーへダウンロードしているようなので、全体的な時間としてはネットワークの状況にも左右されます)

**図 1-2 ソースにした内容の一部**

![URL や参照元のブックマークが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1a2bf96a6d6b4547a4b7ea996ac05b51/generate-audio-summary-with-notebooklm-github-source.png?w=1021\&h=559\&auto=compress%2Cformat)

また、生成された音声の再生時間はソースの内容にもよりますが、7 〜 9 分くらいが多かったです。

生成時間はすこし長くてプログレスバー的なものは表示されないのですが、ページを閉じても処理は継続されたのでバックグラウンドで処理してもらうことも可能です。

なお、ソースの更新にあわせて音声を再生成したいような場合は、生成済みの音声データを一旦削除することになります。このとき、現行の音声データを保存しておきたい場合はダウンロードすることになります。

**図 1-3 ポップアップメニューの項目**

![生成された音声を再生するプレイヤーのメニュー、「再生速度を変更」「ダウンロード」「削除」項目が表示されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/50a388b4c66a4052939c677547a0adcc/generate-audio-summary-with-notebooklm-popupmenu.png?w=537\&h=306\&auto=compress%2Cformat)

また、モバイル版でも「ノートブックガイド」が表示されるようになったので、こちらからも同様の操作を行えます。

**図 1-4 モバイル版での表示**

![Android のブラウザーでノートブックを表示しているスクリーンショット、ノートブックガイドが PC 版のように表示されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/50ec78c7c7424b369e428c861c8f8e58/generate-audio-summary-with-notebooklm-mobile.png?w=1080\&h=2058\&auto=compress%2Cformat)

::: message

生成される音声は英語のみですが、日本語のソースも利用できました。また、以下のように会話することもあったので、日本語だと認識もしているようです。

*   A さん「日本語で書かれた内容を掘り下げてみようじゃないの」
*   B さん「ワォ」

:::

## 聞いてみる

生成が完了すると再生ボタンが表示されます。普通のプレイヤーなのでクリックすれば会話風の解説が再生されます。また、再生速度の変更もできるので、「もう少しゆっくりしていただければ」とか「英語得意だし倍速で時短したい」という場合も安心かと思います。

**図 2-1 再生速度変更**

![プレイヤーの再生速度変更メニューを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b6f6b4a4ab5c4f09b675cd91abb6da8f/generate-audio-summary-with-notebooklm-play.png?w=545\&h=504\&auto=compress%2Cformat)

なお、生成した音声はサーバー側に保持されているようなので、他のデバイスや共有しているノートブックからでも読み込んで再生できます。

**図 2-2 他デバイスからの読み込み**

![モバイル版のノートブックガイドで「読み込み」ボタンが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4520c29180f04dddbd923f5987c83e0b/generate-audio-summary-with-notebooklm-import.png?w=1080\&h=1119\&auto=compress%2Cformat)

では、実際にどのような音声が再生されるのか確認してみます。

と、言いたいところですか音声をアップロードしてもよいのかわからなかったので、代わりに冒頭部分の字幕のスクリーンショットを貼り付けます。 (ノートブックのプレイヤーに字幕機能はなかったので、Android の自動字幕起こしを利用しました)

**図 2-3 会話冒頭付近を自動字幕起こしで表示**

![Android で音声を再生し字幕(英語)を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1b0b4613358f4b2d826a42b77030d32e/generate-audio-summary-with-notebooklm-subtitle.png?w=1080\&h=1173\&auto=compress%2Cformat)

英語苦手なのでなんとなくですが「小粋な会話」から解説が始まっているような気がします。

## もう少し試してみる

せっかくなので、もう少し違った切り口のソースでも試してみます。

### 最近のニュース

以前、ニュースフィードを定期的にノートブックにする仕組みを作ったので、これを元に会話風の音声を生成してみます。

**図 3-1 最近のニュースの例**

![ニュースのタイトルと記事などが一覧になっている Google ドキュメントのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0c734e1eeca248b89af1bdc0aafd8ba8/generate-audio-summary-with-notebooklm-feed-source.png?w=689\&h=440\&auto=compress%2Cformat)

このような感じで Google ドキュメントのファイルとして直近 24 時間分(だいたい 300 記事くらい)のタイトルと概要が保存されています。これをソースとして音声を生成するとこんな感じになります。

**図 3-2 生成された音声の内容(「o1」はなぜか字幕起こしによって「01」にされます)**

![ソースに記載されていたニュースの概要を元に作成された音声を再生しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6df3055957af4caeb2a8e46d444782ae/generate-audio-summary-with-notebooklm-feed-play.png?w=1079\&h=1667\&auto=compress%2Cformat)

なんとなく「小粋な会話」をしている感じです。しかし、会話の流れが「AI のニュース」「ゲームのニュース」「AI のニュース」のように飛ぶこともあったのでもう少しまとめてくれるとうれしいところです。

この辺は、会話している二人の役割(性格)を「A さんはゲーム好き、B さんは自作 PC 好き」のように設定できると、注目する項目を誘導できそうかなと思いました。

あとは、ソースにないことを結構しゃべっていたのは少し気になりました。

**図 3-3 なぜか「Oshinoko」について語り始める**

![「Oshinoko」の内容などについて語っている字幕のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2c9154856ec14b1991e7983378c00fe3/generate-audio-summary-with-notebooklm-oshinoko-play.png?w=1080\&h=1335\&auto=compress%2Cformat)

他にも東京ゲームショウについて語ってからニュースについてしゃべることもありました。 (私はゲームをやらないので内容が正しいのかは不明)

おそらくはソースが「タイトルと概要だけの一覧」では話が膨らまないから「こうなる」のかなとは思いますが、ソースにない情報をしゃべっているので注意は必要そうです。

とは言っても、ヲタ全開でトークをするような音声を生成してもらうのも面白そうではあります。

### ウォーキングのときのメモ(ほぼ写真)

ウォーキングのとき、Notion のページから写真を撮って簡単なテキストを追加しているだけのものです。

**図 3-4 写真が 10 枚程度にテキストが若干のページ**

![Notion のページに河川敷の写真が保存されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/45ec67928922484e8a07c9e00ba280b0/generate-audio-summary-with-notebooklm-walk.png?w=864\&h=1920\&auto=compress%2Cformat)

それでも、6 分くらいの会話が生成されました。

**図 3-5 メモを元にした会話**

![写真の内容をもとに会話している字幕のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4da2af01af9f42fbab9adaffc5ba91b9/generate-audio-summary-with-notebooklm-walk-play.png?w=1080\&h=1056\&auto=compress%2Cformat)

再生してみると暑さの表現がちょっと誇張されている気もしますが、写真の内容につていも語っているようです。

NotebookLM では人物の画像を扱えないようなのでちょっと注意は必要ですが、使い方によっては画像から手軽に解説音声を作れそうかなと思いました。

### きのこたけのこ戦争

小粋なだけでなく熱く議論してもらおうかなと思ったのですが…。

@[card](https://ja.wikipedia.org/wiki/%E3%81%8D%E3%81%AE%E3%81%93%E3%81%9F%E3%81%91%E3%81%AE%E3%81%93%E6%88%A6%E4%BA%89)

思っていたよりも殺伐とはしなかったです(会話の内容をきちんと理解できていないので雰囲気で判断しています)。

なお、何かまずいことしゃべっていたらアレなのでスクリーンショットは載せないでおきます。

## おわりに

NotebookLM の新しい機能を利用して「会話風の解説音声(英語)」を生成してみました。

英語苦手なので正直言うと会話の内容をきちんとは把握できていません。それでも指定したお題(自作のドキュメントやウェブサイト)についての会話を手軽に生成できるのは面白く、いろいろなソースから会話を生成してしまいました。

また、ニュース一覧についての会話は「内容を理解できたら普通にポッドキャストの番組として楽しそうだな」とも思ったので、日本語対応がいつかそのうちされるといいなとも思いました。

あとは今回の内容とは少し離れますが、会話風の音声生成機能は他のサービスでも使われているようです。

@[card](https://gigazine.net/news/20240911-google-illuminate/)

Google フォトなどにも搭載されていったら、振り返り音声みたいなのを簡単に作れそうでおもしろそうだなとも思いました。
