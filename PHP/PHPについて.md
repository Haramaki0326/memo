# プロセス、スレッド、CPU コア

- [参考：【図解】CPU のコアとスレッドとプロセスの違い,コンテキストスイッチ,マルチスレッディングについて](https://milestone-of-se.nesuke.com/sv-basic/architecture/cpu/)
- **プロセス**とは、実行中のプログラムのことです。
  1 つのプロセスには、1 つのメモリ空間 (メモリ領域) が割り当てられます。メモリ空間はプロセスから OS に要求すれば (空きがあれば) 増やしてくれます。
- **スレッド**とは、プロセス内で命令を逐次実行する部分であり、**CPU コア**を利用する単位のことです。
- ![](https://milestone-of-se.nesuke.com/wp-content/uploads/2016/12/cpu-1.png.webp)
- ![](https://milestone-of-se.nesuke.com/wp-content/uploads/2016/12/cpu-2.png.webp)

# PHP について

## モジュール版と CGI 版

- [参考：nginx と PHP-FPM の仕組みをちゃんと理解しながら PHP の実行環境を構築する](https://qiita.com/kotarella1110/items/634f6fafeb33ae0f51dc)

### モジュール版

- Web サーバーのプロセスのなかで PHP を実行してしまう方法

#### メリット

モジュール版のメリットとして、Web サーバーのプロセスで PHP が実行されるため、CGI に比べて動作速度が高速になるという点があります。

#### デメリット

ただ PHP が Web サーバーを動かすユーザー（root 権限など）で動作するため、ユーザーが複数いる共用サーバーではセキュリティ面に不安があります。そこでモジュール版には「セーフモード」という設定があり、ユーザー間のファイル干渉を防止できるようなっています。

#### 例

- Apache の mod_php
  - Apache のプロセス内部で PHP を実行できるようになる（別プロセスを立ち上げる必要がない）
- IIS の ISAPI

### CGI 版

#### メリット

メリットとしては、まずセキュリティ面が挙げられます。CGI（版の PHP）を動かす各ユーザーは、Web サーバー本体を動かすユーザーとは異なります（切り離されています）。そのため誤って他ユーザーに干渉してしまうといった危険がありません。

#### デメリット

一方デメリットとしては、Web サーバーとは別個のプロセスとして動かすぶん、実行するたびにメモリのロードが必要となり、動作速度がモジュール版に比べて遅くなります。

#### FastCGI

FastCGI は、初回リクエスト時に起動したプロセスをメモリ上へ保持を行い、次回リクエストに対してはそのメモリに保持したプロセスの実行を行います。 CGI の問題を解決し、プログラム動作速度の向上およびサーバ負荷の低下が可能です。

##### PHP-FPM

- php-fpm は FastCGI Process Manager の略で PHP5.4.0 から公式サポートされた PHP 標準のアプリケーションサーバ
- Web サーバ（Apache/nginx）が別の PHP プロセスを立ち上げる
- Web サーバー側は「FastCGI として動作するプロセス」にリクエストを転送できればよく、プログラミング言語に依存しません。
- プログラミング言語側は FastCGI として動作するライブラリを用意しておけば、FastCGI に対応した Web サーバーから利用して貰うことができます。
- Apache の場合は、FastCGI にリクエストを転送するために、mod_proxy_fcgi というモジュールが用意されています。
- [参考：CentOS 8 標準の Apache と PHP の関係について](https://laboradian.com/centos8-apache-php/)

## パッケージ管理

### PEAR

- PHP Extension and Application Repository
- 拡張モジュールやアプリケーションが登録されたライブラリ・リポジトリ
- PHP で書かれている
- 古いので今は利用しない。
- 代わりに Composer を利用すること。

### PECL

[公式](https://www.php.net/manual/ja/install.pecl.intro.php)

- ピクルと読む
- PHP Extension Community Library
- C 言語で書かれた拡張ライブラリである
- PECL はリポジトリとしての名称ですが、PECL リポジトリからエクステンションを得る為のコマンドとしての名称でもあります。
- PHP で書かれたライブラリなら Composer でインストールすることができますが、C で書かれたライブラリを Composer からインストールすることは今のところできない
- エクステンションをインストールするには`PECL`コマンドのインストールが便利
  - エクステンションを自分でコンパイルすることもできる
  - コンパイル済みパッケージを導入する方法もある
- 実体は .[* so ファイル] として extension_dir に置かれる
- エクステンションの設定とかは php.ini に書く

#### インストール手順

[参考：【PHP・Laravel】pecl とは何か？pecl install と docker-php-ext-install の違い。docker-php-ext-enable などの使い方を実例で解説（pear との違い）](https://prograshi.com/language/php/pecl-install-docker-php-ext-install/)
pecl は PHP にバンドルされているので、そのままコマンドとして使うことができます。
次のコマンドでパッケージをインストールできます。
バージョンの指定がなければ安定版をインストールします。

```sh
$ pecl install 拡張パッケージ名[-バージョン]
```

実例

```sh
$ pecl install extname-0.1
```

pecl でインストールしたのみではパッケージは有効になりません。
このため、別途、有効化する必要があります。

##### Docker の場合

Docker の docker-compose.yml などで pecl を使っている場合は、pecl でパッケージをインストールした後に、以下も記述しておく必要があります。

`docker-php-ext-enable 拡張パッケージ名`
このため、`pecl install`と`docker-php-ext-enable`は基本的にセットで記述されています。

実例

```yml
RUN pecl install zip \
&& docker-php-ext-enable zip

RUN pecl install redis \
&& docker-php-ext-enable redis
```

##### ローカル環境の場合

###### 方法 1. php.ini を編集する。

「php.ini」ファイルを編集して、拡張パッケージを有効化します。
`Dynamic Extensions`と書かれているところの下に`;extension=拡張パッケージ名`があるので、冒頭の`;`を外します。
拡張パッケージ名がない場合は、`extension=拡張パッケージ名`を追記します。
以上で php.ini の編集は完了です。
実例：

```ini
;extension=bz2
extension=curl
;extension=fileinfo
```

###### 方法 2. dl 関数を使う。

`$ dl ( 拡張機能の名前 ) : true`
**実例**
拡張機能「curl」を有効化する場合は以下のようになります。

```ini
$ dl (curl): true
```

#### pecl で install できる拡張パッケージ一覧

PHP 標準ではなく、pecl で install できる拡張パッケージは [PECL の公式ページ](https://pecl.php.net/package-search.php)で検索することができます。

- redis
- xdebug
- memcached

など

### Composer

## PHP コマンド

### 参考

- [公式](https://www.php.net/manual/ja/features.commandline.options.php)
- [php のコマンドラインオプションの、これなあに？](https://qiita.com/akiko-pusu/items/93dc4cf63d23a1bb9bdb)

### `php -f` または `php <オプションなし>` （ファイルを指定して実行）

php の実行ファイルに、引数で php のスクリプトを渡すと実行できる。
`-f` オプションをつける場合も同様

```sh
$ php /path/to/php/file.php
### `php -r`,`php -R`
```

### `php -a`（対話モード）

```sh
$ php -a
Interactive shell

php > $array = [
php >     "foo" => "bar",
php >     "bar" => "foo",
php > ];
php > echo $array["foo"];
bar
php > quit // quitもしくはexitで抜けます
```

### `php -r` , `php -R` （コマンドライン上で PHP 実行）

- [PHP “-r” “-R”オプションを活用しよう。](https://programwiz.org/2021/04/26/php-option-code/)

`-r <code> Run PHP <code> without using script tags <?..?>`
php ファイルではなく、PHP コマンドの引数にコードを指定して実行するためのオプションです。
使い方は、下記の通り。

```sh
php -r 'echo "Hello World\n";'
```

> 実行結果
> Hello World

`-r` オプションに指定したコードが実行されます。
下記のように、複数行に渡る処理も実行することができます。（最後の`'`を入力するまでは普通に Enter で改行をして入力を続けられる）

```sh
php -r '
for ($i=0; $i<10; $i++){
  print("$i\n");
}
'
```

` -R <code> Run PHP <code> for every input line`
`-R`オプションも、PHP コードを指定して実行するためのオプションですが、パイプラインの入力を一行ずつ処理するために存在します。
説明は省略

### `php -S` （ビルトインサーバの起動）

#### 参考

- [公式](https://www.php.net/manual/ja/features.commandline.webserver.php)

#### 概要

例 1 ウェブサーバーの起動
ドキュメントルートに cd してから、`php -S host:port`を入力

```sh
$ cd ~/public_html
$ php -S localhost:8000
```

ターミナルには次のように表示されます。

```sh
PHP 5.4.0 Development Server started at Thu Jul 21 10:43:28 2011
Listening on localhost:8000
Document root is /home/me/public_html
Press Ctrl-C to quit
```

例 2.1 ドキュメントルートディレクトリを指定した起動(相対パス)
`- t`オプションを利用する

```sh
$ cd ~/public_html
$ php -S localhost:8000 -t foo/
```

ターミナルには次のように表示されます。

```sh
PHP 5.4.0 Development Server started at Thu Jul 21 10:50:26 2011
Listening on localhost:8000
Document root is /home/me/public_html/foo
Press Ctrl-C to quit
```

例 2.2 ドキュメントルートディレクトリを指定した起動(絶対パス)

```sh
$ php -S localhost:8000 -t /Users/username/Desktop/test
```

```sh
Document root is /Users/username/Desktop/test
```

# Linux での psth 設定

# CentOS

## リポジトリ

### epel

### remi

## アップデートの流れ

### CentOS 7 まで？

- sudo yum update
- sudo yum upgrade

### CentOS 8 から？

- sudo dnf update
- sudo dnf upgrade

# Ubuntu

## リポジトリ

## アップデートの流れ

- sudo apt-get update
- sudo apt-get upgrade

# Apache

- [Apache 公式](https://httpd.apache.org/docs/2.4/ja/mod/mod_include.html)
- [Apache 入門](https://www.javadrive.jp/apache/)
- [Apache の設定は後書きでわかりやすく！](https://www.go-next.co.jp/blog/server_network/4084/)
- [Include ディレクティブ：補助設定ファイルを読み込む](https://www.javadrive.jp/apache/setting/index2.html)
- [Apache に組み込まれているモジュールの一覧を取得する(httpd -l,httpd -M)](https://www.javadrive.jp/apache/setting/index5.html)
- [CentOS 7 の Apache 環境に PHP7.2 をインストールして基本的な設定を行う](https://www.rem-system.com/centos-php72-inst/)
- [Apahe から PHP を利用できるように設定する](https://www.javadrive.jp/apache/php/index1.html)

## 注意

Docker の CentOS イメージでは systemctl コマンドが使えないので httpd を start させられない。素直に Apache や PHP のイメージを利用すること。
