# Laravel バリデーションについて

## 参考

- [公式:Laravel 9.x バリデーション](https://readouble.com/laravel/9.x/ja/validation.html)
- [使用可能なバリデーションルール](https://readouble.com/laravel/9.x/ja/validation.html#available-validation-rules)

## 概要

- `validate`メソッド
  - `Illuminate\Http\Request`オブジェクトのメソッド
- バリデーションルールにパスすると、コードは正常に実行され続けます
- バリデーションに失敗すると例外が投げられ、適切なエラーレスポンスが自動的にユーザーに返送されます。
  - `Illuminate\Validation\ValidationException`
  - HTTP リクエストの場合、バリデーションが失敗した場合、直前の URL への**リダイレクト**レスポンスが生成されます。
  - 受信リクエストが XHR リクエストの場合、バリデーションエラーメッセージを含む **JSON** レスポンスが返されます。

```php
/**
 * 新ブログポストの保存
 *
 * @param  \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // ブログポストは有効
}
```

## エラー表示

- バリデーションに失敗した場合、直前の場所へ自動的にリダイレクトを行う
- すべてのバリデーションエラーとリクエスト入力は自動的にセッションに一時保持されます。
- `$errors`変数は、ミドルウェアにより、アプリケーションのすべてのビューで共有されます。
  - `$errors` 変数は `Illuminate\Support\MessageBag` のインスタンスです。このオブジェクトの操作の詳細は、ドキュメントを確認してください。
    - https://readouble.com/laravel/9.x/ja/validation.html#working-with-error-messages

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

## エラーメッセージのカスタマイズ

Laravel の組み込みバリデーションルールは、それぞれエラーメッセージを持っており、アプリケーションの`lang/ja/validation.php`ファイルに格納しています。このファイルに各バリデーションルールの翻訳エントリーがあります。アプリケーションに合うように、これらメッセージを自由に変更・修正してください。
