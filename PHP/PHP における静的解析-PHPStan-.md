# PHP における静的解析-`PHPStan`,`Larastan`-

- PHP_CodeSniffer(phpcs)
- PHPMD
- PHPStan
- Larastan
- PSalm

## URL

- [公式 - PHPStan](https://phpstan.org/)
- [公式 - Larastan](https://github.com/nunomaduro/larastan)
- [参考 - level=0 から始める PHPStan(Larastan) 導入ガイド - Shin x Blog](https://blog.shin1x1.com/entry/getting-stated-with-phpstan)

## 概要

- PHPStan は、Ondřej Mirtes（@OndrejMirtes）さんが開発している PHP コードの静的解析ツール
- MIT ライセンスで公開されています。
- PHP で実装されており、Composer でインストールして利用できます。
- Docker イメージも公式で公開されているので、こちらを利用すれば Composer パッケージとしてインストールせずに利用することも可能
- PHPStan には多くのコード品質をチェックするルールが実装されており、これをアプリケーションコードに適用することでバグの要因となりそうな箇所を検出します。
- これによりアプリケーションを実行した時にはじめて気づくような問題を未然に察知できます。
- 具体的には下記のような問題を検知できます。
  - 存在しないクラスをインスタンス化している。
  - 存在しないメソッド、関数を呼び出している。
  - 参照しているクラス名の大文字小文字が定義と異なる。
  - メソッド仮引数とメソッド呼び出し実引数の型が一致しない。
  - メソッドに型宣言が指定されていない。
- Larastan は、PHPStan の拡張の一つで Laravel アプリケーションで型情報などを PHPStan に認識させるための設定が含まれています。
- Larastan を実行するには、PHPStan のコマンドを利用するので、実行方法は同様になります。

## Playground

- PHPStan は公式サイトに Playground が用意されており、ブラウザで PHP コードを入力すると PHPStan が実行され、結果が表示されます。
- 適用する level も選択でき、permlink も発行できるので、試しに実行してみたり、実行結果を共有するなどに利用できます

Playground | PHPStan

## インストール

### PHPStan

- PHPStan は Composer パッケージとして公開されているので、composer コマンドでインストールできます
  - `$ composer require --dev phpstan/phpstan`
- PHPStan は、PHP 7.1 以上で動作します。`vendor/bin/phpstan` が実行コマンドになります。
  - `$ ./vendor/bin/phpstan -V`

### Larastan

- Larastan は依存として PHPStan をインストールしますので、上記の phpstan コマンドも合わせてインストールされます。
  - `$ composer require --dev nunomaduro/larastan`
- Larastan は PHP 7.2 以上かつ Laravel 6 以上が対象となります。
- こちらもインストールが完了すると vendor/bin/phpstan コマンドが利用可能になります。
  - `$ ./vendor/bin/phpstan -V`

## 設定ファイルの作成

- PHPStan の設定ファイルは NEON という YAML に似たファイル形式で、 `phpstan.neon` というファイル名で作成します
  - [参考：Config Reference | PHPStan](https://phpstan.org/config-reference)

```neon
parameters:
  paths:
    - .
  level: 0
```

- `paths` では、検出対象のコードディレクトリを指定します。ここではカレントディレクトリを指定しています。
- `level` では、適用する level を指定します。ここでは、0 （最も緩い）となっています。
- Larastan を利用する際は、下記のように Larastan の設定ファイルを読み込みようにします。これは Laravel プロジェクトを想定しているので、paths では Laravel アプリケーションディレクトリを指定しています。

```neon
includes:
    - ./vendor/nunomaduro/larastan/extension.neon

parameters:
    paths:
        - app
        - bootstrap
        - config
        - database
        - resources/views
        - routes
    level: 0
```

## PHPStan の実行

- PHPStan を実行するには `phpstan` コマンドを利用します。
- `analyse` サブコマンドを指定すると設定ファイルの内容に従って静的解析を行い、エラーを出力します（`analyse` は省略可能）。
- `$ ./vendor/bin/phpstan analyse`
- なお、Laravel アプリケーションのように多くのパッケージを利用する場合、PHP のメモリ制限で実行が止まる場合があります。こういった場合は、`--memory-limit` オプションで利用するメモリ制限を緩和すると良いです。
- メモリ制限を 1G にする
  - `$ ./vendor/bin/phpstan analyse --memory-limit=1G`
