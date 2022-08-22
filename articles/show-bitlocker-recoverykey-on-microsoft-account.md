---
id: show-bitlocker-recoverykey-on-microsoft-account
title: Microsoft アカウントで管理しているBitLocker 回復キーが表示できない場合
emoji: 🗝️
type: idea
topics:
  - bitlocker
  - microsoft
published: true
---

Microsoft アカウントに保存していた BitLocker 回復キーが**本来表示される画面に表示されなくて**すったもんだした経験があります。

いま、下記のようなことになっているようなので、私の知っている表示方法を共有します。

@[card](https://forest.watch.impress.co.jp/docs/news/1433528.html)

## 前提

以下の手順などは、BitLocker の回復キーを Microsoft アカウントへ保存してある場合についてのものです。

**実際に保存してあるかの確認方法**については**わかりませんので質問されてもお答えできません**。

また、私はメーカーのサポート担当者などでもないので、「そんな奴の書いたことなど信用できるの？」という場合は無視していただくのが良いかと思います。

## 通常の手順

Microsoft アカウントでサインインします。なお、この後の操作では Autheticator などで認証の確認をされることがあります。設定しているスマートフォンの準備もしておいてください。

@[card](https://account.microsoft.com/)

ホーム画面が表示されたらデバイスの一覧から「デバイスの管理」をクリックします。

**図 2-1 デバイスの一覧**

![Microfsoft アカウントでデバイスの一覧を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/875866d80c484b039076221de91bccf1/show-bitlocker-recoverykey-on-microsoft-account-device-list.png?w=773\&h=323\&auto=compress%2Cformat)

デバイスの詳細画面に「BitLocker データ保護」が表示されていたら「回復キーの管理」をクリックします。

**図 2-2 デバイスの一覧**

![デバイス一覧の BitLocker データ保護画面のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e26f8f142f16438eb80a7067c7b7249e/show-bitlocker-recoverykey-on-microsoft-account-manage-keys.png?w=590\&h=275\&auto=compress%2Cformat)

BitLocker 回復キーの一覧が表示されます。

**図 2-3 BitLocker 回復キーの一覧**

![Bitlocker 回復キーの一覧のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/30ba01100d9543d2bb7dae6d9d913142/show-bitlocker-recoverykey-on-microsoft-account-key-list.png?w=1575\&h=209\&auto=compress%2Cformat)

## BitLocker の回復キーの管理が表示されないとき

通常は上記手順で表示されると思います。しかし「デバイスをセットアップしたときのメモに回復キーは Microsoft アカウントへ保存したと書いてある」のに「回復キーの管理」が表示されない、ということがありました。

そのときにサポートへ確認したところ、以下の URL を教えてもらいました[^support]。さきほど試してみましたが、現在でも使えるようです。

*   <https://onedrive.live.com/RecoveryKey>

Microsoft アカウントに保存してある確信があるけど、なぜか回復キーを表示できない場合は**ダメ元で**試してみるとよいかもしれません。

[^support]: 実際は「教えてもらいました」という感じではなく、「本来表示されるべきボタンが表示されない」と納得してもらうのに 1 時間くらいかかりました(恨み節)。

##
