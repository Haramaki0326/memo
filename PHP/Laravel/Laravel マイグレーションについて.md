# Laravel マイグレーションについて

- [公式：Laravel 9.x マイグレーション](https://readouble.com/laravel/9.x/ja/migrations.html)

- [(Laravel) マイグレーション実施手順](https://hara-chan.com/it/programming/laravel-db-migration/)

## マイグレーション

- [参考：PHP フレームワーク『 Laravel 』入門 #4 マイグレーション機能で簡単データベース管理方法](https://knowledge.cpi.ad.jp/howto-cpi/laravel-migration/)

### 概要

- Laravel のマイグレーション機能では、まず migration ファイルという php ファイルを作成して、その中にアプリケーションで利用したいテーブルの定義（カラムの名前・データ型・制約など）を記述します。
- migration ファイルはデータベースの設計書のようなもので、設計書を記述し終わった後で `Artisan migrate` コマンドを実行すると、Laravel がその内容をもとに自動でデータベースを構成します。
- その他にも、Laravel のマイグレーション機能にはシーダーやテスト用ダミーデータの自動生成など便利な機能がたくさんある
- **DB の操作をしたい場合は都度`migrations`ファイルをコマンドで作成する。既存の`migrations`ファイルの内容を更新することはしない**

### マイグレーションファイル（テーブルの操作定義）の作成(`artisan make:migration`)

`artisan make:migration <マイグレーションファイル名> --create=<モデル名>`

- 例：`artisan make:migration create_hello_table --create=hello`
- `create_hello_table`はマイグレーションファイルの名前で、任意の文字列で OK です。
- `-create=hello`の`hello`の部分は作成するテーブル名で、モデル名と合わせます。
- ただし、**モデル名は先頭が大文字**で、**テーブル名は先頭が小文字**になるところにご注意ください。
- `Artisan make:migration` コマンドを実行すると、マイグレーションファイルが `database/migrations/`の直下に作成されます。

### migration ファイル命名規則

- [参考記事：Laravel の migration まとめ](https://codelikes.com/laravel-migration-summary/)
- `migration`コマンドのオプション一覧は以下で確認できる

```sh
$ php artisan make:migration -h

Description:
  Create a new migration file

Usage:
  make:migration [options] [--] <name>

Arguments:
  name                   The name of the migration

Options:
      --create[=CREATE]  The table to be created
      --table[=TABLE]    The table to migrate
      --path[=PATH]      The location where the migration file should be created
      --realpath         Indicate any provided migration file paths are pre-resolved absolute paths
      --fullpath         Output the full path of the migration
  -h, --help             Display help for the given command. When no command is given display help for the list command
  -q, --quiet            Do not output any message
  -V, --version          Display this application version
      --ansi|--no-ansi   Force (or disable --no-ansi) ANSI output
  -n, --no-interaction   Do not ask any interactive question
      --env[=ENV]        The environment the command should run under
  -v|vv|vvv, --verbose   Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
```

#### テーブル作成時(`--create`)

```sh
php artisan make:migration create_{テーブル名(複数形)} --create={テーブル名(複数形)}
```

例）`php artisan make:migration create_users --create=users`

#### テーブル変更時(`--table`)

```sh
php artisan make:migration modify_{テーブル名}_{YYYYMMDD} --table={テーブル名}
```

例）`php artisan make:migration modify_users_20160128 --table=users`

※ 上記フォーマットにしたがわないでファイルを作ると migration 実行時にエラーになる

### マイグレーションファイルの実行（`artisan migrate`）

#### 準備

作成されたファイルには、テーブルを作るための `up` メソッドとテーブルを破棄するための `down` メソッドがはじめから用意されています。
ただ、このままですとカラムが ID とタイムスタンプ（登録、更新日時）しかない状態なので、必要に応じて**カラムの情報を追加**していきます。

`database/migrations/<年月日時分秒>\_create_hello_table.php`

```php
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateHelloTable extends Migration
{
  /**
   * Run the migrations.
   *
   * @return void
   */
  public function up()
  {
    Schema::create('hello', function (Blueprint $table) {
      $table->bigIncrements('id');
      $table->timestamps();
//ここから追加する実装
      $table->string('str_1');
      $table->string('str_2');
//ここまで追加する実装
    });
  }

  /**
   * Reverse the migrations.
   *
   * @return void
   */
  public function down()
  {
    Schema::dropIfExists('hello');
  }
}
```

#### 実行

マイグレーションの実行も Artisan コマンドで行うことができます。
`artisan migrate`

### `migration`テーブルの見方

- `migrations`テーブルは、マイグレーションの実施履歴を保持するテーブルだ。
- マイグレーションファイル毎にレコードが存在する。
- `batch`カラムには、ファイルが migrate された順番が保持されている。

以下では、`create_characters_table` ファイルが直近で実行されたということだ。

```
+------+------------------------------------------------+-------+
| Ran? | Migration | Batch |
+------+------------------------------------------------+-------+
| Yes | 2014_10_12_000000_create_users_table | 1 |
| Yes | 2014_10_12_100000_create_password_resets_table | 1 |
| Yes | 2019_08_19_000000_create_failed_jobs_table | 1 |
| Yes | 2021_01_03_105018_create_skills_table | 2 |
| Yes | 2021_01_03_111401_create_characters_table | 3 | #直近のマイグレーション（3 回マイグレーションが行われたうちの３回目）
+------+------------------------------------------------+-------+
```

migrate 後は、`Ran?`カラムに Yes がつく。

## やり直し

- **基本的には以下の記事が一番参考になる。**
- [(Laravel) migration やり直しコマンドあれこれ](https://hara-chan.com/it/programming/laravel-migrate-commands/)

- やり直しコマンドには複数のコマンドが存在するが、いずれも以下のように、`php artisan migrate:コマンド` と言う風に入力して使用する。

### laravel のマイグレーションコマンドの実行方法

```sh
php artisan migrate:コマンド
```

### マイグレーションやり直しコマンド一覧

- `status` マイグレーションの状態確認
- `rollback` 直前のマイグレーションをロールバック
- `reset` すべてのマイグレーションをロールバック
- `refresh` すべてのマイグレーションをロールバックし migrate を実行。
- `fresh` すべてのテーブルを削除し migrate を実行。

### `rollback`

`rollback` は、直前に行った一連のマイグレーションをなかったことにするコマンド。
`rollback` を実施すると、マイグレーションファイルの **`down()`メソッドに書かれた処理が実行される。**
