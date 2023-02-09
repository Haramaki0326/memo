# Laravel DB の扱い（Seeder,Faker,Factory）

## マイグレーション

## Seeder

### 概要

- `DatabaseSeeder`クラスを利用することで DB のデータ登録が行える
  - `php artisan db:seed`
- `DatabaseSeeder`から個別の `Seeder` クラスを呼び出せる
- 個別の `Seeder` クラスでは

  - 単純な INSERT 処理
  - `Factory`クラス を使ってまとめて登録をすることも可能
    - `Factory` では `faker` ライブラリを使うランダム生成が可能

- Laravel の Seeder は、データベースに初期データを登録するために使用します。
  - 最初から用意しておきたいマスターデータやテスト用のデータを作成するときに使えます。
- 作成した Seeder を実行するには artisan コマンドを使用します。
  - `php artisan db:seed`
- シーダクラスには、デフォルトで１つのメソッド、`run`のみ存在します。
- このメソッドは、`db:seed Artisan`コマンドが実行されるときに呼び出されます。
- run メソッド内で、データベースにデータを好きなように挿入できます。
- クエリビルダを使用してデータを手作業で挿入するか、Eloquent モデルファクトリを使用できます。
- 例として、デフォルトの DatabaseSeeder クラスを変更し、データベース挿入文を run メソッドに追加しましょう。

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    /**
     * データベースに対するデータ設定の実行
     *
     * @return void
     */
    public function run()
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@gmail.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

### Seeder ファイルの用意

- `artisan`コマンドで`seeder`ファイルを作成する
  - 例：`php artisan make:seeder UserSeeder`
- `Model`を作成する際のオプションで一緒に`Seeder`を作ることも可能
  - `artisan make:model Hello -s` seeder のみ
  - `artisan make:model Hello -a` seeder 含めて全部
- Seeder のファイルは`database/seeders`配下に配置されます。

### Seeder ファイルの記述

- `run` メソッドが Seeder を実行したときに処理されるメソッドです。
- ここに登録処理を書いていきます。

<details>
<summary>コーディング例</summary>

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Fruit;

class FruitSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        Fruit::create([
            'name' => 'リンゴ',
            'color' => '赤',
            'price' => '100',
        ]);

        Fruit::create([
            'name' => 'ぶどう',
            'color' => '紫',
            'price' => '300',
        ]);

        Fruit::create([
            'name' => 'みかん',
            'color' => '黄',
            'price' => '110',
        ]);
    }
}
```

</details>

### Seeder ファイルを`DatabaseSeeder`に登録

- 実行するシーダークラスの`class`プロパティを引数にして`call`メソッドを呼び出す
  - `$this->call(シーダークラス::class);`
  - これにより指定したクラスの`run`メソッドが呼び出される

```php
   public function run()
    {
        // \App\Models\User::factory(10)->create();

        // \App\Models\User::factory()->create([
        //     'name' => 'Test User',
        //     'email' => 'test@example.com',
        // ]);
    }
```

### Seeding の実行

```sh
php artisan db:seed
```

`DatabaseSeeder`の`run`メソッドが実行される

## Factory

### 概要

- [Laravel 9.x Eloquent:ファクトリ](https://readouble.com/laravel/9.x/ja/eloquent-factories.html)
- 例：`database/factorys/UserFactory.php`

```php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    /**
     * モデルのデフォルト状態の定義
     *
     * @return array
     */
    public function definition()
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
            'remember_token' => Str::random(10),
        ];
    }
}
```

- ファクトリは基本`Factory`クラスの拡張
- ファクトリは`definition` メソッドを定義するクラス
  - ファクトリを使用してモデルを作成するときに適用する必要がある属性値のデフォルトセットを返します。
- `faker` ヘルパを使うとさまざまな種類のランダムデータを生成可能。
  - `config/app.php` 設定ファイルに `faker_locale` オプションを追加することで、アプリケーションの Faker ロケールを設定できます。

### ファクトリの生成

- ファクトリを作成するには、Artisan コマンドを実行します。
  - `php artisan make:factory PostFactory`
- 新しいファクトリクラスは、`database/factories`ディレクトリに配置されます。

#### モデルとファクトリー個別に生成する

```sh
$ php artisan make:model Post
$ php artisan make:factory PostFactory --model=Post
```

#### モデルとファクトリーを生成する

```sh
$ php artisan make:model Post --factory
$ php artisan make:model Post -f # 省略オプション
```

#### モデルとファクトリー、マイグレーション、シーダー、コントローラを生成する

```sh
$ php artisan make:model Post --all
$ php artisan make:model Post -a # 省略オプション
```

### 利用側

```php
/** @var \App\Models\Book $book */
$book = Book::factory()->create();
```

- `factory` 関数ではなく、モデルから `static` に呼び出すことでテストデータを作成する

### Seeder への追記

`database/seeders/DatabaseSeeder.php`

```php
public function run(): void
{
    \App\Models\Post::factory()->count(3)->create();
}
```

### 状態（state）について

## Faker

### 参考

- [公式](https://fakerphp.github.io/)
- [FakerPHP 非公式リファレンス](https://fwhy.github.io/faker-docs/)

### 使い方

- [Laravel 公式](https://readouble.com/laravel/9.x/ja/eloquent-factories.html#:~:text=fake%E3%83%98%E3%83%AB%E3%83%91%E3%82%92,%E3%81%A7%E3%81%8D%E3%80%81%E4%BE%BF%E5%88%A9%E3%81%A7%E3%81%99%E3%80%82)の説明にあるとおり、`fake` ヘルパを `Factory` クラスの中で利用する
- 公式以外のサイトなどで説明している、以下のような書き方は NG
  - `Faker\Factory::create('ja_JP')->realText()`
- `config/app.php`設定ファイルに`faker_locale`オプションを追加することで、アプリケーションの Faker ロケールを設定できる

サンプルコード
`database/factories/ProductFactory.php`

```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * @extends \Illuminate\Database\Eloquent\Factories\Factory<\App\Models\Product>
 */
class ProductFactory extends Factory
{
    /**
     * Define the model's default state.
     *
     * @return array<string, mixed>
     */
    public function definition()
    {
        return [
            'name' => fake()->realText(),
            'price' => fake()->randomNumber(7),
            'created_at' => now(),
            'updated_at' => now(),
        ];
    }
}

```

#### 日本語テキスト(`realText()`)

- [FakerPHP 非公式リファレンス](https://fwhy.github.io/faker-docs/formatters/text/real_text.html)にある通り、日本語の自然な文章を扱うことも可能
- `fake()->realText()`
