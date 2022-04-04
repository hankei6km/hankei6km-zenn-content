---
id: credentials-contained-files-on-github-actions
title: GitHub Actions でクレデンシャルなどが含まれるファイルを扱うときのアレコレ
emoji: 🔏
type: tech
topics:
  - githubactions
published: true
---

ちょっと長い前置きです。

[`clasp`] を試しているとき「GitHub 上でリリースを公開したら `clasp deploy` したい」と思ったので少し調べてみました。

現状では以下の方法が一般的なようです。

@[card](https://dev.classmethod.jp/articles/github-actions-gas-deploy/)
@[card](https://github.com/google/clasp/issues/707#issuecomment-844364570)

認証まわりが気になるところですが、概要としては「SECRET に設定したリフレッシュトークンなどを `.clasprc.json` へ保存し、トークンの更新はコマンド側にまかせる」という方法のようです。

これまで「できれば SECRET(クレデンシャルなど)を含むファイルの保存は避けたい」と思っていたのですが、そうも言ってられないようなのでその辺を少し試してみました。

## SECRET をファイルへ保存するコマンド

普通に考えると `$ echo -n "${PASSWORD}" > password.txt` とやりたくなりますが、以下によると可能な限り回避しておくべき方法に分類されています[^use-cmd-in-example]。

> Avoid passing secrets between processes from the command line, whenever possible. Command-line processes may be visible to other users (using the `ps` command) or captured by [security audit events](https://docs.microsoft.com/windows-server/identity/ad-ds/manage/component-updates/command-line-process-auditing). To help protect secrets, consider using environment variables, `STDIN`, or other mechanisms supported by the target process.

@[card](https://docs.github.com/en/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow)

たとえば [`jq`] の引数でフィルターに `env.PASSWORD` を使う場合、シェルからは文字列の `"env.PASSWORD"` として扱われ、プロセス側が環境変数から値を読み取ります[^child-proc]。

しかし、引数に `"${PASSWORD}"` などを使うとシェルが展開してからプロセスを開始するので、`ps` などで見えてしまうということかと。

```shell-session
$ echo "${PASSWORD}"
SDf%@m38x"Tc5N4r
$ echo "sleep 10" > long.sh
$ bash ./long.sh "${PASSWORD}" &
[1] 2364315
$ pgrep -af long
2364315 bash ./long.sh SDf%@m38x"Tc5N4r
```

よって、コマンドを選ぶ基準としては「SECRET の値を引数以外の方法で渡せる」ものを優先するということになります。

これを踏まえて、実際にいくつかの方法でファイルに保存してみます。

[^use-cmd-in-example]: 引用元では直後のクオートの例で引数に使っていたりするので、どの程度まで回避すべきかは悩むところです。

[^child-proc]: [`jq`] が引数付きで子プロセスを実行していないという前提です。

## テスト用のファイル

今回は以下のような `name` と `password` を含むファイルを保存する想定でテストしていきます。1 つは簡易なパスワードですが、もう 1 つはエスケープしないと JSON 化できないパスワードになっています。

▼ *リスト 2-1 簡易なパスワードのファイル*

```json:secret_file.json
{
  "name": "test",
  "password": "qwerty",
  "description": "test user"
}
```

▼ *リスト 2-2 エスケープが必要なパスワードのファイル*

```json:secret_file_esc.json
{
  "name": "test",
  "password": "SDf%@m38x\"Tc5N4r",
  "description": "test user"
}
```

フロー内でファイルを再現できているか確認するためにハッシュ値を求めておきます(インデントの書式などをあわせるために [`jq`] を通しています)。

▼ *図 2-1 ファイルのハッシュ値*

```shell-session
$ jq < secret_file.json | sha256sum
0787a289d748b9d98997e8bf0ef0d8c2a3336d34dd8f6dbf74c6bc433701baef  -
$ jq < secret_file_esc.json | sha256sum
9aac047d7b2ab93caaa2f98fc2df7c80e0d458c7ef078bfeb03c0046ee25bde8  -
```

## ファイルを丸ごと SECRET にセットしてしまう

利用する SECRET の数が 1 つで済むので手軽に実現できます。ただし、前述のように「SECRET を引数に使うことはできれば避けるべき」なので定番の `echo` は使わずに [`envsubst`] を利用します。

▼ *図 3-1 GitHub CLI で "file" environment へ SECRET を追加*

```shell-session
$ gh secret set --env file SECRET_FILE < secret_file.json
✓ Set secret SECRET_FILE for hankei6km/test-credentials-contained-file
```

▼ *リスト 3-1 envsubst 用のソース(テンプレート)*

```:secret_file_src_envsubst.json
$SECRET_FILE
```

▼ *リスト 3-2 保存とチェック処理*

````yaml:.github/workflows/file.yaml
jobs:
  save-file-from-secret:
    runs-on: ubuntu-latest
    environment: file

    steps:
      - uses: actions/checkout@v2

      - name: Save the sercret file
        run: envsubst < secret_file_src_envsubst.json > secret_file.json
        env:
          SECRET_FILE: ${{ secrets.SECRET_FILE }}

      - name: Check
        run: |
          echo "-- sha256sum"
          jq < secret_file.json | sha256sum 
          echo "-- ファイル表示 cat secret_file.json"
          cat secret_file.json
          echo "-- フィールド表示 jq .password secret_file.json"
          jq .password secret_file.json
          echo "-- フィールド表示(RAW) jq -r .password secret_file.json"
          jq -r .password secret_file.json```
````

▼ *図 3-2 実行結果*

    -- sha256sum
    0787a289d748b9d98997e8bf0ef0d8c2a3336d34dd8f6dbf74c6bc433701baef  -
    -- ファイル表示 cat secret_file.json
    ***
      ***
      ***
      ***
    ***

    -- フィールド表示 jq .password secret_file.json
    "qwerty"
    -- フィールド表示(RAW) jq -r .password secret_file.json
    qwerty

この方法の場合、ファイルの内容が SECRET に登録されているのでフローの中でファイルを表示してしまってもマスクされます。

しかし、ファイル内のフィールドを抜き出して表示するとマスクされません。

## フィールドの値を個別に SECRET へ登録し、コマンドでファイルを組み立てる

上記のマスクされないことはフィールドの値を個別に SECRET へ登録することで回避できます。

複数の SECRET(環境変数)は[`envsubst`] でも対応できますが、ファイルフォーマット(今回は JSON)を考慮して値をエスケープする必要があります。

ここでは、エスケープが必要なパスワードの方で試してみます。

なお、SECRET の登録が少し面倒ですが、[GitHub CLI ]では `.env` 的なファイルで一括登録できます。

▼ *リスト 4-1 SECRET の一覧を .env へ記述(PASSWORDはあらかじめエスケープ)*

```sh:.env
NAME=test
PASSWORD=SDf%@m38x\"Tc5N4r
```

▼ *図 4-1 GitHub CLI で "escape" environment へ SECRET を追加*

```shell-session
$ gh secret set --env escape --env-file .env
✓ Set secret PASSWORD for hankei6km/test-credentials-contained-file
✓ Set secret NAME for hankei6km/test-credentials-contained-file
```

▼ *リスト 4-2 envsubst 用のソース(テンプレート)*

```json:secret_file_src_envsubst_esc.json
{
  "name": "$NAME",
  "password": "$PASSWORD",
  "description": "test user"
}
```

▼ *リスト 4-3 保存とチェック処理*

```yaml:.github/workflows/escape.yaml
jobs:
  escaped-field-value:
    runs-on: ubuntu-latest
    environment: escape

    steps:
      - uses: actions/checkout@v2

      - name: Save the sercret file
        run: envsubst < secret_file_src_envsubst_esc.json  > secret_file.json
        env:
          NAME: ${{ secrets.NAME }}
          PASSWORD: ${{ secrets.PASSWORD }}

      - name: Check
        run: |
          echo "-- sha256sum"
          jq < secret_file.json | sha256sum 
          echo "-- ファイル表示 cat secret_file.json"
          cat secret_file.json
          echo "-- フィールド表示 jq .password secret_file.json"
          jq .password secret_file.json
          echo "-- フィールド表示(RAW) jq -r .password secret_file.json"
          jq -r .password secret_file.json
```

▼ *図 4-2 実行結果*

    -- sha256sum
    9aac047d7b2ab93caaa2f98fc2df7c80e0d458c7ef078bfeb03c0046ee25bde8  -
    -- ファイル表示 cat secret_file.json
    {
      "name": "***",
      "password": "***",
      "description": "*** user"
    }
    -- フィールド表示 jq .password secret_file.json
    "***"
    -- フィールド表示(RAW) jq -r .password secret_file.json
    SDf%@m38x"Tc5N4r

ファイルとして表示するとフィールドの値が個別にマスクされますが、フィールドの値を取り出して(生の値として)表示させるとマスクされません。これは SECRET に登録しているエスケープさせた値と一致しないためです。

なお、`description` の一部もマスクされていますが、これは逆に SECRET と一致する文字列があるためです。

## 改・フィールドの値を個別に SECRET へ登録し、コマンドでファイルを組み立てる

エスケープを考慮しながら SECRET をセットするのはマスクの問題を別としてもミスを誘発しやすいです(テスト中にも何度か間違えました)。

そこで次は JSON フォーマットを扱える [`jq`] を利用してみます。

なお、`.env` の記述でもエスケープが必要なときもありますが、個別でセットする場合ではあれば標準入力から渡すこともできます。

▼ *リスト 5-1 SECRET の一覧を .env へ記述*

```sh:.env
NAME=test
PASSWORD=SDf%@m38x"Tc5N4r
```

▼ *図 5-1 GitHub CLI で "command" environment へ SECRET を追加*

```shell-session
$ gh secret set --env command --env-file .env
✓ Set secret PASSWORD for hankei6km/test-credentials-contained-file
✓ Set secret NAME for hankei6km/test-credentials-contained-file
```

▼ *リスト 5-2 jq 用のソース(テンプレート)*

```json:secret_file_src_jq.json
{
  "name": "< user name>",
  "password": "< password >",
  "description": "test user"
}
```

▼ *リスト 5-3 保存とチェック処理*

```yaml:.github/workflows/command.yaml
jobs:
  build-file-by-command:
    runs-on: ubuntu-latest
    environment: command

    steps:
      - uses: actions/checkout@v2

      - name: Save the sercret file
        run: |
          jq '.name = env.NAME' secret_file_src.json  \
          | jq '.password = env.PASSWORD' > secret_file.json
        env:
          NAME: ${{ secrets.NAME }}
          PASSWORD: ${{ secrets.PASSWORD }}

      - name: Check
        run: |
          echo "-- sha256sum"
          jq < secret_file.json | sha256sum 
          echo "-- ファイル表示 cat secret_file.json"
          cat secret_file.json
          echo "-- フィールド表示 jq .password secret_file.json"
          jq .password secret_file.json
          echo "-- フィールド表示(RAW) jq -r .password secret_file.json"
          jq -r .password secret_file.json
```

▼ *図 5-2 実行結果*

    -- sha256sum
    9aac047d7b2ab93caaa2f98fc2df7c80e0d458c7ef078bfeb03c0046ee25bde8  -
    -- ファイル表示 cat secret_file.json
    {
      "name": "***",
      "password": "***",
      "description": "*** user"
    }
    -- フィールド表示 jq .password secret_file.json
    "***"
    -- フィールド表示(RAW) jq -r .password secret_file.json
    ***

実は「この方法だとファイルとして表示するとエスケープされているのでマスクされません」と書くつもりだったのですが、実際はマスクされています。

明記されたドキュメントを見つけることはできなかったのですが、`\` などの文字は無視されるように見えます。

少しスッキリしない部分もありますが、都合はよいので今回は良しとしています。

## Action を利用する

ここまではコマンドで保存することを前提にしていましたが、このような用途に利用できる Action がいくつかありました。

今回は以下の Action を試してみます。

@[card](https://docs.microsoft.com/en-us/azure/developer/github/github-variable-substitution)
@[card](https://github.com/marketplace/actions/variable-substitution)

SECRET の設定はコマンドで実施する場合と同じです。

処理の記述的には以下の点でコマンドよりも楽になります。

*   ファイルを上書き保存してくれるので別名のソース(テンプレート)を必要としません(今回は別名で保存しておいて後から `mv` しています)
*   `env` でセットしたいフィールドをキーとして SECRET を指定する
*   JSON 以外のフォーマットにも対応している

▼ *リスト 6-1 保存とチェック処理*

```yaml:.github/workflows/subst.yaml
jobs:
  build-file-by-var-subst-action:
    runs-on: ubuntu-latest
    environment: subst

    steps:
      - uses: actions/checkout@v2

      - run: mv secret_file_src.json secret_file.json

      - uses: microsoft/variable-substitution@v1 
        with:
          files: 'secret_file.json'
        env:
          name: ${{ secrets.NAME }}
          password: ${{ secrets.PASSWORD }}

      - name: Check
        run: |
          echo "-- sha256sum"
          jq < secret_file.json | sha256sum 
          echo "-- ファイル表示 cat secret_file.json"
          cat secret_file.json
          echo "-- フィールド表示 jq .password secret_file.json"
          jq .password secret_file.json
          echo "-- フィールド表示(RAW) jq -r .password secret_file.json"
          jq -r .password secret_file.json
```

▼ *図 6-1 実行結果(簡単なパスワードの場合)*

    -- sha256sum
    0787a289d748b9d98997e8bf0ef0d8c2a3336d34dd8f6dbf74c6bc433701baef  -
    -- ファイル表示 cat secret_file.json
    {
        "name": "***",
        "password": "***",
        "description": "*** user"
    }-- フィールド表示 jq .password secret_file.json
    "***"
    -- フィールド表示(RAW) jq -r .password secret_file.json
    ***

▼ *図 6-2 実行結果(エスケープが必要なパスワードの場合)*

    -- sha256sum
    9aac047d7b2ab93caaa2f98fc2df7c80e0d458c7ef078bfeb03c0046ee25bde8  -
    -- ファイル表示 cat secret_file.json
    {
        "name": "***",
        "password": "***",
        "description": "*** user"
    }-- フィールド表示 jq .password secret_file.json
    "***"
    -- フィールド表示(RAW) jq -r .password secret_file.json
    ***

結果は [`jq`]のときと同じで(JSON のインデントなどが異なりますが意味的には同等です)、 JSON のエスケープを考慮しないで SECRET に値をセットできます。

## ファイル保存後の注意点

ファイルを保存した後にもいくつか注意する点があります。

### アップロード

リリース用に作成するアーカイブや GitHub Actions のキャッシュに SECRET を含むファイルを含めてしまうと、内容を閲覧できてしまいます。

以下、リリースの Assets に間違ってアップロードしてしまった場合の例です[^cache]。

▼ *リスト 7-1 ビルド時にsecret\_file.json をアーカイブファイルに含めてしまっている*

```yaml:.github/workflows/subst.yaml
jobs:
  upload-files:
    runs-on: ubuntu-latest
    environment: upload

    steps:
      - uses: actions/checkout@v2

      - name: Save the sercret file
        run: |
          jq '.name = env.NAME' secret_file_src.json  \
          | jq '.password = env.PASSWORD' > secret_file.json
        env:
          NAME: ${{ secrets.NAME }}
          PASSWORD: ${{ secrets.PASSWORD }}

      - name: Build
        run: zip secret_file_sourc.zip secret_file*.json

      - name: Upload
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ github.event.release.upload_url }}
          asset_path: ./secret_file_sourc.zip
          asset_name: secret_file_sourc.zip
          asset_content_type: application/zip
```

![GitHub 上でリリース表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c1d13ac464644b209a5bc32841f99d25/credentials-contained-files-on-github-actions-upload.png?w=600\&h=358\&auto=compress%2Cformat)*図 7-1 Assets にビルドされたファイルがアップロードされている*

▼ *図 7-2 ファイルをダウンロードすると SECRET が含まれている*

```shell-session
$ wget "https://github.com/hankei6km/test-credentials-contained-file/releases/download/upload-1/secret_file_sourc.zip"

<snip>

2022-01-23 19:32:27 (18.0 MB/s) - `secret_file_sourc.zip' へ保存完了 [940/940]

$ unzip -p secret_file_sourc.zip secret_file.json
{
  "name": "test",
  "password": "qwerty",
  "description": "test user"
}
```

表示時のマスクのような厳密なフィルターはないので、対策としては明示的にディレクトリーをわける(ワークスペースの外に保存するなど)といったところでしょうか。

[^cache]: キャッシュとクレデンシャルについてはリンク先の警告を参照 <https://docs.github.com/ja/actions/advanced-guides/caching-dependencies-to-speed-up-workflows>

### リフレッシュトークン

これはファイルに保存する場合だけの話ではないのですが。

「更新されたパスワードやトークンは SECRET に登録されていない」ので、値を表示してしまうとマスクされません[^refresh]。

▼ *リスト 7-2 保存とチェック処理*

```yaml:.github/workflows/refresh.yaml
jobs:
  refresh-field-value:
    runs-on: ubuntu-latest
    environment: refresh

    steps:
      - uses: actions/checkout@v2

      - name: Save the sercret file
        run: |
          jq '.name = env.NAME' secret_file_src.json  \
          | jq '.password = env.PASSWORD' > secret_file.json
        env:
          NAME: ${{ secrets.NAME }}
          PASSWORD: ${{ secrets.PASSWORD }}

      - name: Check
        run: |
          echo "-- sha256sum"
          jq < secret_file.json | sha256sum 
          echo "-- ファイル表示 cat secret_file.json"
          cat secret_file.json
          echo "-- フィールド表示 jq .password secret_file.json"
          jq .password secret_file.json
          echo "-- フィールド表示(RAW) jq -r .password secret_file.json"
          jq -r .password secret_file.json
          echo "-- password を picture1 へ変更"
          jq '.name = env.NAME' secret_file_src.json  \
          | jq '.password = "picture1"' > secret_file.json
          echo "-- ファイル表示 cat secret_file.json"
          cat secret_file.json
          echo "-- フィールド表示 jq .password secret_file.json"
          jq .password secret_file.json
          echo "-- フィールド表示(RAW) jq -r .password secret_file.json"
          jq -r .password secret_file.json
        env:
          NAME: ${{ secrets.NAME }}
```

▼ *図 7-3 実行結果*

    -- sha256sum
    9aac047d7b2ab93caaa2f98fc2df7c80e0d458c7ef078bfeb03c0046ee25bde8  -
    -- ファイル表示 cat secret_file.json
    {
      "name": "***",
      "password": "***",
      "description": "*** user"
    }
    -- フィールド表示 jq .password secret_file.json
    "***"
    -- フィールド表示(RAW) jq -r .password secret_file.json
    ***
    -- password を picture1 へ変更
    -- ファイル表示 cat secret_file.json
    {
      "name": "***",
      "password": "picture1",
      "description": "*** user"
    }
    -- フィールド表示 jq .password secret_file.json
    "picture1"
    -- フィールド表示(RAW) jq -r .password secret_file.json
    picture1

~~これについて対応策は「表示しない」ということになりそうです。~~

追記: 文字列を一時的にマスクするコマンドがありました。

新しい値が確定した後に以下の `::add-mask` コマンドを使うとマスクできます。

> ログに "Mona The Octocat" を出力すると、"\*\*\*" が表示されます。
>
> Shell
>
> ```bash
> echo "::add-mask::Mona The Octocat"
> ```

@[card](https://docs.github.com/ja/actions/using-workflows/workflow-commands-for-github-actions#masking-a-value-in-log)

この部分については、[別途記事にしました](http://localhost:8000/articles/add-mask-command-in-github-actions)。

[^refresh]: そのためのリフレッシュトークンだとも言えますが、やはりよろしくはないかなと。

## その他(Bash の Process Substitution)

ファイルを利用する側の挙動によっては利用できないことも多いのですが、うまく使えればファイルへの保存を回避できます。

なお、[`clasp`] では利用できませんできした[^clasp-proc-subst]。

▼ *リスト 8-1 記述例*

    foo-cmd <(jq '.name = env.NAME' < secret_file_src.json | jq '.password = env.PASSWORD')

[^clasp-proc-subst]: [Bash の Process Substitution で read write されるファイルの代替(clasp で ](https://zenn.dev/hankei6km/scraps/ac5f94c628520e)[`clasprc.json`](https://zenn.dev/hankei6km/scraps/ac5f94c628520e)[ をファイルにしないで認証させたい)](https://zenn.dev/hankei6km/scraps/ac5f94c628520e)

## おわりに

途中から「SECRET を表示する前提で話を進めるのはどうなのよ」と少し思いましたが、以下の点には気を付けておく方がよさそうです。

*   ファイル保存に使うコマンドは SECRET をコマンド引数以外で扱えるものを選ぶ
    `ps` などで確認されないようにするため
*   設定する値はできるだけ最小単位にする
    個別に値を表示してしまったときにマスクさせるため
*   対象のファイルフォーマット用のコマンド(または Action)を使う
    値を設定するときのエスケープを回避するため
*   ファイルを保存するディレクトリーなどはきちんと管理する
    うっかりアップロードしてしまわないため
*   リフレッシュされた値は表示しない、または `::add-mask` でマスクさせる
    新しい値は SECRET に登録されていないので

なお、現在では以下のような方向に進んでいるようです。今回のようなことは「昔は苦労したもんだ」的な過去の話になると思われます(願望)。

@[card](https://zenn.dev/miyajan/articles/github-actions-support-openid-connect)

[`clasp`]: https://github.com/google/clasp

[GitHub CLI]: https://cli.github.com/

[`jq`]: https://stedolan.github.io/jq/

[`envsubst`]: https://www.gnu.org/software/gettext/manual/html_node/envsubst-Invocation.html
