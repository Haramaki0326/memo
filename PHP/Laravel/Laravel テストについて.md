# Laravel テストについて

## テストのコツ

### メソッドの構成

- 純粋関数にする

### ディレクトリ構成

- ビュー
- コントローラー
- モデル
- サービス

## Laravel におけるテストの概要

- 主に 3 つのテストがある
  - Unit テスト
  - Feature テスト
  - ブラウザテスト
- Laravel には PHPUnit がデフォルトでサポートされている
- `tests`ディレクトリには最初から以下が用意されている
  - `Unit`ディレクトリ
  - `Feature`ディレクトリ
- `Laravel Dusk`というブラウザテストをサポートするツールも用意されている
- `sail test`コマンドでテストを実行する

## Unit テスト

- テストコードの作成
  - ` sail artisan make:test ディレクトリ名 --unit`
    - 例：` sail artisan make:test Services/TweetServceTest --unit`
    - このコマンドで`test/Unit/Services/TweetServiceTest.php`ファイルが作成される
- サンプル：

```php
// 自分のtweetかどうかをチェックするメソッド
public function checkOwnTweet(int $userId, int $tweetId): bool
{
    $tweet = Tweet::where('id', $tweetId)->first();
    if (!$tweet) {
        return false;
    }

    return $tweet->user_id === $userId;
}
```

```php
<?php

namespace Tests\Unit\Services;

use PHPUnit\Framework\TestCase;
use App\Services\TweetService;
use Mockery;

class TweetServiceTest extends TestCase
{
    /**
     * @runInSeparateProcess
     * @return void
     */
    public function test_check_own_tweet()
    {
        $tweetService = new TweetService(); // TweetServiceのインスタンスを作成

        $mock = Mockery::mock('alias:App\Models\Tweet');
        $mock->shouldReceive('where->first')->andReturn((object)[
            'id' => 1,
            'user_id' => 1
        ]);

        $result = $tweetService->checkOwnTweet(1, 1);
        $this->assertTrue($result);

        $result = $tweetService->checkOwnTweet(2, 1);
        $this->assertFalse($result);
    }
}
```

### モック

- 上記のサンプルで、`$tweet = Tweet::where('id', $tweetId)->first();`では DB に問い合わせをしている
- テスト時にはこの部分をモックに差し替えていく必要がある
- `Mockery`というツールを利用する

```php
        $mock = Mockery::mock('alias:App\Models\Tweet');
        $mock->shouldReceive('where->first')->andReturn((object)[
            'id' => 1,
            'user_id' => 1
        ]);
```

- Doc コメントに

```php
@runInSeparateProcess
```

を追加して別プロセスで動かすようにしている

## Feature テスト

- 以下の流れが基本
  - `ModelFactory`でデータベースにデータを作る
  - テストしたい API を実行する
  - 返ってきたレスポンスに対してアサーションをする
- この`ModelFactory`や、`actingAs`というメソッドや、レスポンスに対する`assertOk`みたいものはすべて Laravel の拡張で、PHPUnit 自体のものではない

## DB テスト

### DB の準備

- テスト用データベースを準備する
- モック（`Mockery`）を利用する

### 各テスト後のデータベースリセット

- [Laravel 9.x データベーステスト](https://readouble.com/laravel/9.x/ja/database-testing.html)
- [【Laravel】データベースに影響が出ないようにテストする](https://c-a-p-engineer.github.io/tech/2022/11/15/laravel-test-trait/)
- [Laravel の RefreshDatabase はいつ実行されるか、なにを実行しているか](https://magai.hateblo.jp/entry/2021/01/23/163000)

### データベースに影響が出ないようにテストする

#### 結論

- 以下の２条件を満たすなら`RefreshDatabase`を利用する
  - テスト用の DB を用意している/利用する
  - マイグレーションを用意している/利用する
  - 理由：`RefreshDatabase`は以下の２点を行うから
    - マイグレーションを利用して、テスト前にデータの初期化を行う（なのでマイグレーションファイルがないと駄目）
    - マイグレーション後にトランザクションを張り、テスト後にロールバックを行う。これによりテストで利用した DB にテストデータが残らないようにする。
      - つまり、初期化後のロールバックなので既存データはすべて消える
- それ以外の条件なら`DatabaseTransactions `を利用する
  - これはテスト前にトランザクションを張り、テスト終了後にロールバックを行う
  - これによりテスト前後のデータに影響が出ないようにできる
  - ただし以下２点の懸念点がある
    - 初期化を行ってからテストを行うわけではないので、既存のデータに影響されうる
    - オートインクリメントはロールバックされない

#### ① トランザクションを使用する（早い）

- `DatabaseTransactions` を使用する。
  - [参考：Laravel で RefreshDatabase を使えない場合はどうするべきか](https://qiita.com/aminevsky/items/71e42ffee39c3b77a952)
  - `RefreshDatabase` はテストが終わったらマイグレーションを再度実行するというものなので、 マイグレーションを使わない場合 （例えば、既存 DB を使う場合）には適していないので、そういうときは代わりに`DatabaseTransactions`を利用する
- データベースにトランザクションを張って実行します。
- そのため他のテストに影響を与えませんが代わりにマイグレーションなどは自分で実装する必要があります。
- **オートインクリメントはロールバックされないので注意**
- PHPUnit のフックでテスト実行時にマイグレーションを実行して各テストはこれを使用するようにする
- 複数 DB にまたがってトランザクションを張る必要がある場合は `connectionsToTransact` を指定

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\DatabaseTransactions;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use DatabaseTransactions;

    protected $connectionsToTransact = ['mysql1', 'mysql2'];

    /**
     * 基本的な機能テスト例
     *
     * @return void
     */
    public function test_basic_example()
    {
        $response = $this->get('/');

        // ...
    }
}
```

#### ② `migration` を使用する（遅い）

- `DatabaseMigrations` を使用する。
- マイグレーションを使用してデータベースを完全にリセットしたい場合に使用。
- ただし、毎回マイグレーションを実行するためテストがすごく遅くなる

```php

<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use DatabaseMigrations;

    /**
     * 基本的な機能テスト例
     *
     * @return void
     */
    public function test_basic_example()
    {
        $response = $this->get('/');

        // ...
    }
}
```

#### ③ `migrate`, トランザクションの複合（やや遅い）

- `RefreshDatabase` を使用する。
- スキーマが最新ならマイグレートされない
- その代わりにトランザクション内でテストをするために他のテストに影響を与えません。
- ただし、マイグレーションを実行すると遅くなることがある
- こちらの方法も複数 DB にまたがってトランザクションを張る必要がある場合は `connectionsToTransact` を指定。

```php
-<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    protected $connectionsToTransact = ['mysql1', 'mysql2'];

    /**
     * 基本的な機能テスト例
     *
     * @return void
     */
    public function test_basic_example()
    {
        $response = $this->get('/');

        // ...
    }
}
```

#### `use RefreshDatabase`

- `RefreshDatabase`が何をしているか

  - DB を初期化して、migration を行う (1 度だけ)
    - `php artisan migrate:fresh`
    - ただしテストクラスでの指定によっては、`--drop-views`, `--drop-types`, `--seed`のオプションが有効になる。
  - TestCase 毎にトランザクションを開始して、TestCase 終了時に rollback してくれる

テストクラスでトレイトを使用する

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * 基本的な機能テスト例
     *
     * @return void
     */
    public function test_basic_example()
    {
        $response = $this->get('/');

        // ...
    }
}
```

- `Illuminate\Foundation\Testing\RefreshDatabase`トレイトは、スキーマが最新であれば、データベースをマイグレートしません。その代わりに、データベーストランザクション内でテストを実行するだけです。
- したがって、このトレイトを使用しないテストケースによってデータベースに追加されたレコードは、まだデータベースに残っている可能性があります。
- マイグレーションを使用してデータベースを完全にリセットしたい場合は、代わりに `Illuminate\Foundation\Testing\DatabaseMigrations` トレイトを使用します。
- しかし、 `DatabaseMigrations` は `RefreshDatabase` よりかなり低速です。

### 利用可能な assert

https://readouble.com/laravel/9.x/ja/http-tests.html#available-assertions

## ブラウザテスト

## HTTP テスト

## カバレッジ

- [PHPUnit と Xdebug でコードカバレッジの出力を試す](https://labor.ewigleere.net/2019/08/15/phpunit-coverage-output/)
