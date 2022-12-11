# Laravel バリデーションについて

## 参考

- [公式:Laravel 9.x バリデーション](https://readouble.com/laravel/9.x/ja/validation.html)
- [使用可能なバリデーションルール](https://readouble.com/laravel/9.x/ja/validation.html#available-validation-rules)

## 概要

- `$request->validate([検証設定の配列]);`
- `@errors`変数
  - 条件分岐`@if`などで制御しながらテンプレートを書く必要があり面倒なので、通常は後述の`@error`ディレクティブを利用する

```php
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Postフォームの作成 -->
```

- `@error`ディレクティブ

```php
@error
    $message でメッセージを表示
@enderror
```

例文

```php
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

## バリデーションルール

### 書き方

### 種類

- [使用可能なバリデーションルール](https://readouble.com/laravel/9.x/ja/validation.html#available-validation-rules)

## カスタムバリデーション

### フォームリクエスト

### カスタムメッセージ

### バリデータ(`Validator`)
