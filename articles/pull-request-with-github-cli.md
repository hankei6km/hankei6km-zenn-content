---
id: pull-request-with-github-cli
title: GitHub CLI による一人プルリクエスト
emoji: 🏃
type: idea
topics:
  - githubcli
published: true
---

[GitHub CLI] を利用した「自分でプルリクエスト作成し、チェックからマージまで自分で行う[^hitori]」ときの操作が固まってきたのでメモなど。

[^hitori]: 今回はこれを一人プルリクエストと勝手に呼んでいます。

:::message
2022-04-26 `gh run rerun` の記述を `--failed` 付きに変更しました。
:::

## プルリクエストの作成

ブランチでコミットした後に `gh pr create` で作成しています。タイトルなどはオプションを使わずメニューで指定していますが、ラベルは "Add metadata" からはなく `-l` を使うことが多いです[^completion]。

[^completion]: 少し宣伝など「[GitHub CLI でラベル名の入力補完をできるようにした](https://zenn.dev/hankei6km/articles/completion-label-names-in-github-cli)」。

```shell-session
$ gh pr create -l dependencies
? Where should we push the 'topic/update-parse5' branch? hankei6km/gas-feed2notion

Creating pull request for topic/update-parse5 into main in hankei6km/gas-feed2notion

? Title Update `parse5` to 7.0.0
? Body <Received>
? What's next?  [Use arrows to move, type to filter]
> Submit
  Add metadata
  Cancel
```

## プルリクエストのチェック状況を確認

pull request のチェック状況は `gh pr checks` で確認できますが、各チェックが終了するまで何度かコマンドを実行するのは手間です。これには `--watch` オプションでチェック完了を監視することが多いです。

```shell-session
$ gh pr checks --watch

Refreshing checks status every 10 seconds. Press Ctrl+C to quit.

Some checks are still pending
0 failing, 2 successful, 0 skipped, and 1 pending checks

*  push...         https://github.com/hankei6km/gas-feed2notion/runs/6059983820?check_suite_focus=true
✓  Anal...  1m44s  https://github.com/hankei6km/gas-feed2notion/runs/6059912448?check_suite_focus=true
✓  CodeQL   2s     https://github.com/hankei6km/gas-feed2notion/runs/6059926888

All checks were successful
0 failing, 3 successful, 0 skipped, and 0 pending checks

✓  Anal...  1m44s  https://github.com/hankei6km/gas-feed2notion/runs/6059912448?check_suite_focus=true
✓  CodeQL   2s     https://github.com/hankei6km/gas-feed2notion/runs/6059926888
✓  push...  53s    https://github.com/hankei6km/gas-feed2notion/runs/6059983820?check_suite_focus=true
```

## ワークフローの実行状況を確認

上記はプルリクエストが作成されていないと行えませんが、その他の状況で開始されたワークフローも `gh run wath` で終了を監視できます(こちらはオプションではなくサブコマンド)。

こちらはコマンドを入力するとリポジトリで実行中のワークフローが表示されて選択するようになります。同時に 1 つのワークフローしか選択できませんが、JOB と STEP 単位の進捗状況が表示されます[^tmux]。

STEP が多いときや `.yml` の内容を変更したときなどで利用しています。

[^tmux]: コマンドを複数実行すれば同時に複数の実行状況も確認できます。

```shell-session
$ gh run watch
? Select a workflow run  [Use arrows to move, type to filter]
> * Edit description of `pacakge.json`, CodeQL (topic/edit-description) 6s ago
  * Edit description of `pacakge.json`, Push to GAS project (topic/edit-description) 7s ago

Refreshing run status every 3 seconds. Press Ctrl+C to quit.

* topic/edit-description Push to GAS project #11 · 2182045135
Triggered via push less than a minute ago

JOBS
* push-to-gas (ID 6058216405)
  ✓ Set up job
  ✓ Run actions/checkout@v2
  ✓ Use Node.js 14.x
  ✓ Cache node modules
  * Install clasp
  * Install modules
  * Run tests
  * Build files to push gas project
  * Setup files for clasp
  * Push source files to gas project
  * Cleanup files for clasp
  * Post Cache node modules
  * Post Run actions/checkout@v2
```

## 実行に失敗した STEP のログを表示

`gh run view --log-failed` を使うことでエラーになった STEP のログが表示されます。これも run-id や job-id を指定しなければ一覧から選択できます。

STEP からの出力が多いとスクロールしてしまいますが、端末上ではエラー箇所が色分けされているのでエラー箇所は判別しやすくなっています。

```shell-session
$ gh run view --log-failed
? Select a workflow run  [Use arrows to move, type to filter]
  ✓ Bump parse5 from 6.0.1 to 7.0.0, CodeQL (dependabot/npm_and_yarn/parse5-7.0.0) 1h39m13s ago
> X Bump parse5 from 6.0.1 to 7.0.0, Push to GAS project (dependabot/npm_and_yarn/parse5-7.0.0) 1h39m14s ago
  ✓ Merge pull request #14 from hankei6km/dependabot/npm_and_yarn/rollup/plugin-node-resolve-13.2.1, CodeQL (main) 70h41m48s ago

push-to-gas     Run tests       2022-04-21T04:47:46.5716882Z ##[group]Run npm run test:build
push-to-gas     Run tests       2022-04-21T04:47:46.5717171Z npm run test:build
push-to-gas     Run tests       2022-04-21T04:47:46.5766639Z shell: /usr/bin/bash -e {0}
push-to-gas     Run tests       2022-04-21T04:47:46.5766877Z ##[endgroup]
push-to-gas     Run tests       2022-04-21T04:47:46.7748273Z

  <snip>

push-to-gas     Run tests       2022-04-21T04:47:47.9152997Z
push-to-gas     Run tests       2022-04-21T04:47:47.9154872Z ./src/main.ts → build...
push-to-gas     Run tests       2022-04-21T04:47:52.0783098Z (!) Circular dependency
push-to-gas     Run tests       2022-04-21T04:47:52.0784254Z node_modules/hast-util-select/lib/any.js -> node_modules/hast-util-select/lib/pseudo.js -> node_modules/hast-util-select/lib/any.js
push-to-gas     Run tests       2022-04-21T04:47:52.0785244Z [!] Error: 'default' is not exported by node_modules/parse5/dist/index.js, imported by src/util.ts

  <snip>
```

## 実行に失敗した JOB の再実行

失敗した原因が push したコードではなく他の要因だった場合、原因を取り除いてから再実行したいことがあります。

そのようなときは `gh run rerun --failed` を run-id の指定なしで使うと実行に失敗したワークフローの一覧から再実行でます。このとき `--failed` を付けているので、実行されるのは failed な JOB になります(ウェブ UI での Re-run failed jobs と同じ)。

environment の設定を間違えていたときや、[dependabot] の bump が secret 関連で失敗しているときなどに利用しています。

```shell-session
$ gh run rerun
? Select a workflow run  [Use arrows to move, type to filter]
> X Bump @rollup/plugin-node-resolve from 13.2.0 to 13.2.1, Push to GAS project (dependabot/npm_and_yarn/rollup/plugin-node-resolve-13.2.1) 4m6s ago
  X Bump rollup-plugin-license from 2.6.1 to 2.7.0, Push to GAS project (dependabot/npm_and_yarn/rollup-plugin-license-2.7.0) 124h30m36s ago
```

## プルリクエストのマージ

`gh pr merge` で行えますがブランチは消しておきたいので、 `gh pr merge -m -d` を実行後に `git fetch origin --prune` しています。

## dependabot の bump

ここも自動化などはあまり行っていないので、`gh pr comment -b '@dependabot merge'` のようにコメントで操作しています。

しかし、複数の bump があるときに「1 つをマージしたあとに他が自動で rebase される」条件がわからなかったのでウェブ UI や[GitHub Mobile]を使うこともわりとあります。

[GitHub CLI] で図 7-1 の force-pushed と表示されてる部分[^view]を取得できると確認しやすくなりそうですが、いまのところ方法がわかっていません。

[^view]: この部分の名称自体もわからないので検索も難儀しています。

▼ **図 7-1 自動的に rebase 的な操作が行われる**

![dependabot の bump で作成されたプルリクエストで bot の操作履歴が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/67fa71fc86394c7b89bd74c95cc9c30b/pull-request-with-github-cli-force-pushed.png?w=859\&h=194\&auto=compress%2Cformat)

## おわりに

実は、少し前に「`rerun` などでは一覧から対象を選択できる」ことに遅まきながら気が付きました。これにより再実行などを含めた一連の操作がやりやすくなったので記事にしてみました。

[dependabot]: https://docs.github.com/ja/code-security/dependabot

[GitHub CLI]: https://cli.github.com/

[GitHub Mobile]: https://github.com/mobile
