---
id: vscode-variation
title: いろいろな VS Code の一覧
emoji: ⌨️
type: idea
topics:
  - vscode
published: true
---

VS Code にも種類が結構あると思ったのでメモ。

## VS Code のソースコード(Code - OSS)

https://github.com/microsoft/vscode

VS Code とよばれるエディター(IDE)のソースコードの基本部分。

このソースコードをそのままビルドしても [Visual Studio Code(VS Code)](https://azure.microsoft.com/ja-jp/products/visual-studio-code/)にはならない。上記ソースコードには Marketplace へのアクセス機能などが存在しない。ビルドしたものは OSS 版(Code -OSS)のように表現されることもある。

詳しくは Arch Wiki などを参照。

https://wiki.archlinux.jp/index.php/Visual_Studio_Code

基本的にソースコードとして配布されるが、[Arch Linux] や [Alpine Linux] ではビルドしたバイナリーに Open VSX Registry の設定を含めたパッケージが登録されている。

::: message

**Open VSX Registry**

いきなり「Open VSX Registry」がでてきたので少し補足。

@[card](https://www.eclipse.org/community/eclipse_newsletter/2020/march/1.php)

:::

## デスクトップ用

デスクトップ OS 用エディター(IDE)としての VS Code。

### Microsoft Visual Studio Code(Code)

https://code.visualstudio.com/
https://azure.microsoft.com/ja-jp/products/visual-studio-code/

いわゆる VS Code。とくに注釈なく「Visual Studio Code」「VS Code」「VSCode」「VS-Code」などと表記されていたら、多くの場合でこれを指している。

前述のソースコードに Microsoft 固有の機能が追加されたもの。

*   利用には Microsoft のライセンスに従う必要がある

    *   その代わりに Marketplace や設定の同期などを利用可能になる
    *   テレメトリー関連がデフォルトで有効となっている

とりあえず「VS Code を試してみたい」場合は上記ページからインストーラーをダウンロードしインストールするのが無難。 (ライセンスなどに同意できるならという前提)

::: message

インストールにパッケージマネージャーを利用する方法もある。たとえば、Ubuntu だと Snap 版が利用できる。

@[card](https://code.visualstudio.com/docs/setup/linux#_snap)

しかし、[Ubuntu では Debian パッケージ版を使うことをお薦めされている](https://gihyo.jp/admin/serial/01/ubuntu-recipe/0720)。

どこを重視するかにもよると思われるので、いろいろ試してみるのがよいかと(個人的には更新のしやすさを重視して Snap 版が楽に感じている)。

:::

### Insiders 版 VS Code(Code - Insiders)

https://code.visualstudio.com/insiders/

nightly ビルド版といった位置づけ。アイコンが緑色。上記の安定版と同時にインストールできる。

設定の同期は利用するサービスを安定版と別に分けておくか選択する。別にすることで安定版とは異なる設定を保存できる。

**図 2-1 同期サービスの選択**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/226cf40cfc2042408ef425729dd4ded0/vscode-variation-setting-sync.png?auto=compress%2Cformat)

「Get the latest release each day」ということなのでそこそこの頻度で更新される(自動更新できない環境だとちょっと大変)。

Pre Release 版しかない拡張機能を UI からインストールする場合は Insiders 版が必要(コマンドラインからなら安定にもインストールできる)。

なお、コマンド名や設定を保存するディレクトリー名などには `-insiders` が追加される。

### VSCodium

https://vscodium.com/
https://github.com/VSCodium/vscodium

使ったことはないので実際の使用感などは不明。

Code - OSS をバイナリーとして配布しているもので、テレメトリーなどが無効化されているらしい。こちらも拡張機能の Marketplace として Open VSX Registry 設定が含まれる。

## ブラウザー用

Google Chrome や Microsoft Edge などのブラウザー(含むモバイルデバイス版)で利用できるエディター(IDE)としての VS Code。

### Web 版 VS Code

https://vscode.dev/
https://insiders.vscode.dev/

Web ブラウザー上で動作する VS Code。ローカル側ではとくに環境設定などを行う必要はなく、上記ページにアクセスすると利用できる(下は Insiders 版)。

ローカルのフォルダーを開いてのファイル編集が可能。デスクトップ版と設定の同期(共有)も可能。さらには GitHub などのリモートリポジトリを開いてコミットなども行える。

OS の機能(プロセス)などは利用できないが、Web 版向けに開発されている拡張機能は利用可能で、ローカルでのビルド環境を必要としないような用途では活躍する。

ただし、GitHub Copilot を利用できないので、個人的には少し利用頻度が下がってきた。 (後述の組み合わせによってはよっては利用できる)

なお、セルフホストすることは(おそらく)できないが、下記での `serve web` でデスクトップ版をブラウザーから扱うことは可能。ただし、Web 版とは挙動が異なる。

### Visual Studio Code Server(serve web)

デスクトップ版 VS Code をブラウザーから利用するモード。サーバーとして稼働させるために同意するライセンスが増える。

デスクトップ版 VS Code がインストールされていればコマンドラインから `code serve-web` で開始できる(Insiders 版だと `code-insiders serve-web`)。

見た目的には前述の Web 版と似ているが、サーバーとして利用している環境のリソースを利用する。たとえば、フォルダーを開くとサーバー側のファイルシステムにアクセスし、ターミナルを開くとサーバー上でシェルが動く。

**図 3-1 Web 版と同様の画面だが、サーバー側のプロセスなどが利用できる**

![ブラウザーで serve web サーバーへアクセスし、ターミナルなどを利用しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/43964b5a9c73458f991703b1f28847af/vscode-variation-serve-web.png?w=1440\&h=873\&auto=compress%2Cformat)

なお、設定などは大元のデスクトップ版とは隔離されている(設定の同期はできる)。

デスクトップ用の拡張機能を利用できるが、対応していないものもある。 (例、GitHub Copilot は利用できるが、Remote SSH は利用できない)

## サーバー用

VS Code の機能を提供するバックエンドとしての VS Code。

### vscode-server

リモート開発で VS Code を利用するとき、サーバー側へエージェント的に自動インストールされる実行環境(Remote Extension Host などが動作する)。

通常あまり意識することはない。

### Visual Studio Code Server(tunnel)

https://code.visualstudio.com/docs/remote/vscode-server

デスクトップ版 VS Code を Remote Tunnels のサーバーとして利用するモード。サーバーとして稼働させるために同意するライセンスが増える。

デスクトップ版 VS Code がインストールされていれば UI から有効化できる。またコマンドラインから `code tunnel` でも開始できる(Insiders 版の場合は `code-insiders tunnel` )

Web 版 VS Code などから外部ネットワーク経由でサーバーへ接続できるようなにる(Remote Tunnels については後述)。

### CLI 版 Visual Studio Code Server

主にヘッドレスサーバーで利用するための Visual Studio Code Server( `tunnel` と `serve web` どちらも利用可能)。

実行ファイルを 1 つダウンロードして起動するだけで利用可能。

少しややこしいのだが、CLI 版 Visual Studio Code Server を使っていても利用時には別途 vscode-server がインストールされる。

## リモート開発用

[リモート開発](https://github.com/Microsoft/vscode-remote-release)で使われる拡張機能とそのクライアント(UI)としての VS Code。

### Remote SSH

https://code.visualstudio.com/docs/remote/ssh

`~/.ssh/config` などを利用して SSH サーバーへ接続し、サーバー側のファイルシステムやプロセスを利用する形態。 (サーバーへ接続するときに自動で vscode-server がインストールされる)

ローカル PC には Open SSH クライアントなどが必要。

SSH に限らず後述のリモート開発系の拡張を使う場合、おおきく以下のような使い方になる。

*   サーバーへ接続することでどこからでも同じ環境を利用できる
*   ローカルの環境では動かせない重いプロセスをハイスペックな環境を利用して動かす

この他、特殊なセンサーなどを装着した Raspberry Pi へ接続して開発するなどにも利用できる。

なお、Remote SSH ではクライント側に Web 版 VS Code を利用できない。

### Dev Containers

https://code.visualstudio.com/docs/devcontainers/containers

Docker コンテナを作成し。コンテナ内のファイルシステムやプロセスを利用する形態。 (コンテナへ接続するときに自動で vscode-server がインストールされる)

ローカル PC には Docker 環境などが必要。

コンテナはワークスペース(リポジトリ)内に `devcontainer.json` を配置することで定義できる。よって、プロジェクトに特化した環境を作成し組織内で共有しやすいという特徴がある。

以前は「Remote」が付いた名称だったが現在は Dev Container を主とした名称となっている。

*   リモートの Docker へ接続することが必須と間違えやすい
*   コンテナを定義する `devcontainer.json` は VS Code 以外でも利用できそう

コンテナ作成には Docker 以外(Podman など)も利用できるがすべての機能は利用できない。

なお、Dev Containers ではクライント側に Web 版 VS Code を利用できない。

### Remote SSH + Dev Containers

SSH 接続した後に `devcontaier.json` を含んだフォルダーを開くことで Dev Container を作成し利用できるようになる。

Docker は SSH 接続した先のサーバーにインストールされていればよい。

ローカル PC でコンテナを作成できないようなときに利用できる。

### WSL

https://code.visualstudio.com/docs/remote/wsl

利用したことはないので詳細は不明。

### Remote Tunnels

https://code.visualstudio.com/docs/remote/tunnels

Tunnel サーバー(Tunnel モード)として稼働している VS Code へ接続するリモート開発の形態。

使用感的には Remote SSH に近いが、VPN などを構築しなくとも外部ネットワークから接続可能となる。これは、Tunnel が Microsoft(Azure)のサーバーにより中継されるため。また、Tunnel サーバーは各種環境へインストールしやすい(Android の [UserLAnd](https://play.google.com/store/apps/details?id=tech.ula\&hl=ja\&gl=US) でも動作した)。

よって、Remote SSH と比較すると環境構築のハードルは低い。一方で SSH 接続と同様にサーバーを利用できてしまうので運用には注意が必要。

なお、リモート系では珍しくクライント側に Web 版 VS Code を利用できる。

しかし、現時点では Dev Containers を組み合わせることはできないので、 Remote SSH + Dev Containers のようなことはできない。

Tunnel サーバーをコンテナで隔離したい場合は、コンテナの中から CLI 版の VS Code Sever を利用することになる。

## クラウド開発環境(CDE)用

クラウドサービスのウェブ IDE やクライアントとして利用される VS Code。

### github.dev

https://docs.github.com/ja/codespaces/the-githubdev-web-based-editor

Web ブラウザーで GitHub のリポジトリーを表示しているとき、キーボードから `.` (ドット) を入力すると開始されるエディター。

ドキュメントによると VS Code の利点が提供されているとのこと(普通に使っているぶんには Web 版 VS Code とほぼ同じ)。設定の同期も可能。GitHub 用の機能が用意されているので(認証などの操作は必要)、リポジトリを編集するとき気軽に利用できる。

また、後述する Codespaces の UI として利用も可能。

Insiders 版の利用はアクティビティバー下の設定(歯車)アイコンから切り替える。

### Codespaces

https://docs.github.com/ja/codespaces

GitHub 上で作成される VM を利用する開発環境。通常はリポジトリに Dev Container を定義しておき、github.dev や VS Code などから Dev Container へ接続することで利用する。

Web ブラウザーで GitHub のリポジトリーを表示しているとき、キーボードから `,` (カンマ) を入力するなどで開始される(他にも開始する方法はいくつかある)。

VS Code から利用するときは、Codespaces 用の拡張機能をインストールする(Dev Containers 拡張機能ではない)。ローカル PC に Docker などの機能を必要としない。

https://marketplace.visualstudio.com/items?itemName=GitHub.codespaces

使用感としては Remote Tunnels と Dev Containers のいいとこ取りに近い。ローカルの環境に左右されることは少ない。また、GitHub 固有の機能や設定がそろっているのでその点で困ることもあまりない。

*   GitHub に[シークレットなどを設定できる](https://docs.github.com/ja/codespaces/managing-your-codespaces/managing-your-account-specific-secrets-for-github-codespaces)
*   GitHub CLI が認証済で利用可能になっている

端末一台渡り鳥的に利用できるので使いやすい。ついつい頼ってしまう。が、しかし従量課金(無料枠あり)なので使いすぎには注意。

### 他にもいろいろある

上記(Microsoft 系列のサービス)以外にも、VS Code へ専用の拡張機能インストールすることで利用できるサービスはいろいろある。 (Remote Tunnels などを利用すれば対応サービスはさらに増えると思われる)

が、数が多いので、よく利用している [CodeSandbox] について一言だけ書くと「[クセ強よだが](https://codesandbox.io/docs/learn/environment/devcontainers#limitations) Dev Container を利用できるので Codespaces と環境を共有しやすい」と言える。

https://codesandbox.io/docs/learn/editors/vscode/overview

## テスト用

おもに拡張機能の開発などで UI テスト(E2E テスト)で利用される VS Code。

### @vscode/test-electron

https://github.com/microsoft/vscode-test

正確にはテスティングフレームで、テスト時に実際の VS Code を別途ダウンロードする(インストール済 VS Code も利用できるらしいが、CI などを考えるとダウンロードが無難)。

ダウンロードされる VS Code は Microsoft Visual Studio Code に限定される。よって、VS Code のライセンスへ同意が必要。その代わりに、Marketplace も使えるので、依存する拡張機能なども簡単にインストールできる。

https://github.com/microsoft/vscode-test/blob/62881f0aba43c97734dd540b304569e45269dd18/lib/download.ts#L35-L36

なお、リモート開発や GitHub Actions など CI で実行するときはヘッドレスで利用することになるが、その場合は [Xvfb](https://www.x.org/releases/X11R7.6/doc/man/man1/Xvfb.1.xhtml) などを用意することになる。

### @vscode/test-web

https://github.com/microsoft/vscode-test-web

上記の Web 版。

こちらは UI(ブラウザー)の制御に Playwright を使っているらしい。

## おわりに

いろいろな VS Code を一覧にしてみた。

予想していたよりも分量があったけれども、「こういうコマンドもあったんだ」と再発見もあったので時々確認するのもよいのかもしれない。

[Arch Linux]: https://archlinux.org/

[Alpine Linux]: https://alpinelinux.org/

[CodeSandbox]: https://codesandbox.io/

[Gitpod]: https://www.gitpod.io/customers

[Cloud Code]: https://cloud.google.com/code/docs/vscode/overview?hl=ja
