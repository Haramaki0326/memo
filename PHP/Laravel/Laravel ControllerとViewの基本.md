# Controller と View の基本

## Controller の基本

## View の基本

- [Laravel 9.x ビュー](https://readouble.com/laravel/9.x/ja/views.html)

### `view`ヘルパ

- view ヘルパはルーティングや controller で view ファイルを表示させたい時に使用する
- メソッドに表示させたい`blade`ファイルの名前(`welcome.blade.php`なら`'welcome'`)を渡せば OK
- 引数に表示したい `blade` ファイルの`.blade`より前の名前の文字列を渡せば表示することができます。
- 表示させたい `blade` ファイルは `views` フォルダ(`resources/views`)に保存しなければいけません。
- 第一引数には**遷移先の `blade` ファイル名**
- 第二引数に**渡したい変数についての情報**を渡します。
  - 第二引数は`['渡す先での変数名' => 今回渡す変数]`
  - 渡す側では、渡す先での変数名の頭に`$`は必要ありません。
  - 渡される側（view ファイル）では`$`をつけないと変数と認識されないので注意

```php
// web.php で見かける例
Route::get('/', function () {
  return view('welcome');
})->name('welcome');
```

```php
// controller で見かける例
class TechController extends Controller
{
public function index ()
{
  return view('welcome');
}
}
```

```php
$tech = 'てくてくテック';
$technum = 15;

return view('welcometech', [
  'techtech' => $tech,
  'technum' => $technum,
]);
```

#### ネストしたビューディレクトリ構成

```php
view('hoge.huga.welcome');
```

- これはフォルダの階層を表しています。
- 正確には views フォルダからの、表示したいファイル位置を示しています。
- 例えば、上記`'hoge.huga.welcome'`の場合、`resource/views/hoge/huga/welcome.blade.php`というファイルが存在するのであれば, `welcome.blade.php` が表示されるようになります。
