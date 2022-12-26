---
id: generate-ogimage-by-using-marp
title: Marp で OGP 用の画像を生成するカスタムテーマと Dev Contaner を作ってみた
emoji: 📐
type: tech
topics:
  - markdown
  - marp
  - ogp
published: true
---

単発で OGP 用の画像的なもの(以下 OG 画像と表記)が必要なとき、画像編集ソフトを使うのも少し面倒なので Marp を利用することがあります。

@[card](https://marp.app/)

Marp はスライドを作成するツールですが、「タイトル的なテキストにスタイルを適用した画像の生成」などにも便利に利用できます。

とはいっても画像を生成する場合、下記の点は少し面倒だったりもします。

*   画像のサイズを変更する
*   Puppeteer を動かす環境が必要
*   正しいフォントを利用する(想定していないフォントを利用したくない)

そこで、これらを解消するために OG 画像用のテーマとそれを利用するための Dev Container を作ってみました。

## どのようなテーマと Dev Container？

OG 画像を生成するためのカスタムテーマ(`ogimage`)と、それを利用するための Dev Container です。

### カスタムテーマ(`ogimage`)

カスタムテーマ(`ogimage`)には下記のような OG 画像を生成しやすいサイズのプリセットや class を定義してあります。

**図 1-1 背景画像に黒色を重ねて Avatar アイコンを表示しているサンプル**

![背景画像に黒色を重ねて Avatar アイコンを描画しているサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/070b11a578aa46db85a9bfbba69d307f/generate-ogimage-by-using-marp_sample01.png?w=1200\&h=630\&auto=compress%2Cformat)

**図 1-2 背景画像とタイトルを分割表示しているサンプル**

![画像の左側 25% 部分に背景画像、右側にタイトルと Avatar アイコンを描画しているサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/efa5081b4de64ebbbf76ded98f4f6af5/generate-ogimage-by-using-marp_sample02.png?w=1200\&h=630\&auto=compress%2Cformat)

**図 1-3 フィルターを適用した風景写真を背景にしてフォントをセリフ体にしたサンプル**

![フィルターを適用した風景写真を背景にし、フォントをセリフ体で描画しているサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/886623593d1a456aaf560eb43f1b0911/generate-ogimage-by-using-marp_sample03.png?w=1200\&h=630\&auto=compress%2Cformat)

**図 1-4 HTML を有効化し円グラフを表示しているサンプル**

![画像の中央に円グラフを描画しているサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c660837dd9714b8abe415655bd4d5380/generate-ogimage-by-using-marp-template_sample04.png?w=1200\&h=630\&auto=compress%2Cformat)

### Dev Container

Dev Container は Puppeteer 用の環境と日本語のフォントを設定してあります。また、marp-cli をウォッチモードで開始するスクリプトを定義してあるので、Markdown ファイルを保存すれば画像が出来上がります。

**図 1-5 VSCode で Dev Container を利用しながらの編集**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ee4dc51d1ac64643810028a78e785dc7/generate-ogimage-by-using-marp-edit-and-preview.png?auto=compress%2Cformat)

## 必要な環境

上記のカスタムテーマと Dev Container をローカルの PC で実行するには下記のような環境が必要です。

*   Docker
*   VSCode + Dev Containers 拡張機能

あるいは GitHub のアカウントがある場合は GitHub Codespaces でも利用できます。

## セットアップ

GitHub 上で下記のリポジトリをテンプレートとしてリポジトリ作成します。

@[card](https://github.com/hankei6km/test-marp-ogimage)

**図 3-1 ブラウザーでリポジトリを開き「Use this template」からリポジトリを作成**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/221c3f72d2e649a587922a676d8089ce/generate-ogimage-by-using-marp-template-repo.png?auto=compress%2Cformat)

リポジトリを作成したらローカルへクローンし VSCode で開きます。`devcontaienr.json` があるので Container で開きなおすかの確認の通知があると思うので開きなおしてください。 (通知がなければコマンドパレットから `Dev Containers: Reopen in Container` を入力します)

Dev Container が作成できたら依存するパッケージをインストールします。

**図 3-2 ターミナルを開きパッケージをインストール**

```shell-session
$ npm ci
```

## OG 画像を生成する

セットアップできたので OG 画像を生成してみます。

最初に、画像生成用に NPM スクリプトを開始しておきます。

**図 4-1 ターミナルを開きスクリプトを開始**

```shell-session
$ npm run watch

> test-marp-ogimage@1.0.0 watch
> marp --watch --output images/ --input-dir md/ --image png --theme-set ./themes/

[  INFO ] Converting 2 markdowns...
[  INFO ] md/sample01.md => images/sample01.png
[  INFO ] md/sample02.md => images/sample02.png
[  INFO ] [Watch mode] Start watching...
```

スクリプトが開始されたら、`md/` ディレクトリ内に下記のような Markdown ファイルを作成します。

**リスト 4-1 簡単なサンプル(`md/simple01.md`)**

```md
---
marp: true
theme: ogimage
class:
  - ogimage
  - lead
---

# **Marp** で **OG 画像**を生成してみる
```

基本的には通常の Marp 用ソースファイル(Front-matter + Markdown)ですが、テーマに今回作成した `ogimage` を指定しています。これにより生成成される画像が「OG 画像用のサイズ(1200px x 630px)[^ogimage-size]」となります。

[^ogimage-size]: どのサイズが正解なのかはわからなかったのですが、検索すると最近ではこのサイズが無難そうだったので。

ファイルを保存すると marp-cli が変更を検出し自動的に `images/` へ画像を生成します。

**図 4-2 生成された画像(`images/simple01.png`)**

![中央に「\*Marp で OG 画像を生成してみる」と描画されているサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/05023fac71954734a37dea213436c784/generate-ogimage-by-using-marp_simple01.png?w=1200\&h=630\&auto=compress%2Cformat)

実際に OG 画像として利用すると下記のような感じになります。

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/simple01)

ちょっとシンプルすぎますが、簡単な Markdown を記述するだけで OGP に利用できるそれっぽい画像を生成できました。

## いろいろな OG 画像を生成する

`ogimage` テーマにはいくつかの class とスタイルを設定してあります。また Marp では画像の扱いなどが拡張されているので、それらを利用していろいろな OG 画像を生成したいと思います。

### class の利用

Marp ではテーマに設定されている class を指定することで色や配置などを切り替えるこができます(ダークテーマに切り替えるような感じです)。

今回の `ogimage` テーマでは下記の class が利用できます。

*   `invert` `gaia` `ogimage` - 配色を変更する
*   `lead` - 配置を中央寄せにする
*   `clamp` - タイトル(見出し 1)を 3 行までの表示する

class の指定方法はいくつかありますが、画像を生成する場合は単一のページなので Front-matter の `class` フィールドを利用するのが簡単かと思います。

*   [Directives / Usage / Front-matter](https://marpit.marp.app/directives?id=front-matter)

**リスト 5-1 class を変更したサンプル**

```md
---
marp: true
theme: ogimage
class:
  - invert
  - clamp
---

# **class を変更**したサンプルです、全体的な**配色**が変化し、タイトルに長い文字列を指定しても **3 行まで**しか表示されないようになっています
```

**図 5-1 生成された画像**

![class を変更し、ダークモード的な配色、またタイトルは 3 行までになっているサンプル。](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7bdf7c908e3640c7a44533822047ec21/generate-ogimage-by-using-marp-class.png?w=1200\&h=630\&auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/class)

### Avatar アイコンを配置

`ogimage` テーマには画像をフチありの円形にするスタイルを定義してあります。画像の `alt` を `avatar` とすることで適用されます。また、Avatar などの部品的なものを画像下部の中央に配置するように `layout` のスタイルを変更してあります。

*   [Directives / Local directives/ Header and footer](https://marpit.marp.app/directives?id=header-and-footer)
*   [Theme CSS / Styling / Header and footer](https://marpit.marp.app/theme-css?id=header-and-footer)

GitHub に登録してあるプロファイル画像などで簡単にそれっぽい Avatar アイコンになるかと思います。また、テーマ(`.css` )を編集する必要がありますが、ロゴなども `header` と `footer` を使うと配置しやすいかと。

**リスト 5-2 Avatarアイコンを配置したサンプル**

```md
---
marp: true
theme: ogimage
class:
  - ogimage
  - lead
  - clamp
footer: |
  ![avatar](https://github.com/hankei6km.png) hankei6km
---

# **Avatar**を表示しているサンプル
```

**図 5-2 生成された画像**

![画像の下側中央に Avatar アイコンを描画しているサンプル。](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e76d0ff0b8fd4a668e6ba9d1d85c28ab/generate-ogimage-by-using-marp-avatar.png?w=1200\&h=630\&auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/avatar)

### 背景色を調整する

前述のように class である程度の配色を変更できますが、やはり「背景色をちょっと変更したい」ときはあるかと思います。Marp では `<style>` タグを使えるのでそちらを利用する方法もありますが、Directive を使うと Front-matter に記述するだけで変更できます。

*   [Directives / Local directives / Styling slide/ Backgrounds](https://marpit.marp.app/directives?id=backgrounds)

ページのカテゴリーなどで色分けするようなときに利用できるかと思います。

**リスト 5-3 背景色を変更したサンプル**

```md
---
marp: true
theme: ogimage
class:
  - gaia
  - lead
  - clamp
backgroundColor: darkred
footer: |
  ![avatar](https://github.com/hankei6km.png) hankei6km
---

# **背景色** を変更したサンプル
```

**図 5-3 生成された画像**

![背景色を \`darkred\` へ変更したサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6ccf906719b64f0ca81c9b2bf3124a60/generate-ogimage-by-using-marp-bgcolor.png?w=1200\&h=630\&auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/bgcolor)

### 画像を扱う

Marp では Markdown の記法で画像を表示するときに、リサイズやフィルターの指定ができるようになっています。

*   [Image syntax / Resizing image](https://marpit.marp.app/image-syntax?id=resizing-image)
*   [Image syntax / Image filters](https://marpit.marp.app/image-syntax?id=image-filters)

ドロップシャドウの適用などは OG 画像にスクリーンショットを含めるときに利用できるかと思います。

**リスト 5-4 サイズとドロップシャドウを指定したサンプル**

```md
---
marp: true
theme: ogimage
class:
  - ogimage
  - lead
---

# **Marp** で **OG 画像**を生成してみる

![height:400px drop-shadow](https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/screen_shot01.png)
```

**図 5-4 生成された画像**

![画像が指定されたサイズに縮小され、ドロップシャドウ付きで描画されているサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5da7f2de0b46427dacf829ab9ff7f8e3/generate-ogimage-by-using-marp-image.png?w=1200\&h=630\&auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/image)

::: message

Marp のデフォルト設定ではセキュリティ的な理由からローカルの画像ファイルを指定できません。設定を変更する場合は、下記の内容を参照してください。

*   [Security about local files](https://github.com/marp-team/marp-cli#security-about-local-files)

:::

::: message

画像に Animation GIF 形式を指定した場合、どのフレームが利用されるかはタイミング依存になるので注意してください。

:::

### 背景画像を指定する

Marp で背景画像を指定する方法としては主に「Markdown の画像表示で `alt` を利用する」「Directive に `backgroundImage` を記述する」があります。

*   [Image syntax / Slide backgrounds](https://marpit.marp.app/image-syntax?id=slide-backgrounds)
*   [Directives / Local directives / Styling slide/ Backgrounds](https://marpit.marp.app/directives?id=backgrounds)

位置の指定など利用できる機能が若干異なりますが、専用のフレーム画像(全体にフィットさせるような背景画像)なら `alt` 指定するのが簡単かと思います。

まず、下記のようなフレーム的な画像を背景にしてみます。

**図 5-5 フレーム的な画像**
![黒背景にガラス風スタイルを適用した不透明度 40 % の画像](https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/bg_image01.png)

**リスト 5-5 背景画像を変更したサンプル**

```md
---
marp: true
theme: ogimage
class:
  - gaia
  - lead
  - clamp
backgroundColor: darkred
footer: |
  ![avatar](https://github.com/hankei6km.png) hankei6km
---

![bg](https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/bg_image01.png)

# **背景画像**を指定したサンプル
```

**図 5-6 生成された画像**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0132ebea447d4963ad9a31337232d3be/generate-ogimage-by-using-marp-bgimage.png?auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/bgimage)

次に下記の写真画像にフィルターを適用しながら背景にしてみます。

**図 5-7 風景の写真画像**
![夕暮れときの河川の写真、少し白ぽい](https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/bg_image03.jpg)

**リスト 5-6 フィルターを適用した画像を背景にしたサンプル**

```md
---
marp: true
theme: ogimage
class:
  - gaia
  - lead
  - clamp
footer: |
  ![avatar](https://github.com/hankei6km.png) hankei6km
---

![bg bottom brightness:0.6 hue-rotate:300deg](https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/bg_image03.jpg)

# **背景画像**に**フィルター**を適用
```

**図 5-8 生成された画像**

![風景写真にフィルターを適用し背景画像にしたサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1ee3a05d02c146adba03bb0943e49b70/generate-ogimage-by-using-marp-bgimage-filter.png?w=1200\&h=630\&auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/bgimage-filter)

### 背景画像を分割表示

背景画像は複数並べたり、画像と本文を分割して表示させることもできます。

*   [Image syntax / Advanced backgrounds](https://marpit.marp.app/image-syntax?id=advanced-backgrounds)

分割表示では画像に本文が重ならないので、タイトルの見やすさを優先する場合に利用できるかと思います。

**リスト 5-7 背景画像を分割表示したサンプル**

```md
---
marp: true
theme: ogimage
class:
  - ogimage
  - clamp
footer: |
  ![avatar](https://github.com/hankei6km.png) hankei6km
---

![bg left:25%](https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/bg_image02.png)

# 背景画像を**分割表示**したサンプル
```

**図 5-9 生成された画像**

![左側に背景画像を描画し、右側に本文を描画しているサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a5c8163df92d42c097f99732fe5a1e42/generate-ogimage-by-using-marp-splitbg.png?w=1200\&h=630\&auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/splitbg)

### 通常の背景画像と分割表示

上記方法で背景画像を複数指定すると均等に並べての表示になります。通常の背景画像を別途表示させる場合は `backgroundImage` を併用することになります。

**リスト 5-8 通常の背景画像と分割表示したサンプル**

```md
---
marp: true
theme: ogimage
class:
  - gaia
  - clamp
backgroundImage: url("https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/bg_image01.png")
footer: |
  ![avatar](https://github.com/hankei6km.png) hankei6km
---

![bg left:25%](https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/bg_image02.png)

# **通常背景**と**分割表示**

通常の背景画像を表示しつつ、分割表示も行うサンプル。
```

**図 5-10 生成された画像**

![通常の背景画像を表示しつつ、分割表示で左側にも画像を表示しているサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/cc9f1b53efeb422a992376684a6cab24/generate-ogimage-by-using-marp-bgimage-splitbg.png?w=1200\&h=630\&auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/bgimage-splitbg)

### GitHub 用にサイズを変更する

GitHub のリポジトリにも OG 画像(ソーシャルイメージ)を設定できるのですが、[サイズが1280px x 640px になっています](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/customizing-your-repositorys-social-media-preview)。

`ogimage` テーマでは Front-matter の `size` フィールドで `github` を指定することで、このサイズへ変更できます。

**リスト 5-9 GitHub 用にサイズを指定したサンプル**

```md
---
marp: true
theme: ogimage
size: github
class:
  - gaia
  - lead
  - clamp
backgroundColor: black
footer: |
  ![avatar](https://github.com/hankei6km.png) hankei6km
---

![bg](https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/bg_image01.png)

# **GitHub** 用に**画像サイズ**を設定

hankei6km / test-marp-ogimage
```

**図 5-11 生成された画像**

![画像のサイズを 1280pt x  640pt へ変更したサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/060c437ce8dd44c490f28ac9a327b193/generate-ogimage-by-using-marp-size.png?w=1280\&h=640\&auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/size)

### フォント(スタイル)を変更

今回作成した環境ではデフォルトで `Lato` + `Roboto Mono` + `Noto Sans CJK JP` が利用されます。 (`Lato` と `Roboto Mono` は `gaia` テーマが [import しています](https://github.com/marp-team/marp-core/blob/a012c7b722843fd8e5b38b04fb32eb91a8866172/themes/gaia.scss#L19))

Marp ではフォントを変更する専用の手順はありませんが、Front-matter または Markdown 内に `<style>` を記述できます。また、(利用するテーマにもよりますが)`<section>` にフォントを適用すると全体で利用されます。

*   [Tweak style through Markdown](https://marpit.marp.app/theme-css?id=tweak-style-through-markdown)
*   [Tweak theme style](https://marpit.marp.app/directives?id=tweak-theme-style)

これを利用することで marp-cli(puppeteer)からアクセスできる(インストールされている)フォントを利用できるようになります。

今回の Dev Container は `fonts-liberation` `fonts-noto-cjk` `fonts-noto-cjk-extra` がインストールされています。 (`fonts-liberation` は `google-chrome-stable` をインストールすると依存関係でインストールされます)

その他のフォントを利用する場合は `Dockerfile` でインストールするか、`gaia` テーマがやっているように外部から読み込むことになります。

ここでは、既にインストールされているフォントの `serif` 体を利用してみます。

**リスト 5-10 フォントを変更したサンプル**

```md
---
marp: true
theme: ogimage
size: github
class:
  - gaia
  - lead
  - clamp
backgroundColor: black
style: |
  section {
    font-family: "Liberation Serif", "Noto Serif CJK JP", serif;
  }
footer: |
  ![avatar](https://github.com/hankei6km.png) hankei6km
---

![bg](https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/bg_image01.png)

# **フォント**の設定を**セリフ体**へ変更

hankei6km / test-marp-ogimage
```

**図 5-12 生成された画像**

![フォントの設定をセリフ体へ変更したサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f743ae8e78d6459a9117db64d80ed812/generate-ogimage-by-using-marp-font.png?w=1280\&h=640\&auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/font)

### タイトルのスタイルを変更する

タイトルのスタイルを変更するための専用の機能もないのですが、やはり変更したいことはそれなりにあるかと思います(背景画像へ埋もれないようにするなど)。

タイトルの場合はとくに捻るこもともなく `h1` タグにスタイルを適用すれば変更されます。ここでは、文字を大きくし斜めに傾けてみます。

**リスト 5-11 タイトルのスタイルを変更したサンプル**

```md
---
marp: true
theme: ogimage
class:
  - invert
  # - lead
  #- clamp
backgroundColor: black
style: |
  h1 {
    transform-origin: 30% 50%;
    transform: rotate(15deg);
    font-size: 4em;
  }

footer: |
  ![avatar](https://github.com/hankei6km.png) hankei6km
---

![bg](https://github.com/hankei6km/test-marp-ogimage/raw/main/md/assets/bg_image01.png)

# タイトルを**斜めに**傾ける
```

**図 5-13 生成された画像**

![タイトルのスタイルを変更したサンプル](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/214e345791864e019ae58042c74fe143/generate-ogimage-by-using-marp-rotate.png?w=1200\&h=630\&auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/rotate)

### 円グラフを表示(HTML + JavaScript の利用)

グラフを表示している OG 画像を見かけることもあるので、Marp で HTML を利用する方法の説明もかねて少し試してみます。

まず、HTML を有効化した状態で marp-cli を開始する必票があります。「**OG 画像を生成する**」で開始した NPM スクリプトを `Ctrl` + `C` などで停止し、下記のように再度開始します。

**図 5-14 HTML を有効化した状態で開始**

```shell-session
$ npm run watch -- --html
```

グラフの表示については、今回は Mermaid.js を利用してみます。

*   [Pie chart diagrams](https://mermaid-js.github.io/mermaid/#/pie?id=pie-chart-diagrams)
*   [Support the mermaid · Issue #125 · yhatt/marp · GitHub](https://github.com/yhatt/marp/issues/125#issuecomment-509485121)
*   [How to change pie-chart color？ · Issue #2582 · mermaid-js/mermaid · GitHub](https://github.com/mermaid-js/mermaid/issues/2582)

テーマ側と色などすり合わせる必要はありますが[^mermaid-pie-legend]、下記のように円グラフを表示できました。(セキュリティー的な注意は必要ですが)HTML を有効化することにより他のライブラリーなども利用できるかと思います。

[^mermaid-pie-legend]: Mermaid の円グラフは凡例を大きく出なかったので(フォントは大きくできるが表示領域が広がらない)、実際に使うには他のライブラリーが良いかもしれません。

**リスト 5-12 円グラフを表示するサンプル**

```md
---
marp: true
theme: ogimage
class:
  - ogimage
  - clamp
---

# **Mermaid** で円グラフ

<div class="mermaid">
%%{init: {'theme': 'forest', 'themeVariables': {  'pieSectionTextSize': '32px'} }}%%
pie showData
    "項目 1" : 50
    "項目 2" : 30
    "項目 3" : 10
</div>

<!-- mermaid.js -->
<script src="https://unpkg.com/mermaid@9.2.2/dist/mermaid.min.js"></script>
<script>mermaid.initialize({startOnLoad:true});</script>
```

**図 5-15 生成された画像**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/efb4d66f44b14bd2b739fd5951650463/generate-ogimage-by-using-marp-pie.png?auto=compress%2Cformat)

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/samples/pie)

## SSG での利用

生成した画像を実際に OGP で使うとどうなるのか気になったので、Next.js で簡単なページを作って GitHub Pages へデプロイしてみました。

基本的にはヘッドレスブラウザーで OG 画像を生成するのと違いはないのですが、Dev Container を流用しているので「デプロイしたらフォントが違くなってた」的な事態は避けることができています。

@[card](https://hankei6km.github.io/test-marp-ogimage-pages/)
@[card](https://github.com/hankei6km/test-marp-ogimage-pages)

最近ではエッジとファンクションで生成といった流れだと思いますが「SSG で生成するのもいいじゃない」という場合は Marp でもいけるかなと思います。

## おわりに

Marp を使って OG 画像を生成するカスタムテーマと Dev Container を作り、実際にいくつかの OG 画像を生成してみました。

これまでは Marp で画像を生成するときはサイズやフォントなどの調整をその都度なんとなくで行っていましたが、その辺の手間を解消できそうかなと思っています。
