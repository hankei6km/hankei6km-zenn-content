---
id: install-gh-cli-from-release
title: DevContainer の Image などに従来の方法で GitHub CLI がインストールできなくなったことへの対応
emoji: 🧰
type: idea
topics:
  - githubcli
  - devcontainer
published: true
---

DevContainer 用の Image を定期的にビルドさせているのですが急にエラーになったので調べてみたところ、以下のような原因でした。

@[card](https://github.com/cli/cli/issues/6175)

この問題は長期化するということらしいです(この記事を書いている時点で 3 日経過しているので緊急な対応はなさそう)。

@[card](https://github.com/cli/cli/issues/6175#issuecomment-1235984381)

対応方法としては「Release のパッケージを自分でダウンロードしてインストールしてください」ということですが、[ドキュメントに具体的な手順が掲載される感じもないので](https://github.com/cli/cli/blob/trunk/docs/install_linux.md#debian-ubuntu-linux-raspberry-pi-os-apt)、とりあえずの対応方法など。

:::message
この記事の内容はおそらくすぐに古くなります。とくに後述の [Script Library](https://github.com/microsoft/vscode-dev-containers/blob/main/script-library/docs/github-cli.md) は対応される可能性もあるので最新の情報をチェックしてください。
:::

## 対応方法

`devcontainer.json` で features を使っている場合は対応版が用意されているようです。私は features を使っていないので試していませんが、一時的にエラーになっていても最新版では対応されていると思います[^features]。

[^features]: features については「[Dev container featuresについて調べてみる](https://zenn.dev/nmemoto/scraps/9eee0f54dc30ed)」が参考にります。

@[card](https://github.com/cli/cli/issues/6175#issuecomment-1235998656)

> 👋 For anyone using Codespaces, Remote-Containers, or the [dev container CLI](https://github.com/devcontainers/cli), we have updated the dev container feature to prefer pulling from the latest GitHub Release instead of adding the apt repo. (see: [devcontainers/features#133](https://github.com/devcontainers/features/pull/133))

`Dockerfile` で [Script Library](https://github.com/microsoft/vscode-dev-containers/blob/main/script-library/docs/github-cli.md) を使っている場合(あるいは自分でリポジトリを追加している場合)、まだ上記のようなオプションはないので自分で解決する必要があります。

私は以下のように変更しました(`apt-get update` などは事前に実行している前提)。

**リスト 1-1 Relase からインストールするスクリプト**

```bash
# Install GihHub CLI
    && curl -s https://api.github.com/repos/cli/cli/releases/latest | jq .assets[].browser_download_url | grep linux_amd64.deb | xargs -I '{}' curl -sL -o /tmp/ghcli.deb '{}' \
    && dpkg -i /tmp/ghcli.deb \
    && rm /tmp/ghcli.deb \
```

上記でビルドすると現時点での最新版がインストールされています。

**図 1-1 インストールされた gh のバージョン表示**

```jsx
vscode ➜ ~ $ gh version
gh version 2.14.7 (2022-08-25)
https://github.com/cli/cli/releases/tag/v2.14.7
```

あとは Issue をざっと見た感じだと、以下のコメントの方法でも対応できそうです(こちらの方が Architecture を見ているので柔軟かも)。

@[card](https://github.com/cli/cli/issues/6175#issuecomment-1236264607)

## おわりに

最初は「鍵ファイルをダウンロードできなかったのかな？」と軽い気持ちで re-run したのですが、わりと対応が必要だったのでとりあえず記事にしてみました。
