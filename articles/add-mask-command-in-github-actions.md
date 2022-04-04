---
id: add-mask-command-in-github-actions
title: GitHub Actions で文字列を一時的にマスクする
emoji: '*️⃣'
type: tech
topics:
  - githubactions
published: true
---

ワークフローの中でクレデンシャルを更新する場合、「取得したトークなどを不用意に表示するとマスクされない」ことは少し面倒だと感じていました。

これについて、[別の記事](https://zenn.dev/hankei6km/articles/fetch-github-apps-token-by-google-apps-script)を作成中に [tibdex/github-app-token] Action のドキュメントを読んでいたところ、取得した Token をマスクしていたので方法を調べてみました。

## API での設定

Action のソースを読むと [`@actions/core`] の `setSecret()` を[利用していました](https://github.com/tibdex/github-app-token/blob/cd81e4cb40c3672f41b30a21758d6298e821691b/src/index.ts#L29)。

ドキュメントでは以下のように説明されています。runner への登録なので恒久的な設定ではなくジョブ実行時の一時的なものと思われます。

> Setting a secret registers the secret with the runner to ensure it is masked in logs.
>
> ```js
> core.setSecret('myPassword');
> ```

@[card](https://github.com/actions/toolkit/tree/main/packages/core#setting-a-secret)

## ワークフローコマンドでの設定

[`@actions/core`] の機能はワークフローコマンドでも使える場合が多いです。

> [actions/toolkit]には、ワークフローコマンドとして実行できる多くの関数があります。 `::`構文を使って、YAMLファイル内でワークフローコマンドを実行してください。それらのコマンドは`stdout`を通じてランナーに送信されます。 たとえば、コードを使用して出力を設定する代わりに、以下のようにします。

@[card](https://docs.github.com/ja/actions/using-workflows/workflow-commands-for-github-actions#using-workflow-commands-to-access-toolkit-functions)

今回の `setSecret()` も対象に含まれていたので bash などからも利用できます。

> ログに "Mona The Octocat" を出力すると、"\*\*\*" が表示されます。
>
> Shell
>
> ```bash
> echo "::add-mask::Mona The Octocat"
> ```

@[card](https://docs.github.com/ja/actions/using-workflows/workflow-commands-for-github-actions#masking-a-value-in-log)

## 試してみる

ワークフローコマンドでマスクを試すリポジトリを作成し、確認してみました。

@[card](https://github.com/hankei6km/test-gha-add-mask)

### 通常の設定

まずは普通に `secret_text.txt` に保存されている文字列を設定してみます。

▼ **リスト 3-1**

```yaml
simple-add-mask:
  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@v2

    - name: Check not masked
      run: cat secret_text.txt

    - name: Add mask
      run: |
        echo -n '::add-mask::' > cmd.txt
        cat secret_text.txt >> cmd.txt
        cat cmd.txt
        rm cmd.txt

    - name: Check masked
      run: cat secret_text.txt
```

ログを確認すると `Check masked` ステップではマスクされていることが確認できます。

▼ **図 3-1**

![ジョブのログを表示し、コマンド実行前と後の表示を赤枠で示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9acdfa78d61345f5979dc39e4b98922d/add-mask-command-in-github-actions-simple.png?w=842\&h=612\&auto=compress%2Cformat)

::: message

コマンドと文字列を連結してファイルに書き出してから `cat` をで出力していますが、これは[コマンドライン引数でマスクする文字列を扱わないための対策です](https://docs.github.com/ja/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow)。

> 可能であれば、コマンドラインからプロセス間でシークレットを渡すのは避けてください。

:::

### 有効範囲

上記ジョブの後で実行されるジョブでも確認してみます。

▼ **リスト 3-2**

```yaml
check-in-other-job:
  needs: simple-add-mask
  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@v2

    - name: Check
      run: echo 'ultra super secret'
```

時系列的にはコマンド実行後ですがマスクされていないことが確認できます。

▼ **図 3-2**

![前の図とは異なるジョブのログを表示し、マスクされない文字列を赤枠で示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4ee69a5137884181a4f6f8f19f4f43fe/add-mask-command-in-github-actions-other.png?w=813\&h=347\&auto=compress%2Cformat)

これは runner のインスタンスが分かれているためだと思われます。よって、ジョブ間で値を受け渡す場合は再設定する必要があるようです。

### アンチパターン？

「マスクしたい文字列が確定したステップ」と「マスクするコマンドを実行するステップ」が分かれていたとします。このときに `:env` フィールドを使うと少し問題があります。

▼ **リスト 3-3**

```yaml
anti-add-mask:
  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@v2

    - name: Set secet text to outputs
      id: get_secret
      run: echo '::set-output name=SECRET_TEXT::ultra super secret'

    - name: Add mask
      run: echo "::add-mask::$SECRET_TEXT"
      env:
        SECRET_TEXT: ${{ steps.get_secret.outputs.SECRET_TEXT}}

    - name: Check masked
      run: echo 'ultra super secret'
```

この場合はマスクされる前にフィールドへ設定した内容が表示されてしまいます。また、「通常の設定」のところでも書きましたが、コマンドラインの引数でマスクする文字列を扱っているのもあまり良いことではありません。

▼ **図 3-3**

![ジョブのログを表示し、環境変数の内容が表示されている部分を赤枠で示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d2c50a2ab769431295f71c1fda167a2c/add-mask-command-in-github-actions-anti.png?w=785\&h=538\&auto=compress%2Cformat)

上記の例は少し無理やり状況を発生させていますが、ステップ間の受け渡しは値が表示されやすいので注意が必要です(Action の入力である `with:` でも同様のことが発生します)。

マスクしたい文字列がある場合は「値が確定したステップ内で実施する」か、他ステップで実施する必要がある場合は「ファイルなどで渡す」のがよさそうです。

### 他コマンドで利用

以下が少し気になったのでマスクした文字列を `::set-output` で利用してみました。

> When you mask a value, it is treated as a secret and will be redacted on the runner. For example, after you mask a value, you won't be able to set that value as an output.

@[card](https://docs.github.com/ja/actions/using-workflows/workflow-commands-for-github-actions#masking-a-value-in-log)

マスク前と後で `::set-output` した値の比較はハッシュ値を表示することで対応ています。

▼ **リスト 3-4**

```yaml
use-in-other-command:
  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@v2

    - name: Check not masked
      run: echo -n 'ultra super secret' | sha256sum

    - name: Set secet text to outputs
      id: out_not_masked
      run: echo '::set-output name=SECRET_TEXT::ultra super secret'

    - name: Check outputs
      run: echo -n "${SECRET_TEXT}" | sha256sum
      env:
        SECRET_TEXT: ${{ steps.out_not_masked.outputs.SECRET_TEXT}}

    - name: Add mask
      run: |
        echo -n '::add-mask::' > cmd.txt
        cat secret_text.txt >> cmd.txt
        cat cmd.txt
        rm cmd.txt

    - name: Set secet text to outputs
      id: out_masked
      run: echo '::set-output name=SECRET_TEXT::ultra super secret'

    - name: Check outputs
      run: echo -n "${SECRET_TEXT}" | sha256sum
      env:
        SECRET_TEXT: ${{ steps.out_masked.outputs.SECRET_TEXT}}
```

結果としてもどちらも同じハッシュ値になりました。

▼ **図 3-4**

![ジョブのログを表示し、ハッシュ値が表示されている部分を赤枠で示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d8358cde5b9b4731a007588000fcb016/add-mask-command-in-github-actions-other-command.png?w=853\&h=686\&auto=compress%2Cformat)

少しスッキリしないのですが、`$ gh run view --log` などで取得したログから値を取り出せないという話なのか、他の落とし穴的なものがあるのかは不明です。

## おわりに

ワークフロー実行中に動的に文字列をマスクする方法を試してみました。

少し扱いに注意するところもありますが、bash からも実行できることから利用できる局面は多いと思われます。

クレデンシャル以外にも「ワークフロー実行中に確定した値」をマスクしたい場合もあるので[^file-id]、この辺の機能は活用していきたいところです。

[^file-id]: 今回の API 知ったとき、Google Drive へ Upload したファイルの id を他ステップに渡したいと考えていたところでした。

[tibdex/github-app-token]: https://github.com/tibdex/github-app-token

[actions/toolkit]: https://github.com/actions/toolkit

[`@actions/core`]: https://github.com/actions/toolkit/tree/main/packages/core
