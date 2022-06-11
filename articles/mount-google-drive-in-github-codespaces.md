---
id: mount-google-drive-in-github-codespaces
title: GitHub Codespaces から GoogleDrive をマウントしファイルの永続化と共有
emoji: 💾
type: tech
topics:
  - github
  - codespaces
  - googledrive
published: true
---

先日、順番待ちしていた GitHub Codespace を使えるようになりました[^long-long-ago]。

[^long-long-ago]: 我々は 3 年待ったのだ、とまではいかなくても 2 年近く待ちました。

早速ワクワクで GitHub Codespace を試しているのですが、ローカルでの Remote - Container に比べると「ボリュームの永続化が大変そう」な感じがしています。

そこで、Codespaces のコンテナ(Dev Container)内から Google Drive をマウントするための `.devcontainer` を作ってみました。

## どのようにマウントするのか？

以下のリポジトリは [google-drive-ocamlfuse](https://github.com/astrada/google-drive-ocamlfuse) を利用して Google Drive をマウントするための `.devcontainer` が含まれています。

@[card](https://github.com/hankei6km/test-codespaces-mount-google-drive)

この `.devcontainer` の内容を Google Drive をマウントしたい Codespace(リポジトリ) に組み込むと、認証情報を SECRET に設定するだけで利用できます。

## 利用方法

ここではサンプルのリポジトリを使ってマウントしてみます。

サンプルのリポジトリから Codespace を作成します(認証情報を扱うので、気になる場合はテンプレートから別のリポジトリを作成してください)。

▼ **図 2-1 リポジトリから Codespace を作成**

![ブラウザーでサンプルリポジトリを開き、Code メニューから Codespace を作成しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/be57ebcaee3b46c384afaed9de7b9a82/mount-google-drive-in-github-codespaces-create.png?w=1007\&h=614\&auto=compress%2Cformat)

その後は認証の方法によって、「基本的な認証」「Service Accounts」のいずれかを行います。

### 標準的な認証

認証情報を取得するために、以下の記事などを参考にブラウザー経由で認証しマウントします(この操作は Codespace ではなく通常のデスクトップ環境で行っても大丈夫です)。

@[card](https://qiita.com/boss_ape/items/74bcad7441417c331205)

マウントすると `${HOME}/.gdfuse/<label 名>` のディレクトリが作成されるので、`config` と `state` ファイルの中を確認します。

リポジトリ SECRET の Codespaces 用に以下を作成します(ユーザー SECRET でも動作します)[^secret]。

*   `GDFUSE_CONFIG` - `config` ファイルの内容を丸ごとコピー
*   `GDFUSE_STATE` - `state` ファイルの内容を丸ごとコピー

▼ **図 2-2 SECRET にファイルの内容をコピー**

![Codespaces 用の SECRET に state ファイルの内容を貼り付けているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/87ec32e7a5d9403c8667b5fbd3d63234/mount-google-drive-in-github-codespaces-add-std-secret.png?w=863\&h=419\&auto=compress%2Cformat)

作成後、Codespace を Rebuild(Reload ではないです)すると `${HOME}/gdrive` に Google Drive がマウントされます。

### Service Accounts

Drive API を有効化した GCP プロジェクトでサービスアカウントを作成し、`gcloud iam service-accounts keys` などで鍵ファイルを作成します。

リポジトリ SECRET の Codespaces 用に以下を作成します(ユーザー SECRET でも動作します)[^secret]。

*   `GDFUSE_SA` - 鍵ファイルの内容を丸ごとコピー

▼ **図 2-3 SECRET に鍵ファイルの内容をコピー**

![Codespaces 用の SECRET に鍵ファイルの内容を貼り付けているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0dbcc469a0d24a6d95db66c5132c9494/mount-google-drive-in-github-codespaces-add-sa-secret.png?w=844\&h=425\&auto=compress%2Cformat)

作成後、Codespace を Rebuild(Reload ではないです)すると `${HOME}/gdrive` に Google Drive がマウントされます。なお、サービスアカウントの鍵ファイルは `${HOME}/gd-sa-cred.json` に保存されているので取り扱いには注意してください。

また、サービスアカウントを利用した場合「サービスアカウントのドライブ」がマウントされます。普段使っているドライブでファイルをやりとりしたい場合は、下記の `gdrive` などで共有することになります。

@[card](https://github.com/prasmussen/gdrive)

たとえば、Codespace で `${home}/gdrive/share` フォルダーを作成し他ユーザーと共有する場合は以下のようになります。

▼ **図 2-4 `gdrive` の一時的なインストール(linux\_386 を使う)**

```shell-session
$ mkdir ~/tmp
$ cd ~/tmp
$ curl -sL https://github.com/prasmussen/gdrive/releases/download/2.1.1/gdrive2.1.1linux_386.tar.gz | tar -zxvf -
```

▼ **図 2-5 フォルダーの作成と `gdrive` による folder id の確認**

```shell-session
$ cd ~
$ mkdir gdrive/share
$ ./tmp/gdrive --service-account gd-sa-cred.json -c . list
Id                                  Name                Type   Size      Created
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx   share               dir              2022-06-08 09:33:42
# 実際は複数の項目が表示されます
```

▼ **図 2-6 folder id を指定し共有**

```shell-session
$ ./tmp/gdrive --service-account gd-sa-cred.json -c . share --role writer --type user --email <mail address> xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

これで共有が完了します。

この他、サービスアカウントの利用では制限の解除も必要になることがあるので、以下も参照してください。

@[card](https://github.com/astrada/google-drive-ocamlfuse/wiki/Service-Accounts)

[^secret]: [GitHub Actions のようにマスクなどを考慮する](https://zenn.dev/hankei6km/articles/credentials-contained-files-on-github-actions)必要はなさそうだったので、ファイルを丸ごと貼り付けて `echo` で保存しています。

## サンプル画面

設定が完了すると以下のような感じでドライブ内のファイルを扱えます。

▼ **図 3-1 Google Drive に保存した画像を Codespace 側で参照**

![ブラウザー上で Google Drive と Codespace を表示し、それぞれで同じ画像ファイルを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1eab81ad8bf5403689c22e2dd982f6ad/mount-google-drive-in-github-codespaces-share.png?w=1440\&h=858\&auto=compress%2Cformat)

## 注意点

### 必要な権限

コンテナ内から fuse を使ってマウントするには(試した限りでは) `--privileged` が必要です。`--cap-add` で `SYS_ADMIN` を指定した場合は `fuse: device not found, try 'modprobe fuse' first` となりました。

### Ubuntu ベース

今回の `.devcontainer` は Ubuntu ベースのイメージでないと動きません。[`microsoft/vscode-dev-containers`](https://github.com/microsoft/vscode-dev-containers) でも [typescript-node](https://github.com/microsoft/vscode-dev-containers/tree/main/containers/typescript-node) などは Ubuntu ベースの variant がないのでエラーになります。

できれば対応したかったのですが、「Codespaces のデフォルトが Ubuntu ベースだったので個人用のイメージは Ubuntu に統一しようかな」ということでそのままになってしまいました。

### レスポンス

マウントしたディレクトリのファイルを直接編集するような用途では少し厳しい印象です(Codespaces の Region 設定は automatically で試しています)。

## おわりに

GitHub Codespaces のコンテナ内から Google Drive をマウントしてみました。

現状では以下のような用途で利用していますが、やはり「入力履歴を永続化できるのは非常に良い」です。

*   `.bash_history` などの入力履歴の永続化と Codespace 間での共有([別途記事にしました](https://zenn.dev/hankei6km/articles/persist-command-history-in-github-codesapces))
*   シークレットなどを含むファイルの配置

この他にもちょっとしたファイルのやりとりなどに利用できそうなので、自分が作るリポジトリでは使っていこうかと考えています。
