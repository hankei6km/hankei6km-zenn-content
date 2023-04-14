---
id: read-secrets-from-gha-self-hosted-runner
title: GitHub Actions の セルフホステッドなランナーで実行しているジョブからシークレットを読みだす実験
emoji: 🔓
type: tech
topics:
  - githubactions
published: true
---

ワークフローでシークレットを使うときに下記のような注意点があります。

*   [暗号化されたシークレットのワークフロー内での利用](https://docs.github.com/ja/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow)

> 可能であれば、コマンドラインからプロセス間でシークレットを渡すのは避けてください。 コマンドライン プロセスは、他のユーザーに表示される (`ps` コマンドを使用)、または[セキュリティ監査イベント](https://docs.microsoft.com/windows-server/identity/ad-ds/manage/component-updates/command-line-process-auditing)によってキャプチャされる可能性もあります。 シークレットの保護のために、環境変数、`STDIN`、またはターゲットのプロセスがサポートしている他のメカニズムの利用を検討してください。

なんとなく「セルフホステッドなランナーを考慮した注意点なのかな？」くらいに思っていて、「ドキュメントが言うなら引数にシークレットは使わないようにしておこう」という感じでした。

そして少し前に下記のような記述を見て「少なくともセルフホストテッド用の対策ではある」とわかったのですが、それと同時に「ここに書かれている通りならば引数だけ気を付けてもダメなのでは？」とも思いました[^isolate]。

*   [セルフホストランナーを強化する](https://docs.github.com/ja/actions/security-guides/security-hardening-for-github-actions#hardening-for-self-hosted-runners)

> これは、セルフホストランナーが1つのジョブだけを実行するという保証がないためです。一部のジョブでは、コマンド ライン引数としてシークレットが使われ、同じランナーで実行している別のジョブで見ることができます (`ps x -w` など)。 これにより、シークレットが漏えいする可能性があります。

そこで、今回はセルフホステッドなランナーで動かしているジョブから、どのような操作でシークレットを読みだすことができるかの実験をしてみます(実験は Linux x64 のランナーを利用しています)。

**図 1 実験に使っているイメージ**

![GitHub のウェブ UI でランナーに利用するイメージとして Linux x64 を選択しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9ff6c7b72ac447978e36fbbecc7f71bd/read-secrets-from-gha-self-hosted-runner-image.png?w=813\&h=356\&auto=compress%2Cformat)

なお、対策についても少し触れますが、実際には要件や環境によって変わってくるのであまり突っ込んだ内容は記述していません。

[^isolate]: セルフホステッドなランナーでも実行中のジョブは何らかの方法で隔離されているのかと思っていました。

## 引数にシークレットはなぜダメなのか

コマンドラインの引数に使うと、今回の場合では `ps` などで見えてしまうという問題があります。[以前にも少し試しましたが](https://zenn.dev/hankei6km/articles/credentials-contained-files-on-github-actions#secret-%E3%82%92%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%B8%E4%BF%9D%E5%AD%98%E3%81%99%E3%82%8B%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89)、下記にシークレットが見えてしまう例を記述します。

**リスト 1-1 一定時間待機するシェルスクリプト(`stay_tuned.sh`)**

```bash
#!/bin/bash

sleep 100
```

**図 1-1 シェル変数(または環境変数)でシークレットを渡しても ps では内容が表示される**

```shell-session
$ MY_TOKEN=qwerty
$ ./stay_tuned.sh "${MY_TOKEN}" &
[1] 7205
$ ps aux -w | grep stay_tune 
hankei6+  7205  0.0  0.0   4360  1504 pts/6    S    03:15   0:00 /bin/bash ./stay_tuned.sh qwerty
hankei6+  7872  0.0  0.0   3468  1664 pts/6    S<+  03:16   0:00 grep stay_tune
```

そして、`ps` は([設定などにもよりますが](https://wiki.archlinux.jp/index.php/%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3#hidepid))他のユーザーのプロセスも見えます。

**図 1-2 hankei6km が引数にシークレットを利用している**

```shell-session
$ id                             
uid=2000(hankei6km) gid=2000(hankei6km) groups=2000(hankei6km)
$ ./stay_tuned.sh "${MY_TOKEN}" &
[1] 8502
```

**図 1-3 hankei7km がその様子を見ている**

```shell-session
$ id
uid=3000(hankei7km) gid=3000(hankei7km) groups=3000(hankei7km)
$ ps aux -w | grep stay_tune
hankei6+  8502  0.0  0.0   4360  1456 pts/6    S    03:19   0:00 /bin/bash ./stay_tuned.sh qwerty
hankei7+  8833  0.0  0.0   3468  1524 pts/7    S<+  03:20   0:00 grep stay_tune
```

よって、セルフホステッドなランナーに関連した話だけに限らず、普段の操作でも注意した方がよさそうです。

::: message

**シェル組み込みの** `echo`

シェル組み込みの `echo` を使った場合、シェル内部で扱われるので `ps` の出力や `/proc/pid/cmdline` などに引数は出現しないと言われています。 (実際に確認していないので断言はできませんが)

ただし、どちらにしても次の問題があるので使わない方が良いかと思います。

:::

::: message

**コマンド履歴**

今回の問題点とは別に、引数は履歴に残るという問題もあります。この場合は変数名が記録されるので、今回のようなケースでは回避できますが、やはり注意しておく点になるかとは思います。

:::

## 環境変数や `STDIN` は安全なのか？

冒頭のドキュメントでは環境変数や `STDIN` (あるいは利用するコマンドが持っている仕組み)を利用するように記述があります。では、それらは安全なのか少し確認してみます。

### 環境変数は権限があれば読み取れる

スクリプト内で環境変数を扱っている `stay_tuned.sh` を用意し、それを実行してみます。

**リスト 2-1 環境変数の hash を表示するシェルスクリプト**

```bash
#!/bin/bash

echo "${MY_TOKEN}" | sha256sum
sleep 100
```

**図 2-1 ps では内容が表示されない**

```shell-session
$ export MY_TOKEN=qwerty
$ ./stay_tuned.sh &
[1] 10125
9ceece10cf8b97d1f1924dae5d14c137fd144ce999ede85f48be6d7582e2dd23  -

$ ps aux -w | grep stay_tune
hankei6+ 10125  0.0  0.1   4492  3408 pts/6    S    03:39   0:00 /bin/bash ./stay_tuned.sh
hankei6+ 10219  0.0  0.0   3468  1596 pts/6    S<+  03:40   0:00 grep stay_tune
```

引数にシークレットを使っていないので `ps` では表示されないことは確認できます。

しかし、Linux では環境変数は `/proc` の下にファイルとして平文で保存されています。これを利用すると各プロセスの環境変数も読みだせます(シェル変数はできません)。

**図 2-2 /proc から環境変数を読みだせる**

```shell-session
$ cat /proc/10125/environ | xargs -0 -I{} echo {} | grep MY_TOKEN 
MY_TOKEN=qwerty
```

ただし、`ps` とは異なり権限で保護されているので、他のユーザーがから読みだすのは難しいです。

**図 2-3 権限で保護されている**

```bash
$ ls -l /proc/13251/environ
-r-------- 1 hankei6km hankei6km 0 Apr 12 03:57 /proc/13251/environ
$ id
uid=3000(hankei7km) gid=3000(hankei7km) groups=3000(hankei7km)
$ cat /proc/13251/environ | xargs -0 -I{} echo {} | grep MY_TOKEN 
cat: /proc/13251/environ: Permission denied
```

よって、同一ユーザーが作った複数のプロセス間では(何らかの措置を講じなければ)お互いに読みだすことができます。

::: message

**環境変数は定義しただけでも保存されている**

上記ではわかりやすくするためにスクリプト内で環境変数を利用していますが、実際に利用していなくいても `environ` に保存されています。

たとえば、ワークフローのステップで `env:` による定義が記述されていれば、そのステップ内で作成されたプロセスの `environ` には保存されることになります。

:::

### `STDIN` もタイミングによっては読みだせる

`STDIN` も環境変数と似たような感じです。`ps` で内容が表示されることはないですが、ファイル記述子が `/proc` の下に作成され(権限があれば)アクセスできます。

今回もシェルスクリプトで実験してみます。

**リスト 2-2 環境変数の hash を表示するシェルスクリプト**

```bash
#!/bin/bash

sleep 100
sha256sum -
sleep 100
```

**図 2-4 /proc の下にファイル記述子が作成されている**

```shell-session
$ MY_TOKEN=qwerty    
$ echo "${MY_TOKEN}" | ./stay_tuned.sh &
[1] 26729 26730
$ ls -l /proc/26730/fd/
total 0
lr-x------ 1 hankei6km hankei6km 64 Apr 11 08:32 0 -> 'pipe:[1825888]'
lrwx------ 1 hankei6km hankei6km 64 Apr 11 08:32 1 -> /dev/pts/2
lrwx------ 1 hankei6km hankei6km 64 Apr 11 08:32 2 -> /dev/pts/2
lr-x------ 1 hankei6km hankei6km 64 Apr 11 08:32 255 -> /project/home/hankei6km/tmp/article/stay_tuned.sh
lrwx------ 1 hankei6km hankei6km 64 Apr 11 08:32 29 -> /dev/ptmx
```

このとき `0` が `STDIN` となるので、これを `cat` すると読みだせます。

**図 2-5 `stdin` を読みだす**

```bash
$ cat /proc/26730/fd/0
qwerty
```

ただし、本来のプロセスが先に読み出していると `cat` しても読みだせません。

**図 2-6 スクリプト側が読みだした後に `cat` しても読み出せない**

```bash
$ echo "${MY_TOKEN}" | ./stay_tuned.sh &
[1] 27083 27084
9ceece10cf8b97d1f1924dae5d14c137fd144ce999ede85f48be6d7582e2dd23  -
$ cat /proc/27084/fd/0 
```

なお、パイプによる値の受け渡しはいくつか種類があるので `STDIN` だけの話でもありません。たとえばプロセス置換も同じようになります。

**図 2-7 プロセス置換に使われているファイル記述子から読みだし**

```bash
$ ./stay_tuned.sh <(echo "${MY_TOKEN}") &
[1] 27760
$ ls -l /proc/27760/fd
total 0
lrwx------ 1 hankei6km hankei6km 64 Apr 11 08:49 0 -> /dev/pts/2
lrwx------ 1 hankei6km hankei6km 64 Apr 11 08:49 1 -> /dev/pts/2
lr-x------ 1 hankei6km hankei6km 64 Apr 11 08:49 11 -> 'pipe:[1832955]'
lrwx------ 1 hankei6km hankei6km 64 Apr 11 08:49 2 -> /dev/pts/2
lr-x------ 1 hankei6km hankei6km 64 Apr 11 08:49 255 -> /project/home/hankei6km/tmp/article/stay_tuned.sh
lrwx------ 1 hankei6km hankei6km 64 Apr 11 08:49 29 -> /dev/ptmx
$ cat /proc/27760/fd/11
qwerty
```

### 他ユーザーからの保護という意味では安全

以上のように、環境変数や `STDIN` も外部から読みだすことはできますが、権限によって保護されています。このとこから、コマンドライン引数に比べると安全(利用できる局面が緩和されている)と言えるかと思います。

*   コマンドライン引数 - すべてのユーザーから見えるので基本的には使わない方が良い
*   環境変数と `STDIN` - 権限がないユーザーから守られていれば良いときは使える(基本的には普段の利用では使えるでよいと思います)

ただし、同一ユーザーが作成したプロセス同士では(一般的な構成の場合)保護がされていない状態です。「自分のプロセスなんだからよいのでは？」ということになりそうですが、外部からジョブを投入できるようなときには問題となってきます。

## セルフホステッドなランナーの場合は？

一般的な環境でシークレットが読みだせる状況などがわかってきたので、セルフホステッドなランナーの場合について試してみます。

### ホストしているシステムから見たジョブ

これも構成などによりますが、[ドキュメントの基本的な操作に従ってランナーを作成した場合](https://docs.github.com/ja/actions/hosting-your-own-runners/adding-self-hosted-runners)、ランナーが待ち受け状態になりジョブが投入される形になります。そして投入されたジョブはとくに隔離されることなくランナーと同じ権限で実行されています。

よって、ホストしているシステム上からは `ps` コマンドでジョブ内のプロセスを見ることができます。

**図 3-1 hankei6km がランナーを開始し、そこへジョブを投入**

```bash
$ id
uid=2000(hankei6km) gid=2000(hankei6km) groups=2000(hankei6km)

$ ./run.sh

√ Connected to GitHub

Current runner version: '2.303.0'
2023-04-12 07:09:25Z: Listening for Jobs
2023-04-12 07:12:47Z: Running job: test
```

**図 3-2 ps すると他のユーザーからでも引数が見える**

```bash
$ id 
uid=3000(hankei7km) gid=3000(hankei7km) groups=3000(hankei7km)
$ ps aux -w | grep stay_tuned
hankei6+ 20002  0.0  0.1   4360  2972 pts/6    S<+  07:12   0:00 /bin/bash ./scripts/stay_tuned.sh qwerty
hankei7+ 20008  0.0  0.0   3468  1616 pts/8    S<+  07:12   0:00 grep stay_tuned
```

一方で環境変数などの `/proc` からの読み出しは権限がないと実施できません。

**図 3-3 環境変数は権限がないと読み出せない**

```bash
# hakei6km 以外のユーザー(hankei7km)へ su
$ id 
uid=3000(hankei7km) gid=3000(hankei7km) groups=3000(hankei7km)
$ cat /proc/20002/environ | xargs -0 -I{} echo {} | grep MY_TOKEN
cat: /proc/20002/environ: Permission denied

# hakei6km へ su
$ id
uid=2000(hankei6km) gid=2000(hankei6km) groups=2000(hankei6km)
$ cat /proc/20002/environ | xargs -0 -I{} echo {} | grep MY_TOKEN
MY_TOKEN=qwerty
```

以上のことから、ホスト側から見たジョブは「自分が実行したプロセスと変わらない」扱いだとわかります。

よって、コマンドラインの引数にシークレットを使わないことは対策の 1 つになりますが、ランナーを作成したユーザーからは環境変数を通して見ることができてしまいます。

::: message

**動的に生成されるトークンでも読みだせる可能性について**

OIDC の利用などで GitHub Actions 側にシークレットを保存していない場合でも、トークンのやり取りは発生しています。たとえば、受け取ったトークンを他のステップで利用するために環境変数経由で渡していれば同じような影響があります。

よって、動的なトークンで失効までの時間が短いとしても注意すべき点になるかと思います。

なお、直接の対策ではありませんが、トークンの利用が完了したら下記の記事で解説されているように失効させておくのも有効な対策かと思います。

@[card](https://zenn.dev/tmknom/articles/github-apps-token#installation%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%83%88%E3%83%BC%E3%82%AF%E3%83%B3%E3%81%AE%E5%A4%B1%E5%8A%B9)

:::

### 同時に投入されたジョブ

ここで、再度ドキュメントを引用してみます。

> これは、セルフホストランナーが1つのジョブだけを実行するという保証がないためです。一部のジョブでは、コマンド ライン引数としてシークレットが使われ、同じランナーで実行している別のジョブで見ることができます (`ps x -w` など)。これにより、シークレットが漏えいする可能性があります。

現状、1 つのランナーでのジョブ同時実行はサポートされていないように見えるのですが(将来はサポート可能性がある？)、1 ユーザーで複数ランナーを開始している場合は同じような状況を作れます。

**図 3-4 1 つのユーザーで複数のランナーを開始しておく**

![GitHub のウェブ UI で複数のランナーが IDLE になっているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3c4dcdf83c6b4d11bf9cf5d1f621ac52/read-secrets-from-gha-self-hosted-runner-multiple.png?w=807\&h=294\&auto=compress%2Cformat)

この状態で通常のワークフロー(ジョブ)とは別に、特定プロセスの `ps` や `/proc/PID/environ` を表示する悪意あるワークフロー(ジョブ)を同時に実行します。

**図 3-5 悪意ある風のワークフロー**

```bash
name: "Test: peek other jobs"
on:
  workflow_dispatch:

jobs:
  peek:
    runs-on: [self-hosted]
    steps:
      - uses: actions/checkout@v3

      - name: peek
        run: |
          for i in $(seq 10); do
            echo "--- ${i}"
            P=$(pgrep stay_tuned.sh || echo "")
            echo "${P}"
            if test -n "${P}" ; then
              ps x -w | grep "${P}"
              cat "/proc/${P}/environ" | xargs -0 -I{} echo {} | grep MY_TOKEN 
              exit
            fi
            sleep 10
          done
```

下記のように `ps` および環境変数経由でシークレットを見ることができています。

```bash
31464 pts/14   S<+    0:00 /bin/bash ./scripts/stay_tuned.sh qwerty
31564 pts/6    S<+    0:00 grep 31464
MY_TOKEN=qwerty
```

これは各ランナーのユーザーを変更するなどで緩和できますが、ランナー内でジョブが複数同時に動いていると別の対策が必要になります。

よって、ドキュメントが示唆しているような状況が実現されると、ジョブ間でもお互いの環境変数を参照できる可能性は高くなります。

### ジョブから見たホスト側の情報

ここまでは「ランナーの外側からシークレットを読み取る」ことについて考えていましたが、 `ps` や `/proc` は「ジョブからホストの情報へアクセスする」ことにも応用できます。また、さらによくないことに、ホスト側の環境変数なども普通に表示できてしまいます。

**図 3-6 シークレットを使っているプロセスや環境変数が見える状態でランナーを開始**

```shell-session
$ export MY_TOKEN=QWERTY     
$ export HOST_SECRET=PASSWORD
$ ~/stay_tuned.sh "${MY_TOKEN}" &
[1] 7807
3dfa76fd9a6efad58196035891a31cd59b1e9862d3f11645c632d5f703db738f  -

$ ./run.sh                   

√ Connected to GitHub

Current runner version: '2.303.0'
2023-04-12 11:02:11Z: Listening for Jobs
2023-04-12 11:02:28Z: Running job: peek
```

**図 3-7 ジョブからもホスト側の `ps` や `/proc` を読み取ることができる**

```shell-session
7807
 7807 pts/6    S      0:00 /bin/bash /home/hankei6km/stay_tuned.sh QWERTY
 8202 pts/6    S<+    0:00 grep 7807
MY_TOKEN=QWERTY
```

**図 3-8 環境変数も普通の方法で読み取れる**

    PASSWORD

今回はわかりやすくするために特定のプロセスの情報だけ表示していますが、 `ps ax -w` で全体のコマンドライン引数を表示させることもできます。また、権限があればホスト側の各種ファイルへもアクセスできてしまいます。

よって、ホスト側のシステムからランナーを隔離しておかないとあまりよろしくない状況になります。

## コンテナによる隔離

ここまでのことから「シークレットが見えてしまう対策として環境の隔離」という対策が思い浮かびます。

隔離するにはいくつか方法がありますが、ここではコンテナ環境を自前で用意する場合にシークレットを見えなくするための注意点などを少し[^capability]。

[^capability]: 今回の問題以外も考慮するならば、他にも capability に気を付けるなども必要になってくるかと思います。

### ユーザー名前空間の利用

コンテナでランナーを隔離した場合でもホスト側からはコンテナ内のプロセスの情報を読み取れます。

**図 4-1 コンテナ内で環境変数にシークレットを利用**

```shell-session
$ podman run --rm -it debian:latest bash
root@f41fc8ed1ba2:/# export MY_TOKEN=qwerty
root@f41fc8ed1ba2:/# sleep 100
```

**図 4-2 ホスト側で読みだせる**

```shell-session
$ ps aux -w | grep "sleep 100"
hankei6+  2443  0.0  0.0   2396   552 pts/0    S+   05:40   0:00 sleep 100
hankei6+  2450  0.0  0.0   6720   700 pts/3    S+   05:40   0:00 grep sleep 100
hankei6km@odv96d:~/workspace$ cat /proc/2443/environ | xargs -0 -I {} echo {} | grep MY_TOKEN
MY_TOKEN=qwerty
```

これについてはユーザー名前空間を利用すると緩和できます。 `podman` では `run` で [`--userns auto`](https://docs.podman.io/en/latest/markdown/options/userns.container.html) を指定するとホスト側に存在しないユーザーが利用されるので、 コンテナ内プロセスの `/proc/PID` へアクセスすることは難しくなります。

**図 4-3 ユーザー名前空間で uid をリマップしてコンテナを作成**

```shell-session
$ podman run --userns auto --rm -it debian:latest bash
root@e442e67ef75f:/# export MY_TOKEN=qwerty
root@e442e67ef75f:/# sleep 100
```

**図 4-4 ホスト側に存在しないユーザーなので権限でエラーになる**

```shell-session
$ ps aux -w | grep "sleep 100"
231072    2808  0.0  0.0   2396   556 pts/0    S+   05:45   0:00 sleep 100
hankei6+  2817  0.0  0.0   6720   704 pts/3    S+   05:45   0:00 grep sleep 100
$ ls -l /proc/2808/environ 
-r-------- 1 231072 231072 0 Apr 13 05:45 /proc/2808/environ
$ cat /proc/2808/environ | xargs -0 -I {} echo {} | grep MY_TOKEN
cat: /proc/2808/environ: Permission denied
```

あわせてコマンドライン引数の対策をしておけばホストとジョブ間での参照を制限しやすくなるかと思います。

### ジョブによってランナーをわける

現状のランナーの挙動のまま「1 つのランナーでジョブを同時実行できる」と仮定した場合、コンテナでランナーを隔離してもジョブ同士は隔離できないことになります。

よって、対応としては干渉させたくないジョブ毎にランナー(コンテナ)を用意しておき、ラベルなどでそれぞれのランナーへ投入されるようにワークフローを記述するという対策になるかと思います。

## おわりに

GitHub Actions のセルフホステッドなランナーで実行しているジョブからシークレットを読みだす実験をしてみました。

ドキュメントでも事前に案内はされていますが、実際に試してみると「ランナーを隔離しないと思っていたより読みだせてしまう」というのが正直な感想です。

*   コマンドライン引数で使っているシークレットは `ps` で見えてしまう
*   環境変数と `STDIN` で使っているシークレットは権限があれば `/proc` 経由で読めてしまう

対策としては「ホストの環境」や「誰から何を保護するのか」などにもよるので具体的なことは書けませんが、下記のような方向になるかと思います。

*   ホストと各ランナーを隔離する

*   干渉させたくないジョブについてはランナーを共有しないようにする

    *   現状では 1 つのランナーでジョブの同時実行はできないように思えるが、ドキュメントではその可能性が示唆されている

*   その上でコマンドライン引数について対策する

[セルフホステッドなランナーはパブリックリポジトリでは使わないように言われている](https://docs.github.com/ja/actions/hosting-your-own-runners/about-self-hosted-runners#self-hosted-runner-security)のでそれらに従っていればリスクは軽減されそうですが、シークレットを扱うときには注意しておく方が良いかなとは思いました。
