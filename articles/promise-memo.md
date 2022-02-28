---
id: promise-memo
title: ふわっとした理解で Promise(と Async Generator) を使っていたらいろいろハマってしまったのでメモ
emoji: 📝
type: tech
topics:
  - javascript
  - typescript
published: true
---

使わないとすぐに忘れてしまうのでメモ。

## まえおき

このメモに記述した内容は以下に引用した部分をおさえておくとしっくりくることが多いと思われます。

逆に言うとこのメモは以下のことよく理解していなかった人間が書いているので、アンチパターンもりもりの可能性が高いです。 (「[JavaScript Promise の本](https://azu.github.io/promises-book/)」は少し前に読み始めて、それまでメモしていた内容のいくつかが「アンチパターンだったとは」と涙目になっています)

> 待機状態のプロミスは、何らかの値を持つ履行 (fulfilled) 状態、もしくは何らかの理由 (エラー) を持つ拒否 (rejected) 状態のいずれかに変わります。そのどちらとなっても、then メソッドによって関連付けられたハンドラーが呼び出されます。対応するハンドラーが割り当てられたとき、既にプロミスが履行または拒否状態になっていても、そのハンドラーは呼び出されます。よって、非同期処理とその関連付けられたハンドラーとの競合は発生しません。

@[card](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise#%E8%A7%A3%E8%AA%AC)

> しかし、実際には then で新しいpromiseオブジェクト、catch でも別の新しいpromiseオブジェクトを作成して返しています。

@[card](https://azu.github.io/promises-book/#then-return-new-promise)

## サンプル

各サンプルは以下のリポジトリから利用できます。

@[card](https://github.com/hankei6km/promise-memo)

## 環境

今回のメモは主に Node.js v16 で試しています。

Node.js v16 と v14 では `UnhandledPromiseRejection` の扱いが異なるので注意してください(v14 では警告が表示されるだけで処理は継続)。

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

## Promise の生成

### コールバックは(同期的に)即座に開始される

`new Promise(cb)` を実行すればハンドラーを設定しなくても `cb` は開始されています。

以下のサンプルでは `step0` が表示される前に `cb start` が表示されているので、同期的に実行されていることも確認できます。

:::details (クリックでサンプル表示)

```ts:cb_call_immediately.ts
;(async () => {
  const wait = (to: number) =>
    new Promise<void>((resolve) => setTimeout(() => resolve(), to))

  const p = new Promise<string>((resolve) => {
    console.log('cb start')
    setTimeout(() => resolve('done'), 1000)
  })
  console.log('step0')
  await wait(2000)
  console.log('step1')
  await p.then((v) => console.log(v))
  console.log('step2')
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/cb_call_immediately.ts
cb start
step0
step1
done
step2
```

:::

### `then()` などによるハンドラー設定は新しい Promise(のインスタンス)を返す

ここをおさえておくと後のことが理解しやすくなります。

:::details (クリックでサンプル表示)

```ts:then_return_new_promise.ts
;(async () => {
  const wait = (to: number) =>
    new Promise<void>((resolve) => setTimeout(() => resolve(), to))

  const p = new Promise<string>((resolve) => {
    setTimeout(() => resolve('done'), 1000)
  })

  const t1 = p.then((v) => v)
  const t2 = p.then((v) => v)
  console.log(`t1 === t2 : ${t1 === t2}`)
  console.log(`t1: ${await t1}`)
  console.log(`t2: ${await t2}`)
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/then_return_new_promise.ts
t1 === t2 : false
t1: done
t2: done
```

:::

## ハンドラーと Chain

### `then()` `await` は同時に複数回利用でき、決定後も利用できる(が、アンチパターンらしい)

ただし、決定された状態は変更できません。

なお、複数利用については以下のような点がアンチパターンとなるようです。

*   Chain が分岐するのでコードの記述順に状態が受け渡されるわけではない(同時利用している箇所で同じ状態が取得される)
*   reject が catch されない Chain が出来てしまう可能性

「トリガーとして複数の場所であえて同じ状態を取得したい」といったときはアンチパターンなのかはざっと調べた限りではわかりませんでした。

:::details (クリックでサンプル表示)

```ts:then_await_call_multiple.ts
;(async () => {
  const wait = (to: number) =>
    new Promise<void>((resolve) => setTimeout(() => resolve(), to))

  let r: (value: string) => void = (v) => {
    console.log('called before settled')
  }
  const p = new Promise<string>((resolve) => {
    console.log('cb start')
    setTimeout(() => {
      resolve('done')
      r = resolve
    }, 1000)
  })

  ;(async () => {
    p.then((v) => console.log(`then-1: ${v}`))
  })()
  ;(async () => {
    console.log(`await-1: ${await p}`)
  })()
  ;(async () => {
    p.then((v) => console.log(`then-2: ${v}`))
  })()
  ;(async () => {
    console.log(`await-2: ${await p}`)
  })()

  await p
  console.log('another resolve()')
  r('done-2')
  ;(async () => {
    p.then((v) => console.log(`then-1: ${v}`))
  })()
  ;(async () => {
    console.log(`await-1: ${await p}`)
  })()
  ;(async () => {
    p.then((v) => console.log(`then-2: ${v}`))
  })()
  ;(async () => {
    console.log(`await-2: ${await p}`)
  })()
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/then_await_call_multiple.ts
cb start
then-1: done
await-1: done
then-2: done
await-2: done
another resolve()
then-1: done
await-1: done
then-2: done
await-2: done
```

::::

### `catch()` も複数同時の利用と決定後の利用が可能

基本は `then()` `await` と同じ。  `catch()` のハンドラーが実行されるだけでなく、`try-catch` で `await` しても throw されました。

:::details (クリックでサンプル表示)

```ts:catch_multiple.ts
;(async () => {
  const wait = (to: number) =>
    new Promise<void>((resolve) => setTimeout(() => resolve(), to))

  const p = new Promise<string>((resolve, reject) => {
    console.log('cb start')
    setTimeout(() => reject('rejected'), 1000)
  })

  console.log('before settled')
  ;(async () => {
    p.catch((v) => console.log(`catch-1: ${v}`))
  })()
  ;(async () => {
    try {
      await p
    } catch (r) {
      console.log(`await-1: ${r}`)
    }
  })()
  ;(async () => {
    p.catch((v) => console.log(`catch-2: ${v}`))
  })()
  ;(async () => {
    try {
      await p
    } catch (r) {
      console.log(`await-2: ${r}`)
    }
  })()

  await p.catch((r) => r)

  console.log('after settled')
  ;(async () => {
    p.catch((v) => console.log(`catch-1: ${v}`))
  })()
  ;(async () => {
    try {
      await p
    } catch (r) {
      console.log(`await-1: ${r}`)
    }
  })()
  ;(async () => {
    p.catch((v) => console.log(`catch-2: ${v}`))
  })()
  ;(async () => {
    try {
      await p
    } catch (r) {
      console.log(`await-2: ${r}`)
    }
  })()
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/catch_multiple.ts
cb start
before settled
catch-1: rejected
await-1: rejected
catch-2: rejected
await-2: rejected
after settled
catch-1: rejected
await-1: rejected
catch-2: rejected
await-2: rejected
```

:::

### Promise 決定後に再利用しても chain を再実行するわけではない

決定された Promise に新しいハンドラーを設定すると実行されましたが、既に決定された状態は各インスタンスが保持しているようです。

:::details (クリックでサンプル表示)

```ts:chain-not-replay.ts 
const p = new Promise((resolve) => setTimeout(() => resolve('p')))
const t1 = p.then((v) => {
  console.log('t1')
  return `${v}-t1`
})
const t2 = t1.then((v) => {
  console.log('t2')
  return `${v}-t2`
})
const t3 = t2.then((v) => {
  console.log('t3')
  return `${v}-t3`
})

console.log(await t3)
console.log(await t3)
const t4 = t3.then((v) => {
  console.log('t4')
  return `${v}-t4`
})
console.log(await t4)
console.log(await t2)
export {}
```

```shell-session
$ node --loader ts-node/esm examples/memo/chain-not-replay.ts 
t1
t2
t3
p-t1-t2-t3
p-t1-t2-t3
t4
p-t1-t2-t3-t4
p-t1-t2
```

:::

### 設定したハンドラーを明示的に開放する方法はない(と思われる)

よって、ループや関数の中などで設定したハンドラーはループや関数を抜けても開放されませんでした。

原理的には当然と言われればそうなのですが、少し悩ましいです。

たとえば、ループの内の非同期処理でキャンセルのトリガー的に使おうとすると、Promise が決定されるときにループ内で設定されたハンドラーがすべて実行されます[^cancel-by-promise]。

これは以下の点で注意が必要です。

*   処理が完了した思っていたところで不用意にキャンセル処理が実行されている
*   ループの回数が多ければリソースを圧迫する

いくつか対応方法がありますが、不用意な実行については `Promise.race` で回避できました[^abort-controller][^resource]。

[^cancel-by-promise]: これもアンチパターンに分類されそうなので、そもそも「やるな」となりそうですが。

[^abort-controller]: Abort Controller を使う方法を[別の記事で書きました](https://zenn.dev/hankei6km/articles/promise-or-abort-controller)。

[^resource]: \`Promise.race\` の内部的な実装がわからないので憶測ですが、各 Promise の状態を調べるためのハンドラーは設定していると思われます。よって、最終的には何らかのハンドラーが一斉に実行されている可能性はあります。

:::details (クリックでサンプル表示)

```ts:chain-in-loop.ts
const cancel = new Promise<void>((resolve) =>
  setTimeout(() => {
    console.log('--- timeout')
    resolve()
  }, 2000)
)
console.log('==== start')
for (let idx = 0; idx < 10; idx++) {
  await ((idx) => {
    return new Promise<void>((resolve) => {
      let id: any = setTimeout(() => {
        id = undefined
        console.log(`done ${idx}`)
        resolve()
      }, 100)
      cancel.then(() => {
        if (id) {
          clearTimeout(id)
          id = undefined
          console.log(`clear ${idx}`)
        }
        console.log(`cancel ${idx}`)
      })
    })
  })(idx)
}
console.log('done')

await cancel
console.log('')

const cancelWithReject = new Promise<void>((resolve, reject) =>
  setTimeout(() => {
    reject(new Error('timeout with reject'))
  }, 2000)
)
cancelWithReject.catch(() => {
  console.log('---timeout')
})
console.log('==== start(use Promise.race)')
for (let idx = 0; idx < 10; idx++) {
  await ((idx) => {
    let id: any
    return Promise.race([
      cancelWithReject,
      new Promise<void>((resolve) => {
        id = setTimeout(() => {
          id = undefined
          console.log(`done ${idx}`)
          resolve()
        }, 100)
      })
    ]).catch(() => {
      if (id) {
        clearTimeout(id)
        id = undefined
        console.log(`clear ${idx}`)
      }
      console.log(`cancel ${idx}`)
    })
  })(idx)
}
console.log('done')

export {}
```

```shell-session
$ node --loader ts-node/esm examples/memo/chain-in-loop.ts 
==== start
done 0
done 1
done 2
done 3
done 4
done 5
done 6
done 7
done 8
done 9
done
--- timeout
cancel 0
cancel 1
cancel 2
cancel 3
cancel 4
cancel 5
cancel 6
cancel 7
cancel 8
cancel 9

==== start(use Promise.race)
done 0
done 1
done 2
done 3
done 4
done 5
done 6
done 7
done 8
done 9
done
---timeout
```

:::

## reject と catch

### catch とは

この後も何度か catch という単語が出てきますが、ここでは「`UnhandledPromiseRejection` にさせない、プロセスが正常終了する」という意味で使っています。

たとえば `catch()` でハンドラーを設定しても `Promise.reject()` を返せば `UnhandledPromiseRejection` になるか throw されます。

よって、多くは「ハンドラーが `Promise.reject()` 以外を return している」か「`await` を try-catch で囲んでいる(かつ throw しない)」状態を指しています。

なお、reject 以外を return すると履行状態で値が設定されるのでそれはそれで注意が必要です。 (何も return しない場合は `undefined` になります)

:::details (クリックでサンプル表示)

```ts:behavior-catch.ts 
const p = new Promise((resolve, reject) =>
  setTimeout(() => reject('rejected'), 100)
)

console.log('=== return undefined')
try {
  const v = p.catch((r) => {
    console.log(`catch inner ${r}`)
  })
  console.log(`resolved ${await v}`)
} catch (r) {
  console.log(`catch outer ${r}`)
}
console.log('')

console.log('=== return reject')
try {
  const v = p.catch((r) => {
    console.log(`catch inner ${r}`)
    return Promise.reject(r)
  })
  console.log(`resolved ${await v}`)
} catch (r) {
  console.log(`catch outer ${r}`)
}
console.log('')

console.log('=== return undefined(bare)')
const v1 = p.catch((r) => {
  console.log(`catch inner ${r}`)
})
console.log(`resolved ${await v1}`)
console.log('')

console.log('=== return reject(bare)')
const v2 = p.catch((r) => {
  console.log(`catch inner ${r}`)
  return Promise.reject(r)
})
console.log(`resolved ${await v2}`)
console.log('')

export {}
```

```shell-session
$ node --loader ts-node/esm examples/memo/behavior-catch.ts 
=== return undefined
catch inner rejected
resolved undefined

=== return reject
catch inner rejected
catch outer rejected

=== return undefined(bare)
catch inner rejected
resolved undefined

=== return reject(bare)
catch inner rejected
'rejected'

$ echo "$?"
1
```

:::

### Node.js の v16 と v14 で UnhandledPromiseRejection の扱いが異なる

環境のところでも書きましたが、v16 では exitStatus が 1 で停止します。しかし v14 では警告が表示されるだけで処理は継続します。

```shell-session
$ node -e "Promise.reject('rejected'); setTimeout(()=>console.log('=== END ==='),100)"
node:internal/process/promises:265
            triggerUncaughtException(err, true /* fromPromise */);
            ^

[UnhandledPromiseRejection: This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). The promise rejected with the reason "rejected".] {
  code: 'ERR_UNHANDLED_REJECTION'
}
$ echo "${?}"
1

$ nvm use v14
Now using node v14.19.0 (npm v6.14.16)

$ node -e "Promise.reject('rejected'); setTimeout(()=>console.log('=== END ==='),100)"
(node:192180) UnhandledPromiseRejectionWarning: rejected
(Use `node --trace-warnings ...` to show where the warning was created)
(node:192180) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). To terminate the node process on unhandled promise rejection, use the CLI flag `--unhandled-rejections=strict` (see https://nodejs.org/api/cli.html#cli_unhandled_rejections_mode). (rejection id: 1)
(node:192180) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
=== END ===
$ echo "${?}"
0
```

v14 でも警告メッセージもにもある `--unhandled-rejections=strict` フラグを指定するとエラーになります。

```shell-session
$ nvm use v14
Now using node v14.19.0 (npm v6.14.16)

$ node --unhandled-rejections=strict -e "Promise.reject('rejected'); setTimeout(()=>console.log('=== END ==='),100)"
internal/process/promises.js:213
        triggerUncaughtException(err, true /* fromPromise */);
        ^

[UnhandledPromiseRejection: This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). The promise rejected with the reason "rejected".] {
  code: 'ERR_UNHANDLED_REJECTION'
}
$ echo "${?}"
1
```

### reject の chatch は各 Promise のインスタンス(の chain) に 1 つ必要

Chain (の `reejct` される箇所の後)に 1 つあればよいのですが、以下のような場合は `UnhandledPromiseRejection` になるか throw されます。

*   分岐している Chain に `catch()`ハンドラーがない(await を try-catch していない)
*   `catch()` のハンドラー後の chain で `reject` される

以下は Chain が分岐しているので `await p` でエラーが出ます(`await pp` にすると出ない)。

▼ リスト 6-1 *Chain 分岐と catch*

```ts
(async ()=>{
  const p = new Promise((resolve,reject)=>setTimeout(reject,1000))
  const pp = p.catch(r=>console.log('rejected'))
  await p
})()
```

:::details (クリックでサンプル表示)

```ts:each_chain_need_catch.ts 
;(async () => {
  const wait = (to: number) =>
    new Promise<void>((resolve) => setTimeout(() => resolve(), to))

  const p = new Promise<string>((resolve, reject) => {
    console.log('cb start')
    setTimeout(() => reject('rejected'), 1000)
  })

  const tc1 = p.then((v) => {}).catch((r) => r)
  const tc2 = p.then((v) => {})
  ;(async () => {
    console.log('tc1: then catch -> await chain')
    console.log(`tc1: ${await tc1}`)
  })()
  ;(async () => {
    console.log('tc2: then -> await chain')
    console.log(`tc2: ${await tc2}`)
  })()
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/each_chain_need_catch.ts 
cb start
tc1: then catch -> await chain
tc2: then -> await chain
tc1: rejected
[UnhandledPromiseRejection: This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). The promise rejected with the reason "rejected".] {
  code: 'ERR_UNHANDLED_REJECTION'
}
```

:::

### for await...of などを囲む tyr-chatch は、順番が回ってくる前の Promise の reject を catch しない

配列のループ以外に「await する処理が時間差で実行される」などでも同じようになります。当然と言われればそうなのですが、これに一番ハマりました。

なお「配列なら `Promise.all` を使うべき」という話になりそうですが、「配列順に結果を受け取るけど、決定した Promise があれば先に処理したい」という場合などを想定しています。たとえば Async Generator で結果を順次渡す場合です。

以下のサンプルでは、1 回目の Async Generator 内の for await...of では外側の try で catch できています。

しかし、2 回目のときは(reject するタイミングを早めているので)順番が回ってくる前の Promise が reject 状態になります。そのため `UnhandledPromiseRejection` になります。

:::details (クリックでサンプル表示)

```ts:need_chain_catch_reject.ts
;(async () => {
  const wait = (to: number) =>
    new Promise<void>((resolve) => setTimeout(() => resolve(), to))

  const promiseArray: () => Promise<string>[] = () =>
    new Array(5).fill('').map(
      (_v, i) =>
        new Promise<string>((resolve) => {
          setTimeout(() => {
            resolve(`done-${i}`)
          }, 100 * (i + 1))
        })
    )

  async function* gen(
    p: Promise<string>[]
  ): AsyncGenerator<string, void, void> {
    try {
      for await (let t of p) {
        yield t
      }
    } catch (r) {
      console.log(`generator catch ${r}`)
    } finally {
      console.log('generator done')
    }
  }

  await (async () => {
    console.log('timeout = 1000')
    try {
      const timeout = 1000
      const p = promiseArray()
      p.splice(
        4,
        0,
        new Promise<string>((resolve, reject) => {
          setTimeout(() => reject('rejected'), timeout)
        })
      )
      const g = gen(p)
      for await (let t of g) {
        console.log(`${t}`)
      }
    } catch (r) {
      console.log(`catch ${r}`)
    }
  })()

  await (async () => {
    console.log('timeout = 200')
    try {
      const timeout = 200
      const p = promiseArray()
      p.splice(
        4,
        0,
        new Promise<string>((resolve, reject) => {
          setTimeout(() => reject('rejected'), timeout)
        })
      )
      const g = gen(p)
      for await (let t of g) {
        console.log(`${t}`)
      }
    } catch (r) {
      console.log(`catch ${r}`)
    }
  })()
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/need_chain_catch_reject.ts 
timeout = 1000
done-0
done-1
done-2
done-3
generator catch rejected
generator done
timeout = 200
done-0
done-1
[UnhandledPromiseRejection: This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). The promise rejected with the reason "rejected".] {
  code: 'ERR_UNHANDLED_REJECTION'
}
```

:::

### ループなどで順番待ちになる Promise の catch

前節の対応的な処理。

`catch()` を Chain した Promise を渡すとハンドラーからの `return` が伝わってしまいます(TypeScript だとそれ以前に型が合わずに渡せないことも多いですが)。

以下のサンプルでは `any` で無理やりわたしていますが、これは `undefined` が `yield` されてしまいます。ならばと `Promise.reject()` を `return` すると再度 reject になるので `UnhandledPromiseRejection` になります。

これは別の Chain になるように `catch()` することで for await...of の前で catch できました。

ただし、ループ側で `await` などが行われるとそちらでも reject 状態になるので(chain が異なるため)、ループ側でも catch が必要となります。

なお、この方法だと Chain が 1 つ増えることになります。また、Chain を分岐させる方法なのでアンチパターンに分類される可能性があります(`then()` をこのように使うのはダメだとありますが、`catch()` だけの場合はどうなのかはわからなかったです)。

:::details (クリックでサンプル表示)

```ts:for_await_of_catch.ts
;(async () => {
  const wait = (to: number) =>
    new Promise<void>((resolve) => setTimeout(() => resolve(), to))

  const promiseArray: () => Promise<string>[] = () =>
    new Array(5).fill('').map(
      (_v, i) =>
        new Promise<string>((resolve) => {
          setTimeout(() => {
            resolve(`done-${i}`)
          }, 100 * (i + 1))
        })
    )

  async function* gen(
    p: Promise<string>[]
  ): AsyncGenerator<string, void, void> {
    try {
      for await (let t of p) {
        yield t
      }
    } catch (r) {
      console.log(`generator catch ${r}`)
    } finally {
      console.log('generator done')
    }
  }

  await (async () => {
    console.log('catch in top of chain')
    const timeout = 200
    const p = promiseArray()
    const r = new Promise<string>((resolve, reject) => {
      setTimeout(() => reject('rejected'), timeout)
    })
    const c = r.catch((r) => {
      console.log(`catch ${r}`)
    })
    p.splice(4, 0, c as any)
    const g = gen(p)
    try {
      for await (let t of g) {
        console.log(`${t}`)
      }
    } catch (r) {
      console.log(`for await...of ${r}`)
    }
  })()
  console.log('---')

  await (async () => {
    console.log('catch in another chain')
    const timeout = 200
    const p = promiseArray()
    const r = new Promise<string>((resolve, reject) => {
      setTimeout(() => reject('rejected'), timeout)
    })
    r.catch((r) => {
      console.log(`catch ${r}`)
    })
    p.splice(4, 0, r)
    const g = gen(p)
    try {
      for await (let t of g) {
        console.log(`${t}`)
      }
    } catch (r) {
      console.log(`for await...of ${r}`)
    }
  })()
  console.log('---')
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/for_await_of_catch.ts 
catch in top of chain
done-0
done-1
catch rejected
done-2
done-3
undefined
done-4
generator done
---
catch in another chain
done-0
done-1
catch rejected
done-2
done-3
generator catch rejected
generator done
---
```

:::

## Promise.race

### Promise.race に同一の Promise を何度も渡すことができる

ここまでの挙動を見ていれば普通のことですが、「一度 `Promise.race` を通したらハンドラーは動作しない」と勘違いしていました。

以下は、少しわかりにくいのですが同一の Promise を何度か渡しても「そのときの配列内の Promise の状態」により挙動が変化しています。

この挙動については次節でメモしています。

:::details (クリックでサンプル表示)

```ts:reuse_promise_in_race.ts
;(async () => {
  const wait = (to: number) =>
    new Promise<void>((resolve) => setTimeout(() => resolve(), to))

  const p = [
    new Promise<string>((resolve) => setTimeout(() => resolve('A'), 4000)),
    new Promise<string>((resolve) => setTimeout(() => resolve('B'), 3000)),
    new Promise<string>((resolve) => setTimeout(() => resolve('C'), 2000)),
    new Promise<string>((resolve) => setTimeout(() => resolve('D'), 1000))
  ]
  console.log(await Promise.race(p))
  await wait(1010)
  console.log(await Promise.race(p))
  await wait(1010)
  console.log(await Promise.race(p))
  await wait(1010)
  console.log(await Promise.race(p))
  await wait(1010)
  console.log(await Promise.race(p))
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/reuse_promise_in_race.ts
D
C
B
A
A
```

:::

### Promise.race は配列の先頭側を優先する

前節の補足です。

`Promise.race` は「各 Promise が実際に決定状態になった時刻を比べるということは**していなさそう**」です。

以下のサンプルでは resolve される Promise は後から決定されます。 しかし、配列の先頭にセットされているので、2 回目の `Promise.race` ではそちらが返却されています。

配列内の配置や `Promise.race` 実行のタイミングによって「先に決定されているはずの `Promise` の値が取得できない」ということはありそうです。

:::details (クリックでサンプル表示)

```ts:promise_race_select_fulfilled.ts
;(async () => {
  const wait = (to: number) =>
    new Promise<void>((resolve) => setTimeout(() => resolve(), to))

  const p = [
    new Promise<string>((resolve) =>
      setTimeout(() => resolve('resolved'), 1000)
    ),
    Promise.reject('rejected').catch((r) => r)
  ]
  console.log(await Promise.race(p))
  await wait(1010)
  console.log(await Promise.race(p))
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/promise_race_select_fulfilled.ts
rejected
resolved
```

:::

## Async Generator

### Async Generator の yield に Promise を渡すと awaited にされる

TypeScript では `Promise<T>` を `yield` すると、生成される `value` フィールドは `T` か `Awaited<T>` にしないとエラーになるのでそういうもののようです。

念の為 for await...of を使わずに `next()` から返ってくる値を使ってみましたが以下のようになりました。

1.  Promise が返ってくる
2.  `await` してみると `value` フィールドも `T` になっている

:::details (クリックでサンプル表示)

```ts:async_generator_yeild_await.ts
export {}
const wait = (to: number) =>
  new Promise<void>((resolve) => setTimeout(() => resolve(), to))

const promiseArray: () => Promise<string>[] = () =>
  new Array(5).fill('').map(
    (_v, i) =>
      new Promise<string>((resolve) => {
        setTimeout(() => {
          resolve(`done-${i}`)
        }, 100 * (i + 1))
      })
  )

async function* asyncGen(
  p: Promise<string>[]
): AsyncGenerator<Awaited<string>, void, void> {
  // ): AsyncGenerator<string, void, void> {
  // ): AsyncGenerator<Promise<string>, void, void> { これはエラー
  await wait(100)
  yield p[0]
  await wait(100)
  yield p[1]
  await wait(100)
  yield p[2]
  await wait(100)
  yield p[3]
  await wait(100)
  yield p[4]
  await wait(100)
}

function* syncGen(
  p: Promise<string>[]
): Generator<Promise<string>, void, void> {
  yield p[0]
  yield p[1]
  yield p[2]
  yield p[3]
  yield p[4]
}

;(async () => {
  await (async () => {
    console.log('async generator with for await...of')
    for await (let t of asyncGen(promiseArray())) {
      console.log(`${t}`)
      console.log(`awaited ${await t}`)
    }
  })()

  await (async () => {
    console.log('async generator without for await...of')
    const i = asyncGen(promiseArray())
    const v = i.next()
    console.log(`i.next() = ${v}`)
    let t = await v
    while (!t.done) {
      console.log(`${t.value}`)
      t = await i.next()
    }
  })()

  await (async () => {
    console.log('sync generator')
    for (let t of syncGen(promiseArray())) {
      console.log(`${t}`)
      console.log(`awaited ${await t}`)
    }
  })()

  await (async () => {
    console.log('sync generator with for await...of')
    for await (let t of syncGen(promiseArray())) {
      console.log(`${t}`)
      console.log(`awaited ${await t}`)
    }
  })()
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/async_generator_yeild_await.ts
async generator with for await...of                                                                       
done-0                                                                                                    
awaited don
awaited done-1
done-2
awaited done-2
done-3
awaited done-3
done-4
awaited done-4
async generator without for await...of
i.next() = [object Promise]
done-0
done-1
done-2
done-3
done-4
sync generator
[object Promise]
awaited done-0
[object Promise]
awaited done-1
[object Promise]
awaited done-2
[object Promise]
awaited done-3
[object Promise]
awaited done-4
sync generator with for await...of
done-0
awaited done-0
done-1
awaited done-1
done-2
awaited done-2
done-3
awaited done-3
done-4
awaited done-4
```

:::

### Async Generator の next() を複数同時に await した場合、反復中はそれぞれの値が順次わたされる(ただし、反復後は異なる挙動になる)

反復している間はそれぞれの `next()` に値が順次わたされます。

ただし、反復後は「最初の `next()` は return された値を value フィールドで受け取る」「それ以降の value フィールドは `undefined`」となりました。

これは (同時に await していることは関係ない)Generator の仕様だと思われますが、return に終了以外の意味がある場合は注意が必要です。

:::details (クリックでサンプル表示)

```ts:multiple-next-called-serial.ts
const wait = (to: number) =>
  new Promise<void>((resolve) => setTimeout(() => resolve(), to))

async function* gen(init: number = 0): AsyncGenerator<number, number, void> {
  await wait(100)
  yield Promise.resolve(init++)
  await wait(100)
  yield Promise.resolve(init++)
  await wait(100)
  yield Promise.resolve(init++)
  await wait(100)
  yield Promise.resolve(init++)
  await wait(100)
  return 9999
}

const g = gen()
await Promise.all([
  (async () => {
    console.log(`next 1: wait`)
    console.log(`next 1: ${JSON.stringify(await g.next())}`)
  })(),
  (async () => {
    console.log(`next 2: wait`)
    console.log(`next 2: ${JSON.stringify(await g.next())}`)
  })(),
  (async () => {
    console.log(`next 3: wait`)
    console.log(`next 3: ${JSON.stringify(await g.next())}`)
  })(),
  (async () => {
    console.log(`next 4: wait`)
    console.log(`next 4: ${JSON.stringify(await g.next())}`)
  })(),
  (async () => {
    console.log(`next 5: wait`)
    console.log(`next 5: ${JSON.stringify(await g.next())}`)
  })(),
  (async () => {
    console.log(`next 6: wait`)
    console.log(`next 6: ${JSON.stringify(await g.next())}`)
  })(),
  (async () => {
    console.log(`next 7: wait`)
    console.log(`next 7: ${JSON.stringify(await g.next())}`)
  })(),
  (async () => {
    console.log(`next 8: wait`)
    console.log(`next 8: ${JSON.stringify(await g.next())}`)
  })()
])

console.log('')
const g1 = gen(10)
await Promise.all([
  (async () => {
    for await (const i of g1) {
      console.log(`loop-1 ${i}`)
    }
  })(),
  (async () => {
    for await (const i of g1) {
      console.log(`loop-2 ${i}`)
    }
  })()
])
export {}
```

```shell-session
$ node --loader ts-node/esm examples/memo/multiple-next-called-serial.ts
next 1: wait
next 2: wait
next 3: wait
next 4: wait
next 5: wait
next 6: wait
next 7: wait
next 8: wait
next 1: {"value":0,"done":false}
next 2: {"value":1,"done":false}
next 3: {"value":2,"done":false}
next 4: {"value":3,"done":false}
next 5: {"value":9999,"done":true}
next 6: {"done":true}
next 7: {"done":true}
next 8: {"done":true}

loop-1 10
loop-2 11
loop-1 12
loop-2 13
```

:::

### Async Generator 側で yield の reject を catch しないと、Generator 利用側の next() で reject となる

少し厄介な挙動ですが、Generator で try-catch しておくと Generator の外に reject は伝わらないです。これは for await...of を使っても、直接 `next()` を実行しても同じでした。

なお、(個人的には reject は Generator 内に限定したいですが)以下のようにすると catch しながら `next()` でも reject にできます。

▼ リスト 8-1 *catch と reject*

```ts
try {
  yield p
} catch (r) {
  yield Promise.reject(r)
}
```

同期的な Generator では `yield` を素通りするので引用の通り(おそらくは、for...of を抜けるときに `return()` されていると思われる)。

> 同期ジェネレーター関数の finally ブロックが常に呼び出されるようにするには、非同期のジェネレーター関数の場合は for await...of を、同期ジェネレーター関数の場合は for...of を使用し、ループの中で生成されたプロミスを明示的に待つようにしてくださ

@[card](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/for-await...of)

:::details (クリックでサンプル表示)

```ts:async_generator_reject.ts
export {}
const wait = (to: number) =>
  new Promise<void>((resolve) => setTimeout(() => resolve(), to))

const promiseArray: () => Promise<string>[] = () =>
  new Array(5).fill('').map(
    (_v, i) =>
      new Promise<string>((resolve) => {
        setTimeout(() => {
          resolve(`done-${i}`)
        }, 100 * (i + 1))
      })
  )

async function* asyncGen(
  p: Promise<string>[]
): AsyncGenerator<Awaited<string>, void, void> {
  try {
    await wait(100)
    yield p[0]
    await wait(100)
    yield p[1]
    await wait(100)
    yield p[2]
    await wait(100)
    yield p[3]
    await wait(100)
    yield p[4]
    await wait(100)
    yield p[5]
    await wait(100)
    console.log('async generator done')
  } finally {
    console.log(`async generator: finally`)
  }
}

async function* asyncGenCatch(
  p: Promise<string>[]
): AsyncGenerator<Awaited<string>, void, void> {
  try {
    await wait(100)
    yield p[0]
    await wait(100)
    yield p[1]
    await wait(100)
    yield p[2]
    await wait(100)
    yield p[3]
    await wait(100)
    yield p[4]
    await wait(100)
    yield p[5]
    await wait(100)
    console.log('async generator done')
  } catch (r) {
    console.log(`async generator: ${r}`)
  } finally {
    console.log(`async generator: finally`)
  }
}

function* syncGen(
  p: Promise<string>[]
): Generator<Promise<string>, void, void> {
  try {
    yield p[0]
    yield p[1]
    yield p[2]
    yield p[3]
    yield p[4]
    yield p[5]
    console.log('sync generator done')
  } catch (r) {
    console.log(`sync generator: ${r}`)
  } finally {
    console.log(`sync generator: finally`)
  }
}

;(async () => {
  await (async () => {
    console.log('async generator')
    const p = promiseArray()
    const r = new Promise<string>((resolve, reject) => {
      setTimeout(() => reject('rejected'), 1000)
    })
    r.catch((r) => console.log(`catch ${r}`))
    p.splice(4, 0, r)
    try {
      for await (let t of asyncGen(p)) {
        try {
          console.log(`${t}`)
        } catch (r) {
          console.log(`loop: ${r}`)
          throw r
        }
      }
    } catch (r) {
      console.log(`for await...of: ${r}`)
    }
  })()
  console.log('---')

  await (async () => {
    console.log('async generator without for await...of')
    const p = promiseArray()
    const r = new Promise<string>((resolve, reject) => {
      setTimeout(() => reject('rejected'), 1000)
    })
    r.catch((r) => console.log(`catch ${r}`))
    p.splice(4, 0, r)
    try {
      const i = asyncGen(p)
      let t = await i.next()
      while (!t.done) {
        try {
          console.log(`${t.value}`)
          t = await i.next()
        } catch (r) {
          console.log(`loop: ${r}`)
          throw r
        }
      }
    } catch (r) {
      console.log(`while: ${r}`)
    }
  })()
  console.log('---')

  await (async () => {
    console.log('async generator with catch in generator')
    const p = promiseArray()
    const r = new Promise<string>((resolve, reject) => {
      setTimeout(() => reject('rejected'), 1000)
    })
    r.catch((r) => console.log(`catch ${r}`))
    p.splice(4, 0, r)
    try {
      for await (let t of asyncGenCatch(p)) {
        try {
          console.log(`${t}`)
        } catch (r) {
          console.log(`loop: ${r}`)
          throw r
        }
      }
    } catch (r) {
      console.log(`for await...of: ${r}`)
    }
  })()
  console.log('---')

  await (async () => {
    console.log('sync generator with for await...of')
    const p = promiseArray()
    const r = new Promise<string>((resolve, reject) => {
      setTimeout(() => reject('rejected'), 1000)
    })
    r.catch((r) => console.log(`catch ${r}`))
    p.splice(4, 0, r)
    try {
      for await (let t of syncGen(p)) {
        try {
          console.log(`${t}`)
        } catch (r) {
          console.log(`loop: ${r}`)
          throw r
        }
      }
    } catch (r) {
      console.log(`for await...of: ${r}`)
    }
  })()
  console.log('---')

  await (async () => {
    console.log('sync generator with for...of')
    const p = promiseArray()
    const r = new Promise<string>((resolve, reject) => {
      setTimeout(() => reject('rejected'), 1000)
    })
    r.catch((r) => console.log(`catch ${r}`))
    p.splice(4, 0, r)
    try {
      for (let t of syncGen(p)) {
        try {
          console.log(`awaited ${await t}`)
        } catch (r) {
          console.log(`loop: ${r}`)
          throw r
        }
      }
    } catch (r) {
      console.log(`for...of: ${r}`)
    }
  })()
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/async_generator_reject.ts 
async generator
done-0
done-1
done-2
done-3
catch rejected
async generator: finally
for await...of: rejected
---
async generator without for await...of
done-0
done-1
done-2
done-3
catch rejected
async generator: finally
loop: rejected
while: rejected
---
async generator with catch in generator
done-0
done-1
done-2
done-3
catch rejected
async generator: rejected
async generator: finally
---
sync generator with for await...of
done-0
done-1
done-2
done-3
catch rejected
for await...of: rejected
---
sync generator with for...of
awaited done-0
awaited done-1
awaited done-2
awaited done-3
catch rejected
loop: rejected
sync generator: finally
for...of: rejected
```

:::

### Async Generator 経由で Promise (awaitedでない)を渡す

(Asnc Generator の使い方としての良い悪いはあるとしても)「配列などの型か関数で囲む」ことで対応できました。

配列と関数どちらが良いかは目的などにもよりますが、関数を使った場合は Promise の生成を遅延させることができます。

以下の場合、前者は `const p = new Promise(cb)` の時点で生成されますが、後者では「関数が実行される」ときまで生成されません。

▼ リスト 8-2 *関数で Promise 生成の遅延*

```ts
const p = new Promise(cb)
yield () => p

yield () => new Promise(cb)
```

ただし、`yeild` で await されないのでそのままでは Generator 側で catch できません。これにより同期的 Generator の「finally に到達しない」ような問題が発生します(Generator の外側から `return()` してもらわないと停止できない)。

for await..of を使っていればループの中で await するこにより(throw でループを抜けると)停止されますが、すべてのケースでの対応とはなりません。

対応としては Generator 側で個別に Promise を catch し自身を停止させることですが、これでも Generator の外側に何かしら reject の影響が出てしまいます。

エラーのハンドリングをどこでするかで都合が良い場合もありそうですが、少し悩ましい部分となりそうです。

:::details (クリックでサンプル表示)

```ts:pass_promise_via_async_generator.ts
export {}
const wait = (to: number) =>
  new Promise<void>((resolve) => setTimeout(() => resolve(), to))

const promiseArray: () => Promise<string>[] = () =>
  new Array(5).fill('').map(
    (_v, i) =>
      new Promise<string>((resolve) => {
        setTimeout(() => {
          resolve(`done-${i}`)
        }, 100 * (i + 1))
      })
  )

async function* asyncGenArray(
  p: Promise<string>[]
): AsyncGenerator<[Promise<string>], void, void> {
  try {
    for (let t of p) {
      await wait(100)
      yield [t]
    }
    console.log('async generator(array) done')
  } catch (r) {
    console.log(`async generator(array): ${r}`)
  } finally {
    console.log(`async generator(array): finally`)
  }
}

async function* asyncGenFunc(
  p: Promise<string>[]
): AsyncGenerator<() => Promise<string>, void, void> {
  try {
    for (let t of p) {
      await wait(100)
      yield () => t
    }
    console.log('async generator(func) done')
  } catch (r) {
    console.log(`async generator(func): ${r}`)
  } finally {
    console.log(`async generator(func): finally`)
  }
}

async function* asyncGenCatch(
  p: Promise<string>[]
): AsyncGenerator<() => Promise<string>, void, void> {
  try {
    let err: any
    for (let t of p) {
      const c = t.catch((r) => {
        console.log(`async generatro(catch) loop: ${r}`)
        err = r
        return Promise.reject(r)
      })
      if (err) {
        break
      }
      await wait(100)
      yield () => c
    }
    console.log('async generator(catch) done')
  } catch (r) {
    console.log(`async generator(catch): ${r}`)
  } finally {
    console.log(`async generator(catch): finally`)
  }
}

;(async () => {
  await (async () => {
    console.log('wrap appray')
    try {
      for await (let t of asyncGenArray(promiseArray())) {
        console.log(`${t[0]}`)
        console.log(`awaited ${await t[0]}`)
      }
    } catch (r) {
      console.log(`for await...of: ${r}`)
    }
  })()
  console.log('---')

  await (async () => {
    console.log('wrap func')
    try {
      for await (let t of asyncGenFunc(promiseArray())) {
        const p = t()
        console.log(`${p}`)
        console.log(`awaited ${await p}`)
      }
    } catch (r) {
      console.log(`for await...of: ${r}`)
    }
  })()
  console.log('---')

  await (async () => {
    console.log('reject')
    const a = promiseArray()
    a.splice(
      4,
      0,
      new Promise<string>((resolve, reject) => {
        setTimeout(() => reject('rejected'), 1000)
      })
    )
    try {
      for await (let t of asyncGenFunc(a)) {
        const p = t()
        console.log(`${p}`)
        console.log(`awaited ${await p}`)
      }
    } catch (r) {
      console.log(`for await...of: ${r}`)
    }
  })()
  console.log('---')

  await (async () => {
    console.log('reject catch')
    const a = promiseArray()
    a.splice(
      4,
      0,
      new Promise<string>((resolve, reject) => {
        setTimeout(() => reject('rejected'), 1000)
      })
    )
    try {
      for await (let t of asyncGenCatch(a)) {
        const p = t()
        console.log(`${p}`)
        await p
          .then((v) => {
            console.log(`then ${v}`)
          })
          .catch((r) => {
            console.log(`catch: ${r}`)
          })
      }
    } catch (r) {
      console.log(`for await...of: ${r}`)
    }
  })()
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/pass_promise_via_async_generator.ts
wrap appray
[object Promise]
awaited done-0
[object Promise]
awaited done-1
[object Promise]
awaited done-2
[object Promise]
awaited done-3
[object Promise]
awaited done-4
async generator(array) done
async generator(array): finally
---
wrap func
[object Promise]
awaited done-0
[object Promise]
awaited done-1
[object Promise]
awaited done-2
[object Promise]
awaited done-3
[object Promise]
awaited done-4
async generator(func) done
async generator(func): finally
---
reject
[object Promise]
awaited done-0
[object Promise]
awaited done-1
[object Promise]
awaited done-2
[object Promise]
awaited done-3
[object Promise]
async generator(func): finally
for await...of: rejected
---
reject catch
[object Promise]
then done-0
[object Promise]
then done-1
[object Promise]
then done-2
[object Promise]
then done-3
[object Promise]
async generatro(catch) loop: rejected
catch: rejected
async generator(catch) done
async generator(catch): finally
```

:::

## その他

### Promise の状態を同期的に取得する方法(API)は無いが、人間が目視で確認はできる

`console.log()` で Promise をオブジェクトとして出力すると状態が表示されました。

またデバッガーでも表示されるようです。

![デバッガーで Promise を表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/4e8863e8f55749cabbd5dc91791375fe/promise-memo-status-in-debugger.png?w=332\&h=270\&auto=compress%2Cformat)*▲ 図 9-1 状態も表示される*

ただし `toString()` などではできなかったです。 (これができれば少し前の `isArray` 的なことができそうかと思っていのたですが)

@[card](https://stackoverflow.com/questions/30564053/how-can-i-synchronously-determine-a-javascript-promises-state)

:::details (クリックでサンプル表示)

```ts:promise_is_pending.ts
;(async () => {
  let passResolve: (value: string) => void = (v) => {}
  const p1 = new Promise<string>((resolve) => {
    passResolve = resolve
  })
  console.log(p1)
  console.log(p1.toString())
  passResolve('done')
  await p1
  console.log(p1)

  let passReject: (value: string) => void = (v) => {}
  const p2 = new Promise<string>((resolve, reject) => {
    passReject = reject
  }).catch((r) => r)
  console.log(p2)
  console.log(p2.toString())
  passReject('rejected')
  await p2
  console.log(p2)
})()
```

```shell-session
$ node --loader ts-node/esm examples/memo/promise_is_pending.ts
Promise { <pending> }
[object Promise]
Promise { 'done' }
Promise { <pending> }
[object Promise]
Promise { 'rejected' }
```

:::

### TypeScript は Chain のおかしなところを指摘してくれる

Async Generator のところでも少し書きましたが `catch()` などのハンドラーが return した値によって Chain(が返す Promise のインスタンス)の型が変わります。

TypeScript では型のチェックが入るとエラーになるので、ミスに気が付くことができました。

![VSCode でエラーを表示しているスクリーンショット](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/91d98afc93914548a7437ef215e27e8d/promise-memo-type-error.png?w=600\&h=272\&auto=compress%2Cformat)*▲ 図 9-2 catch のハンドラーで(return していないので)void の可能性が加わり、string 型と合わない*
