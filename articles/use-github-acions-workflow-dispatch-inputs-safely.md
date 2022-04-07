---
id: use-github-acions-workflow-dispatch-inputs-safely
title: GitHub Actions の手動実行で入力された値を安全に使いたい
emoji: 🔣
type: tech
topics:
  - githubactions
  - bash
published: true
---

GitHub Actions の手動実行で値の入力(Workflow Dispatch Inputs)を使ってみたところ、以下の点が気になったので少し試してみました。

*   入力した値をマスクするオプション的なものがなかった
*   ドキュメントが入力値の式構文(`${{ github.event.inputs.tags }}` など)を `:run` に直接記述していた

## 基本的な入力を試してみる

まずは基本的なワークフローを作成し、実行方法(入力方法)などを試してみます。

なお、大まかな挙動と定義などは以下を参考にさせていただきました。

@[card](https://zenn.dev/kesin11/articles/13ca0f40e1eaa0)
@[card](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions#onworkflow_dispatchinputs)

以下は各入力値を `echo` で表示するワークフローです。

▼ **リスト 1-1 基本的なワークフロー**

```yaml:.github/workflows/test.yaml
name: Test inputs
on:
  workflow_dispatch:
    inputs:
      text:
        type: string
        required: true
        description: 'Text data'
      optional:
        type: string
        required: false
        description: 'Optional data'
      secret:
        type: string
        required: true
        description: 'Secret data'
      flag:
        type: boolean 
        required: true 
        description: 'Flag data'

jobs:
  simple-print:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Print inputs
        run: |
          echo "${TEXT_DATA}"
          echo "${OPTIONAL_DATA}"
          echo "${SECRET_DATA}"
          echo "${FLAG_DATA}"
        env:
          TEXT_DATA: ${{ github.event.inputs.text }}
          OPTIONAL_DATA: ${{ github.event.inputs.optional }}
          SECRET_DATA: ${{ github.event.inputs.secret }}
          FLAG_DATA: ${{ github.event.inputs.flag }}
```

ワークフロー手動実行は [GitHub CLI] からも可能です。今回は入力の手間を軽減するためにこちらを利用します。

まず、入力の設定にあわせて以下のような JSON ファイルを用意します。

*   必須ではない項目の挙動を確認するため `optional` フィールドは省略してあります
*   `flag` フィールドも文字列で表現しないとエラーになります[^parse-err][^boolean]

▼ **リスト 1-2 テスト用の入力データ**

```json:test-data.json
{
  "text": "M & M's",
  "secret": "qqqqq",
  "flag": "true"
}
```

[^parse-err]: エラーメッセージは「could not parse provided JSON: json: cannot unmarshal bool into Go value of type string」。

[^boolean]: おそらく、[@actions/core の getBooleanInput()](https://github.com/actions/toolkit/tree/main/packages/core#inputsoutputs) に近い扱いと思われます。

続いてワークフローの ID を調べます。

```shell-session
$ gh workflow list
Test inputs  active  23245105
```

最後に以下のように入力すると JSON ファイルが入力として利用されワークフローが開始されます(`--ref` はそのとき作業しているブランチ名などです) 。

```shell-session
$ gh workflow run 23245105 --json < test-data.json --ref topic/workflow4
✓ Created workflow_dispatch event for test.yaml at topic/workflow4

To see runs for this workflow, try: gh run list --workflow=test.yaml
```

以下がワークフローの実行結果になります。

▼ **図 1-1 実行結果**

![入力した値が表示されているワークフロー実行結果のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1885ccb5f2d2496f83baa7a4c96c5a35/use-github-acions-workflow-dispatch-inputs-safely-simple.png?w=547\&h=596\&auto=compress%2Cformat)

JSON で設定した内容が反映されていることが確認できます。

## JSON で表示する

上記の結果でも大筋で問題はないのですが、`echo` で個別に表示しているため `flag` の型などがわかりません。

そこで入力値全体を JSON で表示してみます。方法はいくつかありますが、今回はコマンドライン引数で入力値を扱わないように、以下を参考にファイルから JSON を取りだすジョブを追加します。

@[card](https://stackoverflow.com/questions/67608874/github-actions-how-to-mask-workflow-dispatch-inputs-like-secrets)

▼ **リスト 2-1 JSON で表示するジョブ**

```yaml:.github/workflows/test.yaml
  json-print:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Print json
        run: jq < "${GITHUB_EVENT_PATH}" -r .inputs
```

結果は以下のようになりました。

*   `flag` は文字列になっている
*   `optional` はフィールドが存在しない

▼ **図 2-1 JSONで値を表示**

![入力した値が JSON として表示されているワークフロー実行結果のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7d05202bb5c44b74b0bc806eba10cd2e/use-github-acions-workflow-dispatch-inputs-safely-jso.png?w=683\&h=459\&auto=compress%2Cformat)

おおざっぱな理解でいくと「型は基本的に文字列」「入力されなかったフィールドは省略」といったところでしょうか。

## マスクする

型の扱いなどがわかりましたが、やはり `secret` が表示されているのは安全面で気になります。

実際の入力でもログに表示させたくない項目は出てくるので、[`::add-mask` ] コマンドでマスクしてみます。これも「JSON で表示する」で参考にした内容を元にコマンドライン引数で値を扱わないように処理しています。

▼ **リスト 3-1 マスクするジョブ**

```yaml:.github/workflows/test.yaml
  mask-print:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Mask secret
        run: |
          echo -n '::add-mask::' > cmd.txt
          jq < "${GITHUB_EVENT_PATH}" -r .inputs.secret >> cmd.txt
          cat cmd.txt
          rm cmd.txt

      - name: Print inputs
        run: |
          echo "${TEXT_DATA}"
          echo "${OPTIONAL_DATA}"
          echo "${SECRET_DATA}"
          echo "${FLAG_DATA}"
        env:
          TEXT_DATA: ${{ github.event.inputs.text }}
          OPTIONAL_DATA: ${{ github.event.inputs.optional }}
          SECRET_DATA: ${{ github.event.inputs.secret }}
          FLAG_DATA: ${{ github.event.inputs.flag }}
```

結果は以下のように `secret` フィールドがマスクされました。

▼ **図 3-1 マスクされた実行結果**

![マスク処理と結果が表示されているワークフロー実行結果のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a6cce410c6514f238bd4a722a0f6a4d1/use-github-acions-workflow-dispatch-inputs-safely-mask.png?w=750\&h=771\&auto=compress%2Cformat)

念のためにテキストデータでも確認しましたが元の文字列は存在しませんでした。

```shell-session
$ gh run view 2090592733

✓ topic/workflow4 Test inputs #4 · 2090592733
Triggered via workflow_dispatch about 1 hour ago

JOBS
✓ mask-print in 3s (ID 5818368696)
✓ quote-print in 2s (ID 5818368857)
✓ json-print in 3s (ID 5818368998)
✓ simple-print in 3s (ID 5818369150)

For more information about a job, try: gh run view --job=<job-id>
View this run on GitHub: https://github.com/hankei6km/test-gha-workflow-dispatch-inputs/actions/runs/2090592733

$ gh run view 2090592733 --job 5818368696 --log | grep qqqqq

$ echo "${?}"
1
```

## OS コマンドインジェクションを避ける

::: message

以下は bash を利用している場合での内容です。

:::

安全面でいうと `type:string` の項目は自由に入力できてしまうので、`:run` などの記述によっては少し危険な状態になります。

[ワークフロー構文のドキュメントで](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_dispatchinputs)は式構文を `:run` へ直接記述していますが、[シークレット関連のドキュメントによると](https://docs.github.com/en/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow)このような場合は環境変数を利用しクオートするよう記載されています。

> If you must pass secrets within a command line, then enclose them within the proper quoting rules. Secrets often contain special characters that may unintentionally affect your shell. To escape these special characters, use quoting with your environment variables.

外部からの入力という意味では同じなので、これを参考に以下のような 2 つのステップを作成しました。

*   `:run` の中に[ワークフローの式構文]を記述しそれをクオート
*   `:env` の中に[ワークフローの式構文]を記述し `:run` では環境変数をクオート

▼ **リスト 4-1 クオートを試すジョブ**

```yaml:.github/workflows/test.yaml
  quote-print:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Print inputs
        run: echo "${{ github.event.inputs.text }}"

      - name: Print inputs via env
        run: echo "${TEXT_DATA}"
        env:
          TEXT_DATA: ${{ github.event.inputs.text }}
```

最初はこれまと同じデータで試してみます。

▼ **図 4-1 実行結果(正常に表示される)**

![ワークフロー実行結果のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2725863ee35a47b382ea00e29f2316f1/use-github-acions-workflow-dispatch-inputs-safely-quote.png?w=744\&h=513\&auto=compress%2Cformat)

データには `&` や `'` が存在していますが、どちらも `M & M's` を正しく表示しています。

次は入力値を `M & M's" && ls "/etc` へ置き換えて試してみます。ここでは入力をわかりやすくするためにウェブ UI から実行します。

▼ **図 4-2 クオートを回避する入力**

![手動実行用のフォームが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d656caa070d44e968d898b41f3256402/use-github-acions-workflow-dispatch-inputs-safely-inject-input.png?w=336\&h=400\&auto=compress%2Cformat)

[ワークフローの式構文]を直接記述した場合、シェル(bash) が入力値内の `"` を解釈してしまうので `ls /etc` が実行されてしまいました。

▼ **図 4-3 実行結果(直接記述)**

![入力した値がコマンドとして実行されているワークフロー実行結果のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6c245a0d0fb640feb23240c8f6356706/use-github-acions-workflow-dispatch-inputs-safely-inject-expression.png?w=756\&h=705\&auto=compress%2Cformat)

一方で環境変数を経由した場合は正しく表示されています。

▼ **図 4-4 実行結果(環境変数経由)**

![入力した値が文字列として表示されているワークフロー実行結果のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d1b033716db84ad3bdc3b7b2f0cd6bf1/use-github-acions-workflow-dispatch-inputs-safely-inject-env.png?w=751\&h=460\&auto=compress%2Cformat)

これは[ワークフローの式構文]を直接 `:run` に記述するとステップ実行時の挙動が以下のようになるためです。

1.  `:run` 内の式構文を文字列へ展開する
2.  展開された後の文字列がシェルへ渡される

回避方法としては以下が考えられます。

*   環境変数をクオートして使う[^quote-env]

    *   可能であれば環境変数はコマンドライン引数ではなくプロセス内で処理する

*   「JSON で表示する」のようにファイルから値を取り出してパイプで処理する

*   ホワイトリスト的な処理を挟む

    *   `type:choice` などで入力内容を限定する
    *   [workflow\_dispatchの誤実行防止策にinputsでの絞りを試してみた | DevelopersIO](https://dev.classmethod.jp/articles/workflow-dispatch-fail-safe-by-inputs/)

[^quote-env]: 厳密にいうとクオートは glob などへの対策なのですが実施しておく方が固いと思われます。<https://github.com/koalaman/shellcheck/wiki/SC2086>

## その他

### [actionlint] と [ShellCheck]

[actionlint] を利用すると bash と sh で動く `:run` に [ShellCheck] を実行してくれます。これは環境変数のクオートを忘れた場合などで警告が出るので便利ですが、今回のような GitHub Actions 固有のものにはやはり反応しませんでした。

▼ **図 5-1 環境変数のクオート忘れ**

    ../.github/workflows/test.yaml:93:9: shellcheck reported issue in this script: SC2086:info:1:6: Double quote to prevent globbing and word splitting [shellcheck]
       |
    93 |         run: echo ${TEXT_DATA}
       |         ^~~~

### `type:number` は？

ドキュメントにはなかったのですが、時々 `type:number` が使われているのを見かけました。しかし、少し試してみたところ [actionlint] では警告がでます。また、実際に動かしてみると数字以外も受け付けてしまうので、入力値の制限には利用できなさそうです。

▼ **図 5-2 number では警告が出る**

    $ ./actionlint ../.github/workflows/test.yaml
    ../.github/workflows/test.yaml:10:15: input type of workflow_dispatch event must be one of "string", "boolean", "choice", "environment" but got "number" [syntax-check]
       |
    10 |         type: number
       |               ^~~~~~

## おわりに

ワークフロー手動実行の入力値を安全に扱う方法を試してみました。

値のマスクは JSON ファイルを使う方法で解決できそうなので、これを利用していこうかと考えています。

インジェクション対策については手動実行に限らず「式構文と `:run` の扱い」全般的な話でもあるので、状況に応じて対策を講じていきたいところです。

[GitHub CLI]: https://cli.github.com/

[@actions/core]: https://github.com/actions/toolkit/tree/main/packages/core

[ワークフローの式構文]: https://docs.github.com/ja/actions/learn-github-actions/expressions

[`::add-mask`]: https://docs.github.com/ja/actions/using-workflows/workflow-commands-for-github-actions#masking-a-value-in-log

[actionlint]: https://github.com/rhysd/actionlint

[ShellCheck]: https://github.com/koalaman/shellcheck
