---
id: notion-databases-to-google-docs-for-gemini
title: Gemini「Notion も Markdown も Google ドキュメントに全部ぶちこめ」
emoji: 🤞
type: idea
topics:
  - gemini
  - notion
  - googledocs
  - notebooklm
  - zenn
published: true
---

[Notion](https://www.notion.so/ja-jp) に保存しているメモを Gemini の[セマンティック取得](https://ai.google.dev/gemini-api/docs/semantic_retrieval?hl=en) で扱えないか試していました。しかし、安全な(取得結果からインジェクションされない)コードを作る方法がいまいちわかりません。そんなとき「メモを Google ドキュメントに変換したら Gemini から簡単に扱えるのでは」と思いつきました。

少し試してみたところ「とくに分類もせず雑多に保存していたメモ」を活用しやすくなったので、同じようにメモをとっている方の参考になればと思い記事にしてみます。

## 想定している読者

*   気になることなどがあれば「とにかくメモしておく」という人
*   メモを AI チャットボットで活用したいけど「RAG とかセマンティック取得とかはちょっと」という人
*   ChatGPT や Claude よりも Gemini を選んでしまうへそ曲がりな人たち

なお、Gemini のチャットアプリなどを使いますが、知識としては基本的な操作の範囲で収まります。また、生成 AI 用のコードは扱わないので **Gemini の API キーは使いません**。

ただし、メモを Google ドキュメントへ変換するので、利用しているサービスやツールから Google ドキュメントへ変換する知識(コマンドやスクリプトなど)は必要となります。

## Google ドキュメントに全部ぶちこめとは？

簡単に言えば「ためこんだメモを Google ドキュメントのファイルとして保存しまくる」という話です。

今回は Notion を Goole Apps Script で Google ドキュメントへ変換する方法を後述しますが、他のサービスやツールであっても変換方法は調べればだいたいは出てくると思います。

たとえば、[QFixHowm](https://sites.google.com/site/fudist/Home/qfixhowm) で Markdown 形式のメモをとってる場合であれば以下のようになります。

1.  `.md` を HTML へ変換

    *   Gemini が理解できればよいだけなので見栄えの考慮は不要
    *   ただし、ローカル画像や howm 独自の要素(TODO など)を使っている場合は少し注意が必要

2.  HTML を Google ドキュメントとしてアップロード

    *   アップロード先はのフォルダーどこでもよい(分類など不要)

これらを実現する CLI ツールは検索するといろいろヒットするので、スクリプトを作れば全部ぶちこめることになります。

## 全部ぶちこんだときの利点とは？

[Gemini アプリ](https://gemini.google.com/)の[拡張機能](https://support.google.com/gemini/answer/13695044?hl=ja\&co=GENIE.Platform%3DDesktop)と [NotebookLM](https://notebooklm.google/) からメモを簡単に扱えるようになります。具体的な利用方法などは後述しますが、たとえば以下のようなことができます。

*   Gemini のチャットアプリからメモの検索と要約

    *   「夏もののシャツを買ったときのメモファイルをさがして」みたいな自然言語による検索
    *   スマートフォントのアシスタントで音声を使ったメモの検索など

*   NotebookLM を使ってメモの分析

    *   追記を繰り返して大きくなりすぎたメモの要約
    *   記事の下書きがガイドラインに沿っているかの構成チェックなど

ここで「NotebookLM なら Markdown でも同じようなことできるのでは」となりますが、Gemini 系のアプリでは Google ドキュメントの方が扱いやすいようになっていました。 (下書きの構成チェックは Google ドキュメント用の機能を使っています)

とくに理由がなければ変換しておくといいことあります、たぶん。

## 外部の情報を Google ドキュメントのファイルにする(Notion データベースの場合)

ここからは具体的手順などを記述していきます。

::: message

今回は Notion のメモを例にしていますが、前述のように他のサービスなどでも Google ドキュメントのファイルとして保存できれば Gemini アプリから利用できます。

Notion を利用していない場合は「メモを Google ドキュメントとして保存してある」と仮定し、このセクションは読み飛ばしてください。

また、Notion を利用している場合でも「もっと良い変換手順があるぜ」ということであれば、やはり読み飛ばして問題ないです。

:::

Notion のデータベースを Google ドキュメントへ変換する方法はいろいろありますが、今回は [Google Apps スクリプト](https://www.google.com/script/start/)でスクリプトを作りました。

*   データベースのページを取得するツールで変更のあったページを取得する

*   [ライブラリー](https://github.com/hankei6km/gas-notion2content)の機能でページを HTML へ変換する

    *   今回は Gemini に認識させるだけなので見栄えは考慮していません

*   [Drive サービス](https://developers.google.com/apps-script/advanced/drive?hl=ja)を使って HTML を Google ドキュメントとして保存する

    *   このとき、ファイル名に Notion 側のページ id を含めることでページとファイルを関連付けます

::: details 保存するスクリプト。

```js
async function myFunction() {
  const props = PropertiesService.getScriptProperties()
  const apiKey = props.getProperty('NOTION2CONTENT_API_KEY')
  const database_id = props.getProperty('NOTION2CONTENT_DATABASE_ID')
  const folderId = props.getProperty('FOLDER_ID')

  const now = Date.now()
  //const before = 3600000 * 1 * 24 * 89
  const after = 3600000 * 2 // 1時間舞に実行するのだが、エラーが発生した後にリカバリーできるように 2 時間にする(直近で編集したメモは複数回対象になる)

  //const extBefore = (now - (before + (60000 * 5)))
  const extAfter = (now - (after + (60000 * 5))) // スケジュール実行のズレを考慮し 5 分追加)
  const filter = {
    and: [
      {
        timestamp: "last_edited_time",
        last_edited_time: {
          // before: new Date(extBefore).toISOString(),
          after: new Date(extAfter).toISOString(),
        }
      }, {
        property: 'タグ',
        multi_select: {
          does_not_contain: '!export'
        }
      }
    ]
  }

  const i = Notion2content.toContent({ auth: apiKey }, {
    target: ['props', 'content'],
    query: {
      database_id: database_id,
      filter,
      sorts: [
        {
          timestamp: "last_edited_time",
          direction: 'ascending'
        }
      ],
      page_size: 100,

    },
    workersNum: 1,
    //skip: 80 + 76 +85 + 37,
    limit: 300,
    toItemsOpts: {},
    toHastOpts: {}
  })

  let cnt = 0
  for await (const c of i) {
    // console.log(JSON.stringify(c.props, null, 2))
    const fileName = `${c.props['名前']} - ${c.id}`
    console.log(cnt++, fileName)
    //console.log(fileName)
    //const body = `${await Notion2content.toFrontmatterString(c)}${await Notion2content.toMarkdownString(c)}`
    //const mediaData = Utilities.newBlob("").setDataFromString(body, "UTF-8").setContentType("text/plain")
    const body = ` <pre><code>${htmlEscape(await Notion2content.toFrontmatterString(c))}</code></pre>${await Notion2content.toHtmlString(c)}`
    const mediaData = Utilities.newBlob("").setDataFromString(body, "UTF-8").setContentType("text/html")
    const existFileId = getExitFileId(folderId, c.id)
    if (existFileId) {
      console.log('- update')
      const resource = {
        name: fileName,
        //parents: [folderId],
        //mimeType: 'application/vnd.google-apps.document'
      }
      const res = Drive.Files.update(resource, existFileId, mediaData)
    } else {
      // https://stackoverflow.com/questions/77752561/how-to-convert-docx-files-to-google-docs-with-apps-script-2024-drive-api-v3
      const resource = {
        name: fileName,
        parents: [folderId],
        mimeType: 'application/vnd.google-apps.document'
      }
      const res = Drive.Files.create(resource, mediaData)
    }
    console.log('---')
  }
}

function getExitFileId(folderId, idInNotion) {
  const f = Drive.Files.list({ q: `name contains '${idInNotion}' and '${folderId}' in parents and trashed=false` })
  if (f.files.length > 0) {
    return f.files[0].id
  }
  return ''
}

const removeRegExp = new RegExp(/-/g)
function getLinkToOriginal(id) {
  return `https://www.notion.so/${id.replace(removeRegExp, '')}`
  //return `[link to original](https://www.notion.so/${id.replace(removeRegExp,'')})`
}

function htmlEscape(str) {
  return str.replace(/&/g, '&')
    .replace(/</g, '<')
    .replace(/>/g, '>')
    .replace(/"/g, '"')
    .replace(/'/g, '&#39;');
}
```

:::

スクリプトを作ったら、あとはデータベースの必要なページを保存し、GAS のトリガーで定期実行しているだけです。 (GAS の場合、実行時間に制限があるので最初の保存は分割したりでちょっと大変でした)

ただし、かなり雑な作りなので(ページを削除しても反映されないなど)、まじめに使うならもう少し作り込んだ方がよいかなという感じです。

以下、ページを変換したときのサンプルです。

**図 4-1 これが**

![Notion でリストや表などを含むページを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6ac5706d007e4c5aa34f704caae5a60a/notion-databases-to-google-docs-for-gemini-samp-org.png?w=959\&h=756\&auto=compress%2Cformat)

**図 4-2 こうなる(メンションや目次など対応していない項目もあります)**

![エクスポートされたページを Google ドキュメントで表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8ad8374830304bb6b4a12a3a59b9d442/notion-databases-to-google-docs-for-gemini-samp-export.png?w=892\&h=625\&auto=compress%2Cformat)

## 使ってみる - 拡張機能の場合

Google ドキュメント形式で保存してしまえば、あとは普通に拡張機能から使えます。

基本的には、Gemini の拡張機能を有効にした後でプロンプトを以下のようにすると、ファイルの内容を参照した結果が返ってきます。

*   「@ Google ドキュメント」を明示的に指定する
*   ファイル(メモ)をさがすように指示する

### 普通の検索

まずは Gemini アプリのプロンプトで検索するとどんな感じかなというのを試してみます。

**図 5-1 Google スプレッドシートを csv でダウンロードする方法のメモファイルをさがして。その中からダウンロード用の url についての記述を説明して**

![一見関係ないメモからダウンロード用 URL の情報を取得し、各種要素について説明されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/95ce9d5df2aa4db296bf39d38cd56ea8/notion-databases-to-google-docs-for-gemini-csv-url.png?w=841\&h=618\&auto=compress%2Cformat)

一見すると無関係なメモから必要とする情報を引き出すことができました。メモをヒットさせるところまでは全文検索を駆使すると対応できそうですが、要約として必要な部分だけ表示されるのは便利だったりします。

また、URL の要素について説明されていますが、これは Gemini が補足したものです。実は「URL の説明メモも作ってある」と思い込んでプロンプトを入力していたので、適度に補足されるのもありがたいところです。 (「存在しない記憶」を植え付けられた気もしますが、深く考えないことにします)

ただし、生成 AI のお約束で Gemini の要約(補足)はわりと正しくないこともあるので注意は必要です。また、ヒットするはずのメモが無視されることもあったので、この辺は性能向上に期待といったころでしょうか。

### 画像を使う

いまどきのチャットボットはマルチモーダルなので画像を使って検索してみます。

**図 5-2 画像の商品を調べてください。関連するメモファイルをさがしてください**

![画像に関連するドキュメントが 2 つ見つかっているスクリーンショット。1 つ目のドキュメントの要約が表示されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3388ef1e935f45339ebff33079711f80/notion-databases-to-google-docs-for-gemini-image.png?w=800\&h=720\&auto=compress%2Cformat)

一発で目的のメモを探すのは難しいので、まず画像を調べてから関連するメモを探す感じになります。ただし、上記の例はわりと上手くいっている方なので、普段使いするにはちょっと厳しいかなという感じです。

::: message

ドキュメント内の画像について。

少し試してみた感じでは、ドキュメント内の画像は認識していないようでした。たとえば、「西口バスのりばの標識が写った写真を含むメモをさがしてください」はうまくいきませんでした。

:::

### アシスタント的な使い方

Gemini には[スマートフォンのアシスタントを置き換える設定があり](https://support.google.com/gemini/answer/14554984?hl=ja\&co=GENIE.Platform%3DAndroid)、簡単な操作で Gemini を呼び出せます。

私は[フィード・リーダー](https://github.com/hankei6km/gas-feed2notion)の内容も Google ドキュメントへ変換しているので「要約をアシスタント(ルーティン)から確認できないかな」と試したのですが、起動方法などによって使える機能が異なっていました。その辺についてなど。

まず、スマートフォンの電源長押しで起動した場合です。これは音声入力でも拡張機能を利用でき、回答もそのまま音声で再生されました。最初の入力に対する回答で Gemini アプリへ切り替わると拡張機能も通常通りに使えるようです。

スマートフォンを取りだす手間や回答までのレスポンスはあまり良くないですが、簡単な検索なら音声でもわりと行ける感じです。たとえば、ドラッグストアで「いま目の前でワゴンセールになっている詰め替え用シャンプーっていつものやつだっけ？」的なときに「シャンプーを買った時のメモ探して」と音声で検索できるのは便利だったりします。 (視線が気になるお年頃なのでちょっと恥ずかしいですけど)

**図 5-3 音声入力でメモを探す**

![スマートフォンの Gemini アプリで音声入力によりメモを検索しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/af7fde5b8820401395f274b31c221068/notion-databases-to-google-docs-for-gemini-voice-search.png?w=864\&h=1226\&auto=compress%2Cformat)

ただし、アシスタントとして利用する場合に把握しておいてほしい実世界の情報はうまく扱えないようです。

たとえば、[位置情報を使えるはず](https://support.google.com/gemini/answer/14554984?hl=ja\&co=GENIE.Platform%3DAndroid)なのですが、一見すると位置情報を取得できているようで、正しくない情報を取得することが多かったです。質問の仕方によっては取得できないこともありました。

**図 5-4 位置情報を取得できない状態**

![現在地がわからないという回答になっているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d0b3ef29fba540959b3bf7625de21fea/notion-databases-to-google-docs-for-gemini-not-access.png?w=864\&h=1108\&auto=compress%2Cformat)

[質問の仕方を工夫する](https://support.google.com/gemini/answer/13594961?sjid=5058994093555957083-AP#location_info)と改善することもありますが、どのような質問の仕方がよいのかは「やってみないとわからない」状況です。他にも現在時刻の把握も結構怪しいので、実世界の情報を組み合わせるのは少し難しいと言えます。 (課金して Advanced にするとまた違うのかもしれませんが、試していないので不明です)

**図 5-5 頑張ればどうにかなるときもある**

![現在地から駅名を調べてから、その駅名をもとにファイルを検索しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/49911eec4ca94272a051a48b7532ea18/notion-databases-to-google-docs-for-gemini-location.png?w=864\&h=1408\&auto=compress%2Cformat)

一方で、ヘッドセットから「OK グーグル」した場合、位置情報を扱えるのですが拡張機能については考慮されなかったり「理解できませんでした」となってしまいました。これはルーティンのカスタムアクション(コマンド)でも同じです。おそらくですが、ヘッドセットからの「OK グーグル」は従来のアシスタント的な処理になり、拡張機能が使えないのだと思われます。

よって「要約をアシスタント(ルーティン)から確認する」は少し難しいようです。代替案として、ニュースファイルを要約するプロンプトをアクティビティとして保存しておき再利用してみましたが、ちょっとめんどうな感じです。 (そんな感じなのでルーティンには対応してほしいところですが、この辺は Gems へ吸収されるのかなという気もしています)

**図 5-6 保存しておいたアクティビティを表示しスピーカーアイコンのタップで読み上げが始まる**

![スマートフォンの Gemini でニュースを要約してあるアクティビティを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/beb6eb0b4ecd418c9abcb18e5e2150d6/notion-databases-to-google-docs-for-gemini-in-mobile.png?w=864\&h=1000\&auto=compress%2Cformat)

::: message

アシスタントとはちょっと違いますが、プロンプト入力のバリエーションとしては Chrome のオムニボックスから `@gemini` を通して使うこともできました。

![Chromeのオムニー(アドレスバー)に「@gemini コミットのハッシュからprを探す方法のメモをさがして」と入力しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6a5c58743f754a63a3a911c2b3d11542/notion-databases-to-google-docs-for-gemini-omini-bar.png?w=741\&h=99\&auto=compress%2Cformat)

Enter するとこうなる。

![Gemini アプリの画面でプロンプトへの返答が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/af528d5a94904caa90f4c7073aa6a802/notion-databases-to-google-docs-for-gemini-omini-bar-res.png?w=928\&h=566\&auto=compress%2Cformat)

ただし、`@gemini` は表示しているページの内容などは参照してくれないので、ページに関連しての検索は無理なようです。

:::

### テンプレートを作る

これは悪意ある情報を実行する可能性があるので考慮点に書くべきか迷ったのですが、今回は利用例として書きます。

以下のようなメモを作ります。

**リスト 5-1 テンプレートの例**

```markdown
プロンプトの回答方法についてのテンプレートです。回答はこのテンプレートにしたがって作成してください。

## 書き出し

回答は「押忍！説明します」から始めてください。

## 文体(トーン)

文体は敬語で回答してください。

## 長さ

できるだけ長文で回答してください。
```

メモが Google ドキュメントとして保存されたら、以下のように操作します。

**図 5-7 メモをさがす**

![Gemini アプリでメモをさがしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/bc68956a918344019e5ff2a4b95cd92a/notion-databases-to-google-docs-for-gemini-search-template.png?w=907\&h=589\&auto=compress%2Cformat)

**図 5-8 見つけたメモを確認し、それに従うようお願いする**

![プロンプトで「おいしいチャーハンの作り方」の説明を求めると、回答の書き出しが「押忍！おいしいチャーハンの作り方を説明します！」になっているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7e30cfb1948448ce969384bc644545af/notion-databases-to-google-docs-for-gemini-run-prompt.png?w=860\&h=565\&auto=compress%2Cformat)

このような感じでテンプレート的なプロンプトをメモとして作成できました。また、今回は単純な例ですが、テンプレートには拡張機能を扱うプロンプトも利用できました。 (「GitHub に関連するメモをさがして知識として利用してください」のようにできる)

::: message alert

このような使い方は「本来ならデータとして扱う情報をコードとして評価した」状態と同じで、参照する項目によっては悪意ある情報を実行する可能性があります(拡張機能はメールを参照するものもある)。たとえば、メールから不正な情報を挿入する実験として以下のような例もあります。

@[card](https://gigazine.net/news/20240304-morris-ii-chatgpt-gemini-malware/)

また、今回の例ではメモを探してから新しくプロンプトを入力していますが、これを 1 つにまとめることもできます。しかし、それをやると、「みつけたテンプレートは悪意ある内容だった」ということに気が付かず実行してしまうかもしれません。テンプレートを使う場合は、そのような点を意識しておく必要があります。

:::

## 使ってみる - NotebookLM の場合

最近、日本語でも利用できるようになった NotebookM での利用について。 (この記事は 2024 年 6 月に書いています)

@[card](https://www.watch.impress.co.jp/docs/news/1597789.html)
@[card](https://forest.watch.impress.co.jp/docs/news/1598767.html)

NotebookLM の使い方はリンク先の記事などが詳しいので。ここでは主に以下の点を見ていきます。

*   拡張機能では扱えなかった大きなサイズのファイルを扱える
*   ドキュメント内の画像が認識される
*   対象とするメモを指定しやすい(テンプレート的な処理が比較的安全)

また、Google ドライブ上のファイルをソースにした場合、元ファイルが変更されると(手動操作になりますが)ソース側へ反映できます。これを応用した文書チェッカーの作り方なども合わせて試してみたいと思います。

### 追記される大きいサイズのメモ

不定期に追記しているメモをノートブックのソースにしている例です(このメモはサイズが大きいからか拡張機能では扱えなかったものです)。

**図 6-1 GitHub で使える少し便利な URL のメモ**

![メモを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/56b843cbd0394b549f49a047e44d9fde/notion-databases-to-google-docs-for-gemini-memo-update.png?w=849\&h=511\&auto=compress%2Cformat)

**図 6-2 上記メモをソースとして作成したノートブック、メモの内容が反映されている**

![ノートブックにメモがソースとして取り込まれたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3a29b3bef3194223bd11d57fde09c5ca/notion-databases-to-google-docs-for-gemini-source-update.png?w=1060\&h=635\&auto=compress%2Cformat)

とくに問題なくノートブックのソースとして追加できました。軽く質問してみます。

**図 6-3 メモの後半に記述されている内容についての質問**

![質問に対して適切な回答が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e5e628c3a06c45cd95e89fb1253824c6/notion-databases-to-google-docs-for-gemini-memo-update-prompt.png?w=1105\&h=475\&auto=compress%2Cformat)

こちらもとくに問題なく質問できました。また、メモを更新した場合(Google ドキュメントの方も更新すれば)、ソースも更新できます。

**図 6-4 メモ(Google ドキュメント)が更新されるとソースを同期するボタンが表示されます**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f58d2ae20a3449539a8c3f2af487f27b/notion-databases-to-google-docs-for-gemini-source-sync.png?auto=compress%2Cformat)

このように、サイズが大きく不定期に更新されるようなメモを扱うことができました。

また、更新を反映しやすいので、記述中の文章をソースにすると構成のチェックにもなります。たとえば、この記事の下書きは Notion で記述しているので Google ドキュメント経由でノートブックを作成し、おかしな要約にならないかチェックしています。 (要約のチェックの他にも、後述する方法でガイドラインに沿っているかもチェックしています)

あとは、定期的に更新される情報をスクリプトで Google ドキュメントに集約しておき、ソースにすると面白いかもしれません。

### 画像を含むメモ

NotebookLM では画像の内容も認識されます。街歩き風のメモで試してみます。

**図 6-5 駅の画像が含まれるメモ**

![メモに 2 枚の駅の画像が含まれている状態を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/450a4cff8a8449db90263843d668110d/notion-databases-to-google-docs-for-gemini-memo-images.png?w=977\&h=667\&auto=compress%2Cformat)

**図 6-6 画像込みのソースとして認識されている**

![ソースのガイドを表示すると画像が含まれている状態のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e5820b64ec6049b8ba85371679dee86c/notion-databases-to-google-docs-for-gemini-source-images.png?w=685\&h=760\&auto=compress%2Cformat)

**図 6-7 画像について質問してみる**

![画像について質問しているスクリーンショット、文字が含まれている写真には正しく返答されるが、入っていない写真については「分からない」という返答になっている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8dc4bc1e2b7c40428ab10fe0b7d03780/notion-databases-to-google-docs-for-gemini-prompt-images.png?w=684\&h=794\&auto=compress%2Cformat)

画像についての質問でも返答されることが確認できました。なお、2 枚めの写真は駅名が標識で隠れるように撮影してあります。妙な推測が入らないのでポイント高いのですが、それでも何度か試すとまれに推測で返答されることもありました。

画像を扱えるのは頼もしいところですが、現状ではあくまでもテキストの補助として使う感じになるかと思います。

::: message

プロンプトでの画像について。

ソースの画像を扱うことができましたが、Gemini アプリのようにプロンプトから画像をアップロードしての質問はできないようです。

:::

::: message

Markdown の画像は認識されなかったです。

試した限りでは `![]()` で外部の画像を参照していても NotebookLM では認識されませんでした。 ([データ URL](https://developer.mozilla.org/ja/docs/Web/HTTP/Basics_of_HTTP/Data_URLs) を使うといけるかもしれませんが、試していないので詳細は不明です)

:::

::: message

OCR にはちょっと厳しそう。

商品パッケージの注意書き画像をソースに含めてみたのですが、思っていた以上に独創的なテキストが出力されました。 (画像周辺の文章に影響されたテキストを出力している感じです)

たとえば、「100 円均一で購入した」という記述があると、それっぽい内容のテキストを作り出す印象でした。

また、「ソースにテキストの情報は記載されていません」のような返答を繰り返してしまうこともありました。

画像の文章をテキスト化するときは、現状ではそれ用のツールを使った方がよさそうです。

:::

### Zenn 記事の下書きチェッカー(ソースをテンプレート的に使う)

NotebookLM では比較的安全にソースをテンプレートとして使えるので、少し実用的に「Zenn 記事の下書きがコミュニティガイドラインに沿っているか」を確認するノートブックを作ってみます。

@[card](https://zenn.dev/guideline)

Zenn 記事の下書きを記述しておき、以下を実施します。

1.  新しいノートブックを作成しソースに `https://zenn.dev/guideline` を指定
2.  下書きも(Google ドキュメント経由で)ソースとして追加
3.  プロンプトに「Zenn のコミュニティガイドラインについて教えて」「ソース”`[ここにソース名]` ”の内容はコミュニティガイドラインに沿っているかチェックしてください」と入力

うまくいくと以下のように表示されます。

**図 6-8 下書きをチェックした結果**

![ガイドラインの項目を満たしていない箇所が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5d4f8efb465641a5ad850eeb5040a5c7/notion-databases-to-google-docs-for-gemini-draft-checker.png?w=1225\&h=599\&auto=compress%2Cformat)

あとは、下書きを更新したらソースを同期してプロンプトでの質問を繰り返します。

これも質問を繰り返すと返答がブレてきますが、大きくガイドラインから外れているところは概ね指摘される感じです。上記スクリーンショットでも指摘されている項目は「だよな」という自覚があった部分です。

ただし、以下の記事のようなかっちりしたチェックは無理そうです。今回、「NotebookML」と何度も記述ミスしたのでノートブックでチェックしてみたのですが、かなり厳しい感じでした。 (この辺はまだまだ [textlint](https://textlint.github.io/) のお世話になりそうです)

@[card](https://www.techno-edge.net/article/2024/03/08/2929.html)

::: message alert

ノートブックであっても共有時のテンプレート的な操作には注意が必要そうです。たとえば、以下のようなメモが記載されているノートブックを共有された場合、不用意に従ってしまうと見た目通りではないソースを実行してしまうことも考えられます。

**図 6-9 見た目通りではないソースを実行させようとするノートブック**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/fdc70441cbdf4acda1383b447a5aee2c/notion-databases-to-google-docs-for-gemini-chou-benri.png?auto=compress%2Cformat)

**図 6-10 不用意に従うとこうなる**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/28909b33afce41b9bb27a4fa8fce277b/notion-databases-to-google-docs-for-gemini-chou-benri-trap.png?w=701\&h=471\&auto=compress%2Cformat)

これはおふざけですが、伏せておきたい情報を含む回答をメモとして固定するように誘導されると少し危ないかなと。

**図 6-11 この回答をメモとしてノートブックに固定すると共有相手にお悩みが見えてしまう**

![回答の中で質問を繰り返したあと、もっともらしいことを言ってメモを固定するように誘導しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4327a0284c794169ab811efb60546737/notion-databases-to-google-docs-for-gemini-lead.png?w=931\&h=298\&auto=compress%2Cformat)

:::

### Notion Web クリッパー

最後に、Notion ならではの使いかたを少しだけ。

NotebookLM では前述のようにウェブサイトを指定できますが、試したかぎりでは取り込む範囲などは指定できませんでした。ある程度はコピペでも対応できますが、Notion Web クリッパーでページを取り込むと Notion 上で編集できます。

@[card](https://www.notion.so/ja-jp/web-clipper)

**図 6-12 前述のコミュニティガイドラインを取り込んで編集**

![取り込んだページを Notion で表示し編集しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8b1a3b40d9624a14b37bf8ddeba3e2f0/notion-databases-to-google-docs-for-gemini-edit-webpage.png?w=902\&h=543\&auto=compress%2Cformat)

編集したものを Google ドキュメント経由でソースに指定することで、コピペの手間を軽減できることになります。他にも、分割されたニュースページを単一ページにしたりなどもできます(対応してないサイトもあり、時々おかしくなりますが)。

取り込むページの利用規約などにもよるので注意は必要ですが、 Web クリッパーを使うことで柔軟な取り込みができるかと思います。

## 考慮点

考慮点については「検索結果からメモを編集したいときは？」「検索範囲を変更する方法は？」などいろいろ出てくるのですが、利用しているツールなどでも変わってくるので、ここでは全般的なことから 1 項目だけ。

### 各サービスでの利用規約

メモを各種サービスで使うということは、メモの内容にはそれぞれの利用規約が適用されます。

たとえば、課金していない Gemini アプリでは、会話内容が人間のレビュアーに参照される場合があり、サービス改善にフィードバックされることが告知されています。

![Gemini アプリ内に表示されている注意書きのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/87d9829571d648e4a6c5170aa2d57077/notion-databases-to-google-docs-for-gemini-message.png?w=852\&h=144\&auto=compress%2Cformat)

@[card](https://support.google.com/gemini/answer/13594961?hl=ja)

拡張機能を使っても Google ドライブの内容は守られるとの情報もありますが、会話の中で参照された内容はその限りではないようにも思えます(実際のところはどのような扱いかは確信を持てていません)。

他にも、今回の場合であれば Google ドライブや NotebookLM などの規約もあるので、メモの扱われ方に問題がある場合は、特定のメモを除外する仕組みなどを検討することになります。

ちなみに、前述のスクリプトではタグプロパティに `!export` が含まれていると除外することになっています。

## おわりに

Notion の情報を Google ドキュメント形式で保存することにより、Gemini から扱えるようにしてみました。

この構成で 1 ヶ月ほど使ってみた感想としては以下のようになります。

*   Gemini アプリ + 拡張機能: 正しくない要約も多いけれど、一見無関係なメモから必要な情報にたどり着くなど利便性の向上も実感できる
*   NotebookLM: 下書きの構成をチェックするときに便利で、今後はフィードリーダーから自動的にソース用ドキュメントを作成するなど応用できればと考えている

各種フレームワークできっちりと構築されたチャットボットとまではいきませんが、雑多なメモを様々な切り口から利用しやすくなったと思います。

問題は、全部ぶちこみすぎると Google の戦略に屈してしまいそうなところでしょうか…。

*   Gemini「Google One AI プレミアムならドライブ容量も増えますよ」
*   Chromebook Plus「いまなら 12 ヶ月お試しできる特典がありますよ(2024 年 6 月の情報)」
*   NotebookLM「いつまでも無料だと思うなよ(2024 年 6 月時点で最終的な料金の詳細は不明です)」
*   motorola razr 50「[折りたたんだままでも Gemini アプリを動かして見せますぜ](https://k-tai.watch.impress.co.jp/docs/news/1603179.html)(2024 年 6 月時点で詳細は不明だが、おそらく拡張機能もいける)」
