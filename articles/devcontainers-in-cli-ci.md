---
id: devcontainers-in-cli-ci
title: Dev Container が VSCode から CLI にもやって来た
emoji: 📦
type: tech
topics:
  - devcontainer
  - cli
  - githubactions
  - vscode
  - codespaces
published: true
---

VSCode の更新情報を見ていたら Dev Container の仕様と、リファレンス実装の CLI ツールについて記載がありました。

> ## Development Container specification
>
> Our development container teams across Microsoft and GitHub continue active development on the new [Dev Container Specification](https://github.com/devcontainers/spec), and this iteration had several exciting highlights.

@[card](https://code.visualstudio.com/updates/v1_68#_development-container-specification)
@[card](https://code.visualstudio.com/blogs/2022/05/18/dev-container-cli)

## どういう風にやってきたのか

Dev Container(devcontainer) の元は VSCode の Remote - Containers で使われている開発コンテナ(とその設定ファイル)のことです。これを VSCode から[独立させた仕様](https://github.com/devcontainers/spec)とし、それを元に実装されたものが Dev Container の CLI ツール([@devcontainers/cli])となります。

@[card](https://github.com/devcontainers/spec)
@[card](https://github.com/devcontainers/cli)

## 試してみる

### 基本操作

[@devcontainers/cli] の README にあるサンプルを試してみます。

リポジトリを clone して `devcontainer up` すると devcontainer が開始されます。なお、現状では[ワークスペース(リポジトリ)をカレントディレクトリ－にしていても ](https://github.com/devcontainers/cli/issues/29)[`.devcontainer`](https://github.com/devcontainers/cli/issues/29)[ を認識しない](https://github.com/devcontainers/cli/issues/29)ので `--workspace-folder` は必須となります。

▼ **図 2-1 clone して decvontaienr を開始**

```shell-session
$ git clone https://github.com/microsoft/vscode-remote-try-rust
$ cd vscode-remote-try-rust/
$ devcontainer up --workspace-folder .
```

devcontainer を開始するとデフォルトでは `.devcontainer` が存在するディレクトリ(通常はリポジトリ)がワークスペースとしてマウントされています。また、`devcontainer exec` ではワークスペースがカレントディレクトリーになっています。

▼ **図 2-2 コンテナと外と内で ls コマンドを実行してみる**

![コンテナの外と内で ls コマンドを実行し、それぞれ同じファイルが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8e5f486892cc4943ab2748fe5c68aa66/devcontainers-in-cli-ci-up-mounted.png?w=999\&h=694\&auto=compress%2Cformat)

これにより「コンテナの外側で Vim によりコードを編集」「`devcontainer exec` を使ってコンテナ内でビルドと実行」などができます。

▼ **図 2-3 Vim で編集しながら exec で実行**

![コンテナの外でファイルを編集しつつ、exec でファイルをビルドし実行しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/15b46b487db84bb2ad477fa783ac2e2d/devcontainers-in-cli-ci-exec.png?w=1023\&h=597\&auto=compress%2Cformat)

devcontainer の停止については(現状では `stop` などが実装されていないので)、`docker stop` などを利用します。

### 引数の展開

`devcontainer exec` では GLOB や環境変数などは展開されませんでした。必要であれば `bash -c` などを使うことになります。

▼ **図 2-4 GLOB の展開**

```shell-session
$ devcontainer exec --workspace-folder vscode-remote-try-rust/ ls src/*.rs
[5 ms] @devcontainers/cli 0.6.0.
ls: cannot access 'src/*.rs': No such file or directory
{"outcome":"error","message":"Command failed: ls src/*.rs","description":"An error occurred running a command in the container."}

$ devcontainer exec --workspace-folder vscode-remote-try-rust/ bash -c 'ls src/*.rs'
[7 ms] @devcontainers/cli 0.6.0.
src/main.rs
{"outcome":"success"}
```

### 環境変数

`devcontainer exec` では `--remote-env NAME=VALUE` を使うことでコンテナ内の環境を変数を一時的に変更できます。

▼ **図 2-5 環境変数を設定**

```shell-session
$ devcontainer exec --workspace-folder . bash -c 'echo "${VAR1}"'
[5 ms] @devcontainers/cli 0.6.0.

{"outcome":"success"}

$ devcontainer exec --workspace-folder . --remote-env VAR1=VALUE1 bash -c 'echo "${VAR1}"'
[5 ms] @devcontainers/cli 0.6.0.
VALUE1
{"outcome":"success"}
```

いまのところ `docker exec` の `--env-file` のようなオプションは無いので、複数変更したい場合は個別に記述する必要があります。

### 出力先

exec で実行されたコマンドの結果は stderr へ出力されました。

▼ **図 2-6 リダイレクトを切り替えての比較**

```shell-session
$ devcontainer exec --workspace-folder . ls 
[5 ms] @devcontainers/cli 0.6.0.
Cargo.lock  Cargo.toml  LICENSE  README.md  src  target
{"outcome":"success"}

$ devcontainer exec --workspace-folder . ls > /dev/null
[5 ms] @devcontainers/cli 0.6.0.
Cargo.lock  Cargo.toml  LICENSE  README.md  src  target

$ devcontainer exec --workspace-folder . ls 2> /dev/null
{"outcome":"success"}
```

### 出力形式

`--log-format json` を指定すると JSONL で出力されます。プログラムから操作するときに、ストリームで扱いやすくしているのかと思われます。

```shell-session
$ devcontainer exec --workspace-folder . --log-format json
{"type":"text","level":3,"timestamp":1655358024631,"text":"@devcontainers/cli 0.6.0."}
{"type":"start","level":2,"timestamp":1655358024631,"text":"Run: docker buildx version"}
{"type":"stop","level":2,"timestamp":1655358024711,"text":"Run: docker buildx version","startTimestamp":1655358024631}
{"type":"start","level":2,"timestamp":1655358024715,"text":"Run: git rev-parse --show-cdup"}
{"type":"stop","level":2,"timestamp":1655358024720,"text":"Run: git rev-parse --show-cdup","startTimestamp":1655358024715}
...
```

### 入力が必要な操作

`devcontainer exec` はプログラムがから実行することも想定しているからか、入力が必要な操作はできませんでした(現状では `-i `的なオプションもないようです)。

▼ **図 2-7 bash を実行しコマンドを入力しても反応なし**

![devcontainer exec で bash を実行し ls コマンドを入力しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8cc4ef1dde244672971e0d842b5d91f9/devcontainers-in-cli-ci-input.png?w=591\&h=118\&auto=compress%2Cformat)

たとえば jest を `--watch` で実行しているときにキー操作したいような場合は `docker exec` を使うことになります。

▼ **図 2-8 docker exec を使う場合の例(作業ディレクトリーなどを明示的に指定)**

```shell-session
$ docker exec -it -u vscode -w /workspaces/notion2hast magical_engelbart npm run test -- --watch
```

###

## Docker Compose

devcontainer では Docker Compose も使えます。これを利用すると「テスト用のモックサーバーが稼働する環境」などの配布もできます。

以下は、[httpbin](https://httpbin.org/) をコンテナとして実行しモックサーバーにするサンプルリポジトリです。

@[card](https://github.com/hankei6km/test-devcontainers-cli-compose)

```json:devcontainer.json
{
  "name": "node",
  "dockerComposeFile": "docker-compose.yml",
  "service": "node",
  "remoteUser": "vscode",
  "workspaceFolder": "/home/vscode/workspace",
  // snip...
}
```

```yml:docker-compose.yml
version: '3.8'

services:
  node:
    # image: 'ghcr.io/hankei6km/h6-dev-containers:node'
    build:
      context: .
      dockerfile: Dockerfile
    hostname: node
    user: vscode
    volumes:
      - ../:/home/vscode/workspace
    tty: true
  httpbin:
    image: 'kennethreitz/httpbin:latest'
    hostname: httpbin
```

サンプルリポジトリを clone し `devcontainer up` を実行するとワークスペース(`node`)とモックサーバー(`httpbin`)が稼働します。

▼ **図 3-1 定義されたコンテナ(サービス)が実行されている**

![コンテナの一覧を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9b93b3fc7dfb4b02b9bb2bddb8c19d61/devcontainers-in-cli-ci-compose-ls.png?w=1008\&h=232\&auto=compress%2Cformat)

この状態で `devcontainer exec` を使ってモックサーバーを使ったテストなどを実行できます。

▼ **図 3-2 コンテナの外からテストを実行**

![devcontainer exec でテストを実行し結果を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/64fb79fcc2ab410380f8fc9cba8e6d16/devcontainers-in-cli-ci-compose-exec.png?w=985\&h=562\&auto=compress%2Cformat)

なお、devcontainer の停止は(現状では `devcontainer` に `stop` と `down` が実装されていないので)、`docker-compose down` を利用する必要があります。

```shell-session
$ docker-compose --project-name test-devcontainers-cli-compose_devcontainer -f .devcontainer/docker-compose.yml down
```

## Dev Container in Dev Container

親となる devcontainer が [Docker in Docker ベース](https://github.com/microsoft/vscode-dev-containers/tree/main/containers/docker-in-docker)の場合、その中から devcontainer を開始できます。

▼ **図 4-1 入子になった devcontainer の情報を表示**

![「devcontainer を実行している環境」「devcontainer」「その中の devcontainer」の情報を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/29d4b7f2f12d454b8c0bddd645b60f60/devcontainers-in-cli-ci-dind.png?w=1012\&h=685\&auto=compress%2Cformat)

devcontainer の中で devcontainer を動かすことの意味についてですが…。

(この後でも少し触れていますが)実は「テストやビルドを行うコンテナへ操作性に関するツールをインストールする」ことに少し違和感を持っています[^permenet-history]。

「最初に開始される devcontainer にだけ各種ツールをインストールし、テストなどは別途開始した devcontainer で行う」ことで緩和できないか少し考えています。

[^permenet-history]: 「[GitHub Codespace で Terminal の入力履歴を永続化し Codepace 間で共有してみる](https://zenn.dev/hankei6km/articles/persist-command-history-in-github-codesapces)」みたいなことをやっているので説得力は皆無ですが。

## CI

これまでは GitHub Actions などでのテストは「テストに必要なランタイムをランナー上にインストールしてから実行」という流れでした。

これが、devcontainer を独立して実行できる状況では「ワークフロー内で開発用コンテナを開始してその中で実行」することが容易になります。さらには、アクション([devcontainers/ci])も開発されています(現時点は docker-compose のサポートは限定的なようです)。

@[card](https://github.com/devcontainers/ci)

以下は、CLI ツールでコンテナを開始しテストしているワークフローです(action では docker-compose がうまくいかなかったので CLI でやっています)。

```yml:test.yml
name: test
on:
  push:
    branches:
      - '**'
    tags:
      - '!v*'
jobs:
  jest:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Use Node.js 14.x to run @devcontainers/cli
        uses: actions/setup-node@v1
        with:
          node-version: 14.x

      - name: Cache node modules
        uses: actions/cache@v2
        env:
          cache-name: cache-node-modules
        with:
          path: ~/.npm
          key: ${{ runner.os }}-build-${{ env.cache-name }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-build-${{ env.cache-name }}-
            ${{ runner.os }}-build-
      - name: Install @devcontainers/cli
        run: npm install -g @devcontainers/cli

      - name: Start devcontainer
        run: devcontainer up --workspace-folder .

      - name: Install modules
        run: devcontainer exec --workspace-folder . npm install

      - name: Run test
        run: devcontainer exec --workspace-folder . npm run test
```

従来のように「ジョブにモックサーバーを起動するステップを追加して…」的なことを考えることなく、開発しているときの環境そのままでテストが実行されています(表示が少し崩れていますが…)。

▼ **図 5-1 ワークフロー内で devcontainer を利用したテストを実行**

![GitHub のウェブ UI でワークスペースの結果を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4b70ab631282488996db5e0761063c5e/devcontainers-in-cli-ci-ci-test.png?w=1020\&h=619\&auto=compress%2Cformat)

## 考慮点

ここまでは便利なところを書いてみましたが、少し考えるところもあります。

### 拡張機能

従来の `devcontainer.json` では VSCode で利用する拡張機能や設定を記述できますが、これが何かに反映されることはなかったです。[現時点での仕様](https://github.com/devcontainers/spec/blob/main/docs/specs/devcontainerjson-reference.md)に `extensions` の記載はなく、[VS Code specific properties](https://code.visualstudio.com/docs/remote/devcontainerjson-reference#_vs-code-specific-properties) として扱わることになるそうです。

@[card](https://zenn.dev/nmemoto/articles/devcontainer-cli)

よって「拡張機能でフォーマッターなどを設定している」場合などで注意が必要です。

```json:devcontainer.json
{
  // 従来の拡張機能関連の定義
  "extensions": ["esbenp.prettier-vscode"],
  "settings": {
    "[typescript]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[json]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[javascript]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[jsonc]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[html]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    }
  }
}
```

### Windows

Windows ホストは[サポートされている感じ](https://github.com/devcontainers/cli/pull/36)ですが、現状で Windows コンテナのサポートは不明です。

普段 Windows で Docker を使ってないので GitHub Actions の `windows-latest` ランナーで試しただけですが、以下のエラーが出て開始できませんでした[^windows]。

    hcsshim::System::CreateProcess: failure in a Windows system call: The system cannot find the file specified.

(私がミスしている可能性もかなりありますが)現時点ではワークフロー内で Windows コンテナを使ってのテストは難しい感じです。

[^windows]: デフォルトの設定ではマウントの dst にドライブ文字が使われない問題もありました。また、致命的なエラーではないですが、gyp でモジュールをビルドしているのかインストールに数分かかるのも悩みどころです。

### コンテナイメージ

#### Terminal

VSCode では Terminal 内でコンテナ(のシェル) が稼働します。このため「Terminal をメインで使う」場合、操作性を向上させるツールをコンテナへインストールします。

しかし、CLI ツールを使った場合は(いまのところ)コンテナの外側からコマンドを実行することになります。イメージを作るとき「コンテナ内でシェルを使うことはあまり考えない」傾向が強くなるかもしれません。

と、考えていたのですが、[@devcontainers/cli] の開発用 [devcontainer](https://github.com/devcontainers/cli/tree/main/.devcontainer) ではあえて「[bash の入力履歴を永続化](https://github.com/devcontainers/cli/pull/32)」しているので、シェルを使う方向に傾く可能性もあります。

#### CI

CI 的な観点からは「イメージの展開にかかるコスト(ビルドの時間、ファイルのサイズなど)」は気になるところです。 `cahce_from` 用のイメージを設定できますが、できれば根本的にサイズを縮小したくなります。`Dockerfile` で CI に関係しないツールをインストールしている場合は、CI 用の `.devcontainer` 作成も検討したが方がよさそうです。

### ランタイムとマトリックス

たとえば「Node.js の 14 と 16 でテストしたい」という場合、GitHub Actions では `actions/setup-node` とマトリックスを利用して切り替えていました。これが利用できなくなるので、`.devcontainer` の build 用引数で variant を切り替えるなどを検討する必要があります(またはどうにかして nvm を使うなど)。

### おわりに

devcontainer を CLI で実行するツールを試してみました。

単体のツールとしてみても「CLI や CI で開発コンテナを手軽に利用できる」というメリットが実感できました。また、単なる CLI ツールとしてだけでなく「[この実装を元に他のエディターや IDE でも開発コンテナを促進していく](https://github.com/devcontainers/cli/issues/7)」という空気も感じています。 (さらにその先の展開「[Orchestrator interop](https://github.com/devcontainers/spec/issues/10)」も視野に入れているようです)

この流れが広まっていけば、エディターや IDE に左右されることなく均一化された開発環境がワンタッチでそろうようになるかもしれませんね。

[@devcontainers/cli]: https://github.com/devcontainers/cli

[devcontainers/ci]: https://github.com/devcontainers/ci
