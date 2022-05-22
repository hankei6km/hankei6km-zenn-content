---
id: make-jog-map-by-geolonia-pwa-map-appsheet-sssapi
title: Geolonia PWA マップに AppSheet と SSSAPI を組み合わせてジョギングをサポートする地図を作ってみる
emoji: 📍
type: idea
topics:
  - appsheet
  - sssapi
  - map
published: true
---

以下の記事を見かけて「面白そうなので何か地図を作ってみよう」と思い少し考える…。夏に向けてジョギングをサポートする地図を作ってみることにしました。

@[card](https://internet.watch.impress.co.jp/docs/news/1409740.html)

###

## どのような地図？

「ジョギングの途中でへろへろになったとき」用に、コース沿いで給水や休息ができそうなスポットを登録しようと考えています。

▼ **図 1-1 地図の表示**

![作成した地図のホーム画面を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4fa266ddba7a40ad98eaffff6e2ef24a/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-intro-1.png?w=1435\&h=773\&auto=compress%2Cformat)

▼ **図 1-2 登録スポットの表示**

![作成した地図に登録されているスポットを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6d767a0124ab4131b38b73ccfdcbe284/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-intro-2.png?w=1440\&h=774\&auto=compress%2Cformat)

## 作り方

今回はいわゆる〇〇スポットや聖地とかの有名な場所ではないので「事前に住所などで検索してパソコンで入力する」のはちょっと大変そうです。

できれば「ジョギング中にスマートフォンの位置情報とカメラ」を使ってゆるく入力していきたいところです。

## Geolonia PWA マップを普通に使ってみる

まずは土台となる「Geolonia PWA マップ(以降 pwamap)」がどのような感じなのか、普通に使ってみます。

基本的には「[Geolonia PWAマップ ユーザーマニュアル（初期設定編）](https://blog.geolonia.com/2022/05/17/pwamap-manual-setup.html)」の手順でデプロイまでは実施できます。また更新についても、スプレッドシートなので「パソコンから行うなら」とくに迷うところもなかったです。

その上で気になった点など。

### 数値の切り貼りはスマートフォンには厳しい

「[Geolonia PWAマップ ユーザーマニュアル（マップ更新編）](https://blog.geolonia.com/2022/05/17/pwamap-manual-map.html)」によると「緯度と経度は数値を切り貼り」「画像は URL を入力」することが前提になっています。また、UI が必要であれば Google フォームを使えますが、これも入力は数値と URL ベースです。

スマートフォンで少し試してみましたが、やはり少し厳しかったです。できればスマートフォンで使いやすい UI が欲しくなります。

### ビルドジョブの起動は `schedule` か `workflow_dispatch`

基本は定期実行で必要であれば `workflow_dispatch` を使うことになりますが、これも屋外からだとちょっと厳しい感じです。

またスケジュール実行は必要のないときも動き続けるので、ジョブの起動は「データが更新されたときだけ」にしたいところです。

### 入力途中でも公開される

いわゆる CMS 的な「公開」「ドラフト」などの区分がないのでビルド処理が走るとスプレッドシートに入力しているものはすべて公開されてしまいます。

ジョギング中に入力する場合「とりあえず位置情報と画像だけ」ということは想定されるので、ここも対策を練る必要があります。

### GCP の設定はやはり面倒

マニュアルに沿って設定を進めていると「その共有の設定なら GCP(Sheets API)を使わなくてもできるのでは」という箇所がありました。

今回の本筋とは関係は薄いのですが、対応してしまおうかなと。

## AppSheet で UI 作成

「スプレッドシートに UI」といえば個人的な定番の [AppSheet](https://www.appsheet.com/) の出番です。

アプリの作り方はネット上に解説記事が豊富にあるので、ここでは、pwamap 用に追加したカラムなどについてのみ記載します。なお、pwamap では既存カラムの位置が特定の範囲を外れると動作しなくなります。よってカラムを追加する場合は「既存のカラムの後ろ」に追加します。

▼ **図 4-1 「公式サイト」カラムの右側が追加したカラム**

![スプレッドシートのカラム名を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e7d733c10b53411fbe51e0995e580d5b/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-cols-appendedpng.png?w=1060\&h=343\&auto=compress%2Cformat)

### id カラム

AppSheet で行を識別するための key が必要なので、`id` カラムを追加して以下のように設定しています。

*   Type - `text`
*   Initial value - `UNIQUEID()`
*   Key にチェック

▼ **図 4-2 id カラムの設定**

![id カラムの設定画面を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8062ee01952f47bc86d4af68b7cce68e/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-id-col.png?w=790\&h=841\&auto=compress%2Cformat)

### 位置情報カラム

AppSheet で「位置情報を扱うための型」は緯度と経度が 1 つのカラムになっています。そのままでは pwamap のスプレッドシートに反映できないので、「位置情報」カラムを追加し、計算式で「緯度」「経度」カラムに分解しています。

*   位置情報カラムの Type は `LatLong`
*   緯度カラムは Decimal digits を `6` 、App formula を `INDEX(SPLIT([位置情報], ","),1)`
*   経度カラムは Decimal digits を `6` 、App formula を `INDEX(SPLIT([位置情報], ","),2)`

▼ **図 4-3 位置情報カラムの設定**

![位置情報カラムの設定画面を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/39cc44b20bfd4b3a9091e3d10b641dad/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-lat-long-col.png?w=793\&h=836\&auto=compress%2Cformat)

▼ **図 4-4 緯度カラムの設定**

![緯度カラムの設定画面を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3708938803234aaf80f792134f8127f5/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-lat-col.png?w=792\&h=843\&auto=compress%2Cformat)

▼ **図 4-5 経度カラムの設定**

![経度カラムの設定画面を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c01bbe7e27d14a279aa8a157b4abfce1/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-long-col.png?w=787\&h=841\&auto=compress%2Cformat)

### 画像入力カラム

AppSheet で画像を扱う場合、スプレッドシートにはアップロードされたファイルへの相対パスが記録されます。そのままだと外部サービスからアクセスできないので、まずは「画像入力」カラムを追加し画像をアップロードできるようにします。

▼ **図 4-6 画像入力カラムの設定**

![画像入力カラムの設定画面を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/408062df812249abb08db3c18cd7682e/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-input-image-col.png?w=799\&h=846\&auto=compress%2Cformat)

その後に、以下の「External URL for image files」手順で URL へ変換し「画像」カラムにセットしています。

@[card](https://support.google.com/appsheet/answer/10107317)

### 公開カラム

スポット情報の「公開」を管理するフラグとしてカラムを追加しています。

*   Type は `Yes/No`
*   Initial value は `false`

▼ **図 4-7 公開カラムの設定**

![公開カラムの設定画面を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f84955916d2e4265974bb5f43f93e906/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-published-col.png?w=800\&h=848\&auto=compress%2Cformat)

### 空行

これは追加カラムではないのですが、補足として記述します。

AppSheet では追加と削除を繰り返すとスプレッドシートに空行ができてしまいます。これは他のサービスと連携させようとすると地味に面倒ですが、今回は pwamap 側で対応することにしました。

## pwamap データ受信スクリプトの改造

スプレッドシートのカラムに公開フラグを追加したので、データを受信するスクリプトにも変更が必要になります。これについては少し分量があるので概要のみ記載します。

*   空行対応 - 前述のように AppSheet が空行を挿入するのでそれの除外
*   公開フラグ -「スポットデータ」シートでは「公開」カラムが `TRUE` の行のみ対象とする

また(今回の地図作成に必須ではないですが)、マニュアルでは「共有範囲をリンクを知っている全員」にしていたので「Sheets API を使わず CSV で受信(エクスポート)」するようにしました。これにより、GCP の設定無しで利用できるようになっています。

改造したスクリプトは以下のリンクから確認できます。

@[card](https://github.com/hankei6km/pwamap-tamariver-jog-map/blob/master/bin/download.js)

## SSSAPI による GitHub Actions ワークフローの開始

スプレッドシートが更新されたときにワークフローが開始されるように、[Apps Script connector for AppSheet](https://developers-jp.googleblog.com/2022/05/apps-script-connector-for-appsheet.html) と[以前に作成したライブラリー](https://zenn.dev/hankei6km/articles/fetch-github-apps-token-by-google-apps-script)で設定しようとしたのですが…。

その途中で [SSSAPI 公式](https://zenn.dev/sssapi)さんの「[GitHub Actionsと連携できるようになりました](https://zenn.dev/sssapi/articles/5782c15b94716a)」の記事に気がつきました。そこで(本来の使い方ではないのが申し訳ないところですが)、こちらの機能を利用することにしました。

設定自体は手順の通りに進めるだけなので、とくに引っかかることもなく完了しました。

@[card](https://sssapi.app/help/api_option_webhook_github_actions)

▼ **図 6-1 SSSAPI 側ではリポジトリなどを登録**

![GitHub Actions 連携の設定を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1d1a68067e694a9d88e84b967fd8dda6/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-gha-hook.png?w=801\&h=309\&auto=compress%2Cformat)

▼ **図 6-2 ワークフローは以下のトリガーを追加**

```yaml
  repository_dispatch:
    types:
      - sssapi-api-build-success   # 成功時
```

これによりスプレッドシートが更新されたときにワークフローが実行されます。

また、ワークフローは以下の点も変更しています。

*   環境変数のクオート対応 - actionlint から警告が出ていたので修正
*   npm モジュールのキャッシュ化 - 実行時間は短縮しておきたい
*   concurrency の設定 - ユーザー操作がトリガーになるので多重起動対応

## 使ってみた

設定ができたので週末のジョギングで使ってみましたところ、おおよそ予定していた感じで入力できました。

▼ **図 7-1 AppSheet での入力**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/218cc228bdc247aebb3cada0fcf771fb/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-edit.png?auto=compress%2Cformat)

公開フラグを設定すれば、数分で入力したものが反映されます。

▼ **図 7-2 ビルドした地図の表示**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e0282950c6ea4531ab910d27909a557b/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-sample.png?auto=compress%2Cformat)

ただし「位置情報の扱い」「スポット名の入力」などの課題もありました。

### 現在位置とタイムスタンプ

スマートフォンの位置情報を使うのは便利ですが、タイムスタンプ付きで地図に登録すると追跡されやすいかなと感じました。

公開時期を制御できるようにはなっていますが「AppSheet では画像のファイル名に日時が入る」ので、同じような構成で pwamap を使う場合は注意が必要です。

### スポット名の入力

位置情報からスポット名を入力する方法は準備していなかったので、現場では「とりあえず手動で入力」することになりました。

この辺については少し対策もあるので次節で。

## OCR

少し前に [Notion と Google Drive で OCR を利用できるようにしていた](https://zenn.dev/hankei6km/articles/easily-use-google-drive-ocr-with-notion)ので、施設の名前などが掲示されていたらそれを利用しました。

ただし、遠くの看板などはわりといけたのですが、石碑などは厳しいようです。

以下は少しおかしな部分もあるが、大体うまく取り込める。

▼ **図 8-1 通りを挟んだところの看板**

![通りを挟んだところの看板を Google Drive でスキャンした PDF の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/88a45340e1e94ffe8e0ece50224dd8f1/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-ocr-ok.png?w=802\&h=564\&auto=compress%2Cformat)

▼ **図 8-2 OCR の結果**

    お買和泉自動車教習所 東京都 安委員会 指定

以下は OCR 自体ができませんでした。

▼ **図 8-3 石碑の文字**

![石碑の文字を Google Drive でスキャンした PDF の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7f69a4d9de534cf288582714edb4b66b/make-jog-map-by-geolonia-pwa-map-appsheet-sssapi-ocr-ng.png?w=801\&h=560\&auto=compress%2Cformat)

## おわりに

Geolonia PWA マップに AppSheet と SSSAPI を組み合わせてジョギングをサポートする地図を作ってみました。

準備に少し手間がかかりましたが、入力自体はこの週末で結構進んだので今後は「暑さでへろへろになったとき」に活用できればと考えています。
