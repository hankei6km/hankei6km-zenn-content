---
id: copilot-cli-seems-to-recognize-japanese-queries
title: Copilot for CLI は日本語クエリーの夢を見るか？
emoji: 🐑
type: idea
topics:
  - github
  - githubcopilot
  - cli
published: true
---

GitHub Copilot for CLI のウェイトリストに登録しておいたら順番がきたので少し試してみました。

@[card](https://githubnext.com/projects/copilot-cli/)

まだ数日しか利用していませんが、`git` の操作をラフな日本語で指示できたりと良い感じです。

```shell-session
$ git? package.json と package-lock.json をHEADに戻す
```

ということで、今回は Copilot for CLI を主に日本語で使ってみた内容についてなど。

## Copilot for CLI とは？

Copilot X を構成する AI サービスの 1 つで、ターミナルでの操作をサポートしてくれるツールです。

詳細は GitHub の記事などを見ていただくのが速いのですが、簡単に書くと「いくつか質問するだけで AI がコマンドラインを組み立てる」「その場で実行も指示できる」といったツールです。

@[card](https://github.blog/2023-03-22-github-copilot-x-the-ai-powered-developer-experience/)

現在(2023-03-29)、Copilot for CLI を利用するにはウェイトリストへ登録する必要があります。

## インストールとセットアップ

順番待ちが解除されたときのメールには NPM のパッケージをインストールするようにありました。Node.js の環境があれば一般的な手順でインストールできます。

@[card](https://www.npmjs.com/package/@githubnext/github-copilot-cli)

**図 2-1 npm コマンドでグローバルへインストール**

```shell-session
$ npm install -g @githubnext/github-copilot-cli
```

ただし、利用には Copilot へ接続するための認証が必要です。認証コマンドを実行するとコードとアドレスが表示されるので、それに従うとホームディレクトリに資格情報が保存され Copilot for CLI を利用できるようになります。

**図 2-2 認証用のコマンドを実行**

```shell-session
$ github-copilot-cli auth
Copy this code: ****-****

Then go to https://github.com/login/device, paste the code in and approve the access.
⠧ It will take up to 5 seconds after approval for the token to be retrieved
```

あとは、必須ではないのですが、[デモ動画](https://githubnext.com/projects/copilot-cli/)のように `??` でクエリーを入力する場合は、下記のようにエイリアスを設定することになります。 (永続化する場合は `~/.bashrc` などへ追加しておくとよいかと思います)

**リスト 2-1 ****`??`**** などのエイリアス追加**

```bash
eval "$(github-copilot-cli alias -- "$0")"
```

::: message

**GITHUB\_TOKEN**

資格情報としては `$GITHUB_TOKEN` も使えるようです。試しに Codespaces で使ってみたところ、デフォルトで設定されている `$GITHUB_TOKEN` の権限で利用できました。

:::

::: message

**資格情報の保存場所**

下記の 2 ファイルに保存されているようです(リネームしたら再認証するように言われる)。

**リスト 2-2 資格情報が保存されていると思われるファイル**

    ~/.copilot-cli-access-token
    ~/.copilot-cli-copilot-token

:::

## 基本的な利用方法

セットアップが完了したので簡単なクエリーを入力してみます。単純な場合は `??` に続けて「**list json files**」のよう入力します。

**図 3-1 デモ動画と同じようなクエリー**

```shell-session
$ ?? list json files

 ──────────────────── Command ────────────────────

find . -name "*.json"

 ────────────────── Explanation ──────────────────

○ find is used to list files.
  ◆ . specifies that we search in the current directory.
  ◆ -name "*.json" stipulates that we search for files ending in .json.

  ✅ Run this command
  📝 Revise query
❯ ❌ Cancel
```

ここで `Run this command` を選択するとそのまま実行されます。

クエリーを続けたい場合は `Revice query` を利用すると追加できます。

**図 3-2 追加で「count lines」を入力**

```shell-session
$ ?? list json files

 ──────────────────── Command ────────────────────

find . -name "*.json"

 ────────────────── Explanation ──────────────────

○ find is used to list files.
  ◆ . specifies that we search in the current directory.
  ◆ -name "*.json" stipulates that we search for files ending in .json.

 ──────────────────── Revision ────────────────────

Enter your revision: count lines
```

`??` の他に `git` の操作に特化した `git?` や GitHub CLI 用の `gh?` なども使えます。

**図 3-3 ****`git?`**** の利用例**

```shell-session
$ git? show merge commits history

 ──────────────────── Command ────────────────────

git log --merges

 ────────────────── Explanation ──────────────────

○ git log is used to list commits.
  ◆ --merges specifies that we only want to list merge commits.

  ✅ Run this command
  📝 Revise query
❯ ❌ Cancel
```

ただし、実際の状況を考慮した生成は行われないようです。たとえば、クエリーによってはインストールされていないコマンドが利用されることもありました。

::: message

**コマンド履歴**

`Run this command `を使うとシェルの履歴には記録されません。実行後に少しだけ修正して実行したいような場合、現状では tmux などでコピーすることになるかと思います。

:::

## 英語以外のクエリー

試しにクエリーを日本語で「**json ファイルの一覧**」にしても上記と同じコマンドが生成されます。ただし、若干の注意点もあります。

*   コマンド実行時の `(y/N)` 選択などで IME をオフにする手間が増える
*   `Revise query` で追加のクエリーを入力するときに入力中の文字(プレエディット)は表示されない
*   説明は英語のまま固定と思われる

**図 4-1 日本語によるクエリー指定**

```shell-session
$ ?? json ファイルの一覧

 ──────────────────── Command ────────────────────

find . -name "*.json"

 ────────────────── Explanation ──────────────────

○ find is used to list files.
  ◆ . specifies that we search in the current directory.
  ◆ -name "*.json" stipulates that we search for files ending in .json.

  ✅ Run this command
  📝 Revise query
❯ ❌ Cancel
```

日本語の他にはドイツ語(正しいドイツ語かは不明)の「**liste der json**」でも同じようになりました。

**図 4-2 ドイツ語によるクエリー指定**

```shell-session
$ ?? liste der json

 ──────────────────── Command ────────────────────

find . -name "*.json"

 ────────────────── Explanation ──────────────────

○ find is used to list files.
  ◆ . specifies that we search in the current directory.
  ◆ -name "*.json" stipulates that we search for files ending in .json.

  ✅ Run this command
  📝 Revise query
❯ ❌ Cancel
```

また、`Revise query` への入力は「**空白区切り**」「**sum**」のような単語の入力でも概ね通るようです(もちろん、詳しく入力する方がより意図に沿ったコマンドが生成されやすいです)。

試してみる前は「英語で文章入力は厳しい」と思っていたのですが、日本語や単語ぶつ切りで入力してもわりと大丈夫そうな感じです。

::: message

**英語の方が良い結果になるのでは？**

[こちら](https://zenn.dev/riya_amemiya/articles/558e30b8875d7a#%E4%B8%8A%E6%89%8B%E3%81%8F%E3%81%84%E3%81%8B%E3%81%AA%E3%81%8B%E3%81%A3%E3%81%9F%E4%BA%8B)でも書かれていますが、英語のクエリーでないとエラーになることが何度かありました。また、英語と日本語のクエリーで結果が異なることもあります。

*   「**wipe first column from test.txt**」は `cut -d " " -f 2- test.txt` を生成
*   「**test.txt から最初のカラムを除去**」は `cut -f 2- test.txt` を生成

このように書くと「英語の方が良い」となりそうですが、上記の例でいうとファイルの内容によっては日本語クエリーの方が意図した動作になることもあります。 (Copilot for CLI は実際のファイルの内容などは確認しない)

この辺は、英語と日本語どちらか一方しか使えないというわけでもないので、使い分けていくのがよいかと思います。

**図 4-3 途中から日本語で入力**

```shell-session
$ ?? transpile ts files

 ───────────────────── Query ─────────────────────

1) transpile ts files 2) デバッグ情報あり

 ──────────────────── Command ────────────────────

tsc --sourceMap
```

:::

::: message

**クエリーにローマ字**

日本語入力を切り替える手間を軽減するために、ローマ字を使う方法も試してみました。

こちらは一見うまくいくのですが、「`fairu` ではなく `file` を使うとエラーになる」など成功率があまり高くありませんでした。

*   「**json fairu no ichiran**」はコマンドが生成される
*   「**json file no ichiran**」はエラー

おそらくですが、「**no**」の直前に英単語があるとローマ字として扱わないのだと思われます。

:::

::: message

**日本語入力を自動的にオフ**

コマンドラインから IME を制御できる環境の場合、`$PROMPT_COMMAND` を利用するとシェルのプロンプトが表示されるときに自動でオフにできます。`(y/N)` 選択時での効果はありませんが、若干は楽になるかと思います。

**リスト 4-1 `PROMPT_COMMAND` 設定例(`.bashrc` 等に記述する)**

```bash
function prompt_func {
  # ここに日本語入力をオフにするコマンド
}
PROMPT_COMMAND="prompt_func"
```

:::

## 少し実際的な場合の例

コーディングしている環境へ Copilot for CLI をインストールしておいて利用した例です。

コーディングしていると `jest` のオプションを調べたくなることはわりとあります。

```shell-session
$ ?? jest で page を含むテスト名のみ実行

 ──────────────────── Command ────────────────────

jest --testNamePattern page

 ────────────────── Explanation ──────────────────

○ jest is the test runner for JavaScript.
  ◆ --testNamePattern page specifies that we want to run tests whose name contains the string page.

❯ ✅ Run this command
  📝 Revise query
  ❌ Cancel
```

これだけでも `--help` を参照したりググる手間が省けるのでうれしいのですが、`jest` は NPM スクリプトから実行していることが多くでそのままでは使えません。

ここでオプションを手動でコピーしてもよいのですが、Copilot for CLI ではクエリーに「**npm run test 経由で実行**」を入れることで対応してくれます。個人的には `npm run test` の後に `--` を付けるのを忘れて「なんでエラーになるんだ？」で悩むことが多く、オプションを調べた後にそのまま `npm` で実行できるのは結構うれしい感じです。

**図 5-1 ****`jest`**** のオプションを調べてそのまま実行**

```shell-session
$ ?? jest で page を含むテスト名のみ実行

 ───────────────────── Query ─────────────────────

1) jest で page を含むテスト名のみ実行 2) npm run test 経由で実行

 ──────────────────── Command ────────────────────

npm run test -- --testNamePattern page

 ────────────────── Explanation ──────────────────

○ npm run test runs the test script defined in package.json.
  ◆ -- is used to separate the arguments to npm from the arguments to the script.
  ◆ --testNamePattern page is passed to the script as an argument.

🕛  Hold on, executing commmand...
```

また、 `git` の操作で「`reset` と `checkout` のどちらか？」で迷ったときも「**package.json と package-lock.json をHEADに戻す**」のように指示できました。

**図 5-2 ****`git`**** に日本語で指示**

```bash
$ git status
On branch topic/rollup-config-js-as-esm
Your branch is up to date with 'origin/topic/rollup-config-js-as-esm'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   package-lock.json
        modified:   package.json
        modified:   rollup.config.js

no changes added to commit (use "git add" and/or "git commit -a")

$ git? package.json と package-lock.json をHEADに戻す

 ──────────────────── Command ────────────────────

git checkout HEAD -- package.json package-lock.json

 ────────────────── Explanation ──────────────────

○ git checkout is used to restore files from a previous commit.
  ◆ HEAD specifies that we want to restore the files from the current commit.
  ◆ -- package.json package-lock.json specifies the files we want to restore.

🕙  Hold on, executing commmand...

$ git status
On branch topic/rollup-config-js-as-esm
Your branch is up to date with 'origin/topic/rollup-config-js-as-esm'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   rollup.config.js

no changes added to commit (use "git add" and/or "git commit -a")
```

Copilot for CLI を試し始めて数日ですが、この辺がサポートされるだけでも「Copilot はお試し期間終わっても使おうかな」とちょっとグラグラきています。

## よろしくないコマンドが混ざることもある

ここまでは良い感じですが、やはり AI 生成の場合はどうしてもよろしくないコマンドが混ざってきます。

たとえば、現行(2023-03-29 時点)の Copilot for CLI では `find` と `xargs` の組み合わせがあまりよろしくないようです。Copilot for CLI が生成するコマンドでは「ファイル名に特殊な意味を持つ文字が含まれている」とエラーになります。

**図 6-1 入力によってはエラーになることも**

```shell-session
$ ?? .jsonファイルの一覧

 ───────────────────── Query ─────────────────────

1) .jsonファイルの一覧 2) 各ファイルの行数を数える

 ──────────────────── Command ────────────────────

find . -name "*.json" | xargs wc -l

 ────────────────── Explanation ──────────────────

○ find is used to list files.
  ◆ -name "*.json" stipulates that we search for files ending in .json.
○ | xargs passes the list of files to the xargs command.
  ◆ wc -l counts the number of lines in each file.

🕔  Hold on, executing commmand...
wc: ./123: そのようなファイルやディレクトリはありません
wc: 456.json: そのようなファイルやディレクトリはありません
1 ./789.json
1 合計
```

パっと見で「これは意図した通りに実行されない」とわかる場合は良いのですが、「普段は動くけど少し変わった入力では不具合が出る」というときに少し困りそうです。

この辺は仕方のない部分ですが[^demo]、`shellcheck` では[検出できる](https://www.shellcheck.net/wiki/SC2038)ので「自動でチェックさせる方法があればうれしい」といった感じです。

[^demo]: [デモ動画](https://githubnext.com/projects/copilot-cli/)で生成されている `find` の例も同じような問題を含んでいるので、「そういうものなのかな」とちょっとあきらめモードです。

## 対応されているコマンド

Copilot for CLI で対応されているコマンドの一覧などは見当たらなかったのですが、試してみた感じでは思っていたよりも各種コマンド(環境)に対応しているようでした。

たとえば、Hyper-V や BitLocker に関連するようなコマンドも(ある程度誘導する必要はありますが)生成されました。

**図 7-1 Hyper-V(PowerShell) 用のコマンドも生成**

```bash
$ ?? list virtual machines in hyper-v

 ───────────────────── Query ─────────────────────

1) list virtual machines in hyper-v 2) show id

 ──────────────────── Command ────────────────────

Get-VM | select id

 ────────────────── Explanation ──────────────────

○ Get-VM is a PowerShell command that lists all virtual machines.
  ◆ | select id selects the id field from the output of Get-VM.

  ✅ Run this command
  📝 Revise query
❯ ❌ Cancel
```

しかし、その反面で同じカテゴリーのコマンドが複数ある場合、どれを利用するのかは都度クエリーで指定するようです。

たとえば「**list virtual machines**」だけで指定すると VirtualBox 用に生成されますが、これを Hyper-V 用に切り替える方法は不明でした。

## おわりに

Copilot for CLI について試してみました。

クエリーの入力などは日本語や単語のみでも通りやすいので、思っていたよりも気軽に使えそうな感じです。ただし、安定度合いはもう少しといった感じなので、繰り返し利用したり外部からの入力を扱うコマンドの生成には向かない印象です。

そのような感じなので「単発利用のコマンドをラフな指示で実行したい」というときに使うと良いのかなと思います。
