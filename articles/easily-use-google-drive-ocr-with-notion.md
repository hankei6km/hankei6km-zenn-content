---
id: easily-use-google-drive-ocr-with-notion
title: Notion で Google Drive の OCR を簡単に使えるんですか！？
emoji: 🙆
type: idea
topics:
  - notion
  - googledrive
  - googleappsscript
published: true
---

Notion から使える OCR 機能が欲しかったので、下記の記事を参考に GAS のライブラリーを作ってみました。

@[card](https://zenn.dev/harachan/articles/d910ef8b89720b)

この記事ではライブラリーの設定方法などを記載していきます。

## どんなライブラリー？

Google Drive 上に保存されている PDF と画像に OCR を実行し、結果のテキストを Notion へ送信するライブラリーです。

@[card](https://github.com/hankei6km/gas-gocr2notion)

定期実行や変更通知に対応しているので、OCR の結果を自動的に Notion へ送信できます。

▼ **図 1-1 PDF を所定のフォルダーに保存すると**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1137debebca04fa4837eacdf452b722c/easily-use-google-drive-ocr-with-notion-mobile-save.png?auto=compress%2Cformat)

▼ **図 1-2 テキストとして Notion へ送信される**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4c04eaa97a9147038fdc64b72822f82a/easily-use-google-drive-ocr-with-notion-ocr-mobile-result.png?auto=compress%2Cformat)

また、少し設定を頑張れば「スマートフォンのカメラで撮影した後、リアルタイムに近い感覚で OCR の実行」も可能です。

▼ **図 1-3 フォルダーに保存後、10 ～ 20 秒くらいでテキストが登録される(実際の画面にタイムコードは表示されません)**
![](/images/easily-use-google-drive-ocr-with-notion/easily-use-google-drive-ocr-with-notion-at-uploaded.gif)

## おおまかな設定の流れ

設定する項目がサービス別に分散していたりするので、最初に少し整理します。

おおまかなには、以下のような設定することで Notion から OCR が利用できるようになります。

*   Notion 側では API キーやテキストを受け取るデータベースを用意

*   Google Drive 側ではスキャンしたファイルを保存するフォルダーを作成

*   Google Apps Script でライブラリーを利用するためのプロジェクトを作成

    *   プロジェクトの定期実行などのカスタマイズを適宜行う

## Notion 側の準備

### Notion API キー(インテグレーション)を作成

外部(Google App Script)から Notion を利用するための API キーが必要です。

「[私のインテグレーション | Notion 開発者](https://www.notion.so/my-integrations)」からインテグレーションを作成し API キーをメモしておきます。

▼ **図 3-1 上記リンクを開いたら「+ 新しいインテグレーション」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2760be00b30240008e834325d6e70306/easily-use-google-drive-ocr-with-notion-new-integration.png?auto=compress%2Cformat)

▼ **図 3-2 基本情報の「名前」へ「OCR」などを入力**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1d6ffef6625e4781bc4b0ec8bd423641/easily-use-google-drive-ocr-with-notion-new-integration-info.png?auto=compress%2Cformat)

▼ **図 3-3 画面を下にスクロールし「ユーザー機能」 は「ユーザー情報なし」へ変更、その後「送信」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/27b0499f2fc946c29e63af94e4661fcc/easily-use-google-drive-ocr-with-notion-new-integration-func.png?auto=compress%2Cformat)

▼ **図 3-4 インテグレーションが作成されたらトークン欄の「表示」をクリックし内容をメモ(コピー)**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3b9996802dd54ec5bb5d07bfdf69e07f/easily-use-google-drive-ocr-with-notion-new-integration-token.png?auto=compress%2Cformat)

### Notion データベースの用意

必須のプロパティはないのでデータベースであれば既存のもの、新規作成のものどちらでも利用できます。

既存のデータベースを利用する場合はブラウザーで開き id をメモします。

▼ **図 3-5 アドレスバーの赤く塗りつぶした部分をメモ**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2f2a05dcdcae4d8e89e53bb521c982f6/easily-use-google-drive-ocr-with-notion-database-id.png?auto=compress%2Cformat)

以下は新規作成する場合の操作例です。

▼ **図 3-6 任意のページで「/data」と入力後、「データベース: フルページ」を選択**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2a5832aa72e243f58f0f04b918a508a2/easily-use-google-drive-ocr-with-notion-new-database.png?auto=compress%2Cformat)

▼ **図 3-7 データベース名を「OCR」などへ変更**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d3435c5988854aac9f61d48b6e1119bf/easily-use-google-drive-ocr-with-notion-database-name.png?auto=compress%2Cformat)

▼ **図 3-8 ビューのレイアウトを「ギャラリー」など扱いやすい設定へ変更**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5ac5f50a7da149d0b10650abae945229/easily-use-google-drive-ocr-with-notion-database-view.png?auto=compress%2Cformat)

▼ **図 3-9 アドレスバーの赤く塗りつぶした部分をメモ**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2f2a05dcdcae4d8e89e53bb521c982f6/easily-use-google-drive-ocr-with-notion-database-id.png?auto=compress%2Cformat)

なお、この方法でデータベースを作成した場合、デフォルトでブランクのレコードが 3 件追加されています。とくに必要はないので削除してください。

### Notion データベースを APIキー(インテグレーション)と共有する

API キーでデータベースを操作できるように共有する必要があります。

用意したデータベースを開き以下のように操作します。

▼ **図 3-10 データベース画面の右上にある「共有」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6f01f5091cd84ac7abde5c79c6c4468f/easily-use-google-drive-ocr-with-notion-share-btn.png?auto=compress%2Cformat)

▼ **図 3-11 「ユーザー、メール…」などが表示されているフィールドをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b43511fe217943fdbc755baabfd91e1a/easily-use-google-drive-ocr-with-notion-share-blank%3Bfld.png?auto=compress%2Cformat)

▼ **図 3-12 作成したインテグレーションを選択**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/76fe317816ea4338b16b66794a152ac0/easily-use-google-drive-ocr-with-notion-share-select.png?auto=compress%2Cformat)

▼ **図 3-13 「招待」ボタンをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/169e0be0de364b0cac0379b456a2018e/easily-use-google-drive-ocr-with-notion-share-invite.png?auto=compress%2Cformat)

▼ **図 3-14 インテグレーションが追加されたら完了です**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2428d4def71b46f9a2f638419b21fa29/easily-use-google-drive-ocr-with-notion-share-done.png?auto=compress%2Cformat)

## Google Drive 側の準備

Google Drive 側ではスキャンされた PDF などを保存するためのフォルダーが必要です。フォルダーを 1 つ作成し id をメモしておきます。

▼ **図 4-1 フォルダーを表示しアドレスバーの赤く塗りつぶした部分をメモ**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/de20762724224e1fbe782a6427cd6c52/easily-use-google-drive-ocr-with-notion-folder-id.png?auto=compress%2Cformat)

## ライブラリーの利用(基本編)

ライブラリーの利用法はいくつかありますが、ここでは簡単なプロジェクトを作成してみます。

### プロジェクトを作成

Google Drive のウェブ UI で Google Apps Script プロジェクト(ファイル)を作成します。

おおまかな手順としては「プロジェクトを作成し名前の設定」「ライブラリーの追加」「Drive API サービスの有効化」「スクリプトコードの編集」となります。

なお、今回は手順の簡略化のために API キーなどをコードにべた書きしていますが、できれば他の方法(プロパティサービスの利用など)を検討してください。

▼ **図 5-1 ブラウザーの Google Drive で右クリックし「Google Apps Script」を選択**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1f4633d2543843148924be9c530389da/easily-use-google-drive-ocr-with-notion-script-new.png?auto=compress%2Cformat)

▼ **図 5-2 「無題のプロジェクト」のところをクリックしプロジェクト名を変更**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0bd1f73776ce49dda29a93b3230bbc94/easily-use-google-drive-ocr-with-notion-script-rename.png?auto=compress%2Cformat)

プロジェクト名を変更したら、OCR のライブラリー(`1phsqy39NYEBOCpGx062N9kezLOdyzRKSEgDBYt6f-zaqVdAOdn9NaWWH`)を追加します。

▼ **図 5-3 「ライブラリー」横の「+」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2d77ba69f7914ab093bcf2e85a0abb2b/easily-use-google-drive-ocr-with-notion-script-library.png?auto=compress%2Cformat)

▼ **図 5-4 スクリプト ID** `1phsqy39NYEBOCpGx062N9kezLOdyzRKSEgDBYt6f-zaqVdAOdn9NaWWH` **を入力し「検索」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a8b95ef9f3cc460b96738b0bf0f435e5/easily-use-google-drive-ocr-with-notion-script-search.png?auto=compress%2Cformat)

▼ **図 5-5 バージョンはその時点で最新のものを選択、ID を「GocrToNotion」へ変更**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/49f05f6667a84dbbbc61ade6886c8d33/easily-use-google-drive-ocr-with-notion-script-id.png?auto=compress%2Cformat)

続いて Drive サービスを有効化します。

▼ **図 5-6 「サービス」横の「+」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/151305a467364b608f3ab57f3764f49a/easily-use-google-drive-ocr-with-notion-service-add.png?auto=compress%2Cformat)

▼ **図 5-7 一覧から「Drive API」 を選択し「追加」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/79cfe18b2f2f4b31b342ff26ff51561d/easily-use-google-drive-ocr-with-notion-service-select.png?auto=compress%2Cformat)

最後にエディター上の `myFunction` を以下のように変更します。このとき各項目をメモしておいた値へ置き換えます。

*   `'notion_api_key'` にはインテグレーションの API キーを記述
*   `'database_id'` には用意したデータベースの id を記述
*   `'folder_id'` には作成したフォルダーの id を記述

```js
function myFunction() {
  const apiKey = 'notion_api_key'
  const database_id = 'database_id'
  const folder_id = 'folder_id'

  const opts = {
    database_id,
    ocrOpts: [
      {
        scanFolderId: folder_id,
        ocrFolderId: folder_id,
        ocrLanguage: 'ja',
        tags: [],
        removeOcrFile: true
      }
    ]
  }

  GocrToNotion.ocr(apiKey, opts)
}
```

### OCR を実行してみる

作成したフォルダーに PDF か画像ファイルを保存します。

スマートフォンのカメラを使う場合は以下のようにします。

▼ **図 5-8 Google Drive のアプリでフォルダーを表示し「+」、「スキャン」をタップ**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e412528a7db2450f944b69b8daa8b8eb/easily-use-google-drive-ocr-with-notion-mobile-scan.png?auto=compress%2Cformat)

▼ **図 5-9 スキャンして PDF をフォルダーへ保存します**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1137debebca04fa4837eacdf452b722c/easily-use-google-drive-ocr-with-notion-mobile-save.png?auto=compress%2Cformat)

フォルダーにファイルが保存されたらスクリプトエディターから `myFunction` を実行します。

▼ **図 5-10 「実行」ボタンをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d65769db804d4cb08677885f15edc48c/easily-use-google-drive-ocr-with-notion-run-my-func.png?auto=compress%2Cformat)

初回実行時には、以下のような画面が表示されるので、今回利用するライブラリーを怪しいと思わなければ画面を進めてください。

▼ **図 5-11 「権限を確認」ボタンをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f4eaf7417dd24188860d9b9380cc9069/easily-use-google-drive-ocr-with-notion-run-dilog1.png?auto=compress%2Cformat)

▼ **図 5-12 赤く塗りつぶした部分が自身のメールアドレスであることを確認し、左下の「詳細」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/42b74ade9f2c43f9b32999e59b20cd22/easily-use-google-drive-ocr-with-notion-run-danger.png?auto=compress%2Cformat)

▼ **図 5-13 画面が展開されるので左下の「(安全ではないページ)に移動」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9bf0534bc89f4e37918df70a2ee680bb/easily-use-google-drive-ocr-with-notion-run-danger2.png?auto=compress%2Cformat)

▼ **図 5-14 利用される権限が表示されるので確認**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c8a117d4a8674d148251084e7a516520/easily-use-google-drive-ocr-with-notion-run-req1.png?auto=compress%2Cformat)

▼ **図 5-15 画面をスクロールさせ「許可」ボタンをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f5641b110f7a4b8db2f7b1aba5fb518b/easily-use-google-drive-ocr-with-notion-run-req2.png?auto=compress%2Cformat)

問題がなければ、少し待った後に OCR の結果のテキストが Notion のデータベースに追加されます。また、このとき元の PDF ファイルの説明にもテキストがセットされます。

▼ **図 5-16 Notion のデータベースにテキストが追加される**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1e46f68e5e244fe2adab49b595721e94/easily-use-google-drive-ocr-with-notion-ocr-result.png?auto=compress%2Cformat)

▼ **図 5-17 Google Drive のファイルに説明テキストが追加される**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/358159ae74a4499bbf28684180d61281/easily-use-google-drive-ocr-with-notion-ocr-description.png?auto=compress%2Cformat)

## ライブラリーの利用(応用編)

### 定期的に実行する

毎回スクリプトエディターから実行するのは大変なので、以下のように設定しておくと定期的に実行できます。

▼ **図 6-1 スクリプトエディターの「時計」アイコンをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/99e2dad5c6d345a6a171e21e528cc2c7/easily-use-google-drive-ocr-with-notion-trigger-add.png?auto=compress%2Cformat)

▼ **図 6-2 画面が切り替わったら、右下の「トリガーを追加」ボタンをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a058d4123e774400bddaf1533bc52572/easily-use-google-drive-ocr-with-notion-trigger-add2.png?auto=compress%2Cformat)

▼ **図 6-3 デフォルトで「1 時間おき」に実行される設定画面が表示される**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d5a44c313e1547918dd37cea879df3d5/easily-use-google-drive-ocr-with-notion-trigger-settings.png?auto=compress%2Cformat)

▼ **図 6-4 画面をスクロールさせ「保存」ボタンをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/831ed60f69e44f33b0c725ae57e43170/easily-use-google-drive-ocr-with-notion-trigger-done.png?auto=compress%2Cformat)

なお、フォルダーに複数のファイルがあった場合、作成日時の新しい順に 100 ファイルを対象とします。また、ファイルの説明が空欄でない場合は OCR 処理をスキップします。

よって、以下のようなファイルは対象にならないので注意してください。

*   作成日時が古いファイルを別フォルダーからコピーした場合
*   ファイルの説明を追加している場合

### フォルダー別に設定を切り替える

以下のように `ocrOpts` の配列で異なるフォルダーを指定すると、フォルダー毎に設定の切り替えができます。

```js
  const opts = {
    database_id,
    ocrOpts: [
      {
        scanFolderId: folder_id1,
        ocrFolderId: folder_id1,
        ocrLanguage: 'ja',
        tags: ['メモ'],
        removeOcrFile: true
      }, {
        scanFolderId: folder_id2,
        ocrFolderId: folder_id2,
        ocrLanguage: 'ja',
        tags: ['レシート'],
        removeOcrFile: true
      }
    ]
  }
```

### PDF へのリンクを追加

Notion のデータベースに `link` プロパティを追加すると、Google Drive 上のファイルへリンクされます。

▼ **図 6-5 link にURL が追加され、クリックすると Google Drive のファイルを開く**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/11f7311c00ee4b37904a8a0e6082d22c/easily-use-google-drive-ocr-with-notion-props-link.png?auto=compress%2Cformat)

### テキストの先頭部分をプロパティにセットする

Notion のデータベースに `excerpt` プロパティを追加すると、テキスト先頭の 1900 バイトがセットされます。

▼ **図 6-6 excerpt にもテキストが追加される**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/37902eaf5df64bd2ba22e5822ff7183a/easily-use-google-drive-ocr-with-notion-props-excerpt.png?auto=compress%2Cformat)

## ライブラリーの利用(ほぼリアルタイム編)

Google Drive の変更通知(プッシュ通知)を利用すると、ファイル追加時にほぼリアルタイムで OCR を実行できます。原理的な部分は長くなるので[^watch]、ここでは設定についてのみを記載します。

[^watch]: GAS と変更通知については「[Google Drive の変更通知を Google Apps Script で受け取る](https://zenn.dev/hankei6km/articles/receive-google-drive-chages-notifications-by-gas)」で記事にしています。

最初に「ライブラリーの利用(基本編)」で作成したプロジェクトのコードを以下のように変更します。

`ocr` 関数の各項目は「ライブラリーの利用(基本編)」のときと同じです。`settings_` 関数の `deploy_url` は後から変更するので、ここではこのままにしておきます。

```js
function ocr_(list) {
  const apiKey = 'notion_api_key'
  const database_id = 'database_id'
  const folder_id = 'folder_id'

  const opts = {
    database_id,
    ocrOpts: [
      {
        scanFolderId: folder_id,
        ocrFolderId: folder_id,
        ocrLanguage: 'ja',
        tags: [],
        removeOcrFile: true
      }
    ]
  }

  GocrToNotion.send(apiKey, opts, list)
}

function settings_(properties) {
  const expiration = 61
  const address = 'deploy_url'

  return {
    resource: {
      id: Utilities.getUuid(),
      type: 'web_hook',
      token: '',
      expiration: `${new Date(
        Date.now() + 60 * expiration * 1000
      ).getTime()}`,
      address
    }
  }
}

function doGet(e) { }
function doPost(e) {
  if (e.postData && e.postData.contents) {
    const lock = LockService.getScriptLock()
    if (lock.tryLock(10 * 1000)) {
      try {
        const props = PropertiesService.getScriptProperties()

        const pageToken = props.getProperty('page_token')

        const res = Drive.Changes?.list({ pageToken })

        ocr_(res)

        props.setProperty('page_token', res?.newStartPageToken || '')
      } catch (e) {
        console.error(e)
      } finally {
        lock.releaseLock()
      }
    }
  }
}

function reset() {
  const props = PropertiesService.getScriptProperties()
  const properties = props.getProperties()
  const settings = settings_(properties)

  const resToken = Drive.Changes?.getStartPageToken()
  const pageToken = JSON.parse(resToken).startPageToken
  props.setProperty('page_token', pageToken)

  const res = Drive.Changes?.watch(settings.resource)
  console.log(JSON.stringify(res, null, ' '))

  try {
    stop(properties)
  } catch (e) {
    console.error(e)
  }

  properties.channle_id = settings.resource.id
  properties.resource_id = res?.resourceId || ''
  props.setProperties(properties)
}

function stop(inProperties) {
  const props = PropertiesService.getScriptProperties()
  const properties = inProperties || props.getProperties()

  const { channle_id, resource_id } = properties

  if (channle_id && resource_id) {
    const res = Drive.Channels?.stop({
      id: channle_id,
      resourceId: resource_id
    })
    console.log(JSON.stringify(res, null, ' '))
  }
}
```

コードを変更したら `deploy_url` の値を確定させます。

▼ **図 7-1 スクリプトエディター右上の「デプロイ」「新しいデプロイ」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a7635e7c63c84d04bc17b132c22c9efb/easily-use-google-drive-ocr-with-notion-notify-deploy.png?auto=compress%2Cformat)

▼ **図 7-2 画面が表示されたら歯車アイコンをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8242585797d8488e8f218b76b4803635/easily-use-google-drive-ocr-with-notion-notify-deploy-type.png?auto=compress%2Cformat)

▼ **図 7-3 「ウェブアプリ」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d5f8dae5fd754b45a50f8eb6b3289ccc/easily-use-google-drive-ocr-with-notion-notify-deploy-type2.png?auto=compress%2Cformat)

▼ **図 7-4 「アクセスできるユーザー」を「全員」へ変更し「デプロイ」ボタンをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2173fae4918b49059c750a9c6cc0fa3b/easily-use-google-drive-ocr-with-notion-notify-deploy-grant.png?auto=compress%2Cformat)

▼ **図 7-5 デプロイの「URL」をメモ(コピー)し「完了」ボタンをクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0bec2417eb4b43c09ad4c13564ff8e2d/easily-use-google-drive-ocr-with-notion-notify-deploy-copy.png?auto=compress%2Cformat)

▼ **図 7-6 コピーした URL を `settings_` 関数の `deploy_url`へセット**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e55d9619f9794d1da00d95768c315e17/easily-use-google-drive-ocr-with-notion-notify-set-deploy-url.png?auto=compress%2Cformat)

これで更新通知にあわせて OCR を実行する設定は完了しました。

動作確認のためにスクリプトエディターから `reset` 関数を実行します。

▼ **図 7-7 スクリプトエディターのツールバーで ****`reset`**** 関数を選択し「実行」をクリック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0ece8d501c18407da98552ceb419cce0/easily-use-google-drive-ocr-with-notion-notify-run-reset.png?auto=compress%2Cformat)

▼ **図 7-8 Google Drive 上のファイル変更にあわせてログが追加される**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ae639aff17ae42d98a85b7eba28c5c9a/easily-use-google-drive-ocr-with-notion-notify-log.png?auto=compress%2Cformat)

この状態で OCR 用のフォルダーに PDF ファイルを追加すると OCR が実行されます[^log]。

[^log]: 今回の設定ではログの詳細は確認できません。プロジェクトを GCP プロジェクトへ変更することで確認できます([Google Cloud Platform Projects | Apps Script | Google Developers](https://developers.google.com/apps-script/guides/cloud-platform-projects))。

ただし、通知設定の有効期限を 61 分としているので、以下のように時間主導のトリガーを設定し 1 時間毎にリセットさせます。

▼ **図 7-9 実行する関数を ****`reset`**** へ変更してトリガーを追加**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0228f6d272e640a6bb43ad127b5a526f/easily-use-google-drive-ocr-with-notion-notify-trigger.png?auto=compress%2Cformat)

これで OCR 用のフォルダーへ PDF ファイルを追加したら OCR が実行される状態が永続化されます。

状態を停止させるときはスクリプトエディターから `stop` 関数を実行させてください。

## おわりに

Notion から Google Drive の OCR を利用できるライブラリーを作ってみました。

作ってからまだ日が浅いのでそれほど使い込んでいませんが、掲示物などをメモするときの負担は減らせたと実感しています。

また、今回の記事では割愛していますが、Notion へ送信する内容を操作できるようにもしてあるので[^transformer]、翻訳などのサービスを組み合わせるのも便利そうかなと考えています。

少し張り切りすぎて設定方法を長々と書いてしまいましたが、定期的な OCR の実行は比較的手軽にできるので、よければ試してみたください。

[^transformer]: 「Notion を通知機能付き RSS フィードリーダーぽくする GAS ライブラリーを作ってみた」の [Transformer](https://zenn.dev/hankei6km/articles/make-gas-library-to-setup-notion-as-feed-reader#transformer) と同じようなことができます。
