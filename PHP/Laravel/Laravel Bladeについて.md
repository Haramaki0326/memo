# Laravel Blade について

- [公式 Blade テンプレート](https://readouble.com/laravel/9.x/ja/blade.html)

## Blade ディレクティブ

- [公式 Blade ディレクティブ](https://readouble.com/laravel/9.x/ja/blade.html#blade-directives)

### `If文`

#### `@if`

```php
@if (count($records) === 1)
    １レコードあります。
@elseif (count($records) > 1)
    複数レコードあります。
@else
    レコードがありません。
@endif
```

#### `@unless`

```php
@unless (Auth::check())
    あなたはサインインしていません。
@endunless
```

#### `@isset`

```php
@isset($records)
    // $recordsが定義済みで、NULLではない…
@endisset
```

#### `@empty`

```php
@empty($records)
    // $recordsは「空」だ…
@endempty
```

## コンポーネント

- [公式 コンポーネント](https://readouble.com/laravel/9.x/ja/blade.html#:~:text=%E3%81%AB%E5%AD%98%E5%9C%A8%E3%81%97%E3%81%AA%E3%81%84%20%2D%2D%7D%7D-,%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88,-%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88%E3%81%A8%E3%82%B9%E3%83%AD%E3%83%83%E3%83%88)
