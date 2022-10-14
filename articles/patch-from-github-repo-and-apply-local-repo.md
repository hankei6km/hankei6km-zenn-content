---
id: patch-from-github-repo-and-apply-local-repo
title: GitHub 上の別のリポジトリーから cherry-pick ぽいことをしてみる
emoji: 🧲
type: tech
topics:
  - github
  - git
published: true
---

少し前に [rollup.js のバージョンアップ対応](https://zenn.dev/hankei6km/articles/rollup-config-js-as-esm)をしたのですが「同一テンプレートから作った複数のリポジトリ」が対象になってしまいました。

**図 1 手動対応が必要な Bump さんの団体**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a6f636b1f68c49b484f867113defdc82/patch-from-github-repo-and-apply-local-repo-bump.png?auto=compress%2Cformat)

以前からこのようなとき「GitHub の別のリポジトリからも [`cherry-pick`] できるとよいのだけど」と思っていたのですが、「パッチ取得して適用するだけでは」と気が付きました。

ということで試してみます。

## GitHub 上にあるコミット

今回は以下のコミットを他のリポジトリへ適用してみます。

**図 1-1 適用したいコミット**

![適用したいコミットをGitHub 上で表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c72d119f40c9423e8767ee4ec3f937e4/patch-from-github-repo-and-apply-local-repo-commit.png?w=1418\&h=668\&auto=compress%2Cformat)

*   <https://github.com/hankei6km/gas-gchanges2notion/commit/8f693b7e4bd65d325c5af14a5737affa070c6a81>

## GitHub のリポジトリからパッチを取得

これは検索するとすぐに出てきました。

> Adding `.patch` (or `.diff`) to the commit-URL will give a nice patch:
>
>     https://github.com/foo/bar/commit/${SHA}.patch

@[card](https://stackoverflow.com/questions/21903805/how-to-download-a-single-commit-diff-from-github)

試してみると、`.patch` と `diff` では取得される情報が異なっていて、`.patch` を付加すると [`format-patch` ] と同様の結果を取得できるようです。

**図 2-1 format-patch と同様の結果が取得される**

```shell-session
$ curl -sL https://github.com/hankei6km/gas-gchanges2notion/commit/8f693b7e4bd65d325c5af14a5737affa070c6a81.patch

From 8f693b7e4bd65d325c5af14a5737affa070c6a81 Mon Sep 17 00:00:00 2001
From: hankei6km <gh-hankei6km@outlook.jp>
Date: Thu, 13 Oct 2022 05:49:18 +0000
Subject: [PATCH] chore: Use `import.meta` instead of `__dirname`

It prepare to use rollup.js v3.x.
---
 rollup.config.js | 7 ++++++-
 1 file changed, 6 insertions(+), 1 deletion(-)

diff --git a/rollup.config.js b/rollup.config.js
index 11ddd21..1c35230 100644
--- a/rollup.config.js
+++ b/rollup.config.js
@@ -4,6 +4,7 @@ import { nodeResolve } from '@rollup/plugin-node-resolve'
 import json from '@rollup/plugin-json'
 import license from 'rollup-plugin-license'
 import * as path from 'path'
+import { fileURLToPath } from 'node:url'
 
 const extensions = ['.ts', '.js']
 
@@ -41,7 +42,11 @@ export default {
       thirdParty: {
         // includePrivate: true, // Default is false.
         output: {
-          file: path.join(__dirname, 'build', 'OPEN_SOURCE_LICENSES.txt'),
+          file: path.join(
+            path.dirname(fileURLToPath(import.meta.url)),
+            'build',
+            'OPEN_SOURCE_LICENSES.txt'
+          ),
           encoding: 'utf-8' // Default is utf-8.
         }
       }
```

## ローカルのリポジトリへ適用

Git のリポジトリにパッチを適用するには [`git apply` ] と [`git am` ] があります。

@[card](https://git-scm.com/book/ja/v2/Git-%E3%81%A7%E3%81%AE%E5%88%86%E6%95%A3%E4%BD%9C%E6%A5%AD-%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E9%81%8B%E5%96%B6)

[`git am` ] で [`format-patch` ] 形式のファイルを使うとファイルに含まれるコミットメッセージや Author が利用されたコミットが作成されます。

今回はコミットも作成したいので、[`git am` ] を利用します。

**図 3-1 パッチをパイプ経由で git am へ渡して適用**

```shell-session
$ curl -sL https://github.com/hankei6km/gas-gchanges2notion/commit/8f693b7e4bd65d325c5af14a5737affa070c6a81.patch | git am                                                                                         
Applying: chore: Use `import.meta` instead of `__dirname`
```

適用されると下記のようにコミットが作成されます。`Author:` の他に `Date:` などもパッチの値が利用されるので、変更したい場合は [`--ignore-date`](https://git-scm.com/docs/git-am#Documentation/git-am.txt---ignore-date) などを利用します。

**図 3-2 作成されたコミット(Date などはパッチの値が利用されている)**

```shell-session
$ git log -n 1
commit 3f7c0504995e88edb3ed88e8db376b4d302f7c02 (HEAD -> topic/rollup-config-js-as-esm)
Author: hankei6km <gh-hankei6km@outlook.jp>
Date:   Thu Oct 13 05:49:18 2022 +0000

    chore: Use `import.meta` instead of `__dirname`
    
    It prepare to use rollup.js v3.x.
```

あとはテストして調整したいところがあれば普通に `--amend` などが使えます。

## コンフリクトで適用できない場合

別のリポジトリーからパッチを持ってきているので、適用できない場合もあります。

今回もやはり適用できないリポジトリがありました。

**図 4-1 コンフリクトで適用できない場合**

```shell-session
$ curl -sL https://github.com/hankei6km/gas-gchanges2notion/commit/8f693b7e4bd65d325c5af14a5737affa070c6a81.patch | git am                                                                                                   
Applying: chore: Use `import.meta` instead of `__dirname`                                                       
error: patch failed: rollup.config.js:4
error: rollup.config.js: patch does not apply                                                                   
Patch failed at 0001 chore: Use `import.meta` instead of `__dirname`
hint: Use 'git am --show-current-patch' to see the failed patch                                                 
When you have resolved this problem, run "git am --continue".                                                   
If you prefer to skip this patch, run "git am --skip" instead.                                                  
To restore the original branch and stop patching, run "git am --abort".
```

調べてみると今回はパッチの周辺行(コンテキスト)が期待されていたものと異なっているためにコンフリクトしていました。

**リスト 4-1 パッチが期待している状態**

    import { nodeResolve } from '@rollup/plugin-node-resolve'
    import json from '@rollup/plugin-json'
    import license from 'rollup-plugin-license'
    import * as path from 'path'
                                     <---- ここに行を追加したい
    const extensions = ['.ts', '.js']

**リスト 4-2 実際の状態**

    import { nodeResolve } from '@rollup/plugin-node-resolve'
    import license from 'rollup-plugin-license'
    import * as path from 'path'
                                     <---- ここに行を追加したい
    const extensions = ['.ts', '.js']

よく見るとパッチを適用したい箇所の 3 行上の内容が異なっています。

対応方法はいくつかありますが、この場合であれば abort させた後、改めて [`-C`](https://git-scm.com/docs/git-apply#Documentation/git-apply.txt--Cltngt) オプション(周辺行の範囲を変更)を利用することで対応できます。

**図 4-2 -C2 でコンフリクトを回避**

```shell-session
# コンフリク状態を abort させる
$ git am --abort
fatal: Resolve operation not in progress, we are not resuming.

# コンテキストの範囲を狭めて適用
$ curl -sL https://github.com/hankei6km/gas-gchanges2notion/commit/8f693b7e4bd65d325c5af14a5737affa070c6a81.patch | git am -C2
Applying: chore: Use `import.meta` instead of `__dirname`
Context reduced to (2/2) to apply fragment at 4
```

`--abort` させない場合は下記の記事などを参照してください。

@[card](https://qiita.com/maueki/items/c8476908f8a8601365a5)

## おわりに

GitHub の別リポジトリからパッチを取得してローカルに適用してみました。

Codespace を使うと「となりのリポジトリ」との距離が遠くなるのでで[^cp]、変更点をちょこっと移植するのが少し手間だなと思っていました。

これからは GitHub を経由することで手間を減らせるかなとちょっと期待しています。

[^cp]: 気軽に親ディレクトリー経由で `cp` やファイルの参照などができなくなるという意味で。

[`cherry-pick`]: https://git-scm.com/docs/git-cherry-pick

[`format-patch`]: https://git-scm.com/docs/git-format-patch

[`git apply`]: https://git-scm.com/docs/git-apply

[`git am`]: https://git-scm.com/docs/git-am
