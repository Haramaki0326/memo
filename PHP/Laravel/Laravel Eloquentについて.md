# DB の取り扱い方法 について

laravel ではデータベースを操作するにはいくつかの方法がります。

- `DB` クラスによる操作
- クエリビルダによる操作
- `Model` クラス（`Eloquent ORM`）による操作
  - `DB` クラスもクエリビルダも `Illuminate\Support\Facades\DB` クラス（ファサード）による記述方法
  - `Model` クラスは `Illuminate\Database\Eloquent\Model` クラスを継承するため仕組みが異なります。

## `DB` クラス

- [公式](https://readouble.com/laravel/9.x/ja/queries.html)
- [【laravel】データベースの操作：DB ファサード編](https://qiita.com/gone0021/items/f1e8223c94dbaab22fca)

a

## クエリビルダ

### 基本 `DB::table`

- `DB::table(テーブル名)`
  - 例：`$users = DB::table('users')->get();`
- 指定したテーブルのビルダクラス(`\Illuminate\Database\Eloquent\Builder`)を取得する
  - DB クラスのビルダークラス
    - `vendor/laravel/framework/src/Illuminate/Database/Query.php`
- `DB::table`でビルダクラスを用意し、ビルダの中に用意されている各メソッドを呼び出すことでテーブル操作などを行っていく。

### `get`メソッド

### `where`メソッド

#### 参考

[参考:【Laravel】Query Builder(クエリビルダー) – 各種 where 句の使い方](https://public-constructor.com/laravel-query-builder-where-clauses/)

`where()`は 3 つの引数を受け取る

#### 基本

- 第一引数がカラム名
- 第二引数が演算子、
- 第三引数が第一引数のカラム名に対する値

```php
$this->post->where('is_public', '=', '1')
           ->get();
```

AND 条件を追加する場合は更に where()を呼び出します。

```php
$this->post->where('title', '=', 'foobar')
           ->where('content', 'like', '%bar%')
           ->get();
```

OR 条件を追加する場合は orWhere()を呼び出します。

```php
$this->post->where('title', 'like', '%foo%')
           ->orWhere('title', 'like', '%builder%')
           ->get();
```

#### `whereIn`メソッド

#### `whereBetween`メソッド

#### 日付に関する`where`メソッド

#### `whereColumn`メソッド

#### 複数の条件に括弧をつけてまとめる方法

#### `whereRaw`メソッド

### 条件にあったレコードの特定カラムを取得する

- `get()`で 1 レコード分のオブジェクトを取得してから、`$item["col"]`とかで取得できる・・・？

### `insert`メソッド

- サンプル

```php
DB::table('users')->insert([
    'email' => 'kayla@example.com',
    'votes' => 0
]);
```

```php
$param = [
    'email' => 'kayla@example.com',
    'votes' => 0
];

DB::table('users')->insert($param);

```

- 二重の配列を渡すことにより、一度に複数のレコードを挿入できます。
- 各配列は、テーブルに挿入する必要のあるレコードを表します。

```php
DB::table('users')->insert([
    ['email' => 'picard@example.com', 'votes' => 0],
    ['email' => 'janeway@example.com', 'votes' => 0],
]);
```

### `update`メソッド

`DB::table(テーブル)->where(更新対象の指定)->update( 配列 )`

```php
$param = [
    'email' => 'kayla@example.com',
    'votes' => 0
];

DB::table('users')
  ->where('id', $request->id)
  ->update($param);

```

### `delete`メソッド

`DB::table(テーブル)->where(更新対象の指定)->delete()`

```php
DB::table('users')
  ->where('id', $request->id)
  ->delete();
```

## `Eloquent ORM`(`Model` クラス)

- [公式](https://readouble.com/laravel/9.x/ja/eloquent.html)
- [【laravel】データベースの操作：Eloquent ORM 編](https://qiita.com/gone0021/items/951cd63a7e591e18cd2a)

### 基本

#### 全件取得(`all()`)

```php
$items = Person::all();
```

- 戻り値は`\Illuminate\Database\Eloquent`名前空間の`Collection`クラスのインスタンス
- クエリビルダとの違いは、コレクションにまとめられているのは配列ではなく **モデルクラスのインスタンス**（1 つ 1 つは`Person`クラスのインスタンス）
- したがって、**プロパティやメソッドを利用することができる**。

##### 活用例

```php
public function getData(){
  return $this->id . ':' . $this->name;
}
```

```php
@section('content')
  <table>
  <tr><th>Data<th><tr>
  @foreach ($items as $item)
  <tr>
    <td>{{$item->getData())</td>
  </tr>
  @endforeach
  </table>
@endsection
```

- **DB クラスのようにただのオブジェクトや配列ではこんな具合に拡張はできない。**

#### ID による検索（`find()`）

`モデルクラス::find(整数)`

- 引数には検索する ID を指定する

### 検索とスコープ

- `where()`メソッドの戻り値
  - `DB` クラスの`where()`メソッドでは`Illuminate/Database/Query`名前空間のビルダクラスのインスタンス
    - `vendor\laravel\framework\src\Illuminate\Database\Query\Builder.php`
  - `Eloquent`クラスの`where()`メソッドでは`Illuminate/Database/Eloquent`名前空間のビルダクラスのインスタンス
    - `vendor\laravel\framework\src\Illuminate\Database\Eloquent\Builder.php`
- `where()`を使えば複数条件を組み合わせてデータを取得することができるが非常に分かりにくくなる
- そのためモデルに予めスコープ（単一条件のモデル）を決めておくと楽

#### ローカルスコープ

#### グローバルスコープ

#### `where()`

### モデルの保存・更新・削除

#### モデルの保存(`save()`)

##### 保存処理の流れ

- バリデーションの実行(`$this->validate($request, Person::$rules)`)
- モデルインスタンスの作成（`$person = new Person;`）
- 値を用意する

```php
$form = $request->all();
unset($form['_token']);
```

- インスタンスに値を設定して保存
  - `$person->fill($form)->save();`
  - `fill()`メソッドは引数の配列のモデルのプロパティに代入する
    - 1 つ 1 つの値をインスタンスに設定しても同じ

#### モデルの更新(`save()`)

- 基本的な流れは登録時と同じ
- `new`ではなく`find()`で更新対象のモデルを指定する
- 更新時でも`update()`ではなく登録と同じ`save()`を利用する。

```php
$this->validate($request, Person::$rules)
$person = Person::find($request->id);
$form = $request->all();
unset($form['_token']);
$person->fill($form)->save();
```

#### モデルの削除(`delete()`)

```php
Person::delete($request->id);
```

### モデルのリレーション

### `created_at`,`updated_at`について

- `created_at`,`updated_at`は自動設定
  - `created_at`は作成された日時
  - `updated_at`は最後に更新された日時
- 値を自動設定しない場合はモデルに以下のプロパティを設定する
  - `protected $timestamps = false;`
- `created_at`,`updated_at`以外のカラムに作成・更新日時を保存したい場合は次のフィールドを用意する。
  - `const CREATED_AT = 作成日時フィールド名;`
  - `const UPDATED_AT = 更新日時フィールド名;`

## クエリビルダと`Eloqunet`の違い

### 参考

- [Laravel のクエリビルダと Eloquent の違いとは？](https://cloudsmith.co.jp/blog/backend/laravel/2021/06/1817294.html)

### 違い

- DB から取得したデータを、クエリビルダはただの配列（コレクション）を扱い、Eloquent（ORM）は PHP のクラスとして扱う
  - クエリビルダ…戻り値がコレクション（`Collection`）
  - Eloquent…戻り値が`Model`オブジェクト

## Eloquent/クエリビルダーの主なメソッド

- [公式：Laravel 8.x データベース：クエリビルダ](https://readouble.com/laravel/8.x/ja/queries.html)
- [公式：Laravel 8.x Eloquent の準備](https://readouble.com/laravel/8.x/ja/eloquent.html)
- [参考：【Laravel】Eloquent/クエリビルダーの主なメソッドまとめ](https://maasaablog.com/development/laravel/1114/)
- Eloquent ビルダークラス
  - `vendor/laravel/framework/src/Illuminate/Database/EloquentBuilder.php`
- Eloquent の Model クラス
  - `vendor/laravel/framework/src/Illuminate/Database/EloquentModel.php `
- Laravel で使用可能な Eloquent/クエリビルダーの主なメソッドと特徴およびその返り値(戻り値)

| メソッド名 | 処理内容                                                                           | 返り値(戻り値)                                                       |
| ---------- | ---------------------------------------------------------------------------------- | -------------------------------------------------------------------- | --------------------------------- |
| find       | 主キーで単一のレコードを取得                                                       | Model のオブジェクト                                                 |
| first      | 単一のレコードを取得                                                               | Model のオブジェクト                                                 |
| get        | 複数のレコードを取得                                                               | Model のオブジェクトの Collection                                    |
| all        | 全レコードを取得                                                                   | Model クラスからの呼び出しなら Collection<br> Builder クラスなら配列 |
| with       | リレーション                                                                       | Builder クラスのオブジェクト                                         |
| where      | クエリによる条件付与                                                               | Builder クラスのオブジェクト                                         |
| orderBy    | 並び順を指定<br>`get`メソッドの直前に書くこと。`where`なども利用する場合はその後。 | Builder クラスのオブジェクト                                         |
| offset     | 指定した位置からレコードを取得<br>(ページネーションで利用することが多い)           | Builder クラスのオブジェクト                                         |
| limit      | 指定した数レコードを取得                                                           | Builder クラスのオブジェクト                                         |
| insert     | クエリビルダのみ？                                                                 | Model のオブジェクト                                                 |
| create     | `Eloquent`のみ？<br>新規追加<br>※Model に fillable(ないし guarded)の設定が必要     | Model のオブジェクト                                                 |
| save       | 新規追加(更新)                                                                     | bool 値                                                              |
| update     | 更新                                                                               | Model クラスの update は、bool 値。                                  | Eloquent ビルダーの update は int |
| delete     | 削除                                                                               | Model クラスなら、bool 値。Eloquent ビルダーなら処理したレコード数。 |
| destroy    | 削除                                                                               | 処理したレコード数                                                   |
