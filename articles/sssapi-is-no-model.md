---
id: sssapi-is-no-model
title: SSSAPI はノーモデルである
emoji: 🧩
type: idea
topics:
  - sssapi
  - api
published: true
---

先日、[SSSAPI] の記事[^article]をみかけて面白そうだったのでスクラップでメモしながら試してみました。

@[card](https://zenn.dev/hankei6km/scraps/d721b82416d6ac)

上記スクラップは少し長くなってしまったので、この記事では特徴的なところをピックアップしてみます。

なお、「ノーモデル」というワードは勢いで使っただけで、そのような技術用語はありません(ないですよね？)。その辺はおおらかな気持ちで流していただければと。

[^article]: <https://zenn.dev/kira_puka/articles/f9496a6a847799>

## SSSAPI は追加スコープ(spreadsheets.\*)を要求しない

もちろんサインイン関連などは必要ですが、このようなサービスで定番の「Google スプレッドシートのすべてのスプレッドシートの参照、編集、作成、削除」などは要求されませんでした。

![Google ドライブへのアクセス許可を求めるスクリーンショットに✖印をつけている状態](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9991f0f9b1924f01a979718e70bdaefd/sssapi-is-no-mode-scope.png?border64=MSw1NTAwMDAwMA\&border-radius64=MQ\&txt64=4p2M\&txt-align64=bWlkZGxlLGNlbnRlcg\&txt-size64=NzI)*図 1-1 すべてのスプレッドシートへの権限は要求されない*

ではファイルへのアクセスはどうなっているのかというと「スプレッドシートを [SSSAPI] のシステムアカウントと共有」することで解決していました(図 1-2)。

![スプレッドシートの共有設定画面でSSSAPI のシステムアカウントを追加しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/510fc2244aff417c815556335c289052/sssapi-is-no-mode-grant-access.png?w=600\&h=432\&auto=compress%2Cformat)*図 1-2 共有設定に [SSSAPI] のシステムアカウントを追加*

スプレッドシート毎に設定するのも少し面倒ですが[^folder]、ドライブ全体ではなくファイル単位で共有を設定できるので気分的な安心感はあります。

[^folder]: 試した限りではフォルダーの共有でも動作しましたがドキュメントには記載がないのでやって良いのかは不明です。

## SSSAPI はステートフルである

「Google スプレッドシートの API 化」という一文を見たとき、データ要求時に毎回スプレッドシートを読みに行く(ステートレスな)サーバーレスファンクション的なものを想像しました(図 2-1)。

```mermaid
graph LR
myapp[[MyApp]] -->|fetch| sssapi[[SSSAPI]]
sssapi -->|fetch| sheet[(SpreadSheet)]
```

▲ *図 2-1 サーバーレスファンクション的なフロー*

実際の挙動としては API を(手動または自動更新で)ビルドすると「その時点のスプレッドシートを読み込み API のステートにする」という構成でした(図 2-2、図 2-3)[^durabble]。

![SSSAPI のコンソールで API の情報を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/be0a76407faf4277902dcb71df3c28dd/sssapi-is-no-mode-update-manyually.png?w=600\&h=238\&auto=compress%2Cformat)*図 2-2 コンソールから「Update」ボタンで API をビルド*

```mermaid
sequenceDiagram
  participant MyApp
  participant SSSAPI
  participant SpreadSheet
  note right of SSSAPI: Update API
  SSSAPI ->> SpreadSheet: fetch
  SpreadSheet ->> SSSAPI: [{"id":1, "item": "data1"}]
  note right of MyApp: Call API
  MyApp ->> SSSAPI: fetch
  SSSAPI ->> MyApp: [{"id":1, "item": "data1"}]
  MyApp ->> SSSAPI: fetch
  SSSAPI ->> MyApp: [{"id":1, "item": "data1"}]
  note right of SSSAPI: Update API
  SSSAPI ->> SpreadSheet: fetch
  SpreadSheet ->> SSSAPI: [{"id":1, "item": "DATA2"}]
  note right of MyApp: Call API
  MyApp ->> SSSAPI: fetch
  SSSAPI ->> MyApp: [{"id":1, "item": "DATA2"}]
```

▲ *図 2-3 ステートの遷移状態(手動)*

自動更新にする場合はコンソールから API Option を変更します(図 2-4)。これでスプレッドシートを更新すると [SSSAPI] 側に通知が届きビルドされるようになります(図 2-5)([連続して更新すると数分待つこともある](https://zenn.dev/link/comments/b7b89ab024221b))。

![SSSAPI のコンソールで API の自動更新を ON にしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8f3a22b5bca74f40b7f2d8da6a55958c/sssapi-is-no-mode-auto-update-opts.png?w=600\&h=342\&auto=compress%2Cformat)*図 2-4 設定により自動更新も可能*

```mermaid
sequenceDiagram
  participant MyApp
  participant SSSAPI
  participant SpreadSheet
  note left of SpreadSheet: Edit Sheet
  SpreadSheet ->> SSSAPI: nofity
  SSSAPI ->> SpreadSheet: fetch
  SpreadSheet ->> SSSAPI: [{"id":1, "item": "data3"}]
  note right of MyApp: Call API
  MyApp ->> SSSAPI: fetch
  SSSAPI ->> MyApp: [{"id":1, "item": "data3"}]
  MyApp ->> SSSAPI: fetch
  SSSAPI ->> MyApp: [{"id":1, "item": "data3"}]
  note left of SpreadSheet: Edit Sheet
  SpreadSheet ->> SSSAPI: nofity
  SSSAPI ->> SpreadSheet: fetch
  SpreadSheet ->> SSSAPI: [{"id":1, "item": "DATA4"}]
  note right of MyApp: Call API
  MyApp ->> SSSAPI: fetch
  SSSAPI ->> MyApp: [{"id":1, "item": "DATA4"}]
```

▲ *図 2-5 ステートの遷移状態(自動)*

この構成は「スプレッドシートの更新にあわせて他サービスの処理を開始したい」ような場合、[SSSAPI] のビルド状況がわかりにくいのは難点ですが、パフォーマンスや耐障害性では有利だと予想できます[^csr]。

また、完了タイミングが読めない複数の更新を 1 つのレスポンスにしたい(いわゆるファンイン的なフロー)の場合、API を任意のタイミングでビルドできるのは使い勝手が良いと思われます[^trigger]。

![複数の担当者が入力するフォーマットですべての項目が入力されているスプレッドシートのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9a19fdb1a870491b94fd064577cf8ee1/sssapi-is-no-mode-fan-in-before.png?auto=compress%2Cformat\&h64=MTMw\&rect64=MCw2LDM4MCwxMzA\&w64=Mzgw)*図 2-6 複数の担当者が入力するシート*

```json
[
  {
    "担当者": "田中",
    "入力時刻": "2022-01-13 10:00",
    "状況": "異常無し"
  },
  {
    "担当者": "佐藤",
    "入力時刻": "2022-01-13 11:00",
    "状況": "設備 A にノイズ"
  },
  {
    "担当者": "林",
    "入力時刻": "2022-01-13 15:00",
    "状況": "異常無し"
  }
]
```

▲ *図 2-7 API レスポンス*

![スプレッドシートに未入力の項目があるスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/716c5c122b8e4c80898497f27e5b7049/sssapi-is-no-mode-fan-in-waiting.png?auto=compress%2Cformat\&h64=MTMw\&rect64=MCw4LDM4MCwxMzA\&w64=Mzgw)*図 2-8 新規に入力を開始する(API は更新待ち)*

```json
[
  {
    "担当者": "田中",
    "入力時刻": "2022-01-13 10:00",
    "状況": "異常無し"
  },
  {
    "担当者": "佐藤",
    "入力時刻": "2022-01-13 11:00",
    "状況": "設備 A にノイズ"
  },
  {
    "担当者": "林",
    "入力時刻": "2022-01-13 15:00",
    "状況": "異常無し"
  }
]
```

▲ *図 2-9 API レスポンスは前日の状態を維持*

![スプレッドシートの全ての項目が入力されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/bcef88657acc4d3fa0a74e9e8c7d3e0f/sssapi-is-no-mode-fan-in-done.png?auto=compress%2Cformat\&h64=MTMw\&rect64=MCwxMSwzODAsMTMw\&w64=Mzgw)*図 2-10 入力完了したので API を更新*

```json
[
  {
    "担当者": "田中",
    "入力時刻": "2022-01-14 10:30",
    "状況": "異常無し"
  },
  {
    "担当者": "佐藤",
    "入力時刻": "2022-01-14 13:00",
    "状況": "異常無し"
  },
  {
    "担当者": "林",
    "入力時刻": "2022-01-14 11:00",
    "状況": "異常無し"
  }
]
```

▲ *図 2-11 API レスポンスが切り替わる*

この他に(利用プランにより件数が変わりますが) Build log には以前のステート(API のレスポンス)が残っていることから履歴を確認できます(図 2-12、図 2-13)。

![Build Log で複数の項目が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/da21180bab1448a0baff7f27e01eaee6/sssapi-is-no-mode-build-log.png?w=600\&h=141\&auto=compress%2Cformat)*図 2-12 ビルドのログを確認できる*

![Build Log から以前の API のステートを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d4302887f87445e09586d45fb397f40e/sssapi-is-no-mode-build-log-preview.png?w=600\&h=401\&auto=compress%2Cformat)*図 2-13 ステートを表示*

[^durabble]: この構成を知ったときに Azure の [Durable Functions](https://docs.microsoft.com/ja-jp/azure/azure-functions/durable/durable-functions-overview) が思い浮かびました(なんとなく似ているかなと)。

[^csr]: おそらくですが、[SSSAPI] は動的な更新に軸足を置いているように思われるので(どこかの時点更新が完了していればよいので)、速度以外にその辺も関係しているのかなと。

[^trigger]: API ビルドを起動する Webhook 的なものが欲しくなってきます。

## SSSAPI はノーモデルである

[各種 Headless CMS を試したとき](https://zenn.dev/hankei6km/articles/test-collage-cms-content)に「どのサービスでも最初にモデルを定義する必要」がありました。

ノーコードツールなどでも言えるようですが「モデルを定義することは難易度が高い」ので、最初にこれがあるのはハードルを高くする要因なのかなと感じていました。

または「管理画面の構成はどうしてもモデルの定義に引っ張られる」部分が多く「編集者用の管理画面を作る管理画面があればいいのに」とも考えていました[^builder]。

そのような中で[SSSAPI 利用事例](https://sssapi.app/articles/20211020_usacese_news)のスクリーンショット(図 3-1)を見たとき、「一目で意図が把握できる」説得力がありちょっとした衝撃を受けました。

![スプレッドシートでコンテンツを入力する記事のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/241a7d4df50344eb93c863bdf865c40b/sssapi-is-no-model-overview.png?auto=compress%2Cenhance\&border64=MSw1NTAwMDAwMA\&h64=MzU2\&rect64=MCwwLDgxMCw0ODA\&w64=NjAw)*図 3-1 「スプシとSSSAPIでお知らせや更新情報をサクッと作成する」より引用*

このスクリーンショットから「[SSSAPI] ではデータの入力が先にあり、モデルはデータにより形作られていく[^card-sheet]」ことが理解出来ます。また「管理画面で苦労するならスプレッドシートを使う」という思い切りの良さも感じます。

このように書くと「おおざっぱな(フラットな)レイアウトしかできないのでは」と思われそうですが、実際に試してみると管理画面の構造に制約されない柔軟なデータ入力が可能でした。

まず、[SSSAPI] では 1 レコード内に[構造化された(配列なども含む)データ](https://sssapi.app/help/api_option/#%E3%83%89%E3%83%83%E3%83%88%E5%8C%BA%E5%88%87%E3%82%8A%E3%81%AE%E3%83%95%E3%82%A3%E3%83%BC%E3%83%AB%E3%83%89%E3%82%92%E3%83%8D%E3%82%B9%E3%83%88%E3%81%AB%E5%A4%89%E6%8F%9B)の入力が可能です(図 3-2、図 3-3)。

![スプレッドシートに構造化されたフィールドを入力しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/27beefa0753d44e48aab382c8364e165/sssapi-is-no-model-structured-field.png?auto=compress%2Cformat\&h64=ODM\&rect64=MCwxNSw4MDAsMTEw\&w64=NjAw)*図 3-2 構造化されたフィールド*

```json
[
  {
    "category": "フルーツ",
    "item": [
      {
        "code": "F001",
        "name": "りんご"
      },
      {
        "code": "F002",
        "name": "みかん"
      },
      {
        "code": "F003",
        "name": "バナナ"
      }
    ]
  },
  {
    "category": "ドリンク",
    "item": [
      {
        "code": "D001",
        "name": "コーヒー"
      },
      {
        "code": "D003",
        "name": "紅茶"
      }
    ]
  }
]
```

▲ *図 3-3 API レスポンス*

また、フィールド(セル)に式や関数などを指定し、結果を通常の値のように利用もできます。以下は計算結果でソートしている例です(図 3-4、図 3-5 )。

![フィールドに計算式が設定されている状態のスプレッドシートが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/739243b5278144a4a6728920a39a7171/sssapi-is-no-model-computed-field.png?auto=compress%2Cformat\&h64=MTcw\&rect64=MCwzLDQ4MCwxNzA\&w64=NDgw)*図 3-4 金額フィールドは計算結果がセットされている*

```json
[
  {
    "品名コード": "E230",
    "個数": 3,
    "単価": 130,
    "金額": 390
  },
  {
    "品名コード": "A089",
    "個数": 5,
    "単価": 90,
    "金額": 450
  },
  {
    "品名コード": "S001",
    "個数": 10,
    "単価": 80,
    "金額": 800
  },
  {
    "品名コード": "R001",
    "個数": 6,
    "単価": 230,
    "金額": 1380
  }
]
```

▲ *図 3-5 金額フィールドでソートされているレスポンス*

そしてスプレッドシートがそのまま API になるということは「レコードを手動で並べ替える」ことも可能になっています [^oreder-manually]。

![スプレッドシートの行が選択されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/947b1b9ac8eb4c91b6c72854b13b010b/sssapi-is-no-model-order0.png?auto=compress%2Cformat\&h64=MTUw\&rect64=MCw0LDQ4MCwxNTA\&w64=NDgw)*図 3-6 レコード移動前*

![選択された行を移動した後のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9c0fbbeed4c04d478d20d6c0d5c3a0c0/sssapi-is-no-model-order1.png?auto=compress%2Cformat\&h64=MTUw\&rect64=MCwxMiw0ODAsMTUw\&w64=NDgw)*図 3-7 レコード移動後*

```json
[
  {
    "品名コード": "A089",
    "個数": 5,
    "単価": 90,
    "金額": 450
  },
  {
    "品名コード": "S001",
    "個数": 10,
    "単価": 80,
    "金額": 800
  },
  {
    "品名コード": "E230",
    "個数": 3,
    "単価": 130,
    "金額": 390
  },
  {
    "品名コード": "R001",
    "個数": 6,
    "単価": 230,
    "金額": 1380
  }
]
```

▲ *図 3-8 API レスポンス*

[^builder]: それこそ AppSheet などのノーコードツールを使えよという話ではありますが。

[^card-sheet]: この辺はカード型データベースとスプレッドシートの違いのような感覚になるのかなと。

[^oreder-manually]: レコードの一覧画面から並べ替えができる Headless CMS は意外と少ないです、[試した中では](https://zenn.dev/hankei6km/articles/test-collage-cms-content) microCMS だけでした。他では複数のモデルを参照関係にするなどの対応が必要でした。

    参考: [Use Reference Fields to Manually Order Content With Contentful and Gatsby](https://betterprogramming.pub/use-reference-fields-to-manually-order-content-with-contentful-and-gatsby-e5808861d875)

## おわりに

[SSSAPI] を試してみてみました。

感想としてはまず「お手軽で使いやすい」がありますが、その他にも「アクセス許可の方針」や「API をビルドするまでステートが保持される」といった予想外(予想以上)の部分も多く、奥が深いとも感じました。

モデルについていえば、ある程度の規模になれば事前にしっかりとしたモデルの定義は必要ですし、ステージングやスケジュールなどを管理する画面も必要になってきます。

それでもスプレッドシートで機能を代替することにより、多くの人が直観的にデータ入力を行えるのは [SSSAPI] の大きな特徴といえるのでないでしょうか。

なお、今回は特徴的な部分を記述しましたが [AppSheet との連携なども試してみた](https://zenn.dev/link/comments/b1eee62ec4f8a4)ので、その辺もいずれ記事にできればと考えています。

[SSSAPI]: https://sssapi.app/
