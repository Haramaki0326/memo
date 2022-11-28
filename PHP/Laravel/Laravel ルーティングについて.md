# Laravel ルーティングについて

## 参考

- [公式](https://readouble.com/laravel/9.x/ja/routing.html)
- [Laravel ルーティングの基本とよく使われるルーティングパターン](https://www.ritolab.com/posts/119)

## 概要

- ルーティングファイル
- ルート定義メソッド
- クロージャによるルーティング
- コントローラへのルーティング
- ビューへのルーティング
- リダイレクト
- 名前付きルート
- ミドルウェア
- ルーティングのグループ化

## ルーティングファイル

### `web.php`

Laravel で通常のブラウザからの HTTP リクエストに対するルーティング設定を行う場合は `laravel/routes/web.php` で行います。

```php
//laravel/routes/web.php
<?php

Route::get('/', function () {
    return view('welcome');
});
```

デフォルトでは１つのルーティングのみが設定されています。このルーティングは、Laravel のウェルカムページ（インストール後にブラウザからアクセスして表示されるページ）のルーティングになります。

### `api.php`

また一方で、API 通信などをルーティングする際は `laravel/routes/api.php` で行います。

### `web.php`と`api.php`の違い

両者の最も大きな違いは、適用されるミドルウェアが違う事

- `web.php` は、主にクライアント操作で起こるリクエスト（リンクを押下して画面遷移する・フォームを送信するなど）でのルーティングを行う
  - CSRF 対策やセッション関係などのミドルウェアがデフォルトでは適用されます
- `api.php` の方はもっとミニマムなミドルウェアが適用されます。

## ルート定義メソッド

ルーティングには `Route` ファサードを使いますが、HTTP リクエストに対して定義できるメソッドの基本項目は以下になります。

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

### `Route::get($uri, $callback);`

GET リクエストに対してのルーティングを定義します。主に通常の URL 遷移やフォームの GET リクエストに対して用います。

### `Route::post($uri, $callback);`

POST リクエストに対してのルーティングを定義します。主にフォームの POST リクエストに対して用いられます。

### `Route::put($uri, $callback);`

put リクエストに対してのルーティングを定義します。

### `Route::patch($uri, $callback);`

patch リクエストに対してのルーティングを定義します。

### `Route::delete($uri, $callback);`

delete リクエストに対してのルーティングを定義します。

### `Route::options($uri, $callback);`

options リクエストに対してのルーティングを定義します。

- `put/patch/delete/options` メソッドに関しては REST API などを作成する時に使われたりしますが、**基本的に使用するのは `GET` と `POST` メソッドがほとんどです。**
- また、`post()` `put()` `delete()` メソッドを用いる際は、送信元から **CSRF トークン**フィールドを渡す必要があります。

## クロージャによるルーティング

ルーティングメソッドの最も基本的なものは、コールバックをクロージャで定義する事です。

```php
Route::get('sample/route', function () {
    return 'PHP Framework Laravel Routing!!';
});
```

上記のルーティングは、間にコントローラもビューも挟まずに、ただ文字列だけを返す、最もシンプルな形になります。
例えばちょっとした処理を確認したりする場合にはこういった書き方をしてサクッと動作を確認したりに使う事も出来ます。

```php
Route::get('sample/route/1', function () {
    $result = 5 * 20;
    return $result;
});
```

例えば上記のような場合は、単純に掛け算を行ってそれを表示します。ブラウザからアクセスすると、「100 」と表示されます。

## DI（依存注入）

ルートのコールバックの引数にタイプヒントにより、そのルートで必要な依存関係を指定できます。宣言した依存関係は、Laravel サービスコンテナにより、自動的に解決されコールバックへ注入されます。たとえば、`Illuminate\Http\Request`クラスのタイプヒントを指定して、現在の HTTP リクエストをルートコールバックへ自動的に注入できます。

```php
use Illuminate\Http\Request;

Route::get('/users', function (Request $request) {
    // ...
});
```

## コントローラへのルーティング

ブラウザからどの URL にアクセスした時に、どのコントローラへ処理を渡すかを設定します。コントローラへのルーティングは、以下のように記述します。

```php
//laravel/routes/web.php
Route::get('sample', 'SampleController@index');
```

この場合は、ブラウザから `http://YOUR-DOMAIN/sample` `にアクセスすると、SampleController` の `index()` メソッドへ処理を渡す。という事になります。

例えば以下の場合では

```php
Route::get('sample/members/1', 'SampleController@member');
```

ブラウザから `http://YOUR-DOMAIN/sample/members/1` にアクセスすると、`SampleController` の `member()` メソッドへ処理を渡す。という事になります。

### URL からパラメータを取得する

URL からコントローラ側へパラメータを渡したい時は以下のように記述します。

```php
//laravel/routes/web.php
Route::get('sample/members/{member_id}', 'SampleController@member');
```

例えば上記のように `{member_id}` と記述すると、コントローラ側で変数 `$member_id` としてパラメータを取得する事が出来ます。コントローラ側では以下のようにして取得できます。

```php
//laravel/app/Http/Controllers/SampleController.php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class SampleController extends Controller
{
    public function member($member_id)
    {
        print_r($member_id);
    }
}
```

もちろん、これはいくつでも設定できます。

```php
Route::get('sample/members/{member_id}/group/{group}/type/{type}', 'SampleController@member');
```

コントローラでは以下のようにして取得できます。

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class SampleController extends Controller
{
    public function member($member_id, $group, $type)
    {
        $params = [
            'member_id' => $member_id,
            'group'     => $group,
            'type'      => $type,
        ];

        print_r($params);
            => Array
                (
                    [member_id] => 12
                    [group] => 1-2
                    [type] => student
                )
    }
}
```

尚、上記の設定の場合はパラメータ部分は必ず渡されなければなりません。例えば、`member_id` のみを渡すルーティングの場合で、`http://YOUR-DOMAIN/sample/members/` の場合は例外（`404`）がスローされます。

#### パラメータなしを許容する

パラメータ部分について、入力を任意のものとしたい（パラメータが無くても許容する）場合は以下のように記述します。

```php
Route::get('sample/members/{member_id?}', 'SampleController@member');
```

`{member_id?}` とする（末尾にクエスチョンマーク「`?`」を付ける）事で、パラメータが無くても許容されます。

もちろん、この場合はコントローラ側でパラメータが無い場合の処理も漏れずに記述しましょう。

#### パラメータの値に制約を設ける

URL からパラメータを受け取るとして、その全てが許容されていると、入力値の評価をいちいちコントローラ側で行わなくてはならなくなります。 URL は誰にでも変更できるものなので攻撃の対象にもなりやすく、受け取るパラメータは出来るだけコントローラに渡す前に評価しておきたいものです。

そんな場合は以下のようにする事で、コントローラへ渡すパラメータ値をルーティングの時点で評価する（制限を設ける）事が出来ます。

```php
Route::get('sample/members/{member_id}', 'SampleController@member')
     ->where('member_id', '[0-9]+');
```

この場合は、パラメータとしてコントローラに渡される `member_id` は、数字の連続のみ受け付けるという制約が設定されます。それ以外、例えば文字列などが渡された場合は例外（404）がスローされます。

複数のパラメータに対して制約を設ける場合は、配列で渡します。

```php
Route::get('sample/members/{member_id}/group/{group}/type/{type}', 'SampleController@member')
     ->where([
         'member_id' => '[0-9]+',
         'group'     => '[A-Za-z]+',
         'type'      => '[0-9]{2}'
     ]);
```

また、たくさんの制限を設ける必要がある場合に、その全てをルーティングに記述するとルーティングファイルが煩雑になります。それを好まない場合は、ルーティングのサービスプロパイダに制約を定義する事が出来ます。

```php
//laravel/app/Providers/RouteServiceProvider.php
public function boot()
{
    Route::pattern('member_id', '[0-9]+');
    Route::pattern('group', '[A-Za-z]+');
    Route::pattern('type', '[0-9]{2}');

    parent::boot();
}
```

`boot()` メソッド内にて、`Route` ファサードの `pattern()` メソッドでそれぞれの制約を定義します。

こうする事で、ルーティング時に記述したパラメータ指定に関しての全てにこのルールが適用される（{id}と書くと id プロパティの制約が適用される）ので、ルーティングにいちいち制約を記述しなくて良くなります。

## ビューへのルーティング

コントローラではなくビューへルーティングを行いたい場合は `view()` メソッドで設定します。

```php
Route::view('sample/view', 'sampleview');
```

上記の場合、コントローラは経由せずに、直接ビューへのルーティングが行われます。

ビューへ値を渡したい場合は、第二引数へ配列で渡します。

```php
Route::view('sample/view', 'sampleview', ['number' => 123456789]);
```

## リダイレクト

ルーティング内でリダイレクトを行う場合は、`redirect()` メソッドを用いて以下のように記述します。

```php
Route::redirect('/sample/from', '/sample/to', 301);
```

上記の場合、`http://YOUR-DOMAIN/sample/from` へのアクセスに対して `http://YOUR-DOMAIN/sample/to` へ 301 リダイレクトを行います。

## 名前付きルート

ルーティングに名前を付ける事ができます。名前をつけると、設定したルート名のみを指定し処理を書けば良くなるので便利です。

```php
Route::get('sample', 'SampleController@index')->name('sample');
```

リダイレクト
例えば以下のようなルーティングがあったとします。

```php
Route::get('sample/from', 'SampleController@from');
Route::get('sample/to', 'SampleController@to')->name('to');
```

to() メソッドへのルーティングに対してルート名「to 」を設定しています。

その場合、コントローラ側ではリダイレクトを以下のように記述できます。

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class SampleController extends Controller
{
    public function from()
    {
        // ルート名「to」へリダイレクト
        return redirect()->route('to');
    }

    public function to()
    {
        print_r('リダイレクト');
    }
}
```

この例では、from から to へ名前付きルートでリダイレクトが行われます。

### URL の生成

ルート名を使って URL を作成できます。

```php
public function from()
{
    $url = route('to');
    print_r($url);
    => http://YOUR-DOMAIN/sample/to
}
```

## ミドルウェア

あるルーティングに対して何らかのミドルウェアを適用させる場合は、以下のように記述します。

```php
Route::get('sample', 'SampleController@index')->middleware('html.minify');
```

上記は、指定のルーティングに対して、HTML を minify するミドルウェアを指定している例になります。

## ルーティングのグループ化

ルーティングの際にミドルウェアを適用させたいページがあるとして、それらが複数ページに及ぶ場合（同じミドルウェアを複数のルーティングに挿したい）があります。その時は、それらのルーティングをグループ化し、ミドルウェアを指定する事で、設定をまとめて行えます。

```php
Route::group(['middleware' => 'basic_auth', 'prefix' => ''], function() {
    Route::get('/login', 'LoginController@index')->name('login');
    Route::get('/members/list', 'MemberController@index')->name('member');
    Route::get('/members/add', 'MemberController@add');
    Route::get('/members/edit', 'MemberController@edit');
});
```

上記は、ログインページと、ログイン認証を必要とするページ全てに、Basic 認証をかけるミドルウェアを適用させる場合の記述例です。

このように、group() メソッドを用いてミドルウェアを指定し、対象のルーティングをコールバックでラップする事で設定を行う事が出来ます。

```php
Route::middleware('basic_auth')->group(function () {
    Route::get('/login', 'LoginController@index')->name('login');
    Route::get('/members/list', 'MemberController@index')->name('member');
    Route::get('/members/add', 'MemberController@add');
    Route::get('/members/edit', 'MemberController@edit');
});
```

こちらはミドルウェアの設定先に対してグループを指定しているので、前者と後者の例は指定が逆になっているだけで、同じ意味になります。
