# docker-compose について

## `Docker-Compose.yml`の書き方

- [Docker Compose - docker-compose.yml リファレンス](https://qiita.com/zembutsu/items/9e9d80e05e36e882caaa)
- docker-compose.yml で定義される各々のサービスは、特定の `image` か `build` を指定する必要があります。
- その他のキーはオプションであり、docker run コマンドラインのものと似ています。
  - docker run と同様に、Dockerfile で指定されたオプションがデフォルトとして尊重されます（例：CMD,EXPOSE,VOLUME,ENV）。そのため、docker-compose.yml で再び定義する必要はありません。
- ひな形

```yml
version: "3.9"

services:
  app:
    container_name: docker-practice-app
    build:
      dockerfile: app/Dockerfile
      context: .
    ports:
      - "8080:80"
    volumes:
      - type: bind
        source: ./app/src
        target: /var/www/html/src
    # command: php -S 0.0.0.0:8000 -t /src

  db:
    container_name: docker-practice-db
    build:
      dockerfile: db/Dockerfile
      context: .
    volumes:
      - type: volume
        source: mysql-db-volume
        target: /var/lib/mysql
      - type: bind
        source: ./db/world.sql
        target: /docker-entrypoint-initdb.d/world.sql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_USER: hoge
      MYSQL_PASSWORD: hoge
      MYSQL_DATABASE: testdb

volumes:
  mysql-db-volume:
```

### `version`

- [公式](https://docs.docker.jp/compose/compose-file/compose-versioning.html)
- 2022.8.19 時点では公式ドキュメントには 3.8 が最新と記載されているが、実際は 3.9 が最新らしい[参考](https://forums.docker.com/t/docker-compose-version-3-8-or-3-9-for-latest/102439/2)

### `services`

Docker-Compose では、アプリケーションを動かすための各要素を Service と読んでいます。
そのため、ComposeFile にも、service として、それぞれの Services の内容をネストさせて記述していきます。
冒頭であげたサンプルコードをもう一度みてみると、db と app が Service として定義されていますね。

### `container_name`

- docker-compose を使用すると、特にコンテナ名を指定しない限り、自動的に名前が付与されます。
- デフォルトで生成される名前の代わりに、カスタム・コンテナ名を指定します。

### `image`

- docker-compose.yml で定義される各々のサービスは、特定の `image` か `build` を指定する必要があります。
- タグや image ID の一部です。ローカルでもリモートでも構いません。ローカルに存在しなければ、Compose はイメージを取得（pull）します。
- 例

```yml
image: ubuntu
image: orchardup/postgresql
image: a4bc65fd
```

### `build`

- docker-compose.yml で定義される各々のサービスは、特定の `image` か `build` を指定する必要があります。
- [参考：docker-compose.yml の build 設定はとりあえず context も dockerfile も埋めとけって話](https://qiita.com/sam8helloworld/items/e7fffa9afc82aea68a7a)

#### ポイント

docker-compose を使った開発では

```yml
docker-compose.yml
version: "3"
services:
  nginx:
    build: ./docker/nginx/
```

のように Dockerfile があるディレクトリを指定するだけでなく

```yml
docker-compose.yml
services:
  nginx:
    build:
      context: .
      dockerfile: ./docker/nginx/Dockerfile
```

のようにコンテキストをルートディレクトリに指定して、Dockerfile の場所も直接指定すること

#### `context`

- `context`とは「`docker image build`コマンドを実行した場所」
- `docker image build` コマンドを実行した場所ってことなので、`docker image build` コマンドは `Dockerfile` があるディレクトリで実行すれば問題はない
- しかし、`docker compose` コマンドを使って開発している場合は Dockerfile があるディレクトリでコマンドを実行することはほぼ無い
- また Docker はコンテキスト(カレントディレクトリ)の外のファイルにはアクセスできない仕様である
  - そのため、もしある `hoge service` に関して context を `hoge service` のディレクトリ内に指定してしまうと、その外側にある別の `fuga service` にアクセスできなくなる
- そのため各 service の`context`はルートディレクトリに指定してやる必要がある

#### `dockerfile`

- それに伴い`dockerfile`はルートディレクトリ(`context`)からの相対パスを指定する

### `ports`

ポートを指定（ホスト:コンテナ）

### `volumes`

```yml
db:
  container_name: docker-practice-db
  build:
    dockerfile: db/Dockerfile
    context: .
  volumes:
    - type: volume
      source: mysql-db-volume
      target: /var/lib/mysql
    - type: bind
      source: ./db/world.sql
      target: /docker-entrypoint-initdb.d/world.sql
```

- `type`に`volume`か`bind`を指定

### `environment`

環境変数を追加します。配列もしくは辞書形式（dictionary）で指定できます。boolean 値 (true、false、yes、no のいずれか) は、YML パーサによって True か False に変換されないよう、クォート（ ‘ 記号）で囲む必要があります。
キーだけの環境変数は、Compose の実行時にマシン上で指定するものであり、シークレット（訳注：API 鍵などの秘密情報）やホスト固有の値を指定するのに便利です。

```yml
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

## volumes について

`volume create` で行っていたボリュームの作成を `volumes:` で置き換えます。
ボリュームの作成は特定のサービスに対して行う操作ではないため、`services:` の下ではなくルートに記述します。

```yml
docker-compose.yml
version: '3.9'

services:
  app:
  db:
  mail:

volumes:
  docker-practice-db-volume:
```

## ネットワークの作成と DB / Mail コンテナのエイリアス設定

Docker Compose では 自動でブリッジネットワークが作成されます。
また サービス名が自動でコンテナのネットワークにおけるエイリアスとして設定されます。
なので `docker-compose.yml` に `app:`, `db:`, `mail:` と書き写した時点で `container run --network` と `container run --network-alias` の置き換えは完了しています。

## 起動と停止

### フォアグラウンドで起動

`$ docker compose up`

### バックグラウンド実行

`container run` と同様に `--detach` オプションを指定するとバックグラウンドで実行できます。

```sh
$ docker compose up --detach
```

### 停止・削除

フォアグラウンドでの実行は `ctrl + c` で止めることができますが、これは正常な Docker Compose の終了フローにならないため、コンテナは停止済のまま残ります。
Docker Compose で起動したサービスは `compose down` というコマンドで一気に停止することが可能で、この場合は `--rm` オプション指定時と同様に 停止済コンテナは削除されます。

```sh
$ docker compose down
```

また、`--build` オプションをつけることにより、コンテナの起動前にイメージの再ビルドをさせることができます。

```sh
$ docker compose up --build
```
