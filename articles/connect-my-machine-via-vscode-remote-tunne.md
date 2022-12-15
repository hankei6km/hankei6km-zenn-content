---
id: connect-my-machine-via-vscode-remote-tunne
title: VSCode の Remote Tunnels で「いつもの開発環境へ」お手軽リモート接続
emoji: 🔌
type: tech
topics:
  - vscode
published: true
---

VSCode の更新情報を見ていたら [Remote Development](https://code.visualstudio.com/updates/v1_74#_remote-development) の項目に Remote Tunnels についての記述がありました。面白そうなので試してみたところ、vscode.dev(girthub.dev) や Codespaces とはまた違ったリモートな開発環境を簡単に構築できてしまいました。

というわけで今回は Remote Tunnels のインストールや使い方、サーバーの少し突っ込んだ設定方法などについて。

## VSCode の Remote Tunnels とは

簡単に言うと「普段使っている PC を外部のデバイスから VSCode として利用できる機能」となります。

この機能は以前から VSCode Server + トンネル機能としてプライベートプレビューでテストされていましたが[^private-prev]、今回の更新からプレビューフィーチャーとなりました。これによりインストールやトンネル作成が誰でも簡単に利用できるようになっています。

<!-- textlint-disable -->

[^private-prev]: プライベートプレビューでの VSCode Serverは「[VS Code Serverの使い方](https://zenn.dev/kato_k/articles/6301d35b3d8d3c)*」「*[自前サーバでCodespacesが使えるVisual Studio Code Serverを動かしてみた - Qiita](https://qiita.com/ku_suke/items/62f1ba1d21667438f2dc)」が参考になります。

<!-- textlint-enable -->

@[card](https://code.visualstudio.com/blogs/2022/12/07/remote-even-better)

*   [Tunnel to anywhere, from one tool](https://code.visualstudio.com/blogs/2022/12/07/remote-even-better#_tunnel-to-anywhere-from-one-tool)

<!-- textlint-disable -->

> Tunneling securely transmits data from one network to another. You can use secure tunnels to develop against any machine of your choosing from a VS Code desktop or web client, without the hassle of setting up SSH or HTTPS (although you can do that if you want as well 😊).
>
> You have two great options for tunneling to remote machines from VS Code:
>
> *   Use the new, enhanced `code` CLI.
> *   Enable tunneling from the VS Code UI directly.
>
> We'll explore both options in the following sections.

<!-- textlint-enable -->

## 必要な環境

*   トンネル(サーバー)を作成するための PC(My Machine)
*   ブラウザーまたはデスクトップ版 VSCode が動作するデバイス(Client Device)
*   vscode.dev 上でトンネルを利用するための GitHub Account
*   インターネット環境

**図 2-1 環境の概要**

![My Machin が vscode.dev とのトンネルを作成し、Client Device が My Machine へ接続している図](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ab42aeacc593486b94f70cca0f86e865/connect-my-machine-via-vscode-remote-tunnel-intro-current.png?w=811\&h=482\&auto=compress%2Cformat)

## インストールとトンネル作成

トンネルを利用(作成する)方法は主に 2 つあります。

*   単体の CLI 版 `code` をインストールしトンネルを作成する
*   通常のデスクトップ版 VSCode でトンネル機能を有効化する

[冒頭の更新情報](https://code.visualstudio.com/updates/v1_74#_remote-development)に掲載されている動画や下記の記事ではデスクトップ版 VSCode を利用しているので、ここでは CLI 版の `code` を Linux 環境で試してみます。

@[card](https://forest.watch.impress.co.jp/docs/news/1462989.html)

### インストール

とくに難しい作業はなく[ダウンロードページ](https://code.visualstudio.com/Download)から CLI 版アーカイブをダウンロードし、実行ファイルを取りだすだけです。もしくは既にデスクトップ版 VSCode を使っているならそちらの `code` コマンドを使うこともできます(ただし挙動が少し異なります)。

Linux の x64 用を使う場合なら下記の操作でファイルを配置できます[^musl]。

[^musl]: ファイル名が alpine となっていますが、Arch Linux でも動作しています。

**図 3-1 CLI 版 `code` のインストール**

```shell-session
$ wget -q -O- 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' | tar -zxf -

$ ./code --version
Visual Studio Code CLI 1.74.0
```

### トンネル作成

上記のようにファイルを配置すると `code` コマンドが利用できるようになります。このコマンドにはトンネルを作成する `tunnel` サブコマンドが組み込まれています。

**図 3-2 `tunnel` サブコマンドが使える**

```shell-session
$ ./code tunnel --help
code-tunnel
Create a tunnel that's accessible on vscode.dev from anywhere. Run `code tunnel --help` for more
usage info

USAGE:
    code tunnel [OPTIONS] [SUBCOMMAND]

<snip...>
```

::: message

デスクトップ版 VSCode の `code` でも `tunnel` サブコマンドを使えますが挙動が微妙に異なります。たとえばデスクトップ版はライセンス受け入れやマシン名をインタラクティブに指定できません(コマンドライン引数からの指定になります)。逆に CLI 版では `--install-extension` などが使えません。

:::

実行するとライセンス受け入れについて表示されるので適宜応答しておいてください。

**図 3-3 初回実行時はライセンス受け入れを確認される**

```shell-session
$ ./code tunnel
*
* Visual Studio Code Server
*
* By using the software, you agree to
* the Visual Studio Code Server License Terms (https://aka.ms/vscode-server-license) and
* the Microsoft Privacy Statement (https://privacy.microsoft.com/en-US/privacystatement).
*
? Do you accept the terms in the License Agreement (Y/n)? (y/n) › yes
```

受け入れると GitHub アカウントへの接続が実施されます。下記のように表示されたら指定のアドレスをブラウザーで開き確認用のコードを入力します。

**図 3-4 GitHub アカウントへの接続用アドレスが表示される**

```shell-session
✔ Do you accept the terms in the License Agreement (Y/n)? · yes
To grant access to the server, please log into https://github.com/login/device and use code xxxx-xxxx
```

**図 3-5 ブラウザーで表示し確認用コードを入力**

![ブラウザーでデバイスの確認コードを入力しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/898eebb52a804139a28a4a967486b9bd/connect-my-machine-via-vscode-remote-tunnel-auth01.png?w=1036\&h=833\&auto=compress%2Cformat)

許可が完了するとマシン名を確認されます。これは vscode.dev 上に登録されるトンネルを識別する名前としても利用されます。

**図 3-6 マシン名を入力**

```shell-session
✔ Do you accept the terms in the License Agreement (Y/n)? · yes
To grant access to the server, please log into https://github.com/login/device and use code xxxx-xxxx
? What would you like to call this machine? (busy-lorikeet) › warehouse-13
```

すべての準備ができたら vscode.dev で利用するためのアドレスが表示されます。

**図 3-7 ブラウザーで利用するためのアドレスが表示される**

```shell-session
✔ Do you accept the terms in the License Agreement (Y/n)? · yes
To grant access to the server, please log into https://github.com/login/device and use code xxxx-xxxx
✔ What would you like to call this machine? · warehouse-13
[2022-12-11 08:22:12] info Creating tunnel with the name: warehouse-13

Open this link in your browser https://vscode.dev/tunnel/warehouse-13
```

トンネルの作成はこれで完了です。

:::message
「トンネルは作成したけど、接続されるサーバー(VSCode Server)は動かさなくてもよいの？」という疑問があるかもしれません。

サーバーは「クライアントがトンネルへ接続された」ときに `code tunnel` を動かしている PC 上に自動的に作成されます。具体的には `~/.vscode-server` 上に実行ファイルや設定が保存されます。

よって、コマンドなどで事前に作成しなくても大丈夫なようになっています(この辺の挙動は Remote - SSH などと同じような感じです)。
:::

## VSCode としての基本的な操作

トンネルを作成したので実際に外部のデバイスから VSCode として利用してみます。

### ブラウザーで開く

トンネル作成時に表示されたアドレスをブラウザーで開くと下記のようにウェブ版の VSCode が表示されます(初回実行時には GitHub への接続許可が必要な場合もあります)。画像からだとわかりにくいのですが、ブラウザーは `code tunnel` を動かしている PC とは別のネットワークにある Windows PC で実行しています。

**図 4-1 ブラウザーで開いた画面**

![ブラウザーでトンネル接続用のアドレスを開いたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/013ab043fe464fd6aac6c08fe8ea652a/connect-my-machine-via-vscode-remote-tunnel-browser-open.png?w=959\&h=635\&auto=compress%2Cformat)

上のスクリーンショットは通常の vscode.dev [^vscode-dev]と同じように見えますが、接続先 PC のターミナルを開くことができます。

<!-- textlint-disable -->

[^vscode-dev]: vscode.dev については「[Visual Studio Code for the Web (vscode.dev) が Public Preview になりました - Qiita](https://qiita.com/uikou/items/b5f6a0cd4228c46159dc)」が参考になります。

<!-- textlint-enable -->

**図 4-2 ターミナルを開くこともできる**

![トンネルに接続した状態でターミナルを開いているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2d5df6448a404578a76cf358d1e6d7ac/connect-my-machine-via-vscode-remote-tunnel-term.png?w=959\&h=635\&auto=compress%2Cformat)

よって、何らかのプロセス(タスク)を稼働させながらの開発が可能となります。感覚的には Remote - SSH を使っているような状態といえば理解しやすかもしれません(SSH 版にくらべて使えない機能もあります)。

**図 4-3 画像変換用のスクリプト(Marp + Puppeteer)を利用している様子**

![接続先 PC のプロセスを利用しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f73f054ee0154cb2b9f16d31827f2af0/connect-my-machine-via-vscode-remote-tunnel-marp.png?w=1036\&h=725\&auto=compress%2Cformat)

また、「リポジトリー別にコンテナで隔離された環境**ではない**」ので普段使っているようにコマンドを操作できます(これはこれでおっかない状態ですが)。たとえば、GitHub CLI の認証が完了していればターミナル上で普通に利用できますし、やろうと思えばサーバー側の USB ポートに接続したデバイスの操作などもできます[^local-usb]。

[^local-usb]: USB ポートはローカルから転送してくれた方がうれしいのですが、そういう機能はなさそうです。

**図 4-4 サーバーの USB ポートに接続した M5 StickC を確認(表示は一部加工しています)**

![ターミナルを開きデバイス一覧を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a07c0b84c46e4fe39b1da8f778dd72b4/connect-my-machine-via-vscode-remote-tunnel-pio-device.png?w=1036\&h=677\&auto=compress%2Cformat)

ただし、接続先のデスクトップ環境の利用は難しい状態です。シークレット情報(キーリングや各種エージェントなど)へのアクセスを「いわゆる GUI 前提で設定している」と使えない状況になるかもしれないので注意が必要です。

この他にはポート転送も使えるのですが、これは他のサービスが関連し制約もあるので後述します。

### ブラウザーで開く(モバイルデバイス)

vscode.dev は [PWA 対応されているので](https://qiita.com/uikou/items/b5f6a0cd4228c46159dc#pwa-%E3%81%A8%E3%82%AA%E3%83%95%E3%83%A9%E3%82%A4%E3%83%B3%E3%81%AE%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88)アプリとしてインストールできます。そうなるとモバイルデバイス対応が気になるところですが、そこはやはり対応されているようです。

*   [vscode.dev をローカル開発ツールとして使用する際にできること](https://qiita.com/uikou/items/b5f6a0cd4228c46159dc#vscodedev-%E3%82%92%E3%83%AD%E3%83%BC%E3%82%AB%E3%83%AB%E9%96%8B%E7%99%BA%E3%83%84%E3%83%BC%E3%83%AB%E3%81%A8%E3%81%97%E3%81%A6%E4%BD%BF%E7%94%A8%E3%81%99%E3%82%8B%E9%9A%9B%E3%81%AB%E3%81%A7%E3%81%8D%E3%82%8B%E3%81%93%E3%81%A8)

> *   **低消費電力マシンでのコード編集**。Chromebook などの環境でコードを編集することができます。
> *   **iPadでの開発**。ファイル アップロード、ダウンロードや、`Files` アプリを使ってクラウドに保存することもできます。ビルトインされた GitHub Repositories エクステンションを使ってリモートでリポジトリを開いたりすることもできます。また、Web ブラウザが`File System Access API` をサポートしていない場合でも、ブラウザを介してファイルをアップロードしたりダウンロードしたりして、ファイルを開くことができます。

手頃なタブレットがなかったので Android スマートフォンで試してみましたが、普通に接続できました。Chromebook や iPad なら結構よい感じになるのでないでしょうか。

**図 4-5 vscode.dev をアプリとしてインストールしての利用**

![vscode,dev をアプリとして利用しながらトンネルに接続したスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a1fb9ee4a8c142759444f51a9c09a5fb/connect-my-machine-via-vscode-remote-tunnel-remote-pwa.png?w=2160\&h=1080\&auto=compress%2Cformat)

### デスクトップ版 VSCode で開く

vscode.dev を利用すると面倒なインストールなしで VSCode を使えるようになるのでお手軽です。

しかし、ブラウザー上ではキーボードショートカットやクリップボードが当たってしまうこともあるのでデスクトップ版 VSCode でも使いたくなります。そのようなときは VSCode に Remote - Tunnels 拡張機能を入れることで対応できます。

@[card](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-server)

::: message

拡張機能は Release 版でもまだ少し不安定なようです。今回は使える機能の多さを優先して Preview 版で試しています。

:::

拡張機能をインストールしたらコマンドパレットから `Remote-Tunnels: Connect to Tunnel…` を入力します。初回利用時は許可を求められるので適宜進めてください。

**図 4-6 拡張機能を GitHub へ接続する許可を求められる**

![拡張機能を GitHub へ接続してよいか確認するダイアログボックス](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/072fa0768d7a44698b94e9043cc0f804/connect-my-machine-via-vscode-remote-tunnel-ext-auth01.png?w=409\&h=120\&auto=compress%2Cformat)

許可が完了すると利用可能なトンネル(サーバー)一覧が表示されるので選択すると接続されます。

**図 4-7 接続先を選択**

![トンネルの一覧が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/45a4b39399ce4744bd26879d85fac0bb/connect-my-machine-via-vscode-remote-tunnel-connect-to.png?w=658\&h=129\&auto=compress%2Cformat)

**図 4-8 接続後はブラウザーと同じように利用できる**

![デスクトップ版 VSCode がトンネルに接続された後、ターミナルを開いているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1d5962a23b5a45dc912471719b6f41ba/connect-my-machine-via-vscode-remote-tunnel-app-open.png?w=900\&h=675\&auto=compress%2Cformat)

また、[リモートエクスプローラー拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-explorer)を入れているとトンネル一覧を表示できます。ここから接続もできますが、まだ少し不安定なようです(スタックしてしまうことがあった)。

**図 4-9 リモートエクスプローラーによるトンネル一覧**

![リモートエクスプローラーでトンネル一覧を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f17abeab05c645c182f1f4d8c74572b2/connect-my-machine-via-vscode-remote-tunnel-remote-explorer.png?w=281\&h=380\&auto=compress%2Cformat)

感覚的には概ね「普通に Remote 系の拡張機能を使ってる」ような使用感ですが、DevContainer は利用できません。また、ポート転送もウェブ版のように他サービスとの組み合わせになります。

## トンネル特有の挙動

Remote Tunnels は Remote - SSH 拡張と似たような感じですが、いくつか目立った違いがありました。また、vscode.dev を利用している場合もトンネル特有の挙動があったのでその辺について。

### 拡張機能と設定

拡張機能の互換性などは気になるところですが、ここでは互換性とは別に Remote 系特有の挙動について。

トンネルで接続している場合でも「リモート(サーバー)側にも拡張機能のインストールや設定を保存をできる」ようになっています。

**図 5-1 拡張機能をサーバーへインストール**

![拡張機能のインストール時にインストール先を選択しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/af1e7c3e256345e3ac74506c1a955a5a/connect-my-machine-via-vscode-remote-tunnel-install-ext-to-server.png?w=789\&h=305\&auto=compress%2Cformat)

**図 5-2 サーバー別の設定**

![VSCode の設定画面でリモートサーバー用のタブを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/86cef1070c9541afb561e785511a5270/connect-my-machine-via-vscode-remote-tunnel-settings.png?w=588\&h=285\&auto=compress%2Cformat)

利用できない拡張機能や設定もありますが、後述のコンテナと組み合わせて「用途別に拡張機能を設定してあるサーバー」を用意しておくと便利かもしれません。

::: message

`code` コマンドの `--cli-data-dir` 引数でメタデータ(マシン名)を分けることで複数のトンネルを作成できますが、作成されるサーバーは 1 つだけです。 (厳密には PC 上の各ユーザー毎に 1 つです)

よって、「トンネルは別だけどサーバーインスタンスは同じ」場合だと「設定画面のタブ上では異なるリモートに見えるけど設定内容などは共有される」ので注意してください。

:::

### ポート転送

接続先 PC 上で動かしているサーバープロセス(Next.js のサーバーなど)のポートをデバイス側へ転送できます。ただし、デバイス側へ直接は転送されずに、devtunnels.ms を経由(トンネル)した形になります。

これはクライアントにデスクトップ版 VSCode を利用している場合でも同じでした。

**図 5-3 転送設定にポートを追加すると devtunnels.ms のアドレスが割り振られる**

![転送しているポート一覧で Local Address に devtunnels.ms のアドレスが割り振られているのを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2512132d388040df962a4465e632af2e/connect-my-machine-via-vscode-remote-tunnel-forward.png?w=720\&h=138\&auto=compress%2Cformat)

**図 5-4 割り振られたアドレスを開くと GitHub との接続許可を確認される**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7698fb5e83a94ec18be57ff09fe4b5d4/connect-my-machine-via-vscode-remote-tunnel-devtunnel-auth01.png?auto=compress%2Cformat)

**図 5-5 許可するとさらに接続先を信頼してよいか確認される**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/91468bd2f5c545aa8717517969cd0874/connect-my-machine-via-vscode-remote-tunnel-devtunnel-auth02.png?auto=compress%2Cformat)

このような形なので `curl` などからの利用は難しそうな感じです(ターミナル内で実行すれば直接接続はできます)。

通常のポート転送ができれば SSH や RDP の踏み台にできるかなと思っていたのですが、「そういうこと」をされないようにこの形になっているのかもしれません。

### Dev Container は利用できない

大変に残念なお知らせですが、(現時点では)Remote - SSH とは異なり接続先 PC の Docker を利用して Dev Container を作る方法はなさそうです。

*   [Can I use the Remote Development Extensions or a dev container with the VS Code Server?](https://code.visualstudio.com/docs/remote/vscode-server#_can-i-use-the-remote-development-extensions-or-a-dev-container-with-the-vs-code-server)

> Not at this time

「接続した後で `.devcontainer` 付きのフォルダーを開く」を試してみましたが、Dev Container を作ることはできませんでした。 (Remote SSH だとこの方法で Dev Container を作成できる)

下記の図を見ると将来対応してくれるような気もするので、それまでは別の方法で対応といったところでしょうか。

**図 5-6 Remote - SSH、Tunnels + Dev Containers が存在している([Remote Development Even Better](https://code.visualstudio.com/blogs/2022/12/07/remote-even-better)より引用)**

![](https://code.visualstudio.com/assets/blogs/2022/12/07/tunneling-blog-remote-spectrum.png)

現時点でどうしてもという場合は、「ターミナル内で [Dev Container CLI](https://github.com/devcontainers/cli) を使う」か後述するように「用途別のコンテナを作ってサーバーを動かす」といった対処になるかと思います。 (DevContainer 拡張機能に比べると制約はあるので完全な代替にはなりませんが)

### コミットの署名はされない

vscode.dev で作成したコミットを [GitHub へプッシュすると署名された状態になります](https://qiita.com/uikou/items/b5f6a0cd4228c46159dc#github-%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E9%96%A2%E9%80%A3%E3%81%AE%E3%82%A2%E3%83%83%E3%83%97%E3%83%87%E3%83%BC%E3%83%88)。しかし、トンネルで利用している場合は署名されないようです。

**図 5-7 ブラウザー版でコミットとプッシュしても Verified は付かない**

![GitHub 上のコミット一覧で Verfied が表示されていないスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d7278fbc50e047f0877038c4ceb7da51/connect-my-machine-via-vscode-remote-tunnel-not-signed.png?w=990\&h=132\&auto=compress%2Cformat)

普段、署名は気にしていなかったので詳しい検証はしていませんが、おそらくは接続先 PC 側に署名できる環境が必要かと思います。

## トンネルやサーバーの管理・設定など

トンネルはお手軽に作成できますが、それゆえ「なんかトンネルがいっぱいできてしまった」となったりします。また、接続先の PC を自由に操作できてしまうので制限をかけたくなります。ここではその辺についてなど。

### トンネルの管理

一度 `code tunnel` を動かすとサーバープロセスを停止しても vscode.dev 側ではトンネルが維持された状態になります。

`code` コマンドでも `unregister` などで接続解除したりなどできますが、複数のトンネルを利用していたりすると状態の把握などがちょっと難しい感じです。

トンネル一覧を確認しながらの操作は、現状ではリモートエクスプローラーを使うのが簡単かと思います。

@[card](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-explorer)

**図 6-1 リモートエクスプローラーから登録解除**

![リモートエクスプローラーのトンネル一覧上で右クリックし表示したメニューから登録解除しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ca09f1d0ea2a47f6984e293e8fef0478/connect-my-machine-via-vscode-remote-tunnel-unregister.png?w=393\&h=451\&auto=compress%2Cformat)

### サービス化

まだプレビューのようですが、`code tunnel` には `service` サブコマンドがあります。

    service       (Preview) Manages the tunnel when installed as a system service,

`code tunnel service install` を実行すると、トンネルがサービスとしてインストール(開始)されます。

**図 6-2 サービスとしてインストール**

```shell-session
$ code tunnel service install
*
* Visual Studio Code Server
*
* By using the software, you agree to
* the Visual Studio Code Server License Terms (https://aka.ms/vscode-server-license) and
* the Microsoft Privacy Statement (https://privacy.microsoft.com/en-US/privacystatement).
*
[2022-12-13 16:31:07] info Successfully registered service...
[2022-12-13 16:31:07] info Successfully enabled unit files...
[2022-12-13 16:31:07] info Tunnel service successfully started
Service successfully installed! You can use `code tunnel service log` to monitor it, and `code tunnel service uninstall` to remove it.
```

上記は Arch Linux で試してみたのですが、この場合は [Systemd のユーザーユニット](https://wiki.archlinux.jp/index.php/Systemd/%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC)を追加しているようです。他の環境では試していませんが、対応していればユーザーが所有するサービスとしてインストールされるのかと思います。

**図 6-3 実体としては Systemd のユーザーユニット**

```shell-session
$ systemctl --user status code-tunnel
● code-tunnel.service - Visual Studio Code Tunnel
     Loaded: loaded (/home/vscode/.config/systemd/user/code-tunnel.service; enabled; preset: enabled)
     Active: active (running) since Tue 2022-12-13 16:42:44 JST; 5s ago

<snup...>
```

個人用の PC で常時実行させておく場合には便利かもしれません。

### サーバーの隔離

前述のように Remote Tunnels を利用すると接続先 PC の機能(ハードウェア)を普通に利用できてしまいます。

便利ではありますが、やはり少しおっかない状態です。制限をかける場合は仮想マシンかコンテナを作成し、その中で `code tunnel` を動かすのがよいかと思います[^arm]。

[^arm]: ARM 用の CLI サーバーもあるので、用途によっては Raspberry PI などを専用サーバーにするのも面白いかもしれません。

下記はコンテナでトンネル機能を試したときの Dockerfile です。

@[card](https://github.com/hankei6km/test-code-server-container)

**リスト 6-1 alpine ベースのとりあえず動く Dockerfile**

```docker
ARG BASE_IMAGE=alpine:latest
FROM ${BASE_IMAGE}

SHELL ["/bin/ash", "-o", "pipefail", "-c"]

# [Option] Install zsh
ARG INSTALL_ZSH="false"

# Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ADD entry-point.sh /entry-point.sh
RUN apk update \
    && mkdir -p /tmp/library-scripts \
    && cd /tmp/library-scripts \
    && wget "https://raw.githubusercontent.com/microsoft/vscode-dev-containers/main/script-library/common-alpine.sh" \
    && ash common-alpine.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" \
    && rm -rf /tmp/library-scripts \
    && rm -rf /var/cache/apk/* \
    && chmod a+x /entry-point.sh

# Install code cli into user env.
USER "${USERNAME}"
RUN mkdir -p "/home/${USERNAME}/.vscode-cli" \
  && mkdir -p "/home/${USERNAME}/.vscode-server" \
  && mkdir -p "/home/${USERNAME}/Documents" \
  && mkdir -p /tmp/code \
  && cd /tmp/code \
  && wget -q -O- 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' | tar -zxf - \
  && mkdir -p "/home/${USERNAME}/.local/bin" \
  && cp code "//home/${USERNAME}/.local/bin/code" \
  && rm -rf /tmp/code 

ENTRYPOINT ["/entry-point.sh"]
```

**図 6-4 コンテナ実行**

```shell-session
$ mkdir vscode-cli vscode-server Documents
$ docker run -it -u vscode -v "${PWD}/vscode-cli:/home/vscode/.vscode-cli" \
    -v "${PWD}/vscode-server:/home/vscode/.vscode-server" \
    -v "${PWD}/Documents:/home/vscode/Documents" \
    --name vscode-tunnel  vscode-tunnel:latest
```

軽く試してみた感じではサーバーをコンテナにしても動くことは確認できました。上記のコンテナは最低限の環境ですが、[DevContainer 用のイメージ](https://github.com/devcontainers/images)を利用すると用途別のコンテナを比較的容易に作れるかと思います。

ただし、少し注意点もあります。コンテナでは資格情報(GitHub アカウントへの接続許可)の永続化は難しい状況です。コンテナの停止までは問題ないのですが、再作成すると再度 GitHub からの許可(ブラウザーから確認用のコード入力)が必要になります。

::: message

ドキュメントは見つけられなかったのですが、`code tunnel` では keyring が利用できない場合はフォールバックとして `~/.vscode-cli/` にトークンを書き込むようです。よって、コンテナ(サーバープロセス)を停止しても永続化されているのかと思います(コンテナ再作成後にトークンが使えなくなる利用は不明です)。

:::

::: message

(トークンとの相性わるいのかなと思い)コンテナでも keyring を使えるようにしてみるとシークレット情報の保存はできるのですが、それでもコンテナを再作成すると再度の許可が必要になってしまいます。

(仮想マシンの場合では再起動しても普通に keyring の資格情報を利用できています)

@[card](https://code.visualstudio.com/docs/editor/settings-sync?ck_subscriber_id=908806214#_troubleshooting-keychain-issues)

@[card](https://code.visualstudio.com/docs/remote/vscode-server#_i-see-an-error-about-keyring-storage-what-should-i-do)

:::

## VSCode セットアップ RTA(動画はないです)

最後に少しネタ的なことを。

ここまでいろいろ書いてきましたが、普通に Remote Tunnels を使うなら下記の手順で設定が完了します。

1.  CLI 版 `code` をいつも使っている PC へインストール
2.  `code tunnel` を実行してトンネル作成(デバイスへ接続許可)
3.  ブラウザーの vscode.dev からトンネルへ接続
4.  設定の同期をオンにする
5.  フォルダーを開く

あらためて書き出してみると「数分あればいつもの VScode 環境が用意できるのでは」という気になってきます。

ということで試してみたところ、いつもの VSCode が使える状態になるまで `7:27.95` でした(ブラウザーはあらかじめ GitHub へサインインした状態で試しています)。

**図 7-1 いつもの状態にした VSCode 環境**

![いつも使っている VSCode でフォルダーを開いているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ec36248a09d545e7894e2c56cacee95e/connect-my-machine-via-vscode-remote-tunnel-rta.png?w=1036\&h=636\&auto=compress%2Cformat)

現実には他にもいろいろ考慮する点はあるとはおもいますが、開発環境の設定が 10 分くらいでできるのならば「お手軽に設定できる」と言ってよいかと思います[^admim]。

[^admim]: 管理者側の立場からすると「お手軽に外部接続するのはやめてください」という感じかとは思いますが。

## おわりに

VSCode の Remote Tunnels を利用し、外部のデバイスから開発に使っている PC へ接続してみました。

少し使った印象だと「個人用の PC をお手軽にリモート開発用サーバーにするツール」といった感じで、似た系統として思い浮かぶ Codespaces とは少し方向性が違うのかなと思いました。 (`service` サブコマンドがもう少し充実するとまた印象が変わるかもしれませんが)

現状だと「ウェブ版を使いつつターミナルも開きたい」「Dev Container 作るほどではないが環境はある程度統一したい」などで補完的に使うと重宝するかもしれません。
