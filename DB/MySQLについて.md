# Docker で MySQL を使う

## サンプル

```sh
$ docker run \
--name some-mysql \
-e MYSQL_ROOT_PASSWORD=my-secret-pw \
-d \
mysql:latest
```

## 環境変数

[Docker Hub](https://hub.docker.com/_/mysql/#:~:text=mysql%3Atag%20%2D%2Dverbose%20%2D%2Dhelp-,Environment%20Variables,-When%20you%20start) に行くと Environment Variables の項に説明があるので、それを参考に次の 4 つを指定します。

| 変数名                                 | 用途                                                | 値                                                           |
| -------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| `MYSQL_ROOT_PASSWORD`                  | ルートユーザのパスワード                            | 自由に決めて指定する                                         |
| `MYSQL_USER` <br>( 設定上は任意 )      | 作成するユーザの名前<br> 指定すると作成される       | 自由に決めて指定する                                         |
| `MYSQL_PASSWORD` <br> ( 設定上は任意 ) | 作成するユーザのパスワード <br>指定すると作成される | 自由に決めて指定する <br>ただし `MYSQL_ROOT_PASSWORD` とは別 |
| `MYSQL_DATABASE` <br>( 設定上は任意 )  | データベースの名前<br>指定すると作成される          | `event`                                                      |

## `/docker-entrypoint-initdb.d`：初期データを登録する

### 参考

- [サンプルデータがあらかじめ入った MySQL を Docker で作成する](https://www.xlsoft.com/jp/blog/blog/2019/10/09/post-7617/)
- [公式](https://hub.docker.com/_/mysql#:~:text=Initializing%20a%20fresh%20instance)
  - > コンテナが初めて起動されると、指定された名前の新しいデータベースが作成され、提供された構成変数で初期化されます。さらに、/docker-entrypoint-initdb.d にある拡張子 .sh, .sql, .sql.gz のファイルがアルファベット順で実行されます。このディレクトリに SQL ダンプをマウントすることで、簡単に mysql サービスを投入することができ、貢献したデータでカスタムイメージを提供することができます。SQL ファイルはデフォルトで MYSQL_DATABASE 変数で指定されたデータベースにインポートされます。
- つまりコンテナ起動時に `/docker-entrypoint-initdb.d` に存在する `.sql` を実行してくれる拡張がされています。
- ただし初期化クエリは既にデータが存在する場合は実行されないため、create 文のつくりを工夫したり、ボリュームの削除と再作成をしたりする必要があります。

# `mysql`コマンド

## 参考

- [MySQL コマンドラインツールの利用](https://www.javadrive.jp/mysql/connect/)

## 接続方法

bash から mysql コマンドで MySQL サーバに接続します。
接続オプションは次の通りです。
| オプション | 意味 | 値 |
|---|---|---|
| `-h`| ホスト | localhost |
| `-u`| ユーザ | ルートではない作成したユーザ |
| `-p`| パスワード | ルートではない作成したユーザ<br>空白を開けずに入力する |
| 引数| データベース名 | 作成したデータベース名 |

```sh
# mysql -h localhost -u hoge -ppassword dbname
mysql>
```

## ユーザー名とパスワードの指定方法

### ユーザー名

ユーザー名は次のオプションを使って指定します。(どちらの形式でも同じです)。

```sh
-u ユーザー名 -h ホスト名
--user=ユーザー名 --host=ホスト名
```

MySQL ではユーザーを`ユーザー名＋接続ホスト名`で管理しています。インストール直後に登録されているのはユーザー名が root でホスト名が localhost のユーザーのみです。その為、ユーザー名には `-u root -h localhost` と指定します。ただホスト名を省略した場合は `localhost` と判定されるため、ユーザー名には単に `-u root` と入力しても構いません。

```sh
-u root -h localhost
-u root
--user=root
```

### パスワード

続いてパスワードです。パスワードは次のオプションを使って指定します。

```sh
-p[パスワード]
--password[=パスワード]
```

`-p` オプションを使ってパスワードを指定する場合は、 `-p` とパスワードの**間に空白をいれずに指定**します。例えばパスワードが mypass だった場合、次のいずれかの方法で指定します。

```sh
-pmypass
--password=mypass
```

MySQL コマンドラインツールを起動するときに -u オプションおよび -p オプションを指定して mysql コマンドを実行するには次のようになります。

```sh
mysql -u root -pmypass
mysql --user=root --password=mypass
```

### ユーザー名とパスワードの指定方法(1)

ただこの方法だと入力したパスワードが画面上にそのまま表示されてしまいます。そこで、次のように `-p` または `--password` オプションだけを指定し実際のパスワードは入力を行わずに実行します。

```sh
mysql -u root -p
mysql --user=root --password
```

## `system`コマンド

## `SOURCE`コマンド
