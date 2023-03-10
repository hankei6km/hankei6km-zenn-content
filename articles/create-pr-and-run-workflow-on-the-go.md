---
id: create-pr-and-run-workflow-on-the-go
title: GitHub Mobile からファイル編集をできるようになったので PR を作ってワークフローを動かしてみた
emoji: 📱
type: idea
topics:
  - github
  - githubactions
published: true
---

GitHub Mobile でファイル編集できないかなと思っていたのですが、いよいよその時はきたようです。

@[card](https://github.blog/2023-03-07-file-editing-on-github-mobile-keeps-leveling-up/)

> On the GitHub Mobile team, one of our missions is to bring your code to you–whenever you are. In the last few months, we’ve brought file editing and pull request creation to your mobile device, so that you can keep contributing on the go. Let’s take a look at what’s new.

:::message
Android 版でしか確認していませんが、編集できるようになったのは `1.103.0` からだと思います(`1.102.0` では編集メニューが表示されませんでした)。
:::

## どのように編集できる？

冒頭でリンクしているブログ記事に詳細があるので、ここでは軽く触れてみます。

1.  リポジトリの画面で「コードをブラウズ」でファイル一覧からファイルを表示
2.  右上のドロップダウンメニューから「ファイルを編集」を選択

これで編集画面になります。

**図 1-1 ファイル編集メニューから編集画面を表示**

![ファイルを表示している画面でドロップダウンを表示、そしてファイル編集画面が表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/629b0add36564e27a44b6cb65ce3059b/create-pr-and-run-workflow-on-the-go-dropdown-edit-file.png?w=1110\&h=1040\&auto=compress%2Cformat)

なお、ローカルへの保存はできないようで「保存 = コミットしてプッシュ」という扱いのようです。このときにブランチを作成すると、あわせて PR を作ることもできます。

**図 1-2 保存の代わりにコミットとプッシュ**

![コミットを作成する画面と PR を作成する画面のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/5aca4331aecb4d12a3d241159a4dda11/create-pr-and-run-workflow-on-the-go-commit-pr.png?w=1110\&h=1040\&auto=compress%2Cformat)

感覚的には github.dev に近い感じなのかなと思っていたのですが、いろいろと違いはありそうです。

*   ブランチ名は指定できない(`hankei6km-patch-1` のようになる)
*   おそらく新規にファイルを作成できない

GitHub Mobile で編集する状況を考えるとシンプルな機能に絞ったのかなというところでしょうか。この辺の操作について「どうしても」というときはブラウザーでウェブ UI を使うことになります。

あとは、拡張機能なども動かないので、シンタックスハイライトやフォーマットなどはなさそうです。

## 少し使ってみた

どのように編集できるかわかったところで、実際にコードなど編集してみました。その感想についてなど。

### 良いところ

やはり、屋外にいるときでも気軽に編集できるのはよいです。

たまに、移動中「さっきのエラーを修正する方法を思いついてしまった」となるのですが、その場で試すことが難しいのでモヤモヤすることがありました。

これが、ファイルを修正して PR を作ったりできるとなれば、リポジトリの設定によってはスマホをタプタプするだけでワークフローを動かして結果の確認までできてしまいます。

ということで、どのようにワークフローを動かしたのか簡単な事例など。

いま、Dev Containers の Feature を作る練習をしているのですが、[Feature 用スターター](https://github.com/devcontainers/feature-starter)を使ってリポジトリを作ると各種ワークフローなども準備されています。またコードの方も基本的には設定ファイル(`.json`)と `Dockerfile` 的なシェルスクリプトになるので「ちょっと修正しようかな」という場合も、GitHub Mobile でわりと編集できました。

下図は、[試しに作った Feature](https://github.com/hankei6km/test-feature-starter/tree/main/src/aicommits) をちょっと編集してワークフローを動かしている画面です(PR 作成についてのスクリーンショットは省略しています)。

**図 2-1 スクリプトファイルを編集してコミット**

![GitHub Mobile でファイルを編集しコミットを作成しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f7ce08936dd04ccf8c04a6c716f7c75e/create-pr-and-run-workflow-on-the-go-edit-script.png?w=540\&h=1040\&auto=compress%2Cformat)

**図 2-2 プルリクエストを作成するとテスト用ワークフローが動きだす**

![GitHub Mobile で PR の画面からテスト用ワークフローが実行されている状態を確認しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d16ae105e7fb4b408d01074dce302945/create-pr-and-run-workflow-on-the-go-release.png?w=540\&h=1040\&auto=compress%2Cformat)

テストが通ったことを確認できたら普通にマージして、さらに(GitHub Mobile の画面から離れますが)リリース用のワークフローを動かすこともできます。

**図 2-3 ウェブ UI からリリース用ワークフローを開始**

![モバイル用ブラウザーでリポジトリの Action タブからワークフローを開始しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/78db710056654ad492f0a4e74c94410e/create-pr-and-run-workflow-on-the-go-run-workflow.png?w=540\&h=1040\&auto=compress%2Cformat)

### とまどったところ

「ブランチ作るの忘れてコミットしてしまった」という操作的な話もありますが[^branch-protection]、ここではもう少し別の話など。

[^branch-protection]: 実はブランチ作り忘れてデフォルトブランチにコミットしてしまい、ブランチを保護は大事だよねと今更ながらに思いました。

当然と言えば当然なのですが「ファイルは GitHub 上に存在しないと編集できない」というのはわりと見落としてしまうかもしれません。

たとえば、Codespaces や CodeSandbox のようなクラウド IDE サービスを使って編集している場合、commit してあるけど push していないことはあります。それでも同じサービスを使えば他の PC からでも編集を継続できますが、Github Mobile ではサービスが異なるので編集を継続できません。

これを忘れていて、移動中に「ちょっと編集しちゃおうっかな」とリポジトリを開いたあと、push していなかったことに気が付き落ち込みました。

## その他

細かいことですが、GitHub Mobile で編集する場合も verified になります。この辺は github.dev と同じ扱いのようです。

**図 3-1 GitHub Mobile からのコミットは署名される**

![GitHub のウェブ UI でコミットを表示し、verified が付いて状態のスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/6423f216dad94a4099b78dd542a056b4/create-pr-and-run-workflow-on-the-go-run-workflow-verified.png?w=531\&h=167\&auto=compress%2Cformat)

@[card](https://zenn.dev/hankei6km/articles/commits-on-github-dev-are-signed-by-web-flow-gpg)

## おわりに

GitHub Mobile でファイルを編集できるようになったので少し試してみました。

編集するという意味では、拡張機能などで完全装備された状況と比べると少し物足りないところもありますが、GitHub 上でワークフローを整備しておけばテストなどを気軽に試せるのは良いと感じました。

たとえば、Zenn 用のリポジトリなどでワークフローから textlint を利用していれば、移動中「typo 見つけてしまった」というときも、GitHub Mobile で対応しやすいかと思います。
