---
id: rollup-config-js-as-esm
title: rollup.config.js がデフォルトで ESM 扱いになっていたので対応
emoji: ⚒️
type: tech
topics:
  - rollup
  - esm
published: true
---

rollup.js が v3.x 系にアップデートされましたが、その中で表題のような変更がありました。

@[card](https://github.com/rollup/rollup)
@[card](https://github.com/rollup/rollup/releases/tag/v3.0.0)

> Configuration files are only bundled if either the `--configPlugin` or the `--bundleConfigAsCjs` options are used. The configuration is bundled to an ES module unless the `--bundleConfigAsCjs` option is used. In all other cases, configuration is now loaded using Node's native mechanisms ([#4574](https://github.com/rollup/rollup/pull/4574) and [#4621](https://github.com/rollup/rollup/pull/4621))

私の場合は `rollup.config.js` で `__dirname` を使っているリポジトリあったので、エラーが発生してしまうことになりました。

その対応についてなど。

## CJS 扱いにする

エラーになると下記のようなメッセージが表示されます。

**図 1-1 エラーメッセージに対応方法が含まれている**

    [!] RollupError: Node tried to load your configuration as an ES module even though it is likely CommonJS.
    To resolve this, change the extension of your configuration to ".cjs" or pass the "--bundleConfigAsCjs" flag.

ここに書いてある通りに拡張子を `.cjs` とするか、オプションで `--bundleConfigAsCjs` で指定すると CJS 扱いになります。

試しにオプションを指定するとエラーは回避されました。

**図 1-2 オプション指定**

```shell-session
$ npx rollup --config --bundleConfigAsCjs
```

## ESM として対応

「どのようなエラーになったのか」で対応は異なりますが、今回の `__dirname` 関連の場合で記述します。

**図 2-1 エラーメッセージ詳細**

    Original error: __dirname is not defined in ES module scope
    This file is being treated as an ES module because it has a '.js' file extension and '/workspaces/gas-gchanges2notion/package.json' contains "type": "module". To treat it as a CommonJS script, rename it to use the '.cjs' file extension.
    https://rollupjs.org/guide/en/#--bundleconfigascjs
    ReferenceError: __dirname is not defined in ES module scope
    This file is being treated as an ES module because it has a '.js' file extension and '/workspaces/gas-gchanges2notion/package.json' contains "type": "module". To treat it as a CommonJS script, rename it to use the '.cjs' file extension.
        at file:///workspaces/gas-gchanges2notion/rollup.config.js:44:27
        at ModuleJob.run (internal/modules/esm/module_job.js:183:25)
        at Loader.import (internal/modules/esm/loader.js:178:24)    at getConfigFileExport (/workspaces/gas-gchanges2notion/node_modules/rollup/dist/shared/loadConfigFile.js:587:17)
        at Object.loadConfigFile (/workspaces/gas-gchanges2notion/node_modules/rollup/dist/shared/loadConfigFile.js:546:59)
        at getConfigs (/workspaces/gas-gchanges2notion/node_modules/rollup/dist/bin/rollup:1679:39)    at runRollup (/workspaces/gas-gchanges2notion/node_modules/rollup/dist/bin/rollup:1656:43)

`__dirname` が使えないのは[以前に自作モジュールを native ESM 対応したとき](https://zenn.dev/hankei6km/articles/native-esm-with-typescript-jest)にそういう記述を見かけていました。  探してみるとやはりあったので参考にしながらソースを修正することで対応できました。

@[card](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c#what-do-i-use-instead-of-__dirname-and-__filename)

**リスト 2-1 `__dirname` の代わりに `import.meta` を利用**

```diff ts
diff --git a/rollup.config.js b/rollup.config.js
index 11ddd21..1c35230 100644
--- a/rollup.config.js
+++ b/rollup.config.js
@@ -4,6 +4,7 @@ import { nodeResolve } from '@rollup/plugin-node-resolve'
 import json from '@rollup/plugin-json'
 import license from 'rollup-plugin-license'
 import * as path from 'path'
+import { fileURLToPath } from 'node:url'
 
 const extensions = ['.ts', '.js']
 
@@ -41,7 +42,11 @@ export default {
       thirdParty: {
         // includePrivate: true, // Default is false.
         output: {
-          file: path.join(__dirname, 'build', 'OPEN_SOURCE_LICENSES.txt'),
+          file: path.join(
+            path.dirname(fileURLToPath(import.meta.url)),
+            'build',
+            'OPEN_SOURCE_LICENSES.txt'
+          ),
           encoding: 'utf-8' // Default is utf-8.
         }
       }
```

基本的にこれにて一件落着なのでが、ちょっと気になるのは `import.meta` が使えるようになったはわりと最近だったはずです[^import-meta]。roolup.js の設定ファイルに対して、この辺を指定する方法はわかりませんでした。

@[card](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/import.meta)
@[card](https://numb86-tech.hatenablog.com/entry/2020/08/08/232535)

[^import-meta]: `import.meta` は以前に別のところで使おうとしたら、対応していないツールなどがあって利用をあきらめた記憶があります。

## おわりに

`rollup.config.js` が ESM として扱われるようになったので、その対応方法を見てみました。

今回はそこそこ簡単に対応できましたが、CJS と ESM 関連は早く落ち着いてほしいなと改めて思いました。
