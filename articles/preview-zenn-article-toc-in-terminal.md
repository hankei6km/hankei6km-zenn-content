---
id: preview-zenn-article-toc-in-terminal
title: Zenn 記事の目次をターミナル内でプレビュー
emoji: "\U0001F30A"
type: idea
topics:
  - zenn
  - markdown
  - cli
  - terminal
published: true
---
作成した Zenn 記事を実環境にデプロイしてみたところ「目次(章)のバランスがよくなかったかな」ということがありました。

本来ならばエディターのアウトラインなどで確認しておけばよいのですが、[現在は CMS 上で編集している](https://zenn.dev/hankei6km/articles/writing-article-with-headless-cms)のでそれも少し難しい。

![エディターでアウトラインを表示させているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7b21e52a28b149e3bda4eb7dfa6e1a69/preview-zenn-article-toc-in-terminal-eidtor-outline.png?w=259\&h=218\&auto=compress%2Cformat)*エディターのアウトライン表示*

そのようなわけで簡単な目次のプレビュー機能を作ってみました。

## 対応案

2 つほど思いついた案があったのですが、最終的には「[記事を CMS からローカルへ保存するスクリプト](https://zenn.dev/hankei6km/articles/writing-article-with-headless-cms#%E7%B7%A8%E9%9B%86%E3%81%A8%E3%83%97%E3%83%AC%E3%83%93%E3%83%A5%E3%83%BC)の表示の中に目次も含める」という方法になりました。

## 実装方法

FrontMatter + Markdown を扱うので本来ならば [`gray-matter`](https://github.com/jonschlinkert/gray-matter) + [`remark`](https://github.com/remarkjs/remark) などを使うのが固いのですが、今回は `grep` などを組み合わせることにしました。

### 見出しの抽出

`grep` で先頭の `#` を検索することで対応します。

以下のような懸念点はありますが、今回はとりあえず保留にしておきます。

*   FrontMatter 中のコメント
*   コードブロック内の `#` 形式のコメント
*   コードブロック内の Markdown
*   `---` を使った見出し

追記: [`remark-cli`](https://github.com/remarkjs/remark/tree/main/packages/remark-cli) を使う方法も[試してみたので](https://zenn.dev/link/comments/4f15dba248724b)問題が多いようなら切り替えることにします。

### 見出しの整形

`sed` と Bash の文字列操作で対応します。

```shell-session:grep + sed
$ grep -e '^##\{1,2\} ' -- tmp/articles/native-esm-with-typescript-jest.md | sed -e 's/^## /+ /' -e 's/^### /| /'
+ モジュールの native ESM 対応
+ ts-jest の ESM 対応設定
+ Jest の native ESM 対応設定
+ native ESM での jest モックモジュール
+ VSCode 関連
| 構文チェック
| デバッグ
| コード補完と動的インポート
+ とりあえず
```

### 表示幅の確認

これは以下のように「実環境での表示で折り返される」行を検出したいというものです。

環境によって表示は異なるので、それほど気にするものではありませんが「長すぎる見出しもよくないかな」ということで。

![目次の項目が折り返されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5e5bd347b1fe4de3b53e6e3fae90dad3/preview-zenn-article-toc-in-terminal-long-headding.png?w=355\&h=410\&auto=compress%2Cformat)*折り返されている項目*

この部分はターミナル上だと難しいので([Puppeteer](https://github.com/puppeteer/puppeteer) でいけるかもですがそこまでは)、Fullwidth などを考慮した値を `wc --max-line-length` で取得し簡易的に使います[^max-line-length] 。

```shell-session:wc
$ echo '👀 Preview:'| wc --max-line-length
11
```

[^max-line-length]: <https://unix.stackexchange.com/a/258551>

## 作ってみた

上記の内容を組み合わせて作ったのもが以下になります。

:::details script(toc.sh)

```bash:toc.sh
#!/bin/bash
set -e

WARN_LEN="33"

COLOR_RESET='\e[0m'
COLOR_DEPTH2='\e[0m'
COLOR_DEPTH3='\e[2m'
COLOR_WARN='\e[33m'

grep -e '^\(##\|###\) ' -- | sed -e 's/^## /+ /' -e 's/^### /| /' | while read -r LINE
do
  HEADDING="${LINE::1}"
  TITLE="${LINE:2}"
  LEN="$(echo "${TITLE}" | wc --max-line-length)"

  # ヘッディング文字
  echo -n "${HEADDING} " 

  # 装飾
  if test "${LEN}" -gt "${WARN_LEN}" ; then
    echo -n -e "${COLOR_WARN}"
  elif test "${HEADDING}" = "+" ;then
    echo -n -e "${COLOR_DEPTH2}"
  else
    echo -n -e "${COLOR_DEPTH3}"
  fi

  # タイトル
  echo -n "${TITLE}"

  # 装飾リセット
  echo -n -e "${COLOR_RESET}"
  echo ""
done 
```

:::

```shell-session
$ bash scripts/toc.sh < articles/native-esm-with-typescript-jest.md
```

![スクリプトを実行して長いタイトルが警告色で表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b9c674437e404af8a23deb9d9c7b9436/preview-zenn-article-toc-in-terminal-result.png?w=572\&h=220\&auto=compress%2Cformat%2Cenhance)*実行結果*

簡単な装飾と長い行は警告色で表示されるようになっています。

## おわりに

とりあえず動作するようになったので CMS からローカルへ保存するスクリプトにも組み込んで記事作成時に利用しています。

![CMS のコンテンツをローカルへ保存する時に目次をプレビューしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/88ce683103534bdaa0d9ae9d9ae10684/preview-zenn-article-toc-in-terminal-save-script.png?w=519\&h=518\&auto=compress%2Cformat%2Cenhance)*CMS からの保存時に目次プレビュー*

やはり目次を事前に確認できるのは良い感じです。

## ちなみに

本筋とは関係ないのですが、現在スクラップの使い方の練習もしています。

その一環として、この記事は以下のスクラップを作成してからそれを元に記述しています。

@[card](https://zenn.dev/hankei6km/scraps/95dbc1f23e695f)
