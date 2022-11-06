---
id: command-command-v
title: alias は勘定に入れません ─ あるいは、消えたコマンドを実行しようとする謎
emoji: 🐶
type: tech
topics:
  - path
  - shell
  - bash
  - shellcheck
published: true
---

シェルスクリプトで `which` を使って「コマンドを実行したときに**どのファイル**が使われるのかを確認」していたところ、ShellCheck から下記のようなメッセージが表示されました[^optional]。

[^optional]: `enable=deprecate-which` で有効化しないと表示されない場合もあります。

@[card](https://github.com/koalaman/shellcheck/wiki/SC2230)

    which vi
    ^---^ SC2230: which is non-standard. Use builtin 'command -v' instead.

    For more information:
      https://www.shellcheck.net/wiki/SC2230 -- which is non-standard. Use builti...

これについて、`command` と下記の議論などを調べてみると「コマンド実行時の挙動は `PATH` だけでは決まらないかな」とちょっと感じました。

@[card](https://github.com/koalaman/shellcheck/issues/1162)

そこで、今回は ShellCheck おすすめの `command` の挙動を確認しながら、「コマンドを実行したときに**何が**実行されるのか？」についてなど。

::: message

今回の記事では「コマンドを実行」という記述は、基本的に「Bash のプロンプトから(`/` を含まない)コマンドの入力あるいはシェルスクリプト内での利用」を指しています。

:::

## `which` は `PATH` 上にある実行可能なファイルを表示するが `alias` などは考慮しない(環境や実装にもよります)

`which` の挙動を改めて考えてみると「どのファイルが？」を調べるにはよいのですが、「何が実行されるのか？」とは相性が悪そうに思えてきます。

たとえば、`chou-bennri` というコマンドについて調べたいとします。普通は「そんなコマンドは無い」となりそうですが、`alias` を使っていると結果が変わってきます。

**図 1-1 `which` は `alias` を考慮しない**

```shell-session
$ alias chou-bennri=figlet # コマンドをエイリアスとして定義

$ which chou-bennri # エイリアスは見つからない

$ echo "${?}"
1

$ chou-bennri OK # しかしエイリアスは置換されコマンドが実行される
  ___  _  __
 / _ \| |/ /
| | | | ' /
| |_| | . \
 \___/|_|\_\
```

エイリアス定義はコマンド検索とは別にシェル内で置換が行われるので、`which` やシェルを通さないファイル実行とは多くの場合で結果が一致しなくなります。

*   [Shell Command Language # Alias Substitution](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_03_01)
*   [Aliases (Bash Reference Manual)](https://www.gnu.org/software/bash/manual/html_node/Aliases.html)

他にもコマンドとして実行できる形態にはシェルの関数や組み込みコマンドなどがあります。

*   [Shell Command Language # Command Search and Execution](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_09_01_01)
*   [Command Search and Execution (Bash Reference Manual)](https://www.gnu.org/software/bash/manual/html_node/Command-Search-and-Execution.html)

以上のことから、`which` はあくまでも `PATH` 上に実行可能なファイルが存在していたら表示するコマンドであり、「何が実行されるのか？」を調べるコマンドとは目的が異なると言えます。また、後述しますが実は「**実行可能**なファイル」という扱いも少し都合が悪かったりします。

なお、少し補足しておくと `which` でも環境や実装によっては期待している動作をする場合もあるようです。下記以外にもいくつかの場所で同様の挙動をみかけました(逆に言うと環境などに左右されてしまうので、扱いにくいとも言えます)。

@[card](https://github.com/koalaman/shellcheck/issues/1162#issuecomment-1157513410)

>     $ alias grep=true
>     $ command -v grep
>     alias grep=true
>
>     $ which grep # on IBM AIX
>     /opt/freeware/bin/grep
>     $ which grep # on RHEL7
>     alias grep='true'
>             /usr/bin/true

## `command` コマンドとは？

ShellCheck おすすめの `command` とはどのようなコマンドなのでしょうか？

`command -v is a POSIX standard builtin` とあるのでシェル組み込みのコマンドということはわかります。それではと Bash のマニュアルなどを参照してみると概要が確認できます。

*   [command](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/command.html)
*   [Bash Builtins (Bash Reference Manual) # command](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html#index-command)

軽く読んでみると下記のような感じです。

1.  引数としてコマンド文字列を渡すとエイリアスなどを無視して実行する
2.  オプション(`-Vv` )を指定することでコマンドについて表示する
3.  オプション(`-p`)を指定することで検索する`PATH` を変更する

## `command` コマンドの基本的な挙動

概要がわかってきたので、ここからは Bash 組み込みの `command` で挙動を確認してみたいと思います。

::: message

確認に利用している Bash では基本的に **POSIX モードを有効化していません**。よって、POSIX 準拠シェルとは異なる挙動の場合があります。

また、逆に POSIX 準拠シェルで共通の挙動だと思われても(実際に確認はしていないので)「Bash は」という記述にしています。

:::

**図 3-1 `command` の基本的な操作**

```shell-session
$ command figlet OK # 実行ファイルとして figlet は実行される
  ___  _  __
 / _ \| |/ /
| | | | ' /
| |_| | . \
 \___/|_|\_\

$ alias chou-bennri=figlet # エイリアスとして定義

$ command chou-bennri OK # エイリアスは実行されない
bash: chou-bennri: command not found

$ unalias chou-bennri

$ function chou-bennri { figlet "${@}"; } # 関数として定義

$ command chou-bennri # 関数は実行されない
bash: chou-bennri: command not found
```

`command` の基本的な挙動としては「コマンド文字列を渡すと組み込みコマンドまたは `PATH` に存在するファイルのみを実行する[^filepath]」になります。上記のようにエイリアス設定されたコマンドなどは実行されません。

[^filepath]: `/` を含むファイル名をコマンド文字列で渡せば `PATH` 検索とは関係なく実行されます。

## `command` コマンドを利用して「何が実行されるのか？」を確認する

基本的な挙動を確認できたので、`command` を利用して「何が実行されるのか？」を確認する方法について見ていきたいと思います。

これには ShellCheck のメッセージでも提示されていた `-v` が利用できます。

エイリアスなどが設定されているとその内容を表示し、見つからなければ `PATH` の内容などからファイル名を表示します。最終的に見つからない場合、終了ステータスは `0` 以外になります。また、`-v` は概要だけですが、`-V` では関数の内容なども表示されます。

**図 4-1 `-v` でコマンドについて表示**

```shell-session
$ command -v chou-bennri # chou-bennri コマンドは見つからない

$ echo "${?}"
1

$ alias chou-bennri=figlet # エイリアスとして定義

$ command -v chou-bennri # エイリアスとして表示される
alias chou-bennri='figlet'

$ echo "${?}"
0

$ unalias chou-bennri

$ function chou-bennri { figlet "${@}"; } # 関数として定義

$ command -v chou-bennri  # 関数として表示される
chou-bennri

$ echo "${?}"
0

$ command -V chou-bennri # -V で詳細が表示される
chou-bennri is a function
chou-bennri ()
{
    figlet "${@}"
}

$ command -v figlet # 実行ファイルを指定するとファイル名が表示される
/usr/bin/figlet
```

上記のような挙動なので `command -v` は「エイリアスなども考慮した `which`」のように見えますが、実際には下記に挙げた Bash の細かい挙動に合致するように表示が行われます。

## Bash のコマンド実行時の挙動と `command -v` について

ここでは Bash が(少し特殊な状況で)実行対象を選択している挙動と、そのときの `command -v` について確認してみたいと思います。

### Bash は実行したコマンドのファイル名などを記憶する、また記憶を書き換えることもできる、そしてコピーはされない

おそらくはパフォーマンス向上のために、Bash は一度実行したコマンドのファイル名などを記憶しています。

*   [hash](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/hash.html)
*   [Bourne Shell Builtins (Bash Reference Manual) # hash](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html#index-hash)

記憶している内容は `hash` コマンドで閲覧・操作できます。

**図 5-1 `hash -l` で記憶している内容を確認**

```shell-session
$ vi

$ hash -l
builtin hash -p /usr/bin/vi vi
```

また、`hash` コマンドで `-p` を利用すると「`PATH` を通してないディレクトリー上のファイルをコマンド名で実行できる」ようになります。

**図 5-2 `hash -p` で任意のファイルを追加**

```shell-session
$ echo -e '#!/bin/bash\necho "fake vi"' > vi # 偽 vi を作成

$ chmod u+x vi

$ hash -p /home/vscode/tmp/vi vi # 偽 vi のファイル名を vi として追加

$ which vi # 通常の vi を指している
/usr/bin/vi

$ command -v vi # 追加したファイルを指している
/home/vscode/tmp/vi

$ vi # Bash は偽 vi(追加したファイル)を実行する
fake vi
```

`which` では `PATH` を検索した情報が表示されますが、`command -v` の表示には `hash` コマンドの操作も反映されています。

これで一件落着となりそうですが、新しい環境が作成さるれときに記憶はコピーされないため、シェルスクリプトなどでは新しく検索されます。

**図 5-3 記憶している内容はコピーされない**

```shell-session
$ echo -e '#!/bin/bash\necho "fake wc"' > wc # 偽 wc を作成

$ chmod u+x wc

$ hash -p /home/vscode/tmp/wc wc # 偽 wc のファイル名を wc として追加

$ echo -n 123 | wc -c # 偽 wc が実行される
fake wc

$ echo -e '#!/bin/bash\necho -n 123 | wc -c' > chk.sh #  wc を実行するスクリプト

$ chmod u+x chk.sh

$ ./chk.sh # 通常の wc が実行されている
3
```

このコピーされない挙動も `command -v` に影響があるので、後の節で少し追記します。

なお、上記のようなこともあるので「何が実行されるのか？」は「まっさらな状態で調べたい」ということもあるかと思います。その場合も`hash` コマンドが利用できます。

**図 5-4 `hash -r` コマンドで記憶されているファイル名を削除**

```shell-session
$ command -v wc
/home/vscode/tmp/wc

$ hash -r

$ command -v wc
/usr/bin/wc
```

`hash -r` の後では `PATH` を検索した結果を表示しています[^hash-d]。

[^hash-d]: `-r` はすべての記憶を削除してしまいます。気になる場合は `$ hasd -d vi` で個別に削除できますが、事前に記憶していない場合はエラーになるので注意してください。

### Bash はプロンプト(現在の環境)とスクリプト(新しい環境)で「何が実行されるのか？」は結構違うことがある

`command -v` は Bash の挙動にあわせて表示していますが、それゆえに注意する点もあります。

とくにシェルスクリプト内ではプロンプトの環境とは各種定義が異なるので、 `command -v` の表示が思っているよりも異なっている状態になります。

全部を検証すると長くなるので、ここではエイリアスについて確認してみます[^others]。

[^others]: この他、`command -v` に影響が出そうなものとしては「シェル変数(エクスポートしていない `PATH` など)と関数、記憶しているファイル名」もコピーなどがされないので定義が異なります。なお、シェル変数は `export` 、関数は `export -f` でエクスポートできますが、これはこれで副作用が大きくなりそうです。

**図 5-5 エイリアスは新しい環境にはコピーされない**

```shell-session
$ alias tmp_alias="echo ALIAS"

$ tmp_alias
ALIAS

$ command -v tmp_alias
alias tmp_alias='echo ALIAS'

$ bash # 新しい環境を開始

$ tmp_alias
bash: tmp_alias: command not found

$ command -v tmp_alias
```

上記のように現在の環境で作成したエイリアスは新しい環境にはコピーされません。ここで「`.bashrc` で定義しているものは自動的に定義されるのでは？」となりますが、シェルスクリプトでは `.bashrc` は読み込まれません[^shell-script]。

[^shell-script]: スクリプトを実行するシェルが Bash 以外のこともあります。その他にも `.bashrc` を強引に読み込んでも不用意に実行されないようになっているかと思います。<https://qiita.com/takaram/items/17e739e9b7d4d7b6de42>

また、エイリアス特有の挙動として(デフォルト設定の場合[^expand-alias-in-non-itaratcive])シェルスクリプト内では置換されないようになっています[^source]。

[^expand-alias-in-non-itaratcive]: 設定を変更すると置換されます。<https://genzouw.com/entry/2020/03/16/090918/1947/>

[^source]: `source` で読み込んだ場合は置換されます。

**図 5-6 シェルスクリプト内では置換されない**

```shell-session
$ cat <<EOF > chk.sh
> #!/bin/bash
> alias tmp_alias="echo ALIAS"
> alias -p
> command -V tmp_alias
> tmp_alias
> EOF

$ chmod u+x chk.sh

$ ./chk.sh
alias tmp_alias='echo ALIAS'
./chk.sh: line 4: command: tmp_alias: not found
./chk.sh: line 5: tmp_alias: command not found
```

このような状況なので、シェルスクリプト内で実行した `command -v` の表示は間違いではないのですが、対話形式で利用する場合「期待している表示ではない」ともいえます。

**図 5-7 シェルスクリプト内で `command -v`**

```shell-session
$ alias ll # .bashrc で定義されているエイリアス
alias ll='ls -alF'

$ alias tmp_alias="echo ALIAS" # 現在の環境でエイリアスを定義

$ command -v ll # .bashrc で定義されているエイリアスを表示
alias ll='ls -alF'

$ command -v tmp_alias # 現在の環境で定義したエイリアスを表示
alias tmp_alias='echo ALIAS'

$ echo -e '#!/bin/bash\ncommand -v "${@}"' > chk.sh #  command -v を実行するスクリプトを作成

$ chmod u+x chk.sh

$ ./chk.sh ll # スクリプト内で実行した command -v は .bashrc で定義されているエイリアスを表示しない
$ echo "${?}"
1

$ ./chk.sh tmp_alias # スクリプト内で実行した command -v は .bashrc で現在の環境でえ定義したイリアスを表示しない
$ echo "${?}"
1
```

一方でシェル関数は現在の環境で実行されるので、試した限りでは上記のような食い違いは発生しませんでした。

**図 5-8 シェル関数内で `command -v`**

```shell-session
$ function chk { command -v "${@}"; }

$ chk ll
alias ll='ls -alF'

$ chk tmp_alias
alias tmp_alias='echo ALIAS'
```

`command -v` の表示をカスタマイズしたいような場合では、シェル関数などを利用するのが良いかと思います。

### Bash は削除されたコマンドを実行しようとするときがある

前述のようにファイル名などは記憶されるので、同じコマンドを複数回実行すると 2 回目以降はコマンドの検索が簡略化されています。

普段はとくに気にする必要はないのですが、記憶されているファイルを削除すると少し奇妙な状況が発生します。

**図 5-9 実行したファイルを削除する**

```shell-session
$ echo -e '#!/bin/bash\necho "fake vi"' > ~/.local/bin/vi # 偽 vi 作成

$ chmod u+x ~/.local/bin/vi

$ vi # vi 実行(偽 vi が実行される)
fake vi

$ rm ~/.local/bin/vi #  偽 vi を削除

$ which vi # 通常の vi を指している
/usr/bin/vi

$ command -v vi # 偽 vi を指したまま
/home/vscode/.local/bin/vi

$ vi # vi 実行(削除されたファイルを実行しようとする)
bash: /home/vscode/.local/bin/vi: No such file or directory
```

一度実行したファイルを削除した場合、`which` は削除されたファイルを表示しませんが、 `command -v` は削除されたファイルを表示しています。

これは「`command -v` は正しくない情報を表示している」ように思えます。しかし、実際には Bash は記憶している情報を元に削除されているファイルを実行対象にしています(結果はエラーになっていますが)。

よって「何が実行されるのか？」だけに注目すると `command -v` の方が実際の状況に沿った情報を表示しています(「正しい」とも少し違う気がしたので回りくどい表現にしています)。

::: message

POSIX モードを有効化すると Bash は削除されたファイルを除外して `PATH` を再検索します。よって、`command -v` の表示とは食い違う状況になります。

> When a command in the hash table no longer exists, Bash will re-search `$PATH` to find the new location. This is also available with `‘shopt -s checkhash’`.

*   [Bash POSIX Mode (Bash Reference Manual)](https://www.gnu.org/software/bash/manual/html_node/Bash-POSIX-Mode.html)

:::

### Bash は実行できないファイルを実行しようとするときもある

続いて実行できないファイルの挙動について確認してみます。

**図 5-10 実行権限がないファイルを配置する**

```shell-session
$ touch ~/.local/bin/tadano-file # 実行権限がないファイルを作成

$ which tadano-file # ファイルが見つからない

$ command -v tadano-file # ファイルが表示される
/home/vscode/.local/bin/tadano-file

$ tadano-file # Bash は実行しようとする
bash: /home/vscode/.local/bin/tadano-file: Permission denied
```

この挙動についても最初は「いやいや表示したらダメでしょ」と思ったのですが、Bash は実行権限のないファイルも実行しようとするので「何が実行されるのか？」でいうと利用しやすい情報といえます。

たとえば、`which` では「実行できない」ことは事前に把できますが「コマンドが存在していない」なのか「実行権限がない」なのかは判別できません。`command -v` はひと手間かかりますが、表示された情報を元に判別できます。 (ちょっと調べたいだけなときに、そこまで確認するのかというと微妙なところですが)

なお、上記の挙動を見ると「実行権限がないファイルも `PATH` 上に配置されていれば常に実行される」と思いたくなりますが、そうでもなかったりします(この辺で少し涙目になりました)。

**図 5-11 実行権限がないファイルを配置する(ただし既存コマンドと同名)**

```shell-session
$ which wc # wc は存在している
/usr/bin/wc

$ command -v wc
/usr/bin/wc

$ echo "echo 'fake wc'" > ~/.local/bin/wc # 実行件がない wc を作成

$ which wc # 通常の wc を指している
/usr/bin/wc

$ command -v wc # 通常の wc を指している
/usr/bin/wc

$ echo -n "123" | wc -c # Bash も通常の wc を実行する
3
```

実行権限のないファイルが配置されている場合でも、`PATH` 上に実行できるファイルがあれば最終的にはそれを実行します。これについても `command -v` は実行されるファイルを表示できています。

また、この状況に前述の「一度実行したコマンドのファイル名などを記憶する」を組み合わせることもできますが、長くなるので今回は省略します。 (試した限りでは `command -v` は期待した通りに表示していました)

::: message

ACL(`setfacl`) でユーザー個別に実行権限を変更した状態などでも試していますが、拡張属性の設定などによっては挙動が異なるかもしれません。

:::

## `command` コマンドを利用して標準の `PATH` からコマンドを探す

`command` の挙動について最後は `-p` の動作を確認してみます。これを指定すると、環境変数の `PATH` ではなくデフォルト `PATH` からコマンドを探します。

**図 6-1 デフォルト `PATH` の確認**

```shell-session
$ getconf PATH
/bin:/usr/bin
```

以下は `fzf` をユーザーローカルにインストールしている場合です。

**図 6-2 `-p` による `PATH` の扱い**

```shell-session
$ command -v fzf
/home/vscode/.fzf/bin/fzf

$ command -pv fzf

$ sudo cp /home/vscode/.fzf/bin/fzf /usr/local/bin/
$ command -pv fzf

$ sudo cp /home/vscode/.fzf/bin/fzf /usr/bin/
$ command -pv fzf
/bin/fzf
```

上記例では `-p` を指定すると `/bin/fzf` には反応していますが、`/usr/local/bin/fzf` は無視していることを確認できます。

ただし、`-p` でもシェルが記憶しているファイル名が反映されます。厳密に確認するなら `hash` での回避が必要となります。

**図 6-3 コマンド実行後の `-p`**

```shell-session
$ command -v vi # 偽 vi が配置されている
/home/vscode/.local/bin/vi

$ command -pv vi # 標準の PATH では通常の vi が表示される
/bin/vi

$ vi # vi を実行(偽 vi が実行される)
fake vi

$ command -pv vi # -p を指定しても記憶しているファイルを表示
/home/vscode/.local/bin/vi

$ hash -r # 記憶を消す

$ command -pv vi # 標準の PATH から検索する
/bin/vi
```

「何が実行されるのか？」という目的ではあまり使わさなそうですが、`-v` を指定しないことで実際にコマンドを実行できます。

(どちらかというと、こちらが `command` 本来の使い方のような気もしますが)「ユーザー設定に左右されないコマンド実行」には便利に使えそうです。

**図 6-4 エイリアスやユーザー定義のシェルスクリプトに左右されないコマンド実行**

```shell-session
$ alias figlet=echo # 既存コマンドと同名のエイリアスを定義

$ echo -e '#!/bin/bash\necho "user-script: ${@}"' > ~/.local/bin/figlet # 既存コマンドと同名のスクリプトを作成

$ chmod u+x ~/.local/bin/figlet

$ hash -r

$ figlet 123 # エイリアスが実行される
123

$ hash -r

$ command figlet 123 # エイリアスは実行されないがスクリプトが実行される
user-script: 123

$ hash -r

$ command -p figlet 123 # 標準 PATH から実行される
 _ ____  _____
/ |___ \|___ /
| | __) | |_ \
| |/ __/ ___) |
|_|_____|____/
```

## その他のコマンドによる確認

[冒頭の ShellCheck の解説](https://github.com/koalaman/shellcheck/wiki/SC2230)にも記載がありますが、Bash には `type` という組み込みコマンドもあります。

*   [Bash Builtins (Bash Reference Manual) # type](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html#index-type)

コマンドについて表示するという点では `command -v` と似たような感じですが、「`-a` を使うと実行ファイルについても全て表示する」などの便利な機能があります。

**図 7-1 `-a` で実行可能ファイルの一覧を表示**

```shell-session
$ which -a ls
/usr/bin/ls
/bin/ls

$ type -a ls
ls is aliased to `ls --color=auto'
ls is /usr/bin/ls
ls is /bin/ls
```

ただし、ちょっと惜しい点としては「`-a` では実行できないファイルは not found 扱い」になってしまいます[^exec-then-error]。

[^exec-then-error]: 結果の違いを利用すると「実行されるけどエラーになるファイル」の判別に使えそうな気もしますが、`-a` の不具合かもしれないのでちょっと悩み所です。

**図 7-2 `-a` では実行できないファイルは not found**

```shell-session
$ touch ~/.local/bin/tadano-file

$ command -v tadano-file
/home/vscode/.local/bin/tadano-file

$ type tadano-file # type コマンドでも実行できないファイルが表示される
tadano-file is /home/vscode/.local/bin/tadano-file

$ type -t tadano-file # -t を指定しても表示される
file

$ type -a tadano-file # -a では not found
bash: type: tadano-file: not found
```

また、POSIX には含まれていないので利用できない場合もあるようです。

@[card](https://en.wikipedia.org/wiki/Type_\(Unix\)#History)
@[card](https://github.com/koalaman/shellcheck/issues/1162#issuecomment-491543368)

## シェルを使わないファイル実行

ここまでは Bash でコマンドを実行した場合について確認しました。一方で「シェルを使わないコマンド実行」の方法もあります。たとえば、Node.js の `execFile` などです。

*   [Child process | Node.js v19.0.0 Documentation # child\_process.execFile()](https://nodejs.org/api/child_process.html#child_processexecfilefile-args-options-callback)

軽く確認してみると「エイリアスや関数」あるいは「記憶されたファイル名」などは考慮されませんでしたが、実行できないファイルを対象にはするようです。

**リスト 8-1 `execFileSync` を利用したコード(簡潔にするため同期版を利用しています)**

```js
import { execFileSync } from "node:child_process";

try {
  execFileSync("tadano-file");
} catch (err) {
  console.log(err.code);
}
```

**図 8-1 実行できないファイルがある場合は `EACCES` となる**

```shell-session
$ node index.mjs
ENOENT

$ touch ~/.local/bin/tadano-file

$ node index.mjs
EACCES
```

(プログラム言語の実装にもよるでしょうが)シェルを通さない場合でも `which` の情報だけでは判別できない状況もありそうです。

## 注意点

ここまで確認してきた内容からすると `command -v` は良い感じですが、実際に利用する場合にはひとつ注意点があります。

慣れの問題かと思いますが、`which` のノリで次のようにやってしまいがちです。

**図 9-1 `-v` を忘れている**

```shell-session
$ command abunai-command
```

これは `abunai-command` が実行されるので注意してください(実はこの記事を書いているときに何度かやってしまいました)。

## おわりに

`command` の挙動と「コマンドを実行したときに何が実行されるのか？」について確認してみました。

長々と書いてしまいましたが、「`command -v` は(POSIX モードでなければ)Bash の下記のような挙動にあわせて表示する」ことを確認できたかと思います。

*   エイリアスによる置換やシェル関数、組み込みコマンドなども実行する

*   実行したコマンドのファイル名を記憶している

    *   `hash` コマンドで記憶を操作できる

*   プロンプトとシェルスクリプトでは「何が実行されるのか？」は結構違う

    *   `command -v` の表示などをカスタマイズするときは関数の利用がよさそう

*   削除されたファイルを実行しようとすることもある

*   実行できないファイルを実行しようとすることもある

そのようなわけで、これからは「何が実行されるのか？」の確認は `command -v` を使っていこうかと考えています。

あとは、`command` の基本機能も「ユーザー設定に左右されたくないとき」便利そうなので別途記事にできればと思っています。
