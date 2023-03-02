---
id: send-a-repo-list-to-notion-database
title: Notion データベースへ GitHub 上のリポジトリ一覧を送信するツールを作ってみた
emoji: 🗃️
type: tech
topics:
  - notion
  - github
published: true
---

Notion 上にリポジトリ一の覧があると「GitHub Mobile でリポジトを開くのに便利かな」と思い、コマンドラインツール `reposition` を作ってみました。

@[card](https://github.com/hankei6km/reposition)

実際に利用してみると GitHub Mobile との連携以外でもわりと良かったので、その辺と下記のような項目についてなど。

*   GitHub Actions でリポジトリ一覧を取得、スケジュール実行する方法
*   ツールを作るときに利用した環境

## 基本的な利用方法

最初に Notion 側でインテグレーション(API Key)と[データベースを準備](https://github.com/hankei6km/reposition#setup)しておきます。

準備ができたら、GitHub CLI から出力されたリポジトリ一覧の JSON(`.ndjson`)をパイプで渡すと Notion のデータベースへ送信します。フォーマットが合致していれば他の方法で生成したデータでも大丈夫です。

**図 1-1 基本的な実行例**

```shell-session
$ export REPOSITION_API_KEY=<Integration API KEY>
$ export REPOSITION_DATABASE_ID=<Database Id>
$ gh repo list \
  --json nameWithOwner,name,owner,url,description,repositoryTopics,createdAt,updatedAt,pushedAt,openGraphImageUrl \
  --jq .[] | reposition
```

**図 1-2 リポジトリ数が多い場合は `-L` で件数を増やす**

```shell-session
$ gh repo list -L 50 \
  --json nameWithOwner,name,owner,url,description,repositoryTopics,createdAt,updatedAt,pushedAt,openGraphImageUrl \
  --jq .[] | reposition
```

また、`reposition` 側には最近更新したリポジトリのみ送信するオプションがあります。これを利用すると、GitHub Actions などで定期的に更新させる場合に処理時間が短縮されます。

**図 1-3 2 日以内に更新のあったものを対象にする**

```shellsession
$ gh repo list -L 50 \
  --json nameWithOwner,name,owner,url,description,repositoryTopics,createdAt,updatedAt,pushedAt,openGraphImageUrl \
  --jq .[] | reposition --filter-time-range 172800 # repositories updated(pushed) within 2 days.
```

なお、`reposition` はデータベースのプロパティを上書きするだけなので、ページの削除や Notion 側の変更を反映させることは残念ながらできません。

## 使ってみた

ギャラリービューでリンクを表示しておけば、名前でフィルタリングしたりリンクのクリックでリポジトリが表示されます。

**図 2-1 ギャラリービューによるカード形式での表示**

![Notion 上でリポジトの一覧がカード形式で表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d76837fa042d47be8964628958e866a0/send-a-repo-list-to-notion-database-card.png?w=1440\&h=900\&auto=compress%2Cformat)

GitHub Mobile をインストールしてあるデバイスであれば、リンクのタップで GitHub Mobile が表示されます[^intent]。当初は Notion アプリで下図のようにフィルタリングすることが目的でしたが、GitHub Mobile でもフィルタリングできると途中で気が付きました[^home-tab] 。

**図 2-2 Notion のアプリで一覧をフィルタリングして GitHub Mobile で表示**

![Notion アプリでリポジトリ一覧を表示、GitHub Mobile で特定のリポジトリを表示としているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b8615238fd4b48b48ab63cd3d0b864f8/send-a-repo-list-to-notion-database-mobile.png.png?w=1110\&h=1040\&auto=compress%2Cformat)

[^intent]: 手元の Android 端末では(おそらく intent の設定で)そのような挙動になっています。

[^home-tab]: GitHub Mobile でもプロフィールタブの下にあるリポジトリボタンを使えば同じようなことをできます。ホームタブにあるリポジトリボタンの場合だとソートとフィルタリングのない画面だったので気が付きませんでした、と言い訳しておきます。

では意味がないのかというと、GitHub Mobile では直接開くことができない Actions タブなどのリンクも関数で追加できるので、各種ページにアクセスしやすくなりました。 (リンクとして有効にするにはテーブルビューが必要です)

**図 2-3 関数プロパティで actions タブへのリンク作成**

![Notion 上で関数プロパティの編集画面を開き、リンク文字列を合成する式を入力しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/40ec9831ac5946a99b72ea8097290a3c/send-a-repo-list-to-notion-database-function-to-actions.png?w=588\&h=374\&auto=compress%2Cformat)

あとは、当初は想定していなかったのですが、Notion ならではの使い方としてリポジトリに関連する細々したメモとリマインダーを追加しやくなりました。

**図 2-4 リポジトリのページにメモやリマインダーを記述**

![追加されたリポジトリのページにメモやリマインダーを記述しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/bcfa9876776f4bc782156b5cc4424083/send-a-repo-list-to-notion-database-memo.png?w=894\&h=607\&auto=compress%2Cformat)

メモ的なものを設定するには GitHub Projects を使う方法もあると思いますが、Notion では AI を利用できるので「個人的なメモは Notion 側に集約かな？」に落ち着きそうです。

ちなみに、Notion AI だと[メモ上のテーブル操作など細々した編集が楽になりそう](https://zenn.dev/hankei6km/scraps/239bb1e204ac31)なので、サブスクリプションを設定しようか絶賛お悩み中です。

::: message

`@` によるリマインダーで通知させる場合は、各リポジトリのページで通知設定を「すべてのコメント」にする必要があります。設定はページ右上の時計アイコンから行えます(モバイルアプリでは `…` メニューから「ページの更新」で表示できます)。

![ページの履歴を表示し、通知設定のプルダウンメニューを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/02026986d7b94b9aa6a7e90714191748/send-a-repo-list-to-notion-database-notification.png?w=403\&h=380\&auto=compress%2Cformat)

できればページを作成するときにツールから設定したかったのですが、Notion AI に訪ねても実現方法がわかりませんでした。しかたがないので、テンプレートに「設定を変更すること」という文言を入れることで対応しています。

:::

## データベースの定期更新

定期更新する方法はいくつかありますが、今回はそれほどリアルタイム性を求めていないので普通に GitHub Actions を使うことにしました。

ワークフローの作成自体は Node.js のパッケージを実行するだけなので複雑ではないのですが、トークン関連などが込み入っていたので少し説明など。

### アクセストークンの生成

GitHub Actions の場合、デフォルトで `GITHUB_TOKEN` が設定されていますがこれは他のリポジトリに対しての権限がありません。よって、何らかの方法で権限を取得する必要が出てきます。

個人的にちょっと動かすだけなら Personal Access Token が手っ取り早いかと思いますが、有効期間の問題などもあるので今回は GitHub Apps でトークンを生成しています。

https://github.com/hankei6km/reposition-schedule/blob/c766ef60339b0c1dd58c3bfc22721c7909de8077/.github/workflows/send.yml#L36-L51

<!-- textlint-disable --> 

具体的な GitHub App の作成とインストールの手順については「[GitHub Apps + GitHub Actionsで必要なアクセス権限のみ付与した一時的なアクセストークンを発行する | DevelopersIO](https://dev.classmethod.jp/articles/getting-an-access-token-with-only-the-necessary-permissions-on-github-appsgithub-actions/)」がわかりやすいかと思います。

<!-- textlint-enable --> 

権限の設定は下記のようにしました。この辺は一覧にしたいリポジトリによって変わるかと思います。

*   作成した Github App の Permissions は `Repository permission` の `metadata` に `Read only`
*   インストール後の `Repository access` は `All repositories`(一覧に含めたいリポジトリだけでも良いはずです)

権限まわりについては「[GitHub Actions から別プライベートリポジトリにアクセス](https://zenn.dev/yuki0n0/articles/8eb75018575569)」が参考になるかと思います

### スケジュール設定

1 日に 2 回実行する指定となっています。

https://github.com/hankei6km/reposition-schedule/blob/c766ef60339b0c1dd58c3bfc22721c7909de8077/.github/workflows/send.yml#L2-L5

時刻については以前に作成した「[GitHub Actions のスケジュール実行による待機時間を Google Data Portal で可視化した](https://zenn.dev/hankei6km/articles/visualization-of-waiting-time-in-github-actions)」の内容から混雑していなさそうな時間帯にしました。

なお、スケジュール実行するだけのリポジトリは[しばらく放置しておくと停止される](https://zenn.dev/hankei6km/articles/avoid-disabling-workflow-in-github-actions)可能性があります。今回は dependabot を設定してあるので bump されて大丈夫かなと予想していますが、同じような構成にする場合は注意が必要です。

### 送信するリポジトリ数を減らす

定期更新でリポジトリの一覧を全件送信すると毎回それなりに時間がかかってしまいます。

GitHub CLI (`gh repo list`)は一覧のソートがきないようなので、「直近で更新された(push された)リポジトリの一覧だけ送信したい」という場合は下記のような対応になります。

*   GitHub CLI では `-L` で大きな値を指定しておいて全件出力させる
*   `reposition` 側で `--filter-time-range` を使いフィルタリングする

オプションの値は固定でもよいのですが、あとから変更しやすいように構成変数で指定してあります。[構成変数は少し前に追加された機能で](https://github.blog/2023-01-10-introducing-required-workflows-and-configuration-variables-to-github-actions/#configuration-variables)、秘匿する必要がないオプションの設定などに向いていると思います(変数によって扱いが変わるのは[うっかりミスやりそうでちょっと怖い気もしていますが](https://zenn.dev/hankei6km/scraps/e47e1acfda55f7))。

**図 3-1 構成変数によるオプション指定(UI 上で値が表示される)**

![GitHub のウェブ UI で構成変数の一覧を表示し、変数の内容を確認しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/09dd9d77874747b09cdb9327b58011dc/send-a-repo-list-to-notion-database-configuration-value.png?w=828\&h=322\&auto=compress%2Cformat)

## 新しく利用したツールなど

本題から少し外れますが、新しく利用したものなどについて少しだけ。

### Notion AI

プライベートアルファで順番が回ってきたので、下記のスクラップような感じで利用しました。他にもサンプリングしたデータから [Ajv](https://github.com/ajv-validator/ajv) のスキーマやテスト用のモックデータを生成したりと、細々したところを補助してもらいました。

*   [Express で req.query のバリデーションをしたい【Notion AI を使って作ったメモ】](https://zenn.dev/hankei6km/scraps/528516115b142f)
*   [Node.js で ndjson(jsonl)を扱う](https://zenn.dev/hankei6km/scraps/bebce6ecf7b7e0)
*   [Notion AI でコミットメッセージを生成する実験](https://zenn.dev/hankei6km/scraps/8fb539e0bc2a12)
*   [Node.js の process.stdin を Jest の mock にする(たぶん Jest でなくても動く)](https://zenn.dev/hankei6km/scraps/85df38120a407c)

AI からの返答は結構怪しいですが(存在しない VSCode の拡張機能をお薦めされたりしました)、下記のような用途は楽になりそうだと思いました。

*   漠然と「〇〇するにはどうすればいいんだ？」というとき、軽くメモをまとめる
*   Vim の q マクロなどで自動化するのはちょっとめんどうなデータの生成

正確さについては「使うこっちも言うほど正確でもないので、まぁお互い様かな」ということにしています(上記の記事を見返すと Stream のデータ切り出しはちょっとよろしくなかった気がしてきました)。

どちらかというと「一旦作ったスキーマをちょっと変更したい」というとき、再生成を依頼するとガラッと違うものができ上ったりするので少し困りました。あとから修正というときは、まだ人間が対応になるのかなと(この辺 Copilot だとまた違うのかもしれませんが)。

### CodeSandbox

以前にも使っていてしばらくご無沙汰だったのですが、[リポジトリ機能](https://codesandbox.io/docs/learn/repositories/overview)という便利なのものが追加されていたので最近は強引に Dev Container 化して利用しています。

*   [CodeSandbox の podman で Dev Containers を試す](https://zenn.dev/hankei6km/scraps/4ef9618f5035b2)

また、Nix とコンテナ環境(rootless の Docker or Podman)が使えるので、act などによる GitHub Actions のデバッグが気軽にできます。今回は定期更新用のリポジトリに Dev Container を設定しなかったので利用しました。

*   [GitHub Actions のワークフローを CodeSandbox でデバッグする](https://zenn.dev/hankei6km/scraps/4c504c25ab4214)

と、べた褒めで締めたいところですが、CodeSandbox はわりと頻繁に挙動が変更されます。そこはもうちょっとお手柔らかにしてほしいかなと。

## おわりに

Notion のデータベースにリポジトリの一覧を送信するツールを作ってみました。

当初の目的とは少しズレてしまいましたが「リポジトリに関連したちょっとしたリマインダーを設定しておく」はわりと良い感じかとな思っています。
