---
id: receive-google-drive-chages-notifications-by-gas
title: Google Drive の変更通知を Google Apps Script で受け取る
emoji: 📡
type: tech
topics:
  - googleappsscript
  - googledrive
published: true
---

[Google Apps Script] で [Google Drive] のファイル変更を検出したくなり、変更通知(プッシュ通知)を [Google Apps Script] で受信する方法を試してみました[^schedule]。

[^schedule]: 最初は時間指定を利用してみましたが、編集してからしばらく更新されないのはやはり違和感が大きかったです。

:::message
以下の内容では [clasp] と [TypeScript] を使っています。
:::

## Google Drive の変更通知とは

ざっくり書くと「[Google Drive] のドライブやファイルに変更があったとき Webhook で通知を受け取る」機能です。

詳細については丸投げになってしまいますが、以下の記事が参考になります。

@[card](https://qiita.com/wezardnet/items/85161194d35baf0b18c7)

## Google Apps Script プロジェクトのセットアップ

[Google Drive API] の変更通知を受け取るためには以下の設定が必要となります。

*   Drive サービスの追加
*   「アクセスできるユーザーが全員」の Web アプリとしてデプロイ

これらは `appsscript.json` を編集することで対応できます。

```json:appsscript.json
{
  "timezone": "america/new_york",
  "dependencies": {
    "enabledadvancedservices": [
      {
        "usersymbol": "drive",
        "version": "v2",
        "serviceid": "drive"
      }
    ]
  },
  "exceptionlogging": "stackdriver",
  "runtimeversion": "v8",
  "webapp": {
    "executeas": "user_deploying",
    "access": "anyone_anonymous"
  }
}
```

## Google Drive から変更通知を受け取るコード

基本的には前述の記事にある Go のコードを [Google Apps Script] で記述することになりますが、いくつか固有の対応が必要になります(リクエストヘッダーの扱いなど)。

今回はドライブ全体を扱いたいのもあったので、以下のようにしました。

*   `startPageToken` などはスクリプトプロパティへ保存

*   `X-Goog-Resource-State` での通知の種類判定は行わない

    *   かわりに `doPost` の `e.postData.contents` 有無で判定する

*   Channel の有効期限対策として、時間指定のトリガーで Channel を再作成する

*   変更の記録としてスプレッドシートへファイル名と id を書き込む

    *   `startPageToken` がよれてしまわないように念のためスクリプトロックで順序を制御

以下のリポジトリにコードが記載されています。

@[card](https://github.com/hankei6km/test-drive-chages)

## 実際に受信してみる

プロジェクトとコードが用意できたので変更通知を受信してみます。

### スプレッドシート

通知されたファイル名が書き込まれるスプレッドシートを作成し id とシート名をメモしておきます。

### デプロイと設定

*   `git clone https://github.com/hankei6km/test-drive-chages.git . && npm install` でリポジトリーをクローン
*   クローンしたリポジトリで `clasp create` を実行し webapp のスクリプトを作成
*   `clasp push` と `clasp deploy` でデプロイを作成

ここでデプロイの URL が必要になります。これは [clasp] では確認できないので、スクリプトエディターの「デプロイ」「デプロイの管理」を利用します。

▼ **図 4-1 URL をコピー**

![スクリプトエディターでデプロイの URL をコピーできる画面を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7c9d661b744c40d29da548453621d2bc/receive-google-drive-chages-notifications-by-gas-deploy-url.png?w=700\&h=552\&auto=compress%2Cformat)

その後、スクリプトプロパティへ以下の内容を設定します。

*   `address` - デプロイの URL
*   `sheet_id` - スプレッドシートの id
*   `sheet_name` - シート名

### 受信開始

スクリプトエディターで `reset()` 関数を実行すると [Google Drive] から通知が行われます。ドライブ内のファイルを変更するとスプレッドシートへファイル名と `id` が書き込まれます。

▼ **図 4-2 更新の状況がスプレッドシートへ書き込まれる**
![Google Drive に複数ファイルをコピーし、その後ファイルを 1 つ削除している動画](/images/receive-google-drive-chages-notifications-by-gas/receive-google-drive-chages-notifications-by-gas-write-to-ss.gif)

### 永続化

channel の有効期限より短い間隔(今回にコードでは 31 分)で `reset()` 関数を実行すると永続的に受信できます。

### 受信終了

確認が終わったら `stop()` 関数を実行して channel を破棄します。

### ログ表示

今回はスプレッドシートへ書き込んでいますが、通常は `console.log` などを利用したくなります。しかし、自分以外がリクエストした POST による関数実行ではログの内容が確認できません。

これは GCP プロジェクトへ変更することで対応できますが、今回は割愛します(検索すると方法はすぐに見つかります)。

## おわりに

[Google Drive] の変更通知を [Google Apps Script] で受信してしました。

今回のコードを 1 日ほど動かしてみたところ自分の使い方ではパンクすることなく、概ね変更を検出できていました[^throttle]。

[^throttle]: 通知には throttle 的な処理が入っているようなので抜けは出てしまいます。

これで [Google Apps Script] でもファイルの変更を扱いやすくなったので、作成中のツールでも変更のあったファイルをほぼリアルタイムに操作できそうです。

[Google Apps Script]: https://workspace.google.co.jp/intl/ja/products/apps-script/

[clasp]: https://github.com/google/clasp

[Google Drive]: https://www.google.com/intl/ja_jp/drive/

[Google Drive API]: https://developers.google.com/drive/api

[TypeScript]: https://www.typescriptlang.org/
