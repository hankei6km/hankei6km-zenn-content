---
id: access-control-to-secrets-used-in-containers
title: コンテナ内で利用するシークレットへのアクセスを制限する
emoji: 🛂
type: tech
topics:
  - container
  - podman
published: true
---

GitHub Actions のセルフホストランナーを試してみると「コンテナへシークレットを渡す必要があるけど、特定プロセス(ジョブ)にはシークレットを見せたくない」という場面がそこそこあります[^regist-runner]。そして、現在のランナーの構造では「シークレットを環境変数で扱っていると少しやりにくい」と感じました。

[^regist-runner]: 具体的にはランナーの登録を自動化する場合などです。コンテナの外側で登録しておく方法もありますが、登録後の状態(資格情報)を受け渡しすることになるので、それはそれで手間が増えるかと思います。

そのようなわけで、コンテナ環境が用意してくれているシークレットを扱う方法を試してみました。

## シークレットを環境変数で扱うことについて

この辺のことを検索してみると、シークレットを環境変数で渡すのはやめた方が良いよという記事がわりとヒットします。

@[card](https://www.trendmicro.com/ja_jp/research/22/j/analyzing-hidden-danger-of-environment-variables-for-keeping-secrets.html)

そして、環境変数だと「アクセスの制限をやりにくい」とは皆さん考えていらっしゃるようです。

@[card](https://blog.diogomonica.com//2017/03/27/why-you-shouldnt-use-env-variables-for-secret-data/)

> Environment variables are passed down to child processes, which allows for unintended access. This breaks the principle of least privilege. Imagine that as part of your application, you call to a third-party tool to perform some action—all of a sudden that third-party tool has access to your environment, and god knows what it will do with it.

この辺は状況にもよるかとは思いますが、やはり環境変数を利用しない方法を調べておくことも必要かなと感じます。

## コンテナでシークレットを扱う方法

少し調べると Docker (Swarm) と Podman では下記のような方法が用意されています。

Docker Swarm の場合。

@[card](https://docs.docker.com/engine/swarm/secrets/)

Podman の場合。

@[card](https://www.redhat.com/sysadmin/new-podman-secrets-command)
@[card](https://docs.podman.io/en/latest/markdown/podman-secret.1.html)

今回は環境が用意しやすい Podman で試してみます。

なお、クラウドサービスなどでコンテナを動かしている場合は、シークレットを扱うサービスを使うことになるかと思います。その辺については今回は省略します。

## Podman にシークレットを追加

今回はテストで使うだけなので [CodeSandbox でサンドボックスを作成し](https://codesandbox.io/docs/learn/sandboxes/overview?tab=cloud)、そのターミナル内で検証します。

**図 3-1 CodeSandbox の Podman(おそらくルートレス設定)**

```shell-session
$ podman version
Client:       Podman Engine
Version:      4.4.0
API Version:  4.4.0
Go Version:   go1.19.1
Git Commit:   3443f453e28169a88848f90a7ce3137fc4a4bebf
Built:        Sun Apr 16 13:36:25 2023
OS/Arch:      linux/amd64
```

準備ができたら、下記のような手順で Podman にシークレットを追加できます。

**図 3-2 `secret.txt` ファイルの内容をシークレットとして追加**

```shell-session
$ cat secret.txt 
abcdef
$ podman secret create my_secret secret.txt
d81cef0cd0251a1cff1e68e2f
$ podman secret ls
ID                         NAME        DRIVER      CREATED        UPDATED
d81cef0cd0251a1cff1e68e2f  my_secret   file        8 seconds ago  8 seconds ago
```

なお、`create` コマンドのデフォルトで追加したシークレットは**暗号化されていない**ので注意してください。今回は概要をつかむのが目的なので、この記事ではこのまま進めます(暗号化については下記にメモがあります)

::: message

Podman ではシークレット情報を扱うためにドライバーが利用されます。

公式なドキュメントは見つからなかったのですが、現状では下記の 2 つを利用できるようです。

*   `file` : デフォルトで利用されるドライバー(コンテナホストに暗号化されず保存される)
*   `pass` : [pass](https://www.passwordstore.org/) のパスワードストアを利用するドライバー

`pass` ドライバーを利用するときは、シークレット作成時に `--driver` で指定します。

```shell-session
$ podman secret create --driver pass MY_SECRET secret.txt
```

また、スクラップの方で実際に利用したメモを記述してあります。

@[card](https://zenn.dev/hankei6km/scraps/0fc4f1cbf9885c)

:::

## シークレットを環境変数として利用

まずは、シークレットを環境変数として利用した場合、「特定のプロセスにはシークレットを見せたくない」を実現するにはどのくらい気を使うか確認してみます。

確認に利用するコンテナイメージは下記のように作成しています。

**リスト 4-1 Dockerfiel の抜粋**

```docker
FROM alpine:latest

RUN adduser -D -u 2000 hankei6km && \
  adduser -D -u 3000 hankei7km
```

Podman では `run` に `--secret` を指定するとコンテナ内で利用できます。

**図 4-1 シークレットを環境変数として利用**

```shell-session
$ podman run --rm -it --secret my_secret,type=env,target=MY_SECRET test1               
/ # echo $MY_SECRET
abcde
```

ここから、環境変数の場合を簡単に試してみます。

### 各プロセスにコピーされる(変更を各プロセスへ反映させにくい)

**図 4-2 子プロセスにコピーされるが、子プロセスの変更は親プロセスには伝播しない**

```shell-session
$ podman run --rm -it --secret my_secret,type=env,target=MY_SECRET test1
/ # echo $MY_SECRET
abcdef
/ # ash
/ # echo $MY_SECRET
abcdef
/ # unset MY_SECRET
/ # echo $MY_SECRET

/ # exit
/ # echo $MY_SECRET
abcdef
/ # ash
/ # echo $MY_SECRET
abcdef
```

**図 4-3 `exec` すればセットされている**

```shell-session
$ podman run --rm -it --secret my_secret,type=env,target=MY_SECRET test1               
/ # echo $MY_SECRET
abcdef
/ # unset MY_SECRET
/ # echo $MY_SECRET

$ podman exec -it sharp_lamport ash 
/ # echo $MY_SECRET
abcdef
```

**図 4-4 `unset` してからの子プロセスでも見る方法はある**

```shell-session
$ podman run --rm -it --secret my_secret,type=env,target=MY_SECRET test1
/ # echo $$
1
/ # unset MY_SECRET
/ # ash
/ # echo $MY_SECRET

/ # tr '\0' '\n' < /proc/1/environ | grep MY_SECRET
MY_SECRET=abcdef
```

### アクセス制御が難しい

**図 4-5 どのユーザーからも見える**

```shell-session
$ podman run --rm -it --secret my_secret,type=env,target=MY_SECRET -u hankei6km test1
/ $ id
uid=2000(hankei6km) gid=2000(hankei6km) groups=2000(hankei6km)
/ $ echo $MY_SECRET
abcdef
```

**図 4-6 `sudo` や `su -` などで環境をコピーしなければ緩和できる**

```shell-session
$ podman run --rm -it --secret my_secret,type=env,target=MY_SECRET test1             
/ # echo $$
1
/ # echo $MY_SECRET
abcdef
/ # su - hankei6km
20e574abadf5:~$ echo $MY_SECRET

20e574abadf5:~$ tr '\0' '\n' < /proc/1/environ | grep MY_SECRET
-ash: can't open /proc/1/environ: Permission denied
```

**図 4-7 しかし、他の環境変数も見えなくなる**

```shell-session
$ podman run --rm -it --secret my_secret,type=env,target=MY_SECRET --env PARAM1=12345 test1
/ # echo $PARAM1
12345
/ # su - hankei6km
d6d8e16b160a:~$ echo $PARAM1

```

**図 4-8 また、同じユーザーの別プロセスが環境をコピーしていたら見える**

```shell-session
$ podman exec -it -u hankei6km determined_mclaren ash
/ $ echo $$
8
```

```shell-session
20e574abadf5:~$ id
uid=2000(hankei6km) gid=2000(hankei6km) groups=2000(hankei6km)
20e574abadf5:~$ echo $$
2
20e574abadf5:~$ tr '\0' '\n' < /proc/8/environ | grep MY_SECRET
MY_SECRET=abcdef
```

### 引数につかいがち

**図 4-9 どこかで引数に使ってると `ps` で見える**

```shell-session
$ podman run --rm -it --secret my_secret,type=env,target=MY_SECRET test1
/ # cat /usr/local/bin/runner.sh 
#!/bin/sh
unset MY_SECRET
su - hankei6km
/ # runner.sh "${MY_SECRET}"
25a5743be06e:~$ echo $MY_SECRET

25a5743be06e:~$ ps
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    3 root      0:00 {runner.sh} /bin/sh /usr/local/bin/runner.sh abcdef
    4 hankei6k  0:00 -ash
    5 hankei6k  0:00 ps
```

## シークレットをファイルとして利用

シークレットを環境変数にした場合、アクセスを制御するのは少し難しいことを確認してみました。

次は、ファイルとして配置した場合にどれくらい緩和できるか確認してみます。

::: message

下記の結果は Podman の場合です。他の環境でファイルとして利用した場合でもファイルの扱いなどは異なる結果になる可能性があります。

:::

### 1 つのファイルとして扱われる(変更を各プロセスへ反映させやすい)

**図 5-1 シークレットをファイルとして利用(`--secre` でシークレット名だけ渡すとファイルになる)**

```shell-session
$ podman run --rm -it --secret my_secret test1
/ # ls -al /run/secrets/
total 12
drwxr-xr-x    2 root     root          4096 Apr 19 09:16 .
drwxr-xr-x    3 root     root          4096 Apr 19 09:16 ..
-r--r--r--    1 root     root             6 Apr 19 09:16 my_secret
/ # cat /run/secrets/my_secret
abcdef
/ # cat /run/secrets/my_secret
abcdef
/ # su - hankei6km
116b2838aa8f:~$ cat /run/secrets/my_secret 
abcdef
116b2838aa8f:~$ exit
/ # rm /run/secrets/my_secret 
rm: can't remove '/run/secrets/my_secret': Resource busy
/ # echo 123456 > /run/secrets/my_secret 
/ # cat /run/secrets/my_secret
123456
```

**図 5-2 ファイルを変更すると各プロセスにも反映される**

```shell-session
$ podman run --rm -it --secret my_secret test1
/ # echo $$
1
/ # cat /run/secrets/my_secret
abcdef
/ # ash
/ # echo $$
3
/ # cat /run/secrets/my_secret
abcdef
/ # echo 123456 > /run/secrets/my_secret 
/ # cat /run/secrets/my_secret
123456
/ # exit
/ # cat /run/secrets/my_secret
123456

$ podman exec -it -u hankei6km mystifying_hugle ash
/ $ cat /run/secrets/my_secret 
123456
```

*   シークレットのファイルはオーナーが `root:root` で全員に読める権限がある

*   何回でも読める

    *   1 回だけというオプションがあるとうれしかったが、それはなさそう

*   削除はできない

*   `root` は権限に関係なく読み書きできる(これについて後述します)

*   ファイルは 1 つなので変更は各プロセスに反映される

なお、書き込み権限がない状態でも `root` で書き込みできるのは、今回の環境では `CAP_DAC_OVERRIDE` が有効になっているためです。
**図 5-3 `--cap-drop CAP_DAC_OVERRIDE` で厳密にチェックさせる**

```shell-session
$ podman run --rm -it --secret my_secret --cap-drop CAP_DAC_OVERRIDE test1 
/ # ls -l /run/secrets/my_secret 
-r--r--r--    1 root     root             6 Apr 19 13:45 /run/secrets/my_secret
/ # cat /run/secrets/my_secret
abcdef
/ # echo 123456 > /run/secrets/my_secret 
/bin/sh: can't create /run/secrets/my_secret: Permission denied
/ # cat /run/secrets/my_secret
abcdef
/ # exit
$ podman run --rm -it --secret my_secret,mode=0644 --cap-drop CAP_DAC_OVERRIDE test1
/ # ls -l /run/secrets/my_secret 
-rw-r--r--    1 root     root             6 Apr 19 13:47 /run/secrets/my_secret
/ # cat /run/secrets/my_secret
abcdef
/ # echo 123456 > /run/secrets/my_secret 
/ # cat /run/secrets/my_secret
123456
```

このような感じでシークレットを利用した後、内容を上書きしておけば利用できなくなります(コンテナを再作成すれば再度利用できるようになります)。

::: message

`shred` を使うこともできますが、こちらも削除はできません。

```shell-session
$ podman run --rm -it --secret my_secret,mode=0644 --cap-drop CAP_DAC_OVERRIDE test1
/ # shred -z /run/secrets/my_secret 
/ # tr '\0' . < /run/secrets/my_secret 
......
/ # shred -u /run/secrets/my_secret 
shred: can't remove file '/run/secrets/my_secret': Resource busy
```

なお、シークレットとして配置されたファイルを上書きするとき、`shred` を使うことによる利点が活かされるのかはわかりませんでした。

:::

### ユーザーによる制限

**図 5-4 ファイルのオーナーと権限を変更できる**

```shell-session
$ podman run --rm -it --secret my_secret,uid=2000,gid=2000,mode=0400 test1
/ # ls -al /run/secrets/
total 12
drwxr-xr-x    2 root     root          4096 Apr 19 09:38 .
drwxr-xr-x    3 root     root          4096 Apr 19 09:38 ..
-r--------    1 hankei6k hankei6k         6 Apr 19 09:38 my_secret
/ # su - hankei6km
1f007745f494:~$ cat /run/secrets/my_secret
abcdef
1f007745f494:~$ exit
/ # su - hankei7km
1f007745f494:~$ cat /run/secrets/my_secret 
cat: can't open '/run/secrets/my_secret': Permission denied
```

**図 5-5 環境変数は別の仕組みになるので影響されない**

```shell-session
$ podman run --rm -it --secret my_secret,uid=2000,gid=2000,mode=0400 -u hankei7km --env PARAM1=12345 test1
/ $ cat /run/secrets/my_secret 
cat: can't open '/run/secrets/my_secret': Permission denied
/ $ echo $PARAM1
12345
```

こちらも環境変数とは異なりシークレットへのアクセスだけを制御をしやすくなります。

### コマンドラインでは扱いにくいこともある

ここで少しデメリットも。

環境変数とは異なり、コマンドライン引数としては扱いにくくなります。

これについては引数にシークレットを使うこと自体があまりよくないこととされています。シークレットをコマンドライン引数にしないコマンドも増えてきているので、ファイルとして渡すことを検討するのが良いかと思います。

たとえば、最近使ったなかでは [`oras`](https://oras.land/cli_reference/oras_login/) コマンドはパスワード(トークン)を `STDIN` から読み取るフラグがありました。

**図 5-6 `oras` コマンドでは `STDIN` からもパスワードを渡せる**

```shell-session
$ oras login "${REGISTRY}" --username "${USER}" --password "$(cat /run/secrets/my_secret)"
$ oras login "${REGISTRY}" --username "${USER}" --password-stdin < /run/secrets/my_secret
```

## おわりに

コンテナ内で利用するシークレットへのアクセスを制御するために、Podman のシークレット機能を試してみました。

シークレットがファイルとして配置されていると「いつまで見せるか」「誰に見せるか(あるいは見せない)」を制御しやすいと感じました。たとえば、「シークレットは初期化処理のときだけ必要で、後続の処理では利用しない」のようなときに活用できそうです。

各種コンテナ実行環境で共通化された仕組みでないのが少し難点ですが、シークレットをコンテナ内のファイルとして配置できるオプションがあるときは状況に応じて使い分けていれけばと思っています。
