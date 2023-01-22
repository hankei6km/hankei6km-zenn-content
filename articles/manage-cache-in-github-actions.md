---
id: manage-cache-in-github-actions
title: GitHub Actions のキャッシュを管理
emoji: 🎚️
type: tech
topics:
  - githubactions
  - githubcli
published: true
---

少し前、久々に GitHub Actions でキャッシュの設定を作ろうとしたところ「想定したようにキャッシュができているかの確認がちょっと大変」と感じました。そのようなときに、キャッシュ管理用の extension が公開されたのでその辺を少し。

@[card](https://github.blog/changelog/2022-07-28-github-cli-extension-to-manage-actions-cache/)

:::message
2023-01-22 追記: 2023 年 1 月の状況にあわせて新しい記事を作成しました。こちらでは GitHub 上のウェブ UI による管理と、ワークフローを使った自動化について記述してあります。

@[card](https://zenn.dev/hankei6km/articles/manage-cache-in-github-actions-2023-01)
:::

:::message
2022-08-02 追記: extension でもブランチ名での絞り込みが可能だったので修正しました。
:::

## API を使う

まずは extension ではなく GitHub の API を使ってリポジトリのキャッシュを管理する場合について。

@[card](https://docs.github.com/en/rest/actions/cache)

GitHub API の利用方法はいくつかありますが、今回は簡単に GitHub CLI から利用してみます。

### 使用量の確認

どれくらい利用しているかの確認です。

@[card](https://docs.github.com/en/rest/actions/cache#get-github-actions-cache-usage-for-a-repository)

```shell-session
$ gh api -H "Accept: application/vnd.github+json" "/repos/{owner}/{repo}/actions/cache/usage"
{
  "full_name": "hankei6km/xquo",
  "active_caches_size_in_bytes": 10572695430,
  "active_caches_count": 58
}
```

### 一覧表示

新しい設定でキャッシュを作成したとき「キーの設定などがうまくいっているか」の確認をしたくなります。そのようなときに利用します。

@[card](https://docs.github.com/en/rest/actions/cache#list-github-actions-caches-for-a-repository)

**図 1-1 API でリポジトリのキャッシュ一覧を表示**

```shell-session
$ gh api -H "Accept: application/vnd.github+json" "/repos/{owner}/{repo}/actions/caches"                           
{                                                                                                                     
  "total_count": 43,                                                                                                  
  "actions_caches": [                                                                                                 
    {                                                                                                                 
      "id": 81,                                                                                                       
      "ref": "refs/heads/main",                                                                                       
      "key": "Windows-install-b0ca1dec0c00f9f3444c0b897b128153c05286105c0fbdade5753f42a5e9c533",                      
      "version": "ec0fe727cf292ba34588c5de04f72efd6f7301ff9bb3ce383753f7b108f30051",                                  
      "last_accessed_at": "2022-07-31T10:39:20.456666700Z",                                                           
      "created_at": "2022-07-26T07:40:25.280000000Z",                                                                 
      "size_in_bytes": 174681688                                                                                      
    },                                                                                                                
    {                                                                                                                 
      "id": 90,                                                                                                       
      "ref": "refs/heads/topic/fix-wrong-cache-key",                                                                  
      "key": "Linux-cargo-clippy-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6",                   
      "version": "4464b218375e2702da53c92c2e51ffdd3cdd5aeba23dec304d9c1239b4ef594a",                                  
      "last_accessed_at": "2022-07-31T10:39:06.903333300Z",                                                           
      "created_at": "2022-07-30T07:05:09.946666700Z",                                                                 
      "size_in_bytes": 181828878                                                                                      
    },

    # snip...
```

必要な項目だけ表示する場合。jq のオプションで指定できます。

**図 1-2表示する項目を指定**

```shell-session
$ gh api -H "Accept: application/vnd.github+json" "/repos/{owner}/{repo}/actions/caches" --jq ".actions_caches[] | {id, key, last_accessed_at, ref}"                                 
{"id":81,"key":"Windows-install-b0ca1dec0c00f9f3444c0b897b128153c05286105c0fbdade5753f42a5e9c533","last_accessed_at":"2022-07-31T10:39:20.456666700Z","ref":"refs/heads/main"}                                                              
{"id":90,"key":"Linux-cargo-clippy-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:39:06.903333300Z","ref":"refs/heads/topic/fix-wrong-cache-key"}
{"id":79,"key":"Linux-install-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:39:00.980000000Z","ref":"refs/heads/main"}  
{"id":91,"key":"Linux-cargo-test_release_jemalloc-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:39:00.900000000Z","ref":"refs/heads/topic/fix-wrong-cache-key"}
{"id":89,"key":"Linux-cargo-test_release-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:39:00.106666700Z","ref":"refs/heads/topic/fix-wrong-cache-key"}
# snip...
```

ref での絞り込みもできます。「作業中のプルリクエストで操作したキャッシュ」の確認に便利です。

**図 1-3表示する項目を指定**

```shell-session
$ gh api -H "Accept: application/vnd.github+json" "/repos/{owner}/{repo}/actions/caches?ref=refs/heads/topic/fix-wrong-cache-key" --jq ".actions_caches[] | {id, key, last_accessed_at}"
{"id":90,"key":"Linux-cargo-clippy-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:39:06.903333300Z"}
{"id":91,"key":"Linux-cargo-test_release_jemalloc-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:39:00.900000000Z"}
{"id":89,"key":"Linux-cargo-test_release-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:39:00.106666700Z"}
{"id":88,"key":"Linux-cargo-test_debug-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:38:56.250000000Z"}
```

### キャッシュの削除

間違えて作ってしまったキャッシュを削除したいときに利用します。こちらも ref などで対象を指定できますが、`id` を使うのがお手軽でよいかと思います。

@[card](https://docs.github.com/en/rest/actions/cache#delete-a-github-actions-cache-for-a-repository-using-a-cache-id)

**図 1-4 id が 90 のキャッシュを削除**

```shell-session
$ gh api --method DELETE -H "Accept: application/vnd.github+json" /repos/{owner}/{repo}/actions/caches/90

$ gh api -H "Accept: application/vnd.github+json" "/repos/{owner}/{repo}/actions/caches?ref=refs/heads/topic/fix-wrong-cache-key" --jq ".actions_caches[] | {id, key, last_accessed_at}"
{"id":91,"key":"Linux-cargo-test_release_jemalloc-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:39:00.900000000Z"}
{"id":89,"key":"Linux-cargo-test_release-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:39:00.106666700Z"}
{"id":88,"key":"Linux-cargo-test_debug-745cab0288fc37d8864b0178ed01e0dc115354d51702a55b62bc511f28d37ef6","last_accessed_at":"2022-07-31T10:38:56.250000000Z"}
```

ただし、 Codespaces で自動的に付与される `GITHUB_TOKEN` ではエラーになります。

**図 1-5 Codespaces ではエラーになる**

```shell-session
$ gh api --method DELETE -H "Accept: application/vnd.github+json" /repos/{owner}/{repo}/actions/caches/90
{
  "message": "Resource not accessible by integration",
  "documentation_url": "https://docs.github.com/rest/actions/cache#delete-a-github-actions-cache-for-a-repository-using-a-cache-id"
}
gh: Resource not accessible by integration (HTTP 403)
```

## gh-actions-cache

API でもそれなりに利用できますがやはり少し面倒です。今度は extension を利用してみます。

@[card](https://github.com/actions/gh-actions-cache)

### インストール

extension なので GitHub CLI 環境にインストールする必票があります。とはいっても下記のコマンド実行するだけです[^container]。

[^container]: `gh` コマンドは GitHub へログインしてる必要があります。よって `Dockerfile` へ単純にコマンドを記述してもインストールできません。Dev Contaioenr へインストールする場合は、postStartCommand などを使うのが楽かと思います。

**図 2-1 インストールコマンド**

```shell-session
$ gh extension install actions/gh-actions-cache
```

### 使用量と一覧表示

`list` コマンドで表示されます。このコマンドでは使用量もあわせて表示されます。

**図 2-2 一覧表示**

```shell-session
$ gh actions-cache list --limit 10
Total caches size 7.24 GB

Showing 10 of 43 cache entries in hankei6km/xquo

Linux-cargo-test_release_jemalloc-745cab0288fc37d8... [203.50 MB]     refs/heads/to...  4 hours ago        
Linux-cargo-clippy-745cab0288fc37d8864b0178ed01e0d... [173.41 MB]     refs/heads/to...  4 hours ago        
Linux-cargo-test_release-745cab0288fc37d8864b0178e... [136.02 MB]     refs/heads/to...  4 hours ago        
Linux-cargo-test_debug-745cab0288fc37d8864b0178ed0... [170.82 MB]     refs/heads/to...  4 hours ago        
Windows-install-b0ca1dec0c00f9f3444c0b897b128153c0... [166.59 MB]     refs/heads/main   4 hours ago        
Linux-install-745cab0288fc37d8864b0178ed01e0dc1153... [204.45 MB]     refs/heads/main   4 hours ago        
Linux-x86_64-unknown-linux-musl-cargo-745cab0288fc... [176.99 MB]     refs/tags/v0.2.0  4 days ago         
Linux-x86_64-pc-windows-gnu-cargo-745cab0288fc37d8... [118.54 MB]     refs/tags/v0.2.0  4 days ago         
Linux-cargo-$test_release_jemalloc-745cab0288fc37d... [203.51 MB]     refs/heads/main   4 days ago         
Linux-publish-745cab0288fc37d8864b0178ed01e0dc1153... [133.08 MB]     refs/tags/v0.2.0  4 days ago
```

API の ref と同じようにブランチ名での絞り込みもできます。

**図 2-3 ブランチ名での絞り込み**

```shell-session
$ gh actions-cache list --branch topic/fix-wrong-cache-key
Showing 3 of 3 cache entries in hankei6km/xquo

Linux-cargo-test_release_jemalloc-745cab0288fc37d8... [203.50 MB]     refs/heads/to...  2 days ago
Linux-cargo-test_release-745cab0288fc37d8864b0178e... [136.02 MB]     refs/heads/to...  2 days ago
Linux-cargo-test_debug-745cab0288fc37d8864b0178ed0... [170.82 MB]     refs/heads/to...  2 days ago
```

キーでの絞り込みもできますが、GLOB などは使えないようです。

**図 2-4 キーでの絞り込みを試す**

```shell-session
$ gh actions-cache list --limit 10 --key windows
Showing 6 of 6 cache entries in hankei6km/xquo

Windows-install-b0ca1dec0c00f9f3444c0b897b128153c0... [166.59 MB]     refs/heads/main   4 hours ago        
Windows-install-f0fe93585e34ba5d2b2a8fcfe58bac58ac... [164.25 MB]     refs/heads/main   4 days ago         
Windows-install-f0fe93585e34ba5d2b2a8fcfe58bac58ac... [164.26 MB]     refs/heads/de...  5 days ago         
Windows-install-fd8e39df0ffc26a77d1c14ef70e41736dc... [164.24 MB]     refs/heads/main   5 days ago         
Windows-install-fd8e39df0ffc26a77d1c14ef70e41736dc... [164.10 MB]     refs/heads/de...  5 days ago         
Windows-install-fb7c9cbf667a82540d3d444f3e547b3291... [164.08 MB]     refs/heads/de...  5 days ago         

$ gh actions-cache list --limit 10 --key install
There are no Actions caches currently present in this repo or for the provided filters

$ gh actions-cache list --limit 10 --key "*install*"
There are no Actions caches currently present in this repo or for the provided filters
```

なお、表示項目の変更などはできないようです。

こう書くと良くない印象になりそうですが、表示は見やすいのでキー指定のミスなどは発見しやすいです。実際、キーに余分な `$` を含めていたのを見つけてしまいました。

@[card](https://github.com/hankei6km/xquo/pull/24)

### 削除

`delete` コマンドで削除できますがキーを指定する必票があります。部分一致ではエラーになるので、`list` コマンドでキーを確認する場合は省略されないようにする必要があります。ちょっと削除したいときの利用では少し面倒かもしれません。

**図 2-5 削除はキーを利用する**

```shell-session
$ gh actions-cache list --limit 10 --key windows
Showing 6 of 6 cache entries in hankei6km/xquo

Windows-install-b0ca1dec0c00f9f3444c0b897b128153c05286105c0fbdade5753f42a5e9c533                        [166.59 MB]     refs/heads/main                  5 hours ago        
Windows-install-f0fe93585e34ba5d2b2a8fcfe58bac58ac56ab2810e67109118e4dd176c11b49                        [164.25 MB]     refs/heads/main                  4 days ago         
Windows-install-f0fe93585e34ba5d2b2a8fcfe58bac58ac56ab2810e67109118e4dd176c11b49                        [164.26 MB]     refs/heads/dependabot/cargo/...  5 days ago         
Windows-install-fd8e39df0ffc26a77d1c14ef70e41736dcc0444514249786fbcbdfd6c838c386                        [164.24 MB]     refs/heads/main                  5 days ago         
Windows-install-fd8e39df0ffc26a77d1c14ef70e41736dcc0444514249786fbcbdfd6c838c386                        [164.10 MB]     refs/heads/dependabot/cargo/...  5 days ago         
Windows-install-fb7c9cbf667a82540d3d444f3e547b32919a2c6fc8621caa7eadf7e6aa111f46                        [164.08 MB]     refs/heads/dependabot/cargo/...  5 days ago         

$ gh actions-cache delete Windows-install-fb7c9cbf667a82540d3d444f3e547b32919a2c6fc8621caa7eadf7e6aa111f46
You're going to delete 1 cache entry

Windows-install-fb7c9cbf667a82540d3d444f3e547b32919a2c6fc8621caa7eadf7e6aa111f46                        [164.08 MB]     refs/heads/dependabot/cargo/...  5 days ago         

? Are you sure you want to delete the cache entries?  [Use arrows to move, type to filter]
> Delete
  Cancel
```

`delete `については実際の利用に[便利そうな改善案](https://github.com/actions/gh-actions-cache/issues/20)などが出ているので、そちらに期待といった感じでしょうか。

## おわりに

GitHub Actions のキャッシュ管理について見てみました。

ワークフローのキャッシュは動いてしまえば気にする機会も減ります。ですが「初めて使う言語用に依存関係をキャッシュする」「Docker のビルドキャッシュを設定する」ような場合には確認や削除がしたくなります。

そのようなとき「どうやって確認するんだっけ？」となりがちなので、 `gh-actions-cache list` のお手軽な使いやすさはありがたい存在となりそうです。
