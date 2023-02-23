# Laravel 機能別テスト実例（検索一覧、登録、更新、削除）

- [Laravel コントローラテストのテンプレまとめ](https://www.engilaboo.com/laravel-controller-test/)

## Feature テストの具体例

- 以下は、Laravel の Feature テストを利用して ProductController クラスのテストを行う例です。

```php
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithFaker;
use Tests\TestCase;
use App\Models\Product;

class ProductControllerTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function test_product_index()
    {
        // テスト用データの作成
        $products = factory(Product::class, 3)->create();

        // 商品一覧を取得するリクエストを送信する
        $response = $this->get('/products');

        // 正常なレスポンスであることを確認する
        $response->assertStatus(200);

        // 商品一覧が画面に表示されていることを確認する
        foreach ($products as $product) {
            $response->assertSee($product->name);
            $response->assertSee($product->price);
        }
    }

    /** @test */
    public function test_product_create()
    {
        // 商品情報を作成
        $product = factory(Product::class)->make();

        // 商品情報を登録するリクエストを送信する
        $response = $this->post('/products', [
            'name' => $product->name,
            'price' => $product->price,
            'description' => $product->description,
        ]);

        // 商品登録後に商品一覧画面に遷移することを確認する
        $response->assertRedirect('/products');

        // 商品情報がDBに登録されていることを確認する
        $this->assertDatabaseHas('products', [
            'name' => $product->name,
            'price' => $product->price,
            'description' => $product->description,
        ]);
    }

    /** @test */
    public function test_product_update()
    {
        // 商品情報を作成
        $product = factory(Product::class)->create();

        // 商品情報を更新するリクエストを送信する
        $response = $this->put('/products/'.$product->id, [
            'name' => 'updated name',
            'price' => 2000,
            'description' => 'updated description',
        ]);

        // 商品更新後に商品一覧画面に遷移することを確認する
        $response->assertRedirect('/products');

        // 商品情報がDBに更新されていることを確認する
        $this->assertDatabaseHas('products', [
            'id' => $product->id,
            'name' => 'updated name',
            'price' => 2000,
            'description' => 'updated description',
        ]);
    }

    /** @test */
    public function test_product_delete()
    {
        // 商品情報を作成
        $product = factory(Product::class)->create();

        // 商品情報を削除するリクエストを送信する
        $response = $this->delete('/products/'.$product->id);

        // 商品削除後に商品一覧画面に遷移することを確認する
        $response->assertRedirect('/products');

        // 商品情報がDBから削除されていることを
```

以下は、商品新規登録機能の Feature テストの例です。

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseTransactions;
use Illuminate\Foundation\Testing\WithFaker;
use Tests\TestCase;

class ProductTest extends TestCase
{
    use DatabaseTransactions;
    use WithFaker;

    /** @test */
    public function user_can_create_new_product()
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->post('/products', [
            'name' => $this->faker->word(),
            'description' => $this->faker->sentence(),
            'price' => $this->faker->numberBetween(100, 1000),
            // 他に必要な情報があれば追加する
        ]);

        $response->assertRedirect('/products');
        $this->assertDatabaseHas('products', [
            'name' => $data['name'],
            'description' => $data['description'],
            'price' => $data['price'],
            // 他に必要な情報があれば追加する
        ]);
    }
}
```

- このテストコードは、商品新規登録機能をテストしています。
- `DatabaseTransactions` トレイトを使っているため、テスト前にデータベースをトランザクションで囲み、テスト後にデータベースをロールバックすることで、テスト中に作成されたデータをデータベースに残さずに済ませることができます。
- `WithFaker` トレイトを使うことで、ランダムな値を簡単に生成することができます。この例では、商品名、商品説明、価格をランダムに生成しています。
- `actingAs` メソッドを使って、ユーザーとしてログインしている状態をシミュレートしています。
- `post` メソッドを使って、商品新規登録 API を呼び出しています。
- 最後に、レスポンスが正常なリダイレクトを返しているか、そして、商品がデータベースに登録されているかどうかをアサーションしています。

## リポジトリクラスの代わりにスタブやモックを使用した場合

- 例えば、以下のような UserRepository クラスがあるとします。

```php
class UserRepository
{
    public function findUserById($id)
    {
        return User::find($id);
    }
}
```

- このリポジトリクラスをテストするためには、通常は実際のデータベースに接続してテストを行う必要があります。
- しかし、モックやスタブを使用すると、テスト中にデータベースへのアクセスをシミュレートすることができます。
- 例えば、以下のようにテストを記述することができます。

```php
use App\UserRepository;
use Illuminate\Support\Facades\DB;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UserRepositoryTest extends TestCase
{
    use RefreshDatabase;

    public function testFindUserById()
    {
        $user = factory(User::class)->create();
        $repository = new UserRepository();
        DB::shouldReceive('find')->once()->with($user->id)->andReturn($user);
        $this->assertEquals($user->id, $repository->findUserById($user->id)->id);
    }
}
```

- 上記のテストコードでは、`DB::shouldReceive('find')->once()->with($user->id)->andReturn($user);` という行で、データベースアクセスをシミュレートしています。
- この行によって、データベースにアクセスすることなく、指定したユーザーのオブジェクトを返すことができます。

- スタブやモックを使用することで、テストの速度が向上するため、開発効率が上がるというメリットがあります。
- また、リポジトリクラスがデータベースアクセス以外の処理を行っている場合にも、その処理をシミュレートすることができます。
- これによって、テストの信頼性が向上し、バグを見つけやすくなります。

## 上述の`ProductService`クラスの`updateProduct`メソッドをモックを利用してテストするコードはどのようなものになりますか？

- `ProductService`クラスの`updateProduct`メソッドをモックを利用してテストするためには、`ProductRepository`クラスの代わりにモックオブジェクトを使用します。
- 具体的には、`ProductRepository`クラスのインスタンスを作成する代わりに、`Mockery`パッケージを使用してモックオブジェクトを作成します。
- 以下は、`ProductService`クラスの`updateProduct`メソッドをテストするためのコード例です。

```php
use App\Product;
use App\ProductRepository;
use App\ProductService;
use Mockery;
use Tests\TestCase;

class ProductServiceTest extends TestCase
{
    public function testUpdateProduct()
    {
        // モックオブジェクトを作成
        $productRepositoryMock = Mockery::mock(ProductRepository::class);

        // updateメソッドが1回呼び出されることを期待する
        $productRepositoryMock->shouldReceive('update')
            ->once()
            ->andReturn(true);

        // モックオブジェクトを使って ProductService クラスを作成
        $productService = new ProductService($productRepositoryMock);

        // 商品情報を作成
        $product = new Product();
        $product->id = 1;
        $product->name = 'updated product';

        // updateProductメソッドを呼び出し、trueが返ることを確認する
        $result = $productService->updateProduct($product);
        $this->assertTrue($result);
    }
}
```

- このテストコードでは、`ProductRepository`クラスの`update`メソッドが 1 回呼び出されることを期待しています。
- `$productRepositoryMock->shouldReceive('update')->once()->andReturn(true)` の部分で、モックオブジェクトに`update`メソッドをモック化し、1 回呼び出されたら true を返すように指定しています。
- テストコードの最後で、`$productService->updateProduct($product)` を呼び出し、戻り値が`true`であることを確認しています。
- このようにして、モックオブジェクトを使用して、`ProductService`クラスの`updateProduct`メソッドをテストすることができます。

## 正しい内容で DB が更新されることを確認するためには サービスクラスではなくリポジトリクラスのテストを行うべきですか？`ProductService`クラスと`ProductRepository`クラスのそれぞれの責務と何をテストするべきなのかがよく分かりません。

- サービスクラスはビジネスロジックを実装し、複数のリポジトリを利用することがあります。
- 一方、リポジトリクラスは、データの永続化を担当します。
- そのため、`ProductService`クラスの`updateProduct`メソッドのテストでは、以下のようなケースを考慮する必要があります。
  - ProductRepository クラスの update メソッドが呼び出されること
  - ProductRepository クラスの update メソッドに渡される引数が正しいこと
  - ProductRepository クラスの update メソッドが正常に実行されること
- 上記のようなテストを行うことで、DB が更新されることを確認できます。

- 一方、ProductRepository クラス自体をテストする場合、以下のようなケースを考慮する必要があります。
  - Product モデルの作成が正常に行われること
  - Product モデルの更新が正常に行われること
  - Product モデルの削除が正常に行われること
  - Product モデルの取得が正常に行われること
- つまり、サービスクラスとリポジトリクラスは役割が異なるため、それぞれの責務に応じたテストを行う必要があります。
- ただし、ProductRepository クラスのテストには、DB に接続する必要があるため、実行に時間がかかります。
- そのため、テストの実行時間を短縮するために、スタブやモックを使用することが一般的です。

### `ProductRepository` クラスのテストコード

- ProductRepository クラスの update メソッドをモックを利用してテストする例を示します。

```php
use PHPUnit\Framework\TestCase;
use App\Models\Product;
use App\Repositories\ProductRepository;

class ProductRepositoryTest extends TestCase
{
    public function testUpdate()
    {
        $mockProduct = new Product();
        $mockProduct->id = 1;
        $mockProduct->name = 'mock name';
        $mockProduct->price = 1000;
        $mockProduct->stock = 10;

        $mockQueryBuilder = $this->getMockBuilder(Illuminate\Database\Query\Builder::class)
                                 ->disableOriginalConstructor()
                                 ->getMock();
        $mockQueryBuilder->expects($this->once())
                         ->method('where')
                         ->with('id', 1)
                         ->willReturnSelf();
        $mockQueryBuilder->expects($this->once())
                         ->method('update')
                         ->with([
                             'name' => 'mock name',
                             'price' => 1000,
                             'stock' => 10,
                         ])
                         ->willReturn(true);

        $mockDb = $this->getMockBuilder(Illuminate\Database\ConnectionInterface::class)
                       ->disableOriginalConstructor()
                       ->getMock();
        $mockDb->expects($this->once())
               ->method('table')
               ->with('products')
               ->willReturn($mockQueryBuilder);

        $productRepository = new ProductRepository($mockDb);
        $result = $productRepository->update($mockProduct);

        $this->assertTrue($result);
    }
}
```

- このテストでは、ProductRepository クラスの update メソッドが、Product モデルのインスタンスを受け取り、データベースの products テーブルの該当するレコードを更新することを想定しています。
- 具体的には、Illuminate\Database\ConnectionInterface クラスと Illuminate\Database\Query\Builder クラスをモックして、update メソッドが正しく実行されることを確認しています。

- このように、リポジトリクラスのテストでは、データベースに対する操作をモックすることで、依存するオブジェクトの影響を受けずにテストを行うことができます。
