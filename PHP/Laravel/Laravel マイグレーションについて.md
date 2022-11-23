# Laravel マイグレーションについて

- [公式：Laravel 9.x マイグレーション](https://readouble.com/laravel/9.x/ja/migrations.html)

## マイグレーション実施

- [(Laravel) マイグレーション実施手順](https://hara-chan.com/it/programming/laravel-db-migration/)

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
