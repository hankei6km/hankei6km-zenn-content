---
id: vscode-deco-server
title: 接続したときだけ VSCode をかわいくするサーバーを作りましょう
emoji: 🐳
type: idea
topics:
  - vscode
published: true
---

少し前、[Remote Tunnels についての記事](https://zenn.dev/hankei6km/articles/connect-my-machine-via-vscode-remote-tunne)を書きました。

このときに「サーバー側の環境構築の実例もあった方がよいのかな？」と少し考えていたのですが、コンテナの設定などをダラダラ書いても「リモート系の設定ってお堅いのね」となりそうだなとも思っていました。

ということで下記の記事を参考に、かわいくするためのサーバーを用意しておくと Remote Tunnels でどのような効果が出るのか試してみたいと思います。

@[card](https://zenn.dev/kobito/articles/5a51afaa19abda)

## 対象とする読者

*   かわいくするためなら Remote Tunnels とかってやつも使いこなしてやりますぜ、という人

<!-- textlint-disable -->

または

<!-- textlint-enable -->

*   用途別のサーバーを用意しておいて切り替えたい人

そのような感じなので、Remote Tunnels や VSCode Server の説明は省略します。詳細は下記のドキュメントなどを参照してください。

@[card](https://code.visualstudio.com/docs/remote/tunnels)

## 何をするつもりですか？

この記事では接続したときだけ VSCode をかわいくするサーバーを作ってみます。

設定は[最近ちょっとお気に入りの ](https://zenn.dev/hankei6km/scraps/00831ff83f48ea)[`window.titleSeparator`](https://zenn.dev/hankei6km/scraps/00831ff83f48ea) と、冒頭の記事で参考にせていただいたものを組み合わせてみます。

以下、サーバーに接続してから切断するまでの概要。

**図 2-1 普段は普通の VSCode**

![サーバーとの接続を閉じた後に通常の表示に戻った VSCode のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/35b63ec9de484b22807ed4e86c39e5cf/vscode-deco-server-diconnected.png?w=869\&h=638\&auto=compress%2Cformat)

**図 2-2 サーバーに接続するとかわいい**

![サーバーに接続した VSCode の見た目が変わっているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5afd464da0ed429581ace952a87e5b5f/vscode-deco-server-connected01.png?w=869\&h=638\&auto=compress%2Cformat)

**図 2-3 さらに YAML のインデントもわかりやすくなる**

![YAML ファイルを開いてインデントが色分けされているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/20c839962fa449adb6509235266cfcee/vscode-deco-server-connected02.png?w=869\&h=638\&auto=compress%2Cformat)

**図 2-4 拡張機能も切り替わるのでエフェクトも付くよ**
![エディターで入力するとエフェクトが表示されている動画](/images/vscode-deco-server/vscode-deco-server-kitahhnn.gif)

**図 2-5 接続を閉じるといつもの VSCode**

![サーバーとの接続を閉じた後に通常の表示に戻った VSCode のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/35b63ec9de484b22807ed4e86c39e5cf/vscode-deco-server-diconnected.png?w=869\&h=638\&auto=compress%2Cformat)

## 作りましょう

まず、何らかの方法でサーバー(トンネル)を作成し VSCode から接続します。方法はいくつかありますが、[ここ](https://code.visualstudio.com/updates/v1_74#_remote-development)の動画のとおりにやるのが手っ取り早いかと思います。

接続した側の VSCode から `Ctrl` + `,` などで設定画面を開き、リモートのタブを選択(クリック)します。

**図 3-1 設定画面を開いたらリモートのタブをクリック**

![設定画面のリモート用のタブに赤枠でクリックする箇所を示したスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/21c7758bf31c442580910e2bd6e4764b/vscode-deco-server-settings-tab.png?w=869\&h=638\&auto=compress%2Cformat)

その後、右上のひっくり返すようなアイコンをクリックするとエディターが開きます。

**図 3-2 タブをクリックしたら右上のアイコンもクリック**

![設定画面の右上アイコンに赤枠でクリックする箇所を示したスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a0c3f915f937498ea197385405bfeaf5/vscode-deco-server-settings-json.png?w=869\&h=638\&auto=compress%2Cformat)

**図 3-3 エディターが開く**

![エディターが開いたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/77a3707ff97143039b15918dbc1f9860/vscode-deco-server-settings-editor.png?w=869\&h=638\&auto=compress%2Cformat)

::: message

通常は上記のように空の JSON ファイルが表示されますが、既に設定内容がある場合はバックアップしておくことをおすすめします。

:::

開いたエディターの `{}` 内に下記のリストの内容を追加します。

**リスト 3-1 設定内容**

```json
    "window.titleSeparator": " 💛 ",

    // https://zenn.dev/kobito/articles/5a51afaa19abda
    "workbench.colorTheme": "Illusion",
    "indentRainbow.colors": [
        "rgba(128, 64, 64, 0.5)", // 1インデント目の色
        "rgba(128, 96, 32, 0.5)", // 2インデント目の色
        "rgba(128, 128, 32, 0.5)", // 3インデント目の色
        "rgba( 32, 128, 32, 0.5)", // 4インデント目の色
        "rgba( 32, 64, 128, 0.5)", // 5インデント目の色
        "rgba(128, 64, 128, 0.5)" // 6インデント目の色
    ],
    "powermode.enabled": true,
    "powermode.combo.counterEnabled": "hide",
    "powermode.combo.counterSize": 0,
    "powermode.shake.intensity": 0
```

**図 3-4 設定を追加して保存(この時点でタイトルバーに💛が付きます)**

![エディターに設定を追加して保存した状態のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/00ebe43c9cee4bfda8f0c8c9791904e4/vscode-deco-server-settings-json-merged.png?w=869\&h=638\&auto=compress%2Cformat)

最後にターミナルを開いて下記のコマンドを実行します。

**リスト 3-2 ターミナルで実行するコマンド一覧**

```sh
code --install-extension rwietter.Illusion
code --install-extension oderwat.indent-rainbow
code --install-extension hoovercj.vscode-power-mode
```

コマンドの実行が完了したら完成です(リロードしてください的な通知が出たら適宜リロードしてください)。

**図 3-5 完成です**

![設定が適用されたサーバーのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/fe8ba11c1929446bb832b31b40a3fe45/vscode-deco-server-make-deco.png?w=869\&h=638\&auto=compress%2Cformat)

サーバーとの接続を閉じたり、再接続すると VSCode の設定が切り替わります。また、設定はサーバー側に保存されているので、他のデバイスから接続しても同じような状態になります。

**図 3-6 ブラウザーで vscode.dev から接続した場合**

![vscode.dev から接続してもデスクトップ版と同じような表示になっているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7e6d125cbb504dcc96d88f49e74bdfde/vscode-deco-server-other-device.png?w=1036\&h=702\&auto=compress%2Cformat)

## 他の設定でサーバーを作るには

上記以外の設定でサーバーを作る方法として、少しだけお堅い説明をします。

VSCode のリモート系の機能を使うとだいたいの場合でサーバー側に設定や拡張機能を保存できるようになっています。

今回の手順ですと、サーバー用の設定ファイル(`settings.json`)をエディターで開いて設定を貼り付けたことになります。今回は手間を省くためにコピペしましたが、設定画面の UI からの指定でも大丈夫です。

例として設定をちょっと変更してみます。クリスマスも近いのでタイトルにツリー生やしてみましょう。

「作りましょう」のときのようにサーバーの設定タブを開いて以下のように操作します。

**図 4-1 フィルターに titleSeparator と入力して設定項目を絞り込む(💛が設定されている)**

![設定画面でフィルターに titleSeparator を入力し項目が絞り込まれたスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/140a1bccc01e4f2193dc7ed54a9d2526/vscode-deco-server-customie-tree-filter.png?w=869\&h=638\&auto=compress%2Cformat)

**図 4-2 設定内容を 🎄 へ変更するとタイトルバーも変更される**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/bd3a97f8aa534e15a54f9475e8511694/vscode-deco-server-tree-replaced.png?auto=compress%2Cformat)

**図 4-3 UI での変更にあわせて `settings.json` も自動的に書き換わる**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9bbde098b2234a7eae17d4f5ceed7b78/vscode-deco-server-tree-replaced-json.png?auto=compress%2Cformat)

設定はこのような感じで変更して `settings.json` を控えておくとよいかと思います。

では、拡張機能はどうやってインストールしているのかというと…。これは `code` コマンド(VSCode の CLI 側インターフェース)をターミナルで実行することによりサーバー側へ一括インストールしています。

下記の場合ですと Extension ID に `oderwat.indent-rainbow` を指定することで [indent-rainbow 拡張機能](https://marketplace.visualstudio.com/items?itemName=oderwat.indent-rainbow)をインストールしています。

```sh
code --install-extension oderwat.indent-rainbow
```

この Extension ID 部分を変更することで他の拡張機能もインスールできます。Extension ID は VSCode で拡張機能のページを開き、歯車アイコンをクリックするとコピーできます。

**図 4-4 Extension ID をコピーするメニュー項目表示**

![VSCode 内の拡張機能ページで歯車アイコンをクリックしてポップメニューが表示されているスクリーンショット\*\*](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/979f05653fc24637b0d96fe16a7be13a/vscode-deco-server-copy-ext-id.png?w=682\&h=362\&auto=compress%2Cformat)

よい感じの拡張機能をインストールしたら、Extension ID を控えておくとサーバーへ一括インストールしやすくなるかと思います。

::: message

拡張機能によっては外部にファイルが必要だったり、リモート環境では動かせないこともあります。試してみた感じでは、アイコン置き換え系はダメぽいです(今回の用途ではできれば使いたかったのですが)。

:::

::: message

気が付いている方もいらっしゃると思いますが、今回の記事ではフォントについて触れていませんでした。フォントも指定自体はできますが、フォントファイルのインストールはローカル側で実施する必要があります。

:::

## おわりに

お堅い印象なリモート系の機能ですが、楽しく使うと親しみやすくてよい感じかなと。
