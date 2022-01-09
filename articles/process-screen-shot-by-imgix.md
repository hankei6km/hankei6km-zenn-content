---
id: process-screen-shot-by-imgix
title: imgix はスクリーンショットの加工でも便利
emoji: ✂️
type: idea
topics:
  - imgix
  - image
published: true
---

「[imgix](https://www.imgix.com/) は写真の最適化に使うもの」という先入観がなんとなくありました。

しかし、最近になって「Zenn 記事用のスクリーンショットを imgix で扱う」ようになり「imgix はスクリーンショットの加工でも使い勝手がよいのでは」と感じることが増えました。

[写真の加工については少し書いたので](https://zenn.dev/hankei6km/articles/edit-pitctures-with-imgix)、今回はそのスクリーンショットの加工について書いてみます。

## 特定位置の切り出し

imgix はサーバー上のファイルを GET する時にクエリーパラメーターを指定することで加工された画像を取得できます。

この特性を利用すると 1 枚のスクリーンショット(図 1-1)から複数の部分的なスクリーンショットを [Source Rectangle Region](https://docs.imgix.com/apis/rendering/size/rect) で切り出せます(図 1-2、図 1-3)。

![切り出しのサンプル、オリジナル画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8b9d100bcdb54fd7a7de5606852e2f78/process-screen-shot-by-imgix-overview.png?w=600\&h=383\&auto=compress%2Cformat)*図 1-1 オリジナル画像 - 全体表示*

![切り出しのサンプル、左上を切り出した画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8b9d100bcdb54fd7a7de5606852e2f78/process-screen-shot-by-imgix-overview.png?auto=compress%2Cformat\&w=600\&h=430\&rect=40%2C80%2C600%2C430)*図 1-2 rect=40,80,600,430で左上を切り出し*

![切り出しのサンプル、右上を切り出した画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8b9d100bcdb54fd7a7de5606852e2f78/process-screen-shot-by-imgix-overview.png?w=600\&h=600\&auto=compress%2Cformat\&rect=650%2C100%2C600%2C600)*図 1-3 rect=650,100,600,600で右上を切り出し*

また、tmux の pane を切り出したいときも大雑把にスクリーンショットをとってから(図 1-4)、パラメーターで調整するといったことも可能です(図 1-5)。

![切り出しのサンプル、大雑把な位置決めてとったスクリーンショットの画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f29f5e54770c46d2af4fe504b1b700f0/process-screen-shot-by-imgix-rough-cut.png?w=600\&h=203\&auto=compress%2Cformat%2Cenhance)*図 1-4 オリジナル - 大雑把なスクリーンショット*

![切り出しのサンプル、ボーダー位置の内側を画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f29f5e54770c46d2af4fe504b1b700f0/process-screen-shot-by-imgix-rough-cut.png?w=600\&h=203\&auto=compress%2Cformat%2Cenhance\&rect=14%2C14%2C600%2C200)*図 1-5 rect=14,14,600,200で切り出し*

:::message
画像を切り出して表示した後もサーバー上にはオリジナルのファイルが残っています。

パラメーターを除去して表示すればオリジナルの画像が表示できるので注意してください。
:::

## ボーダーとパディング

前節の「図 1-2」は背景色が白に近いため画像の境界がわかりにくいです。そのようなときに[ボーダーを追加](https://docs.imgix.com/apis/rendering/border-and-padding/border)できます(図 2-1)。

![ボーダーのサンプル、ボーダーを追加した画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8b9d100bcdb54fd7a7de5606852e2f78/process-screen-shot-by-imgix-overview.png?w=600\&h=430\&auto=compress%2Cformat\&rect=40%2C80%2C600%2C430\&pad=8\&border=1%2C55000000\&border-radius=1)*図 2-1 pad=8\&border=1,55000000\&border-radius=1でボーダー追加*

また、切り出したスクリーンショットの余白が狭いときは(図 2-2)、[背景色を指定](https://docs.imgix.com/apis/rendering/fill/bg)しながら[パディングの追加](https://docs.imgix.com/apis/rendering/border-and-padding/pad)もできます(図 2-3)。

ただし、余白的な背景色を指定するには注意点があります。[`Automatic`](https://docs.imgix.com/apis/rendering/auto/auto) パラメーターなどの加工により画像の色が変更されることもあるため、背景色を指定する場合は「実際に表示されている色」をスポイトツールなどで調べる必要があります[^palette]。また、拡大するとどうしても境目が目立つこともあります。

[^palette]: imgix の [Color Palette](https://docs.imgix.com/apis/rendering/color-palette) で調べることもできます。

![パディングのサンプル、切り出した画像の余白が狭い画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f29f5e54770c46d2af4fe504b1b700f0/process-screen-shot-by-imgix-rough-cut.png?w=600\&h=203\&auto=compress%2Cformat%2Cenhance\&rect=14%2C14%2C600%2C200)*図 2-2 オリジナル表示 - 狭い余白*

![パディングのサンプル、パディングを追加した画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f29f5e54770c46d2af4fe504b1b700f0/process-screen-shot-by-imgix-rough-cut.png?w=600\&h=203\&auto=compress%2Cformat%2Cenhance\&rect=14%2C14%2C600%2C200\&bg=ff252525\&pad-left=4)*図 2-3 bg=ff252525\&pad-left=4でパディング追加*

## テキスト

画像にちょっとしたテキストを追加したい場合は、[Text](https://docs.imgix.com/apis/rendering/text) を利用できます(図 3-1)。

![テキストのサンプル、簡単なテキストをオーバーレイ表示させた画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2cedc61358844c938d9dfc28aa3102af/process-screen-shot-by-imgix-text.png?w=600\&h=257\&auto=compress%2Cformat\&txt=%3Aset+listchars%3Dnbsp%3A%26\&txt-color=ffffffff\&txt-pad=20\&txt-size=40)*図 3-1 txt=%3Aset%20listchars%3Dnbsp%3A%26 などでテキスト追加*

ただし、日本語や emoji などもある程度指定できますが、エンコードの問題もあるのでクエリーパラメーターにするには [Base64 Variants](https://docs.imgix.com/apis/rendering#base64-variants) を使うなどの対応が必要です(図 3-2)。

![テキストのサンプル、emoji を含むテキストを追加した画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2cedc61358844c938d9dfc28aa3102af/process-screen-shot-by-imgix-text.png?w=600\&h=257\&auto=compress%2Cformat\&txt64=44OG44K544OI4pyP77iP\&txt-color=ffffffff\&txt-pad=20\&txt-size=40)*図 3-2 txt64=44OG44K544OI4pyP77iP でテキスト追加*

また、日本語のフォントを指定できないようなので日本語での説明文の追加などには向いていません(図 3-3、図 3-4)。

![テキストのサンプル、パラメーターとして「テスト、直径」と指定しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2247512ec81a46c5ac2060b5df5532d8/process-screen-shot-by-imgix-font.png?auto=compress%2Cformat\&border=1%2C55000000\&border-radius=1)*図 3-3 「テスト、直径」と指定している*

![テキストのサンプル、日本語のフォントではないため「直径」などのテキスト表示に違和感がある画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2cedc61358844c938d9dfc28aa3102af/process-screen-shot-by-imgix-text.png?w=600\&h=257\&auto=compress%2Cformat\&txt64=44OG44K544OI44CB55u05b6E\&txt-color=ffffffff\&txt-pad=20\&txt-size=40)*図 3-4 違和感のある表示になる*

## 見やすくする

ターミナルのスクリーンショット(図 4-1)はそのままだと文字が判別しにくかったので、実は [`enhance`](https://docs.imgix.com/apis/rendering/auto/auto#enhance) パラメーターで加工していました(図 4-2)。

![enhanceのサンプル、文字などが暗くて判別しにくい画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f29f5e54770c46d2af4fe504b1b700f0/process-screen-shot-by-imgix-rough-cut.png?w=600\&h=203\&rect=14%2C14%2C600%2C200\&bg=ff121212\&pad-left=4)*図 4-1 オリジナル表示 - 文字が判別しにくい*

![パディングのサンプル、パディングを追加した画像](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f29f5e54770c46d2af4fe504b1b700f0/process-screen-shot-by-imgix-rough-cut.png?w=600\&h=203\&auto=compress%2Cformat%2Cenhance\&rect=14%2C14%2C600%2C200\&bg=ff252525\&pad-left=4)*図 4-2 auto=enhanceで見やすくしてました*

## 画像 URL の文字数制限

Zenn 記事の場合は画像プロキシが入るので画像 URL の文字数に制限があります[^limit]。上記の例ではテキストの追加サンプル(図 3-1 など)はわりとギリギリなところまで行っています。

もしも制限を超えてしまう場合は、加工された画像をダウンロードし再アップロードすることで対応できます(クエリーパラメーターを指定しなくてよい分だけ URL を短縮できる)。

なお、画像をダウンロードする場合は「`auto` などの圧縮関連のパラメーターは外す」ようにしないと画像が劣化してしまうので注意が必要です。

[^limit]: <https://github.com/zenn-dev/zenn-community/issues/316> <https://support.cloudinary.com/hc/en-us/articles/207746165-Why-do-I-get-a-public-ID-too-long-error-when-trying-to-fetch-from-a-remote-URL->

## おわりに

今回あらためて記事にしてみて「クエリーパラメーターでのスクリーンショットの加工はやはり便利」と再確認できました。

とくに[記事の作成に Headless CMS を使っている](https://zenn.dev/hankei6km/articles/writing-article-with-headless-cms)関係から「画像を調整してからアップロード」を繰り返さないですむのは使い勝手が良いと言えます。

そのようなわけで、今後はスクリーンショットの加工にも適度に使っていければと思っています[^undocument]。

[^undocument]: 加工をやりすぎると文字数の制限以外にも「今はたまたまうまくいってるけど仕様上はアウト」みたいなものを踏む可能性もあるので程々に。
