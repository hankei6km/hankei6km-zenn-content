---
id: no-break-space-busters
title: No-Break Space Busters
emoji: "\U0001F575️"
type: idea
topics:
  - cli
  - vscode
  - vim
published: true
---
ちょっとタイトルが紛らわしいですが `&nbsp;` で見た目を調整するのはやめましょう的な内容ではありません[^title]。

[^title]: タイトルは UI 上の文字化けを解決するツールをもじってつけました。

[Headless CMS の RichText 系フィールドを調べていたとき](https://zenn.dev/hankei6km/articles/test-collage-cms-content)に「連続した空白をコピペしたらどうなる？」などを試していたら、あちこちに `U+00A0` 文字が入り込んで頭を抱えました。

今回はそのときに使った「`U+00A0` の存在を調べる」方法などのメモになります(`&nbsp;` と実態参照で記述されている分にはよいのですが、文字として入力されていると普通の空白と見分けがつきにくい)。

## サーチ

`U+00A0` を表示や検索する方法です。

### VSCode

#### 表示

(おそらく Ver.1.63 から)`U+00A0` などはデフォルトでハイライト表示されます。

![VSCode で No-Break Space が黄色く表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3537b2501d0b4714a58044edea2d1690/no-break-space-busters-disp-in-vscode.png?w=600\&h=356\&auto=compress%2Cformat)*VSCode での表示*

> Unicode highlighting All uncommon invisible characters in source code are now highlighted by default:

@[card](https://github.com/microsoft/vscode-docs/blob/vnext/release-notes/v1_63.md#unicode-highlighting)

#### 検索

Find(Ctrl + F)で正規表現を有効にし `\u00a0` を入力します。これは Search(Ctrl + Shift + F)や [VSCodeVim](https://github.com/VSCodeVim/Vim) の `/` でも同じです。

### Vim

#### 表示

デフォルトでは通常の空白と同じように表示されます。

![Vim で No-Break Space が判別できないスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6b83c950434c491089eb968ee2f9c14d/no-break-space-busters-disp-in-vim-before.png?auto=compress%2Cformat\&w=600\&h=514\&rect=500%2C64%2C700%2C600)*Vim のデフォルト表示*

[`listchars`](http://vimdoc.sourceforge.net/htmldoc/options.html#'listchars') で `nbsp` を設定すると区別できます。

```vim
:set listchars=nbsp:&
:set list
```

![Vim で No-Break Space が "&" として表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7ac38c7952ab4b6bb507aeb6f7b6d458/no-break-space-busters-disp-in-vim-after.png?auto=compress%2Cformat\&w=600\&h=514\&rect=500%2C64%2C700%2C600)*listcharsを設定した表示*

#### 検索

`/` で `\%u00a0`

### CLI

No-Break Space が含まれる行を表示します。

```shell-session
$ grep -H $'\u00a0' -- *.md
memo.md:  input: './src/index.ts',
memo.md:  output: {
memo.md:    file: './dist/index.cjs',
memo.md:    format: 'cjs',
memo.md:    exports: 'default'
memo.md:    },
memo.md:  plugins: [
memo.md:    typescript({
memo.md:      //module: 'commonjs'
memo.md:    }),
memo.md:    nodeResolve(),
memo.md:    commonjs({
memo.md:      // extensions: ['.js', '.ts']
memo.md:    })
memo.md:  ]
memo.md:  node1["start"] --> node2["end"]
```

@[card](https://stackoverflow.com/questions/602912/how-do-you-echo-a-4-digit-unicode-character-in-bash)

## デストロイ

エディターの場合は検索の要領で文字を入力し置換します。

CLI では `sed` などで対応できます。

```shell-session
$ sed -e 's/'$'\u00a0''/ /g' -i -- *.md
```

## ちなみに

Zenn 記事を書く場合「mermaid 用のコードブロック内に `U+00A0` があったらどうなる？」は気になるところですが、軽く試した感じでは「インデントで使ってもエラーにならなかった」です。

危ないので実際には普通の空白文字を使いますが。

## おわりに

冒頭の問題はとりあえず対応できました。

しかし、No-Break Space の他にも「なんとか Space」がいくつかあったりします。「文字コード関連は奥が深い」と毎回痛感いたします。
