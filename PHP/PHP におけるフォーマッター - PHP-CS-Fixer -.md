# PHP におけるフォーマッター - PHP-CS-Fixer -

## URL

- [公式 - PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer)
- [参考 - ソースコードの“赤ペン先生”PHP-CS-Fixer のインストールと設定 - Qiita](https://qiita.com/suin/items/4242aec018d086312fe7)

## 概要

-

## インストール

- `composer require --dev friendsofphp/php-cs-fixer`

## 使い方

- [公式](https://github.com/FriendsOfPHP/PHP-CS-Fixer/blob/master/doc/usage.rst)

### 整形予定を確認する`--dry-run`

- `--dry-run`オプションを付けると整形無しに変更予定のファイル一覧を見ることができる
- 以下の例では、7 つのファイルが変更予定だ。

```sh
❯ ./vendor/bin/php-cs-fixer fix --dry-run ./src
Loaded config default.
Using cache file ".php_cs.cache".
   1) src//Json.php
   2) src//Json/EncodingContext.php
   3) src//Json/DecodingException.php
   4) src//Json/Encoder.php
   5) src//Json/DecodingContext.php
   6) src//Json/EncodingException.php
   7) src//Json/Decoder.php

Checked all files in 0.725 seconds, 10.000 MB memory used
```

### 差分確認 `--diff`

- `--dry-run` オプションに加えて`--diff` を加える。
- 更にお好みで差分の表示形式指定`--diff-format udiff` を加えても良い。
  - `./vendor/bin/php-cs-fixer fix --dry-run --diff --diff-format udiff ./src`

## 設定ファイル

設定ファイル`.php-cs-fixer.dist.php`内を以下のように記述します。 ルールはここで確認することができ、PSR12 等複数のルールをまとめたルールセットを指定することも可能です。

```php
<?php
/*
 * This document has been generated with
 * https://mlocati.github.io/php-cs-fixer-configurator/#version:3.1.0|configurator
 * you can change this configuration by importing this file.
 */
$config = new PhpCsFixer\Config();
return $config
    ->setRules([
        '@PSR12' => true,
    ])
    ->setFinder(PhpCsFixer\Finder::create()
        ->exclude('vendor')
        ->in(__DIR__)
    )
;
```

- `setRiskyAllowed()` : risky なルール(コードを書き換える破壊的な修正を含むルール)を有効にするかを設定します。
- `setRules()` : 使用したいルールをここで列挙します。
- `setFinder()`...->exclude(['']) : 除外したいディレクトリ名を列挙することでそのディレクトリ内のファイルは無視されます。

### ルール設定方法

ルール生成用のツールで生成ファイルを作ることができます。
https://mlocati.github.io/php-cs-fixer-configurator/#version:3.1
