# Laravel コンポーネントについて

## 概要

- [公式-Laravel 9.x Blade テンプレート](https://readouble.com/laravel/9.x/ja/blade.html#components)
- [【Laravel】 blade のコンポーネントの使い方](https://progtext.net/programming/laravel-blade-component/)

## クラスコンポーネント

- 例：`php artisan make:component Alert`
- `make:component` コマンドで以下 2 ファイルを作成する
  - コンポーネントクラスを`app/View/Components`
  - ビューを`resources/views/components`
- サブディレクトリ内にコンポーネントを作成することもできます。
  - `php artisan make:component Forms/Input`
  - `app/View/Components/Forms`ディレクトリに`Input`コンポーネントを作成
  - ビューは`resources/views/components/forms`ディレクトリに配置します。

## 匿名コンポーネント

- Blade テンプレートのみでクラスを持たないコンポーネント
- `make:component` コマンドで`--view` オプションを使用
  - `php artisan make:component forms.input --view`
- `resources/views/components/forms/input.blade.php` へ Blade ファイルを作成
- このファイルは`<x-forms.input />`により、コンポーネントとしてレンダできます。

## 使い方

- コンポーネントを表示するために、Blade テンプレートの中で Blade コンポーネントタグを使用できます。
- Blade コンポーネントタグは、文字列`x-`で始まり、その後にコンポーネントクラスのケバブケース名を続けます。
  - 例：`<x-alert/>`
  - 例：`<x-user-profile/>`
- コンポーネントクラスが`app/View/Components`ディレクトリ内のより深い場所にネストしている場合は、`.`文字を使用してディレクトリのネストを表せます。
  - 例：`app/View/Components/Inputs/Button.php`の場合、
    - `<x-inputs.button/>`
