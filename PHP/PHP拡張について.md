# PHP 拡張

## 参考

- [拡張モジュールの一覧/分類](https://www.php.net/manual/ja/extensions.membership.php)

## 分類

- コア拡張
- バンドルされている拡張
- 外部拡張
- PECL 拡張

## Docker における拡張モジュールについて

### 参考

- [Docker Hub -How to install more PHP extensions-](https://hub.docker.com/_/php#:~:text=How%20to%20install%20more%20PHP%20extensions)
- 基本的にはここに全て書いている
- > 多くの拡張モジュールはすでにイメージにコンパイルされているので、 さらにコンパイルする前に `php -m` あるいは `php -i` の出力をチェックする価値があります。
  > PHP 拡張モジュールをより簡単にインストールするためのヘルパースクリプト `docker-php-ext-configure`,`docker-php-ext-install`, `docker-php-ext-enable` が用意されています。
  > イメージを小さくするために、PHP のソースは圧縮された tar ファイルで保管されます。PHP のソースを任意の拡張モジュールとリンクしやすくするために、ヘルパースクリプト `docker-php-source` も提供しており、tar ファイルの展開や展開したソースの削除が簡単にできます。注意: もし `docker-php-source` を使ってソースを展開した場合は、必ず docker イメージと同じレイヤーでソースを削除してください。

### デフォルトの拡張

いくつかの拡張モジュールは、デフォルトでコンパイルされています。これは、使用している PHP のバージョンに依存します。コンテナ内で `php -m` を実行すると、特定のバージョンの一覧が表示されます。

### コア拡張のインストール

例えば、PHP-FPM のイメージに gd 拡張を持たせたい場合、好きなベースイメージを継承し、以下のように独自の Dockerfile を記述することが可能です。

```docker
FROM php:7.4-fpm
RUN apt-get update && apt-get install -y \
		libfreetype6-dev \
		libjpeg62-turbo-dev \
		libpng-dev \
	&& docker-php-ext-configure gd --with-freetype --with-jpeg \
	&& docker-php-ext-install -j$(nproc) gd
```

拡張モジュールの依存関係を手動でインストールする必要があることを忘れないでください。拡張機能に独自の `configure` 引数が必要な場合は、この例のように `docker-php-ext-configure` スクリプトを使用します。この場合、`docker-php-source` を手動で実行する必要はありません。これは `configure` および `install` スクリプトで処理されるからです。

`docker-php-ext-install` の前にインストールする必要がある Debian や Alpine のパッケージがわからない場合は、[install-php-extensions プロジェクト](https://github.com/mlocati/docker-php-extension-installer)を見てください。このスクリプトは `docker-php-ext-\*` スクリプトをベースに構築されており、Debian (apt) や Alpine (apk) のパッケージを自動的に追加・削除して PHP 拡張機能のインストールを簡略化します。例えば、GD 拡張をインストールするには、単に `install-php-extensions gd` を実行すればよいのです。このツールはコミュニティメンバーによって提供されており、イメージには含まれていません。インストール、使用法、問題点については、彼らの Git リポジトリを参照してください。

Tianon がソフトウェアのビルド時に必要な依存関係を判断するために使っている手法については、 [コンパイル済みソフトウェアの Docker 化](https://ram.tianon.xyz/post/2017/12/26/dockerize-compiled-software.html) も参照してください (これは PHP 拡張モジュールのコンパイルにそのまま適用できます)。

### PECL 拡張のインストール

拡張モジュールの中には PHP のソースとともに提供されていないものもありますが、 その場合は PECL を通して提供されます。PECL 拡張モジュールをインストールするには、 `pecl install` でダウンロードとコンパイルを行い、 `docker-php-ext-enable` で有効化します。

```docker
FROM php:7.4-cli
RUN pecl install redis-5.1.1 \
	&& pecl install xdebug-2.8.1 \
	&& docker-php-ext-enable redis xdebug
```

```docker
FROM php:5.6-cli
RUN apt-get update && apt-get install -y libmemcached-dev zlib1g-dev \
	&& pecl install memcached-2.2.0 \
	&& docker-php-ext-enable memcached
```

PHP のバージョンとの互換性を確保するために、 `pecl install` の実行時にはバージョン番号を明示することを強く推奨します (PECL は、インストールする拡張モジュールのバージョンを選択する際には PHP のバージョンとの互換性をチェックしませんが、 インストールしようとする際にはチェックします)。
例えば、memcached-2.2.0 には PHP のバージョン制約がありませんが、 memcached-3.1.4 には PHP 7.0.0 以降が必要です。PHP 5.6 で pecl install memcached (バージョン指定なし) を行うと、PECL は最新リリースをインストールしようとし、失敗します。

互換性の問題だけでなく、依存関係が更新されるタイミングを確実に把握し、その更新を直接制御できるようにすることも良い方法です。
PHP のコア拡張モジュールとは異なり、 PECL 拡張モジュールは何か問題が発生したときに適切に対処できるように 直列にインストールする必要があります。そうしないと、エラーが発生したときに PECL がスキップしてしまいます。たとえば、

```sh
誤
pecl install memcached-3.1.4 && pecl install redis-5.1.1
```

の代わりに

```sh
正
 pecl install memcached-3.1.4 redis-5.1.1
```

でインストールします。
ただし

```docker
docker-php-ext-enable memcached redis
```

は、1 つのコマンドで全て済んでしまうので、問題ないです。
