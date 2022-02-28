---
id: promise-or-abort-controller
title: 順次実行される非同期処理(コンテキスト)を停止させるには Promise か AbortContoller か？
emoji: 🎬
type: idea
topics:
  - javascript
  - typescript
published: true
---

最初は [Promise のメモの記事](https://zenn.dev/hankei6km/articles/promise-memo)内に書いていたのですが、少し毛色が違うのわけました。

## まえおき

複数 Promise を並列処理させるモジュール([`chanpuru`] )を作っていて、実際にモジュールを使ってみると停止させる機能が欲しくなりました[^go-context]。

いまのところは Promise をベースにして AbortController を使う形にしていますが、正直なところ「いまひとつしっくりこない部分もある」といった感じです。

[^go-context]: Go の channel に影響されているので、やはり Context も欲しくなったという流れです。

## 環境

主に Node.js v16 で試しています。

v14 では AbortConroller は使えないので別途 polyfill 用のパッケージを導入してください。

:::details (クリックで設定などを表示)

```shell-session
$ node --version
v16.13.2

$ npx tsc --version
Version 4.5.5
```

```json:package.json
{
  "name": "promise-memo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "directories": {
    "example": "examples"
  },
  "scripts": {
    "test": "node --loader ts-node/esm examples/memo/promise_is_pending.ts"
  },
  "author": "",
  "license": "MIT",
  "devDependencies": {
    "@types/node": "^17.0.16",
    "ts-node": "^10.5.0",
    "typescript": "^4.5.5"
  }
}
```

```json:tsconfig.json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "es2022",
    "moduleResolution": "node",
    "strict": true,
    "skipLibCheck": true,
    "declaration": true,
    "pretty": true,
    "newLine": "lf",
    "outDir": "dist",
    "esModuleInterop": true
  },
  "exclude": ["node_modules", "src/**/*.test.ts"],
  "include": [
    "examples/**/*.ts",
    "src/**/*.ts",
    "test/**/*.spec.ts",
    "src/**/*.tsx"
  ]
}
```

:::

## サンプル

各サンプルは以下のリポジトリから利用できます。

なお、発端は [`chanpuru`] ですが、サンプルは [`chanpuru`] を使わずに一般的な方法で記述しています。

@[card](https://github.com/hankei6km/promise-or-abort-controller)

## Promise だけの場合

まず Promise だけで記述する場合を考えてみます。

1.  メイン処理側で停止用 Promise を作成
2.  ループの外側で停止用 Promise へハンドラーを設定、Promise 決定時の理由を確認したらフラグを変更してループを停止
3.  非同期処理内では `Promise.race` を使うことでその場限りのハンドラーを設定[^persist-handler]、タイムアウトのエラーが返却されたら非同期処理を停止
4.  全ての処理が完了したら(クリーンアップとして)停止用 Promise を決定状態にする

以下のサンプルは少し長めですが、`async function proc(timeoutPromise: Promise<void>)` が実際の処理部分になります。

[^persist-handler]: 停止用 Promise へ直接ハンドラーを設定してしまうと各非同期処理が完了してもハンドラーは設定されたままです。この場合、停止用 Promise が決定状態になるとそれらが一斉に動作します。

:::details (クリックでサンプル表示)

```ts:chain-in-loop.ts
class cancelByTimeoutError extends Error {
  constructor(message: string) {
    //https://stackoverflow.com/questions/41102060/typescript-extending-error-class
    super(message)
    Object.setPrototypeOf(this, cancelByTimeoutError.prototype)
  }
  get reason() {
    return this.message
  }
}

function cancelPromise(timeout: number): [Promise<void>, () => void] {
  let c: () => void
  const p = new Promise<void>((resolve, reject) => {
    c = () => {
      if (id) {
        id = undefined
        clearTimeout(id)
      }
      resolve()
    }
    let id: any = setTimeout(() => {
      id = undefined
      reject(new cancelByTimeoutError('timeout'))
    }, timeout)
  })
  return [p, () => c()]
}

async function proc(timeoutPromise: Promise<void>) {
  let cancelled = false
  timeoutPromise
    .catch((r) => {
      if (r instanceof cancelByTimeoutError) {
        console.log(`catch ${r}`)
      }
    })
    .finally(() => {
      cancelled = true
    })
  // 非同期処理の定義
  const asyncProc = (timeoutPromise: Promise<void>, idx: number) => {
    let id: any
    let pickReject: (r: any) => void
    return Promise.race([
      timeoutPromise,
      new Promise<string>((resolve, reject) => {
        pickReject = reject
        id = setTimeout(() => {
          id = undefined
          resolve(`done ${idx}`)
        }, 100)
      })
    ]).catch((r) => {
      if (r instanceof cancelByTimeoutError) {
        if (id) {
          clearTimeout(id)
          id = undefined
          console.log(`clear ${idx}`)
        }
        pickReject(`cancel resone: ${r.reason} ${idx}`) // race の後なのでこの内容が伝わることはない(クリーンアップ用).
        return Promise.reject(`cancel reasone: ${r.reason} (handler in race)`)
      }
      return Promise.reject(r)
    })
  }
  for (let idx = 0; !cancelled && idx < 10; idx++) {
    // 非同期処理の実行.
    await asyncProc(timeoutPromise, idx)
      .then((v: string | void) => {
        console.log(`then: ${v}`)
      })
      .catch((r: any) => {
        console.log(`catch: ${r}`)
      })
  }
}

{
  console.log('==== start timeout=2000')
  const [timeoutPromise, cancel] = cancelPromise(2000)
  timeoutPromise.catch(() => {
    console.log('---timeout')
  })

  await proc(timeoutPromise)

  cancel()
  console.log('done')
}
console.log('')
{
  console.log('==== start timeout=500')
  const [timeoutPromise, cancel] = cancelPromise(500)
  timeoutPromise.catch(() => {
    console.log('---timeout')
  })

  await proc(timeoutPromise)

  cancel()
  console.log('done')
}

export {}
```

```shell-session
$ node --loader ts-node/esm src/chain-in-loop.ts 
==== start timeout=2000
then: done 0
then: done 1
then: done 2
then: done 3
then: done 4
then: done 5
then: done 6
then: done 7
then: done 8
then: done 9
done

==== start timeout=500
then: done 0
then: done 1
then: done 2
then: done 3
---timeout
catch Error: timeout
clear 4
catch: cancel reasone: timeout (handler in race)
done
```

:::

これでも動作しますが、非同期処理の結果を取りだすのが少し手間です。また、結果の型にも影響が出ます(リスト 4-1)。やはり、通常の結果とキャンセルの通知が `Promise.race` のハンドラーに集約されるとやりにくい感じはします。

▼ リスト 4-1 *then の v が string | void になっている*

```ts
// 非同期処理の実行.
await asyncProc(timeoutPromise, idx)
  .then((v: string | void) => {
    console.log(`then: ${v}`)
  })
  .catch((r: any) => {
    console.log(`catch: ${r}`)
  })
```

また `Promise.race` は配列内の配置(優先度)を考えないと停止用 Promise の決定が伝わらない可能性もあります [^handlers]。

[^handlers]: あとは`Promise.race` 内での Chain の扱いによってはリソースが圧迫される可能性などもあります。

## AbortController を組み合わせる

上記処理のハンドラーまわりを改善するために AbortController の `signal` を組み合わせてみます。

`signal` では `signal.aborted` で状態の(同期的な)取得と、`signal` へ設定したハンドラーの明示的な開放ができます。

これを利用して以下のように変更します。

1.  AbortController を作成し、停止用 Promise が決定されたら `abort()` を実行

2.  ループをブレイクさせるために `signal.aborted` を利用

3.  非同期処理では開始時に `signal.aborted` を確認後、` signal` のハンドラーを設定

    *   ハンドラーは非同期処理が完了したら開放する

:::details (クリックでサンプル表示)

```ts:signal-in-loop.ts
class cancelByTimeoutError extends Error {
  constructor(message: string) {
    //https://stackoverflow.com/questions/41102060/typescript-extending-error-class
    super(message)
    Object.setPrototypeOf(this, cancelByTimeoutError.prototype)
  }
  get reason() {
    return this.message
  }
}

function cancelPromise(timeout: number): [Promise<void>, () => void] {
  let c: () => void
  const p = new Promise<void>((resolve, reject) => {
    c = () => {
      if (id) {
        id = undefined
        clearTimeout(id)
      }
      resolve()
    }
    let id: any = setTimeout(() => {
      id = undefined
      reject(new cancelByTimeoutError('timeout'))
    }, timeout)
  })
  return [p, () => c()]
}

async function proc(timeoutPromise: Promise<void>) {
  const ac = new AbortController()
  let cancelReason: any
  timeoutPromise
    .catch((r) => {
      if (r instanceof cancelByTimeoutError) {
        console.log(`catch ${r}`)
        cancelReason = r.reason
      }
    })
    .finally(() => {
      ac.abort()
    })
  // 非同期処理の定義.
  const asyncProc = (signal: AbortSignal, idx: number) =>
    new Promise<string>((resolve, reject) => {
      let id: any
      const handleAbort = () => {
        if (id) {
          id = undefined
          clearTimeout(id)
          console.log(`clear ${idx}`)
        }
        reject(`cancel reason: ${cancelReason} ${idx}`)
      }
      if (!signal.aborted) {
        id = setTimeout(() => {
          id = undefined
          signal.removeEventListener('abort', handleAbort)
          resolve(`done ${idx}`)
        }, 100)
        signal.addEventListener('abort', handleAbort, { once: true })
      } else {
        reject(`aborted reason: ${cancelReason} ${idx}`)
      }
    })
  for (let idx = 0; !ac.signal.aborted && idx < 10; idx++) {
    // 非同期処理の実行.
    await asyncProc(ac.signal, idx)
      .then((v: string) => {
        console.log(`then: ${v}`)
      })
      .catch((r: any) => {
        console.log(`catch: ${r}`)
      })
  }
}

{
  console.log('==== start timeout=2000')
  const [timeoutPromise, cancel] = cancelPromise(2000)
  timeoutPromise.catch(() => {
    console.log('---timeout')
  })

  await proc(timeoutPromise)

  cancel()
  console.log('done')
}
console.log('')
{
  console.log('==== start timeout=500')
  const [timeoutPromise, cancel] = cancelPromise(500)
  timeoutPromise.catch(() => {
    console.log('---timeout')
  })

  await proc(timeoutPromise)

  cancel()
  console.log('done')
}

export {}
```

```shell-session
$ node --loader ts-node/esm src/signal-in-loop.ts
==== start timeout=2000
then: done 0
then: done 1
then: done 2
then: done 3
then: done 4
then: done 5
then: done 6
then: done 7
then: done 8
then: done 9
done

==== start timeout=500
then: done 0
then: done 1
then: done 2
then: done 3
---timeout
catch Error: timeout
clear 4
catch: cancel reason: timeout 4
done
```

:::

Node v16 の AbortController は `abort(reason)` が使えないので停止理由の受け渡しがあまりよくないですが 、ハンドラーまわりの取り回しは改善されたかと思われます。

## AbortController だけの場合は？

停止理由を(Node v16 では)保持できなかったので保留にしました。

## おわりに

最初に書いたようにいまひとつしっくりこない部分もありますが、少し整理したかったのでアウトプットしておくことにしました。

[`chanpuru`]: https://github.com/hankei6km/chanpuru/blob/main/README_ja.md
