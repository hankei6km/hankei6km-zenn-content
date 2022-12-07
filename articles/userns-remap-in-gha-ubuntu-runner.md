---
id: userns-remap-in-gha-ubuntu-runner
title: Ubuntu runner の Docker でもユーザー名前空間でコンテナを分離(userns remap)
emoji: 👥
type: tech
topics:
  - githubactions
  - docker
published: true
---

GitHub Actions の `ubuntu-22.04` runner で `gid` が変更になった影響で、Docker コンテナを動かしているジョブが落ちるようになってしまいました。 (この辺は後述しますが [DevContaner の uid/gid をホスト側に合致させる機能](https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user)が使えなくなって権限関連でエラーになる)

[上記のエラーは安直に回避](https://github.com/hankei6km/h6-dev-containers/pull/48/commits/d832b48c9572be29e8ec2c06ea7366fb8363d520)したのですが「それはそれとして、コンテナから runner 側にファイルを書き出す方法は少し考えておいた方がよいかな」と思うようになりました。

というわけで今回は、「コンテナから runner(ホスト側)へファイルを書き出すときの問題点」と「ユーザー名前空間によるコンテナの分離を使うとどうなるのか」についてなど。

## Ubuntu runner でコンテナからホスト側へファイルを書き出すときの問題点

コンテナ側からホスト側へ書き出す方法はいくつかありますが、ここでは[バインドマウント](https://docs.docker.jp/storage/bind-mounts.html)について記述します。

Ubuntu runner を使っている場合に限らないのですが、コンテナでホスト側のディレクトリーをバインドマウントしたとき「コンテナ内の uid/gid とホスト側のアクセス権」の問題がでてきます。

たとえば、 ホスト側で id が `1000:100` なユーザーを利用している場合、普通に操作していると「コンテナ側でも `1000:100` で操作しないと権限のエラーになる」ことがあります。

**図 1-1 ホストとコンテナで異なる uid/gid を使うとバインドマウントしたディレクトリーで権限関連のエラーになりやすい**

```shell-session
$ mkdir chk

$ docker run -u guest -v "${PWD}:/mnt" --rm alpine:latest ash -c 'echo "hey, taishou" > /mnt/chk/hey.txt'
ash: can't create /mnt/chk/hey.txt: Permission denied
```

**図 1-2 ホスト側で `chk` のグループ(今回 gid は一致している)に `w` の権限を追加**

```shell-session
$ chmod g+w chk

$ docker run -u guest -v "${PWD}:/mnt" --rm alpine:latest ash -c 'echo "hey, taishou" > /mnt/chk/hey.txt'

$ cat chk/hey.txt
hey, taishou
```

これについての対応方法は検索するといくつか出てきて、上記の他に「コンテナ作成時にホスト側のユーザーにあわせて動的にユーザーを作成する」などの方法があります。

しかし、それらも「ホスト側で操作しているユーザーの uid/gid がすでにコンテナ側でも使われている」場合には「その id をファイルの読み書き用に使って問題ないのか？」などの問題が発生します。

たとえば、[DevContaienr ではデフォルトユーザーの uid/gid をホスト側と合致させる仕組み](https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user)が導入されていますが、これも gid のコンフリクトから逃れることはできていません。

@[card](https://github.com/microsoft/vscode-remote-release/issues/2402)

ちなみに `ubuntu-22.04` runner では `docker` グループの id が `122` から `123` に変更されたので、 `alpine:latest` イメージではコンフリクトしています[^test-repo]。よって、`alpine:latest` ベースのコンテナを [devcontainer-ci](https://github.com/devcontainers/ci) などで利用していると上記機能が動作しなくなります。

[^test-repo]: Ubuntu runner としては [gid のコンフリクトは(破壊的な)変更とみていぽかった](https://github.com/actions/runner-images/issues/6631)ので、変更点は[テストリポジトリ](unsafe\:hankei6km/test-id-in-ubuntu-runner)を作って調べました。

**図 1-3 `alpine:latest` では gid の `123` は利用されている**

```shell-session
$ docker run --rm alpine:latest grep 122 /etc/group
$ echo "${?}"
1

$ docker run --rm alpine:latest grep 123 /etc/group
ntp:x:123:
```

## ユーザー名前空間でコンテナを分離とは

この流れで書くと「バインドマウント用に uid/gid をなんかよい感じにしてくれるのかな」となりそうですが、**そうではありません**。

uid/gid をよい感じにしてくれますが、どちらかというと「コンテナ内で `root` を使っているとホスト側でも `root` として振舞えてしまう」状態の回避が主な目的のようです。

@[card](https://docs.docker.jp/engine/security/userns-remap.html)

> コンテナ内からの権限昇格攻撃（privilege-escalation attack：一般的に、一般ユーザ権限で root に準じる権限を得られるようにする攻撃のこと）を防ぐベストな方法は、特権のないユーザ（unprivileged user）としてコンテナのアプリケーションを実行するよう設定することです。

**図 2-1 `root` として読み書きできてしまう例**

```shell-session
$ ls /etc/docker/
ls: ディレクトリ '/etc/docker/' を開くことが出来ません: 許可がありません

$ docker run -u guest -v "/etc/docker:/mnt" --rm alpine:latest ash -c 'echo "hey, taishou" > /mnt/hey.txt'
ash: can't create /mnt/hey.txt: Permission denied

$ docker run -v "/etc/docker:/mnt" --rm alpine:latest ash -c 'echo "hey, taishou" > /mnt/hey.txt'

$ sudo cat /etc/docker/hey.txt
hey, taishou
```

上記の挙動に対して、ユーザー名前空間を使うと「コンテナ内での uid は `0`」だけど「ホスト側では異なる uid(ホスト側で使われていない uid)が割り当てられる」ようにできます。

*   [ユーザ名前空間でコンテナを分離 / ユーザとグループ ID の再割り当てとサブオーディネイト](https://docs.docker.jp/engine/security/userns-remap.html#about-remapping-and-subordinate-user-and-group-ids)

> もしもプロセスが名前空間の外に権限を昇格させようとしても、ホスト上におけるこのプロセスは、権限を持たない遙かに大きな UID として実行しています。つまり、ホスト上の実際のユーザとしては動作していないのです。つまり、このプロセスはホスト・システム上で全く権限を持ちません。

## バインドマウントの問題点とユーザー名前空間

バインドマウントの問題点としては「uid/gid の違いによる権限のエラー」と「uid/gid のコンフリクト」がありました。

一方でユーザー名前空間は権限を抑制すためのものですが、別の見方をすると「ホストとコンテナに存在していない新しい uid/gid が割り当てられる」のでコンフリクトを回避しやすいという利点もあります。

また、ユーザー名前空間で割り当てられた id は「ホスト側では権限のないユーザー」という扱いですが、権限の調整はできます(`chown` で id を番号で指定するなど)。

> 3.  Docker ホスト上のどこかに対し、権限のないユーザが書き込む必要がある場合は、適切な場所に対する権限（パーミッション）を調整する必要があります。これは Docker によって自動的に作成される **`dockremap`** を使う場合でも同様ですが、設定を変更し、 Docker の再起動をした後でないと権限を変更できません。

よって、バインドマウントの問題点はユーザー名前空間を使うと(少し追加設定が必要ですが)回避できるかと思われます。

:::message
この辺の権限関連では(おそらく)ID mapped mount のサポートが必要と思われますが、Ubuntu runner では利用できています。(Storage Driver が `overlayfs2` )

@[card](https://techfeed.io/entries/637487b126abe52d164e7851)
:::

## Ubuntu runner の Docker でユーザー名前空間は使えるのか？

バインドマウントの問題はユーザー名前空間で回避できそうな感じです。では、Ubuntu runner の Docker 環境で利用できるのかというと…。

ちょっと検索したかぎりではドキュメントとして「使える or 使えない」という情報はなさそうでした。そこで簡単なワークフローを作って調べてみたところ「サブオーディネイト UID と GID は設定されている」が「機能は有効化されてない」状態でした。

@[card](https://github.com/hankei6km/test-userns-remap-gha)

**リスト 4-1 関連情報を表示するステップ**

```yaml
      - name: snif
        run: |
          id
          echo ---
          ls -l /etc/subuid /etc/subgid
          echo ---
          cat /etc/subuid
          echo ---
          cat /etc/subgid
          echo ---
          sudo ls -l /etc/docker/
          sudo cat /etc/docker/daemon.json
          echo ---
          docker info
          echo ---
          docker image ls
          echo ---
          sudo systemctl status docker
        shell: bash
```

<!-- textlint-disable -->

:::details 実行結果の表示

<!-- textlint-enable -->

    uid=1001(runner) gid=123(docker) groups=123(docker),4(adm),101(systemd-journal)
    ---
    -rw-r--r-- 1 root root 45 Dec  1 11:22 /etc/subgid
    -rw-r--r-- 1 root root 45 Dec  1 11:22 /etc/subuid
    ---
    runneradmin:100000:65536
    runner:165536:65536
    ---
    runneradmin:100000:65536
    runner:165536:65536
    ---
    total 8
    -rw-r--r-- 1 root root  83 Dec  1 11:24 daemon.json
    -rw------- 1 root root 244 Nov 27 21:13 key.json
    { "exec-opts": ["native.cgroupdriver=cgroupfs"], "cgroup-parent": "/actions_job" }
    ---
    Client:
     Context:    default
     Debug Mode: false
     Plugins:
      buildx: Docker Buildx (Docker Inc., 0.9.1+azure-2)
      compose: Docker Compose (Docker Inc., 2.12.2+azure-1)

    Server:
     Containers: 0
      Running: 0
      Paused: 0
      Stopped: 0
     Images: 20
     Server Version: 20.10.21+azure-1
     Storage Driver: overlay2
    debian           11                                 c31f65dd4cc9   3 weeks ago     124MB
    node             14-alpine                          87112681acad   3 weeks ago     121MB
    node             16-alpine                          c4ee3c9d7bc1   3 weeks ago     116MB
    node             18-alpine                          0fa08f92e64b   3 weeks ago     167MB
    alpine           3.16                               bfe296a52501   3 weeks ago     5.54MB
    moby/buildkit    latest                             383075513bdc   3 weeks ago     142MB
    ubuntu           22.04                              a8780b506fa4   4 weeks ago     77.8MB
    ubuntu           20.04                              680e5dfb52c7   6 weeks ago     72.8MB
    ubuntu           18.04                              71eaf13299f4   6 weeks ago     63.1MB
    alpine           3.14                               dd53f409bf0b   3 months ago    5.6MB
    alpine           3.15                               c4fc93816858   3 months ago    5.58MB
    alpine           3.10                               e7b300aee9f9   20 months ago   5.58MB
    ---
    ● docker.service - Docker Application Container Engine
         Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
         Active: active (running) since Tue 2022-12-06 06:22:24 UTC; 2min 0s ago
    TriggeredBy: ● docker.socket
           Docs: https://docs.docker.com
       Main PID: 884 (dockerd)
          Tasks: 10
         Memory: 115.4M
            CPU: 1.018s
         CGroup: /system.slice/docker.service
                 └─884 /usr/bin/dockerd -H fd:// --containerd /var/run/containerd/containerd.sock

    Dec 06 06:22:23 fv-az254-348 dockerd[884]: time="2022-12-06T06:22:23.281340587Z" level=info msg="[graphdriver] using prior storage driver: overlay2"
    Dec 06 06:22:23 fv-az254-348 dockerd[884]: time="2022-12-06T06:22:23.531484705Z" level=info msg="Loading containers: start."
    Dec 06 06:22:23 fv-az254-348 dockerd[884]: time="2022-12-06T06:22:23.980471445Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
    Dec 06 06:22:24 fv-az254-348 dockerd[884]: time="2022-12-06T06:22:24.096312865Z" level=info msg="Loading containers: done."
    Dec 06 06:22:24 fv-az254-348 dockerd[884]: time="2022-12-06T06:22:24.251196762Z" level=warning msg="Not using native diff for overlay2, this may cause degraded performance for building images: kernel has CONFIG_OVERLAY_FS_REDIRECT_DIR enabled" storage-driver=overlay2
    Dec 06 06:22:24 fv-az254-348 dockerd[884]: time="2022-12-06T06:22:24.251684467Z" level=info msg="Docker daemon" commit=3056208812eb5e792fa99736c9167d1e10f4ab49 graphdriver(s)=overlay2 version=20.10.21+azure-1
    Dec 06 06:22:24 fv-az254-348 dockerd[884]: time="2022-12-06T06:22:24.252345573Z" level=info msg="Daemon has completed initialization"
    Dec 06 06:22:24 fv-az254-348 systemd[1]: Started Docker Application Container Engine.
    Dec 06 06:22:24 fv-az254-348 dockerd[884]: time="2022-12-06T06:22:24.303794070Z" level=info msg="API listen on /run/docker.sock"
    Dec 06 06:24:22 fv-az254-348 dockerd[884]: time="2022-12-06T06:24:22.145353585Z" level=info msg="Layer sha256:d3e084821e53ef189b21c70b25fbbfdf9fc2c5850e0d3aae71cc219fd4a40c93 cleaned up"

:::

## ユーザー名前空間を有効化するときの注意点

runner 上ではユーザー名前空間が有効化されていないので、何らかの方法で有効化することになります。有効化の方法は後述しますが、その前に少し注意点など。

先に挙げたようにホスト側から見るとコンテナ側の uid/gid が変更されるので、それに関連した副作用(権限の問題)が出てきます。

> この再割り当てはコンテナに対して透過的です。しかし、コンテナが Docker ホスト上のリソースに対してアクセスを必要とするような場合は、状況によっては導入がいささか複雑になります。たとえばホスト上のファイルシステムの領域にバインド・マウントする方法では、システム・ユーザは書き込みができません。セキュリティの観点からは、これらの状況を避けるのがベストでしょう。

また、ホスト側では各種オブジェクトの管理方法(保存ディレクトリー)が変わるので、イメージのキャッシュなどが利用できなくなります。

<!-- textlint-disable -->

> これらの手順に従い、 **`userns-remap`** を無効化したら、有効化後に作成したリソースには一切できなくなります。（訳者注：userne-remap を有効化時、無効化時、 /var/lib/docker/ 以下の異なるディレクトリに Docker オブジェクトを保存します。そのため、有効化する前にあったコンテナやイメージはは有効化によって見えなくなりますし、無効化によっても有効化時のコンテナやイメージが見えなくなります）

<!-- textlint-enable -->

その他、いくつか互換性に制限があるようです。

*   [ユーザ名前空間における既知の制限](https://docs.docker.jp/engine/security/userns-remap.html#user-namespace-known-limitations)

今回はジョブを動かすための隔離された環境なので影響は少なそうに思えますが、実際には [Docker container action](https://docs.github.com/ja/actions/creating-actions/creating-a-docker-container-action) が動かないなどの問題が出てきます[^image-cache-for-action]。よって「必要なときだけ有効化する」などの対策を考えておいた方がよさそうです。

[^image-cache-for-action]: Docker container action を動かすには runner がキャッシュしているイメージが必要なのですが、ユーザー名前空間を有効化すると既存のキャッシュが使えないので動かなくなります。また、キャッシュを移動してもやはり権限関連で動かないと思われます(Docker 用のソケットをマウントしているのでホスト側の `root` 権限が必要そう)。

## Ubuntu runner の Docker でユーザー名前空間を有効化/無効化

注意点も確認したので実際に有効化と無効化を試してみます。

runner 上の Docker 環境では前述のようにある程度の設定はされているので、あとは下記の手順で有効化すればよさそうです。

*   [ユーザ名前空間でコンテナを分離 / デーモン上で userns-remap の有効化](https://docs.docker.jp/engine/security/userns-remap.html#userns-remap)

> **`dockerd`** の開始時に **`--userns-remap`** フラグを有効化するか、以下の手順にある、デーモンが使う設定ファイル **`daemon.json`** の設定を変更できます。

今回は `/etc/docker/daemon.json` を書き換えて `docker.service` を再始動してみます。

まずは有効化してみます。

**リスト 6-1 `"userns-remap":"runner:docker"` を指定し `docker.service` を再始動するステップ**

```yaml
      - name: enable
        run: |
          SW_JSON=""
          SW_JSON="$(sudo cat /etc/docker/daemon.json | jq '.+{"userns-remap":"runner:docker"}')"
          echo "${SW_JSON}" | sudo bash -c 'cat -- > /etc/docker/daemon.json'
          sudo cat /etc/docker/daemon.json
          sudo systemctl restart docker || sudo journalctl -xeu docker.service
          echo ---
          docker info
          echo ---
          docker image ls
          echo ---
          sudo systemctl status docker
        shell: bash
```

**図 6-1 実行すると Security Options に userns が追加される(長いので省略しています)**

    <snip...>

     Security Options:
      apparmor
      seccomp
       Profile: default
      userns
      cgroupns

    <snip...>

(もう少し苦労するかと予想していたのですが)有効化できました。無効化は逆に定義を戻して再始動すれば実施できます。

**リスト 6-2 `"userns-remap":"runner:docker"` を削除し `docker.service` を再始動するステップ**

```yaml
      - name: disable
        run: |
          SW_JSON=""
          SW_JSON="$(sudo cat /etc/docker/daemon.json | jq 'del(."'"userns-remap"'")')"
          echo "${SW_JSON}" | sudo bash -c 'cat -- > /etc/docker/daemon.json'
          sudo cat /etc/docker/daemon.json
          sudo systemctl restart docker || sudo journalctl -xeu docker.service
          echo ---
          docker info
          echo ---
          docker image ls
          echo ---
          sudo systemctl status docker
        shell: bash
```

**図 6-2 実行すると Security Options から userns が外れる(長いので省略しています)**

    <snip...>

     Security Options:
      apparmor
      seccomp
       Profile: default
      cgroupns

    <snip...>

なお、有効化と無効化を実施したあとに Docker container action を動かしてみましたが、想定した通りに動作することも確認してあります。

:::message
ユーザー名前空間を有効化すると `/var/lib/docker` に名前空間化ディレクトリーが作成されています。無効化しても削除されないので、気になる場合は手動で削除することになります(今回は runner が終了すれば消えるのでそのままにしています)。
:::

## 分離されたコンテナから runner (ホスト側)へファイルを書き込む

分離されたコンテナからホスト側へファイルを書き込む場合、書き出し先に権限を設定する必要があります。このときホスト側から見える(割り当てられる) uid/gid の番号が必要になります。

割り当ては下記のような法則になります(今回の設定では `165536` から割り当てられる)。

*   [ユーザ名前空間でコンテナを分離 / ユーザとグループ ID の再割り当てとサブオーディネイト](https://docs.docker.jp/engine/security/userns-remap.html#about-remapping-and-subordinate-user-and-group-ids)

> UID **`231072`** は名前空間内（この例では、コンテナ内のことです）では UID が **`0`** （ **`root`** ）として割り当てられます。 **`231073`** は UID **`1`** として割り当てられ、以降も同様です。

これにあわせて「コンテナの `guest(405:100)`ユーザー用にホスト側ディレクトリーのオーナーを変更」してから書き出してみます。

**リスト 7-1 コンテナを実行し runner 側へファイルを書き出すステップ**

```yaml
      - name: run
        run: |
          mkdir chk
          # 165941 = 165536 + 405
          # 165636 = 165536 + 100
          # 405(guest)
          # 100(users)
          sudo chown "165941:165636" chk
          docker run -u guest -v "${PWD}:/mnt" --rm alpine:latest \
            ash -c 'id && echo "hey taishou" > /mnt/chk/hey.txt'
          ls -l chk
          cat chk/hey.txt
        shell: bash
```

**図 7-1 書き出した結果**

    uid=405(guest) gid=100(users) groups=100(users)
    total 12
    drwxr-xr-x 2 165941 165636 4096 Dec  4 11:26 .
    drwxr-xr-x 5 runner docker 4096 Dec  4 11:26 ..
    -rw-r--r-- 1 165941 165636   12 Dec  4 11:26 hey.txt
    hey taishou

コンテナから `guest(405:100)` ユーザーでファイルを書き出すと `165941:165636` が操作してる状態になることが確認できました。

この例では `405:100` のユーザーで試したので id の計算が面倒ですが、「コンテナの中では `root` (`0:0`) でもいいかな」という場合はもう少し手軽に使えるかと思います[^map-to-user]。

[^map-to-user]: 実はコンテナの `root` をホスト側の `1000` などにマッピングする方法もあるようですが「ホスト側では権限がないユーザーにマッピングさせる」で考えるとあまりよくはないのかなと。

    *   [linux namespaces - Files owned by Docker userns-remap user end up owned by nobody inside the container - Stack Overflow](https://stackoverflow.com/questions/67008342/files-owned-by-docker-userns-remap-user-end-up-owned-by-nobody-inside-the-contai)
    *   [Jujens' blog – Use Linux user namespaces to fix permissions in docker volumes](https://www.jujens.eu/posts/en/2017/Jul/02/docker-userns-remap/)

## 考慮点

### ユーザー名前空間をこのような目的で有効化していもよいのか？

ユーザー名前空間は権限昇格に対応するのが主な目的と思われるので、別の目的で有効化していると本来の用途で利用したいときにやりにくくなるかもしれません。

### runner 更新時の互換性

runner 上のユーザー名前空間利用については、とくにドキュメント化されたものではないので runner 更新時に動作しなくなる可能性はあります。また、今回は `/etc/subuid` などは既に設定されている値を流用していますが、これが変更される可能性もあります。

実際に利用する場合は、runner はイメージのタグでピン止めしておくか、`ubuntu-latest` を使うときはできるだけ影響を受けないように考慮しておく方がよさそうです。

@[card](https://zenn.dev/snowcait/articles/73e42154ed4654)

> GitHub としては自動アップデートされる `ubuntu-latest` が推奨のようですが対応時期を GitHub に決められてしまうので仕事で使用している場合などいきなり動かなくなると困る場合はバージョンを固定しておくのが推奨です

## おわりに

Ubuntu runer の Docker でユーザー名前空間を利用し uid/gid のコンフリクトを軽減させつつ、コンテナからファイルを書き出してみました。

機能をピンポイントで有効化させる必要があるなど面倒なところもありますが、利用できることは確認できたので機会があれば使っていこうかと思います。
