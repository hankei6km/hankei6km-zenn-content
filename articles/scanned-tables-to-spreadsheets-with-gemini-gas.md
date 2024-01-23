---
id: scanned-tables-to-spreadsheets-with-gemini-gas
title: Gemini API を使いスマホでスキャンした表やグラフから自動的にスプレッドシートを作ってみる
emoji: 📷
type: tech
topics:
  - gemini
  - googleappsscript
  - googlespreadsheet
  - googledrive
published: true
---

[Gemini API](https://ai.google.dev/docs?hl=ja) を少し触ってみて、画像関連でいくつか応用できそうに思えました。

そこで、[Google Drive でスキャンした画像(PDF)](https://support.google.com/drive/answer/3145835?hl=ja\&co=GENIE.Platform%3DAndroid)から自動的にスプレッドシートを作成するような GAS 用スクリプトを試作してみました。

## どのようなスクリプト？

Google Drive で以下のような画像をアップロード(スキャン)しておくと Gemini API を利用してスプレッドシートを作成します。

**図 1-1 紙の表をスキャンしたファイル**

![motorola razr 40s のスペック表をスマホのカメラで撮影し Google Drive で表示しているスクリーンショット。](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/30256fa9dd394130af927e06405068b6/scanned-tables-to-spreadsheets-with-gemini-gas-paper.png?w=799\&h=533\&auto=compress%2Cformat)

**図 1-2 紙の表から作成したスプレッドシート**

![スキャンした画像(motorola razr 40s のスペック表)が変換されたスプレッドシートを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d55578371b3d46c89942dbc8ed86841c/scanned-tables-to-spreadsheets-with-gemini-gas-converted.png?w=997\&h=675\&auto=compress%2Cformat)

また、スキャン元の画像にはグラフも利用できます。ただし、グラフ内に値が表示されていないと多くの場合で正確な値にはなりません。

**図 1-3 画面に表示されているグラフをスキャンしたファイル**

![みかん、りんご、パイナップル、バナナの4種類の果物の数量を表す円グラフが描かれています。](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0ef410e411524405a36db88f8b625b46/scanned-tables-to-spreadsheets-with-gemini-gas-chart.png?w=784\&h=495\&auto=compress%2Cformat)

**図 1-4 グラフから作成したスプレッドシート**

![グラフから変換されたスプレッドシートを表示しているスクリーンショット。](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/da8e0ccd28fb4185829835d8e4ba53d3/scanned-tables-to-spreadsheets-with-gemini-gas-chart-sheet.png?w=560\&h=321\&auto=compress%2Cformat)

## 表として取り込む仕組み(プロンプト)

基本的には[ドキュメントに従って画像を含むマルチモーダル用クエリーを POST](https://ai.google.dev/tutorials/rest_quickstart#text-and-image_input) しているだけですが、テキストプロンプトを以下のようにしています。

**リスト 2-1 画像の種類別に情報を取りだすプロンプト**

    画像の内容について。

    以下の中から最も適切な方法で説明してください。

    ## 表が含まれている画像の場会

    返答の最初に「## 表が含まれている画像」と記述したあと、表の内容をマークダウン形式で記述してください。

    ## グラフが含まれている画像の場合

    返答の最初に「## グラフが含まれている画像」と記述したあと、グラフの内容をマークダウンの表として記述してください。

    ## テキスト(文章または簡単な説明文など)が含まれている画像の場合

    返答の最初に「## テキストが含まれている画像」と記述したあと、テキストの内容を記述してください。

    ## その他の画像の場合

    返答の最初に「## その他の画像」と記述したあと、画像の説明を記述してください。

このようにしておくと、返答の 1 行目を調べることで画像が表へ変換されているか判定できます。また、表は Markdown のテーブル風に返答されます。これによりスプレッドシートへの変換処理などを追加しやすくなります。

::: message

表は CSV 形式で受け取りたかったのですが、私のプロンプト力ではうまくいきませんでした。

今回はとりあえず、GAS 側のコードで「区切りが `|` である CSV」という扱いでパースしています(よって、ヘッダー行の扱いなどが少しおかしい)。

:::

## Gemini API を GAS で利用

Google Spreadsheets と Drive 関連の処理なので今回は GAS でスクリプトを作成します。

しかし、GAS には Gemini サービスなどはないので、次の記事を参考に REST API を利用します(API キーの取得についても同記事が参考になります)。

https://qiita.com/gas-suke/items/3ce261be244f9efd54b2

## 実装してみる

### スキャン用のスクリプト(ライブラリー)作成

ライブラリーとして再利用しやすくするために、以下のようなスタンドアローンスクリプトを作成します。

**リスト 4-1 Gemini で画像をスキャンするスクリプト(****`ScanImageWithGemini`**** )**

@[gist](https://gist.github.com/hankei6km/ac0324db986be1a6d16000f4e6f8e93a?file=scan-image-with-gemini.gs)

処理的に難しいことはしていませんが、送信サイズを小さくするためにファイルのサムネイルを利用しています。また、サムネイルを利用することで通常の画像以外のファイルも利用が容易になっています(画像以外のファイルについては後で少し触れます)。

::: message

マルチモーダルで画像を扱う場合、以下のような方針が提示されています。

[プロンプト設計の基礎](https://ai.google.dev/docs/multimodal_concepts?hl=ja#prompt-design-fundamentals)

> *   **単一画像のプロンプトでは画像を最初に配置する:** Gemini では画像とテキストの入力順序は問いませんが、1 つの画像を含むプロンプトでは、その画像をテキストプロンプトの前に配置したほうがパフォーマンスが向上する可能性があります。

この順番が何を指すのか不明だったのですが、Payload 内の `contents.parts` 配列を指していると推測し、[API ドキュメントのサンプル](https://ai.google.dev/tutorials/rest_quickstart#text-and-image_input)からは配列の順番を変更しています。 (サンプルはテキストが先だった)

:::

### フォルダー内のファイルをスキャンする

ライブラリーを作成できたので、スフォルダーに保存されている画像から自動的にスプレッドシートを作成してみます。

前準備として、Google Drive に以下のフォルダーを作成し ID を控えておきます。

*   スキャン(アップロード)により画像が保存されるフォルダー
*   スプレッドシートを作成するフォルダー

**図 4-1 スキャン用とシート作成用フォルダー**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4ba0752300cc4697a07b7044dac14712/scanned-tables-to-spreadsheets-with-gemini-gas-folders.png?auto=compress%2Cformat)

以下のようなスタンドアローンスクリプトを作成し、上記ライブラリーを追加します。

**リスト 4-2 フォルダー内のファイルを処理するスクリプト**

@[gist](https://gist.github.com/hankei6km/ac0324db986be1a6d16000f4e6f8e93a?file=scan-folder-with-gemini.gs)

スクリプトを作成したらスクリプトプロパティーを設定します。

*   `GEMINI_API_KEY`: API キー
*   `SRC_FOLDER`: スキャンにより画像が保存されるフォルダーの ID
*   `DEST_FILDER`: スプレッドシートを作成するフォルダーの ID

これで準備ができました。

スキャン用フォルダーに画像を保存し、`start` 関数を実行するとスプレッドシートが作成されます。また、元ファイルの説明欄にも表(マークダウン形式のテキスト)が追加されます。

**図 4-2 スキャンされた画像**

![Google Drive のスキャン用フォルダーを表示しているスクリーンショット。ファイルが 2 つ保存されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/119a95e2563e4e84973bed1bc3687ca7/scanned-tables-to-spreadsheets-with-gemini-gas-folder-src.png?w=510\&h=305\&auto=compress%2Cformat)

**図 4-3 作成されたスプレッドシート**

![Google Drive のシート作成用フォルダーを表示しているスクリーンショット。スプレッドシートが 2 つ保存されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ba8f511d88b14e0bb0ccc241d4583186/scanned-tables-to-spreadsheets-with-gemini-gas-folder-dest.png?w=510\&h=305\&auto=compress%2Cformat)

ファイルの内容が表でない場合、スプレッドシートは作成されませんが、ファイルの説明は追加されます。

**図 4-4 表でない場合**

![Pixel Buds の充電器が写っている画像を Google Drive で表示しているスクリーンショット。説明欄には適切な説明が付与されている。](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/372bd8decbb34c0abcbcabc5dc0efa36/scanned-tables-to-spreadsheets-with-gemini-gas-description.png?w=793\&h=536\&auto=compress%2Cformat)

`start` 関数を複数回実行した場合、ファイルの説明が空白でないものはスキップされるので、定期実行しておくと自動化ぽい感じになるかと思います。

また、フォルダーとスクリプトを複数用意しておき、それぞれのプロンプトを変更しておくと用途別の結果を得やすくなります。

私は[以前に作成した OCR ライブラリー](https://github.com/hankei6km/gas-gocr2notion)も併用しているので、各フォルダーをスマホのホーム画面へ登録することで使い分けています。

**図 4-5 スキャン用フォルダーをホーム画面へ登録**

![ホーム画面に登録された Google Drive のフォルダーアイコンのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/531bd4c51a9c432a9d11fd4cc745f22f/scanned-tables-to-spreadsheets-with-gemini-gas-home-icon.png?w=482\&h=670\&auto=compress%2Cformat)

::: message

少し手間ですが、[Google Drive の変更通知を GAS で受信している場合](https://zenn.dev/hankei6km/articles/receive-google-drive-chages-notifications-by-gas)、`Drive.Changes.list()` の情報を `scan` 関数に渡すとスキャン後に処理を実行できます。

また、Notion を利用している場合、下記のライブラリーも利用するとスプレッドシートのリンクなどを Notion へ送信できます。

@[card](https://github.com/hankei6km/gas-gchanges2notion)

Notion 側のページに実際の表は挿入されませんが、リンクをタップするとスプレッドシートが開くのでコピペはしやすくなるかと思います。

![送信された変更ファイルの一覧をモバイル版の Notion で表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/46c49e706a89493f8066a2d68ea40374/scanned-tables-to-spreadsheets-with-gemini-gas-notion.png?w=864\&h=880\&auto=compress%2Cformat)

![モバイル版のスプレッドシートで表を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f4a312cfd9e7413eb216d8f683a3cf85/scanned-tables-to-spreadsheets-with-gemini-gas-mobile-sheet.png?w=864\&h=580\&auto=compress%2Cformat)

:::

## 応用

今回のスクリプトは Google Drive のサムネイルを利用しているので、Gemini が対応していないようなファイルでもサムネイル部分を入力できます[^movie]。本筋から外れますが、スプレッドシートを UI 代わりにして少し実験してみます。

[^movie]: 今回のスクリプトでは動画のサムネイルを入力できますが、動画そのものは入力できません。動画は「[GoogleのマルチモーダルAI「Gemini Pro Vision」は、動画についてどこまで正しく答えられるか？【イニシャルB】 - INTERNET Watch](https://internet.watch.impress.co.jp/docs/column/shimizu/1561167.html)」などが参考になります。

### 簡単な UI を作成(スプレッドシートにメニューを追加)

スプレッドシートで「拡張機能 / Apps Script」メニューからスリプトを作成します。

**リスト 5-1 メニューを追加するスクリプト**

@[gist](https://gist.github.com/hankei6km/ac0324db986be1a6d16000f4e6f8e93a?file=sheet-menu.gs)

スクリプトに以下を設定します。

*   前述の `ScanImageWithGemini` ライブラリーを追加
*   スクリプトプロパティーへ `GEMINI_API_KEY` を追加

これで、スプレッドシートに「スキャン / プロンプト指定でスキャン」メニューが追加されます。

セルに「ファイルの URL」と「プロンプト」を入力し、メニューから「プロンプト指定でスキャン」を選択すると返答がセル内に転記されます。 (ファイルの URL は共有メニューからコピーしたものが使えます)

**図 5-1 ファイルURL とプロンプトを入力しメニューを選択**

![スプレッドシートで「スキャン」メニューを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4ef0c0bf522843a28b0f3a57d8bb7cd9/scanned-tables-to-spreadsheets-with-gemini-gas-menu.png?w=943\&h=275\&auto=compress%2Cformat)

### diagrams.net(draw\.io)ファイルを扱う

現行の Bard では少し面倒な処理として [diagrams.net](https://www.diagrams.net/)(draw\.io)ファイルから [Tailwind CSS](https://tailwindcss.com/) + HTML のソースコードを作成してみます。

**図 5-2 diagrams.net でカード的な図を作成**

![diagrams.net でプロフィールカードのような図を作成しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e9d71b97797540f5ad61fe29fa571a55/scanned-tables-to-spreadsheets-with-gemini-gas-drawio-drawing.png?w=886\&h=586\&auto=compress%2Cformat)

**図 5-3 ****diagrams.net 用ファイルは****サムネイルを取得できる**

![Google Drive 上で上記ファイルのサムネイルが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/05748135c8f5480687d4c650a81e77f4/scanned-tables-to-spreadsheets-with-gemini-gas-drawio-thumb.png?w=728\&h=286\&auto=compress%2Cformat)

**リスト 5-2 ソースコードを作成するプロンプト**

    画像内のカードから Tailwind を使った HTML ソースコードを作成してください。

**図 5-4 プロンプトを指定して HTML ソースを取得**

![プロンプトを指定しスキャンメニューを実行したスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6bb7fc402e03468496be84f5f93600fd/scanned-tables-to-spreadsheets-with-gemini-gas-drawio-scan.png?w=980\&h=511\&auto=compress%2Cformat)

**図 5-5 Tailwind Play へコピペしてプレビュー**

![Tailwind Play でプレビューを表示しているスクリーンショット。元の図をある程度再現できているが見出しの扱いが少しおかしい。](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/327ca7358d9c4c3ea9bbbd9be1d74ecb/scanned-tables-to-spreadsheets-with-gemini-gas-drawio-preview.png?w=1142\&h=566\&auto=compress%2Cformat)

見出しの配置が少し変化していますが、こちらの方が良い感じに見えるので深く考えないことにします。

ここでは、diagrams.net 用ファイルを使いましたが、いろいろなファイルとプロンプトを試して良い感じなったら自動化してみると面白いかと思います。

## 考慮点

### プロンプトと通常ソースコードの構成

今回は画像の種類をプロンプトで特定しておき、通常のソースコード(GAS)で処理を分岐させる方針としました。

このとき、プロンプトで指定したキーワードを深く考えずに GAS 側で使ってしまったので、扱う画像の種類を増やす時に少し苦労しました。 (実はグラフ関連の処理を後から追加したので「うーん」となりました)

プロンプトもベタ書きするのではなく、配列とテンプレートから組み立てたりといったことを検討した方が良いのかもしれません。

次に作るときはこの辺に気をつけたいなと考えています。

### パラメーター(`generationConfig`)

いくつか文章を取り込んでみた感じでは、前後の文脈からテキストを補完しているような印象です。気が利いていると感じる場合もあるのですが[^roll-chan]、OCR としてみると少し困った状態でもあります。

パラメーターの変更でランダムさを減らしてみたりはしたのですが、この辺は [Google AI Studio](https://ai.google.dev/tutorials/ai-studio_quickstart?hl=ja) も使って調整した方が良かったかなと感じました。

https://ai.google.dev/docs/prompt_best_practices?hl=ja#experiment-with-different-parameter-values

[^roll-chan]: レシートに記載されている「ロールチャン」を「ロールちゃん」にするなどがありました。レシートを見返したときに「ロールチャンとは？」と考え込んでしまったのでありがたいとは思ったのですが、悩ましいところでもあります。

## おわりに

Gemini API と GAS を利用して画像をスプレッドシートへ変換するツールを試作してみました。

作ってみたものを利用してみた感触としては「OCR として見ると気を利かせすぎかも」な感じもありますが、ちょっとした表の取り込みは楽になりそうかなと考えています。

また、今回はフォーマットの変換などを主に試しましたが、画像の内容をトリガーとした自動化などにも応用できそうだと感じました。
