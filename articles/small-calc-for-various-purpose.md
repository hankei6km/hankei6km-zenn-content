---
id: small-calc-for-various-purpose
title: ちょっと計算したいときのオラ電卓選手権
emoji: 🧮
type: idea
topics:
  - cli
  - awk
  - jq
published: true
---

テキストを入力しているとき「四則演算、基数変換、日時(UNIX 時間、時差など)、単位変換」などの計算をしたくなることがあります。

そんなときの「各種ツールを電卓的に利用する方法」を列挙していきます。

*   CLI

    *   bc - 四則演算、基数変換など
    *   Node.js - 四則演算、配列操作、UNIX 時間(ms)など
    *   awk - 集計など
    *   jq - 単位変換、文字列のバイト数、時間関連(UNIX 時間操作)など

*   エディター

    *   Vim(filter) - CLI ツールとの組み合わせ

*   GUI

    *   calc.exe - 四則演算、基数変換など
    *   Google(検索) - 時差など

## CLI

### bc - 四則演算、基数変換など

ターミナルを使っているときは `bc` を使うことが多いです。

基本的な使い方はシンプルでコマンドラインから `bc` を実行するとその後に入力した式を計算してくれます。終了は `quit` か `Ctrl` + `d` です。

実際に使うときは、`tmux` で新しい pane を開いて `bc` で計算し(必要であればコピーして)閉じる感じです。

▼ **図 1-1 基本的な操作**

```shell-session
$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
10+20*3-5
65
quit
```

`last` 変数には直前の結果が代入されています。

▼ **図 1-2 直前の結果**

```shell-session
$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
100*20
2000
last-100
1900
```

`scale` で小数点の桁数を指定できます(が、四捨五入などは面倒なのでその辺が必要なときは別の方法にしています)。

▼ **図 1-3 小数点の表示**

```shell-session
$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
5/2
2
scale=1
5/2
2.5
```

`obase=16` を使うと 16 進数で表示されます。

▼ **図 1-4 16 進数表示**

```shell-session
$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
obase=16
10+20
1E
A+1
B
```

`ibase=16` を使うと 16 進数で入力できます(アルファベットは大文字)。なお、この状態から 10 進数入力に戻すには `ibase=A` となります。この辺はこんがらがってきたらいったん終了してしまいます。

▼ **図 1-5 16 進数入力**

```shell-session
$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
10
10
ibase=16
10
16
A
10
a
0
ibase=A
10
10
```

パイプで式を渡すこともできます。コマンドラインからはあまり使いませんが、後述の Vim の filter ではよく使います。

▼ **図 1-6 パイプで式を渡す**

```shell-session
$ echo "10 * 20" | bc
200
```

## Node.js(REPL) - 四則演算、UNIX時間(ms)など

VSCode の Remote Container でターミナルを使っているときに「`bc` はインストールされていないけど `node` は動く」という場合などで利用しています。

コマンドラインから `node` を実行し式を入力(Enter)すれば結果が表示されます。直前の結果は `_` にセットされています。終了は `Ctrl` + `d` です。

以下ではわかりにくいですが、実際の操作では「補完の候補」「確定している情報での結果」を表示してくれるのが便利です。

▼ **図 2-1 基本的な操作**

```shell-session
$ node
Welcome to Node.js v16.14.2.
Type ".help" for more information.
> 10+20*3-5
65
> _-10
55
>
$
```

普通に四捨五入などもできます。

▼ **図 2-2 四捨五入**

```shell-session
$ node
Welcome to Node.js v16.14.0.
Type ".help" for more information.
> 5/2
2.5
> Math.round(5/2)
3
```

UNIX 時間(ms)と日時を変換するときにも利用しています(が、最近は後述の `jq` を使うことが増えています)。

▼ **図 2-3 Date による日時操作**

```shell-session
$ node
Welcome to Node.js v17.9.0.
Type ".help" for more information.
> Date.now()
1651659937086
> new Date(_).toString()
'Wed May 04 2022 19:25:37 GMT+0900 (日本標準時)'
```

Node.js らしい利用も可能。

▼ **図 2-4 配列**

```shell-session
$ node
Welcome to Node.js v16.14.2.
Type ".help" for more information.
> [1,2,3,4,5].map(v=>v*10)
[ 10, 20, 30, 40, 50 ]
```

ただし、表示されるのは式の結果なので `splice` で配列の操作結果を確認するような場合は変数に代入しておく必要があります[^let]。

[^let]: 変数はテンポラリー的なものを再利用することが多いので `const` ではなく `let` を使っています。

▼ **図 2-5 変数の利用**

```shell-session
$ node
Welcome to Node.js v16.14.2.
Type ".help" for more information.
> [1,2,3,4,5].splice(1,1,20)
[ 2 ]
> let a=[1,2,3,4,5]
undefined
> a.splice(1,1,20)
[ 2 ]
> a
[ 1, 20, 3, 4, 5 ]
```

関数などのブロックは複数行で入力できます。

▼ **図 2-6 ブロックの入力**

```shell-session
$ node
Welcome to Node.js v16.14.0.
Type ".help" for more information.
> function add(a,b){
... return a+b
... }
undefined
> add(10,20)
30
> if(add(10,20)>100){
... console.log('>100')
... }else{
... console.log('<=100')
... }
<=100
undefined
```

インタラクティブな入力以外では `node -e` でも式を評価できます。 `-p` を指定すると結果を表示します。

▼ **図 2-7 引数による評価**

```shell-session
$ node -e '10+20*3-5'

$ node -e 'console.log(10+20*3-5)'
65

$ node -p -e '10+20*3-5'
65
```

### awk - 集計など

`awk` は複数の値の合計を求めるときに使っています。方法としては `awk '{sum+=$1} END{print sum}'` を実行した後に数値を貼り付けて、最後に `Ctrl` + `d` といった感じです。

▼ **図 2-8 awk で合計**

```shell-session
$ awk '{sum+=$1} END{print sum}'
# 標準入力待ちになるので以下を貼り付ける
10
20
30
# ここで Ctrl + d を押下
60
```

`awk '{sum+=$1} END{exit sum}'` とすることで結果を終了コードにできます(整数の場合)。`awk` 側のコードを変更せず一捻り加えたいときに扱いやすいです。

▼ **図 2-9 結果を終了コードにする**

```shell-session
$ awk '{sum+=$1} END{exit sum}'
# 標準入力待ちになるので以下を貼り付ける
10
20
30
# ここで Ctrl + d を押下し、以下を実行
$ expr 100 - "${?}"
40
```

集計以外に簡易的な変換ツールの定義としても使えます。同じ計算を連続して行う場合、少し楽になります(が、最近は後述の jq を使うことが多いです)。

▼ **図 2-10 変換ツールとして定義**

```shell-session
$ alias f2m="awk '{print \$1/3.281}'"

$ f2m
1
0.304785
12
3.65742
```

この他にも「awk ワンライナー」などで検索すると平均を求める方法などいろいろでてきます。

@[card](https://leetmikeal.hatenablog.com/entry/20130117/1358423717)

### jq - 単位変換、文字列のバイト数、時間関連(UNIX 時間操作)など

`jq` は JSON を処理するツールですが、電卓的な利用法でも活躍してくれています。

まずはあまり `jq` らしくない利用方法など。

`jq` では標準入力から以下のように複数の値(JSON)を入力できます[^jsonl]。

[^jsonl]: 書式としては JSONL なのかと思ったのですが、複数行でもいけるようです。

▼ **図 2-11 標準入力の利用**

```shell-session
$ jq .
# 標準入力待ちになるで以下を入力
[10,20,30]
# Enter で結果が表示される
[
  10,
  20,
  30
]
# 以下同様
[5,6,7]
[
  5,
  6,
  7
]
# Ctrl + d で終了
```

これを利用すると `awk` のときのように簡易的な変換ツールの定義に使えます。

▼ **図 2-12 変換ツールとして定義**

```shell-session
$ alias f2m='jq "./3.281"'

$ f2m
1
0.30478512648582745
12
3.65742151782993
```

`jq` 組み込みの関数も同じように使えます。

▼ **図 2-13 関数の利用**

```shell-session
# 四捨五入
$ jq "./3.281 | .*100 | round | ./100"
1
0.3
12
3.66

# 合計
$ jq "add"
[10,20,30,40]
100
[50,60,70]
180

# 平均
$ jq "add/length"
[10,20,30,40]
25
[50,60,70]
60
```

配列での入力が面倒な場合は、他のツールと組み合わせて `awk` と同じように入力できます。

▼ **図 2-14 複数行の入力を配列にする**

```shell-session
# 平均
$ tr '\n' ',' | sed -e 's/^/[/' -e 's/,$/]/' | jq 'add/length'
# 標準入力待ちになるので以下を貼り付ける
100
20
50
90
# ここで Ctrl + d を押下
65
```

終了コードを利用する場合は以下のようになります(`awk` とは異なり表示もされます)。

▼ **図 2-15 結果を終了コードにする**

```shell-session
$ tr '\n' ',' | sed -e 's/^/[/' -e 's/,$/]/' | jq 'add | halt_error(.)'
10
20
30
# ここで Ctrl + d を押下
60
$ expr 100 - "${?}"
40
```

少し変わったところでは UTF-8 文字列のバイト数も表示できます。

▼ **図 2-16 バイト数**

```shell-session
$ jq utf8bytelength
"あいうえお"
15
"🙂"
4
```

続いて、`jq` らしい(？)利用方法など。

トークンなどの有効期限が気になることがありますが、「JSON 内に UNIX 時間で記述されている」とパッと見で判断しにくいです。そのようなときは jq で変換しています。

▼ **図 2-17 日時表示(UTC)**

```shell-session
$ jq ".token.expiry_date"  < .clasprc.json
1651375874641

$ jq ".token.expiry_date / 1000 | todate"  < .clasprc.json
"2022-05-01T03:31:14Z"
```

localtime も表示できますが、書式の指定が必要になります。

▼ **図 2-18 日時表示(localtime)**

```shell-session
$ jq '.token.expiry_date / 1000 | strflocaltime("%Y-%m-%d %H:%M:%S")' < .clasprc.json
"2022-05-01 12:31:14"
```

計算もできるので有効期限が終了するまでの分数も表示可能。

▼ **図 2-19 終了までの分数**

```shell-session
$ jq -r '(.token.expiry_date / 1000 - now) / 60 | round | "\(.) min"' < .clasprc.json
55 min
```

また、GitHub CLI では `jq` が使えるので、API の結果に含まれる UNIX 時間にも使えたりします。

▼ **図 2-20 GitHub CLI での利用**

```shell-session
$ gh api rate_limit --jq '.resources.core.reset'
1651665381

$ gh api rate_limit --jq '.resources.core.reset | todate'
2022-05-04T11:56:23Z
```

## エディター

### Vim(filter)

本来の使い方から少し外れますが、Vim の filter を使うことも多いです。filter だと行が置き換わってしまうのが難点ですがコピペとアンドゥで対応できるので、ちょっと確認したいときはそれほど手間ということもありません。

また、プレイグラウンド的なタブを開いて(単純に`:tabnew` しているだけです)、値をコピペしながら使うこともあります。

#### bc

編集中のテキストに `10 + 20 * 30 - 5` のような行があった場合、カーソルをその行に置き `:.!bc` を実行すると計算結果に置き換わります。ここまでキレイに「式だけ」の行はあまり無いですが、あちこちからコピペして計算したいときに重宝します。

また、編集する手間を許容できるなら、以下のようなカンマ(またはタブ)区切りの行の合計を求めることもできます。

▼ **図 3-1 bc による各行の合計**

    10,20,30
    40,50,60

まず、カンマを `+` に置き換えます(方法はお好みで)。

▼ **図 3-2 bc による各行の合計(+ へ置き換え)**

    10+20+30
    40+50+60

その後、`Shift` + `v` などで範囲指定してから` :'<,'>!bc` を実行すると計算結果に置き換わります。

▼ **図 3-3 bc による各行の合計(結果)**

    60
    150

これは、記号の入力が面倒そうですが、範囲選択してから `:` を押下すると`:'<,'>` になります。よって、あとは `!bc` を入力するだけです。なお、 `!{motion}` の方が打鍵数を削減できますが、入力履歴の使いやすさから範囲選択を利用しています。

#### awk, jq など

filter を使えばファイルに書き出さなくとも上記と同様に `awk` や `jq` を利用できます。

以下のようなテキストから合計を求める場合、行を `+` で連結して `bc` もできますが、範囲選択しておき、 `:'<,'>!awk '{sum+=$1} END{print sum}'` も利用できます。

▼ **図 3-4 awk による合計**

    10
    20
    30

▼ **図 3-5 awk による合計(結果)**

    60

カンマ区切りの場合は、`bc` の他に `jq` を使う方法もあります。

▼ **図 3-6 jq による各行の合計**

    10,20,30
    40,50,60

行頭と行末に `[` と `]` を付けます(方法はお好みで)。これを範囲選択して `:'<,'>!jq add` とすると行の合計に置き換わります。

▼ **図 3-7 jq による各行の合計(編集)**

    [10,20,30]
    [40,50,60]

▼ **図 3-8 jq による各行の合計(結果)**

    60
    150

`bc` を使う方法に比べると `:'<,'>!jq 'add/length'` のように平均なども扱えます。

▼ **図 3-9 jq による各行の平均**

    20
    50

## GUI

### calc.exe - 四則演算、基数変換、単位変換など

普段は Windows 上で何らかのターミナルエミュレーターを使っているので calc.exe を使うことも多いです。クリップボード経由で値の受け渡しができるのでコピペしなが使うなら CLI 系と比べても手間が増える感じでもありません。

また、いろいろなモードがあるのでなんだかんだいっても便利です。

▼ **図 4-1 calc.exe でプログラマー電卓**

![calc.exe でプログラマー電卓を利用しつつモード一覧を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ad0e98f6b94440319609767b750e3dc2/small-calc-for-various-purpose-calc-exe.png?w=510\&h=534\&auto=compress%2Cformat)

### Google - 時差など

メールの内容に EDT で 12 時 10 分 30 秒などの記述があった場合に Google で「jst edt 12 時 10 分 30 秒」と入力しています。ただし、「2022 年 5 月 6 日 12 時 10 分 30 秒」のように日付をつけるとうまくいかないのが惜しいところです。

▼ **図 4-2 Google で時差を表示する検索**

![「jst edt 19:00」を Google で検索しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0d2caab1234f4d9bb5f7893356d87f9b/small-calc-for-various-purpose-google.png?w=846\&h=252\&auto=compress%2Cformat)

基数変換、単位変換などもできるので、ブラウザー(含むスマホアプリ)で入力する機会も多い昨今ではそこそこ利用しています。

@[card](https://qiita.com/pocket8137/items/2f3e5ce628bf410ca89b)

ちなみに `w3m` でも利用できるのでターミナルから使うことも可能です。

    Google

    [jst edt 12 時 10 分 ]  [検索]

    すべて 画像 ニュース  動画

    時間換算 / Eastern TimeとJapan Standard Time

    Japan Standard Timeの月曜日 1:10 です
    Eastern Timeの日曜日 12:10 です

## おわりに

各種ツールを電卓的に利用する方法を列挙してみました。

途中で「スプレッドシートを使った方が幸せになれるのでは？」と思ったりもしましたが、頭の体操にもなるので(個人の感想です)今後もいろいろな計算方法をストックしていければと考えています。
