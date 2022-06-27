---
id: lf-in-filenames-is-confusing
title: 本当は怖いファイル名の改行文字
emoji: 🙅
type: tech
topics:
  - security
  - cli
published: true
---

以前にファイル名の改行文字で悩んだのを思い出していたら「ディレクトリトラバーサル的なことに利用できてしまうのでは」となったので。

## どういうこと？

ファイル名(PATH 名、ディレクトリ名)に改行文字を使えることは一般的でないかもしれないので少し説明を。

Linux[^windows-mac] ではファイルシステム的に使えない文字はあまり無かったりします。普段ファイルを操作していて使えない(ように見える)文字がある場合は「シェルなどの UI の振舞い」に影響されていることがほとんどです。

[^windows-mac]: Windows、Mac についてはきちんとは調べていないので Linux に限定しています。

よって、UI の制約を回避して `touch` コマンドなどの引数に改行文字を含めることができれば、ファイル名に改行文字を含めることができます。

そして、改行文字に限らず「普段なにげなく考えていることの裏を突くような文字」もわりと簡単に含めることができます。有名なところではファイル名の途中で「文字の並びを逆にする」制御文字などがあります。

@[card](https://forest.watch.impress.co.jp/docs/serial/yajiuma/1388917.html)

今回は制御文字だけでなく、身近な改行文字などでもファイル名の扱い方によっては怖いことになってしまう様子を見てみたいと思います。

## 改行文字を含めてみる

以下は Arch Linux 上の bash、ファイルシステムは ext4 で試しています。

:::message
WSL を使っていないので WSL でファイル名に改行文字などが使えるかは試していません。最悪は削除できないファイルができる可能性もあるので注意してください。
:::

引数に改行を渡す方法はいくつかありますが、今回は `"` のクオートを利用します。

`touch` を実行するときに、 `touch "one` と入力した後 `enter` し `>` が表示されたら続けて `two.txt"` と入力します。

▼ **図 2-1 クオートを利用して改行文字を渡す**

```shell-session
$ touch "one
> two.txt"
```

作成されたファイルを `ls` で見ると以下のようになります。1 行になっていますが、これは端末へ出力されるときの表現です。

▼ **図 2-2 普通に `ls` で表示すると 1 行で表現される**

```shell-session
$ ls
'one'$'\n''two.txt'
```

pipe を通すと以下のように改行文字が含まれていることがわかりやすくなります[^alpine]。

▼ **図 2-3 pipe を通すと改行して表示される**

```shell-session
$ ls | cat
one
two.txt
```

[^alpine]: Alpine だと端末向けと pipe どちらの出力も 1 行の表示になりますが、実際には改行文字が含まれたファイル名になっています。

## 改行文字を含むファイル名を使ってみる

ファイル名に改行文字を含めることができたので、今度は CLI 系のツールで少し使ってみます。なお、いわゆる TUI / GUI 系のツールでは表示が崩れるなどあるのですが、多岐にわたりすぎるので今回は触れないことにします。

### find

以下のようにファイルを配置し `find` から利用してみます。

▼ **図 3-1 ファイル一覧(各ファイルには 1 行だけテキストが入力されています)**

```shell-session
$ ls
 abc.txt   efg.txt  'one'$'\n''two.txt'
```

普通に find すると `ls` と同じような結果になります。

▼ **図 3-2 `find` の実行結果**

```shell-session
$ find . -type f
./abc.txt
./one?two.txt
./efg.txt

$ find . -type f | cat
./abc.txt
./one
two.txt
./efg.txt
```

続いて `-exec` から利用してみます。`echo` で末尾に `=` を付けてみたところ、改行文字が含まれていても 1 つの引数として扱われています。

▼ **図 3-3 `-exec` で `echo` を実行**

```shell-session
$ find . -type f -exec echo {}= \;
./abc.txt=
./one
two.txt=
./efg.txt=
```

`echo` 以外のコマンドでも 1 つの引数として利用できます。

▼ **図 3-4 `wc` でもファイル名は 1 つの引数となる**

```shell-session
$ find . -type f -exec wc -l {} \;
1 ./abc.txt
1 './one'$'\n''two.txt'
1 ./efg.txt
```

### bash の for

改行文字が含まれていてもファイル名毎に変数へセットされています。

▼ **図 3-5 変数内のファイル名を表示(末尾に `=` を追加)**

```shell-session
$ for i in * ; do echo "${i}=" ; done
abc.txt=
efg.txt=
one
two.txt=
```

▼ **図 3-6 他コマンドでもファイル名は 1 つの引数になる**

```shell-session
$ for i in * ; do wc -l "${i}" ; done
1 abc.txt
1 efg.txt
1 'one'$'\n''two.txt'
```

上記のように改行文字を含んでいていも普通に変数へセットされるので、ファイル名の整形などもできます。

▼ **図 3-7 文字列の置き換えで拡張子を変更**

```shell-session
$ for i in * ; do echo "${i%.*}.md" ; done
abc.md
efg.md
one
two.md
```

### Node.js

Node.js の API でファイル名の一覧を扱う場合も通常通りです。

▼ **図 3-8 ファイル名一覧の取得(末尾に `=` を追加)**

```shell-session
$ node
Welcome to Node.js v16.14.0.
Type ".help" for more information.
> fs.readdirSync('./').map(v=>`${v}=`)
[ 'abc.txt=', 'efg.txt=', 'one\ntwo.txt=' ]
```

▼ **図 3-9 ファイルの読み取り**

```shell-session
> fs.readdirSync('./').map(v=>fs.readFileSync(v).toString())
[ 'aaa\n', 'bbb\n', 'one\n' ]
```

他の言語でも「ファイル一覧が配列的なもので扱える」なら同じようになると思われます。

### Git

具体的な操作は省略しますが、`git status` の表示などを見る限りでは「ファイル名に改行文字が含まれることは想定済み」という感じです。 ただし、周辺ツールについては検証していないので組み合わせによっては不安定になる可能性もあります。

▼ **図 3-10 ファイル名の改行文字は `\n` で表示される**

```shell-session
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   "one\ntwo.md"

no changes added to commit (use "git add" and/or "git commit -a")
```

なお、Git 本体が改行文字に対応しているのは良いことなのですが、別の見かたをすると「GitHub と連携しているサービスに改行文字を含むファイル名を送り込める」とも言えます。これについては後で少し触れます。

実験に使ったリポジトリは GitHub に上げてあります(後述のディレクトリトラバーサル的に振舞う仕掛けは入れてありません)。

@[card](https://github.com/hankei6km/test-lf)

## エラーになるように使ってみる

ここまでは「ファイル名を個別にコマンドラインの引数として(IFS などに影響されないよう)受け渡している」ためいつものように利用できています。これを「ファイル名の一覧を複数行のテキストとして受け渡す」と雲行きが怪しくなってきます。

たとえば、`find` を使うとき「パフォーマンス的に有利」ということで `xargs` を利用している方も多いかと思います。

@[card](https://qiita.com/legitwhiz/items/e609537fb6226081f5b5)
@[card](https://precure-3dprinter.hatenablog.jp/entry/2018/05/06/xargs%E3%81%AF%E5%87%BA%E5%8A%9B%E3%82%92%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E5%BC%95%E6%95%B0%E3%81%AB%E6%B8%A1%E3%81%99%E3%81%A0%E3%81%91%E3%81%AE%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89)

この組み合わせを試すと以下のように、改行のところでファイル名が分割されてしまいます。

▼ **図 4-1 ファイル名が分割されてエラーとなる**

```shell-session
$ find . -type f | xargs wc -l
1 ./abc.txt
wc: ./one: そのようなファイルやディレクトリはありません
wc: two.txt: そのようなファイルやディレクトリはありません
1 ./efg.txt
2 合計
```

このエラーは「xargs で空白を含むファイル名をまとめてわたすと発生するエラー」に似ています。ならばと、試しにファイル名をまとめて渡さないようにしても同様です。

▼ **図 4-2 同じくファイル名が分割されてエラーとなる**

```shell-session
$ find . -type f | xargs -I {} wc {} -l
1 ./abc.txt
wc: ./one: そのようなファイルやディレクトリはありません
wc: two.txt: そのようなファイルやディレクトリはありません
1 ./efg.txt
```

上記の現象は、pipe を通すことで「ファイル名の一覧」が「(特別な意味を持たない)複数行のテキスト」になることが影響しています。そのため、「ファイル名は 1 行のテキスト」という前提で操作しているとエラーになります。

では方法がないのかというと、改行文字の代わりに null 文字を区切りにすることで対応できます。

▼ **図 4-3 区切りが null 文字になるようにオプション指定**

```shell-session
$ find . -type f -print0 | xargs -0 wc -l
 1 ./abc.txt
 1 './one'$'\n''two.txt'
 1 ./efg.txt
 3 合計
```

ちなみに、null 文字を区切りにする方法ですが、テキスト行を扱う有名所のコマンドではサポートされていることが多いです。

以下 `sort` の例です。

▼ **図 4-4 改行区切りのまま(わかりやすくするために `qqq.txt` を追加してあります)**

```shell-session
$ find . -type f | sort
./abc.txt
./efg.txt
./one
./qqq.txt
two.txt
```

▼ **図 4-5 null を区切りにする(sort は出力も null 区切りになるので `tr` で改行区切りに戻しています)**

```shell-session
$ find . -type f -print0 | sort -z  | tr '\0' '\n'
./abc.txt
./efg.txt
./one
two.txt
./qqq.txt
```

:::message
ファイル名に null 文字が含まれることはあるのか？

簡単に検索してみただけですが「現状は含まれない」という認識で良さそうです[^null-in-filename]。

@[card](https://stackoverflow.com/questions/1976007/what-characters-are-forbidden-in-windows-and-linux-directory-names)
:::

[^null-in-filename]: 「null 文字を含めようとするとエラーになる」ところを確認したかったのですが、null 文字を含める操作方法を思いつけませんでした。

## 怖くなるように使ってみる

上記のエラーになるのも怖いのですが、もう少し怖くなる「ディレクトリトラバーサル的」に想定外のファイルへアクセスさせる方法を試してみます。

なお、これには「ファイル名一覧の扱いをミスしている」ことが前提になります。よって、厳密にはディレクトリトラバーサルとは違いそうなので「的」としています(あとは必ずしも `..` を必要としないので見た目的にも「ぽく」ないかなと)。

### 概要

これまでは「ファイル名の途中を改行文字」にしていたので、分割されたファイル名を個別に利用しても「エラーになるだけ」でした。たとえば `abc\n123.txt` が `abc` と `123.txt` になってもファイルが存在しないだけです(存在する可能性もなくはないですが)。

それでは「ディレクトリ名の末尾を改行文字」にしたらどうなるでしょうか？

`abc\n/etc/passwd` ば改行で分割されると `abc` と `/etc/passwd` になります。少し怖くなってきました。

### 手順

まず、踏み台となるディレクトリを作成し、その中にアクセスさせたい PATH を再現します。

▼ **図 5-1 踏み台ディレクトリを作成**

```shell-session
$ mkdir "jump
> "
$ mkdir "jump
> /etc"
$ touch "jump
> /etc/passwd"

$ find . -type f
./jump?/etc/passwd
```

この状態で「正しく動作しない find + xargs」で cat を実行すると対象のファイルが表示されます。

▼ **図 5-2 絶対 PATH で指定されたファイルが cat される**

```shell-session
$ find . -type f | xargs cat
cat: ./jump: そのようなファイルやディレクトリはありません
# この下に `/etc/passwd` の内容が表示される
```

ディレクトリトラバーサルぽくはないですが、より直接的に閲覧したい対象を指定できることがわかりました。

なお、ディレクトリ名の末尾を `改行文字` + `..` にすると 1 つ上の親ディレクトリへのアクセスになります。

### プログラムコードからは？

CLI 的な操作では想定外のファイルにアクセスしてしまいましたが、プログラム的にはどうなるでしょうか？

今回の問題は「ファイル名の一覧を複数行のテキストとして扱う」ことが根本的な原因です。Node.js で試したときのように各言語のライブラリーを適切に利用していれば、ファイル名の一覧は配列などで扱われるので影響は少ないと思われます[^tui-gui]。

ただし「ディレクトリ内を再帰的に操作するとき」にコードで walk 的なものを用意するのでなく、外部コマンドとして `find` などを利用しているようなときは注意が必要そうです。

[^tui-gui]: 今回は掘り下げませんが、TUI / GUI ツールだとファイル名が複数行で分割表示され、分割された文字列をファイル名として扱ってしまう可能性はあります。

## 対策

改行文字対策でいうなら「ファイル名一覧を複数行のテキストで扱わない」ことにつきますが、一般的な UI で行われているように「そもそも改行文字を許可しない」という方法もあります。

### ホワイトリスト

Git リポジトリは「こちらの意図したように」改行文字をファイル名に含めることがでるので、危ないファイル名をサービスへ送り込めます。

たとえば、Zenn は GitHub と連携することでも記事を作成できますが、この場合は「ファイル名が記事の slug になる」ので改行文字の影響をうける可能性もなくはないです。そこで、試しに「ファイル名を `test-test-test-test\naaa.md` として push」してみました。

@[card](https://github.com/hankei6km/hankei6km-zenn-content/commit/bf051f62f529611cd57e73a74736a52ecb0ea509)

結果は以下のとおりです。

▼ **図 6-1 push した結果**

![slug が不正であったためデプロイが中断されたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d3c2ad756426472283b5161b8cdee1e8/lf-in-filenames-is-confusing-reject.png?w=825\&h=120\&auto=compress%2Cformat)

改行文字は「許可された文字ではない」ので slug 名のチェックできちんと弾かれました。

## おわりに

ファイル名に改行文字などを含める方法、含めるとどのように怖くなるのかを見てみました。

*   改行文字などを含めるのはわりと簡単

*   ファイル名の一覧を複数行のテキストとして扱うと怖くなる

    *   改行文字区切りではなく null 文字区切りにすることで緩和できる

*   改行文字が必須でないならホワイトリストなどで弾くのも 1 つの手である

自分が作るファイルの名前に改行文字を含めることはほとんどないとは思いますが、どこから紛れ込んでくるかはわからないので気を付けたいところであります。

## おまけ

「エディターやスプレッドシートに `find` などの結果を貼り付けてコマンド行を生成」「`xargs` で `bash -c` を使う」などもファイル名をテキストとして扱うことになります。 (さらに悪いことにシェルから解釈されるべた書きのテキスト扱い)

よって、(IFS にもよりますが)空白文字でもファイル名が分割されたり、あっさり OS コマンドインジェクションなどが成立するので注意が必要です。

▼ **図 8-1インジェクションしているファイル一覧**

    ./e_hoba_profile.md
    ./av-98_specs.md
    ./type-zero_specs.md
    ./babylon_project&sh hos.md
    ./shinohara_heavy_industrial.md
    ./hos.md

▼ **図 8-2 hos.md の内容**

```md
# HOS User's Guide

## Introduction

: << ---
Go to, let us go down, and there confound
their language, that they may not understand
one another's speech.

---

## Examples

echo "\e[31m" ; for i in $(seq 500); do echo -n "BABEL "; sleep 0.05 ; done
```

これに対して `find . -print0 | xargs -0 -I {} bash -c 'test -f {} && wc -l {}'` を実行すると以下のようになります。

▼ **図 8-3 BABEL BABEL BABEL**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7f530c61fb524fc4b8dfb31cbaf2b3bf/lf-in-filenames-is-confusing-babel.png?auto=compress%2Cformat)

10 ファイルもないようなときは「ファイル一覧を見ればが付くでしょ」という感じですが、100 ファイルあたりになると「かすみ目が気になるお年頃」だとちょっと厳しいかなと。

ちなみに下記の記事にあるようなことを組み合わせると `&sh hos.md` を `&. hos.md` にもできます[^library]。

@[card](https://zenn.dev/kariya_mitsuru/articles/2f05751ccf4cca)

[^library]: さらにちなみに、`LD_LIBRARY_PATH` でも同じようなことになった記憶があります。
