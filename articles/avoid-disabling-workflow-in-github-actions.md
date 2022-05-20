---
id: avoid-disabling-workflow-in-github-actions
title: GitHub Actions ワークフローが 60 日後に停止されるのを手っ取り早く回避する
emoji: 🗓️
type: tech
topics:
  - githubactions
published: true
---

GitHub Action から「このワークフローはもうすぐ無効化されるよ」的なメールが届きました。

> The "append-04-13" workflow in hankei6km/gha-now-to-sheet will be disabled soon

[実験的なリポジトリ](https://github.com/hankei6km/gha-now-to-sheet)のワークフローですが無効化されるとちょっと悲しいので、なるべく手間をかけない回避方法を試してみました。

## ワークフローが無効化されるとは

「スケジュール実行」を設定しているワークフローはリポジトリに 60 日間アクティビティ(コミットなど)がないと無効化されるということらしいです。

@[card](https://github.community/t/no-notification-workflow-disabled-after-60-days/182169/)
@[card](https://zenn.dev/hellorusk/articles/fc6d4696f5b269)
@[card](https://qiita.com/tommy_aka_jps/items/65193185b520a1e2be21)

## 回避方法

アクティビティがあれば無効化されない([別の記事](https://zenn.dev/hankei6km/articles/automatically-update-github-profile)で作ったリポジトリは無効化されていない)ので bump 的なコミットなどを行えばよいことになりますが少し面倒です。

今回、対象となるワークフローが複数あったので少し試してみたところ、「ワークフローを手動で無効化し再度有効化」することで回避できることがわかりました。

▼ **図 2-1 GitHub CLI による手順例**

```shell-session
# Workflow ID を確認
$ gh workflow list
append-00-13  active  21631109
append-00-43  active  21631110
append-01-13  active  21631111
append-01-43  active  21631112

# 21631109 の無効化と有効化
$ gh workflow disable 21631109
$ gh workflow enable 21631109
```

これで対象のワークフロー(上記の場合 `21631109`)の定義される期間が延長されます。

ただし、コミットを作成する場合にと比べて以下のような注意点があります。

*   対象以外のワークフローは延長されない
*   無効化される日時がわからない(コミットでもわからないですが、コミットログからあたりをつけられる)[^workflow-status]
*   (おそらく)ワークフロー内で実行しようとするとそれなりの権限が必要

[^workflow-status]: ワークフローが「有効化された、または無効化される日時」の取得方法は不明。[API を使って取得したワークフローの情報](https://docs.github.com/ja/rest/actions/workflows#get-a-workflow)には含まれていませんでした。

## 実際に無効化されたときは通知されない

実は、回避をしそこねたワークフローが 1 つありました。ですが、予告のときと違って実際に無効化されたときはメールが送信されませんでした。

再有効化だと実際に延長されたかの確認が難しいので、無効化されそうな時期には実行状況を確認しておいた方がよさそうです。

## おわりに

GitHub Actions でワークフローが無効化されることを簡単なコマンドで回避してみました。

本来ならば延長処理は自動化しておきたいところですが、少し考えた感じでは「わりと面倒そう」なのでその辺はいずれということで[^zubora]。

[^zubora]: こういう場合は、ズルズルと手動でコマンド実行するパターンになりそうですが。
