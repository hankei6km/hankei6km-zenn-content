---
id: edit-pitctures-with-imgix
title: imgix が好き
emoji: "\U0001F5BC️"
type: idea
topics:
  - imgix
  - image
published: true
---
突然ですが [imgix](https://imgix.com/) が好きです。

Headless CMS から [Rendering API](https://docs.imgix.com/apis/rendering) のみの利用ですが[^cms-imgix]「[パラメーターを作成するウェブアプリ](https://github.com/hankei6km/image-url-workbench)」「[WYSIWYG エディターからパラメーターを指定するツール](https://hankei6km.github.io/rehype-image-salt-doc/)」を作るくらいに好きです。

以前にも「[imgix Rendering API Showcase](https://hankei6km.github.io/mardock/deck/try-imgix-rendering-api)」スライドを作ったりしたのですが、せっかく Zenn を始めたのでその辺をもう少し書いてみようかと。

なお、今回は imgix を使った(個人ブログなどでの)画像の加工をとりとめもなく書いているだけです。画像の最適化(均一化)などの実用的な情報は以下の記事が詳しく解説されています。

@[card](https://zenn.dev/manalink/articles/manalink-imgix)

[^cms-imgix]: [microCMS](https://microcms.io/) と [Prismic](https://prismic.io/) が画像バックエンドに imgix を利用しています。

## どこが好き

[ジョギングのメモ](https://hankei6km.github.io/mardock/deck/category/jog)を作っているのですが「写真はスマホでなんとなく撮影している」状態です。なので写真を利用するときに「ちょっとだけ修正したい」的なことが発生します。

そんなときに imgix がいろいろを手助けしてくれます。

また、今時は [nuxt-image](https://image.nuxtjs.org/) などがあるので意識することは少ないですが「1 枚の画像からクエリーパラメーターだけで複数サイズの画像を生成できる」点もありがたいです。

## 画像をちょっと調整したい

以下は「思っていた写真とちょっと違う」というときに使っているパラメーターを指定したときのサンプルです。

:::message

この記事全体で `auto=compress,format` とサイズ指定がされているのでオリジナル画像も画質は若干低下しています。

:::

### Automatic で良さげにする

[Automatic](https://docs.imgix.com/apis/rendering/auto) は主に画像を圧縮したりフォーマットを環境にあわせて選択させることに使われますが、他にも [`enhance`](https://docs.imgix.com/apis/rendering/auto/auto#enhance) などがあります。

[`enhance`](https://docs.imgix.com/apis/rendering/auto/auto#enhance) は大雑把にいうと「画像を良さげにする」指定です。Rendering API には [Adjustment](https://docs.imgix.com/apis/rendering/adjustment) などもありますがこれはこれで手間がかかります。

「ちょっと暗めの写真になってしまった」というときなどに [`enhance`](https://docs.imgix.com/apis/rendering/auto/auto#enhance) なら指定するだけで大体良さげな画像になります(個人の感想です)。

![enhance 用の画像(オリジナル)](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/62f516d903e540a5a09243f905acc411/edit-pitctures-with-imgix-enhance.jpg?w=600\&h=300\&auto=compress%2Cformat)*オリジナル*

![enhance 用の画像(パラメーター指定)](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/62f516d903e540a5a09243f905acc411/edit-pitctures-with-imgix-enhance.jpg?w=600\&h=300\&auto=compress%2Cformat%2Cenhance)*auto=enhance*

![enhance 用の画像(オリジナル)](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f62544aa8a2d419dabb9ff52208405d6/edit-pitctures-with-imgix-focalpoint.jpg?w=600\&h=300\&auto=compress%2Cformat)*オリジナル*

![enhance 用の画像(オリジナル)](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f62544aa8a2d419dabb9ff52208405d6/edit-pitctures-with-imgix-focalpoint.jpg?w=600\&h=300\&auto=compress%2Cformat%2Cenhance)*auto=enhance*

### Rotation で傾きを修正する

読んで字のごとくですが「ちょっと傾いてしまった」というときに [`rot`](https://docs.imgix.com/apis/rendering/rotation/rot) を使っています。以下はわかりやすく回転させています。

![Rotation 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a661a61f54ec4b5288333cce3e5d3949/edit-pitctures-with-imgix-rotation.jpg?w=600\&h=300\&auto=compress%2Cformat)*オリジナル*

![Rotation 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a661a61f54ec4b5288333cce3e5d3949/edit-pitctures-with-imgix-rotation.jpg?w=600\&h=300\&rot=340)*rot=340*

### Focal Point Crop で任意の位置を切り出す(ズームする)

ジョギングのメモでは写真を背景画像にしてテキストをかぶせているので「建物などが重なってテキストが読みにくい」といときがたまにあります。そんなときに利用しています[^rect]。

これは手順がちょっとわかりにくいのですが、以下のような感じで指定します。

[Size](https://docs.imgix.com/apis/rendering/size) 指定で [Fit mode](https://docs.imgix.com/apis/rendering/size/fit) を [`crop`](https://docs.imgix.com/apis/rendering/size/fit#crop)、[Crop Mode](https://docs.imgix.com/apis/rendering/size/crop) を [`focalpoint`](https://docs.imgix.com/apis/rendering/size/crop#focalpoint) にします(この時点で画像は変化しない)。

![focalpoint 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f62544aa8a2d419dabb9ff52208405d6/edit-pitctures-with-imgix-focalpoint.jpg?w=600\&h=300\&auto=compress%2Cformat%2Cenhance\&fit=crop\&crop=focalpoint)*fit=crop\&crop=focalpoint*

[Focal Point Zoom](https://docs.imgix.com/apis/rendering/focalpoint-crop/fp-z) でズームします。

![focalpoint 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f62544aa8a2d419dabb9ff52208405d6/edit-pitctures-with-imgix-focalpoint.jpg?w=600\&h=300\&auto=compress%2Cformat%2Cenhance\&fit=crop\&crop=focalpoint\&fp-z=1.8)*fit=crop\&crop=focalpoint\&fp-z=1.8*

[Focal Point X Position](https://docs.imgix.com/apis/rendering/focalpoint-crop/fp-x) と [Focal Point Y Position](https://docs.imgix.com/apis/rendering/focalpoint-crop/fp-y) でズームする領域を変更します。

![focalpoint 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f62544aa8a2d419dabb9ff52208405d6/edit-pitctures-with-imgix-focalpoint.jpg?w=600\&h=300\&auto=compress%2Cformat%2Cenhance\&fit=crop\&crop=focalpoint\&fp-z=1.8\&fp-x=0.68\&fp-y=0.55)*fit=crop\&crop=focalpoint\&fp-z=1.8\&fp-x=0.68\&fp-y=0.55*

:::message

画像を切り出して表示した後もサーバー上にはオリジナルのファイルが残っています。

パラメーター除去して表示すればオリジナル画像が表示できるので注意してください。

:::

[^rect]: 似たようなことは [Source Rectangle Region](https://docs.imgix.com/apis/rendering/size/rect) でもできます。

## フィルター的に見た目を変更する

以下は「imgix だとこんなこともできるよ」というショーケース的なパラメーターの指定です。

「CSS で指定できる」項目もありますが、blur などは imgix 側で加工しておくと「転送量を減らせる」という利点もあります。

### blur

![blur 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/abb6bbc126ec4e62a4dd5dfb2d6c0d11/edit-pitctures-with-imgixi-blur.jpg?w=600\&h=300\&auto=compress%2Cformat)*オリジナル*

![blur 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/abb6bbc126ec4e62a4dd5dfb2d6c0d11/edit-pitctures-with-imgixi-blur.jpg?w=600\&h=300\&blur=50)*blur=50*

### hue shift

![blur 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c530685f5e2945c583ad41322600fc49/edit-pitctures-with-imgix-hue-shift.jpg?w=600\&h=300\&auto=compress%2Cformat)*オリジナル*

![blur 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c530685f5e2945c583ad41322600fc49/edit-pitctures-with-imgix-hue-shift.jpg?w=600\&h=300\&hue=316)*hue=316*

### vibrance

![vibrance 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ea854b9a431440c7a585b3cdd8bea587/edit-pitctures-with-imgix-vibrance.jpg?w=600\&h=300\&auto=compress%2Cformat)*オリジナル*

![vibrance 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ea854b9a431440c7a585b3cdd8bea587/edit-pitctures-with-imgix-vibrance.jpg?w=600\&h=300\&vib=100)*vib=100*

### sepia tone

![sepia tone 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1a3ee1b2642d4fd9b729f1ddb9d6b8ec/edit-pitctures-with-imgix-sepia-tone.jpg?w=600\&h=300\&auto=compress%2Cformat)*オリジナル*

![sepia tone 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1a3ee1b2642d4fd9b729f1ddb9d6b8ec/edit-pitctures-with-imgix-sepia-tone.jpg?w=600\&h=300\&sepia=80)*sepia=80*

### monochrome

![monochrome 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7705ec3d5d804ffa903a7bb97f244168/edit-pitctures-with-imgix-monochrome.jpg?w=600\&h=300\&auto=compress%2Cformat)*オリジナル*

![monochrome 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7705ec3d5d804ffa903a7bb97f244168/edit-pitctures-with-imgix-monochrome.jpg?w=600\&h=300\&monochrome=ff4a4a4a)*monochrome=ff4a4a4a*

### duo tone

![duo tone 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3fd1988e07af41fa98a1d7abf3287564/edit-pitctures-with-imgix-duo-tone.jpg?w=600\&h=300\&auto=compress%2Cformat)*オリジナル*

![duo tone 用の画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3fd1988e07af41fa98a1d7abf3287564/edit-pitctures-with-imgix-duo-tone.jpg?w=600\&h=300\&duotone=000080%2CFA8072\&duotone-alpha=100)*duotone=000080,FA8072\&duotone-alpha=100*

## パラメーターを作るのは大変なのでは

クエリーパラメーターだけで画像を加工できるのが Rendering API の良さですが、写真ごとに手動でパラメーターを作るのはやはり大変です。

### ImageURL Workbench

そんなときのために「[ImageURL Workbench](https://image-url-workbench.vercel.app/)」を作成しました(宣伝)。

[![ImageURL Workbench でパラメーターを指定し画像をプレビューしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/959ee4f71196411d9c01994f799ac703/edit-pitctures-with-imgix-image-url-workbench.png?w=600\&h=383\&auto=compress%2Cformat)*ImageURL Workbench*](https://image-url-workbench.vercel.app/)

@[card](https://image-url-workbench.vercel.app/)

プレビューや画像サイズなどを確認しながらパラメーターを指定できます。

![ImageURL Workbench で画像のプレビューやサイズを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/959ee4f71196411d9c01994f799ac703/edit-pitctures-with-imgix-image-url-workbench.png?w=600\&h=383\&rect=50%2C80%2C600%2C430)*プレビューとサイズ表示*

![ImageURL Workbench でパラメーターを指定している領域のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/959ee4f71196411d9c01994f799ac703/edit-pitctures-with-imgix-image-url-workbench.png?w=600\&h=383\&rect=650%2C100%2C600%2C600)*パラメーターを指定*

### レスポンス対応のタグを作るデモ

@[youtube](Nj6RsEriwzQ)

### 顔検出を利用したデモ

@[youtube](p6C0qZHndz8)

## おわりに

写真ごとにクエリーパラメーターでポチポチ編集するのは手間ですが、ホビーでの利用ならそういうのも楽しみの 1 つなのかなと。

という感じで「やっぱり imgix が好き」を再確認した記事でした。

なお、この記事は「imgix はスクリーンショットでも便利に使える(仮)」に続く予定です。
