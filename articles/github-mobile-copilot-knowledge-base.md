---
id: github-mobile-copilot-knowledge-base
title: GitHub Mobile でリポジトリを開くだけの簡単なナレッジベースです
emoji: 💬
type: idea
topics:
  - githubmobile
  - githubcopilot
  - github
published: true
---

GitHub Mobile と GitHub Copilot を使って各種ドキュメントなどに質問する方法がわりと簡単だったのでその辺についてなど。

## 想定している読者

*   GitHub のリポジトリとして公開されているドキュメントなどを簡単に調べたり要約したい人

::: message

Enterprise 版の Copilot では[チャットを GitHub.com(ブラウザー)から利用するオプション](https://docs.github.com/ja/enterprise-cloud%40latest/copilot/quickstart)などがあるようですが、Enterprise を使っていないのでその辺には触れません。

:::

## GitHub Mobile と GitHub Copilot

GitHub Mobile は GitHub Copilot を有効化しているとチャット機能を使えるようになっています。

@[card](https://docs.github.com/ja/copilot/using-github-copilot/asking-github-copilot-questions-in-github-mobile)

このチャット機能の良い点として「開いている画面をコンテキスト的に使える」ことにあります。たとえば、特定のリポジトリの画面でチャットを開始すると、そのリポジトリのコードなどについて質問できます。 (Edge ブラウザーで Copilot をサイドバーから利用するような感じです)

**図 2-1 リポジトリを開いて画面右下の Copilot アイコンをタップ**

![GitHub Mobile で特定のリポジトリを開いた画面のスクリーンショット、画面右下の Copilot アイコンに赤枠が付いている](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a3c1c9a659434b39a9a70e8d06e499f9/github-mobile-copilot-knowledge-base-copilot-button.png?w=864\&h=1920\&auto=compress%2Cformat)

**図 2-2 リポジトリについて質問すると調べてくれる**

![「タイトルをクリップボードへコピーしている方法は？」と質問し、返答が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/72ef63124abc47e9bb5bb0be30f8aab8/github-mobile-copilot-knowledge-base-chat-in-repo.png?w=864\&h=1920\&auto=compress%2Cformat)

似たことは VS Code 用のチャット拡張機能で `@workspace` を使ってもできますが、準備が少し面倒です。ちょっとした質問だけならばアプリからリポジトリを開くだけで入力できるのは楽かと思います。

また、GitHub Mobile の会話履歴は同じアカウントでサインインしている他デバイスの GitHub Mobile でも参照できます。よって、「軽く調べておいてからタブレットに引き継ぐ」的なこともできるかと思います。

::: message

> 「軽く調べておいてからタブレットに引き継ぐ」的なこともできるかと思います。

逆に言うと、GitHub Mobile アプリが動作するデスクトップ環境を用意しないと、モバイル環境以外での共有はできないようです。 (Enterprise で使える [GitHub.com](http://GitHub.com) 上のチャットの履歴が共有されるかは使ってないのでわかりません)

提案されたソースコードなどをデスクトップ環境で利用したい場合は、他の情報共有サービス(とクリップボード)などを利用することになります。

:::

## ドキュメントのソースとしてのリポジトリ

[GitHub Copilot のキュメントではリポジトリに含まれるコードの内容について質問していますが](https://docs.github.com/ja/copilot/using-github-copilot/asking-github-copilot-questions-in-github-mobile#asking-exploratory-questions-about-a-repository) 、ドキュメントが主体のリポジトリ(静的なウェブページのソースなど)でも質問できます。

たとえば、Zenn のドキュメントもリポジトリになっているので試してみます。

@[card](https://github.com/zenn-dev/zenn-docs)

このリポジトリを GitHub Mobile で開いてチャットを開始すると Markdown 記法などについて質問できます。

**図 3-1 ざっくりした質問で返答が表示される(されないこともある)**

![「ノート(追記的な表示)はどのように記述しますか？」という質問に対して「\`::: message\` を使った記法」を返答しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a1fa6950cd384b26abd76e17ccd7fa9c/github-mobile-copilot-knowledge-base-qa-zenn-markdown.png?w=864\&h=1920\&auto=compress%2Cformat)

これについては、該当ページを Edge ブラウザーで開いて Edge 側の Copilot を使う方法もありますが、ページがフラットになっている必要があります。

たとえば、[Tailwind CSS のドキュメント](https://tailwindcss.com/)はセクションなどでページ(ファイル)が分割されています。このような場合もドキュメントのリポジトリが公開されていれば横断的に質問できます。

@[card](https://github.com/tailwindlabs/tailwindcss.com)

**図 3-2 ドキュメント全体に対して質問が適用される**

![「ボタンの定義方法についてのドキュメントは？」という質問に対して、スタイル定義の例が記述されたページの一覧が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e16f160d84904634b15c052ae4ef04c0/github-mobile-copilot-knowledge-base-qa-pages.png?w=864\&h=1920\&auto=compress%2Cformat)

こちらの場合も、従来の全文検索で対応できることも多いのですが、(私は英語が苦手なので)上記方法だと日本語で質問できるところに利点があると思っています。

::: message

「ソースコードのリポジトリ」と「ドキュメントページのリポジトリ」について少し。

ライブラリーのドキュメントページなどを参照すると GitHub アイコンからリポジトリへリンクされていることが多いと思います。

**Tailwind CSS ドキュメントページからリポジトリへリンクしているアイコン(右端)**

![各種ボタンリンクと並んで GitHub アイコンが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/731bd2b398144e01bf9a93e41c87844a/github-mobile-copilot-knowledge-base-repo-ink.png?w=493\&h=74\&auto=compress%2Cformat)

このようなリンクは多くの場合で「ソースコードのリポジトリ」へリンクされています。これは、さきほど試した手順でのリポジトリとは異なりますので、「ドキュメントページのリポジトリ」は改めて探す必要があります。 (「ソースコードのリポジトリ」にドキュメントが含まれていることもあります)

*   ソースコードのリポジトリ - <https://github.com/tailwindlabs/tailwindcss>
*   ドキュメントページのリポジトリ - <https://github.com/tailwindlabs/tailwindcss.com>

とくに決まった探し方はありませんが、「ソースコードのリポジトリ」のオーナー(組織)のページから探すと見つかりやすいかと思います。

:::

## 考慮点

リポジトリを開くだけで使えるのでお手軽でよいのですが、ちょっと不得意な点もあるので状況によってツール(サービス)を使い分けるとよいかと思います。 (2024 年 8 月の状況です)

### GitHub Copilot は英語での開発に特化したサービス(らしい)

ドキュメントが行けるならメモとかのリポジトリでも行けそうかな思ったのですが、開発から離れすぎた内容について日本語で質問すると上記のようなことを言われてしまいます。

### 複数のコンテキストを組みせるのは苦手

例えば「あるリポジトリを開いたまま、外部のドキュメントを参照する」的な方法は結構苦労します。

### マルチモーダルではないもよう

画像のファイルを開いた状態でチャットを開始し、画像について質問しても「自分で画像を見てください」的な返答になりました。

### ファイルツリー以外はあまり調べてくれないもよう

Issue などの画面でもチャットを開始できるのですが、質問や要約をお願いしても想定したような返答にはならなかったです(全く関係ない要約をすることもありました)。

また、Copilot アイコンが表示されない画面もあります。

## おわりに

GitHub Mobile で任意のリポジトリ(ドキュメント)を開くことにより、簡易的なナレッジベースとして利用してみました。

制限も多いのですが、GitHub Mobile と GitHub Copilot を利用できる環境であれば手軽に使えるのが良いなと思いました。

なお、もう少し込み入った使い方を探している場合は、NotebookLM との組み合わせた「[GitHub のリポジトリを 1 ファイルにまとめるとノートブックにしやすい](https://zenn.dev/hankei6km/articles/github-repo-as-notebooklm-source)」もあります。
