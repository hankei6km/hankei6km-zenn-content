---
id: simple-nodejs-debugging-vscode
title: 大体なんでも接続する VS Code の Node.js 用デバッガーを気軽に使いたい
emoji: 🔌
type: tech
topics:
  - vscode
  - javascript
  - typescript
  - debug
  - contest2025ts
published: true
---

VS Code でデバッグを行う場合、ググったり AI に質問すると `launch.json` を作成する方法が上位に表示されます(されないこともあります)。しかし、状況によっては事前に設定を作るのは少しばかり手間に感じることもあります。

*   実験用に作った `.js` `.ts` ファイルの挙動を確認したい場合
*   `npm run test` でちょっとだけ変数の内容を確認したい場合

今回の記事は VS Code + Node.js の組み合わせにおいて、組み込みのデバッガーで気軽にデバッグする方法のメモとなります。

## 気軽にデバッグする方針

VS Code でのデバッグは「各言語用のデバッグアダプター(拡張機能)」と「デバッグ用に開始されたプロセス(ランタイム)」の接続により実現されます。

**図 1-1 **[**Introducing Logpoints and auto-attach**](https://code.visualstudio.com/blogs/2018/07/12/introducing-logpoints-and-auto-attach)** より引用**

![VS Code debugging architecture](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ec2ef072c03b410c84405225f5548cb7/simple-nodejs-debugging-vscode-debugging_architecture.png?w=1798\&h=856\&auto=compress%2Cformat)

標準的なデバッグでは、デバッグ用にプロセスを開始する方法(引数など)を `launch.json` へ記述します。これは、複雑なデバッグの手順を「設定としてプロジェクト内で永続化しやすい(共有しやすい)」という利点を持っています。しかし、ちょっとしたコードのデバッグで設定を記述するのは少し手間だと感じることもあります。

一方で、VS Code + Node.js 用のデバッグ機能(組み込みの js-debug 拡張機能)は `node` プロセスへ自動的にアタッチする機能も用意されています。これを利用すると `launch.json` の作成や追加の拡張機能などを必要とせず、おまかせでデバッグを開始できるようになっています。

ここからは、Node.js 用のデバッグ機能で提供される「自動アタッチ(Auto Attach)」の利用方法や、ハマりどころなどについて見ていきます。

## `node` プロセスへ自動的にアタッチしデバッグする

前述のように組み込みのデバッガーは、事前設定なしで `node` プロセスへアタッチできるようになっています。

これを利用すると、「単一の `.js` `.ts` ファイル」や「NPM スクリプト」を気軽にデバッグしたり、「開始タイミングを事前に決定できない `node` プロセス」への接続も柔軟に対応できます。

### 素のスクリプトファイルをデバッグ

まずは、「`index.js` を作っただけで `package.json` を用意していない状態」でのデバッグ方法です。

デバッグ対象のスクリプトファイル(`index.js`)を適当に用意します。内容は `console.log(123)` だけでもよいですし、「それっぽい」操作を試したい場合は以下のサンプルをベタっと貼り付けてください。

**リスト 2-1 サンプルのスクリプトファイル(****`https://github.com`**** から ****`fetch`**** する)**

```javascript
async function getHtml(){
    let ret = ''
    const r = await fetch('https://github.com');
    if (r.ok) {
        for await(const b of r.body){
            const text = new TextDecoder().decode(b);
            ret += text;
        }
    } else {
        ret= `Failed to fetch the page: ${r.status}, ${r.statusText}`;
    }
    return ret
}
async function main(){
    console.log(await getHtml())
}
main()
```

スクリプトファイルを用意したら、コマンドパレットから `Debug: Toggle Auto Attach` を実行し自動アタッチを有効化します。

**図 2-1 自動アタッチ設定用のリスト**

!["Always" "Smart" などが表示されているリストのスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1c37314b46f1467b8f552d57801be10e/simple-nodejs-debugging-vscode-enable-setting-list.png?w=719\&h=203\&auto=compress%2Cformat)

有効化するための項目はいくつかありますが、今回は単一の `.js` を動かすだけなので `Smart` を選択します。選択した後にターミナルを開き `node index.js` のように実行すると、以下のようにデバッガーが自動的に `node` プロセスへ接続し、デバッグ操作を行えるようになります。

**図 2-2 単独のスクリプトファイルを実行している ****`node`**** プロセスへアタッチしてのデバッグ**

![ターミナルで実行された "node index.js" へデバッガーが接続しデバッグしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/fae78da0240c40a58b3aad89fcc4c24d/simple-nodejs-debugging-vscode-debug-auto-attach.png?w=1052\&h=598\&auto=compress%2Cformat)

また、自動アタッチで開始したデバッグの場合でも、ブレイクポイントでの式やログメッセージなども利用できまます。以下は `fetch` した body を何回で受信しているか確認している例です。

**図 2-3 カウント表示する式を設定**

![ソースコードの 7 行目のブレイクポイントを "Expression" 設定へ変更し "console.count('text')" を設定しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ea02eb42a7ce4e1fa4d81b36dba03f08/simple-nodejs-debugging-vscode-debug-logpoint-expression.png?w=1209\&h=627\&auto=compress%2Cformat)

**図 2-4 デバッグを開始するとカウント表示される**

![ターミナルに設定したカウントの結果が出力されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/578b4d19538f4cf0949149eb69c775db/simple-nodejs-debugging-vscode-debug-logpoint-run.png?w=1247\&h=1097\&auto=compress%2Cformat)

このように、ちょっと実験したらすぐに消すようなコードをデバッグする場合、自動アタッチは重宝するかと思います。

::: message

自動アタッチの有効化で「`Always` と `Smart` どちらを利用するか？」について、私は以下のような基準で選択しています。

*   フレームワーク(Jest など)のコマンドを利用する場合: `Always`
*   簡単なスクリプトを実行するだけの場合: `Smart`

これは、「フレームワークのコマンドは内部的に `node_modules` 内に配置されてるスクリプトを利用することが多い、よって `Smart` での利用は難しいだろう」という判断からです。

ただし、スパッときれいには分類できません。まず、ランナー(ユーザースクリプトを実行するためのスクリプト)の中でも一般的なものは、例外として `node_modules` 内に配置されていても `Smart` で利用できます。

> <https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_auto-attach>
>
> *   smart - If you execute a script outside of your folder or use a common 'runner' script like mocha or ts-node, the process will be debugged.

そして、ランナーによっては「それ自体は例外になっていないが、内部的に例外のランナー(`ts-node` など)を使っている」という場合もあり、「結局なところどれが例外なんだ？」状態になります。

この辺は深く考えると混乱してくるので、よくわからない場合は「とりあえず `Always`」にしておけば大体は接続できると思います。

:::

### TypeScript でデバッグ

TypeScript のソースコードをちょっと動かしたい場合は、 `ts-node` や `node --experimental-strip-types` などを利用する方法があります。

この場合、とくに意識することなく JavaScript と同じようにデバッグできています。

**図 2-5 ****`node --experimental-strip-types`**** へアタッチしてのデバッグ**

![node --experimental-strip-types で実行している ".ts" ファイルをデバッグしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/19cc077b386f4efcacb47a6197bc19fa/simple-nodejs-debugging-vscode-debug-strip-types.png?w=1281\&h=582\&auto=compress%2Cformat)

一方で TypeScript コードの実行方法としては「`tsc` やビルドツールにより `.ts` から `.js` へトランスパイルする」方法もあります。この場合、「ソースコードとなる `.ts` ファイル」と「実際に `node` で実行している `.js` ファイル」は異なるファイルになるため、ソースファイル上のブレイクポイントなどは認識されません。

対応方法としてはトランスパイル時にソースマップを作成することになります。

https://ja.wikibooks.org/wiki/JavaScript/%E3%82%BD%E3%83%BC%E3%82%B9%E3%83%9E%E3%83%83%E3%83%97

ちょっと面倒そうな感じですが、一般的なツールではソースマップ用のオプションが用意されていることが多いので、それらを利用すれば大体は解決できます。

**リスト 2-2 ****`tsc`**** コマンドでソースマップを出力する例**

```shell
tsc --sourcemap
```

ソースマップ付きでトランスパイルした後は、`.js` を `node` で実行することによりデバッグを行えます。

**図 2-6 ****`tsc --sourcemap`**** したコードでのデバッグ**

![ソースマップ付きでトランスパイルした後に、nodeで".js"ファイルを実行しデバッグしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d309f979b479463c8acc055f093a2dc9/simple-nodejs-debugging-vscode-debug-tsc-sourcemap.png?w=1079\&h=616\&auto=compress%2Cformat)

このような感じでソースマップを作成することにより、ソースファイルと実行用ファイルを関連付けできますが、少し注意点もあります。実行方法などによって「型関連のコードなどに設定したブレイクポイントの挙動」が少し異なっています。

たとえば、ぱっと見では関数に見える [import() type](https://www.typescriptlang.org/docs/handbook/modules/reference.html#import-types) 行へブレイクポイントを設定してしまった場合の挙動です。この場合 、`ts-node` などでは後続のステートメントで停止します。しかし、`tsc` でトランスパイルした `.js` を `node` で実行すると停止しませんでした。

以下は 3 行目と 8 行目にブレイクポイントを設定している状態で試しています。

**図 2-7 ****`ts-node`**** では 4 行目で停止する(****`ts-node`**** は ****`10.9.2`**** を利用)**

![デバッグ UI でエディターの 4 行目で停止しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f76dc2d6e1dd4ee392bb20806387a8b0/simple-nodejs-debugging-vscode-debug-import-type-ts-node.png?w=1124\&h=675\&auto=compress%2Cformat)

**図 2-8 ****`tsc --sourcemap`**** では 3 行目が無視される(****`tsc`**** は ****`5.8.3`**** を利用)**

![デバッグ UI でエディターの 8 行目で停止している。また、3 行目のブレイクポイントは白丸となり無効化されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c417d5eeb4584147b7fc094638c3c063/simple-nodejs-debugging-vscode-debug-import-type-tsc.png?w=1124\&h=675\&auto=compress%2Cformat)

ソースマップの設定によっても挙動は変化しそうですが、細かく注意するのも少し面倒です。型関連に限らず「トランスパイル後に残らないコードへのブレイクポイント」は避けるように意識した方が良いかと思います。

::: message

TypeScript を利用しているとき以外でも、デバッグ時にソースマップを必要とする場合があります。

*   Babel などにより、古い実行環境用へ変換している場合
*   rollup などにより、複数 `.js` ファイルを 1 ファイルへ結合している場合

基本的には「エディターで編集しているファイル」と「実際に実行しているファイル」が異なる場合、デバッグ時にはソースマップを意識しておくと良いかと思います。

:::

### NPM スクリプトに引数を渡しながらデバッグ

ここまではターミナル上で `node` (`ts-node`)コマンドを利用した例ですが、自動アタッチはターミナルから開始したコマンドの子プロセスも対象となっています。よって、ユニットテストや開発者モード実行用の NPM スクリプトなどから開始される `node` プロセスにも有効です。

こちらも自動アタッチを有効にしたあと、ターミナルから NPM スクリプトを実行するとデバッグが開始されます。なお、以下の例では TypeScript + Jest の場合でデバッグしていますが、`jest.config.js` で TypeScript 用に設定してあればとくに意識することは少ないと思います。 (`ts-jest` と `ts-node` がうまいことやってくれる)

**図 2-9 NPM スクリプトへの自動アタッチによるデバッグ**

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c72db653e94c403fba6dd76d52ec0d4c/simple-nodejs-debugging-vscode-debug-auto-attach-npm-scrippt.png?auto=compress%2Cformat)

この方法の場合、NPM スクリプトはターミナルから実行になるので、引数や環境変数を渡しやすいなどの利点もあります。

**図 2-10 NPM スクリプトに引数を渡す例**

```shell
npm run test -- --testNamePattern 'should return blank'
```

注意点としては、フレームワークのコマンドへ自動アタッチする場合は `Smart` だと編集中のソースファイルを認識しないことが増えます(ブレイクポイントで一時停止しないなど)。たとえば、Jest では環境(CPU コア数)などにもよりますがこのような状態になります。

フレームワークのコマンド利用時にブレイクポイントで停止しないようなときは、とりあえず `Always` を試してみるのがよいかと思います。

::: message

自動アタッチを行う方法としては、コマンドパレットから `Debug: JavaScript Debug Terminal` でデバッグ用に設定されたターミナルを開く方法もあります。

このコマンドを利用すると、 `Always` と同じような状態でターミナルが開くようになっています。 (後述する NPM Scripts Views などで開始されるターミナルも JavaScript Debug Terminal です)

自動アタッチの `Always` は接続しすぎるので、特定のターミナル内だけで自動アタッチしたい場合、こちらの利用を検討するのもよいかと思います。

:::

### 非同期に開始される複数の `node` プロセスでデバッグ

自動アタッチでもデバッグを開始した後、さらに他の `node` プロセスへアタッチできます(いわゆる [Multi-target debugging](https://code.visualstudio.com/docs/debugtest/debugging#_multitarget-debugging) です)。また、アタッチしたプロセスが 1 つでも残っていればデバッグは継続されます。よって、ユーザーからのリクエストで非同期に開始・終了される `node` プロセスでもデバッグを開始できます。

たとえば、以下のような express のサーバーで「ルートによって異なる `node` プロセスを開始し結果を受け取る」場合、プロセス開始毎にアタッチしデバッグを行えます。

<!-- textlint-disable -->

::: details サンプルソースコード

<!-- textlint-enable -->

`index.js`

```javascript
const express = require('express');
const { spawn } = require('child_process');
const app = express();
const port = 3000;

function getMiddleware(apiPath) {
    return (req, res, next) => {
        const child = spawn('node', [`lib/${apiPath}.js`]);

        let stdout = '';
        let stderr = '';

        child.stdout.on('data', (data) => {
            stdout += data;
        });

        child.stderr.on('data', (data) => {
            stderr += data;
        });

        child.on('close', (code) => {
            if (code) {
                res.status(500).json({ error: `Child process exited with code ${code}`, stderr });
                return;
            }
            try {
                const json = JSON.parse(stdout);
                res.json(json);
            } catch (e) {
                res.status(500).json({ error: 'Invalid JSON output' });
            }
        });
    };
}

app.use('/api1', getMiddleware('api1'));
app.use('/api2', getMiddleware('api2'));

app.listen(port, () => {
    console.log(`server listening at http://localhost:${port}`);
});
```

`lib/api1.js`

```javascript
const res = {
    name: 'API-1'
}
process.stdout.write(JSON.stringify(res));
```

`lib/api2.js`

```javascript
const res = {
    name: 'API-2'
}
process.stdout.write(JSON.stringify(res));
```

:::

**図 2-11 リクエスト(****`GET`****)により開始されたプロセスでのデバッグ(左下の CALL STACK で ****`api2.js`**** を実行している ****`node`**** へ接続していることが確認できる)**

!["curl" コマンドで "api2" をゲットし、対応するプロセスのスクリプトがデバッグ UI で停止しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/db60e7ee617a454aab59c2df05f80836/simple-nodejs-debugging-vscode-debug-multi-route.png?w=1920\&h=1143\&auto=compress%2Cformat)

また、異なるプロセス上で同時に一時停止させることもできます。ただし、ソースファイルの該当箇所への切り替え(表示)は自動的には行われないようです。複数箇所で一時停止しているとき、注目する箇所を変更するには CALL STACK からの切り替え操作になります。

**図 2-12 CALL STACK 上で切り替え操作**

![CALL STACK 上で対象とする一時的位置を切り替えているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3b28133416e34423a4694eefab7952cc/simple-nodejs-debugging-vscode-debug-call-stack.png?w=457\&h=703\&auto=compress%2Cformat)

なお、上記の例では最初にアタッチした `node` からの子プロセスにアタッチしていますが、プロセスが親子の関係になっていなくとも複数の `node` にアタッチもできています。

**図 2-13 2 つのサーバー(****`node`****)を個別に開始しデバッグ**

![2 つのターミナルでそれぞれ "node" コマンドを手動で開始し、デバッグしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/90538a9f53ff4705b5fe0f58a724c875/simple-nodejs-debugging-vscode-debug-multi-server.png?w=1920\&h=1143\&auto=compress%2Cformat)

## NPM スクリプト用 の UI からデバッグを開始する

ここまで見てきたように、自動アタッチは柔軟にデバッグを開始できるのですが、ターミナルでの操作は避けたいと思うかもしれません。

そのようなとき、NPM スクリプトのデバッグに限れば、NPM Scripts View からデバッグを開始できます。

NPM Scripts View はファイルエクスプローラーの下の方に配置されています。ここで「NPM SCRIPTS」横の `>` をクリックするとスクリプト一覧が表示されます。 (配置されていないときは、コマンドパレットから `Explorer: Focus on NPM Scripts View` を選択すると表示されます)

**図 3-1 左下に表示される NPM Scripts View**

![VS Code の画面左したに NPM スクリプトが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/2f0593622577453fa8695cc37afc0192/simple-nodejs-debugging-vscode-npm-scripts-view.png?w=1146\&h=790\&auto=compress%2Cformat)

スクリプト一覧が表示されたらマウスカーソルを合わせるとデバッグ用と通常 Run 用のアイコンが表示されます。

**図 3-2 デバッグアイコン**

![スクリプト名の上にデバッグアイコンと Run 用アイコンが表示されているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f90c80236b9d4b56a222edfa5c5f309e/simple-nodejs-debugging-vscode-degub-icon.png?w=427\&h=108\&auto=compress%2Cformat)

ここでデバッグ用アイコンをクリックするとターミナル(JavaScript Debug Terminal)内で NPM スクリプトが開始され、デバッガーがアタッチするようになっています。

**図 3-3 アタッチした ****`node`**** プロセスのスクリプトをデバッグ**

![ターミナル上の開始された node プロセスへデバッガーがアタッチし、デバッグしているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a1c4488ab317473485086f083502a4fa/simple-nodejs-debugging-vscode-degub-npm-scripts-view.png?w=1152\&h=796\&auto=compress%2Cformat)

このような感じで、NPM スクリプトから開始されるコードをデバッグしたい場合は、UI の操作だけでも気軽に利用できます。

ただし、以下のようなデメリットもあります。

*   NPM スクリプト(`package.json`)を用意する必要がある
*   VS Code に `package.json` を認識させる必要がある(NPM Scripts View はワークスペース直下に `package.json` がないと使えないようです)
*   NPM スクリプト開始時に一時的な引数や環境変数を渡せない(たぶん)

::: message

NPM スクリプトのデバッグ方法としては、コマンドパレットから `Debug: Debug npm Script` も利用できます。コマンドを実行すると、以下のように NPM スクリプトの一覧が表示されるので、スクリプトを選択するとデバッグが開始されます。

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/65fdf847bbab45479714cae86fe5eb44/simple-nodejs-debugging-vscode-npm-scripts-list.png?auto=compress%2Cformat)

:::

## Node.js へ自動アタッチできるカラクリ

通常、Node.js をデバッグ用に開始する場合、 `--inspect` などのフラグを指定するようになっています。

https://nodejs.org/en/learn/getting-started/debugging

ところが、ここまでに試した方法では `node` 開始時にフラグを指定しないでもアタッチできました。

実は、現在の JS 用デバッガー(拡張機能)では自動アタッチを有効化しているとターミナル内では `$NODE_OPTIONS` 環境変数などが設定されます。これにより、デバッグ用のブートローダースクリプトが読み込まれるようになっています。

以下は、 `Smart` を指定している状態でコマンドパレットから `Terminal: Show Environment Contribution` を実行した結果の抜粋です。

```markdown
# Terminal Environment Changes

## Extension: ms-vscode.js-debug

Enables Node.js [auto attach](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_auto-attach) debugging in "smart" mode

Enables Node.js [auto attach](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_auto-attach) debugging in "smart" mode (workspace)

- `NODE_OPTIONS= --require /home/node/.vscode-server/data/User/workspaceStorage/2bff38b2ddf6c672f14e0e680fdcadc2/ms-vscode.js-debug/bootloader.js ${env:NODE_OPTIONS}`
- `VSCODE_INSPECTOR_OPTIONS=${env:VSCODE_INSPECTOR_OPTIONS}:::{"inspectorIpc":"/tmp/node-cdp.362-77ca0357-0.sock.deferred","deferredMode":true,"waitForDebugger":"","execPath":"/usr/local/bin/node","onlyEntrypoint":false,"autoAttachMode":"smart","mandatePortTracking":true,"aaPatterns":["/workspaces/test-auto-attach-node-ts/**","!**/node_modules/**","**/$KNOWN_TOOLS$/**"]}`
```

逆に言うと、これら環境変数やブートローダーのファイル(と内部で利用される IPC)などが正しく設定されていないと想定通りに動作しなくなります。

*   利用しているフレームワークが `$NODE_OPTIONS` を上書きしている
*   実行環境がコンテナ内にあり、ブートローダーや IPC を参照できない状態でスクリプトが開始される

使っているフレームワークなどによりもますが、想定通りにデバッグが開始されないときは、上記のような点を意識していると原因を切り分けしやすいかと思います。

## デバッグを上手く開始できないとき

デバッグ機能を利用していたときに遭遇した「上手くいかない状況とその回避方法」についてのメモです。

### アタッチされない

よくあるのは、自動アタッチを有効化した後に新しくターミナルを作成していない場合です。

自動アタッチは新しくターミナルを開く時に `$NODE_OPTIONS` などを設定することで実現されます。よって、すでに開いていたターミナルでは環境変数と実際の設定が食い違うため、自動アタッチが行われないことになります。

なお、ターミナルの環境変数などが食い違っているときは、以下のように ⚠ 表示で警告されることもあります(理由は不明ですが、されないこともあります)。

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3fe68ef30bb1410b97188423dd639578/simple-nodejs-debugging-vscode-debug-warn-icon.png?auto=compress%2Cformat)

よって、基本的な対処方法としては「まずはターミナルを開き直してみる」です。

### アタッチはされるがソースファイルを認識しない (`Smart` を利用している)

一見するとプロセスにアタッチしてデバッグできるように見えるが、ソースファイル上のブレイクポイントで一時停止しないようなこともあります。

これは、自動アタッチの `Smart` とフレームワーク(Jest や Next.js など)の組み合わせで発生することが多いです。

:::: details アタッチされても一時停止しない状況について、Jest の場合で少し詳細を見ていきます(長いの折りたたんでいます)。

::: message

`Smart` の挙動として「スクリプトを開始するランナーが一般的なものであれば無条件にアタッチする」ようになっています。Jest はこの条件に合致するようなので、`Smart` でもアタッチ可能となっています。

> <https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_auto-attach>
>
> *   smart - If you execute a script outside of your folder or use a common 'runner' script like mocha or ts-node, the process will be debugged.

しかし、Jest では必要に応じて並列処理用にワーカープロセスを生成しています。これは、デバッグ UI の CALL STACK にワーカープロセス(`processChild.js` を実行)が複数表示されていることから確認できます。

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/fd42e384eeae49ec9507dd6c9550db2a/simple-nodejs-debugging-vscode-debug-jest-worker.png?auto=compress%2Cformat)

この場合、`node_modules` に配置されている `processChild.js` がランナーとなりますが、これはアタッチされる条件からは外れるようです。よって、親プロセス(ランナー)とはアタッチされますが、デバッグしたいソースファイルを扱うワーカーにはアタッチされないので、デバッグは開始されていてもブレイクポイントで一時停止しない状況となります。

このため Jest では `Allways` が必要となることが増えます。

なお、Jest は常にワーカーを生成するわけではありません。実行環境の資源状況(CPU コア数など)によってはワーカーを生成しないこともあります。たとえば、GitHub Codespaces で 2 コア構成の環境を作成したときは、以下のようにワーカーが生成されないため `Smart` でもアタッチできます。 (Codespaces でも 4 コア構成だとワーカーが生成さます)

![](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/d3b8b39c64844efea8e07b940e656bbd/simple-nodejs-debugging-vscode-debug-jest-codepaces.png?auto=compress%2Cformat)

CPU コア数などでワーカープロセスの状況が変わることは他のフレームワークでもあると思いますので、少し意識しておくと「環境によってなぜか接続できない」で悩むことを減らせるかもしれません。

ちなみに、引数に `-w 1` を渡してもワーカープロセスが生成されなくなります。今後の Jest でも同じ挙動になるのかは不明なのであまり推奨はできませんが、デバッグ操作の反応が良くなることもあるので重宝しています。

```shell
npm run test -- -w 1
```

:::

::::

対処方法としては「自動アタッチを `Always` にしてみる」です。あとは、設定を追加することになりますが、[Auto Attach Smart Patterns](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_additional-configuration) を指定するという方法もあります。

### アタッチはされるがソースファイルを認識しない (ソースマップがない)

ビルド(バンドル)した CLI ツールをデバッグしようとしたときにわりと遭遇しました。 (最近も、ちょっとしたツール作っているとき「なぜ止まらん？」と少し悩んでしまいました)

たとえば、 `npm run start` で実行する `.js` をバンドルツールでビルドしている場合、プロダクション用としてソースマップを作成しないことが多いかと思います。この場合、「`node` プロセスから実際に利用される `.js` ファイル」と「エディター上で編集しているソースファイル」は別ファイル扱いなので、ブレイクポイントなどの指定には利用できないことになります。

対処方法としては「ソースマップを適切に作成する」です。

**リスト 5-1 デバッグ用の NPM スクリプトを追加した例(****`build.sh`**** は ****`--sourcemap`**** でソースマップを作成する前提)**

```json
  "scripts": {
    "build": "scripts/build.sh",
    "clean": "scripts/clean.sh",
    "start": "node dist/index.js",
    "prerestart": "npm run clean && npm run build",
    "debug": "node dist/index.js",
    "predebug": "npm run clean && npm run build -- --sourcemap"
  },
```

なお、一般的なバンドルツールではなく「独自にファイル連結などでスクリプトファイルを加工している」ような場合だと対応はちょっと難しいかもしれません。

### `npm ci` などが失敗する

デバッグとはちょっと離れた話になりますが、自動アタッチ関連のハマり所なので少し記述しておきます。

原因は不明なのですが、 `npm ci` などのコマンドでエラーとなることもあります。 (自動アタッチを有効化している状態で Dev Container を作成した直後に `npm ci` すると発生するように感じるが確証はないです)

    node:internal/modules/cjs/loader:1404
      throw err;
      ^

    Error: Cannot find module '/home/node/.vscode-server/data/User/workspaceStorage/5ad91349dab35d9a960785501ba3c09b/ms-vscode.js-debug/bootloader.js'
    Require stack:
    - internal/preload
        at Function._resolveFilename (node:internal/modules/cjs/loader:1401:15)
        at defaultResolveImpl (node:internal/modules/cjs/loader:1057:19)
        at resolveForCJSWithHooks (node:internal/modules/cjs/loader:1062:22)
        at Function._load (node:internal/modules/cjs/loader:1211:37)
        at TracingChannel.traceSync (node:diagnostics_channel:322:14)
        at wrapModuleLoad (node:internal/modules/cjs/loader:235:24)
        at Module.require (node:internal/modules/cjs/loader:1487:12)
        at node:internal/modules/cjs/loader:2045:12
        at loadPreloadModules (node:internal/process/pre_execution:743:5)
        at setupUserModules (node:internal/process/pre_execution:208:5) {
      code: 'MODULE_NOT_FOUND',
      requireStack: [ 'internal/preload' ]
    }

    Node.js v22.16.0

この場合はターミナルを開き直しても回避できず、一旦 `Disable` してからターミナルを開きなおす必要があるようです。

### ターミナル多重化環境との相性

私はターミナル内で Tmux(ターミナル多重化)を使うこともあるので、Tmux ならでの注意点を少し。

環境変数を必要とする拡張機能と、ターミナル多重化環境の組み合わせは想定通りに動作しないことも多いです。これは、VS Code による環境変数書き換え処理が、多重化ツールのセッション(サーバープロセス)まで波及しないことが原因です。 (ターミナル多重化を活用している人にはいまさらな話であり、上手く回避している人も多いかとは思いますが…)

ターミナルを新しく開き直しても想定したようにデバッグが開始されないときは、多重化ツールのセッションも再作成されているか意識すると解決しやすいかと思います。

## 考慮点

### 手順を永続化しにくい

`launch.json` により設定を準備するということは、以下のような項目を永続化されたコードとしてプロジェクト(リポジトリ)へ保存することを意味します。

*   スクリプト開始時の引数指定
*   環境変数の設定
*   実行時のカレントディレクトリ設定
*   Node.js のバージョンやランタイム設定
*   リモートアタッチの設定

よって、一時的には手間ではありますが、デバッグ開始手順の再利用や共有がされるやすくなると言えます。

一方で自動アタッチは気軽に利用できますが、ターミナル上での入力は設定として残りにくいと言えます。

裏技的な引数などが増えてしまいそうなときは、スクリプト化しておくようなことも意識した方が良いかと思います。 (あるいは、いまどきだと「AI エージェントなどが参照するナレッジベースへ手順を記述しておく」という方法もありかもしれません)

### デバッグ用拡張機能

今回は気軽に開始できることを優先していたので触れませんでしたが、各種フレームワーク用に用意されている拡張ではデバッグを支援するものもあります。

たとえば、デバッグするテスト関数を引数で指定するのは手間だと感じることもありますが、そのような操作を UI で行える拡張機能もあるようです。

デバッグ用の環境が固まってきたら、デバッグ操作を支援してくれる拡張機能を探してみるのも良いかもしれません。

## おわりに

VS Code と Node.js の組み合わせにおいて、`launch.json` を作成することなくデバッグする方法を確認しました。

Node.js への自動アタッチ機能は気軽に利用できのと同時に柔軟な接続も可能なため、これまでデバッグをためらっていたような場面でもコードの挙動を確認しやすくなるかと思います。
