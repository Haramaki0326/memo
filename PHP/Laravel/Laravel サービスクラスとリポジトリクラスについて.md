# Laravel サービスクラスとリポジトリクラスについて

## サービスクラスの概要

- サービスクラスは、アプリケーションのビジネスロジックを処理するために使用されるクラス
- コントローラーやモデルのような他のコンポーネントと同じように、サービスクラスは Laravel の依存性注入（Dependency Injection）システムと統合されています。
- サービスクラスは、以下のような機能を提供します。
  - ビジネスロジックの集中管理：ビジネスロジックをサービスクラスに集中させることで、コントローラーやモデルに依存することなく、アプリケーションのロジックを管理できます。
  - コードの再利用性：サービスクラスは、アプリケーション全体で再利用できる汎用的なコードを記述することができます。
  - テストの容易性：サービスクラスは、依存関係を注入することができるため、単体テストを実行しやすくなります。

## サービスクラスの導入するメリット

- コードの保守性：サービスクラスを使用することで、アプリケーションのビジネスロジックを分離できます。そのため、コードの保守性が向上します。
- テストの容易性：サービスクラスは、依存関係を注入することができるため、単体テストを実行しやすくなります。
- 拡張性：サービスクラスは、Laravel の依存性注入システムと統合されているため、拡張性が高く、アプリケーションの機能を容易に追加できます。
- コードの再利用性：サービスクラスは、アプリケーション全体で再利用できる汎用的なコードを記述することがで

## 具体例

```php
<?php

namespace App\Services;

use App\Models\Category;

class CategoryService
{
    protected $category;

    public function __construct(Category $category)
    {
        $this->category = $category;
    }

    public function getCategoryById($id)
    {
        return $this->category->find($id);
    }

    public function getAllCategories()
    {
        return $this->category->all();
    }

    public function createCategory($data)
    {
        return $this->category->create($data);
    }

    public function updateCategory($id, $data)
    {
        return $this->category->where('id', $id)->update($data);
    }

    public function deleteCategory($id)
    {
        return $this->category->where('id', $id)->delete();
    }
}
```

- この例では、CategoryService という名前のサービスクラスを定義しています。
- このクラスは、Category モデルを依存関係として受け取ります。
- そして、カテゴリーデータを取得、作成、更新、削除するためのメソッドを提供します。
- これらのメソッドは、Category モデルのクエリービルダーメソッドを使用して、カテゴリーテーブルからデータを操作します。
- サービスクラスは、ビジネスロジックを分離して、コントローラーやモデルに依存しないようにしています。
- また、依存性注入システムを使用して、コントローラーからサービスクラスを呼び出すことができます。

このように、サービスクラスは、ビジネスロジックを分離してコードの再利用性を高めるために、Laravel アプリケーションで広く使用されます。

## 具体例２

- 商品検索一覧画面：検索条件を入力し、それに該当する商品を一覧で表示する画面
- 商品新規登録画面：DB に登録されていない商品情報を入力し、DB に登録するための画面
- 商品詳細画面：DB に登録済みの商品情報を参照する画面
- 商品更新画面：DB に登録済みの商品情報を別の内容に更新するための画面

商品に関する画面を作成する場合、Laravel アプリケーションでの一般的なアーキテクチャは、コントローラー、サービス、リポジトリの 3 層構造を使用することです。以下は、各画面に対応するコントローラー、サービス、リポジトリの例です。

### 商品検索一覧画面

#### コントローラー

```php
<?php

namespace App\Http\Controllers;

use App\Services\ProductService;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    protected $productService;

    public function __construct(ProductService $productService)
    {
        $this->productService = $productService;
    }

    public function index(Request $request)
    {
        $products = $this->productService->getProductsBySearchCriteria($request->all());
        return view('products.index', ['products' => $products]);
    }
}
```

#### サービス

```php
<?php

namespace App\Services;

use App\Repositories\ProductRepository;

class ProductService
{
    protected $productRepository;

    public function __construct(ProductRepository $productRepository)
    {
        $this->productRepository = $productRepository;
    }

    public function getProductsBySearchCriteria($searchCriteria)
    {
        return $this->productRepository->getProductsBySearchCriteria($searchCriteria);
    }
}
```

#### リポジトリ

```php
<?php

namespace App\Repositories;

use App\Models\Product;

class ProductRepository
{
    protected $product;

    public function __construct(Product $product)
    {
        $this->product = $product;
    }

    public function getProductsBySearchCriteria($searchCriteria)
    {
        return $this->product->where('name', 'like', '%' . $searchCriteria['name'] . '%')
            ->where('price', '>=', $searchCriteria['min_price'])
            ->where('price', '<=', $searchCriteria['max_price'])
            ->get();
    }
}
```

### 商品新規登録画面

#### コントローラー

```php
<?php

namespace App\Http\Controllers;

use App\Services\ProductService;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    protected $productService;

    public function __construct(ProductService $productService)
    {
        $this->productService = $productService;
    }

    public function create()
    {
        return view('products.create');
    }

    public function store(Request $request)
    {
        $this->productService->createProduct($request->all());
        return redirect()->route('products.index');
    }
}
```

#### サービス

```php
<?php

namespace App\Services;

use App\Repositories\ProductRepository;

class ProductService
{
    protected $productRepository;

    public function __construct(ProductRepository $productRepository)
    {
        $this->productRepository = $productRepository;
    }

    public function createProduct($productData)
    {
        return $this->productRepository->create($productData);
    }
}
```

#### リポジトリ

```php
<?php

namespace App\Repositories;

use App\Models\Product;

class ProductRepository
{
    protected $product;

    public function __construct(Product $product)
    {
        $this->product = $product;
    }

    public function create($productData)
    {
        return $this->product->create($productData);
    }
}
```

### 商品詳細画面

#### コントローラー

```php
<?php

namespace App\Http\Controllers;

use App\Services\ProductService;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    protected $productService;

    public function __construct(ProductService $productService)
    {
        $this->productService = $productService;
    }

    public function show($id)
    {
        $product = $this->productService->getProductById($id);
        return view('products.show', ['product' => $product]);
    }
}
```

#### サービス

```php
<?php

namespace App\Services;

use App\Repositories\ProductRepository;

class ProductService
{
    protected $productRepository;

    public function __construct(ProductRepository $productRepository)
    {
        $this->productRepository = $productRepository;
    }

    public function getProductById($productId)
    {
        return $this->productRepository->getProductById($productId);
    }
}
```

#### リポジトリ

```php
<?php

namespace App\Repositories;

use App\Models\Product;

class ProductRepository
{
    protected $product;

    public function __construct(Product $product)
    {
        $this->product = $product;
    }

    public function getProductById($productId)
    {
        return $this->product->find($productId);
    }
}
```

### 商品更新画面の場合

#### コントローラー

```php
<?php

namespace App\Http\Controllers;

use App\Services\ProductService;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    protected $productService;

    public function __construct(ProductService $productService)
    {
        $this->productService = $productService;
    }

    public function edit($id)
    {
        $product = $this->productService->getProductById($id);
        return view('products.edit', ['product' => $product]);
    }

    public function update(Request $request, $id)
    {
        $this->productService->updateProduct($id, $request->all());
        return redirect()->route('products.show', ['id' => $id]);
    }
}
```

#### サービス

```php
<?php

namespace App\Services;

use App\Repositories\ProductRepository;

class ProductService
{
    protected $productRepository;

    public function __construct(ProductRepository $productRepository)
    {
        $this->productRepository = $productRepository;
    }

    public function updateProduct($productId, $productData)
    {
        return $this->productRepository->update($productId, $productData);
    }
}
```

#### リポジトリ

```php
<?php

namespace App\Repositories;

use App\Models\Product;

class ProductRepository
{
    protected $product;

    public function __construct(Product $product)
    {
        $this->product = $product;
    }

    public function update($productId, $productData)
    {
        $product = $this->product->find($productId);
        $product->update($productData);
        return $product;
    }
}
```

- 上記の例では、各コントローラーはサービスクラスを使用して必要なビジネスロジックを実行し、リポジトリクラスを使用してデータベースと通信します。
- サービスクラスでは、リポジトリクラスをラップし、コントローラーから渡された引数を処理してリポジトリクラスに渡します。リポジトリクラスでは、データベースにアクセスし、必要なデータの取得、更新、作成などを行います。
- これにより、ビジネスロジックとデータベースのアクセスを分離し、コードの再利用性とテストのしやすさが向上します。
- また、サービスクラスを使用することで、複数のコントローラーが同じビジネスロジックを共有できるため、コードの重複を避けることができます。
- さらに、サービスクラスを使用することで、アプリケーションの機能を簡単に拡張することができます。新しい機能を追加する場合は、新しいサービスクラスを作成し、必要なビジネスロジックを実装するだけです。これにより、コードのメンテナンス性と拡張性が向上します。

以上のように、Laravel プロジェクトにおいて、サービスクラスを導入することで、ビジネスロジックとデータベースアクセスを分離し、コードの再利用性、テストのしやすさ、メンテナンス性、拡張性を向上させることができます。

## テスト向上について

- サービスクラスを使用することにより、ビジネスロジックとデータベースのアクセスを分離することができます。
- この分離により、ビジネスロジックのテストとデータベースアクセスのテストを分離できます。
- 具体的には、サービスクラスにビジネスロジックを実装
- コントローラーはビューとのやりとりを行うことになります。
- このため、ビジネスロジックをテストする場合は、サービスクラスだけを対象にテストを実行することができます。
- これにより、ビジネスロジックに問題がある場合は、サービスクラスだけを修正すればよくなります。
- また、データベースのアクセスを行う場合は、データベースとの接続が必要となります。
- しかし、サービスクラスを使用することにより、データベースアクセスの処理をサービスクラスにまとめることができます。
- このため、データベースアクセスのテストを行う場合は、サービスクラスだけを修正することで、テストが実行できるようになります。

このように、サービスクラスを使用することにより、ビジネスロジックとデータベースアクセスを分離し、それぞれを独立してテストすることができます。これにより、テストの精度が向上し、テストの作成や実行が容易になるため、品質の高いアプリケーションを作成することができます。

## サービスクラスとリポジトリクラスを分けるメリット

### ビジネスロジックとデータアクセスの分離

- サービスクラスはビジネスロジックを実装し、リポジトリクラスはデータベースアクセスを実装することで、ビジネスロジックとデータアクセスの責務を明確に分離することができます。
- これにより、アプリケーションの保守性や拡張性が向上
- データアクセスの実装が変更された場合、ビジネスロジックに影響を与えずに変更ができるため、保守性が向上します。

### テストのしやすさ

- サービスクラスとリポジトリクラスを分けることで、ビジネスロジックとデータアクセスのテストを分離できます。
- ビジネスロジックのテストを実行する場合は、リポジトリクラスの代わりにスタブやモックを使用することができます。
- これにより、ビジネスロジックのテストが簡単になります。
- また、データアクセスのテストを実行する場合は、リポジトリクラスだけを対象にテストを実行することができます。

### コードの再利用性

- リポジトリクラスはデータベースアクセスの実装を行うため、複数のサービスクラスで同じリポジトリクラスを使用することができます。
- これにより、コードの再利用性が高まり、アプリケーションの保守性が向上します。

### 可読性の向上

- サービスクラスとリポジトリクラスを分けることにより、コードが明確に分かれるため、可読性が向上します。
- また、ビジネスロジックやデータアクセスの処理が複雑になっても、それぞれの責務が明確に分かれているため、コードを理解しやすくなります。

以上のようなメリットがあります。サービスクラスとリポジトリクラスを分けることで、アプリケーションの保守性や拡張性を高めることができます。

## リポジトリクラスを導入しない場合

### コントローラクラスの例:

```php
use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use App\Services\ProductService;

class ProductController extends Controller
{
    protected $productService;

    public function __construct(ProductService $productService)
    {
        $this->productService = $productService;
    }

    public function index(Request $request)
    {
        $products = $this->productService->searchProducts($request->input('search_query'));
        return view('products.index', compact('products'));
    }

    public function create()
    {
        return view('products.create');
    }

    public function store(Request $request)
    {
        $this->productService->createProduct($request->all());
        return redirect()->route('products.index');
    }

    public function show($id)
    {
        $product = $this->productService->getProduct($id);
        return view('products.show', compact('product'));
    }

    public function edit($id)
    {
        $product = $this->productService->getProduct($id);
        return view('products.edit', compact('product'));
    }

    public function update(Request $request, $id)
    {
        $this->productService->updateProduct($id, $request->all());
        return redirect()->route('products.show', $id);
    }
}
```

- `__construct()` メソッドで `ProductService` クラスのインスタンスを注入し、各メソッドで `$this->productService` 経由でメソッドを呼び出すことで、サービスクラスを利用しています。

### サービスクラスの例:

```php
<?php

namespace App\Services;

use App\Product;
use Illuminate\Support\Facades\DB;

class ProductService
{
    public function searchProducts($search)
    {
        return Product::query()
            ->when($search, function ($query, $search) {
                $query->where('name', 'like', "%{$search}%");
            })
            ->paginate(10);
    }

    public function createProduct($name, $price, $description)
    {
        return DB::transaction(function () use ($name, $price, $description) {
            $product = new Product();
            $product->name = $name;
            $product->price = $price;
            $product->description = $description;
            $product->save();
            return $product;
        });
    }

    public function updateProduct(Product $product, $name, $price, $description)
    {
        return DB::transaction(function () use ($product, $name, $price, $description) {
            $product->name = $name;
            $product->price = $price;
            $product->description = $description;
            $product->save();
            return $product;
        });
    }
}
```
