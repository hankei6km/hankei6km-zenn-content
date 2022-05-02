---
id: timeout-in-github-actions
title: GitHub Actions でワークフローの実行時間≒ジョブ実行時間の合計にならなかった話
emoji: ⏳
type: tech
topics:
  - githubactions
published: true
---

「[GitHub Actions のスケジュール実行による待機時間を Google Data Portal で可視化した](https://zenn.dev/hankei6km/articles/visualization-of-waiting-time-in-github-actions)」で実行しているワークフローは 30 分毎にスケジュール実行しています。そのためサービスの障害の影響を受けることが時々あります。その中で少し興味深い現象があったのでメモなど。

## ジョブは 1 つでタイムアウトは 10 分

上記で実行している各ワークフローのジョブは 1 つだけで、ジョブには 10 分のタイムアウトを設定しています。

```yaml
jobs:
  append:
    environment: append

    permissions:
      contents: 'read'
      id-token: 'write'

    runs-on: ubuntu-latest
    timeout-minutes: 10
```

## しかし、ワークフローの実行時間は 20 分を超えることも

GitHub で障害発生していた近辺の出来事になりますが、ワークフロー開始から終了までの時間が 20 分を超えることがありました。

@[card](https://github.com/hankei6km/gha-now-to-sheet/actions/runs/2001834047)

調べてみると、「ジョブは数秒で完了しているがワークフローの実行時間が長い」ということがわかりました。

▼ **図 2-1 ワークフローとジョブの実行時間**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/25dd437137df4d109c9e2ffd4d108bd9/visualization-of-waiting-time-in-github-actions-timeout-pick.png?auto=compress%2Cformat)

さらにログを確認してみると「runner がジョブをピックアップするのを待っている時間」が長いという状況です。

▼ **図 2-2 runner がオンラインになるのを待っている時間**

    2022-03-18T01:50:37.2662072Z Requested labels: ubuntu-latest
    2022-03-18T01:50:37.2662156Z Job defined at: hankei6km/gha-now-to-sheet/.github/workflows/append-00-13.yaml@refs/heads/main
    2022-03-18T01:50:37.2662198Z Waiting for a runner to pick up this job...
    2022-03-18T01:50:37.5434531Z Job is waiting for a hosted runner to come online.
    2022-03-18T02:11:48.4331373Z Job is about to start running on the hosted runner: Hosted Agent (hosted)
    2022-03-18T02:11:50.8795737Z Current runner version: '2.288.1'

オーバーヘッドがあるので「ジョブの実行時間 = ワークフローの実行時間ではない」ということは頭では理解していましたが、ここまで違うことがあるのは少し意外でした。

## おわりに

上記のことはイレギュラーなこととは思われますが、普段まったく発生しないとも言い切れません。障害が発生していないときでも 5 分近くピックアップ待ちになることもありました[^inventry]。

@[card](https://github.com/hankei6km/gha-now-to-sheet/actions/runs/2238089573)

「ジョブにタイムアウトを設定しているからワークフローの実行時間は一定の範囲に収まる」という前提でフローを組んでいると思わぬ落とし穴があるやもしれません。

[^inventry]: 実際の状況はわかりませんが、[GitHub Status - Incident History](https://www.githubstatus.com/history) には記録がない状態です。
