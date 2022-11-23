# `gitconfig`について

## 参考

- [gitconfig の基本を理解する
  ](https://qiita.com/shionit/items/fb4a1a30538f8d335b35)
- [[Git]これだけ押さえる！gitconfig の基本](https://qiita.com/C_HERO/items/c35e679f0b03a5f06469)

## 概要

### gitconfig の設定は 3 段階

- git の config 情報の設定は、3 段階に分かれている。
  - システム全体 `system`
  - ユーザ全体 `global`
  - 対象リポジトリのみ `local`
- 設定は、 `system`, `global`, `local` の順に読み込まれ、最後に読み込まれた設定で上書きされるため、よりスコープの狭い設定が有効になる。
- それぞれの設定箇所の内容をリスト出力するには `git config` コマンドにオプションを追加すればいい。例えば `global` の場合は
  - `$ git config --global --list`

### 設定ファイルの格納場所

#### `local`

- 対象リポジトリ内の `.git/config`

#### `global`

- ログインユーザの`HOME`ディレクトリ配下の `.gitconfig`

| OS        | .gitconfig path                                           |
| --------- | --------------------------------------------------------- |
| Linux/Mac | `~/.gitconfig`                                            |
| Win       | `[home]¥.gitconfig` (通常は `C:¥Users¥[user]¥.gitconfig`) |

#### system

- git インストールディレクトリの `gitconfig` ファイルだが、どんなアプリにバンドルされた git か？によって場所はまちまちの模様。
