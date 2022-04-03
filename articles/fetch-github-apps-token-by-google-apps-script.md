---
id: fetch-github-apps-token-by-google-apps-script
title: GitHub Apps の Access Token を取得するGoogle Apps Script 用ライブラリーを作った
emoji: 🎫
type: tech
topics:
  - githubactions
  - googleappsscript
  - jwt
published: true
---

Google Apps Script から GitHub Actions のワークフローを開始したくなったのですが、開始するには対象リポジトリの許可(permission)が必要になります。

定番としては Personal Access Token の利用になりますが、GitHub では有効期限を短く設定するよう警告も出るので他の方法を探したところ以下の記事がヒットしました。

@[card](https://zenn.dev/tatsuo48/articles/72c8939bbc6329)

そこで Google Apps Script で同じようなことが出来ないか試してみました。

## GitHub Apps アプリの用意

まずはテスト用のアプリを用意します。

### アプリ作成

基本的には冒頭記事の通りです。

> `GitHub App name`はアプリの名前をつけてあげて下さい。
> `Homepage URL`は適当でOKです。
> `Webhook`は`Active`のチェックを外してあげてください。
> 権限は必要なものをつけていきます。

権限はワークフローの開始(`workflow_dispatch` を利用)用に Repository permissions の Actions に Read and Weite を指定します。

作成したら App Id を控えておきます。

▼ **図 1-1 アプリを作成すると App Id が表示される**

![GitHub Apps でアプリを作成したスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7d0c8905fc8c40908ed237629f71b59f/fetch-github-apps-token-by-google-apps-script-app-created.png?w=596\&h=327\&auto=compress%2Cformat)

また、あわせてプライベートキーをダウンロード(Generate)しておきます。

▼ **図 1-2 下にスクロールさせると Generate ボタンが表示される**

![アプリ作成後に画面を下へスクロールさせたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/630623a64b2149218b0a619a0ef124e3/fetch-github-apps-token-by-google-apps-script-gen-private-key.png?w=766\&h=360\&auto=compress%2Cformat)

### インストール

これも冒頭の記事の通りです。少し補足すると後の処理で Installation Id が必要になるのでそれも控える必要があります。

具体的にはアプリをインストールした後、ブラウザーに表示されている URL から値を控えます。

▼ **図 1-3 マスクした部分が Installation Id**

![アプリをインストールした後の URL を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0d7c308eacf0478f8fb6b5f4ffe38e35/fetch-github-apps-token-by-google-apps-script-installation-id.png?w=512\&h=152\&auto=compress%2Cformat)

## Google Apps Script で Access Token を取得する方法

冒頭記事で紹介されている Action が利用している [API](https://github.com/octokit/auth-app.js) や以下の記事を参照すると、「GitHub Apps のプライベートキーで署名したトークン([JWT])」を利用することがわかります。

@[card](https://dev.classmethod.jp/articles/register-github-app-and-get-access-token/)

よって、今回のことを実現するには Google Apps Script で [JWT] を扱えればよいことになります。

### Google Apps Script から JWT

Google Apps Script で [JWT] を利用する方法を検索してみると現時点では ZOOM API で利用する記事がヒットしやすいです。

@[card](https://qiita.com/coticoticotty/items/56ff2a3324fe408b06ca)
@[card](https://オンライン将棋教室・香.com/instructor-blog/210201how-to-use-zoom-api-with-google-apps-script/)

しかし ZOOM API では署名のアルゴリズムに `HS256` を利用していますが、GitHub の API では `RS256` を利用するため要件に少し合いません。

今度は `RS256` を条件に加えて検索したところ LINE WORKS の API を利用する方法がヒットしました。

@[card](https://qiita.com/kunihiros/items/a94221ad7c9f4de84cf8)

前述の記事と比較すると署名の関数に [`Utilities.computeRsaSha256Signature()`] を利用していました、これを使うことで対応できそうです。

### プライベートキーの変換

上記の関数を使うことで一件落着になりそうですが実際にやってみると `Exception: Invalid argument: key` エラーになります。

これは GitHub Apps からダウンロード(Generate)したファイルのフォーマットが関数の利用するものと合致していないためです。

▼ **図 2-1 関数が想定しているフォーマット**

```js
var signature = Utilities.computeRsaSha256Signature("this is my input",
    "-----BEGIN PRIVATE KEY-----\nprivatekeyhere\n-----END PRIVATE KEY-----\n");
```

▼ **図 2-2 GitHub Apps からダウンロードしたファイル**

    -----BEGIN RSA PRIVATE KEY-----
    ...
    -----END RSA PRIVATE KEY-----

これは以下を参考に `openssl` で変換します 。

@[card](https://stackoverflow.com/questions/69133483/generate-json-web-token-rs256-to-access-docusign-using-google-apps-script/69133623#69133623)

▼ **リスト 2-1 プライベートキーを変換**

```shell-session
$ openssl pkcs8 -topk8 -inform pem -in xxxxx-private-key.pem -outform pem -nocrypt -out new-private.pem
```

### とりあえず実装して Access Token を取得

以下が試しに実装したコードです[^apiurl]。

[^apiurl]: API の PATH が `/installations` ではじまる記事を複数みかけましたが、(どこかの時点で変更があったのか)現在では `/app/installations` を使うようです。 `/installations` を使うと `Not Found` になります。 <https://docs.github.com/ja/rest/reference/apps#create-an-installation-access-token-for-an-app>

:::details (クリックで表示)

```js
/**
 * jwt の要素(?)用にエンコードする.
 * @param {string | any[]} s - ソース.
 */
function encodeItem_(s) {
  if (Array.isArray(s)) {
    return Utilities.base64Encode(s)
  }
  return Utilities.base64Encode(JSON.stringify(s))
}

/**
 * リクエスト用の jwt を生成.
 * @param {string} appId - App Id
 * @param {string} key - private key(RSA key は扱えない)
 */
function jwtToken_(appId, key) {
  const payload = {
    exp: Math.floor(Date.now() / 1000) + 60,  // JWT expiration time
    // ちょっとだけ時間を手前にしておくとアクセストークンの発行に失敗し辛いらしい。
    // https://qiita.com/icoxfog417/items/fe411b94b8e7ae229e3e#github-apps%E3%81%AE%E8%AA%8D%E8%A8%BC
    iat: Math.floor(Date.now() / 1000) - 10,       // Issued at time 
    iss: appId
  }
  const header = {
    'alg': 'RS256',
    'typ': 'JWT'
  };

  const src = `${encodeItem_(header)}.${encodeItem_(payload)}`
  const signed = Utilities.computeRsaSha256Signature(src, key)
  return `${src}.${encodeItem_(signed)}`
}

/**
 * Application Token を取得する.
 * @param {string} appId - App Id
 * @param {string} installationId - Installation Id
 * @param {string} key - private key(RSA key は扱えない)
 */
function ghToken(appId, installationId, key) {
  const t = jwtToken_(appId, key)
  try {
    const api_url = 'https://api.github.com'
    const path = `/app/installations/${installationId}/access_tokens`
    const res = UrlFetchApp.fetch(`${api_url}${path}`, {
      method: 'post',
      "headers": {
        Authorization: `Bearer ${t}`,
        Accept: "application/vnd.github.machine-man-preview+json"
      }, muteHttpExceptions: true
    })
    return res.getContentText()
  } catch (e) {
    console.error(e)
  }
}

function main() {
  const props = PropertiesService.getScriptProperties()
  const appId = props.getProperty('appId')
  const installationId = props.getProperty('installationId')
  const privateKey = props.getProperty('privateKey')

  const res = JSON.parse(ghToken(appId, installationId, privateKey))
  console.log(res)

  const now = Date.now()
  const e = new Date(res.expires_at)
  console.log((e.getTime() - now) / 1000 / 60)
}
```

:::

スクリプトプロパティーに変換したプライベートキー等をセットし[^property]、実行すると以下のような Access Token を取得できます。

[^property]: セットは `setProperty()` で行っています。クラシックエディターからの設定は動作しませんでした(改行の扱い？)。

```js
{ token: 'xxxxx',
  expires_at: '2022-03-31T13:08:52Z',
  permissions: { metadata: 'read', workflows: 'write' },
  repository_selection: 'selected' }
```

また、本題と少し外れますが、`expires_at` から取得時刻を引いてみると有効期限は 60 分であることが確認できます。

▼ **図 2-3 上記の引いた値を分にしたもの**

    59.9884

## Google Apps Script のライブラリーにする

上記コードはそれほど長くないのですが、切り貼りするのも手間なのでライブラリーにしました。

@[card](https://github.com/hankei6km/gas-github-app-token)

利用方法でとくに変わったところはありませんが、「ライブラリーからは GitHub API を実行せずに、実行用 URL とオプションを生成」します。これを `UrlFetchApp.fetch()` に渡すことで実際に Access Token を取得できます。

```js
  const [url, opts] = GitHubAppToken.generate({
    appId,
    installationId,
    privateKey,
  })

  const res = UrlFetchApp.fetch(url, opts)
  const token = JSON.parse(res).token
```

## 取得した Access Token を利用

取得した Access Token は GitHub API 実行時に`Authorization` ヘッダーへセットすることで利用できます。

ここでは元々の目的であった[ワークフロー開始用の API](https://docs.github.com/ja/rest/reference/actions#create-a-workflow-dispatch-event) で試してみます。

:::details (クリックでコードを表示)

```js
  const props = PropertiesService.getScriptProperties()
  const appId = props.getProperty('appId')
  const installationId = props.getProperty('installationId')
  const privateKey = props.getProperty('privateKey')

  const owner = props.getProperty('owner')
  const repo = props.getProperty('repo')
  const workflowId = props.getProperty('workflowId')
  const ref = props.getProperty('ref')

  const apiBaseUrl = 'https://api.github.com'

  const [url, opts] = GitHubAppToken.generate({
    appId,
    installationId,
    privateKey,
  })

  const res = UrlFetchApp.fetch(url, opts)
  const token = JSON.parse(res).token

  const path = `/repos/${owner}/${repo}/actions/workflows/${workflowId}/dispatches`
  const runRes = UrlFetchApp.fetch(`${apiBaseUrl}${path}`, {
    method: 'post',
    "headers": {
      Authorization: `token ${token}`,
      Accept: "application/vnd.github.v3+json",
    },
    payload: JSON.stringify({ ref }),
  })
  console.log(runRes.getContentText())
```

:::

上記コードを実行するとワークフローが開始されます。このときワークフローは bot(アプリ) が開始したことになります。

▼ **図 4-1 ワークフローの開始理由に bot ラベルが付く**

![Actions タブで実行中のワークフローを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/254bcbde30c647b9a7d406eb8760f26a/fetch-github-apps-token-by-google-apps-script-dispatch2.png?w=599\&h=270\&auto=compress%2Cformat)

## Personal Access Token との比較

GitHub Apps を試した理由は「Personal Access Token で有効期限を短くしていると更新が手間」ということにありました。その点では「必要なときに Access Token を取得できる」ので目的は達成されたと言えます。

しかし、Personal Access Token で有効期限を短くする理由の 1 つには Token の流出対策があります。これついては「Personal Access Token を保存する予定だった場所に期限のないプライベートキーを保存する」ことの是非を少し考えてしまいます。

一方で GitHub Apps を使った場合、以下のようなメリットとデメリットもあります。

*   メリット

    *   利用できるリポジトリを制限できる
    *   Permissions を変更できる(強い権限へ変更するにはインストール先での受け入れが必要)

*   デメリット

    *   すべてのケースで Personal Access Token を置き換えることはできない

たとえば NPM パッケージを GitHub Packages レジストリからインストールするためのトークンとしては使えませんでした[^gh-pkg]。

▼ **図 5-1 npm install で使うとエラーになる**

```shell-session
$ npm install @hankei6km/guratan
npm ERR! code E401
npm ERR! 401 Unauthorized - GET https://npm.pkg.github.com/@hankei6km%2fguratan 
- This credential type is not supported for registry. 
Please use a Personal Access Token or GitHub Actions token instead.
```

よって、GitHub Apps の Access Token を利用するかを決める場合、有効期限の回避だけではなく他の利点なども考慮するのが良いと言えそうです。

[^gh-pkg]: アプリのインストール設定で「All repositories」、Permissions は「Packages に Read and write」を指定しましたが結果は同じでした。

## おわりに

前節で「GitHub Apps の Access Token もそれなりにデメリットがある」的なことを書きましたが、それでも「許可をリポジトリ単位で制御できる」ことは大きな利点になります。

Google Apps Script からワークフローを開始する場合もその利点は変わらないので、今回作ったライブラリーをうまく利用できればと考えています。

[JWT]: https://ja.wikipedia.org/wiki/JSON_Web_Token

[`Utilities.computeRsaSha256Signature()`]: https://developers.google.com/apps-script/reference/utilities/utilities#computeRsaSha256Signature\(String,String\)
