---
id: gdocs-za2tab
title: ざつタブ
emoji: 📑
type: tech
topics:
  - googledocs
  - gemini
  - notebooklm
  - googleappsscript
  - zennfes2025free
published: true
---

Google の AI サービス(Gemini チャットアプリの Gem や NotebookLM など)でファイルを利用するとき、Google ドキュメント形式で追加するといくつかの利点があります。

*   更新されたドキュメントファイルを同期できる

    *   Gem は自動で同期される
    *   NotebookLM は手動で同期させる

*   ドキュメント内の画像を考慮してくれる(例外はある)

また、Google ドキュメント形式で追加した場合は、ドキュメント内のタブ構造も考慮されるので「スクリプトでドキュメント内の特定箇所(タブ)だけを更新しやすい」というありがたい状況になります。

そこで、Google の AI サービス用にドキュメントファイルのタブへ「ざつに情報を詰め込むスクリプト」を試作してみました。

## 基本の形

今回試作したスクリプトは「タブにリンクを記述しておくと、リンクの内容をタブへ書き込む」「ただし書き込まれた内容は人間が編集することを考慮していない」といった感じになります。

利用する方法としてはいくつかありますが、Google ドキュメントのファイルにスクリプトを追加して、上記ファイルを貼り付けるのが楽かと思います。

**リスト 1-1 スクリプト**

https://github.com/hankei6km/test-gas-gdocs-za2tab/blob/main/code.js

貼り付けたら、サービスに Drive API v3 を追加し`myFunction` をリストのようにします。

**図 1-1 スクリプトエディターで Drive API v3 を追加**

![スクリプトエディターで「サービス」の下に「Drive」が追加されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/06f29cde45044d5eaf9c8456d3e40eac/gdocs-za2tab-basic-drive-api.png?w=335\&h=303\&auto=compress%2Cformat)

**リスト 1-2 ****`myFunction`**** の例**

```js
function myFunction() {
  const doc = DocumentApp.getActiveDocument()
  try {
    const now = new Date()
    refreshTabsByLinks(doc, now)
  } finally {
    doc.saveAndClose()
  }
}
```

その後、以下のように取り込みたい対象(RSS フィードなど)をリスト形式で指定します。

**図 1-2 リンクをリスト形式で指定**

![Google ドキュメントの編集画面で、リンクをリスト形式で記述しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/76608e7ae10a4b4f88f083a945b04c3e/gdocs-za2tab-basic-list-item.png?w=1148\&h=688\&auto=compress%2Cformat)

::: message

Google ドキュメントでリンクを記述する方法としては[スマートチップ](https://support.google.com/docs/answer/10710316?hl=ja)もありますが、スクリプトから扱う方法が不明でったので残念ながら対応していません。

:::

`myFunction` を実行すると、リストの下に各 URL から受信したテキストが配置されます。

**図 1-3 スクリプトで更新された状態**

![リンクの下に取得したフィードのソースコードが挿入されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/513897fecc2e41a78272944750115fc1/gdocs-za2tab-basic-updated.png?w=1311\&h=717\&auto=compress%2Cformat)

試しにドキュメントファイルを Notebook LM へ追加してみます。「設定のリストは残ったまま」「フィードのソースをベタ貼り」の状態ですが、わりと普通に扱ってくれます。

**図 1-4 NotebookLMへソースとして追加しチャット**

![NotebookLMでチャットしているスクリーンショット。質問に対してフィード内の記事が返答されている。](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5949d6e642b64e31a07a4c9b3510315e/gdocs-za2tab-basic-notebooklm.png?w=1311\&h=717\&auto=compress%2Cformat)

あとは、スケジュール実行などで `myFunction` を指定しておくと、定期的にタブの内容が更新されます。

なお、スクリプトの更新対象にならないタブはそのまま残ります。これを利用し、以下のようにカバーページなど作成しておくと、ドキュメントファイルの概要(作成された目的など)を AI サービスへ伝えやすくなるかと思います。

**図 1-5 カバーページ用のタブに概要を記述**

![「カパーページ」タブの中にドキュメントの概要が記述されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f0b6a428b3cc44409a6a5c0d751d1781/gdocs-za2tab-basic-cover.png?w=1320\&h=387\&auto=compress%2Cformat)

この後は、リストに指定できる対象などについて説明していきます。

::: message

上記の AI サービスによる利用例では NotebookLM を使っていますが、Gemini アプリ(Gem)でもわりといけます。ただし、ドキュメントファイルが大きくなると、2.5 flash では内容を扱えないことが増えました。

:::

::: message

RSS フィードや HTML などをそのままソースに使う場合、ソースの参照箇所を表示すると意味がない(人間では理解しにくい)状態になってしまうという難点はあります。

:::

::: message

大きなデータを頻繁に転記すると以下のエラーが発生し、それ以降 `DocumetApp` から扱えなくなることが何度かありました。こうなると、そのドキュメントファイルの内容を手動で削除などしても `DocumentApp` から操作できなくなってしまいます。 (通常の UI からも手動で操作できなくなることもありました)

    Exception: Service unavailable: Documents

対応としては「あきらめてコピーを作ってそちらを使う」しかなさそうです。

**AI による概要より**

> *   **Corrupted documents**: In rare cases, a specific Google Doc might become corrupted, leading to this error.
>     *   **Solution**: Try creating a new document and copying the content over, if feasible.

:::

## 各種 URL

上記の RSS フィード以外に、ウェブページ(HTML)や JSON を指定した場合でも、NotebookLM ではわりと普通に扱ってくれます。

私は以下のようなドキュメントを作成し、NotebookLM の音声概要でニュース番組風ポッドキャストを作ることに利用しています。

**図 2-1 各種情報を取得するドキュメント**

![「ニュース」「天気予報」「路線情報」を作成してあるドキュメントファイルのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/11d84c5d5a654c19bdb89bfc2a191586/gdocs-za2tab-url-news.png?w=1363\&h=534\&auto=compress%2Cformat)

タブの構成は以下のようになっています。

*   ニュース: 以前に作成した「[Feed リーダーの直近の更新を Google ドキュメント化したもの](https://zenn.dev/hankei6km/articles/generate-audio-summary-with-notebooklm#%E6%9C%80%E8%BF%91%E3%81%AE%E3%83%8B%E3%83%A5%E3%83%BC%E3%82%B9)」を取り込む(Google ドキュメント取り込みは後述します)

*   天気予報: 気象庁から以下の JSON ファイルを取り込む

    *   <https://www.jma.go.jp/bosai/forecast/data/forecast/130000.json>
    *   <https://www.jma.go.jp/bosai/forecast/data/overview_forecast/130000.json>

*   路線情報: 鉄道会社の路線情報などのページを取り込む

調子よいときは天気情報を話しているときに、「暑さが続く中で、自動販売機の設定温度を 2 度下げるという情報もありますね」のようにニュース番組風になります(ならないことも多いです)。なお、路線情報は鉄道会社の情報ページをそのままだからか(サイズが大きくなりがち)、音声概要だと「概ね平常なようです」みたいにされてしまいます。

**図 2-2 チャットで路線について確認(この状態でも音声概要は「平常」となる)**

![路線の状況を確認すると、遅延している路線を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b77e2b32b15b4adcb9853f6d7794675a/gdocs-za2tab-url-rail.png?w=906\&h=540\&auto=compress%2Cformat)

また、ソースとして各記事の概要しか渡していないと NotebookLM は話を盛ってきます。

音声概要でのニュース要約は以下のような利点もあり便利に使っていますが、だらだら聞きながら「なんか[抹茶の価格が高騰してるらしいよ](https://technews.tw/2025/07/05/japans-heat-stressed-matcha-tea-output-struggles-to-meet-soaring-global-demand/)」みたいな話のネタを仕入れるくらいのノリがよいのかなと思っています。 (念の為に書いておくと、この状況は「ソースにニュース本文が含まれない」ことが原因だと思われるので、NotebookLM の要約が悪いというわけではないとは思います)

*   日本語以外のニュースフィードを混在させても日本語で要約してくれる
*   移動中などでも利用しやすい

::: message

音声概要はどれくらい話を盛ってくるか。

たとえば、何等かのアンケート集計のニュースの場合「記事本文を参照できないので集計結果を確認できないはず」なのですが、それっぽい結果を作り出してくるのはよくあります。

他には、CD2WAV32 が約 20 年ぶりに更新されたニュースについて、音声概要では更新された理由を「サブスク全盛の現代において、物理メディアを扱うことの意味を〜」のように説明することもありました。 (実際は、以下の記事全文にあるようにまったく別の理由です)

@[card](https://forest.watch.impress.co.jp/docs/news/2019313.html)

最近(2025/08)、日本語向け音声概要でも制約が緩和されたようなので、また違った印象になるかもしれませんが、なんとなく「昔ながらの良さ」「点と点を結ぶ」的な良い話風になる傾向かなと感じました。

:::

## Google ドキュメント

Google ドキュメントをブラウザーで開いているときの URL を指定することで、他の Google ドキュメントの内容を転記するようになっています。ただし、制限は結構あります。

*   書式はほとんどコピーされません

*   以下のいずれかのタブが対象となる

    *   URL にタブが含まれる: 含まれているタブ
    *   URL にタブが含まれない: アクティブタブ(通常は最初のタブ)

書式をできるだけ保持する方法としては、Markdown 形式でエクスポートしたものを貼り付ける指定もあります。こちらの場合は、Markdown で表現できる書式もソースとして取り込まれます。ただし、画像についてはデータ URL の参照リンクとして表現されるため「テキストが大きくなりがち」「AI サービスが画像として認識できない」状態となります。

以下は Markdown 形式で指定する URL の書式です。

**リスト 3-1 ドキュメントの ID だけで指定する場合(アクティブタブが対象となります)**

    https://docs.google.com/feeds/download/documents/export/Export?exportFormat=markdown&id={DOCUMNET_ID}

**リスト 3-2 タブを指定する場合**

    https://docs.google.com/feeds/download/documents/export/Export?exportFormat=markdown&id={DOCUMNET_ID}&tab={TAB_ID}

Google ドキュメントを転記する場合のスクリプトは以下のようになります。 (Google ドキュメントとして転記、Markdown 形式でエクスポートしたものを貼り付けのどちらにも対応します)

**リスト 3-3 サンプルコード**

```js
function myFunction() {
  const doc = DocumentApp.getActiveDocument()
  try {
    const now = new Date()
    refreshTabsByDocuments(doc, now)    
  } finally {
    doc.saveAndClose()
  }
}
```

**図 3-1 通常の転記と Markdown での貼り付け**

![通常の転記で取り込まれたドキュメントとマークダウンで転記されたドキュメントが表示されているスクリーンショット、マークダウンの方はソースとして書式が表現されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d49153b1e5194ae88362ffb69e570183/gdocs-za2tab-gdocs-sample.png?w=1488\&h=1025\&auto=compress%2Cformat)

「Google ドキュメントとして転記」「Markdown 形式で貼り付け」のどちらを使うかは悩むところですが、画像無しでテキストが多い場合は Markdown 形式の方が処理は軽くなるようです。 (Markdown のテキストを 1 つのパラグラフとして挿入しているためだと思われる)

::: message

AI サービスでは Google ドキュメントの書式をすべては認識しないようです。

試した限りでは、文字の色設定などは無視されるようでした。おそらくですが Markdown(HTML を許容しない状態)で表現できる書式に対応しているのかなと思いました。

:::

## Google スプレッドシート

Google ドキュメントと同じように Google スプレッドシートの URL を指定した場合です。シートの各セルをテキストにしたものを転記するだけですが、Google フォームや AppSheet で更新されるシートなどには使えるかなと思います。

**リスト 4-1 サンプルコード**

```js
function myFunction() {
  const doc = DocumentApp.getActiveDocument()
  try {
    const now = new Date()
    refreshTabsBySpreadsheets(doc, now)
  } finally {
    doc.saveAndClose()
  }
}
```

**図 4-1 テキストの表として転記される**

![Google ドキュメントのタブの中に表(テーブル)が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/11c4a6e2899147fc872cc7218cda9502/gdocs-za2tab-sheets-sample.png?w=1363\&h=632\&auto=compress%2Cformat)

Gemini チャットアプリでは、このようなことをしなくても Google スプレッドシートを直接追加できます。 (試してみたところ、2025/08 時点では Google AI プランを利用していなくても追加はできました)

しかし、残念なことに NotebookLM ではいまのところ(少なくとも無課金ユーザーは)対応していないようなので、Google ドキュメントへ転記するとある程度は対応できるようになります。また、転記するときに Google フォームと AppSheet 形式の画像をとりあえず取り込むようにしているので、一部制約はありますが画像を含むシートでも対応できます。

**図 4-2 Google フォームから保存しているスプレッドシート**

![セルの中にドライブ上の画像ファイルへのリンクが挿入されているスプレッドシートのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e87941939e804e649ac6252f917bda37/gdocs-za2tab-sheets-forms-sheet.png?w=1338\&h=814\&auto=compress%2Cformat)

**図 4-3 転記するとき画像も取り込まれる**

![Google ドキュメントへ転記された表では画像が埋め込まれているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/32e2f074cde844ea8a74308d62731e06/gdocs-za2tab-sheets-forms-gdocs-table.png?w=1338\&h=814\&auto=compress%2Cformat)

::: message

画像の扱いについて、試した限りでは以下のようになりました。

*   人物が含まれる画像は NotebookLM ではソースとして(表とは関係なしに)常に取り込まれない
*   ドキュメントの表内の画像は Gemini チャットアプリ(Gem)では認識されない(NotebookLM では人物が含まれていなければソースとして取り込まれる)

:::

なお、Gemini チャットアプリの表内画像を認識しない点については、部分的に回避する方法も後述しています。

## フォルダーのファイルをタブへマッピング(ドキュメント、スプレッドシート、画像)

「最新」タブにフォルダーのリンクを記述しておくと、フォルダー内のファイルの内容を後続の「n 個前」タブへ転記します。 (対象にするタブ名はスクリプトで指定できます)

**図 5-1 サンプルコード**

```js
function myFunction() {
  const doc = DocumentApp.getActiveDocument()
  try {
    const now = new Date()
    refreshTabsByFolderFiles(doc, now)
  } finally {
    doc.saveAndClose()
  }
}
```

**図 5-2 マッピング用タブのサンプル**

![「最新」タブと「1 個前」から「4 個前」タブが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/31ec4b46510e4bdbbed23d326b884ef3/gdocs-za2tab-folder-sample.png?w=1134\&h=684\&auto=compress%2Cformat)

いまのところ、以下のフォーマットに対応しています。

*   Google ドキュメント - アクティブタブ(通常は最初のタブ)をドキュメントとして転記
*   Google スプレッドシート - アクティブシート(通常は最初のシート)の表をテキストとして転記
*   画像 - 画像の内容を転記
*   PDF - ファイルのサムネイル画像をコピー

リンクテキストの中に[フィルター](https://developers.google.com/workspace/drive/api/guides/search-files?hl=ja)や[ソート](https://developers.google.com/workspace/drive/api/reference/rest/v3/files/list?hl=ja)も指定できるようにしたので、特定の用途向けに作成した直近のメモを 1 ドキュメントへまとめるために利用しています。

**リスト 5-1 リンクテキストの書式**

     ラベル : フィルター : ソート

**リスト 5-2 URL リンクとファイルマッピングを行うコード**

```js
function myFunction() {
  const doc = DocumentApp.getActiveDocument()
  try {
    const now = new Date()
    refreshTabsByLinks(doc, now)
    refreshTabsByFolderFiles(doc, now)
  } finally {
    doc.saveAndClose()
  }
}
```

以下は天気予報とウォーキング(時々ジョギング)のメモをまとめた例です。このファイルを追加した Gem を作成しておき、ウォーキング前に「この後ウォーキングする予定です、注意点は？」みたいな確認に使っています。

**図 5-3 ウォーキングメモ用を作成日順に転記**

![「最新」タブにフォルダー指定のリンクが記述され、その下に転記されたメモファイルが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5fb1b3261c62419e9ac211e057d15fdb/gdocs-za2tab-folder-walking.png?w=1363\&h=826\&auto=compress%2Cformat)

上記メモはフリーフォマットで「暑い」「腰に来た」とかをざつに入力しているだけですが、知識ある人が書式をきちんと作れば「ワークアウト用の有益な Gem を作れるのかな」という気はしています。

::: message

ウォーキングのメモは、Notion のモバイルアプリからウォーキングの合間に適当に入力しておき、[スクリプトでGoogle ドライブの特定のフォルダーへエクスポートしたもの](https://zenn.dev/hankei6km/articles/notion-databases-to-google-docs-for-gemini#%E5%A4%96%E9%83%A8%E3%81%AE%E6%83%85%E5%A0%B1%E3%82%92-google-%E3%83%89%E3%82%AD%E3%83%A5%E3%83%A1%E3%83%B3%E3%83%88%E3%81%AE%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AB%E3%81%99%E3%82%8B\(notion-%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E3%81%AE%E5%A0%B4%E5%90%88\))を使っています。

(メモの入力も、ボイスメモなどで「だらだらしゃべったものを AI がいい感じのログにしてくれる」みたいにしたいなと思っています)

:::

::: message

AI Pro などで課金している Gemini ではスケジュール実行の機能が追加されたそうです。

スケジュール実行時に Google ドキュメントを知識として追加できるのなら、定期的にアドバイスを受けるようなワークフローを組めるような気もしています。課金してないので憶測ですが。

:::

## スプレッドシートの行をタブへマッピング

上記のファイルと同じような感じで、スプレッドシートの行をタブへマッピングします。マッピングする順序は最後尾の行を最新として、上の行を n 個前として扱います。フォームや AppSheet などで最新の数レコードだけを扱うような場合を想定しています。

**図 6-1 サンプルコード**

```js
function myFunction() {
  const doc = DocumentApp.getActiveDocument()
  try {
    const now = new Date()
    refreshTabsBySheetRows_(doc, now)
  } finally {
    doc.saveAndClose()
  }
}
```

**図 6-2 行が 1 つのタブへ転記される(列は見出し 2 として分割される)**

![タブの中で行が表現されているスクリーンショット。Google フォームや AppSheet の画像が取り込まれている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/75d7a69a07034588be8a684dd48a8bcf/gdocs-za2tab-rows-sample1.png?w=1409\&h=764\&auto=compress%2Cformat)

**図 6-3 1 つ上の行**

![「1 個前」タブでも行が表現されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/32f376db9909422197b0e929ebfe72d6/gdocs-za2tab-rows-sample2.png?w=1409\&h=764\&auto=compress%2Cformat)

この方法の場合、表を使わずに行を表現しているので、表内の画像を Gemini が認識しないという制限を回避できます。

## ローテーション

「最新」タブ以降の内容を 1 つ後ろのタブに転記させる方法です。更新履歴を扱いたいときを想定しています。 (対象にするタブ名はスクリプトで指定できます)

**図 7-1 サンプルコード**

```js
function myFunction() {
  const doc = DocumentApp.getActiveDocument()
  try {
    const now = new Date()
    rotateTabs(doc)
    refreshFirstTabByLinks(doc, now)
  } finally {
    doc.saveAndClose()
  }
}
```

**図 7-2 「最新」タブで URL リンクの内容を取得しておく**

![「最新」タブにフィードのソースが保存されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4fdf497a9cdb40a4b322678d765ed0b7/gdocs-za2tab-rotate-sample1.png?w=1302\&h=650\&auto=compress%2Cformat)

**図 7-3 スクリプトを実行すると「1 個前」へ転記される**

![「1  個前」タブに「最新」タブの内容が転記されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/87d1cf4e1043430abde682f3db7477f5/gdocs-za2tab-rotate-sample2.png?w=1302\&h=650\&auto=compress%2Cformat)

::: message

ローテーションについてはタブを転記するのではなく、ファイル単位で転記するという方法も考えられます。 (名前変更によってファイルをズラす方法では同一ファイルと認識されなくなるので、ファイル単位でも内容を転記する必要はあります)

この場合、1 ファイルのサイズを減らせるので多くの対象から情報を保存できるようになります。ただし、NotebookLM で利用する場合は、各ファイルを個別に手動で同期させる必要があるので少し面倒です。

:::

## 考慮点

### 先頭の方の記述が優先される(ように思える)

NotebookLM での音声概要や要約のときに個人的に感じているだけのことですが、先頭(1 行目)に近い記述が優先されるように思えます。たとえば、複数の RSS フィードを 1 つのタブにまとめたとき、音声概要を作成すると先頭に近いフィードについての説明が増えてきます(ように思える)。

カバーページなどを設けて「どのようにドキュメントを扱ってほしいか」を記述するとある程度回避できましたましたが、扱ってほしい優先度などから保存する位置を考慮した方がよいのかなとは思いました。

### 人間用に情報を保存しないデメリット

今回は更新されたタブの内容は「人間が編集(参照)することを考慮しない」方針にしています。これにより、従来のような「受信した HTML などをパースして必要な部分を抜き出して整形する」といった地味に手間のかかる処理が必要なくなります。

しかし、以下のような問題も出てきます。

*   AI サービスへ送信するテキスト量が増える傾向になる
*   AI サービスからの返答の正確性に影響する
*   AI サービスが参照元を提示してきたとき確認が難しい

HTML については、[`hast-util-select`](https://github.com/syntax-tree/hast-util-select) などを使ってセレクターで必要な部分を抜き出せるようにしてもよかったかなと少し後悔しています。

## おわりに

Google ドキュメントのタブへ各種情報を動的に集約させ、Google の AI サービスから利用してみました。

軽いノリで作ったスクリプトなので作りはいつも以上にグダグダですが、ウェブや Google ドライブ上に分散している情報を 1 つにまとめやすくなったかなと思っています。

たとえば、上の方に挙げた「天気予報とウォーキングメモを組み合わせたドキュメントファイル」から Gem を簡単に作れるようになったのは結構重宝しています。
