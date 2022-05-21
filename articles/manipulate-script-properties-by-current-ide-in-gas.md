---
id: manipulate-script-properties-by-current-ide-in-gas
title: Google Apps Script の現行エディターでスクリプトプロパティを編集できるようになっていた
emoji: 😊
type: tech
topics:
  - googleappsscript
published: true
---

遅まきながら先ほど気が付いたのですが…。

ついに「クラシックエディターを使用する」ボタンをクリックする作業をやめてもよい時がきたようです。

## どこで編集できる？

現行のエディターの場合、「エディター」タブではなく「プロジェクトの設定」タブに編集用の UI が追加されていました。

▼ **図 1-1 スクリプトプロパティの UI が追加されている**

![スクリプトエディターで「プロジェクトの設定」タブを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ea87f6267190467c893ff5c9087586df/manipulate-script-properties-by-current-ide-in-gas-project-settings.png?w=891\&h=568\&auto=compress%2Cformat)

## いつから使えるようになっていたのか

検索してみたところ以下のような感じでした。4 月中旬にリリースでアカウントの種類などで時差がある感じでしょうか。

> Today, I confirmed that Script Properties got to be able to be managed with a gmail account. So, I posted this.

@[card](https://stackoverflow.com/a/72008088)

@[card](https://developers.google.com/apps-script/releases/#april_13_2022)

## おわりに

リリース時期からするといまさら感が強いですが、ちょっとうれしい機能追加だったので記事にしてしまいました[^updated-values]。

[^updated-values]: スクリプトプロパティに動的な値をセットしていると「いまどんな値になっているのか？」の確認が地味に面倒でした。
