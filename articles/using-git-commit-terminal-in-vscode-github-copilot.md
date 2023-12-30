---
id: using-git-commit-terminal-in-vscode-github-copilot
title: VS Code + GitHub Copilot 環境でも git commit しちゃうんだな、これが
emoji: ✨
type: idea
topics:
  - vscode
  - git
  - github
  - copilot
published: true
---

VS Code でコミットするときに GitHub Copilot を使っていると [コミットメッセージを生成](https://code.visualstudio.com/docs/editor/github-copilot#_sparkles)してくれたりします。

**図 1 コミットメッセージを Copilot で生成**

![VS Code のソース管理タブでコミットメッセージを自動生成しているスクリーショット、画面右側には変更箇所も表示されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e744b41094554977b5e954bcf1bd123f/using-git-commit-terminal-in-vscode-github-copilot-intro.png?w=826\&h=389\&auto=compress%2Cformat)

英語苦手な自分からすると「マジうれしいんですけど」という感じなのですが、コミットメッセージはできればエディターで記述したいと考えてしまいます。

そこで今回は「コミットメッセージをエディターで編集する利便性」を維持しつつ、「GitHub Copilot による生成機能もできるだけ利用しよう」という内容のメモになります。

## VS Code のエディターでコミットメッセージを記述するとは

VS Code でコミットメッセージを記述する方法としてはソース管理タブの利用が一般的かと思われます。

**図 1-1 ソース管理タブのフィールドから普通に入力**

![ソース管理タブで複数行のコミットメッセージを入力しているスクリーショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/94ec56c540e84c5aab645155a241ebc5/using-git-commit-terminal-in-vscode-github-copilot-intro-source-controle.png?w=404\&h=205\&auto=compress%2Cformat)

一方で上記とは別に、コミット用のコマンドを実行しエディターの中で記述する方法もあります。

*   ターミナルから `git commit` を実行する
*   コマンドパレットで `Git: Commit` を実行する

こちらはコマンドを別途実行する必要がありますが、コミットメッセージ(`.git/COMMIT_EDITMSG`)をエディタータブで編集するので、エディターならではの編集支援を受けられます。

**図 1-2 エディターで入力(シンタックスハイライトや文字数のガイドルーラーが表示される)**

![エディターで前述にソース管理タブと同じコミットメッセージを入力しているスクリーショット、こちらはガイドルーラーが表示されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/dca1dd949b854c4b9a1d0fd78fd9c8f6/using-git-commit-terminal-in-vscode-github-copilot-editmsg.png?w=621\&h=256\&auto=compress%2Cformat)

また、コミットメッセージにも[言語 ID](https://code.visualstudio.com/docs/languages/overview) が割り当てられているので、キー配列や拡張機能によるカスタマイズも適用しやすくなっています。

私は「Markdown として表示するとどう見える？」が気にときがあるので、以下のようなショートカットを登録しています。

**リスト 1-1 ****`ctrl`**** + ****`shift`**** + ****`v`**** で Markdown プレビューを開くキーボードショートカット**

```json
{
  "key": "ctrl+shift+v",
  "command": "markdown.showPreview",
  "when": "!notebookEditorFocused && editorLangId == 'git-commit'"
}
```

::: message

`git commt` しても VS Code ではなく他のエディターが開くこともあります。そのような場合は以下を参考に、エディターとして `code --wait` を指定すると VS Code で開くようになります。

*   [Git-のカスタマイズ-Git-の設定](https://git-scm.com/book/ja/v2/Git-のカスタマイズ-Git-の設定)
*   [Gitの内側-環境変数](https://git-scm.com/book/ja/v2/Gitの内側-環境変数)

なお、とりあえず試してみるだけならば `GIT_EDITOR` 環境変数を変更するのが簡単かと思います(後述の [GitHub CLI] にも反映されるようです)。

```shell-session
$ export GIT_EDITOR="code --wait"
```

:::

## デフォルトで利用できる Copilot の機能

普通にエディターなので Copilot が有効であればデフォルト状態でもコミットメッセージのサジェストなどをやってくれます。おそらくは編集中の `COMMIT_EDITMSG` の内容がコンテキストになっていると思われますが、ブランチ名や変更のあったファイル名一覧が含まれるているので、わりと良い感じにサジェストしてくれます。

**図 2-1 Copilot によるサジェスト**

![「ci: Add "test,yml"」と入力したときに、「for GitHub Actions」とサジェストされているスクリーショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/814629f833094c5ab4f4e581f7f11209/using-git-commit-terminal-in-vscode-github-copilot-suggest.png?w=584\&h=226\&auto=compress%2Cformat)

また、Copilot のインラインチャットも利用できます。

**図 2-2 インラインチャットの利用**

![コミットメッセージの一部を範囲選択し、インラインチャットへ「to english」と指示しているスクリーショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9fd3e534bb384002a06d3e1679ff395c/using-git-commit-terminal-in-vscode-github-copilot-inline.png?w=748\&h=279\&auto=compress%2Cformat)

この辺の編集支援だけでもやりやすくなるので「もうデフォルトのままでもいいんじゃないかな」という気にもなってきます。

::: message

インラインチャットは [GitHub Copilot Chat] 拡張機能をインストール後、以下の方法で開始できます。

*   エディターで `ctrl` + `i`(vscode-vim とコンフリクトするかもしれません)
*   コマンドパレットから `Inline Chat: Start Inline Chat`
*   エディターで右クリックし「Copilot / Start Inline Chat」

:::

## コミットメッセージ生成機能に近づける

「もうデフォルトのままでもいいんじゃないかな」と言いましたが、やはり隣の芝生は青く見えてきます。

対応として `git commit` でエディターを開くときに最初から生成されたテキストが挿入される方法を探したのですが、見つかりませんでした。

その代わりでもないのですが、サジェストなどの精度を上げる方法として、[Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) を利用しています。以下のようなスクリプトを `.git/hooks/prepare-commit-msg` へ設定しておくと `git commit` でエディターを開いたとき、diff がコメントとして挿入されます。

**リスト 3-1 ****`COMMIT_EDITMSG`**** へ diff を挿入するスクリプト**

```sh
#!/bin/sh

set -e

COMMIT_MSG_FILE="${1}"
COMMIT_SOURCE="${2}"
SHA1="${3}"

print_context() {
    echo "" >> "${COMMIT_MSG_FILE}"
    git diff --cached | sed 's/^/# /' >> "${COMMIT_MSG_FILE}"
}

case "${COMMIT_SOURCE},${SHA1}" in
    ",")
        print_context;;
    "template,")
        print_context;;
    *) ;;
esac
```

前述のように `COMMIT_EDITMSG` の内容をコンテキストにしているようなので、diff の挿入によりサジェストやインラインチャットの応答が具体的になったと感じています。 (「なった」と言い切りたいところですが、やはりバラ付きはあります)

また、この状態になればインラインチャットで “make commit message” のように指示すると、生成機能ぽい感じになります。 (以下の例は上手くいっている方なので、あまり過度な期待はしないでください)

**図 3-1 生成機能とインラインチャットの比較(diff なし)**

![自動生成とインラインチャットへの指示を同時に行っているスクリーショット、インラインチャットの応答は簡素なものになっている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3c5e9e4392b44e85b4e07188609fef20/using-git-commit-terminal-in-vscode-github-copilot-comp.png?w=1134\&h=465\&auto=compress%2Cformat)

*   生成機能: **Add readline functionality to index.mjs**
*   インラインチャット: **Add new feature**

**図 3-2 生成機能とインラインチャットの比較(diff あり)**

![自動生成とインラインチャットへの指示を同時に行っているスクリーショット、どちらも同じ応答で内容も具体的になっている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e41c97af7f194c68846f579775359735/using-git-commit-terminal-in-vscode-github-copilot-comp-diff.png?w=1131\&h=663\&auto=compress%2Cformat)

*   生成機能: **Add readline functionality to index.mjs**
*   インラインチャット: **Add readline functionality to index.mjs**

あとは、まだ試行錯誤中ですが[^history]、コミットテンプレートなどから指示を埋め込んでおくとより精度が良くなるかなと考えています。

[^history]: [Hook 用スクリプトを Dev Container 内に配置する Feature](https://github.com/hankei6km/h6-devcontainers-features/tree/main/src/prepare-commit-msg-context) を作ったとき履歴も含たのですが、単純な前方一致で「似ているコミットメッセージをサジェストする」ことが増えました。「使えそうなものはとにかく追加」な方針ではダメそうです。

そして、ここまで書いておいてアレなのですが、それでも生成機能に頼りたくなることもあります。その場合は普通に生成してから「コピペしちゃうんだな、これを」という対応にしています。

## ターミナル内で `git commit` した後

GitHub を利用している場合、`git commit` した後はプルリクエストを作成することになるかと思います。話はそれてしまいますが、プルリクエストの説明などもエディターで記述する利点を少しだけ。

プルリクエスト作成は [GitHub Pull Requests and Issues] を使う方法もありますが、ターミナル内で操作を継続するなら [GitHub CLI]\(`gh` コマンド)もおすすめです。

たとえば、コマンドラインから `gh pr create` を実行すると、ボディ(プルリクエストの説明)を Markdown として記述するので、こちらもエディターから Copilot を利用できます。

**図 4-1 ****`gh pr create`**** で編集時にインラインチャットを利用**

![エディターで PR のボディを編集しているときに、インラインチャットで mermaid のダイアログを挿入しているスクリーショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/917448a244f2421cbcd31bdaf903a56b/using-git-commit-terminal-in-vscode-github-copilot-pr-create.png?w=774\&h=334\&auto=compress%2Cformat)

**図 4-2 作成されたプルリクエスト**

![実際に作成されたプルリクエストのスクリーショット、インラインチャットで挿入した図が表示されている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/deb2b36e92ab48cd9caf0872abfbd959/using-git-commit-terminal-in-vscode-github-copilot-pr-created.png?w=923\&h=537\&auto=compress%2Cformat)

ただし、こちらもエディターでの記述がベストと一概には言えないのが悩み所です。

*   [GitHub Pull Requests and Issues] にも Copilot による生成機能がある
*   コメント記述時に @ などの入力補完を利用できない

私は `gh pr checks --watch` なども使いたいので Git 関連の操作はターミナルに寄せていますが、それでも場合によってツールを使い分けているといった感じです[^copilot-cli]。

[^copilot-cli]: 最近、Copilot CLI が [GitHub CLI] の [extension になった](https://docs.github.com/ja/copilot/github-copilot-in-the-cli)ので `gh copilot` コマンドが使えるようになりました。ここに「`gh copilot pr` みたいな機能が追加されないかな」と妄想しています。

::: message

[GitHub CLI] も設定によっては他のエディターが開くこともあります。変更方法は下記を参照してください。

*   [GitHub CLI クイックスタート - GitHub Docs](https://docs.github.com/ja/github-cli/github-cli/quickstart#customizing-github-cli)

:::

## おわりに

コミットメッセージを VS Code のエディターで記述するとき、GitHub Copilot の生成機能をできるだけ利用してみました。

Copilot はなんとなくデフォルトのままで使っていましたが、コンテキストとなる情報を少し追加するだけでも生成物の精度が向上することを体験できました。

今後も生成 AI におまかせするだけでなく、工夫しながら利用できればと思っています。

[GitHub Copilot Chat]: https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat

[GitHub Pull Requests and Issues]: https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github

[GitHub CLI]: https://docs.github.com/ja/github-cli
