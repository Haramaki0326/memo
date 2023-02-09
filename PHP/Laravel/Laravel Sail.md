# Laravel Sail

## インストール手順

- [公式 - ReadDouble](https://readouble.com/laravel/9.x/ja/installation.html#:~:text=%E3%81%A6%E3%81%8F%E3%81%A0%E3%81%95%E3%81%84%E3%80%82-,Windows%E3%81%A7%E5%A7%8B%E3%82%81%E3%82%8B,-Windows%E3%83%9E%E3%82%B7%E3%83%B3%E3%81%AB)
- WSL2(Ubuntu)の CLI 上で以下のコマンドを実行
- `curl -s https://laravel.build/example-app | bash`
  - `example-app`の名前は任意
  - インストールの最後にパスワードを求められるので入力
- インストールが完了したらあとは以下コマンドで VSCode 上で作業が可能
  - `cd example-app`
  - `code .`

## 起動と停止

- ※以下は基本的に VSCode 上の CLI で可能。CLI の種類を`git bash`ではなく`wsl`に指定すること
- 以下コマンドを実行し Laravel Sail コンテナを起動する
  - `./vendor/bin/sail up`
  - 起動後、`http://localhost`でアクセス
- 停止する場合は、以下のコマンドを実行
  - `$ sail down`

## `artisan`

- artisan のコマンド一覧を確認します。
  - `sail artisan list`
- ルーティングの一覧を表示してみましょう。
  - `sail artisan route:list`

## Node の実行

- Node のバージョン確認
  - `sail node --version`
- パッケージをインストール
  - `sail npm install`

### NPM で JS/CSS をビルド

- 本番環境向けファイル出力
  - `$ sail npm run build`
- 開発時
  - `$ sail npm run dev`

## テーブルの作成

- `.env` ファイルが既に存在し、データーベースの接続設定も記載されているため、何も設定せずにテーブルの作成が可能
- `sail artisan migrate`

## phpMyAdmin のインストール

- [【Laravel Sail】phpMyAdmin をインストール](https://chigusa-web.com/blog/laravel-sail-phpmyadmin/)
- Sail の中身は Docker ですので、Laravel プロジェクト直下にある`docker-compose.yml`を修正します。
  - phpMyAdmin に関する情報を、services ブロック内に追記します。

```yml
version: "3"
services:
  laravel.test:
    phpmyadmin:
      image: phpmyadmin/phpmyadmin
      links:
        - mysql:mysql
      ports:
        - 8080:80
      environment:
        #PMA_USER: "${DB_USERNAME}"
        #PMA_PASSWORD: "${DB_PASSWORD}"
        PMA_HOST: mysql
      networks:
        - sail
```

## vscode の設定

- Laravel Sail は WSL 上の各コンテナ上で実行されるため以下の点に注意
  - vscode を開く際、`Dev Container`で直接コンテナを開くこと
    - そうしないと、vscode の各種 linter（php-cs-fixer,phpcs,phpstan）が正常に動作しないため
  - `Dev Container`上で git を有効化する
    - git の認証情報を`Dev Container`とホスト側と共有する機能がデフォルトで存在する
      - [VS Code で Docker 開発コンテナを便利に使おう -qiita](https://qiita.com/Yuki_Oshima/items/d3b52c553387685460b0#git-1)
      - 注意点としては、Laravel Sail の場合はホストは Windows ではなく WSL になるので、WSL 上でも正しく git 認証情報（username, email）を登録しておく必要がある。
  - 保存時に`PHP-CS-FIXER`の自動フォーマットを有効化するには VSCode の setting.json に PHP ファイルのフォーマッターを指定する必要がある
    - `"[php]": { "editor.defaultFormatter": "junstyle.php-cs-fixer" },`
  - `git` コマンド実行時に WSL だと`fatal: detected dubious ownership in repository`エラーが発生する場合があるのでその場合は、セーフディレクトリとして追加する対処をする。
    - 結論：wsl 側の`.gitconfig`に`safe.directory=*`を追加する
    - [git 2.35.2 ～ / config.lock failed いきなり git が使えなくなった - 2022 いそいで使った safe.directory](https://www.nda.co.jp/memo/git_safe_directory/)
    - [Git で「fatal: detected dubious ownership in repository」が出力されコミットできない場合](https://chigusa-web.com/blog/git-fatal-error/)
