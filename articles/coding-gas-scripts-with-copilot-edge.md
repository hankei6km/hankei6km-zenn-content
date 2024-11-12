---
id: coding-gas-scripts-with-copilot-edge
title: GAS のスクリプトエディターでも Copilot を使いたい
type: idea
topics:
  - googleappsscript
  - edge
  - copilot
published: true
---

GAS のちょっとしたスクリプトを試したいとき、スクリプトエディターは気軽につかえて重宝しています。しかし、AI による支援機能がないため、少し厳しく感じるようにもなってきました。

[Google Colab](https://colab.research.google.com/?hl=ja) では編集支援などで Gemini が使えるようになったので GAS にも期待したいところですが、現状では対応されていません。そんなこんなで「GAS のエディターでも ~~楽したい~~ 効率良く記述したい」と考えていたところ、Edge ブラウザーにも Copilot がいることを思い出しました。

## Edge の Copilot

Edge ブラウザーには「表示中のページをコンテキストとして Copilot と会話できる」モードがあります。これを使えば「スクリプトエディター内のコードについても質問できるのは？」と試してみました。

**図 1-1 Edge の中で Copilot を利用**

![Edge ブラウザーのスクリーンショットに Copilot を開始するためのアイコンを示した赤枠を追加してある(アイコンは Edge ブラウザーの右上に表示されている)](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3461d8ef32d740d5af4d86627ec5f227/coding-gas-scripts-with-copilot-edge-in-edge.png?w=1440\&h=873\&auto=compress%2Cformat)

## GAS のコードについて会話してみる

基本的な操作は以下のようになります。

1.  Edge でスクリプトエディターを開く
2.  普通にコードを記述する
3.  会話したくなったらブラウザー右上の Copilot ボタンをクリック
4.  プロンプトでコードを考えてもらったりする

**図 2-1 GASのスクリプトエディターで編集しているコードについて会話**

![GAS のスクリプトエディターを Edge ブラウザーで開き、Copilot を使って編集中のコードについて会話しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2b35d5867bf749a79440f27f58abed5c/coding-gas-scripts-with-copilot-edge-side-gas.png?w=1440\&h=873\&auto=compress%2Cformat)

Copilot 側からコードを直接編集したりはできませんが、編集中のコードに対して「"menuFunction1" 関数のコードをコメントに沿って作成してください」的な質問は通るのでわりと便利です。

ただし、スクリプトエディターの変更内容を認識できないときがあるようなので、その場合は手動で Copilot をリロードする必要はありました。

**図 2-2 Copilot 側でリロード操作が必要なときもある**

![Copilot が表示されているパネル上部のボタン(リロード)を赤枠で強調しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f7b298a5920c42678789c4ca0643a523/coding-gas-scripts-with-copilot-edge-reload.png?w=396\&h=267\&auto=compress%2Cformat)

専用のチャットボットではないので少し手間もありますが、簡単なコードを記述していく場合によい感じかと思います。

## 考慮点

### ページ読み取りの許可

上記のこと実施する場合、表示しているページの内容へのアクセスを Copilot へ許可することになります。

この設定は Edge の「Copilot とサイドバー / アプリと通知の設定 / Copilot」からオフにできるのですが、サービス(ドメイン)別の指定はできないようです。

**図 3-1 Copilot の設定項目**

![設定画面を開き、アクセス許可関連の項目を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e4bcc7a3113d455b9702e81b90a5ca91/coding-gas-scripts-with-copilot-edge-grant.png?w=895\&h=213\&auto=compress%2Cformat)

スクリプトエディターだけ許可したいようなときは、アカウントを分離するなどの対応が必要になるかと思います。

## その他の組み合わせ

原理的には「ウェブブラウザーで動くエディターなら GAS 以外でもいけるかな」と思ったので TypeScript などのプレイグラウンドを試してみたところ「それっぽい回答」が返ってきました。

**図 4-1 TypeScript のプレイグラウンドで Copilot を利用**

![Edge ブラウザーで TypeScript プレイグラウンドを開き Copilot で編集中コードについて会話しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8a0d60dd1d6d43239fc5e60944a7f3c4/coding-gas-scripts-with-copilot-ts.png?w=1440\&h=873\&auto=compress%2Cformat)

TS Config の設定を考慮しないなどもありますが、ちょっと実験するときの編集支援としてはよい感じかと思いました。

**リスト 4-1 Target が ES2017 のときに生成されたコード(ES2017 で ****`Array.prototype.flat`**** は使えない)**

```typescript
function test1(a: number[][]): number[] {
  // 入力: [[2,3,1],[6,0,3,8],[5,3]]
  // 出力: [20,30,10,60,0,30,80,50,30]

  // 各サブ配列の要素を10倍にしてフラットな配列に変換
  return a.flat().map(num => num * 10);
}

// テストケース
console.log(test1([[2, 3, 1], [6, 0, 3, 8], [5, 3]])); // [20, 30, 10, 60, 0, 30, 80, 50, 30]
console.log(test1([[], [30, 5, 6, 1], [9, 23]])); // [300, 50, 60, 10, 90, 230]
```

この他にも Rust のプレイグラウンドで少し試しましたが、エディターで編集中のコードは認識するようです。すべての場合でうまくいくとも限らないですが、ブラウザーで動作しているエディターを使っているとき試してみると楽できるかもしれません。

::: message

Edge の Copilot は 2024 年の夏から秋頃にアップデートされました。この前のバージョンでは以下のような制限がありました。

*   GAS のエディターで大きいコードを編集していると全体を認識できなかった
*   TypeScript や Rust のプレイグラウンドのエディターでは動作しなかった(コードを認識しなかった)

今後も Copilot の更新などにより利用可否の状況は変化するかもしれません。

:::

## おわりに

GAS のスクリプトエディターで AI による支援機能を使う方法として Edge の Copilot を試してみました。

少し前から、GAS 独自の機能について実験するときに利用しているのですが、記述中のコードを元に会話できるのは便利だなと感じています。

ただし、コードを直接編集できないのはやはり手間なので、「GAS のエディターでも Gemini が使えるようになってほしい」とも思っています。
