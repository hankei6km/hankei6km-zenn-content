---
id: vscode-profiles
title: VSCode の Profiles 機能で拡張機能や設定を共有してみる
emoji: 🚚
type: tech
topics:
  - vscode
published: true
---

VSCode の設定を簡単に切り替える方法はないかなと思っていたところ、2023-01(v1.75.0) の更新で Profiles 機能が追加され、複数 Profile を持てようになったとのこと。

@[card](https://code.visualstudio.com/updates/v1_75#_profiles)

そこで、どんな感じかなと試してみたところ、設定の切り替え以外にも共有などで便利そうだったのでその辺についてなど。

*   2023-02-10: 通常版の vscode.dev でもプレビューを利用できるようになったので、注意点などを更新しました
*   2023-02-10: Profile の切り替えについての追記などを行いました

## Profiles 機能とは？

これまで VSCode の UI ではユーザーの設定を 1 つしか持てませんでした。

今回の更新で導入された Profies 機能を使うとその辺を良い感じにしてくれるとのこと。

*   [Profiles](https://code.visualstudio.com/updates/v1_75#_profiles)

<!-- textlint-disable -->

> If you have different VS Code setups based on workflow such as "Work" or "Demo", you can also save those as different profiles. You can open multiple workspaces (folders) with different profiles applied simultaneously.

<!-- textlint-enable -->

また、Profile をエクスポートする機能ではローカルにファイルを保存するだけでなく、Gist を使って他のユーザーとの共有もしやすくなっているそうです。

> Users that receive the profile link can preview the shared profile in [VS Code for the Web](https://code.visualstudio.com/docs/editor/vscode-web) and import it to their local VS Code instance.

## Profile を作ってみる

Profiles の機能は歯車アイコンにメニューが追加されていました(アカウントアイコンに追加されてると勘違いして少し探してしまいました)。

**図 2-1 Profiles のメニュー項目**

![VSCode の左下にある歯車アイコンをクリックしメニューの中に Profile 項目を確認しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/8dfb725864704aa8a355ee560eab0647/vscode-profiles-menu.png?w=367\&h=315\&auto=compress%2Cformat)

`Profiles` 項目の中に `Create Profile...` があるので選択すると、空の Profile か、現在の Profile を元にするか選択するようになります。どちらを使っても問題ないですが、Profiles の機能を試す場合は空の Profile の方が挙動を確認しやすいかと思います。

**図 2-2 Profile のベースを指定**

![作成する Profile のベースを選択するリストが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/fde37a29101b44189fd6ed26d5d8d905/vscode-profiles-cerate.png?w=641\&h=107\&auto=compress%2Cformat)

作成すると `Profile` メニューの中に項目が追加されます(今回は `demo1` を追加しました)。

**図 2-3 追加された Profile**

![Profiles のメニューを開き \`demo1\` 項目が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/761ae54d74bd4ed0a40b945aacfa06c7/vscode-profiles-created-demo1.png?w=526\&h=532\&auto=compress%2Cformat)

## Profile 別の設定

Profile を切り替えた状態で拡張機能をインストールしたり、`ctrl` + `,` などで設定を変更することでカレントの Proile に反映されます。また、設定画面では右上の回転させるようなアイコンをクリックすると、Profile 別の `settings.json` をエディターで開くこともできます。

**図 3-1 Profile 別の JSON を開く項目**

![設定画面の JSON を開くアイコンをクリックし、Profile 用の JSON を開く項目が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/0d0cde2040e345c8a6731544b83f76d5/vscode-profiles-current-openjson.png?w=369\&h=179\&auto=compress%2Cformat)

ただし、すべての項目が Profile 別に設定できるわけではないようです。たとば Dev Containers 拡張機能の `Docker path` などはユーザー単位でしか設定できませんでした。

設定できない項目は UI 上で `Applies to all profiles` と表示されるようです。そのような項目を変更した場合は Profile 別の `settings.json` に書き込まれませんでした。

**図 3-2 UI 上の表示**

![設定画面で項目を変更しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7afa211df7b741b99dbb2e9d3919db29/vscode-profiles-change-settings-by-ui.png?w=720\&h=275\&auto=compress%2Cformat)

なお、エディター上で設定項目を `settings.json` へコピペしても下記のように(他の項目に比べて)表示が暗くなり無効な状態となります。

**図 3-3 エディターで書き込むと無効状態**

![JSON ファイルを開き、無効な項目を書き込んでいるスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/62d866c44b3349418e0b36fc477a1235/vscode-profiles-user-scope.png?w=483\&h=141\&auto=compress%2Cformat)

また、この項目にマウスカーソルをあててメッセージを確認すると下記のように表示されます。

    This setting has an application scope and can be set only in the user settings file.

## Profile を切り替える

`Profile` メニューに利用可能な Profile が表示されているのでクリックで切り替えることができます。

また、「手動で切り替えるのはちょっと大変かな」と思っていたのですが、Profile はワークスペース別に記録されているようです。ワークスペースを開くと前回利用していた Profile が適用されました。

**図 4-1 Profile は各ワークスペース別に設定される**

![VSCode のウィンドウを 2 つ開き、それぞれが利用しているワークスペースによって異なる Profile が適用されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/53f7395579354cacb5ce7fb829626396/vscode-profiles-each-workspaces.png?w=1440\&h=837\&auto=compress%2Cformat)

## 設定の同期にも対応

設定の同期をオンにしてあれば、各 Profile も同期されます(逆に、特定の Profile を同期から外す指定はなさそうです)。よって、デスクトップ版の VSCode で作成した Profile はブラウザー版からも利用できるようになっていました。

**図 5-1 作成した Profile は各環境にも同期される**

![ブラウザー版の VSCode で Profiles のメニュー項目を開き、作成した Profile が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1b068c9c0ea14ea594c57c59483c3bfb/vscode-profiles-settings-sync.png?w=916\&h=548\&auto=compress%2Cformat)

## エクスポート

メニューからエクスポートを選択すると、エクスポートする項目の選択画面が表示されます。

ここで、 `settings.json` をクリックするとエディターが開き内容を編集できます。編集した内容はエクスポート側にのみ反映され、現在の Profile 側に影響しないようです(Git のステージングで `add` した方のファイルが編集できるイメージに近いのかなと)。

**図 6-1 エクスポート用の設定を編集**

![エクスポート用の画面で settings.json を編集しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/b72c4549d8854129b55b47d79e8e5ec3/vscode-profiles-export-edit.png?w=1091\&h=383\&auto=compress%2Cformat)

Export ボタンをクリックすると GitHub(gist)かローカルファイルのどちらに書き出すか選択するようになります。

**図 6-2 エクスポート先の選択**

![GitHub と  Local を選択するリストが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/29ad3a53239b4dfe840cac7752114d5b/vscode-profiles-export-to-gist.png?w=698\&h=109\&auto=compress%2Cformat)

GitHub の方を選択すると Gist が作成されそこに書き込まれます[^gist]。作成された Gitst はシークレットでしたが、デリケートな設定は含めない方が良いかと思います。

[^gist]: Gist については「[Gist の作成 - GitHub Docs](https://docs.github.com/ja/get-started/writing-on-github/editing-and-sharing-content-with-gists/creating-gists)」などを参照してください。

なお、エクスポート後にリンクをコピーする(または開く)か確認のダイアログボックスが表示されます。ここでコピーすると **エクスポートされた Gist ではなく** ブラウザー版で設定を共有するためのリンクがコピーされます(利用法は後述します)。

**図 6-3 共有に使えるリンクをコピーするダイアログ**

![Copy Link と Open Link および Close が表示されているダイアログボックスのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6db4121b993d4d44a3c123b88ca8db9a/vscode-profiles-export-copy-link.png?w=362\&h=120\&auto=compress%2Cformat)

実際にエクスポートした Gist は下記になります。

@[card](https://gist.github.com/hankei6km/2ccbd6a63c40079af02d9325d49eda26)

また、コピーされた共有用のリンクは下記になります。

    https://vscode.dev/profile/github/2ccbd6a63c40079af02d9325d49eda26

## インポート

インポートもエクスポートと同じように Gist とローカルファイルからできます。今回は Gist からのインポートについて記述します。

### Gist を読み込む

`Profiles` の `Import profie...` を選択するとインポート用のファイル名を入力できます。ここで Gitst のアドレスを入力するとインポート項目の確認画面になります。

操作的にはエクスポートと似たような感じで、事前に `.json` ファイルの内容を確認したり(編集はできないようです)、インポート項目を選択できます。

**図 7-1 インポート前の確認(編集はできない)**

![インポート用の画面で settings.json を開いているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/433f19b1cddb4bc787f783912fde3537/vscode-profiles-import-not-edit.png?w=861\&h=322\&auto=compress%2Cformat)

::: message

これを書いている時点(2023-02-05)では、ブラウザー版で Gist のアドレスを利用するとエラーになりました。Raw ファイルをブラウザー上で実際に開き、そのアドレスを利用すると回避できましたが、これが仕様なのかは不明です。

:::

### 共有用のリンクを利用する

こちらは[更新履歴](https://code.visualstudio.com/updates/v1_75#_profiles)にもデモ動画があるので、あわせて見ていただくとわかりやすいかと思います。

エクスポート時にコピーしたリンクをブラウザーで開くことで、ブラウザー版 VSCode が開始され Profile の動作を確認するプレビュー画面が表示されます(表示されるまで少し時間がかかります)。

なお、ブラウザー側でのサインインなどは必須ではなさそうです(ゲストモードのブラウザーでも動作しました)。

**図 7-2 プレビュー画面で設定はある程度反映される**

![プレビュー画面でカラーテーマが反映されている状態のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/e6235756a8844764bfbf3182ec02903f/vscode-profiles-import-via-browser.png?w=914\&h=441\&auto=compress%2Cformat)

このプレビュー画面では、ある程度の動作が確認できるようになっています(上の画像だとカラーテーマが反映されているのが確認できます)。また、拡張機能についてはインストールしても問題なさそうであれば、この状態から手動でインストールできました(`拡張機能` 項目右側のダウンロードアイコンをクリック)。

挙動を確認できたら、`Import Profile in Visual Studio Code` のクリックで**デスクトップ版の VSCode** が開きます。デスクトップ版ではインポート用の画面(Gist を読み込んだときと同じ画面)が表示されるので、実際にインポートできます。

事前に挙動を確認できるので便利なのですが、ちょっと操作がややこしいかなという印象もありました。この辺はまだ調整中といった感じのようです。

@[card](https://github.com/microsoft/vscode/issues/172671)

::: message

`Import Profile` をクリックすると同名プロファイルが存在しているという警告が表示されます。それでも続行するとインポートはされるのですが永続化はされませんでした(他のタブやデスクトップ版に同期されない)。

:::

::: message

通常版の VSCode とインサイダー版の VSCode ではリンクの URL が異なります(インサイダー版は `insiders.vscode.dev` になる)。そのため、インサイダー版でエクスポートした場合、プレビューはインサイダー版で動作します。

:::

## 設定の共有

これまで、VSCode の設定についての記事を書くような場合、主に設定の手順を記載していましたが、上記の共有用リンクを記事内に記載すれば簡単に設定を共有できそうです。

ということで、[以前の記事](https://zenn.dev/hankei6km/articles/vscode-deco-server)で作った設定を [Gist へエクスポート](https://gist.github.com/hankei6km/9e8fc04a558f7ca9c4fb2c7cf8a60642)してみました。

下記のリンクをブラウザーで開くと Profile のプレビュー画面が表示されます。

**リスト 8-1 通常版 VSCode 用のプレビューリンク**

    https://vscode.dev/profile/github/9e8fc04a558f7ca9c4fb2c7cf8a60642

**リスト 8-2 インサイダー版 VSCode 用のプレビューリンク**

    https://insiders.vscode.dev/profile/github/9e8fc04a558f7ca9c4fb2c7cf8a60642

これまでは便利そうな `settings.json` の設定があっても「いきなり自分へ環境にコピペして大丈夫？」という不安もありました。Profiles 機能で作成したリンクは「まずはプレビューで動作確認してみる」ができるので少し安心できるかと思います。

**図 8-1 ゲストモードのブラウザーでプレビュー**

![インポート用のアドレスをゲストモードのブラウザーで開き、設定が反映された vscode が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/039d298931a84a0096b00660afc8aab0/vscode-profiles-preview-deco.png?w=1440\&h=860\&auto=compress%2Cformat)

プレビューの状態で試したあと、問題がなさそうであれば `Import Profile in Visual Studio Code` のクリックでインポートできるようになります。また、「今回はやめておく」場合はそのままタブを閉じるだけです。

## 考慮点

### Profile の組み合わせはいまのところ難しそう

Profile の組み合わせ(マージ)ができると Extension pack のような使い方もできるかなと思ったのですが、現状では Profile を組み合わせるのは難しそうです。

関連しそうな Issue ができていたので何か方法ができるかもしれません。

@[card](https://github.com/microsoft/vscode/issues/172756)

### リモート系も現状では要注意とのこと

更新履歴にリモート接続と関連する使い方(Codespaces など)での問題が記載されています。

> **Note**: Profiles currently do not work in remote scenarios like [GitHub Codespaces](https://github.com/features/codespaces), but we are working on enabling this. You can track progress in [issue #165247](https://github.com/microsoft/vscode/issues/165247).

Profile の切り替えにより、リモート接続用の拡張機能がアンロードされてしまうと影響がでるようです。

## おわりに

VSCode の新しい Profiles 機能で Profile を複数作成し、Gist を経由しての共有を試してみました。

今回の機能追加ではやはり「拡張機能とその設定を Gist で簡単に共有できる」ことが一番の変化かと思います。

今後、Profile を自由に組み合わせる機能などが追加されていけば、便利な(あるいはちょっと面白そうな)設定をさらに共有しやすくなるかもしれません。
