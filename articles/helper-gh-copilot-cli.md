---
id: helper-gh-copilot-cli
title: GitHub Copilot for CLI でコマンドを実行する前に編集などを行えるヘルパー関数を作ってみた
emoji: 🧩
type: tech
topics:
  - githubcopilot
  - cli
  - bash
published: true
---

先日から GitHub Copilot for CLI を便利に利用しているのですが、生成されたコマンドが実行される時の挙動が少し気になっています。

*   実行前にちょっとだけ編集するようなことができない
*   実行前に `shellcheck` でチェックしたい

ということで、その辺を解消するための Bash 用ヘルパー関数を作りました(Copilot for CLI にヘルパー関数という機能があるわけでなく、勝手に言ってるだけです)。

@[card](https://github.com/hankei6km/helper-gh-copilot-cli)

## どんなヘルパー？

クエリーを Copilot for CLI へ渡し、コマンドが生成されたら下記のようなことを実施します。

1.  `shellcheck`でチェックする[^continue]
2.  コマンドラインで入力中のテキストとしてセットする

[^continue]: 警告があってもコマンドラインへのセットは行われます。少し悩んだのですが、現行の Copilot for CLI はクエリー入力をやりなおす手間が大きいので、コマンドライン上で編集する方が良いだろうという判断です。

ただし、コマンドラインにセットするため、キー割り当てから Copilot for CLI を開始したりと操作体系が若干異なっています。

**図 1-1 コマンドラインへのセット、チェックの様子**

![Ctrl + A を押下することによって Copilot for CLI を開始し、生成したコマンドをコマンドラインへセットしたりチェックしている様子の動画](https://raw.githubusercontent.com/hankei6km/helper-gh-copilot-cli/main/images/demo.gif)

## 利用方法

リポジトリの `helper-gh-copilot-cli.sh` をどこかへ保存し、`source helper-gh-copilot-cli.sh` を実行します。あるいは下記のようにもできます。

**リスト 2-1 リポジトリから直接読み込む**

```bash
. <(curl -sL https://github.com/hankei6km/helper-gh-copilot-cli/raw/2ab2229d1ca955237684361f2d1f32d88275b4d3/helper-gh-copilot-cli.sh)
```

`source` を実行したあと、コマンドラインで何かクエリーを入力している状態で `Ctrl` + `a` を押下すると、Copilot for CLI が開始されます。生成されたコマンドを run すると `shellcheck` でチェックした後に入力中のコマンドへセットします。

セットされた後は通常のコマンドラインなので、編集や実行もいつも通りに行えます。

## よろしくないところ

勢いで作ったこともあり「ここはちょっとよろしくなかったかな」という部分もあります。

### `enter` を押しがち

これは慣れの問題かと思いますが、コマンドラインで `?? ファイル転送` のように入力するとそのままの勢いで `enter` してしまうことがわりとありました。

### `shellcheck` は思っていたより警告を出す

`ls *.sh` みたいな場合も警告が出るので、場合によっては「そこはちょっとご勘弁を」となることもあります。

## おわりに

Copilot for CLI 用のヘルパー関数を作ってみました。

勢いで `enter` してしまう対策はしたいところですが、わりと満足しています。
