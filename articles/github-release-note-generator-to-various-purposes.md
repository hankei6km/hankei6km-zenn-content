---
id: github-release-note-generator-to-various-purposes
title: GitHiub のリリースノート自動生成機能をリリースノート以外にも使ってみる
emoji: ⚗️
type: idea
topics:
  - github
  - githubapi
  - githubactions
  - githubcli
published: true
---

リリースノートに含まれるプルリクエストによってワークフローの処理を制御したいと考えていたので、その一環として GitHub のリリースノートの自動生成機能を試してみました。

@[card](https://docs.github.com/ja/repositories/releasing-projects-on-github/automatically-generated-release-notes)

## 普通に使う

新しく作成するリリースのタグとその 1 つ前のタグの間にプルリクエストがあれば以下のように自動生成してくれます。

標準の設定で使うなら追加のファイルなども必要なくお手軽に利用できます。

![GitHub のウェブ UI でリリースノートを編集しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6aea7026ad474f80a49aeef3b97d502d/github-release-note-generator-to-various-purposes-web-ui-edit.png?w=600\&h=299\&border64=MSw1NTAwMDAwMA\&border-radius64=MQ)*▲ 図 1-1 編集画面で赤枠のボタンクリックで生成される*

![GitHub 上で作成されたリリースを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/30e7c964ea834dedb66427fedf651931/github-release-note-generator-to-various-purposes-view.png?w=600\&h=372\&border64=MSw1NTAwMDAwMA\&border-radius64=MQ)*▲ 図 1-2 実際に作成されたリリース*

図 1-3 は [GitHub CLI] で利用している例です。こちらも設定などは必要なかったのですが、メニューなどから利用できるのは [v2.4.0](https://github.com/cli/cli/releases/tag/v2.4.0) からだと思われます。

> *   `release create`: add `--generate-notes` functionality by [@Sixeight](https://github.com/Sixeight) in [#4467](https://github.com/cli/cli/pull/4467)

▼ *図 1-3 GitHub CLI ではフラグやメニューから選択*

```shell-session
$ gh release create v0.6.0 -t 0.6.0
? Title (optional) 0.6.0
? Release notes  [Use arrows to move, type to filter]
  Write my own
> Write using generated notes as template
  Leave blank
```

## 設定ファイルの利用

リスト 2-1 のようなファイルを `.github/release.yml` として GitHub 上のデフォルトブランチに保存しておくと項目(プルリクエスト)をカテゴリー別に表示できます[^config-file]。

▼ *リスト 2-1 設定ファイルのサンプル*

```yaml:release.yml
changelog:
  categories:
    - title: Features
      labels:
        - enhancement
    - title: Types Changes
      labels:
        - types
    - title: Bug Fixes
      labels:
        - bug
    - title: Other Changes
      labels:
        - '*'
```

少し試した感じでは定型文的なものを挿入したりはできなさそうです(テンプレートではなく設定ファイルという位置づけのようです)。

![設定ファイルを適用したリリースノートを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/fb45df872c374e4f984ba42c26ec3953/github-release-note-generator-to-various-purposes-config.png?w=593\&h=423\&border64=MSw1NTAwMDAwMA\&border-radius64=MQ)*▲ 図 2-1 設定によりカテゴリー別に表示される*

なお、設定内容に混乱するような項目はないのですが、「リリース作成で指定するタグよりも前に追加しておく必要がある」という点には注意が必要です。

たとえば、`v0.4.0` タグを作成した後に設定ファイルを追加しても、`v0.4.0` を指定したリリースの作成には使えません。一旦ドラフトリリースの編集画面を開いて切り貼りするなどの回避策が必要になります。

また「複数のラベルが設定されている項目は最初にマッチしたカテゴリーにのみ追加される」ようなので、カテゴリーの優先度は少し考える必要があります。

[^config-file]: 他のブランチやローカルにだけ存在する場合は機能しなかったです。

## API から利用

リリースノートの自動生成は [単体の API としても提供](https://docs.github.com/ja/rest/reference/releases#generate-release-notes-content-for-a-release)されています[^graphql]。

API ではウェブ UI や `gh release create` から使えないパラメーターも指定できるので、 [GitHub Actions] からの利用やちょっと確認したいという場合に重宝します。

▼ *図 3-1 GitHub CLI からタグの指定範囲を広げての生成*

```shell-session
$ gh api /repos/{owner}/{repo}/releases/generate-notes -f tag_name=v0.4.0 -f previous_tag_name=v0.1.0 --jq .body
<!-- Release notes generated using configuration in .github/release.yml at v0.4.0 -->

## What's Changed
### Features
* Make function two by @hankei6km in https://github.com/hankei6km/test-release-generate-notes/pull/4
* Make function three by @hankei6km in https://github.com/hankei6km/test-release-generate-notes/pull/6
* Add "filter-bug.yml" to filter pr by labels by @hankei6km in https://github.com/hankei6km/test-release-generate-notes/pull/9
### Bug Fixes
* Fix query by @hankei6km in https://github.com/hankei6km/test-release-generate-notes/pull/7
### Other Changes
* Add section to README by @hankei6km in https://github.com/hankei6km/test-release-generate-notes/pull/5
* Add release config files by @hankei6km in https://github.com/hankei6km/test-release-generate-notes/pull/8


**Full Changelog**: https://github.com/hankei6km/test-release-generate-notes/compare/v0.1.0...v0.4.0
```

この他、`.github/release.yml` 以外の設定ファイルも指定できます。「プレリリースのときは異なる設定で生成したい」というような場合などに利用できそうです。

[^graphql]: ちょっと試した限りでは GraphQL では利用できなさそうです。

## GitHub Actions でフィルターとして使う

「特定のタグ間に含まれているプルリクエストのラベルによって処理を変更する」というのは記述がわりと大変そうです[^explorer]。

このようなとき、以下のような設定ファイルを利用すると「ラベルに `bug` が設定されたプルリクエストを含む」という判定が簡単にできます。

▼ *リスト 4-1 特定のラベルが存在するときだけ表示*

```yaml
changelog:
  categories:
    - title: bug
      labels:
        - bug
```

▼ *図 4-1 フィルター用の設定ファイルを指定して API を実行*

```shell-session
$ gh api /repos/{owner}/{repo}/releases/generate-notes -f tag_name=v0.7.0 -f previous_tag_name=v0.1.0 -f configuration_file_path="rel-filter/filter-bug.yml" --jq .body
<!-- Release notes generated using configuration in rel-filter/filter-bug.yml at v0.6.0 -->

## What's Changed
### bug
* Fix query by @hankei6km in https://github.com/hankei6km/test-release-generate-notes/pull/7


**Full Changelog**: https://github.com/hankei6km/test-release-generate-notes/compare/v0.1.0...v0.6.0
```

リスト 4-2 は GitHub Actions で利用している例です。特定のタグ間でリリースノートを生成し、含まれているプルリクエストのラベルで処理を制御しています。

▼ *リスト 4-2 GitHub Actions のワークフローで利用*

```yaml
name: Filter by note
on:
  push:
    branches:
      - "**"

jobs:
  filter-by-note:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Filter refactoring
        id: filter-refactoring
        run: >-
          OWNER="$(cut -d '/' -f 1 <<< ${REPOSITORY})";
          NAME="$(cut -d '/' -f 2 <<< ${REPOSITORY})";
          gh api "/repos/${OWNER}/${NAME}/releases/generate-notes" -f "tag_name=${TAGNAME}" -f "previous_tag_name=${PREV_TAGNAME}" -f "configuration_file_path=${CONFIG_FILE}" --jq .body
          | sed -e 's/^### /::set-output name=label::/'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          REPOSITORY: ${{ github.repository }}
          TAGNAME: v0.7.0
          PREV_TAGNAME: v0.1.0
          CONFIG_FILE: rel-filter/filter-refactoring.yml

      - name: Filter bug
        id: filter-bug
        run: >-
          OWNER="$(cut -d '/' -f 1 <<< ${REPOSITORY})";
          NAME="$(cut -d '/' -f 2 <<< ${REPOSITORY})";
          gh api "/repos/${OWNER}/${NAME}/releases/generate-notes" -f "tag_name=${TAGNAME}" -f "previous_tag_name=${PREV_TAGNAME}" -f "configuration_file_path=${CONFIG_FILE}" --jq .body
          | sed -e 's/^### /::set-output name=label::/'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          REPOSITORY: ${{ github.repository }}
          TAGNAME: v0.7.0
          PREV_TAGNAME: v0.1.0
          CONFIG_FILE: rel-filter/filter-bug.yml

      - name: Check refactoring
        if: ${{ contains(steps.filter-refactoring.outputs.label, 'refactoring') }}
        run: echo "refactoring"

      - name: Check bug
        if: ${{ contains(steps.filter-bug.outputs.label, 'bug') }}
        run: echo "bug"
```

生成されたリリースノートに `refactoring` ラベルが設定されたプルリクエストはなかったので Check refactoring ステップは実行しなかったことを確認できます。

![GitHub 上で実行結果を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6ed541d5f34a4215aa64c8b63b2b46a9/github-release-note-generator-to-various-purposes-run.png?w=442\&h=402\&auto=compress%2Cformat)*▲ 図 4-2 生成されたノートによって処理が制御されている*

なお、「カテゴリー名に `set-output` コマンドを直接記述したら `sed` での整形なくてもよいのでは」と考えたのですが、行頭に見出しの `###` が付くのでダメでした[^inline]。

[^explorer]: [GraphQL Explorer] とにらめっこしてみましたが挫折しました。

[^inline]: うまくったとしても、コマンドがワークフローファイルの外にあると記述しにくかったので良い方法ではないです。

## 応用

以下のスクラップでリリースノートからラベル一覧の取得を試しました。

スクラップでは実際のリリースから HTML を取得して使っていますが、自動生成 API でも同じように「生成されたリリースノートを JSON に変換しての利用」も可能と思われます[^rehype]。

@[card](https://zenn.dev/link/comments/c4a8a511802f32)

[^rehype]: 利用する API を変更して [`rehype-cli`] のかわりに [`remark-cli`] を使えばいけるかと。

## おわりに

フィルター的に使う目的で GitHub のリリースノート自動生成を試してみましたが、この機能は普通に使っても便利だと感じました。

フィルターについては想定されていない使い方なのでよろしくない部分もありますが、リリースノート生成が API から単体で利用できることは使い方によって自動化などに重宝すると思われます。

[GitHub CLI]: https://cli.github.com/

[GitHub Actions]: https://github.co.jp/features/actions

[GraphQL Explorer]: https://docs.github.com/en/graphql/overview/explorer

[`rehype-cli`]: https://www.npmjs.com/package/rehype-cli

[`remark-cli`]: https://www.npmjs.com/package/remark-cli
