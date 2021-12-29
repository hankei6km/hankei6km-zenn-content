---
id: set-secret-to-repo-with-githubcli
title: GitHub CLI でリポジトリへ secret を設定
emoji: "\U0001F4CC"
type: tech
topics:
  - github
  - githubcli
published: true
---
これまではウェブの UI から設定していたのですが、そこそこ多い個数の変数を設定する必要がでてきたので [GitHub CLI](https://cli.github.com/) を使うことにしました。

## マニュアル

Secret をリポジトリ(Environment) へ設定するには `secret` コマンドの `set` を利用します。

@[card](https://cli.github.com/manual/gh_secret)

## 試してみる

ここでは設定と削除を試してみます。

### リポジトリ Secret へ設定

`set` では Secret の内容を `--body` と標準入力などから指定することになっています。

#### `--body` で設定

```shell-session
$ gh secret set BASE_URL --body "foo"
✓ Set secret BASE_URL for hankei6km/test-collage-cms-content
$ gh secret list
BASE_URL  Updated 2021-12-28
```

![GitHub リポジトリに secret が設定されたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9500b865d7d14ec38898f8da217450f7/set-secret-to-repo-with-githubcli-body.png?w=600\&h=126\&auto=compress%2Cformat)*ウェブ UI でリポジトリ secrets を確認*

#### 標準入力で設定

```shell-session
$ gh secret set BASE_URL < base_url_body.txt
✓ Set secret BASE_URL for hankei6km/test-collage-cms-content
$ gh secret list
BASE_URL  Updated 2021-12-28
```

:::message

標準入力から渡される値は改行もそのまま設定されます。

:::

### Environment secrets へ設定

Environment は `--env` で指定します。

```shell-session
$ gh secret set --env pages BASE_URL < base_url_body.txt
✓ Set secret BASE_URL for hankei6km/test-collage-cms-content
$ gh secret list --env pages
BASE_URL  Updated 2021-12-28
```

![GitHub リポジトリの Environment に secret が設定されたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d07f9e4607ff473fbbfcf7ce3e9f16e6/set-secret-to-repo-with-githubcli-to-environment.png?w=600\&h=185\&auto=compress%2Cformat)*ウェブ UI で Environment secrets を確認*

### `.env` から設定

::: message

GitHub CLI 2.4.0 以降が必要です。

> secret set: allow importing secrets from a dotenv file by @lpessoa in #4534

<https://github.com/cli/cli/releases/tag/v2.4.0>

:::

`.env` からの一括設定にも対応していました。

試した限りでは、コメントや空行があっても問題はなさそうです。`"` などのエスケープ方法については今回は試していません(必要になったら調べようかと)。

```shell:.env
# test secret
BASE_PATH="/test-collage-cms-content/"

BASE_URL="https://hankei6km.github.io"
```

```shell-session
$ gh secret set --env pages-staging --env-file .env
✓ Set secret BASE_PATH for hankei6km/test-collage-cms-content
✓ Set secret BASE_URL for hankei6km/test-collage-cms-content
```

![.env で指定されていた変数が設定されている状態のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/14c995dc797448dfb5da89f39308311e/set-secret-to-repo-with-githubcli-dotenv.png?w=600\&h=205\&auto=compress%2Cformat)*ウェブ UI で一括登録されていることを確認*

### 削除

削除コマンド実行時にウェブ UI のような確認はありません。そのまま実行されます。

```shell-session
$ gh secret remove BASE_URL
✓ Removed secret BASE_URL from hankei6km/test-collage-cms-content
$ gh secret list
// 削除されている
```

![GitHub リポジトリから secret が削除された状態のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a4eb5e2999a84759bfaef06affad39ae/set-secret-to-repo-with-githubcli-removed.png?w=600\&h=178\&auto=compress%2Cformat)*ウェブ UI で削除されていることを確認*

## 改行には注意

標準入力からファイル内の値などを渡す場合、値はそのまま利用されるので末尾の改行に注意が必要です。

最初 Action が失敗する理由がわからなくてしばらく悩みました(ワークフローのログに **`****`**`\n` という表示があったので早めに対応できました)。

```shell-session:NG
$ echo "foo" | gh secret set BASE_URL
✓ Set secret BASE_URL for hankei6km/test-collage-cms-content
```

```shell-session:OK
$ echo -n "foo" | gh secret set BASE_URL
✓ Set secret BASE_URL for hankei6km/test-collage-cms-content
```

## スクラップ

この記事は以下のスクラップから作成しました。

@[card](https://zenn.dev/hankei6km/scraps/b875aede4fbd60)
