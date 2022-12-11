# Laravel Request について

## 参考

- [公式](https://readouble.com/laravel/9.x/ja/requests.html)
- [[Laravel] Request クラスのメソッドまとめ - Qiita](https://qiita.com/gentuki/items/54d9bfd88f66208c1709)

## 概要

### リクエストへのアクセス

依存注入を使い、現在の HTTP リクエストのインスタンスを取得するには、

- ルートクロージャ
- コントローラメソッド

で `Illuminate\Http\Request` クラスをタイプヒントする必要がある

#### コントローラーメソッドで DI

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 新しいユーザーを保存
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');

        //
    }
}
```

#### ルーティングで DI

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    //
});
```

### 依存注入とルートパラメータ

コントローラメソッドがルートパラメータからの入力も期待している場合は、他の依存関係の後にルートパラメータをリストする必要があります。たとえば、ルートが次のように定義されているとしましょう。

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

以下のようにコントローラメソッドを定義することで、Illuminate\Http\Request をタイプヒントし、id ルートパラメーターにアクセスできます。

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 指定ユーザーを更新
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  string  $id
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, $id)
    {
        //
    }
}
```
