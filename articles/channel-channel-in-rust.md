---
id: channel-channel-in-rust
title: Rust ã§ã chan chan
emoji: ð²
type: tech
topics:
  - rust
  - ä¸¦åå¦ç
  - channel
published: true
---

åæ¥ãRust ã®åå¼·ç¨ã«[ãã­ã¹ãè¡ãã¯ãªã¼ããããã¼ã«](https://github.com/hankei6km/xquo)ãä½ã£ãã®ã§ãããããã«ã¯ãä¸¦åå¦çãæå¹åããã¨è¡ã®é çªãä¿æãããªããã¨ããæ®å¿µãªç¹ãããã¾ãã

ããã§ãGo ã® channel ã§ä½¿ããã¦ããææ³ããåèã«ãã¦åºåã®é çªãä¿æãããããã«ãã¦ã¿ã¾ããã

## channel ã§ channel ãéä¿¡ãã(chan chan)

Go ã§ã¯é çªä¿æãªã©ã«ãchannel ã§ channel ãéä¿¡ããææ³ããç¥ããã¦ãã¾ãã

@[card](https://qiita.com/hogedigo/items/15af273176599307a2b2)

ä»åã¯ä¸è¨è¨äºã®ã[ä½¿ç¨ä¾ 4: é åºã®ä¿è¨¼](https://qiita.com/hogedigo/items/15af273176599307a2b2#%E4%BD%BF%E7%94%A8%E4%BE%8B-4-%E9%A0%86%E5%BA%8F%E3%81%AE%E4%BF%9D%E8%A8%BC)ããåèã«ãã¦ã¿ããã¨æãã¾ãã

## åºåã®é çªãä¿æããæ¹æ³

åè¿°ã®è¨äºã§ã¯ãã¬ã³ã¼ã 1 ä»¶ã«ã¤ã 1 åéåæå¦çãè¡ãããã¨ãåæã«ãªã£ã¦ããã®ã§ãä½æãããã¼ã«ã¨ã¯å°ãæ§é ãç°ãªã£ã¦ãã¾ãã

ããã§ãä»¥ä¸ã®ãããªã¯ã¼ã«ã¼ã¹ã¬ãããå©ç¨ããå½¢æã§åºåã®é çªãä¿æãããæ¹æ³ãèãã¦ã¿ã¾ãã

**å³ 2-1 ã¯ã¼ã«ã¼ã¹ã¬ããã®å¦çæ¦è¦**

```mermaid
flowchart LR
  generator

  generator --> worker1["worker1"]
  generator --> worker2["worker2"]


  printer["printer"]

  worker1 --> printer
  worker2 --> printer
	
  printer --> output[("output")]

  style generator fill:#EBECED
  style worker1 fill:#FAEBDD
  style worker2 fill:#FAEBDD
  style printer fill:#DDEDEA
```

### ã¹ã¬ããéã® channel è¨­å®

ã¾ããæåã«ã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ããããåã¹ã¬ããã¸ã¡ãã»ã¼ã¸ãéä¿¡ããããã® channel ãè¨­å®ãã¾ã(ã¯ã¼ã«ã¼ã¹ã¬ããããããªã³ã¿ã¼ã¹ã¬ããã¸ã¯è¨­å®ãã¾ãã)ã

```mermaid
flowchart LR
  generator["generator"]

  generator ==> worker1["worker1"]
  generator ==> worker2["worker2"]

  generator ==> printer["printer"]

  style generator fill:#EBECED
  style worker1 fill:#FAEBDD
  style worker2 fill:#FAEBDD
  style printer fill:#DDEDEA
```

### ãã¼ã¿ã¨ channel ã®çæ

å¦çãéå§ãããããã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ããã¯ãã¼ã¿(`data`)ã¨å¯¾å¿ãã channel (`tx` + `rx`)ãé æ¬¡çæãã¾ãã

```mermaid
flowchart LR
  subgraph generator
    direction LR
  	subgraph gen1
      direction RL
      data1
      rx1((rx1)) -..-  tx1((tx1))
    end
  	subgraph gen2
      direction RL
      data2
      rx2((rx2)) -..-  tx2((tx2))
    end
  	subgraph gen3
      direction RL
      data3
      rx3((rx3)) -..-  tx3((tx3))
    end
  end

  style generator fill:#EBECED
```

### ã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ããããã®éä¿¡

`data` + `tx` ã®ã»ãããã¯ã¼ã«ã¼ã¹ã¬ããã¸å²ãæ¯ãã`rx` ã¯ããªã³ã¿ã¼ã¹ã¬ããã¸é æ¬¡éä¿¡ãã¾ãã

```mermaid
flowchart LR
  subgraph generator
  end
  subgraph worker1
  end
  subgraph worker2
  end
  subgraph printer
  end
  generator --- tx3(data3 + tx3) ---- tx1(data1 + tx1)--> worker1
  generator ---- tx2(data2 + tx2) ---> worker2
  generator --- rx3((rx3)) --- rx2((rx2)) --- rx1((rx1))--> printer

  style generator fill:#EBECED
  style worker1 fill:#FAEBDD
  style worker2 fill:#FAEBDD
  style printer fill:#DDEDEA
```

### ã¯ã¼ã«ã¼ã¹ã¬ããã§å¦çãããªã³ã¿ã¼ã¹ã¬ããã¸ã®éä¿¡

ã¯ã¼ã«ã¼ã¹ã¬ããã¯ `data` ãå¦çããå¾ã`tx` ãä½¿ã£ã¦çµæ(`res` )ãããªã³ã¿ã¼ã¹ã¬ããã§å¾æ©ãã¦ãã `rx `ã¸éä¿¡ãã¾ãã

```mermaid
flowchart LR
  subgraph worker1
    tx1((tx1))
    tx3((tx3))
  end
  subgraph worker2
    tx2((tx2))
  end
  subgraph printer
    rx1((rx1))
    rx2((rx2))
    rx3((rx3))
  end
  tx2 -..- res2 -..-> rx2
  tx1 -..- res1 -..-> rx1

  style worker1 fill:#FAEBDD
  style worker2 fill:#FAEBDD
  style printer fill:#DDEDEA
```

ãã®ã¨ãããªã³ã¿ã¼ã¹ã¬ããã§ã¯ãã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ãããã `rx` ãåãã¨ã£ãé çªã§ã`res` ãåä¿¡ãã¾ãã

ããã `data3` ã®å¦çãåã«å®äºãã¦ãã¦ãã`rx2` ã®åä¿¡ãçµãã£ã¦ããªããã°ã«ã¼ãããã­ãã¯ãããã®ã§ã`rx3` ã®åä¿¡ãè¿½ããããã¨ã¯ããã¾ããã

ä»¥ä¸ãé çªãä¿æããæ¹æ³ã«ãªãã¾ãã

## å®è£ãã¦ã¿ã

å®éã«åä½ããã³ãã³ããä½æãã¾ããã

@[card](https://github.com/hankei6km/test-rust-chan-chan)

1.  ã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ããã¯ `0..20` ã®å¤ãçæ
2.  ã¯ã¼ã«ã¼ã¹ã¬ããã§ã¯å¤ã 10 åãã¦ãã `String` ã¸å¤æããã®ã¨ã 0.5 ç§Â± 0.001 ç§ã§ã¹ãªã¼ããã
3.  ããªã³ã¿ã¼ã¹ã¬ããã¯å¤æçµæãåä¿¡ãè¡¨ç¤º

å¼æ°ãªãã§å®è¡ããã¨åè¿°ã® **å³ 2-1** ã®ãããªéå¸¸ã®ä¸¦åå¦çã§å®è¡ããã¾ãã

**å³ 3-1 éå¸¸ã®å¦çã§ã¯åºåã®é çªã¯ä¿æãããªã**

```shell-session
$ ./target/x86_64-unknown-linux-musl/release/test-rust-chan-chan
0
10
20
30
40
50
70
80
60
90
100
130
110
120
140
150
170
160
180
190
```

`-e` ãæå®ããã¨ chan chan ãå©ç¨ããä¸¦åå¦çã§å®è¡ããã¾ãã

**å³ 3-2 chan chan ã§ã¯åºåã®é çªãä¿æããã**

```shell-session
$ ./target/x86_64-unknown-linux-musl/release/test-rust-chan-chan -e
0
10
20
30
40
50
60
70
80
90
100
110
120
130
140
150
160
170
180
190
```

:::details (ã½ã¼ã¹ã³ã¼ã)

```rust:src/normal.rs
use std::{sync::mpsc, thread};

use crossbeam_channel::bounded;

use crate::Wait;

pub fn proc() {
    let (out_tx, out_rx) = bounded::<i32>(0);
    let (in_tx, in_rx) = mpsc::sync_channel::<String>(0);

    // generator thread
    thread::spawn(move || {
        for i in 0..20 {
            out_tx.send(i).unwrap();
        }
    });

    // worker thread
    for _i in 0..5 {
        let out_rx = out_rx.clone();
        let in_tx = in_tx.clone();
        thread::spawn(move || {
            let mut w = Wait {
                rng: rand::thread_rng(),
            };
            for i in out_rx {
                w.wait(i);

                // compute in worker
                let out = (i * 10).to_string();

                in_tx.send(out).unwrap();
            }
        });
    }
    drop(in_tx);

    // prrinter loop in main thread
    for i in in_rx {
        println!("{}", i);
    }
}
```

```rust:src/chanchan.rs
use std::{sync::mpsc, thread};

use crossbeam_channel::{bounded, Receiver, Sender};

use crate::Wait;

struct ChanChanTx<T, U> {
    payload: T,
    tx: Sender<U>,
}
struct ChanChanRx<U> {
    rx: Receiver<U>,
}

pub fn proc() {
    let (out_tx, out_rx) = bounded::<ChanChanTx<i32, String>>(0);
    // ä»¥ä¸ã® channel ã®å®¹éãå¢ããã¨ `normal.proc()` ã«è¿ãå®è¡æéã¨ãªã(ãªã¼ãã¼ãããåã¯éããªã)
    let (in_tx, in_rx) = mpsc::sync_channel::<ChanChanRx<String>>(0);

    // generator thread
    thread::spawn(move || {
        for i in 0..20 {
            let (tx, rx) = bounded::<String>(0);
            out_tx.send(ChanChanTx { payload: i, tx }).unwrap();
            in_tx.send(ChanChanRx { rx }).unwrap();
        }
    });

    // worker thread
    for _i in 0..5 {
        let out_rx = out_rx.clone();
        thread::spawn(move || {
            let mut w = Wait {
                rng: rand::thread_rng(),
            };
            for i in out_rx {
                w.wait(i.payload);

                // compute in worker
                let out = (i.payload * 10).to_string();

                i.tx.send(out).unwrap();
            }
        });
    }

    // prrinter loop in main thread
    for i in in_rx {
        println!("{}", i.rx.recv().unwrap());
    }
}
```

:::

ã³ã¼ãã®åå®¹çã«ã¯åç¯ã®ã¨ããã§ãããRsut ã§è¨è¿°ããã¨ãã®è£è¶³ãªã©ã

channel ã§éä¿¡ããã channel ã¯ [crossbeam\_channel](https://crates.io/crates/crossbeam-channel) ãå©ç¨ãã¦ãã¾ãã

@[card](https://crates.io/crates/crossbeam-channel)

ããã¯ mpsc ã® [`Reciver`](https://doc.rust-lang.org/std/sync/mpsc/struct.Receiver.html) ã¯ Trait Implementations ã« `!Sync` ãå«ã¾ããã®ã§ããã®ã¾ã¾ã ã¨ `send` ã§ããªããã¨ã«ããã¾ãããã£ã¦ã[crossbeam\_channel](https://crates.io/crates/crossbeam-channel) ãå©ç¨ãã¦ãã¾ã[^use-mpsc]ã

[^use-mpsc]: mpsc ãå©ç¨ããå ´åã¯ `Arc` ã¨ `Mutex` ãä½¿ããã¨ã«ãªãã¾ããåè:ã[Rustå¥é](https://zenn.dev/mebiusbox/books/22d4c1ed9b0003) ã® Chapter 18 ä¸¦åå¦çãã[Rustã«ãããã¹ã¬ããéã§ã®ãã¼ã¿å±æã¨std::thread::scope](https://zenn.dev/toru3/articles/ce9232f53c47c8)ã

**å³ 3-3 !Sync ãå«ã¾ãã**

![Trait Implementations ã®ä¸è¦§ã§ !Sync ã«èµ¤æ ãä»ããã¹ã¯ãªã¼ã³ã·ã§ãã](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/54c7ceae073946b9878cdf7dbdc36e42/channel-channel-in-rust-receiver.png?w=225\&h=221\&auto=compress%2Cformat)

ã¾ããã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ãããããã¡ã³ã¢ã¦ãçãªå¦çãè¡ãããã«å©ç¨ãã¦ãã¾ãããããã¯ chan chan ã¨ã¯é¢ä¿ãªãéå¸¸å¦çã§ãåãã§ãã

@[card](https://qiita.com/termoshtt/items/b4561180894104c1c4c2)

**å³ 3-4 è©²å½é¨åã®æç²**

```rust
use crossbeam_channel::{bounded, Receiver, Sender};

// snip ...

struct ChanChanTx<T, U> {
    payload: T,
    tx: Sender<U>,
}
struct ChanChanRx<U> {
    rx: Receiver<U>,
}

pub fn proc() {
    let (out_tx, out_rx) = bounded::<ChanChanTx<i32, String>>(0);
    let (in_tx, in_rx) = mpsc::sync_channel::<ChanChanRx<String>>(0);

    // generator thread
    thread::spawn(move || {
        for i in 0..20 {
            let (tx, rx) = bounded::<String>(0);
            out_tx.send(ChanChanTx { payload: i, tx }).unwrap();
            in_tx.send(ChanChanRx { rx }).unwrap();
        }
    });

    // snip ...
```

ãã¨ã¯ channel ã®ç¨®é¡ã§ãããéåº¦çãªæ¡ä»¶ç¢ºèªã®ããã«å®¹éãæå®ã§ãã channel ãä½¿ã£ã¦ãã¾ã(ã¨ããããã¯å¨ã¦ `0 `ãæå®ãã¦ãã¾ã)ã

*   mpsc - sync\_channel(0)
*   crossbeam\_channel - bounded(0)

## å¦çæéã®å¢å 

chan chan ãä½¿ã£ãå ´åãchannel çæã¨ send åæ°å¢å ãã®ãªã¼ãã¼ãããã¯äºæ³ããã¾ã¾ãããããã§ã¯å¦çãã­ã¼ã®æ§é çãªè¦å ã«ããå¦çæéã®å¢å ã«ã¤ãã¦èãã¦ã¿ã¾ãã

ã¾ããåè¿°ã®å¦çã®å ´åãè¨ç®ä¸ã®æéã¯ä»¥ä¸ã®ããã«ãªãã¨äºæ³ããã¾ã(ä»åã¯ã¯ã¼ã«ã¼ã¹ã¬ããã 5 ã¤ç¨¼åããè¨­å®ã«ãã¦ããã¾ã)ã

    ã¯ã¼ã«ã¼ã¹ã¬ããåã®ã¹ãªã¼ãæé * çæããããã¼ã¿æ° / ã¯ã¼ã«ã¼ã¹ã¬ããã®æ°
    = ã»ã¼0.5ç§ * 20 / 5 = ã»ã¼ 2.0 ç§

å®éã¯ããã«åç¨®ãªã¼ãã¼ããããªã©ãå ç®ããã¾ãããä»åã¯å¾ã¡æéãã¹ãªã¼ãã«ãã£ã¦çä¼¼çã«ä½ãåºãã¦ããã ããªã®ã§ãã®æ°å¤ä»è¿ã«è½ã¡çãã¨æããã¾ãã

### å channel ã®å®¹éã 0 ã®å ´å

ã¾ãã¯å channel ã®å®¹éã 0 (send ãåæçã«åä½ãã)ç¶æ³ã§è©¦ãã¦ã¿ã¾ãã

**å³ 4-1 å®è¡çµæ**

```shell-session
# éå¸¸ã®å¦ç
$ time ./target/x86_64-unknown-linux-musl/release/test-rust-chan-chan > /dev/null

real    0m2.001s
user    0m0.003s
sys     0m0.000s

# chan chan ã«ããå¦ç
$ time ./target/x86_64-unknown-linux-musl/release/test-rust-chan-chan -e > /dev/null

real    0m5.001s
user    0m0.005s
sys     0m0.000s
```

|           | å®è¡æé(ç§) |
| --------- | ------- |
| éå¸¸å¦ç      | 2.001   |
| chan chan | 5.001   |

éå¸¸ã®å¦çã¯ã»ã¼è¨ç®ããå¤ã«ãªã£ã¦ãã¾ãããchan chan ã®æ¹ã§ã¯éããªã£ã¦ãã¾ãã

ããã¯ã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ããããããªã³ã¿ã¼ã¹ã¬ããã¸éä¿¡ãã channel ã§å¾ã¡ãçºçããã¯ã¼ã«ã¼ã¹ã¬ãããä¸éã¾ã§ä½¿ããªããªãããã§ãã ( ããªã³ã¿ã¼ã¹ã¬ããã `rx` ãåãåã£ã¦ãããªãã¨ã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ããã¯æ°ããå¤ãçæã§ããªã)

### ã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ãã / ããªã³ã¿ã¼ã¹ã¬ããé channel ã®å®¹éãå¢ãã

ä»åº¦ã¯å¾ã¡ãçºçãã¦ãã channel ã®å®¹éã +1 ããªããè©¦ãã¦ã¿ã¾ãã

**ãªã¹ã 4-1 å¤æ´ããç®æ**

```rust:src/chanchan.rs
// post é¢æ°ã®ä»¥ä¸ã®å¤ãå¤æ´ã build ãç´ã
let (in_tx, in_rx) = mpsc::sync_channel::<ChanChanRx<String>>(1);
```

**å³ 4-2 å¤ãå¤æ´ããã build ãã¦å®è¡**

```shell-session
$ cargo build --target=x86_64-unknown-linux-musl --release
   Compiling test-rust-chan-chan v0.1.0 (/workspaces/test-rust-chan-chan)
    Finished release [optimized] target(s) in 6.54s

$ time ./target/x86_64-unknown-linux-musl/release/test-rust-chan-chan -e > /dev/null

real    0m3.502s
user    0m0.000s
sys     0m0.007s
```

| 1     | 2     | 3     | 4     | 5     | 6     |
| ----- | ----- | ----- | ----- | ----- | ----- |
| 3.499 | 2.500 | 2.002 | 2.003 | 2.002 | 2.002 |

å®¹éãå¢ãããã¨ã§è¨ç®ä¸ã®å¤ã«å°éãã¾ããããªãå®¹é 3 ã§å°éãã¦ãã¾ãããããã¯ã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ããã®ã«ã¼ãã¨ããªã³ã¿ã¼ã¹ã¬ããã®åä¿¡ãéå»¶ãªãå®è¡ãããããã§ã[^slow-generator]ãå®éã«ã¯åç¨®ãªã¼ãã¼ããããããã®ã§ãå®ç°å¢ã§ç¢ºèªããªããèª¿æ´ãããã¨ã«ãªããã¨æãã¾ãã

[^slow-generator]: è©¦ãã«ã¸ã§ãã¬ã¼ã¿ã¼ã¹ã¬ããã®ã«ã¼ãã sleep ãªã©ã§éãããã¨å®¹é 4 ã«ããªãã¨å°éããªããªãã¾ãã

### å¦çæéã®é·ãå¤ãæ··ãã¦ã¿ã

ä¸è¨ã®å ´åãåãã¼ã¿ã®å¦çæéã¯ã»ã¼åããªã®ã§çæ³çãªä¸¦åå¦çã«ãªãã¾ãã

ããã§ããã¼ã¿ã®å¤ãå¶æ°ã®ã¨ãã¯ 3 ç§å¾ã¤ããã«ãã¿ã¾ãã

**ãªã¹ã 4-2 å¤æ´ããç®æ**

```rust:src/lib.rs
impl Wait {
    pub fn wait(&mut self, _i: i32) {
        // ä»¥ä¸ã®ããã«å¤æ´ã build ãç´ã
        let range = if _i % 2 == 0 { 3000..3001 } else { 499..501 };
        // let range = 499..501;
        thread::sleep(Duration::from_millis(self.rng.gen_range(range)));
    }
}
```

**å³ 4-3 å®è¡çµæ**

```shell-session
# éå¸¸ã®å¦ç
$ time ./target/x86_64-unknown-linux-musl/release/test-rust-chan-chan > /dev/null

real    0m7.999s
user    0m0.004s
sys     0m0.001s

# chan chan ã«ããå¦ç(channel ã®å®¹éã¯ 5 ã«ãã¦ããã¾ã)
$ time ./target/x86_64-unknown-linux-musl/release/test-rust-chan-chan -e > /dev/null

real    0m12.003s
user    0m0.002s
sys     0m0.003s
```

|                 | å®è¡æé   |
| --------------- | ------ |
| éå¸¸ã®å¦ç           | 7.99   |
| chan chan ã«ããå¦ç | 12.003 |

å®è¡ãã¦ã¿ãã¨ chan chan ã®æ¹ãéããªãã¾ãã

ããã¯ chan chan ã®å¦çã§ã¯éããã¼ã¿ãããå ´åãå¾ãã«éããã¼ã¿ããã£ã¦ãè¿½ãè¶ããªããã¨ãåå ã§ãã

éå¸¸ã®å¦çã§ã¯ä»¥ä¸ã®ããã«éããã¼ã¿ãåã«è¡¨ç¤ºããã¦ãã¾ã(å¹çããä¸¦åå¦çãå®è¡ããã¦ãã)ã

**å³ 4-4 ãã¼ã¿ã®è¡¨ç¤º**

```shell-session
$ ./target/x86_64-unknown-linux-musl/release/test-rust-chan-chan
30
10
50
70
0
40
20
110
90
60
130
150
80
170
100
120
190
140
160
```

## channel çæã¨ send åæ°å¢å ã«ãããªã¼ãã¼ããã

åé ­ã®ã³ãã³ãã chan chan å¯¾å¿ãã¦ã¿ãã®ã§ãå¤æ´åã¨å¾ã®å®è¡æéãæ¯ã¹ã¦ã¿ã¾ãã

å®è¡ç°å¢ããã¼ã¿ãªã©ã«ã¤ãã¦ã¯ã[Rust ã§ jemalloc ãä½¿ã£ããä¸¦åå¦çãéããªã£ã](https://zenn.dev/hankei6km/articles/using-jemalloc-in-rust-speeds-up-parallelism#jemalloc-%E7%84%A1%E3%81%97%E3%81%A8%E6%9C%89%E3%82%8A%E3%81%A7%E3%81%AE%E6%AF%94%E8%BC%83)ãã§èª¿ã¹ãã¨ãã¨åãã§ãã

ãã ããä»¥ä¸ã®ããã«æ¡ä»¶ãããã£ã¦ããªãé¨åãããã®ã§ç®å®ç¨åº¦ã«è¦ã¦ãã ããã

*   channel ã¯å®¹éãæå®ãããã®ã¸å¤æ´
*   ç°ãªãæ¥æã« Codespaces ã§å®è¡ãã¦ãã
*   ã·ã¹ãã ã¢ã­ã±ã¼ã¿ã¼(jemalloc ç¡ã)ã¯ãã®ä»ã®ãªã¼ãã¼ããããå¤§ãã(å¦çãã¼ã¿ã«å½±é¿ããããã)

çµæã¯ä»¥ä¸ã®ã¨ããã§ããmusl ã® jemalloc ç¡ãã§ã¯ chan chan ã®æ¹ãéãã¨ããå¤ãããã¾ãã[^data]ãåºæ¬çã«ã¯éããªãã¾ãããã¯ã channel çæã¨ send ãå¢ããåã®ãªã¼ãã¼ãããã¯ããããã§ãã

[^data]: ãã¹ããã¼ã¿ãå·®ãæ¿ããã¨ chan chan ã®æ¹ãéããªãã¨ããããã®ã§ãã¹ã¬ãããã­ãã¯ãããã¿ã¤ãã³ã°ãªã©ã®å½±é¿ããªã¨äºæ³ãã¦ãã¾ãã

**è¡¨ 5-1 chan chan å¯¾å¿åã®çµæ**

|                                        | 1      | 2      | 3      | 4      | 5      | 6      |
| -------------------------------------- | ------ | ------ | ------ | ------ | ------ | ------ |
| x86\_64-unknown-linux-musl jemalloc ç¡ã | 23.972 | 42.668 | 39.447 | 43.407 | 56.113 | 58.653 |
| x86\_64-unknown-linux-musl jemalloc æã | 11.903 | 6.724  | 6.112  | 5.889  | 5.907  | 5.943  |
| x86\_64-unknown-linux-gnu  jemalloc ç¡ã | 11.075 | 24.725 | 26.267 | 30.876 | 31.399 | 31.864 |
| x86\_64-unknown-linux-gnu  jemalloc æã | 8.566  | 4.919  | 4.829  | 4.77   | 4.732  | 4.71   |

**è¡¨ 5-2 chan chan å¯¾å¿å¾ã®çµæ**

|                                        | 1      | 2      | 3      | 4      | 5      | 6      |
| -------------------------------------- | ------ | ------ | ------ | ------ | ------ | ------ |
| x86\_64-unknown-linux-musl jemalloc ç¡ã | 21.229 | 33.035 | 43.952 | 54.154 | 55.481 | 57.397 |
| x86\_64-unknown-linux-musl jemalloc æã | 13.188 | 8.535  | 7.700  | 8.908  | 8.898  | 8.703  |
| x86\_64-unknown-linux-gnu jemalloc ç¡ã  | 14.696 | 27.383 | 30.797 | 45.350 | 44.607 | 42.380 |
| x86\_64-unknown-linux-gnu jemalloc æã  | 10.090 | 7.376  | 6.491  | 7.084  | 7.166  | 7.094  |

å¯¾ç­ã¨ãã¦ã¯ãã¡ãã»ã¼ã¸ãéä¿¡ããåæ°ãæ¸ãã = channel çææ°ãæ¸ãããã¨ãããã¨ã«ãªããã¨æãã¾ãããã¨ãã°ãä»åã®ãã¼ã«ã¯ 1 åã®ã¡ãã»ã¼ã¸ã«å«ããè¡æ°ãå¤æ´ã§ããã®ã§ãå¤æ´ããã¨å®è¡æéãç­ç¸®ããã¾ãã

**å³ 5-1 ã¯ã¼ã«ã¼ã¹ã¬ããæ°ã 2 ã§ 1000 è¡ã¾ã¨ãã¦éä¿¡ããè¨­å®ã§å®è¡**

```shell-session
$ ./target/x86_64-unknown-linux-musl/release/xquo -h
xquo 0.2.0
Quote null splited lines for Bash command line

USAGE:
    xquo [OPTIONS] < /path/to/file

OPTIONS:
    -b, --bulk-lines <BULK_LINES>
            The number of lines bundled in a single bulk [default: 100]

    << snip >>

$ time ./target/x86_64-unknown-linux-musl/release/xquo -w 2 -b 1000 < tmp/tmp_large.txt > /dev/null

real    0m6.738s
user    0m13.899s
sys     0m0.293s
```

ãã¨ã¯ãéä¿¡ããã channel ãæ¯åçæããã®ã§ã¯ãªããåå©ç¨ã§ããã°å¹çãããªããããªæ°ããã¦ãã¾ããããããã¾ã§è©¦ãåæ°ã¯ãªãã£ãã§ãã

## ãããã«

ãchannel ã§ channel ãéä¿¡ãããã¨ã§åºåã®é çªãä¿æãããã Rust ã§è©¦ãã¦ã¿ã¾ããã

éå¸¸ã®å¦çã«æ¯ã¹ãã¨éããªãå¾åã«ããã¾ãããåºåã®é çªãä¿æãããã¡ãªããã¯å¾ããã¾ããã

[åã«å¼ç¨ããè¨äº](https://qiita.com/hogedigo/items/15af273176599307a2b2)ã«ãããããã« channel ã§ channel ãéä¿¡ãããã¿ã¼ã³ã¯ããã¤ãããã®ã§ãä»ã®æ¹æ³ãæ´»ç¨ãã¦ãããã°èã¨ãã¦ãã¾ãã
