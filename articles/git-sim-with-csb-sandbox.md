---
id: git-sim-with-csb-sandbox
title: Git 操作を視覚化する git-sim を CodeSandbox で使ってみた
emoji: 🎞️
type: idea
topics:
  - git
  - codesandbox
published: true
---

下記の記事を見て [`git-sim`] が面白そうだったのと、CodeSandbox の[リポジトリ](https://codesandbox.io/docs/learn/repositories/overview)と [Docker インテグレーション](https://codesandbox.io/blog/introducing-docker-support-in-codesandbox)で何か動くものを試してみたかったので環境を作ってみました。

@[card](https://codezine.jp/article/detail/17240)

## 何ができるようになる？

[`git-sim`] は Git で `merge` などを実行するときに、事前に実行結果を視覚化できるツールとのこと。

@[card](https://github.com/initialcommit-com/git-sim)

今回は、その [`git-sim`] をターミナルで動かせる CodeSandbox のリポジトリを作りました。これを利用することで下記のような画像などを作成できるようになります。

**図 1-1 サンプル**
![Git のブランチをマージしている動画](/images/git-sim-with-csb-sandbox/git-sim-with-csb-sandbox-intro01.gif)

[`git-sim`]: https://github.com/initialcommit-com/git-sim

## 必要な環境

利用するためには GitHub と CodeSandbox のアカウントが必要になります。また、動画ファイルの確認などがあるので、今回は CodeSandbox のクラウド ID は使わずに VSCode を利用します。

## セットアップ

::: message

CodeSandbox のパブリックなリポジトリ(サンドボックス)を使う場合は扱う内容に気を付くてください。

:::

下記のリポジトリをテンプレートにするかクローンするなどで、GitHub 上にリポジトリを作成します。

@[card](https://github.com/hankei6km/test-git-sim-csb-docker)

作成したリポジトリを CodeSandbox の Dashboard から Import します。

**図 3-1 Dashboard 右上の `+ Create` ボタンクリックし Import する**

![CodeSandbox 上で Create 用のパネルから GitHub のリポジトリを Import するパネルを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/34281dc54b924395b2ca9170633e191d/git-sim-with-csb-sandbox-import.png?w=994\&h=620\&auto=compress%2Cformat)

しばらく待つと下記のような画面になります(少しわかりにくいですが、右下の「Keep your project ～」の下に `dummy` と表示されます)。ここで CodeSandbox の GitHub アプリ利用を提案されていますが、今回は利用しなくてもとくに影響はありません。

**図 3-2 Import が完了した状態**

![CodeSandbox の IDE で Import 用のタスクが完了しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5f83086353b5424b80f7a8994512c3fa/git-sim-with-csb-sandbox-import-done.png?w=1119\&h=529\&auto=compress%2Cformat)

## 画像を作ってみる

この後はターミナルでコマンドを実行することになりますが、CodeSandbox の IDE 上のファイルはターミナルによる処理が反映(同期)されないこと場合もあります。

よって、ここからは VSCode から利用することにします。VSCode から利用する方法はいくつかありますが、今回の場合は IDE の右上に表示されている `VS Code` ボタンをクリックするのが簡単かと思います。

**図 4-1 IDE 右上の `VS Code` ボタン**

![IDE 右上の VS Code ボタンに赤枠を付けたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/50e37de2be734698b5030e6092e912e6/git-sim-with-csb-sandbox-open-vscode.png?w=501\&h=572\&auto=compress%2Cformat)

クリックすると各種認証や拡張機能のインストールなどが行われるので問題なければ適宜進めてください。

VSCode 上でリポジトリを開いたらターミナルで下記のコマンドを実行します(`-d` は、作成されたファイルの自動再生を抑止するフラグです)。なお、`root` で操作していますが、Docker インテグレーションの仕様に合わせたもので、[`git-sim`]は通常ユーザーでも操作できます。

```shell-session
# git-sim -d log
```

少し待つと `git-sim_media/images` の下に画像が作成されます。テンプレートから作成した場合はコミットが 1 つだけなのでちょっと寂しいですが、これで [`git-sim`] が動作していることは確認できました (画像の内容はそのときのリポジトリの履歴で異なります)。

![作成された画像を VSCode 上で表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/53460f28f857428e97912cb456a85ca0/git-sim-with-csb-sandbox-usage-log.png?w=1440\&h=860\&auto=compress%2Cformat)

::: message

現在、[`git-sim` ] は活発に更新されている状況なのでビルドのタイミングによっては不具合が出るかもしれません。その場合は下記のようにバージョンを指定して再インストールすると解決することがあります。

```bash
pip3 install "git-sim==0.1.6"
```

:::

もう少し実際的な画像作成を試す場合、それ用にダミーのリポジトリを作成してあります。下記のような感じで利用できます。

    source /workspace/scripts/alias-git-sim
    cd ~/tmp
    git clone https://github.com/hankei6km/test-git-sim-csb-docker-mock.git
    cd test-git-sim-csb-docker-mock
    git checkout branch1
    git checkout -
    git-sim merge branch1

少し補足すると、`source /workspace/scripts/alias-git-sim` でエイリアスを作成し、[`git-sim`]が作成するファイルをワークスペースへ保存するようにしています。 (これがないとコマンド実行時のカレントの下へ保存されます)

## 動画を作成してみる

動画の作成も [`git-sim` ]に渡すオプションが少し増えるだけで、基本的な流れは画像の作成と同じようになります。

上記のダミーリポジトリを clone した状態で、下記のコマンドを実行します。

```shell-session
# git-sim --animate merge branch1
```

少し待つと `git-sim_media/videos/1080p60/GitSim.mp4`に動画が保存されます。VSCode の場合はファイルを開くと動画を再生できるようになります(ちょっと時間がかかると思います)。

**図 5-1 作成された動画は VSCode で再生できる**

![動画ファイルを VSCode で開き、再生用の UI が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/19a696243afb435b9ab2d764dfec7b2e/git-sim-with-csb-sandbox-video-merge.png?w=1440\&h=860\&auto=compress%2Cformat)

また、animation gif へ変換する簡易スクリプトも作成してあります。`/workspace/scripts/agif.sh` を実行すると上記ファイルを入力として、`output.gif` が作成されます。

**図 5-2 変換された画像**
![上記の動画を animation gif へ変換した画像](/images/git-sim-with-csb-sandbox/git-sim-with-csb-sandbox-video-merge.gif)

動画はタイトル文字列とロゴなどのカスタマイズができるので、いろいろ試してみると面白いかと思います。下記はライトモードにロゴ画像(`my-profile-round.png` は別途用意)などを指定している例です。

```shell-session
# git-sim --animate --light-mode --show-intro --logo /workspace/my-profile-round.png --title "git merge branch1" merge branch1
```

**図 5-3 カスタマイズした画像**
![ライトモードを指定した動画](/images/git-sim-with-csb-sandbox/git-sim-with-csb-sandbox-video-light-mode.gif)

## 日本語の扱い

これを書いている時点(`git-sim` の `0.1.8`)ではタイトルなどの日本語は文字化けするときがあります。先頭にいわゆる半角ブランクを挟んだりすると文字化けしなかったりと、原因と対応方法は残念ながらわかっていません。

## おわりに

[`git-sim`] を CodeSandbox のサンドボックスで動かしてみました。

[`git-sim`] でサポートされる Git 操作はまだ限定的ですが[^revert]、 視覚的にわかりやすい動画を作れるのがうれしいところです。

また、この記事の隠れた？テーマである CodeSandbox のリポジトリと Docker インテグレーションについてはあまり触れませんでしたが、こちらも良い感じなので[^nix-podman]最近はよく利用しています。

[^revert]: 今回の環境を作っているときにマージコミットを `revert` しようとして「ちょうどよいので `-m` を視覚化してもらおうかな」と思ったのですが、まだサポートされていませんでした。ですが、Issue がいい勢い解決されていってるようなので、充実していきそうかなと思います

[^nix-podman]: Docker インテグレーション以外にも Nix やコマンドラインからも Docker(Podman) を使えたりするので、ちょっと実験するのに便利だったりします。
