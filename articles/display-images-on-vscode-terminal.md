---
id: display-images-on-vscode-terminal
title: VSCode のターミナル内で画像を表示できるようになったので試してみた
emoji: 🖼️
type: tech
topics:
  - vscode
  - terminal
  - image
published: true
---

Zenn と連携しているリポジトリへ push するとエラーになるので、問題切り分けの記事。切り分けができたら上げなおす。

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c5f6a3162860436d9e481d689dbba334/Untitled.png?auto=compress%2Cformat)

VSCode の更新情報で Preview features に「[Images in the terminal](https://code.visualstudio.com/updates/v1_79#_images-in-the-terminal)」という項目がありました。

> There is now experimental support for images in the terminal. Images in a terminal typically work by encoding the image pixel data as text, which is written to the terminal via a special escape sequence. The current protocols that are supported are [sixel](https://en.wikipedia.org/wiki/Sixel) and the [inline images protocol pioneered by iTerm](https://iterm2.com/documentation-images.html).

個人的にはわりとうれしい機能なので少し試してみることに。

## ターミナル内で画像を表示させるとは？

画像を表示させる方法はいくつかありますが、VSCode では [sixel](https://en.wikipedia.org/wiki/Sixel) と [ITerm2 のインライン画像表示プロトコル](https://iterm2.com/documentation-images.html)に対応されたとのこと。

大雑把に書くとどちらも「画像のデータを特定の手順で出力(cat)すると、テキストではなく画像が表示される」といった動作をします。詳しい内容については下記のページが参考になるかと思います。

**Sixel に対応しているツールなどについて:**

@[card](https://qiita.com/arakiken/items/3e4bc9a6e43af0198e46)

**Sixel と iTerm2 の画像表示プロトコルについて:**

@[card](https://io.cyberdefense.jp/entry/%E3%82%BF%E3%83%BC%E3%83%9F%E3%83%8A%E3%83%AB%E5%86%85%E3%81%A7%E7%94%BB%E5%83%8F%E8%A1%A8%E7%A4%BA%E3%81%99%E3%82%8B%E6%89%8B%E6%B3%95%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E3%81%8B%E3%82%93%E3%81%8C%E3%81%88%E3%82%8B/)

VScode で sixel を利用すると、(実際に使うかは置いておいて)このようなこともできます。

**図 1-1 w3m で画像表示**

![VSCode にターミナルで w3m を利用し Google 画像検索の結果を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0007e6ed2a9e4a16abd4d65718236cfd/display-images-on-vscode-terminal-w3m.png?w=1440\&h=860\&auto=compress%2Cformat)

これはこれで面白いのですが、画像を表示するという目的に限定した場合は iTerm2 の画像表示プロトコルの方が簡単に試すことができました。よって、この記事では主に iTerm 2 の画像表示プロトコルを利用する方法を記述していきます。

## 機能の有効化

現状では明示的に有効化する必要があります。

**図 2-1 有効化するための設定**

    "terminal.integrated.experimentalImageSupport": true

## 画像ファイルを表示する

単純に画像を表示するだけなら iTerm2 用のシェルスクリプトを使うのが簡単かと思います。

@[card](https://iterm2.com/documentation-images.html)

**リスト 3-1 シェルスクリプトの利用**

```sh
wget https://iterm2.com/utilities/imgcat
chmod u+x imgcat
./imgcat --width 60 /path/to/image.png
```

また、同様の機能を実現する Python のパッケージもあります。

@[card](https://pypi.org/project/imgcat/)

**図 3-1 imgcat で画像を表示**

![VSCode のターミナルで imgcat を利用し画像ファイルを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c1be3ce2b48a455a9327ac1113a1494d/display-images-on-vscode-terminal-imgcat.png?w=1440\&h=860\&auto=compress%2Cformat)
