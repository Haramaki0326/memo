# PHP における静的解析-PHP_CodeSniffer(phpcs)-

- PHP_CodeSniffer(phpcs)
- PHPMD
- PHPStan
- Larastan
- PSalm

## 概要

- [公式 - GitHub](https://github.com/squizlabs/PHP_CodeSniffer)
- [PHP_CodeSniffer の使い方 - LAZE SOFTWARE](https://lazesoftware.com/blog/210905/)
- コーディング規約に沿って記述されているか検査
- コーディング規約に違反している個所を自動的に修正
- PHP_CodeSniffer は、2 つのコマンドで構成されている

## コマンドについて

### `phpcs` コマンド

- コーディング規約に沿って記述されているか検査
- `phpcs` というコマンド名は、"PHP Code Sniffer" の略

### `phpcbf` コマンド

- コーディング規約に違反している個所を、自動的に正しいコードに修正
- `phpcbf` というコマンド名は、"**PHP Code Beautifier and Fixer**" の略
- あくまで自動的に修正できる箇所のみ。それ以外の個所は手動で修正する必要がある

## インストール（`Composer`）

- `composer require --dev squizlabs/php_codesniffer`
- インストールできたか確認する
  - `phpcs` コマンドが使えるか確認する
    - `.\vendor\bin\phpcs -h`
  - `phpcbf` コマンドが使えるか確認する
    - `.\vendor\bin\phpcbf -h`

## 基本的な使い方

### 単一のファイルを検査する

- 1 つのファイルを検査する場合は、検査したいファイル名を指定します。
- 検査したいファイル名はスペース区切りで複数指定できます。

#### ".\src\hoge.php" ファイルを検査する場合

`.\vendor\bin\phpcs .\src\hoge.php`

#### ".\src\hoge.php" ファイルと ".\src\fuga.php" を検査する場合

`.\vendor\bin\phpcs .\src\hoge.php .\src\fuga.php`

### 複数のファイルを検査する

- 複数のファイルを検査する場合は、検査したいディレクトリ名を指定します。
- 検査したいディレクトリ名はスペース区切りで複数指定できます。

#### ".\src" ディレクトリを検査する場合

`.\vendor\bin\phpcs .\src`

#### .\src ディレクトリと .\tests ディレクトリを検査する場合

`.\vendor\bin\phpcs .\src .\tests`

## オプション

### WARNING を非表示にする

- PHP_CodeSniffer を実行した際に表示される指摘点には、その内容によって `ERROR` と `WARNING` の 2 種類がある
- 指摘点が大量に出る場合など、`ERROR` だけを表示して、`WARNING` を非表示にしたい場合は、`-n` オプションを付けます。
- `.\vendor\bin\phpcs -n .\src`

### 出力内容に色を付ける

- 色を付けるには `--colors` オプションを付けます。
- `.\vendor\bin\phpcs --colors .\src`

### 進捗状況を表示する

- 大量のファイルがある場合などでは、進捗状況が表示されていた方が便利かもしれません。
- その場合は、`-p` オプションを付けます。 `-p` オプションを付けるとコマンドの出力の最初に実行結果が表示されます。
- `.\vendor\bin\phpcs -p .\src`

## コーディング規約を変更する

- PHP_CodeSniffer で使用されるデフォルトのコーディング規約は、`PEAR` コーディング標準になっています。

### 使用できるコーディング規約の一覧を表示する

- PHP_CodeSniffer で使用できるコーディング規約の一覧を表示するには `-i` オプションを使用します。
- `.\vendor\bin\phpcs -i`

### コーディング規約を変更する

- コーディング規約を変更する場合は、`--standard` オプションを使用します。
- コーディング規約を PSR-12 に変更する場合
  - `.\vendor\bin\phpcs --standard=PSR12 .\src`

### 違反したルールの名前を表示する

- phpcs コマンドに、`-s` オプションを付けると、メッセージの後に違反したルールの名前を表示してくれます。
- どのルールが動作したのかが確認しやすくなりますので、独自のコーディング規約を使用する際には便利でしょう。

## Composer スクリプトとして登録する

- 毎回"`.\vendor\bin`" を記述するのは面倒
- Composer スクリプトでは、 "`.\vendor\bin`" に PATH が通っているので、"`.\vendor\bin`" を入力する必要はありません。

**composer.json**

```json
{
  "name": "hoge/hoge",
  "require-dev": {
    "squizlabs/php_codesniffer": "^3.6"
  },
  "scripts": {
    "cs-check": "phpcs --colors -p ./src ./tests",
    "cs-fix": "phpcbf --colors -p ./src ./tests"
  }
}
```

### 登録したスクリプト cs-check を実行する

`php composer.phar cs-check`

### 登録したスクリプト cs-fix を実行する

`php composer.phar cs-fix`

## Laravel Sail で利用する

- `composer.json`に phpcs 用意のスクリプトを登録する

```json
    "scripts": {
        "cs-check": "phpcs --standard=PSR12 --colors -p ./app/Http/Controllers ./app/Models",
        "cs-fix": "phpcbf --standard=PSR12 --colors -p ./app/Http/Controllers ./app/Models"
    },
```

- CLI で`sail`コマンドの`composer`を実行する。具体的には以下
  - `$ sail composer cs-check`
  - `$ sail composer cs-fix`
