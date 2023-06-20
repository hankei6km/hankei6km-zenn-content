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

## どのようなときに利用できそうか

VSCode では `code` コマンドからでも画像を表示できるのですが、タブが切り替わってしまうので「ちょっとプレビュー的に見たいときは大げさかな」と思っていました。

よって、コマンドで動的に生成された画像データなどをプレビュー的に表示したいときは、ターミナルで表示させると軽快な操作性になるかと思います。

**図 1-1 jest-image-snapshot の diff 画像を表示**
![npm run でテスト実行時に画像の比較が失敗し、その diff 画像をターミナルで表示しているスクリーンショット](/images/display-images-on-vscode-terminal/display-images-on-vscode-terminal-diff.gif)

## WASM WASI 用ターミナル

Web Shell でターミナルを開き Python 版の imgcat を利用してみたところ、こちらも利用できました。画像を表示する WASI 拡張機能を作るとき、一時ファイルへの保存や WebView の利用などは考えなくても良くなりそうです。

**図 2-1 Web Shell で画像表示**

![Web Shell で imgcat を実行しターミナル上で画像を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/17dea80dc57347af8af8aac314e5d2a2/display-images-on-vscode-terminal-webshell.png?w=1440\&h=860\&auto=compress%2Cformat)

## おわりに

VSCode のターミナル内で画像を表示してみました。

VSCode では `code` コマンドからも画像を表示できるので、表示するだけならそれほど困ってはいなかったのですが、ターミナルで表示できるとやはり操作性が良くなると思いました。

しかしながら、まだツールの相性もあるようなので(理屈では ranger の画像プレビューも使えるはずなのですが、何故かうまくいかない)、しばらくは試行錯誤も必要そうかなといったところでしょうか。
