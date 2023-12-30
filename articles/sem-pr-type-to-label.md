---
id: sem-pr-type-to-label
title: semantic-pull-request の情報を使ってラベルを付ける Action を作った
emoji: 🏷️
type: tech
topics:
  - githubactions
  - pullrequest
published: true
---

GitHub でリリースを作成するときに、[自動生成リリース ノート]を使いたいのでプルリクエストにはラベルを付けるようにしています。

しかし、ラベルを手動で追加しているので少し苦労しています。

*   必要なときに(機能追加や修正のプルリクエストに)ラベル追加するのを忘れてしまう
*   [Conventional Commits]を導入しているので type を利用したい

そこで、遅まきながらプルリクエストへのラベル追加を GitHub Actions で行うようにしました。

## ラベルを追加するルール

GitHub Actions でラベルを追加するといっても「このプルリクエストはどんな目的で作成したのか？」を Action へ提示する必要があります。

まず、思いつくのは「変更したファイル名から絞り込む」ですが、「新しい機能を追加」か「バグ修正」などを機械的に判断することは難しいです(そのうち Copilot がやってくれそうかな)。

一方で、少し前からコミットメッセージの記述には [Conventional Commits] を導入していたので、コミット単位で機械的に種類(type)を判定できるようになっています。

**リスト 1-1 コミットメッセージの構造(**[**引用元**](https://www.conventionalcommits.org/en/v1.0.0/#summary)**)**

    <type>[optional scope]: <description>

    [optional body]

    [optional footer(s)]

**リスト 1-2 サンプル("feat" を指定することで機能追加を表す)**

    feat: "--format-json" オプションを追加

type の指定などは(いまのところは)人間がやることなので少し手間に感じますが、毎回記述することなので「付けるの忘れてしまった」ということは発生しにくいです([検証するツール](https://github.com/conventional-changelog/commitlint)もあります)。

このように機械的な判定をできる情報はそろっているので、これをプルリクエストに利用できないか探したところ、[semantic-pull-request]という考え方がありました。

## semantic-pull-request とは

[Action のリポジトリ](https://github.com/amannn/action-semantic-pull-request)を見ていただくのが速いのですが、大雑把に言うと以下のような感じの取り決めです。

*   [Conventional Commits] 風にプルリクエストのタイトルや説明を作成しましょう
*   type などを決めるときはコミットメッセージと矛盾しないようにしましょう

**リスト 2-1 サンプル(**[**引用元**](https://github.com/amannn/action-semantic-pull-request#configuration)**)**

    feat(ui): Add `Button` component
    ^    ^    ^
    |    |    |__ Subject
    |    |_______ Scope
    |____________ Type

この取り決めを導入すると、機械的に判定しやすいワード(type)がプルリクエストのタイトルなどにも埋め込まれるので、あとからリリースノートを作成するようなとき便利になるという寸法です。

## type からラベル

type は機能的にラベルに近いので「type からラベルを付ける Action は豊富にありそう」と思っていたのでしたが、意外と見当たりませんでした[^changelog]。

[^changelog]: [Conventional Changelog] の利用が活発なようでして、ラベルベースでの管理はあまりしないのかなと。

*   上記の [semantic-pull-request] は取り決めに沿っているか判定する Action
*   人気の [Labeler] は変更されたファイルかブランチ名でラベルを付ける Action

最初は「ブランチ名の先頭に `feat_` とか付けるかな」と考えていたのですが、少し試してみたところいまいちだったのでやめました。

*   プルリクエストのブランチ名は変更できない[^close]
*   タイトルとブランチ名に同じもの(type)を 2 回記述するのは手間に感じた

[^close]: type が変わるなら一旦閉じるのではという話ではありますが。

## Action を作る

前節のことから、あきらめて Action を作ることにしました。

### `sem-from-title`

@[card](https://github.com/hankei6km/gha-sem-from-title)

タイトル部分から「type と scope および breaking change か？」の情報を取りだすだけの Action です。「type などの情報は他にも利用できるかな」ということでラベルを追加する Action とは別に作りました。

また、取り決めに沿っているか(type のバリデーションなど)を判定するとオプション(with)指定が複雑になりがちなので、あくまでも情報を取りだすだけにしてあります。

なお、プレフィックス(`[WIP]`) やフッター(`BREAKING CHANGE:`)はリリースノートの自動生成に対応する機能がないので、いまのところ対応していません。

### `sem-pr-labeler`

@[card](https://github.com/hankei6km/gha-sem-pr-labeler)

「type とおよび breaking change か？」を渡すと対応するラベルを追加する Action です。

対応するラベルは `sem-pr:` 接頭詞を付けて作成します。これは主に利用するラベルをグループ化しておくためのものです。接頭詞は変更できますが指定は必要です(ロジックとオプション指定の簡素化にも利用しているため)。

また、存在しないラベルを追加する場合はエラーとなります。これはバリデーションを兼ねるために行っています。

## 使ってみる

使い方は [README](https://github.com/hankei6km/gha-sem-pr-labeler/blob/main/README.md) にある通りですが、少し補足など。

### ラベル作成

上の方にも書きましたが対応するラベルを事前定義する必要があります。

ちょっと数が多いのでスクリプトにしました。

**リスト 5-1 ラベル追加のサンプル**

https://github.com/hankei6km/gha-sem-pr-labeler/blob/main/scripts/init-labels.sh

### [semantic-pull-request] Action は利用しない

[semantic-pull-request] Action との依存関係はないのでとくに必要としません(併用もできます)。

### ワークフロー

あまり凝ったことをしなければ READM のサンプルで動くと思います。

なお、[Labeler] Action と併用する場合、トリガーの `type` が異なるのでワークフローをわけたくなります。しかし、同時に動かすとコンフリクトするようなので(コンフリクトについては確認中)以下のように 1 つにまとています[^concurrency]。

[^concurrency]: サンプルでは `if` で制御していますが、ワークフローを複数にわけたまま [`concurrency`](https://docs.github.com/ja/actions/using-jobs/using-concurrency) を使う方法もあります。

https://github.com/hankei6km/h6-devcontainers-features/blob/main/.github/workflows/label.yml

## リリースノートの構成

ラベル名に独自の接頭詞を入れてあるので、対応できるように `.github/relase.yml` を変更します。

**リスト 6-1 リリースノート構成のサンプル**

https://github.com/hankei6km/gha-sem-pr-labeler/blob/main/.github/release.yml

上記のように設定した後、プルリクエストを作成しリリースノートを自動生成すると以下のようになります。

**図 6-1 生成されたリリースノート**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/64edcbf4d7374cb0ae2b786985099b6d/sem-pr-type-to-label-release.png?auto=compress%2Cformat)

## 考慮点

### scope

scope も扱うべきですが、以下の点から今回は保留にしました。 (「サーバー」や「クライント」のような scope を区別させたい場合は、とりあえず [Labeler] Action でラベルを追加しています)。

*   scope は逆にラベルからタイトルを更新できないか考えている
*   マルチ scope の扱いがよくわからなかった(ツールによって解釈が違いそう)

### リリースノート作成にラベルを使わない

[Conventional Commits] を導入している場合、リリースノート作成は [Conventional Changelog] などの利用が活発な感じです。

[自動生成リリース ノート]も捨てがたいのですが、[Conventional Commits] を使い続ける場合は将来的に長いものに巻かれようかなとも考えています。

## おわりに

[semantic-pull-request] の情報を利用しプルリクエストへラベルを追加する Action を作ってみました。

考慮点で少し後ろ向きなことを書きましたが、現状ではそれなりに満足しているので当面は使ってみようかと思っています。

[自動生成リリース ノート]: https://docs.github.com/ja/repositories/releasing-projects-on-github/automatically-generated-release-notes

[Conventional Commits]: https://www.conventionalcommits.org/en/v1.0.0/

[semantic-pull-request]: https://github.com/marketplace/actions/semantic-pull-request

[Conventional Changelog]: https://github.com/conventional-changelog/conventional-changelog

[Labeler]: https://github.com/marketplace/actions/labeler

[semantic-release]: https://github.com/semantic-release/semantic-release 
