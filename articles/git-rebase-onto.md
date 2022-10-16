---
id: git-rebase-onto
title: Git のブランチを別のブランチへ付け替える (--onto)
emoji: 🦥
type: tech
topics:
  - git
published: true
---

以前に [Gist で書いていたメモ](https://gist.github.com/hankei6km/4301ebc7e320b987de1f7a5b0acb994a#gistcomment-4334778)にコメントを頂いて知識が更新されたので、せっかくなので Zenn にも転載しようかと。 ([Mermaid](https://mermaid-js.github.io/mermaid/#/) の [Gitgraph Diagrams](https://mermaid-js.github.io/mermaid/#/gitgraph) を使ってみたかったというのもあります)

そのような感じで、たまに `--onto` を使おうとすると忘れているのでメモ。

<!-- textlint-disable -->

## 基本(普通に rebase)

これを

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  branch branch1
  checkout branch1
  commit id: "D"
  commit id: "E"
  commit id: "F"
  checkout develop
  commit id: "a"
  commit id: "b"
```

こうするには

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  commit id: "a"
  commit id: "b"
  branch branch1
  checkout branch1
  commit id: "D-1"
  commit id: "E-1"
  commit id: "F-1"
```

以下のように `rebase` を使う。

```shell-session
$ git rebase develop branch1
```

現在のブランチが `branch1` だった場合はこうでも大丈夫。

```shell-session
$ git rebase develop
```

なお、実行するとコミット「`D` `E` `F`」のハッシュ値などは変更されます(この記事では変更されたコミットに枝番を付けます)。

## 例1

これを

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  branch branch1
  checkout branch1
  commit id: "D"
  commit id: "E"
  commit id: "F"
  branch branch2
  checkout branch2
  commit id: "G"
  commit id: "H"
  commit id: "I"
  checkout develop
  commit id: "a"
  commit id: "b"
```

こうするには

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  branch branch1
  checkout branch1
  commit id: "D"
  commit id: "E"
  commit id: "F"
  checkout develop
  commit id: "a"
  commit id: "b"
  branch branch2
  checkout branch2
  commit id: "G-1"
  commit id: "H-1"
  commit id: "I-1"
```

以下のように `--onto` を使う。

```shell-session
$ git rebase --onto develop branch1 branch2
```

意味としては、

*   `develop` へ付け替える
*   付け替えるのは `branch2`
*   `branch2` の上流(`branch2` の分岐元)は `branch1`

となります。

## 例2

これの `branch1` を `rebase` したら

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  branch branch1
  checkout branch1
  commit id: "D"
  commit id: "E"
  commit id: "F"
  branch branch2
  checkout branch2
  commit id: "G"
  commit id: "H"
  commit id: "I"
  checkout develop
  commit id: "a"
  commit id: "b"
```

こうなってしまった

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  branch branch2 order:2
  checkout branch2
  commit id: "D"
  commit id: "E"
  commit id: "F"
  commit id: "G"
  commit id: "H"
  commit id: "I"
  checkout develop
  commit id: "a"
  commit id: "b"
  branch branch1
  checkout branch1
  commit id: "D-1"
  commit id: "E-1"
  commit id: "F-1"
```

### そのまま `branch2` も `rebase` する

`branch2` を新しい `branch1` へ `rebase` すると

```shell-session
$ git rebase branch1 branch2
warning: skipped previously applied commit <HASH値>
warning: skipped previously applied commit <HASH値>
warning: skipped previously applied commit <HASH値>
hint: use --reapply-cherry-picks to include skipped commits
hint: Disable this message with "git config advice.skippedCherryPicks false"
```

こうなる。

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  checkout develop
  commit id: "a"
  commit id: "b"
  branch branch1
  checkout branch1
  commit id: "D-1"
  commit id: "E-1"
  commit id: "F-1"
  branch branch2
  checkout branch2
  commit id: "G-1"
  commit id: "H-1"
  commit id: "I-1"
```

`rebase` のデフォルトの挙動では対象になるコミットの内容を読みとって比較します(詳細は[こちら](https://git-scm.com/docs/git-rebase#Documentation/git-rebase.txt---reapply-cherry-picks))。よって、`branch2` に含まれている「`D` `E` `F`」は「`D-1` `E-1` `F-1`」と同一として扱われるのでスキップされます。 (Gitst のコメントで教えてもらいました、ありがとうございます)

また、上記のスキップは `branch1` にコミットを追加していても機能します。

`branch2` の存在を忘れていて `branch1` を更新してしまった(コミット `c` ができている)

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  branch branch2 order:2
  checkout branch2
  commit id: "D"
  commit id: "E"
  commit id: "F"
  commit id: "G"
  commit id: "H"
  commit id: "I"
  checkout develop
  commit id: "a"
  commit id: "b"
  branch branch1
  checkout branch1
  commit id: "D-1"
  commit id: "E-1"
  commit id: "F-1"
  commit id: "c"
```

`branch2` の存在を思い出して `rebase` すると

```shell-session
$ git rebase branch1 branch2
warning: skipped previously applied commit <HASH値>
warning: skipped previously applied commit <HASH値>
warning: skipped previously applied commit <HASH値>
hint: use --reapply-cherry-picks to include skipped commits
hint: Disable this message with "git config advice.skippedCherryPicks false"
Successfully rebased and updated refs/heads/branch2.
```

コンフリクトがなければこうなる。

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  checkout develop
  commit id: "a"
  commit id: "b"
  branch branch1
  checkout branch1
  commit id: "D-1"
  commit id: "E-1"
  commit id: "F-1"
  commit id: "c"
  branch branch2
  checkout branch2
  commit id: "G-1"
  commit id: "H-1"
  commit id: "I-1"
```

### `-onto` が必要な場合

`F-1` を `--amend` して `F-2` がで出来てしまった

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  branch branch2 order:2
  checkout branch2
  commit id: "D"
  commit id: "E"
  commit id: "F"
  commit id: "G"
  commit id: "H"
  commit id: "I"
  checkout develop
  commit id: "a"
  commit id: "b"
  branch branch1
  checkout branch1
  commit id: "D-1"
  commit id: "E-1"
  commit id: "F-2"
```

この後に `rebase` すると `F` がスキップされないのでコンフリクトしやすい。

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px', 'git2': '#ff0000', 'gitBranchLabel2': '#ffffff' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  checkout develop
  commit id: "a"
  commit id: "b"
  branch branch1
  checkout branch1
  commit id: "D-1"
  commit id: "E-1"
  commit id: "F-2"
  branch branch2 order:2
  checkout branch2
  commit id: "F" type: REVERSE
  commit id: "G"
  commit id: "H"
  commit id: "I"
```

以下のように、`branch2` の分岐元としてコミット `F`(元の `branch1`)のハッシュ値(今回の例では `53c6b6` とする)を指定

```shell-session
$ git rebase --onto branch1 53c6b6 branch2
```

コンフリクトがなければこうなる。

```mermaid
%%{init: { 
 'gitGraph': { 'rotateCommitLabel': false, 'mainBranchName': 'develop' },
 'themeVariables': { 'commitLabelFontSize': '18px' }
} }%%
gitGraph
  commit id: "A"
  commit id: "B"
  commit id: "C"
  checkout develop
  commit id: "a"
  commit id: "b"
  branch branch1
  checkout branch1
  commit id: "D-1"
  commit id: "E-1"
  commit id: "F-2"
  branch branch2
  checkout branch2
  commit id: "G-1"
  commit id: "H-1"
  commit id: "I-1"
```

<!-- textlint-enable -->
