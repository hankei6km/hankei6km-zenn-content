---
id: readmemd-has-been-run-in-my-terminal
title: README.md が実行できるようになっていた
emoji: 📒
type: idea
topics:
  - markdown
  - cli
  - readme
published: true
---

先日、下記のタイトルを見て「どういうことだってばよ？」となったので。

@[card](https://dev.to/sourishkrout/run-readmemd-in-your-terminal-19i)

あと、Markdown のコードブロックの言語指定を置き換えるコマンド作ろうかなと考えていたので(何が関係あるんだ？というのは後で出てきます)。

## つまり、どういうこと？

ちょっと触ってみた感じだと、`README.md` の中からコードブロックを抜き出し「説明付きで一覧にしたり」「実行したり」する CLI ツールを作ってみたということのようです。

そう聞くと「それはコードブロックをコピーして実行すればいいんじゃね」となりそうですが、 ドキュメント内でのコマンド実行のサンプルは以下のように行頭に `$` が付いていたり。実行結果が併記されていたりすることがあります[^prompt]。

[^prompt]: と、思ったのですが、有名所のツールの `README.md` 改めて見てみるとそうでもないかなという気もしてきました。

`rdme` はその辺をなんかいい感じに実行してくれます。

**リスト 1-1 サンプルの README.md**

    # テスト

    ## Installation

    ```sh
    npm install remark-cli
    ```

    ## プロンプト付き

    `$` が付いている場合は、いい感じに処理してくれる。

    ```sh
    $ for i in $(seq 1 8); do echo "${i}"; done
    1
    2
    <snip...>
    7
    8
    $ echo "実行してみた"
    実行してみた
    ```

**図 1-1 run するとコマンドとして成立している部分が実行される**

```shell-session
$ rdme list
NAME         FIRST COMMAND                           # OF COMMANDS  DESCRIPTION
npm-install  npm install remark-cli                  1              Installation
for-i        for i in $(seq 1 8); do echo "${i}"...  2              $ が付いている場合は、いい感じに処...

$ rdme run npm-install

added 149 packages, and audited 150 packages in 12s

64 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities

$ rdme run for-i
1
2
3
4
5
6
7
8
実行してみた
```

## 入力補完

入力補完にも対応していて `eval “$(rdme completaion bash) `などで利用できるようになります。

これが地味に便利でして、前述のような README の場合で「パッケージをインストールしたいな」と思ったら `rdme run n` で TAB を押下するだけとなります。

## エンジンの指定はちょっとめんどう

「どのエンジン(コマンド)を使ってコードを実行するか？」はコードブロックの言語指定で決まるようで、ここに `console` などを指定していると以下のようになってしまいます[^console]。

[^console]: これもちょっと調べてみると、最近だと `console` はあまり使われないのかなという感じです(私はよく使っているのですが…)。

**図 3-1 言語指定が console だとエラーになる**

    $ head -n 11 README.md
    # xquo

    Command line utility to quote null splited lines for Bash command line.

    ## Installation

    ### Using `cargo`

    ```console
    $ cargo install xquo
    ```

    $ rdme run cargo-install
    unknown executable: "console"

これを外部から変更する方法はいまのところないようなので、別途やりくりする必要があります。

簡単なのは `sed` を使うことですが、pipe や process substitution は使えなかったので上書きするか別ファイルに書き出す必要がありました[^path]。

[^path]: 現状では「カレントディレクトリ以下にあるファイル」のみ指定可能なようです。

**図 3-2 sed で言語指定を置き換え**

````shell-session
$ sed -e 's/^```console/```sh/' < README.md > README_sh.md 
$ rdme run --filename README_sh.md cargo-install
````

ちなみに、冒頭でちょっと書いた言語指定を置き換えるコマンド用に [`remark-cli`](https://github.com/remarkjs/remark/tree/main) を試していたので、それも載せておきます。

**図 3-3 remark-cli で言語指定を置き換え**

```shell-session
$ npx remark --quiet --tree-out < README.md | jq '.children |= map(select(.type == "code" and .lang == "console").lang="sh" )' | npx remark --quiet --tree-in > README_sh.md
$ rdme run --filename README_sh.md cargo-install
```

## 危険じゃね？

自分で `rdme` を実行しなければ何も動かないので大丈夫なような気もしますが…。何か落とし穴があるかもしれないので、利用するならその辺は十分に注意するべきかなとは思います。

## おわりに

最初は「話のネタにちょっとだけ」と思って触ってみたのですが、一通り使ってみて「`cargo install` などをコピペ無しで実行できて実用的なのでは」という感想になったので記事にしてみました。

この先の展開はわかりませんが、GitHub 上の README.md を扱えるとさらに便利になるのかなという気はしています([この辺](https://github.com/stateful/rdme/issues/19)がそういう話なのかなと)。

## そして

この記事を書いているときに↓も見つけてしまいました。ちょっと興味ありますが、とりあえず今回はここまでということで。

@[card](https://dev.to/prasant/how-i-run-code-on-twitter-3h79)
