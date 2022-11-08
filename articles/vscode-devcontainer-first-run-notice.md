---
id: vscode-devcontainer-first-run-notice
title: VSCode + DevContainer で最初にターミナルを開いたときに通知を表示する
emoji: 📑
type: idea
topics:
  - vscode
  - codespaces
  - devcontainer
published: true
---

VSCode(または Codespace) + DevContainer では、利用するイメージにもよりますが「コンテナ作成後、最初にターミナルを開いたとき簡単な通知を表示させる機能」が利用できます。

今回は、この通知をカスタマイズする方法などについて。

**図 1 通知のカスタマイズ例**

![VSCode でターミナルを開き、カスタマイズした通知メッセージを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/11b4566cd46f4e948ff56232a48fbc34/vscode-devcontainer-first-run-notice-example.png?w=1014\&h=432\&auto=compress%2Cformat)

![VSCode(Light テーマ)でターミナルを開き、カスタマイズした通知メッセージを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/becbb71cead24a1d91f22420cc6477fc/vscode-devcontainer-first-run-notice-example-light.png?w=993\&h=409\&auto=compress%2Cformat)

## 利用できるイメージは？

<!-- textlint-disable -->

公式なドキュメントなどはなさそうですが、[`devcontainers/features`](https://github.com/devcontainers/features) の [Common Debian Utilities (common-utils)](https://github.com/devcontainers/features/tree/main/src/common-utils) などに表示用の処理が含まれています。

<!-- textlint-enable -->

上記 feature では `.bashrc` と `.zshrc` に下記のような処理を設定しています。

@[card](https://github.com/devcontainers/features/blob/a8cb375d460840bbf8c91599d16fc87d9ee8b996/src/common-utils/install.sh#L238-L248)

```bash
# Display optional first run image specific notice if configured and terminal is interactive
if [ -t 1 ] && [[ "${TERM_PROGRAM}" = "vscode" || "${TERM_PROGRAM}" = "codespaces" ]] && [ ! -f "$HOME/.config/vscode-dev-containers/first-run-notice-already-displayed" ]; then
    if [ -f "/usr/local/etc/vscode-dev-containers/first-run-notice.txt" ]; then
        cat "/usr/local/etc/vscode-dev-containers/first-run-notice.txt"
    elif [ -f "/workspaces/.codespaces/shared/first-run-notice.txt" ]; then
        cat "/workspaces/.codespaces/shared/first-run-notice.txt"
    fi
    mkdir -p "$HOME/.config/vscode-dev-containers"
    # Mark first run notice as displayed after 10s to avoid problems with fast terminal refreshes hiding it
    ((sleep 10s; touch "$HOME/.config/vscode-dev-containers/first-run-notice-already-displayed") &)
fi
```

よって、通知を表示するにはこの処理が含まれているイメージをベースにする必要があります。とは言っても[`devcontainers/images`](https://github.com/devcontainers/images) に含まれている大体のイメージでは上記 feature は使われているようです(Apline 用イメージにも[同様の処理が含まれています](https://github.com/devcontainers/images/blob/main/src/base-alpine/.devcontainer/library-scripts/common-alpine.sh))[^vscode-devv-containers]。

[^vscode-devv-containers]: 従来の [`microsoft/vscode-dev-containers`](https://github.com/microsoft/vscode-dev-containers/) のイメージでも大体は含まれていると思います。

## 表示される仕組み

上記コードを読んでみると対話形式でターミナルを開いたとき、下記のファイルが存在しているとその内容が表示(`cat`)されるようになっています。

*   `/usr/local/etc/vscode-dev-containers/first-run-notice.txt`
*   `/workspaces/.codespaces/shared/first-run-notice.txt`

<!-- textlint-disable -->

通知が表示されると、フラグとして `$HOME/.config/vscode-dev-containers/first-run-notice-already-displayed` が作成され、それ以降は表示されなくなります。

<!-- textlint-enable -->

## 通知の内容を変更する

仕組みとしては対象となるファイルを配置すればよさそうです。いくつか方法はありますが、ここでは `Dockerfile` でファイルを配置してみます。

**リスト 3-1 ファイルを配置する `Dockerfile`**

```docker
# [Choice] Ubuntu version (use ubuntu-22.04 or ubuntu-18.04 on local arm64/Apple Silicon): ubuntu-22.04, ubuntu-20.04, ubuntu-18.04
ARG VARIANT=ubuntu-20.04
FROM mcr.microsoft.com/devcontainers/base:0-${VARIANT}

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

COPY first-run-notice.txt /usr/local/etc/vscode-dev-containers/first-run-notice.txt
```

**リスト 3-2 `first-run-notice.txt`**

```
🚀 DevContainer で実行しています。
📗 利用方法(Ctrl + Click): README.md

```

コンテナを再作成するとカスタイマイズした通知が表示されます。

**図 3-1 ターミナルを開くと通知が表示される**

![VSCode でターミナルを開いて通知が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f64d95a888ab4e969d7a0e40acb84a08/vscode-devcontainer-first-run-notice-basic.png?w=638\&h=173\&auto=compress%2Cformat)

テキストファイルが `cat` されるだけなので、Markdown のような装飾はできませんが、ファイル名や URL を表示すれば端末上で `Ctrl` + `Click` はできます。これを利用すればドキュメントにリンクできるかと思います。

## 少し装飾する

カスタマイズできるならやはり色などは付けてみたくなります。よって、少しだけ装飾してみたいと思います。

::: message

下記の方法はターミナルの設定や幅などで想定外の表示になることもあります。こういう記事を書いておいて言うのもアレですが、カスタマイズは程々が良さそうです。

:::

### 色を付ける

色を付ける方法として `tput` などがありますが、メッセージ表示時にコマンドを実行できないので、テンプレート的なスクリプトを作成してファイルを静的に作成しておくのが簡単かと思います。

**リスト 4-1 テンプレート的なスクリプト**

```bash
#!/bin/bash

cat <<EOF
🚀 $(tput setaf 2)DevContainer で実行しています。$(tput sgr0)
📗 $(tput setaf 3)利用方法(Ctrl + Click)$(tput sgr0): $(tput setab 4) README.md $(tput sgr0)

EOF
```

**図 4-1 スクリプトを実行して `first-run-notice.txt` を生成**

```shell-session
@hankei6km ➜ /workspaces/test-vscode-first-run-notice (main ✗) $ ./gen-notice.sh > .devcontainer/first-run-notice.txt
```

**図 4-2 色付きで通知が表示される**

![VSCode でターミナルを開いて通知が色付きで表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a1e5f0c8f058400bb442afee7ec18e12/vscode-devcontainer-first-run-notice-color.png?w=676\&h=162\&auto=compress%2Cformat)

### バナー風

ターミナルで文字を大きくする方法としては `figlet` を使うのが簡単かと思います。これもコマンドを実行できないので、別の端末で実行したものを貼り付けることになります。

**図 4-3 `figlet` でバナー風文字を出力**

```shell-session
$ figlet -f banner Container
```

**リスト 4-2 `figlet` の出力を貼り付ける**

```bash
#!/bin/bash

cat <<EOF
🚀 $(tput setaf 2)DevContainer で実行しています。$(tput sgr0)
$(tput setaf 6)
 #####                                                    
#     #  ####  #    # #####   ##   # #    # ###### #####  
#       #    # ##   #   #    #  #  # ##   # #      #    # 
#       #    # # #  #   #   #    # # # #  # #####  #    # 
#       #    # #  # #   #   ###### # #  # # #      #####  
#     # #    # #   ##   #   #    # # #   ## #      #   #  
 #####   ####  #    #   #   #    # # #    # ###### #    # 
$(tput sgr0)
📗 $(tput setaf 3)利用方法(Ctrl + Click)$(tput sgr0): $(tput setab 4) README.md $(tput sgr0)

EOF
```

**図 4-4 バナー風通知が表示される**

![VSCode でターミナルを開いて通知がバナー風に表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b31c25a5c3ab49a68d4004c8ff66bcc9/vscode-devcontainer-first-run-notice-banner.png?w=684\&h=335\&auto=compress%2Cformat)

なお、ターミナルの幅に左右されるので注意してください。

**図 4-5 ターミナルの表示幅がせまいと表示がくずれる**

![VSCode で表示幅が狭いターミナルに通知が表示され、内容が崩れているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/99c1097035444d32a12d241cfe2bcb02/vscode-devcontainer-first-run-notice-narrow.png?w=450\&h=451\&auto=compress%2Cformat)

## お試し用リポジトリー

通知をカスタマイズした DevContainer を含むリポジトリーを作成しました。

@[card](https://github.com/hankei6km/test-vscode-first-run-notice)

クローンして VSCode から開くか Codespace を作成した後、ターミナルを開くと「図 1 通知のカスタマイズ例」のようなメッセージが表示されます。

## 考慮点

### 明確な仕様ではない(たぶん)

<!-- textlint-disable -->

もともと DevContainer の仕様というわけではなさそうでったのですが、[DevContainer をよりオープンにする流れ](https://github.com/microsoft/vscode-dev-containers/issues/1589)のなかで、[Common Debian Utilities (common-utils)](https://github.com/devcontainers/features/tree/main/src/common-utils) を[見直す動きもあるようです](https://github.com/devcontainers/features/issues/67)。利用方法が変更される、あるい利用できなくなるかもしれません。

<!-- textlint-enable -->

### Codespaces の通知

Codespace を作成した後、下記のような通知が表示されます。今回のカスタマイズを適用するとこれが表示されなくなります。

**図 6-1 Codespaces での通知**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c1d145375ee640a2a1f6f4d5ef8941c8/vscode-devcontainer-first-run-notice-in-codespace.png?auto=compress%2Cformat)

## おわりに

VSCode + DevCotainer で最初にターミナルを開いたときに表示される通知を変更してみました。

ベースにしているイメージが通知の表示をサポートしている必要はありますが、ファイルを配置するだけで簡単に表示できました。コンテナ固有の機能がある場合、簡易的な注意点の表示などに利用できそうかなと思います。
