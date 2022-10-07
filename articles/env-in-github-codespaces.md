---
id: env-in-github-codespaces
title: GitHub Codespaces でのシークレットを含む環境変数(と .env)について考えてみる
emoji: 🙊
type: tech
topics:
  - env
  - github
  - codespaces
published: true
---

プログラムの設定やシークレットを環境変数で扱うために「リポジトリ内のディレクトリーに `.env` ファイルを配置する」ことはわりとあるかと思います。私も `source` コマンドで切り替えるくらいの使い方ですが、やはり便利に使っています。

一方で「`.gitignore` で除外しているとはいってもシークレットがリポジトリ内のディレクトリーに配置されてるのはやっぱりおっかないよなぁ」とも思っています[^secret-external]。

[^secret-external]: git secret を使うかリポジトリの外に置くという方法もありますが、それはそれで管理が少し手間かと。

また、GitHub Codespaces を使うようになってから、以下のようなこともぼんやりと考えていました。

*   Codespace を作る毎に `.env` ファイルをコピーするのは面倒
*   Codespaces は GitHub 上で Secrets を設定できるので利用したい

というわけで、今回は GitHub Codespaces で Secrets を使った環境変数の扱いについてなど。

## `.env` ファイルについて

本題へ行く前に `.env` ファイルについて少し。

`.env` ファイルはいろいろなツールやプラットフォームなどで使われていますが、統一されたフォーマットは存在していません。

@[card](https://qiita.com/ko1nksm/items/1c3ae628081e4e9c8ad2)

たとえば、冒頭で「`source` コマンドで切り替える」と書きましたが、状況によっては `source` などで(シェルスクリプト的に) `.env` ファイルを扱うことは許容できないこともあるようです。一方で [dotenv](https://github.com/bkeepers/dotenv) 用の `.env` では `export` を付けて`source` コマンドなどで利用することは想定されています[^escape]。

[^escape]: エスケープ文字の扱いなどのすり合わせは必要になります。

> You may also add `export` in front of each line so you can `source` the file in bash:
>
> ```shell
> export S3_BUCKET=YOURS3BUCKET
> export SECRET_KEY=YOURSECRETKEYGOESHERE
> ```

また、最近では下記のような議論もありました。

@[card](https://dev.to/gregorygaines/stop-using-env-files-now-kp0)
@[card](https://dev.to/wiseai/continue-using-env-files-as-usual-2am5)

このような混とんとした状況であることと、Codespaces では構造上「リポジトリの履歴に含まれないファイル」は永続化しにくいので、今回は下記の方向で考えることにします。

*   ファイルとして `.env` を使うことはできれば避ける
*   ただし、環境変数を切り替えるフォーマットとして [dotenv](https://github.com/bkeepers/dotenv) の記法を尊重する

## Codespaces 用の Secrets を普通に使う

前置きが少し長くなりましたが、ここからは実際に設定してみます。

GitHub Codespaces では 2 種類の Secrets を設定できます。

1.  [Codespaces 全体で共有される Secrets](https://docs.github.com/ja/codespaces/managing-your-codespaces/managing-encrypted-secrets-for-your-codespaces)
    Secret を追加した後、参照を許可したリポジトリで Codespace を作成すると利用される
2.  リポジトリ単位で設定される Secrets
    Secret を追加しておくと、そのリポジトリで Codespace を作成したときに利用される

今回はリポジトリ単位で設定される Secrets を使ってみます。

リポジトリ Secrets は GitHub 上のポジトリの `Setting / Secret / Codespace` を開くと設定できます。

**図 2-1 リポジトリの設定メニュー**

![GitHub でリポジトリの設定を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/98c84342cd8d4b5784dfec8fbf1d7d3a/env-in-github-codespaces-repo-secret1.png?w=359\&h=241\&auto=compress%2Cformat)

GitHub CLI からは `gh secret` コマンドで設定できます(下記は Codespace を作成後に Terminal から実行しています)。

**図 2-2 GitHub CLI で Secret を設定**

```shell-session
$ gh secret set --app codespaces SECRET
? Paste your secret ****************************************************************

✓ Set Codespaces secret SECRET for hankei6km/test-codespace-env
```

設定した後、Codespace を開くかリロードすると Secrets が環境変数として設定されています。なお、Terminal などで表示するとそのまま出力されます(GitHub Actions の Secrets のようなマスクは行われません)。

**図 2-3 Terminal で Secret を確認**

```shell-session
$ env | grep -e "^SECRET="
SECRET=412f0711fb20f9c558e5147bb789b3726d070d19c1b502bf7c3aed009416915b
```

この方法では下記の点がメリットになるかと思われます。

*   追加のツールを必要としない
*   設定してあるリポジトリで Codespace を作成すると自動的に反映されている
*   (共有範囲に注意は必要だが)シークレットを扱うこができる
*   シークレットを `.env` ファイルに保存しないでよい

一方でデメリットもあります。

*   リポジトリで 1 つの設定しか行えない[^share]

    *   ブランチや Codespace 単位で設定を切り替えることはできない

*   リポジトリを削除すると Secrets も消える

*   設定内容を確認しにくい(ウェブ UI や GitHub CLI からは内容を表示できない)

[^share]: 組織での利用など、状況によっては都合がよい場合もあるので一概にデメリットとも言えないのですが。

この他、利用できない場所もあるので状況によってはデメリットになるかもしれません。

@[card](https://docs.github.com/ja/codespaces/managing-your-codespaces/managing-encrypted-secrets-for-your-codespaces#using-secrets)

あとは少し特殊なケースですが、tmux のようなツールを使っていると「Codespace をリロードしても稼働中のセッションに単純には反映されない」という問題もあります。

## 設定を動的に切り替える方法を考える

上記のデメリットの中で「リポジトリで 1 つの設定しか行えない」は工夫の余地がありそうです。まずは「枝番的な Secrets を設定しておいて `export` で切り替える」ことで対応してみます。

以下 GitHub CLI を使って操作します。

**図 3-1 `SECRET1` `SECRET2` を設定**

```shell-session
$ gh secret set --app codespaces SECRET1
? Paste your secret ****************************************************************

✓ Set Codespaces secret SECRET1 for hankei6km/test-codespace-env

$ gh secret set --app codespaces SECRET2
? Paste your secret ****************************************************************

✓ Set Codespaces secret SECRET2 for hankei6km/test-codespace-env
```

**図 3-2 Codespace をリロード後に内容を確認**

```shell-session
$ gh secret list --app codespaces
SECRET1                 Updated 2022-10-06
SECRET2                 Updated 2022-10-06

$ env | grep -e "^SECRET1="
SECRET1=594bf655565aec479951f38dff42ebad54199a0df7a5e9321bac32086f5c6ff5

$ env | grep -e "^SECRET2="
SECRET2=67c163325cbdbe6792175027d7ea4573e3bd340c8dc1f3734ad2ea90ba738b0d
```

設定された `SECRET1` と `SECRET2` を使って`SECRET` を切り替えてみます。

**図 3-3 `export` で切り替え**

```shell-session
$ export SECRET="${SECRET1}"

$ env | grep -e "^SECRET="
SECRET=594bf655565aec479951f38dff42ebad54199a0df7a5e9321bac32086f5c6ff5

$ export SECRET="${SECRET2}"

$ env | grep -e "^SECRET="
SECRET=67c163325cbdbe6792175027d7ea4573e3bd340c8dc1f3734ad2ea90ba738b0d
```

比較的容易に切り替えることができました。また、切り替え対象が増えた場合は「Secrets に環境変数を切り替えるスクリプトを埋め込む」方法もあります。

ここでは [dotenv](https://github.com/bkeepers/dotenv) の書式に沿って `export` を追加した `.env` を利用してみます。

:::message
今回は `.env` として埋め込んでいるだけですが、通常のスクリプトも埋め込めるので `source` での実行時には注意してください。
:::

**リスト 3-1 `export` 付きで `.env` を作成**

```bash
export SECRET=594bf655565aec479951f38dff42ebad54199a0df7a5e9321bac32086f5c6ff5
export DATABASE_ID=67c163325cbdbe6792175027d7ea4573e3bd340c8dc1f3734ad2ea90ba738b0d
```

作成したファイルを 1 つの Secret として追加します。

**図 3-4 `.env` から 1 つの Secret を追加**

```shell-session
$ gh secret set --app codespaces DOTENV1 < ~/tmp/denv/.env
✓ Set Codespaces secret DOTENV1 for hankei6km/test-codespace-env
```

**図 3-5 Codespace をリロード後に内容を確認**

```shell-session
$ gh secret list --app codespaces
DOTENV1                 Updated 2022-10-06
SECRET1                 Updated 2022-10-06
SECRET2                 Updated 2022-10-06

$ echo "${DOTENV1}"
export SECRET=594bf655565aec479951f38dff42ebad54199a0df7a5e9321bac32086f5c6ff5
export DATABASE_ID=67c163325cbdbe6792175027d7ea4573e3bd340c8dc1f3734ad2ea90ba738b0d
```

`souce` コマンドで各環境変数を設定してみます(パイプを使えないのでプロセス置換を利用しています)。

**図 3-6 `srouce` コマンドで設定**

```shell-session
$ source <(echo "${DOTENV1}")

$ env | grep -e ^SECRET=
SECRET=594bf655565aec479951f38dff42ebad54199a0df7a5e9321bac32086f5c6ff5

$ env | grep -e ^DATABASE_ID=
DATABASE_ID=67c163325cbdbe6792175027d7ea4573e3bd340c8dc1f3734ad2ea90ba738b0d
```

普通に `srouce` コマンドを使う感覚で設定されました。

これで従来のような使い勝手に近づきましたが、`export` や `source` を使う方法では以下のようなデメリットがあります。

*   コマンドを実行した Terminal 内でしか環境変数は有効にならない

デバッガーなどは Terminal 内で稼働させれば設定した環境変数にアクセスできますが、たとえば [`launch.json`](https://code.visualstudio.com/docs/editor/debugging#_launchjson-attributes) 内で [`${env:SECRET}`](https://code.visualstudio.com/docs/editor/variables-reference#_environment-variables) のようには利用できません。

## やっぱり `.env` ファイルとして保存したい

方針として「できれば避ける」としましたが、VSCode の機能で `.env` を設定することもあるので少しだけ。

単純なことですが、Secrets へ設定した値は以下のようにするとファイルへ保存されます。

**図 4-1 `.env` ファイルへ保存**

```shell-session
$ echo "${DOTENV1}" > "${CODESPACE_VSCODE_FOLDER}/.env"
```

保存した `.env` ファイルの永続期間は配置したディレクトリーによって下記のようになります。

*   **リポジトリ(Codespace ワークスペース)内** - Codespace の停止と再開、コンテナのリビルドを行っても永続する
*   **ホームディレクトリなど** - コンテナのリビルドを行うと削除される

一旦保存しておけば(普通に使う分には)保持されるので、`.env` を扱うツールなどでの利用が可能になります。

## その他の設定方法

シークレットを扱うには向いていませんが、他にも環境変数を設定する方法はいくつかあります。

### DevContainer 用の設定

Codespaces では DevContainer を利用しているので、イメージのビルド時や [`devcontainer.json`](https://containers.dev/implementors/json_reference/) から環境変数を設定できます。

ここでは [`devcontainer.json`](https://containers.dev/implementors/json_reference/) の `containerEnv` を利用してみます。

Codespace を作成した後に、 `.devcontainer/devcontainer.json` へ以下のような設定を追加します。

**リスト 5-1 `devcontainer.json` の `remoteEnv`**

```json
{
  "remoteEnv": {
    "LOCAL_PORT": "8000"
  },
}
```

Codespace をリビルド後に Terminal を開くと環境変数が設定されています。

**図 5-1 設定された環境変数を確認**

```shell-session
$ env | grep -e "^LOCAL_PORT="
LOCAL_PORT=8000
```

Secrets を利用する方法に近い感覚で利用できますが、`.devcontainer/devcontainer.json` はリポジトリの履歴に含めることが多いのでシークレットを扱うには向いていません。

### Dotfiles

GitHub Codespaces のパーソナライズについてのドキュメントでは dotfiles リポジトリを設定する方法が記載されています。

@[card](https://docs.github.com/ja/codespaces/customizing-your-codespace/personalizing-github-codespaces-for-your-account#dotfiles)

こちらは具体的には試しませんが、FAQ を見た感じではシークレットを含めることは想定していないようです。

@[card](https://dotfiles.github.io/faq/)

## 考慮点

### 設定の確認と管理が面倒

上の方でも少し書きましたが、一旦 Secrets に保存してしまうとウェブ UI などからは内容を確認できません。確認するには Codespace を作成して Terminal などのセッションを使う必要があります。

また、設定が各リポジトリの Secrets に分散してしまうので、「各リポジトリ共通で使っている API キーを変更した」といった場合は少し面倒なことになります[^common]。

[^common]: Codespaces 共通の Secrets を使うとある程度は緩和できます。共通の Secrets を利用する場合は Secrets がどこまで共有されるか注意は必要です。 <https://dev.classmethod.jp/articles/github-organization-level-secrets/>

### 設定数の上限

Codespaces 全体での Secret の制限は下記のようになっています。リポジトリの場合については見つけることができなかったのですが、やはり制限はあるかと思います(もしかするとリポジトリの Secrets も下記にカウントされるかもしれません)。

@[card](https://docs.github.com/ja/codespaces/managing-your-codespaces/managing-encrypted-secrets-for-your-codespaces#limits-for-secrets)

> GitHub Codespaces には最大 100 個のシークレットを保存できます。
>
> シークレットの容量は最大64 KBです。

### スキャナー

たとえば、脆弱性のスキャンで「〇〇サービスの API キーをリポジトリの Secrets にべた書きするのは危険ですよ」的な警告が出る場合を考えてみます。

「API キーなどを定義した `.env`」をまるごと Secret に追加した場合、追加されたテキストは「API キーとして認識されない」可能性があります。少し考えすぎな気もしますが、この例に限らず「スキャンをすり抜ける」可能性は注意しておくのが良いかもしれません。

## おわりに

GitHub Codespaces で環境変数を設定する方法について考えてみました。

現在、Secrets に埋め込んだ `env` を `source` で切り替える方法を試していますが「短期的に使うなら悪くはないかな」という感じです。

しかし、継続して利用するなら「Secrets の内容確認と更新はもう少し手軽にできないと厳しいな」とも思いました。とくに内容確認は「この Secret は何を設定してる？」となることがありました(その辺はメモしとけよという話ではありますが)。

シークレットを外部へ保存することが許容されるなら、シークレットを扱うサービスや 1Password CLI などを検討するのもよいのかもしれません。

@[card](https://developer.1password.com/docs/cli)
@[card](https://developer.1password.com/docs/vscode)

実際に利用されている記事。

@[card](https://zenn.dev/lambdasawa/articles/op-cli-and-environment-variables)
