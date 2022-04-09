---
id: actionlint-with-vim-ale
title: actionlint を Vim + ALE で利用する Custom Linter を作ってみた
emoji: ✅
type: idea
topics:
  - githubactions
  - vim
  - lint
published: true
---

「[GitHub Actions の手動実行で入力された値を安全に使いたい](https://zenn.dev/hankei6km/articles/use-github-acions-workflow-dispatch-inputs-safely)」を書いているとき、[actionlint] という GitHub Actions(ワークフローファイル)用のリンターを知りました。

これまでは GitHub へ `push` したあとに「エラーになってしまった」と頭をかかえていたので、大変に魅力的なツールです。そこで [Vim] + [ALE] で利用するための Custom Linter を作ってみました。

@[card](https://github.com/hankei6km/ale-linter-actionlint.vim)

## 何ができるようになるのか

[Vim] で GitHub Actions のワークフローファイルを編集しているとき、エラー箇所が表示されます。

▼ **図 1-1 サンプル**
![Vim で表示されているエラーを編集している動画](/images/actionlint-with-vim-ale/actionlint-with-vim-ale-examples.gif)

## インストール

以下のような感じでインストールできます。

### actionlint

[actionlint] の実行ファイルが必要なので何らかの方法でインストールしておきます。

@[card](https://github.com/rhysd/actionlint/blob/main/docs/install.md)

Arch Linux だと AUR にパッケージが登録されていました。

@[card](https://aur.archlinux.org/packages?O=0\&K=actionlint)

私は VSCode の Remote Container 用のコンテナ内で使いたかったので Dockerfile で以下のようにしています。

```dockerfile:Dockerfile
RUN set -ex; \
    \
    snip..
    # Setup actionlint for user.
    curl -s https://api.github.com/repos/rhysd/actionlint/releases/latest | jq .assets[].browser_download_url | grep linux_amd64 | xargs -I '{}' curl -sL '{}' | tar -zxf - -C "/home/${USERNAME}/.local/bin" actionlint; \
```

### ale-linter-actionlint.vim

[Vundle] などで [`hankei6km/ale-linter-actionlint.vim`] をインストールします。

```vim
Plugin 'hankei6km/ale-linter-actionlint.vim'
```

## どのように表示しているのか

[ALE] の[対応ツール](https://github.com/dense-analysis/ale/blob/master/supported-tools.md)に [actionlint] が含まれていなかったので Custom Linter を作りました。

Custom Linter は以下を参考に作成できます。

@[card](https://github.com/dense-analysis/ale/issues/1405)

また、標準で搭載されているリンターのソースも参考になります。

@[card](https://github.com/dense-analysis/ale/tree/master/ale_linters)

少し悩んだ点としては、GitHub Actions のワークフローファイルに処理を限定する箇所です。

これは、 [`circleci.vim` ] を参考に「ファイルの PATH に `.github/workflows` が含まれているか」確認することで対応しています。ただし、PATH の区切り文字で手を抜いているのでおそらく Windows では動作しないです[^windows]。

[^windows]: Windows 上では開発用の環境をあまり整えていないのでくじけてしまいました。

:::details (クリックでスクリプト表示)

```vim:ale_linters/yaml/actionlint.vim
function! ale_linters#yaml#actionlint#Handle(buffer, lines) abort
    let l:output = []

    for l:err in ale#util#FuzzyJSONDecode(a:lines, [])
        call add(l:output, {
        \   'text': l:err['message'],
        \   'type': 'E',
        \   'code': l:err['kind'],
        \   'lnum': l:err['line'],
        \   'col' : l:err['column']
        \})
    endfor

    return l:output"
endfunction

call ale#linter#Define('yaml', {
\   'name': 'actionlint',
\   'executable': {b -> expand('#' . b . ':p:h') =~? '\.github/workflows$' ? 'actionlint' : ''},
\   'command': 'actionlint -format "{{json .}}" - < %s',
\   'callback': 'ale_linters#yaml#actionlint#Handle',
\   'output_stream': 'stdout',
\   'lint_file': 1,
\})
```

:::

## 付録

[Vim] + [ALE] の他にどのような環境でサポートされているかざっと調べてみました。

*   [node-actionlint] - npm script などで [actionlint] を実行しやすくした npm パッケージ(WebAssembly を利用しているのが興味深い)
*   [trunk] - VScode で各種リンターなどを実行する拡張機能？(対応リンターに [actionlint] が含まれている)
*   [coc.nvim + coc-diagnostic + actionlintで GitHub Actions workflowを快適にlintできるようにしてみた | DevelopersIO](https://dev.classmethod.jp/articles/workflow-lint-by-actionlint/)

## おわりに

[actionlint] を [Vim] + [ALE] で利用できるようにしてみました。

久しぶりに Vim Script を使ったのでちょっと苦労しましたが、ワークフローのエラーをローカルで発見できるようになったので、それだけの価値はあったと言えそうです。

[Vim]: https://www.vim.org/ 

[ALE]: https://github.com/dense-analysis/ale

[`circleci.vim`]: https://github.com/dense-analysis/ale/blob/master/ale_linters/yaml/circleci.vim

[Vundle]: https://github.com/VundleVim/Vundle.vim

[`hankei6km/ale-linter-actionlint.vim`]: https://github.com/hankei6km/ale-linter-actionlint.vim

[actionlint]: https://github.com/rhysd/actionlint

[node-actionlint]: https://zenn.dev/sosukesuzuki/articles/195f9fc1c7ed73

[trunk]: https://marketplace.visualstudio.com/items?itemName=Trunk.io
