---
id: ogura-shuffle-api
title: 小倉百人一首かるたをランダムに取得する API を オープンデータ + Google Apps Script + SSSAPI で作った
emoji: 🎎
type: idea
topics:
  - api
  - gas
  - sssapi
published: true
---

[GitHub の Profile ページを自動更新](https://zenn.dev/hankei6km/articles/automatically-update-github-profile)させるときに作った「[小倉百人一首かるたデータ]から 1 レコード取得する」API についてです。

## API の概要

今回は「とりあえず動作する単機能な API」という方針で以下のような API になっています。

1.  サーバー側では一定間隔でランダムに 1 レコード選択しておく
2.  API を実行すると上記レコードを返却する

## データ

以下のデータを利用させていただきました。

*   Title: 小倉百人一首かるたデータ
*   Author: [Nanako Takahashi](http://linkdata.org/user/tnanako)
*   Source: <http://linkdata.org/work/rdf1s6834i>
*   License: <http://creativecommons.org/licenses/by/3.0/deed.ja>

データの形式としては LOD(Linked Open Data)なのですが、今回は時間をかけない方針だったので EXCEL ファイルとしてダウンロードしました[^lod]。

[^lod]: 本来なら各種データセットを組み合わせたり、さらに便利な使い方ができるようです。LOD については <https://www.nic.ad.jp/ja/materials/iw/2014/proceedings/s15/s15-takeda.pdf> など。

## 全件受信は避ける

話は少し前後しますが、上記データは RDF や JSON などで取得できるので、以下のような手順でもランダムな取得はできます。

1.  JSON で取得する
2.  ランダムにレコードを選択する

しかし、この方法だと毎回全件受信することになります。また、受信側にレコードを選択するためのロジックを持つ必要があります。

できれば受信側は API からデータを受信するだけにしたいので以下のようにしました。

## スプレッドシートを作業領域に使う

データには EXCEL 形式もあるので以下のようにしました。

1.  エクセル形式のファイルを Googl Drive へアップロード
2.  新規にスプレッドシートを作成
3.  スプレッドシートに `content` シートを作成
4.  `content` シートに `=IMPORTRANGE("アップロードしたファイルの URL","Sheet0!A1:Y118")` を入力

![アップロードした EXCEL データを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/981ecf24736a49da89967ac3a1372f5f/ogura-shuffle-api-source-data.png?w=600\&h=358\&auto=compress%2Cformat)*▲ 図 4-1 EXCEL 形式ファイルをアップロード*

![EXCEL データを参照しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/cb4c8175afff4d5aabe738944e2eba1a/ogura-shuffle-api-content-sheet.png?w=600\&h=358\&auto=compress%2Cformat)*▲ 図 4-2 EXCEL データを参照*

アップロードしたファイルを直接利用するのではなく、新規に別のスプレッドシートを作成した理由ですが「元データ更新時の対応を簡易化」するためです。

上記のように参照する関係にしておけば、更新時は(カラム構成が変わっていなければ)「新規ファイルをアップロードして、`IMPORTRANGE()`の URL を変更する」ことで対応できます。

## ランダムなレコード選択を定期実行

API が実行された時点で毎回ランダムなレコードを返却する方法もありますが、今回は一定期間で切り替わればよいので以下のようにしました。

1.  スプレッドシートに `shuffle-api` シートを作成
2.  `content` シートからランダムに 1 レコードを `shuffle-api` へ転記するスクリプトを作成
3.  スクリプトを定期実行

ここでもシートを新しく作成していますが、これは API に渡すデータを静的なレコードとして扱うためです。

静的なレコードとしてシートを分離しておくと、元データ更新時には「定期実行の設定を一時的に解除する」ことで API を停止することなく更新できます。

▼ *リスト 5-1 レコードを転記するスクリプト*

```js
function shuffle() {
  const start = 18
  const rowNum = 100
  const colNum = 25
  const pick = Math.floor(Math.random() * ((rowNum + start + 1) - start) + start)

  const ss = SpreadsheetApp.getActiveSpreadsheet()
  const content = ss.getSheetByName('content')
  const api = ss.getSheetByName('shuffle-api')

  const v = content.getRange(pick, 1, 1, colNum).getValues()

  api.getRange(2, 2, 1, colNum).setValues(v)
}
```

![シートに 1 レコードだけ表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e09ccc4e37a34e5eba18dddbc5ed7aae/ogura-shuffle-api-shuffle-api-sheet.png?w=600\&h=358\&auto=compress%2Cformat)*▲ 図 5-1 1 レコードのみ転記*

![スクリプトのトリガー設定を確認しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d8b60bae6c7640849f4cfcb8cca26d2c/ogura-shuffle-api-trigger.png?w=600\&h=358\&auto=compress%2Cformat)*▲ 図 5-2 定期実行の設定*

実際のスプレッドシートは以下で閲覧できます。現在の設定は 1 時間おきに更新しています。

@[card](https://docs.google.com/spreadsheets/d/1QkMEbZJnCOrl-cRHv5OoTFmlVr1KzPpECxQBIcw325k/edit?usp=sharing)

## SSSAPI で API 化

`shuffle-api` にランダムなデータが 1 レコード転機されている状態になりましたが、このままでは外部から取得できません。

アクセスを制限しないので [csv でエクスポートする方法](https://zenn.dev/katoaki/articles/c89562a3cd955e)もありますが、やはり JSON で扱いたいので [SSSAPI] を利用します。

今回は「常に 1 件のみ」「シートが定期的に更新される」なので API の設定は以下のようにします。

1.  1 行の場合は配列にしないを ON
2.  自動更新を ON

![API Option を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e2c787d402fd4fbf9fbdefa6cdaceaaa/ogura-shuffle-api-option.png?w=577\&h=472\&auto=compress%2Cformat)*▲ 図 6-1 API の設定を変更*

また、とくにアクセス制限は設定しないので、承認済みドメインを `*` にしてあります。

## API を実行

`curl` などでレコードを取得できます。なお、レコードは 1 件なので配列にはなっていません。

:::details (クリックで実行結果を表示)

```shell-session
$ curl https://api.sssapi.app/4CiDkGyE9Yb5KH2JSwYgf -s | jq
{
  "content-license": "#attribution_name        Nanako Takahashi\n#license        http://creativecommons.org/licenses/by/3.0/deed.ja\n#file_name        ogura\n#download_from        http://linkdata.org/work/rdf1s6834i",
  "#property": "ogura_023",
  "karuta:text": "月見れば 千々に物こそ 悲しけれ わが身ひとつの 秋にはあらねど",
  "karuta:historicalTranscription": "つきみれば ちゞにものこそ かなしけれ　わがみひとつの あきにはあらねど",
  "karuta:romanTranscription": "Tsuki mireba Chiji ni mono koso Kanashi kere Waga mi hitotsu no Aki ni wa aranedo.",
  "dc:creator": "大江千里",
  "dcterms:creator": "http://linkdata.org/resource/rdf1s6833i#kajin_023",
  "bibo:number": 23,
  "karuta:textOfYomi": "月見れば千々に物こそ悲しけれ わが身ひとつの秋にはあらねど",
  "karuta:textOfTori": "わかみひとつのあきにはあらねと",
  "karuta:imageOfYomi": "https://commons.wikimedia.org/wiki/File:Hyakuninisshu_023.jpg",
  "karuta:uniqueSyllable": "つき",
  "karuta:firstHalf": "月見れば 千々に物こそ 悲しけれ",
  "karuta:transcriptionOfFirstHalf": "つきみれば ちぢにものこそ かなしけれ　",
  "karuta:secondHalf": "わが身ひとつの 秋にはあらねど",
  "karuta:transcriptionOfSecondHalf": "わがみひとつの あきにはあらねど",
  "karuta:bonze": "殿",
  "dc:source": "古今集",
  "dc:subject": "秋",
  "dc:spacial": null,
  "geo:lat": null,
  "geo:long": null,
  "dcterms:refenreces": "http://linkdata.org/resource/rdf1s6837i#ndl_hishikawa_023|http://linkdata.org/resource/rdf1s6838i#ndl_nazorae_023|http://linkdata.org/resource/rdf1s6840i#osakaml_utena_023|http://linkdata.org/resource/rdf1s6856i#nijl_izumiya_023|http://linkdata.org/resource/rdf1s6839i#ndl_shikishi_023|http://linkdata.org/resource/rdf1s8472i#ndl_azuma_023|http://linkdata.org/resource/rdf1s8473i#ndl_toyokuni_023|http://linkdata.org/resource/rdf1s8474i#nijl_sugata_023|http://linkdata.org/resource/rdf1s8475i#kyoto_nakanoin_023",
  "bf:translation": "http://linkdata.org/resource/rdf1s8014i#porter_023|http://linkdata.org/resource/rdf1s8015i#clay_023|http://linkdata.org/resource/rdf1s8099i#dickins_023|http://linkdata.org/resource/rdf1s8100i#noguchi_023",
  "dcterms:hasFormat": "http://linkdata.org/resource/rdf1s8931i#audio_nhk_023",
  "dcterms:isPartOf": "http://linkdata.org/resource/rdf1s6836i#ogura"
}
```

:::

## おわりに

「とりあえず動くものを」的に作ってみましたが、スクリプト([Google Apps Script])部分は [clasp] で記述すればそこそこ込み入った処理も実装できます。

定期的に状態を変える API を簡単に作りたいときには便利に使える構成になりそうです。

[SSSAPI]: https://sssapi.app/

[Google Apps Script]: https://workspace.google.co.jp/intl/ja/products/apps-script/

[小倉百人一首かるたデータ]: http://linkdata.org/work/rdf1s6834i

[clasp]: https://github.com/google/clasp
