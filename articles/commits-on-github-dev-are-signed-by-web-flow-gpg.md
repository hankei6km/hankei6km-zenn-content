---
id: commits-on-github-dev-are-signed-by-web-flow-gpg
title: github.dev 上のコミットは 4AEE18F83AFDEB23 さんが署名している
emoji: 🤖
type: tech
topics:
  - github
  - git
  - gpg
published: true
---

github.dev (または vscode.dev)でコミットを作成すると GitHub 上で Verified が付くようになっています。

**図 1 github.dev でコミットすると GitHub 上で Verified が付く**

![GitHub 上のコミット一覧でコミットに Verfied が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7f31262a735a4f61b0f91078c66d9c48/commits-on-github-dev-are-signed-by-web-flow-gpg-verified.png?w=770\&h=199\&auto=compress%2Cformat)

最初はあまり気にしていなかったのですが、落ち着いて考えてみると「誰が署名してるの？」と気になってきました。

## `4AEE18F83AFDEB23` さんが署名している

署名に使われた鍵の情報については、とくに捻りもなく Verified をクリックすれば表示されます。

**図 1-1 表示された情報**

![Verified をクリックして鍵の情報が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6dd386744f75473684ef081d94c27020/commits-on-github-dev-are-signed-by-web-flow-gpg-signed-by.png?w=313\&h=230\&auto=compress%2Cformat)

では、この `4AEE18F83AFDEB23` さんは誰なのかというと…。

@[card](https://security.stackexchange.com/questions/173493/who-owns-the-gpg-key-4aee18f83afdeb23-and-how-did-it-sign-a-commit-in-my-github)
@[card](https://docs.github.com/ja/authentication/managing-commit-signature-verification/about-commit-signature-verification)
@[card](https://github.com/web-flow)

> GitHub は GPG を自動的に使用して、Web インターフェイスで行ったコミットに署名します。 GitHub によって署名されたコミットは、確認済みの状態になります。 <https://github.com/web-flow.gpg> で入手可能な公開キーを使用して、署名をローカルで確認できます。 キーの完全な指紋は、`5DE3 E050 9C47 EA3C F04A 42D3 4AEE 18F8 3AFD EB23` です。

GitHub (のウェブ)上での操作により作成されたコミットなどに署名する鍵とのことです。他には Pull Request で Merge Commit を作るときでも使われているようです。

なお、調べてみた限りでは、署名させない設定などはなさそうでした。

## ローカルで確認してみる

上記の鍵に限った話でもないのですが、「GitHub で Verified になっているのはよいけど、ローカルで検証ってどうやるの？」と思っていたので試しに署名を確認してみます。

確認は下記のリポジトリを clone して行います(冒頭のスクリーンショットに使ったリポジトリです)。また、GPG の `homedir` は一時的に作ったものを利用しています。

@[card](https://github.com/hankei6km/h6-dev-containers)

まず、準備なしで `$ git log --show-signature` を実行してみます。

**図 2-1 公開鍵がないので検査(確認)できない**

![コミットログの表示に赤く「検査できない」と表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/728d6b55c4c044c1a701d78d541143be/commits-on-github-dev-are-signed-by-web-flow-gpg-failed.png?w=652\&h=353\&auto=compress%2Cformat)

対応のために公開鍵を追加してみます。

といきたいところですが、いきなり追加するのもアレなのでいちおう事前に確認してみます。今回は下記の記事で紹介されている `gpgpdump` を利用してみます。

@[card](https://zenn.dev/spiegel/articles/20201014-openpgp-pubkey-in-github)

**図 2-2 鍵の内容と指紋を確認**

```shell-session
$ ./gpgpdump github web-flow --keyid 4AEE18F83AFDEB23 -u
Public-Key Packet (tag 6) (269 bytes)
        Version: 4 (current)
        Public key creation time: 2017-08-16T15:44:01Z
        Public-key Algorithm: RSA (Encrypt or Sign) (pub 1)
        RSA public modulus n (2048 bits)
        RSA public encryption exponent e (17 bits)
User ID Packet (tag 13) (53 bytes)
        User ID: GitHub (web-flow commit signing) <noreply@github.com>
Signature Packet (tag 2) (290 bytes)
        Version: 4 (current)
        Signiture Type: Positive certification of a User ID and Public-Key packet (0x13)
        Public-key Algorithm: RSA (Encrypt or Sign) (pub 1)
        Hash Algorithm: SHA2-256 (hash 8)
        Hashed Subpacket (22 bytes)
                Signature Creation Time (sub 2): 2017-08-16T15:44:01Z
                Issuer (sub 16): 0x4aee18f83afdeb23
                Key Flags (sub 27) (1 bytes)
                        Flag: This key may be used to certify other keys.
                        Flag: This key may be used to sign data.
                Primary User ID (sub 25): Primary (1 bytes)
        Hash left 2 bytes
                99 01
        RSA signature value m^d mod n (2048 bits)

$ ./gpgpdump github web-flow --keyid 4AEE18F83AFDEB23 --raw | gpg --show-keys
pub   rsa2048 2017-08-16 [SC]
      5DE3E0509C47EA3CF04A42D34AEE18F83AFDEB23
uid                      GitHub (web-flow commit signing) <noreply@github.com>
```

確認できたのでインポートと署名をしておきます。

**図 2-3 インポートと署名**

```shell-session
$ gpg --fetch-keys https://github.com/web-flow.gpg
gpg: 鍵を'https://github.com/web-flow.gpg'から要求
gpg: 鍵4AEE18F83AFDEB23: 公開鍵"GitHub (web-flow commit signing) <noreply@github.com>"をインポートしました
gpg:           処理数の合計: 1
gpg:             インポート: 1

$ gpg --edit-key 4AEE18F83AFDEB23 trust quit
gpg (GnuPG) 2.2.40; Copyright (C) 2022 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg: 信用データベースの検査
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: 深さ: 0  有効性:   1  署名:   0  信用: 0-, 0q, 0n, 0m, 0f, 1u
gpg: 次回の信用データベース検査は、2024-12-15です
pub  rsa2048/4AEE18F83AFDEB23
     作成: 2017-08-16  有効期限: 無期限      利用法: SC
     信用: 充分        有効性: 不明の
[  不明  ] (1). GitHub (web-flow commit signing) <noreply@github.com>

pub  rsa2048/4AEE18F83AFDEB23
     作成: 2017-08-16  有効期限: 無期限      利用法: SC
     信用: 充分        有効性: 不明の
[  不明  ] (1). GitHub (web-flow commit signing) <noreply@github.com>

他のユーザの鍵を正しく検証するために、このユーザの信用度を決めてください
(パスポートを見せてもらったり、他から得たフィンガープリントを検査したり、などなど)

  1 = 知らない、または何とも言えない
  2 = 信用し ない
  3 = まぁまぁ信用する
  4 = 充分に信用する
  5 = 究極的に信用する
  m = メーン・メニューに戻る

あなたの決定は? 4

pub  rsa2048/4AEE18F83AFDEB23
     作成: 2017-08-16  有効期限: 無期限      利用法: SC
     信用: 充分        有効性: 不明の
[  不明  ] (1). GitHub (web-flow commit signing) <noreply@github.com>

$ gpg --lsign 4AEE18F83AFDEB23

pub  rsa2048/4AEE18F83AFDEB23
     作成: 2017-08-16  有効期限: 無期限      利用法: SC
     信用: 充分        有効性: 不明の
[  不明  ] (1). GitHub (web-flow commit signing) <noreply@github.com>

pub  rsa2048/4AEE18F83AFDEB23
     作成: 2017-08-16  有効期限: 無期限      利用法: SC
     信用: 充分        有効性: 不明の
  主鍵フィンガープリント: 5DE3 E050 9C47 EA3C F04A  42D3 4AEE 18F8 3AFD EB23

     GitHub (web-flow commit signing) <noreply@github.com>

本当にこの鍵にあなたの鍵"hankei6km <gh-hankei6km@outlook.jp>"で署名してよいですか
(B28DC515982DFE08)

署名は、エクスポート不可に設定されます。

本当に署名しますか? (y/N) y
```

準備できたので再度 `$ git log --show-signature` を実行してみます。

**図 2-4 署名を確認できる**

![コミットログの表示に青文字で署名の情報が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7e85f84321b644aab53570a9dc6a101e/commits-on-github-dev-are-signed-by-web-flow-gpg-ok.png?w=652\&h=349\&auto=compress%2Cformat)

GitHub 上から鍵をインポートすることで署名を確認できました。ただし、あくまでも `web-flow` が署名したと確認できるだけなので、GitHub 上で Verified 扱いになるかはぱっと見で判定が難しいような気もしています。

## Codespaces の場合

Codespaces の場合は設定から `web-flow` による署名の有無を切り替えることができます(デフォルトでは無しです)

@[card](https://docs.github.com/ja/codespaces/managing-your-codespaces/managing-gpg-verification-for-github-codespaces)

試してみると専用の署名用プログラム(だと思われる)を Git の設定に指定しているので、ターミナルでの操作でも署名されるようになっています。ただし、`$ git log --show-signature` で確認すると公開鍵がないので検査できない状態になります。

## おわりに

github.dev で作成したコミットの署名について、誰が署名しているのか確認してみました。

とくに設定しなくとも Verified になるのは便利なような気もしますが、GitHub の外で使う(確認する)となると少し面倒なこともありそうです。

他にも[警戒モード](https://docs.github.com/ja/authentication/managing-commit-signature-verification/displaying-verification-statuses-for-all-of-your-commits)など GitHub での枠組みがあるようなので、必要に応じて調べていこうかと思っています。
