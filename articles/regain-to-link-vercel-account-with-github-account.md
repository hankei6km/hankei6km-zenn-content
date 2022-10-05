---
id: regain-to-link-vercel-account-with-github-account
title: Vercel へ GitHub account でログインできない場合
emoji: 🔗
type: idea
topics:
  - vercel
  - github
published: true
---

先日、しばらくぶりに Vercel へログインしようとしたところ下記のようなエラーが表示されるようになっていました。

> There is no Vercel account linked with this GitHub account.

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b282de91337749b88f70365104ef4b0a/regain-to-link-vercel-account-with-github-account-failed.png?auto=compress%2Cformat)

その対応と原因についてなど。

## 問い合わせ

エラーメッセージでネットを検索したり、少し時間をおいて試したりしたのですが解決しないのでサポートへメールを送ることにしました。

内容としては以下のような項目を記載して「Vercel account 自体は有効だと思われます、再度ログインする方法はありますか」という感じにしました。

*   GitHub account でログインしようとするとエラーになる(エラーメッセージと GitHub account 名も記載する)
*   Vercel CLI ではログインしていてこちらでは現在も操作できる(日付付きで操作しているログもコピー)
*   Vercel からのニュースメールをいまでも受け取っている(受け取っているメールアドレスも記載)

サポートへのメールアドレスは 2022-10-04 時点では下記のページで確認できました。

@[card](https://vercel.com/guides/how-to-get-vercel-support)

メールを送信すると「あなたのプランだと返答は 3 ～ 4 日、混んでるともう少しかかるかも」的な自動応答メールがきました。が、翌朝には返信が届いていました(ちょっとうれしい)。

## 対応

対応としては「GitHub account とのリンクが切れているように見えるのでメールアドレス(お知らせメールを受け取っているアドレス)でログインしてほしい」とのこと。

ここで「メールアドレスでログインってパスワードとかいるのでは？」となりますが、とりあえずログインしてみます。

ログインしようとすると以下のような画面が表示され、該当メールアドレス宛に Verify リンク付きのメールアドレスが送られてきます。

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/80a33477f17d41ae9a5bf017abf794c4/regain-to-link-vercel-account-with-github-account-verify.png?auto=compress%2Cformat)

表示されているコードが一致されていることを確認し Verify することでログインできました。

## 原因

GitHub account とのリンクが切れる原因としては「GitHub account の Primaly メールアドレスを変更した」場合が多いとのこと。

そして、おっしゃる通りで GitHub account の Primaly メールアドレスを変更していました(原因も判明してスッキリです)。

## おわりに

Vercel へ GitHub account でログインできない場合について書いてみました。

「再ログインできるまで時間かかるな？」と予想していたのですが、サポートからの的確な指示であっさり解決しました(ありがとうサポートの人)。

GitHub account の Primary メールアドレスを変更することはあまりないとはおもぃますが、同じようになってしまった場合は「メールアドレスでログイン」を試してみるとよいかもしれません。
