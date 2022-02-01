---
id: completion-label-names-in-github-cli
title: GitHub CLI でラベル名の入力補完をできるようにした
emoji: 🏷️
type: idea
topics:
  - githubcli
  - bash
published: true
---

GitHub にリリースノートの自動生成機能が付いたそうなので、「そろそろ自動生成にそなえてプルリクエストに label は設定しておこうかな[^use-label]」と思いました。

ところが、[GitHub CLI] ではラベル名の入力補完ができなかったので、入力補完をできるようなスクリプトを作成しました。

@[card](https://www.publickey1.jp/blog/21/github_releases.html)
@[card](https://docs.github.com/ja/repositories/releasing-projects-on-github/automatically-generated-release-notes)

[^use-label]: 自動生成用のツールでは label が使われていることが多そうな感じだったので。

## ラベル名の一覧表示

まずは一覧の表示。

### リポジトリに登録されているラベル

これも [GitHub CLI] では表示できなさそうだったので API を使って取得します。

▼ *図 1-1 リポジトリのラベル一覧取得*

```shell-session
$ gh api repos/{owner}/{repo}/labels --jq '.[].name'
bug
documentation
duplicate
enhancement
good first issue
help wanted
invalid
question
wontfix
```

### PR や Issue に登録されているラベル

これは [GitHub CLI] でできました。図 1-2 は Issue から取得していますが、PR の場合はコマンドを `pr` へ変更すると対応できます。

▼ *図 1-2 Issue #2 のラベル一覧取得*

```shell-session
$ gh issue view 2 --json labels --jq '.labels[].name'
bug
documentation
question
```

## 入力補完

当初は上記ラベル一覧からコピペする予定でしたがやはり面倒になったので補完できるようにしました。

### 補完候補を作成する関数

ラベル名の一覧が取得できたのでこれを補完候補にする関数を作成します。

本来ならサブコマンドなどで場合分けしてから候補を作成すべきですが、今回はその辺のところはやっつけになっています(重複しているコードが多い)。

:::details (リスト 2-1 一部抜粋したソース)

```sh:gh-label-completion
__gh_label_completion_labels_get_repo_from_words() {
  local REPO=""
  if [ "${words[2]}" = "-R" ] || [ "${words[2]}" = "--repo" ] ; then
    local SAVE_IFS="${IFS}"
    IFS="/"
    local ELM
    read -r -a ELM <<< "${words[3]}"
    IFS="${SAVE_IFS}"
    if [ "${#ELM[@]}" -eq 2 ] ; then
      REPO="${ELM[0]}/${ELM[1]}" 
    elif [ "${#ELM[@]}" -eq 3 ] ; then
      REPO="${ELM[0]}/${ELM[1]}/${ELM[2]}" 
    fi
  fi
  echo -n "${REPO}"
}

__gh_label_completion_labels_from_repo() {
  local SAVE_IFS="${IFS}"
  local REPO_PATH="{owner}/{repo}"
  test -n "${1}" && REPO_PATH="${1}"
  mapfile -t COMPREPLY < <(IFS=$'\n'; compgen -W "$(gh api "repos/${REPO_PATH}/labels" --jq '.[].name')" -- "${cur}";IFS="${SAVE_IFS}")
}

__gh_label_completion_labels_from_target() {
  local SAVE_IFS="${IFS}"
  mapfile -t COMPREPLY < <(IFS=$'\n'; compgen -W "$(gh "${@}" --json labels --jq '.labels[].name')" -- "${cur}";IFS="${SAVE_IFS}")
}

__gh_label_completion() {
  # local cur prev words cword split
  local cur prev words
  _init_completion -s || return

  if [ "${words[1]}" = "issue" ]; then
    if [ "${words[2]}" = "create" ] || [ "${words[5]}" = "create" ] || \
       [ "${words[2]}" = "list" ] || [ "${words[5]}" = "list" ]; then
      case "${prev}" in
        -l | --label)
          __gh_label_completion_labels_from_repo "$(__gh_label_completion_labels_get_repo_from_words)"
          return
          ;;
      esac

    elif [ "${words[2]}" = "edit" ] || [ "${words[4]}" = "edit" ]; then
      case "${prev}" in
        --add-label)
          __gh_label_completion_labels_from_repo "$(__gh_label_completion_labels_get_repo_from_words)"
          return
          ;;
        --remove-label)
          local ISSUE=""
          if [ "${words[2]}" = "edit" ] && [ "${words[3]:1:1}" != "-" ] ; then
            ISSUE="${words[3]}"
          fi
          if [ "${words[4]}" = "edit" ] && [ "${words[5]:1:1}" != "-" ] ; then
            ISSUE="${words[5]}"
          fi
          if [ -n  "${ISSUE}" ]; then
            local ARGS=("issue" "view" "${ISSUE}")
            local REPO=""
            REPO="$(__gh_label_completion_labels_get_repo_from_words)"
            test -n "${REPO}" && ARGS=("${ARGS[@]}" "--repo" "${REPO}")
            __gh_label_completion_labels_from_target "${ARGS[@]}"
          fi
          return
          ;;
      esac
    fi

  elif [ "${words[1]}" = "pr" ]; then
    if [ "${words[2]}" = "create" ] || [ "${words[5]}" = "create" ] || \
       [ "${words[2]}" = "list" ] || [ "${words[5]}" = "list" ]; then
      case "${prev}" in
        -l | --label)
          __gh_label_completion_labels_from_repo "$(__gh_label_completion_labels_get_repo_from_words)"
          return
          ;;
      esac

    elif [ "${words[2]}" = "edit" ] || [ "${words[4]}" = "edit" ]; then
      case "${prev}" in
        --add-label)
          __gh_label_completion_labels_from_repo "$(__gh_label_completion_labels_get_repo_from_words)" 
          return
          ;;
        --remove-label)
          local PR=""
          local REPO=""
          if [ "${words[2]}" = "edit" ] && [ "${words[3]:1:1}" != "-" ] ; then
            PR="${words[3]}"
          fi
          if [ "${words[4]}" = "edit" ] && [ "${words[5]:1:1}" != "-" ] ; then
            PR="${words[5]}"
          fi
          local ARGS=("pr" "view")
          local REPO=""
          REPO="$(__gh_label_completion_labels_get_repo_from_words)"
          test -n "${REPO}" && ARGS=("${ARGS[@]}" "--repo" "${REPO}")
          test -n "${PR}" && ARGS=("${ARGS[@]}" "${PR}")
          __gh_label_completion_labels_from_target "${ARGS[@]}"
          return
          ;;
      esac
    fi
  fi
}
```

:::

基本的には以下の記事を参考にし、ShellCheck で警告が出たところなどを変更しています。

@[card](https://qiita.com/nil2/items/8a1544e206928c753a2e)
@[card](https://qiita.com/nil2/items/3ae0c06732af35a0f7f2)
@[card](https://github.com/koalaman/shellcheck/wiki/SC2207)

### 関数を `gh` に設定する

「もともとの補完設定を拡張する形で自前の関数を設定する方法」がわからなかったので、今回は以下のようにしました。

*   オリジナルの関数を展開しておく(`complete` が実行されるが後から上書きする)
*   補完候補が 0 件だったら自前の関数を実行する関数で上書き

:::details (リスト 2-2 一部抜粋したソース)

```sh:gh-label-completion
# オリジナルの補完関数を展開する
eval "$(gh completion -s bash)"


<snip>


__gh_label_completion_start() {
  # 通常の入力補完.
  __start_gh
  # 候補がなければ label 用の補完.
  test "${#COMPREPLY[@]}" -eq 0 && __gh_label_completion
}

if [[ $(type -t compopt) = "builtin" ]]; then
  complete -o default -F __gh_label_completion_start gh
else
  complete -o default -o nospace -F __gh_label_completion_start gh
fi
```

:::

### とりあえず完成

[こちらの記事](https://serverfault.com/questions/506612/standard-place-for-user-defined-bash-completion-d-scripts) などを参考に `gh-label-completion` を配置すると補完ができるようになります。

![入力補完のデモ表示しているスクリーンショットの agif](https://raw.githubusercontent.com/hankei6km/gh-label-completion/main/docs/gh-label-completion.gif?raw=true)
*図 2-1 入力のデモ*

@[card](https://github.com/hankei6km/gh-label-completion)

## 課題

毎回 API を実行するので負荷かけてしまわないかは心配なところです(操作性でももっさり感がありありです)。

他にもいろいろありますが、`-R` `--repo` フラグ対応は妙なコードを挿入できないか少し心配なので外してしまおうかと考え中です[^repo]。

[^repo]: 個人的には使っていなかったフラグなのと、` HOST/OWNER/REPO` 形式でリポジトリを指定して `gh api` を使う方法がわからなかったというのもあります。

## おわりに

コミットログの内容から補完候補を絞り込めると面白いかなと考えていたのですが、最近は label 付けも GitHub Actions でやるのが主流のようなのでとりあえずはこの辺で使っていこうかなと。

## スクラップ

この記事は以下のスクラップを元に作成しました。

@[card](https://zenn.dev/hankei6km/scraps/37fab052cc812d)

[GitHub CLI]: https://cli.github.com/
