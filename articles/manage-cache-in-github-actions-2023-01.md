---
id: manage-cache-in-github-actions-2023-01
title: GitHub Actions の Cache 管理(2023 年 1 月の場合)
emoji: 🎚️
type: tech
topics:
  - githubactions
  - githubcli
published: true
---

以前にも同じ内容で記事を書いたのですが、GitHub 上のウェブ UI でキャッシュを管理できるようになったり、GitHub CLI の extension も機能アップしたりと変化がありました。

@[card](https://zenn.dev/hankei6km/articles/manage-cache-in-github-actions)

今回は以前の記事に加えて、下記の点について書いてみたいと思います。

*   **お手軽** - ウェブ UI による一覧表示と削除
*   **自動化** - [`gh-actions-cache`] とワークフローによる管理の自動化

[`gh-actions-cache`]: https://github.com/actions/gh-actions-cache

## キャッシュをブラウザーからお手軽に管理

以前の記事では API と GitHub CLI の extension([`gh-actions-cache`]) を使う方法を書きました。2023 年 1 月現在ではウェブ UI でキャッシュを管理する方法も実装されているので、今回はこちらについて書いてみます。

@[card](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#managing-caches)

ブラウザーで GitHub 上のリポジトリから Action をタブを開くと、左側のパネルに `Management / Caches` が表示されます。

**図 1-1 左側パネルの一覧に Cache が表示される**

![リポジトリの Actions タブを開き、左側に Caches  項目が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a1fb3166614c44dab2cc1894923fdb43/manage-cache-in-github-actions-2023-01-webui-menu.png?w=991\&h=417\&auto=compress%2Cformat)

クリックするとキャッシュの一覧が表示されます。一覧の表示は GitHub にサインしていない状態でも確認できるようです(アイコン表示などは少し異なります)。

**図 1-2 キャッシュ一覧**

![キャッシュ一覧が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8aa2f18248fc42f6ae5ae737151943e1/manage-cache-in-github-actions-2023-01-webui-list-caches.png?w=1237\&h=547\&auto=compress%2Cformat)

並び順の変更や、キーやブランチでのフィルタリングもできます。ただし、フィルタリングは先頭がマッチしていればヒットしますが、部分一致には対応していないようです。下記の例だと `Windows` はヒットしますが、`install` ではヒットしません。

**図 1-3 フィルターの利用**

![キャッシュ一覧の右上にあるフィルター用フィールドを利用しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/65374a8f1c3d4521a9fea6bd36e8a7f4/manage-cache-in-github-actions-202-013-webui-filter.png?w=1228\&h=495\&auto=compress%2Cformat)

(権限がある場合は)キャッシュの横にあるゴミ箱アイコンをクリックすることで個別に削除できます。ただし、ブランチまとめての一括削除などはないようです。

**図 1-4 ゴミ箱アイコンクリックした後の確認**

![ゴミ箱アイコンをクリックし後に表示される確認ダイアログのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/82682b1245cc44ae81cc9184f39cbb04/manage-cache-in-github-actions-2023-01-webui-delete.png?w=704\&h=220\&auto=compress%2Cformat)

このような感じでブラウザーからお手軽にキャッシュを管理できようになっています。事前の設定やローカルでコマンドを動かす必要はないので github.dev(あるいは vscode.dev)のようなブラウザーベースでのコーディング環境を利用している場合でも使いやすいかと思います。

## `gh-actions-cache` は自動化も対応しやすくなった

ワークフローによる自動化の前に少し補足。

GitHub CLI extension の [`gh-actions-cache`] は、パイプを利用すると一覧がタブ区切りで出力されるようになりました[^tty]。これにより、人間が確認しやすいように表示するだけでなく、コマンドを連携させた自動化も対応しやすくなりました。

[^tty]: 実際には端末でなければタブ区切りになると思います。

**図 2-1 パイプを利用するタブ区切りで出力される**

```shell-session
$ gh actions-cache list --limit 10 --key "Windows"
Showing 2 of 2 cache entries in hankei6km/xquo

Windows-install-f55c7f933...  171.66 MB  refs/heads/main              a day ago
Windows-install-f55c7f933...  171.60 MB  refs/heads/dependabot/ca...  4 days ago

$ gh actions-cache list --limit 10 --key "Windows" | cut -f 1
Windows-install-f55c7f9330655b32c91858cedc1df6af68fb939379268f75dd3209a48b5b776a
Windows-install-f55c7f9330655b32c91858cedc1df6af68fb939379268f75dd3209a48b5b776a
```

## ワークフローによる管理の自動化(は、まだ試行錯誤が必要かも)

詳しいことは下記のドキュメントに丸投げしてしまいますが、たとえばプルリクエストが増えるとキャッシュ用ストレージの割り当てを圧迫してしまうこともあります。

@[card](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#force-deleting-cache-entries)

> For example, a repository could have many new pull requests opened, each with their own caches that are restricted to that branch. These caches could take up the majority of the cache storage for that repository.

これに対して、(GitHub CLI で自動化しやすくなった影響もあってか)ワークフローで解決していきましょうという方向になっているようです。

> You can use the [`gh-actions-cache`] CLI extension to delete caches for specific branches.

ただし、この辺はまだドキュメントが固まっていないようで、上記ドキュメントのサンプルもブランチの指定について issue ができています。

@[card](https://github.com/github/docs/issues/22727)

このような感じで当面は試行錯誤が必要な感じですが、うまく対応できるとキャッシュの増加を抑制できます。

ちなみに、上記サンプルは(手元のリポジトリでは)permission の指定とブランチの指定を調整することで利用できました。せっかくなので少し試してみます。

https://github.com/hankei6km/test-cleanup-caches-by-branch/blob/main/.github/workflows/cleanup.yml

下記は、[Dependabot を設定しているリポジトリ](https://github.com/hankei6km/notion2content)のキャッシュ一覧です(このリポジトリでは PR を `push` イベントで扱っています[^event])。先頭の 2 つに対応する PR はオープンされていますが、それより後ろの PR はすでにマージしてあります。

[^event]: `pull_request` イベントを使っている場合は `ref` の表示が異なります。PR とイベントの関係などについては「[PR に紐付けたいワークフローは push ではなく pull\_request イベントを使おう \[GitHub Actions\]](https://zenn.dev/snowcait/articles/dc1851e421e600)」が参考になります。(`actions/checkout@v3` で `ref` を指定してもキャッシュには反映されない感じもするので、その辺は試行錯誤が必要そうです)

**図 3-1 プルリクエストをマージしてもキャッシュは残る**

```shell-session
$ gh actions-cache list
Total caches size 6.65 GB

Showing 12 of 12 cache entries in hankei6km/notion2content

Linux-build-cache-node-modules-f6ab3c2...  619.99 MB  refs/heads/dependabot/npm_and_yarn/ty...  7 hours ago
Linux-build-cache-node-modules-28fecdb...  619.98 MB  refs/heads/dependabot/npm_and_yarn/ty...  7 hours ago
Linux-build-cache-node-modules-9a88c8e...  619.93 MB  refs/heads/main                           8 hours ago
Linux-build-cache-node-modules-e617a31...  619.91 MB  refs/heads/main                           3 days ago
Linux-build-cache-node-modules-9a88c8e...  619.93 MB  refs/heads/dependabot/npm_and_yarn/ri...  3 days ago
Linux-build-cache-node-modules-497d7f3...  494.84 MB  refs/heads/main                           4 days ago
Linux-build-cache-node-modules-e617a31...  619.91 MB  refs/heads/dependabot/npm_and_yarn/sw...  4 days ago
Linux-build-cache-node-modules-9238b4c...  619.93 MB  refs/heads/dependabot/npm_and_yarn/sw...  4 days ago
Linux-build-cache-node-modules-26b9381...  494.81 MB  refs/heads/main                           4 days ago
Linux-build-cache-node-modules-497d7f3...  494.84 MB  refs/heads/dependabot/npm_and_yarn/ri...  4 days ago
Linux-build-cache-node-modules-bac97ea...  494.79 MB  refs/heads/main                           5 days ago
Linux-build-cache-node-modules-26b9381...  494.81 MB  refs/heads/dependabot/npm_and_yarn/ri...  5 days ago
```

とくに対策していない場合は、このようマージに後もキャッシュはそのまま残っています(そして、このキャッシュが再利用されることはおそらく無いです)。また、Dependabot の bump による複数の PR はその性質上、rebase でキャッシュが増える傾向にあります。 (4 days ago の `sw...` が 2 つあるのは rebase されたためです)

現在、このリポジトリには上記ワークフローを試しに設定してあるので、マージ(クローズ)するとブランチのキャッシュは削除されるようになっています。

**図 3-2 PR がマージされるとキャッシュが削除される**

```shell-session
$ gh actions-cache list
Total caches size 7.26 GB

Showing 12 of 12 cache entries in hankei6km/notion2content

Linux-build-cache-node-modules-53b58...  619.99 MB  refs/heads/main                          a minute ago
Linux-build-cache-node-modules-28fec...  619.98 MB  refs/heads/main                          2 minutes ago
Linux-build-cache-node-modules-9a88c...  619.93 MB  refs/heads/main                          13 minutes ago
Linux-build-cache-node-modules-e617a...  619.91 MB  refs/heads/main                          3 days ago
Linux-build-cache-node-modules-9a88c...  619.93 MB  refs/heads/dependabot/npm_and_yarn/r...  3 days ago
Linux-build-cache-node-modules-497d7...  494.84 MB  refs/heads/main                          4 days ago
Linux-build-cache-node-modules-e617a...  619.91 MB  refs/heads/dependabot/npm_and_yarn/s...  4 days ago
Linux-build-cache-node-modules-9238b...  619.93 MB  refs/heads/dependabot/npm_and_yarn/s...  4 days ago
Linux-build-cache-node-modules-26b93...  494.81 MB  refs/heads/main                          4 days ago
Linux-build-cache-node-modules-497d7...  494.84 MB  refs/heads/dependabot/npm_and_yarn/r...  4 days ago
Linux-build-cache-node-modules-bac97...  494.79 MB  refs/heads/main                          5 days ago
Linux-build-cache-node-modules-26b93...  494.81 MB  refs/heads/dependabot/npm_and_yarn/r...  5 days ago
```

デフォルトブランチのキャッシュが積みあがっていくのでトータルのサイズは減らせませんが、PR ブランチのキャッシュは削除されるので増分を抑えるとはできました。

## おわりに

GitHub Actions のキャッシュ管理について再度ざっと確認してみましたが、2023 年 1 月現在の状況としては下記のような感じになるかと思います。

*   ウェブ UI 上でお手軽に確認と削除ができるようになっている
*   自動化のための環境や資料も充実していきそう

個人的には、Dev Container 用イメージをプレビルドするリポジトリでキャッシュがごちゃごちゃになっているので、この辺の整理に活用できないかなと考えています。
