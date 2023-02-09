# Laravel Pint

### [`Laravel Pint`](https://readouble.com/laravel/9.x/ja/pint.html)

#### 概要

- **まだバグがありそうなので導入は見送る**
- Pint は PHP-CS-Fixer 上に構築されている
- Pint は、すべての新しい Laravel アプリケーションに自動的にインストールされている
- デフォルトで Pint は設定を必要とせず、Laravel の主張を取り入れたコーディングスタイルに従い、コードスタイルの問題を修正します。

#### 実行

- コマンド
  - `./vendor/bin/pint`
- 特定のファイルやディレクトリに対して
  - `./vendor/bin/pint app/Models`
  - `./vendor/bin/pint app/Models/User.php`
- オプション
  - `-v`
  - `-vv`
  - `--test`
  - `--dirty`

#### 設定

- 前述したように、Pint は設定を一切必要としません。
- しかし、プリセットやルール、インスペクトフォルダをカスタマイズしたい場合は、プロジェクトのルートディレクトリに、`pint.json` ファイルを作成する

```json
{
  "preset": "laravel"
}
```

また、特定のディレクトリにある pint.json を利用したい場合は、Pint を起動する際に`--config` オプションを指定してください。

```sh
pint --config vendor/my-company/coding-style/pint.json
```

#### プリセット

- Pint が現在サポートしているプリセットは、`laravel`、`psr12`、`symfony` です。
- プリセットは、コード内のスタイルの問題を修正するために使用するルールセットを定義しています。
- デフォルトで `Pint` は、`laravel` プリセットを使用します。
  - これは、Laravel の意見に基づいたコーディングスタイル
- Pint に`--preset` オプションを指定することで、別のプリセットも指定できます。

```sh
pint --preset psr12
```

お望みならば、プロジェクトの pint.json ファイルにプリセットを設定できます。

```json
{
  "preset": "psr12"
}
```
