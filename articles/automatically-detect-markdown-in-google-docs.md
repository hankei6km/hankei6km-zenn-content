---
id: automatically-detect-markdown-in-google-docs
title: Google Docs が Markdown のように見出しを入力できるようになっていた
emoji: 👨‍🎨
type: idea
topics:
  - google
  - googledocs
  - markdown
published: true
---

調べものをしていたら以下の記事を見つけました。

> テキストに書式設定を簡単に追加するために使われるのが「Markdown(マークダウン)」です。Googleが提供する組織向けオンラインアプリケーションセットのGoogle Workspaceの開発チームが、Googleドキュメントに「マークダウンを自動認識してその場で適用する機能」を追加すると発表しました。

@[card](https://gigazine.net/news/20220330-google-document-markdown/)

同じような機能は microCMS のリッチエディタにもあって便利に使っていますが、Google Docs でも使えるとなればかなりうれしい機能追加です。

## 何ができるようになるのか

これまでも行頭で「`-` + `Space`」と入力すると箇条書きのリストになるような機能がありましたが、これが他の Markdown の書式もサポートするようになったとのことです。

たとえば行頭で「`#` + `Space`」と入力すると「見出し 1」の書式、文中で「`Space` + `__あいうえお__`」と入力すると「あいうえお」に太字がセットされます。

## 機能を有効にする

この機能を使うには Google Docs での設定が必要になります。メニューから「ツール」「設定」を選択し「Markdown を自動検出する」にチェックを入れると有効になります。

なお、この設定は Google Docs 全体の設定になり、ファイル単位での設定はなさそうでした。

▼ **図 2-1 「Markdown を自動検出する」にチェック**

![Google Documents の設定画面を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/9c889c7a1b90472383f63f5bb6fe0ded/automatically-detect-markdown-in-google-docs-settings.png?w=528\&h=561\&auto=compress%2Cformat)

## 試してみる

文章だと伝わりにくいので動画にしました。

▼ **図 3-1 入力のサンプル**
![Google Docs 上で実際に入力している Animation GIF の動画](/images/automatically-detect-markdown-in-google-docs/automatically-detect-markdown-in-google-docs-examples.gif)

以下、気が付いた点でなど。

### 日本語

書式の設定範囲がズレるようなことはなかったのですが、`_` などの入力を開始する区切りに「全角スペース」は使えないようです。また全角での `＿` などは認識されないので IME の切り替えが頻繁になります。

### 誤検出の対策

Markdown 関連の機能では「ドキュメントで変数の説明を書いていると`_` で書式が設定される」ということがわりとあります。

これについては Google Docs の場合、「いったん書式が設定されたあと `Backscape` キーで戻るか Undo する」とその後の自動検出は回避されるようです。

### 書式クリア

書式をクリアするときに「Markdown でどうやってクリアするのか？」少し悩みましたが、書式のクリアは従来通り `Ctrl + Alt + 0` `Ctrl + \` などで行うようです。

### テーブル、コードブロックなど

後述の参考記事にもありますが、テーブルには対応していないようでした。

また、コードやコードブロックなど元々存在していない書式にも対応していないようです。

## 参考

@[card](https://workspaceupdates.googleblog.com/2022/03/compose-with-markdown-in-google-docs-on.html)

## おわりに

Google Docs で「Markdown ライクな入力を自動検出し、書式を適用する機能」を試してみました。

たまに Google Docs を利用すると「見出しの書式設定が面倒」と感じることが多く、利用を敬遠する理由の 1 つとなっていました。

今回の機能追加でここが改善されるので、個人的なメモに Google Docs を利用する頻度が上がりそうです。
