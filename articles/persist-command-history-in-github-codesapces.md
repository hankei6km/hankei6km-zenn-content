---
id: persist-command-history-in-github-codesapces
title: GitHub Codespace で Terminal の入力履歴を永続化し Codepace 間で共有してみる
emoji: 🖥️
type: idea
topics:
  - github
  - codespaces
  - googledrive
  - bash
published: true
---

GitHub Codespaces でコードを書くとき、いつものノリで Terminal から GitHub CLI を使ってみたところ「入力履歴が消えるとやりにくい」と感じました。

ダメ元で「特定のファイルを同期するような機能ないかな」と少し調べてみましたが、やはり同期されないようなので対策を考えてみました。

## Terminal の入力履歴を永続化するための基本方針

まずは基本となる VSCode の Terminal の入力履歴の永続化ですが、これはドキュメントに記載があります。

@[card](https://code.visualstudio.com/remote/advancedcontainers/persist-bash-history)

*   永続化するための Volume を用意しコンテナ内でどこかのディレクトリにマウントする
*   履歴を永続化したいコマンドの設定を変更し、履歴ファイルを上記のディレクトリへ確実に保存させる

シンプルで実現しやすい方法なのでローカルの Remote - Containers ではこの方法に少し手を加えて使っていました。

しかし、コマンドによっては「単純に保存先を変更するだけでは対応できない」場合があります。また、Codespaces のようなサービスの場合「どこかに永続化された領域を確保する必要がある」ためその辺を考える必要があります。

## bash の入力履歴を共有する

「bash の入力履歴は `.bash_history` に保存されるのだからそれを永続化すればよいよね」といきたいところですが、そうでもなかったりします。

たとえば、Tmux や GNOME Terminal で複数のタブなどを使っていると、それぞれのプロンプトで入力履歴が異なる状況になっていることがあります。「履歴はプロセス毎に保持されていて、ファイルへの書き込みは特定のタイミングで行われる」ことから発生する現象であり、今回のように永続化したい場合でも少し都合が悪い状況です。

ではどうするかというと「Tmux やタブなどでの多重化と bash の入力履歴」で調べてみると定番な回避策が出てきます。

@[card](https://askubuntu.com/questions/339546/how-do-i-see-the-history-of-the-commands-i-have-run-in-tmux)
@[card](https://unix.stackexchange.com/questions/1288/preserve-bash-history-in-multiple-terminal-windows)

記事によって少し違いはありますが、大筋はプロンプトが表示されるタイミングで履歴を同期(共有)させています[^append]。

▼ **図 2-1 プロンプト表示時に同期させる `.bash_rc` の設定**

```bash:.bash_rc
HISTCONTROL=ignoreboth
shopt -s histappend
HISTFILE="/${HOME}/.history/.bash_history"

# 履歴の同期.
function share_history {
  history -a
  history -c
  history -r
}
# プロンプトで実行するコマンド.
function prompt_func {
  share_history
}
PROMPT_COMMAND="prompt_func"
```

これを利用すると「プロンプトが表示される毎に履歴が `.bash_history` と同期される」のでタブ間で履歴が共有される状態になります。また、永続化するときも、`.bash_hitstory` を抜き出すだけで対応できるようになります。

[^append]: 前述の VSCode のドキュメントでも対応はされていますがファイルへの追加書き出しだけなので、Terminal のタブを複数作ると共有されない状況になります。

ちなみに、この方法は数年前の日経 Linux のシス管系女子で初めて知りました。そして、いまならなんとそのエピソードも収録された書籍が出たばかりです(冗談ぽく書いていますが参考になることが多いのでわりと真面目におすすめですよ)。

@[card](https://info.nikkeibp.co.jp/media/LIN/atcl/books/031700030/)

## Codespaces 用に永続化された領域を確保

Codespace を Rebuild したり別のブランチやリポジトリで Codespace を作成した場合、各コンテナ間で履歴を引き継ぐためには永続化された領域が必要になります。

これはいくつか方法がありそうですが「他のサービスに移行するときにも再利用しやすいかな」ということで外部のストレージを利用します(今回は GoogleDrive をマウント)。詳細については以下の記事を参照してください。

@[card](https://zenn.dev/hankei6km/articles/mount-google-drive-in-github-codespaces)

ただし、レスポンスがあまりよくないので「履歴ファイルの保存先ディレクトリに直接は指定できない」という点は別途解決する必要があります。

## 履歴用ディレクトリをバックグラウンドで同期させる

上記のレスポンスは「フォアグラウンドでの操作性」に対する問題なので、バックグラウンドで同期させることにしました。

正攻法で考えるなら「ファイル更新時に処理」「定期実行で処理」などがありますがデーモン的なものが必要です。そこで、今回は bash の履歴共有で使った「プロンプト表示のときにコマンド実行」を利用してみます。

具体的には以下のようにしています。

### コンテナ開始時にマウントしたドライブから履歴用ディレクトリをコピー

最初の 1 回はプロンプトの表示とは関係なく、無条件にドライブ側の履歴をコンテナ側へ上書きします。

これはについては、コンテナ開始時に Google Drive をマウントするスクリプトを実行しているので、以下のように SECRET からコマンドを挿入して実行しています[^bootstrap]。

▼ **図 4-1 コンテナ開始時にスクリプトを実行**

```json:.devcontainer/devcontainer.json
{
  // 一部抜粋
  "postStartCommand": [
    "/home/vscode/.local/bin/mount-gd.sh",
    "/home/vscode/gdrive"
  ]
}
```

▼ **図 4-2 実行されたスクリプトに SECRET 経由でコマンドを挿入**

```bash:~/.local/bin/mount-gd.sh
# 一部抜粋
if test -n "${BOOTSTRAP_CODE}" ; then
    source <(echo "${BOOTSTRAP_CODE}")
fi
```

▼ **図 4-3 挿入されるコマンド**

```bash:$BOOTSTRAP_CODE
# 一部抜粋
cp -r "${HOME}/gdrive/codespaces/history" "${HOME}/.history"
```

[^bootstrap]: 実際にはこれ以外に細々とした初期化用コマンドも挿入しています。たとえば clasp で開発しているリポジトリ向けに設定を展開したり等々。

### プロンプトが表示されたらバックグラウンドでコマンドを実行する

以下のようなスクリプトを作成し、`PROMPT_COMMAND` からバックグラウンドで開始させています。

*   先行してコマンドが実行されているか調べる
*   5 秒待機する
*   コンテナ側の履歴用ディレクトリのファイルをドライブ側へ上書きでコピー

▼ **図 4-4 バックグラウンドで実行されるスクリプト**

```bash:~/.local/bin/push-hist.sh
#!/bin/bash

set -e

HIST_TEMP="/tmp/hist-temp"
PID_FILE="${HIST_TEMP}/pid"
WAIT_SEC="5"

if test -f "${PID_FILE}" ; then
  exit 0
fi
trap 'test -f "${PID_FILE}" && rm "${PID_FILE}"' EXIT

echo "$$" > "${PID_FILE}" 

sleep "${WAIT_SEC}"

find "${HOME}/.history" -type f ! -name ".bash_history" -exec cp {} "${HOME}/gdrive/codespaces/history/" \;
```

▼ **図 4-5 プロンプト表示時にバックグラウンドで実行する設定**

```bash:.bash_rc
# プロンプトで実行するコマンド.
function prompt_func {
  share_history
  # https://stackoverflow.com/questions/3683910/executing-shell-command-in-background-from-script
  push-hist.sh &>/dev/null & disown
}
PROMPT_COMMAND="prompt_func"
```

ここまでで大体は思ったとおりの処理になりますが、複数 Codespace を同時に使っている場合「最後に同期した Codespace の履歴で上書きされる」ことになります。

この辺はある程度の欠落を許容することにしたのですが、それでも「bash の履歴はできるだけ守りたい」ということで個別に対応しています。

## 複数の Copdespace 間で bash の入力履歴を共有する

最初は「複数の `.bash_history` を連結させて重複を除去」的なことをやっていたのですが、状況によって履歴の順番がかなりヨレてしまいます。

そこで以下のように bash の履歴をマージする処理を追加しました。

```bash:~/.local/bin/push-hist.sh
# 以下の処理を追加
HISTFILESIZE=2000
HISTFILE="${HOME}/.history/.bash_history"

touch "${HOME}/gdrive/codespaces/history/.bash_history"
awk '{a[$0]++; print a[$0], $0}' "${HOME}/gdrive/codespaces/history/.bash_history" "${HISTFILE}" \
  | tac | awk '$1!=2{$1=""; b=substr($0,2); if(!c[b]++){print b}}' | tac \
  | tail -n "${HISTFILESIZE}" > "${HIST_TEMP}/.bash_history"

cp "${HIST_TEMP}/.bash_history" "${HISTFILE}"
history -c
history -r

cp "${HIST_TEMP}/.bash_history" "${HOME}/gdrive/codespaces/history/.bash_history"
```

少し解説すると以下のような感じになっています。

まず、ドライブ側の `.bash_history` とコンテナ側の `.bash_history` を連結し各エントリー(行)の出現回数を付与する。

`tac` で順番を反転させた後に、以下のようにフィルタリングする。

*   出現回数

    *   1 回 - その位置へ残す(ドライブ側に元々存在していたか、新しく入力されたエントリー)
    *   2 回 - 削除対象(ドライブ側とコンテナ側で重複しているエントリー)
    *   3 回以上 - その位置へ残す(重複しているが新しく入力されたエントリーでもある)

*   出現順序を維持しつつ重複を削除

フィルタリングしたら `tac` で順番を戻す。最後にサイズを調整しそれぞれの場所へ戻す。

なお、重複の削除は下記の記事を参考にしています。

@[card](https://zenn.dev/creationup2u/articles/5981b5ea331455)

これでも以下のような点に改善の余地はありますが、極端に不自然な感じもでないのでとりあえずこのまま使っています。

*   プロンプト表示時とは異なるタイミングで同期される
*   複数 Codespace を利用しているときは、それぞれで履歴の順番が異なっている

## その他の方法(ファイル転送)

当初はマウントではなくファイル転送の利用を考えていました。現行の方法でもマウントは必須でないので、好みによってはファイル転送もありかと思われます[^vscode-sync]。

[^vscode-sync]: 他には、VSCode の同期を有効にしている場合 `settings.json` も同期されるので「コメントなどに偽装して履歴を保持できないかな」なども考えていました。

## おわりに

Terminal の入力履歴を永続化し Codespace 間で共有してみました。

現時点で bash の入力履歴については概ね期待通りに動作しています。その他のコマンド(vim、tmux、ranger など)についてはどこまで対応するか検討中ですが、とりあえずはこれで使っていこうかと考えています。
