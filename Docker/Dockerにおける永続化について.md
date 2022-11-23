# Docker における永続化について

## volume

### volume の作成

```sh
$ docker volume create [option]
```

具体例

```sh
$ docker volume create \
    --name docker-practice-db-volume
```

### 一覧

```sh
$ docker volume ls

DRIVER VOLUME NAME
local docker-practice-db-volume
```

### DB コンテナにボリュームをマウントする

作成したボリュームは、コンテナ起動時にコンテナ内の領域にマウントして使います。

`container run` には　`--volume` オプションと `--mount` オプションというほぼ同じことができるオプションがある

#### `--volume` オプションによるマウント

`--volume` オプションは 設定を定められた順番で : 区切りで列挙して指定します。: で区切られた設定のうち１つめはボリューム名であり、これは先ほど作成した docker-practice-db-volume を指定します。
２つめはマウント先であり、これは MySQL データベースのテーブル情報が実際に保存されている `/var/lib/mysql` を指定します。

```sh
$ docker container run                                \
    --name db                                         \
    --rm                                              \
    --detach                                          \
    --platform linux/amd64                            \
    --env MYSQL_ROOT_PASSWORD=rootpassword            \
    --env MYSQL_USER=hoge                             \
    --env MYSQL_PASSWORD=password                     \
    --env MYSQL_DATABASE=event                        \
    --volume docker-practice-db-volume:/var/lib/mysql \
    docker-practice:db
```

⇒ この例ではボリューム`docker-practice-db-volume`をコンテナ内の領域`/var/lib/mysql`にマウントしている

#### `--mount` オプションによるマウント

`--mount` オプションは key=val 形式で設定を列挙 して指定します。主なキーは type と source と destination で、ほかに readonly のような任意オプションのキーもあります。また、source は src など、destination は dst や target などの略記も存在します。
ボリュームをマウントするには type に volume を指定します。
source はボリューム名の docker-practice-db-volume を指定します。
destination は /var/lib/mysql を指定します。

```sh
$ docker container run                                                   \
    --name db                                                            \
    --rm                                                                 \
    --detach                                                             \
    --platform linux/amd64                                               \
    --env MYSQL_ROOT_PASSWORD=rootpassword                               \
    --env MYSQL_USER=hoge                                                \
    --env MYSQL_PASSWORD=password                                        \
    --env MYSQL_DATABASE=event                                           \
    --mount type=volume,src=docker-practice-db-volume,dst=/var/lib/mysql \
    docker-practice:db
```

#### `--volume` オプションと `--mount` オプションの使い分け

`--volume` オプションの方が短くかけますが、バインドマウントも混じるとかなりわかりづらい記述になります。: 区切りのルールなどを知らないと正しく読むこともままならないので、マウントするときは読みやすい `--mount` オプションを使う方が良い でしょう。
`--mount` オプションの方が Docker Compose と互いに読み換えやすいという利点もあります。

## bind

### バインドマウントとは

バインドマウントは ホストマシンの任意のディレクトリをコンテナにマウントする 仕組みです。
ホストマシンとコンテナ双方がファイルの変更に関心がある という場合に有用で、たとえばソースコードの共有などに活用できます。
ソースコードのディレクトリをコンテナにバインドマウントしてホストマシン側と共有すれば、ホストマシンでコードを変更した際に同期や転送が不要 になります。

### App コンテナにソースコードをマウントする

バインドマウントはボリュームと違い既存のディレクトリをそのままマウントするので、事前作成などはありません。バインドマウントも `--volume` オプションと `--mount` オプションどちらでも行えます。

### `--volume` オプションによるマウント

バインドマウントを行う場合も、: 区切りで設定を列挙します。
１つめを ボリューム名ではなく絶対パスにする とバインドマウントと判断されます。値はソースコードのある `$(pwd)/src` を指定します。
２つめはマウント先であり、これは自分で都合の良い場所にします。今回は /src とします。
３つめのオプションは特に指定しません。
また、ビルトインウェブサーバーのドキュメントルートをソースコードをマウントして配置する `/src` に変更したいため、`<command>` の末尾に `-t /src` を付け加えます。

```sh
$ docker container run          \
    --name app                  \
    --rm                        \
    --detach                    \
    --interactive               \
    --tty                       \
    --volume $(pwd)/src:/src    \
    docker-practice:app         \
    php -S 0.0.0.0:8000 -t /src
```

### `--mount` オプションによるマウント

バインドマウントを行う場合も、同じキーを使って key=value 方式で設定します。

`type` は volume ではなく bind を指定します。
`source` はソースコードのある $(pwd)/src を指定します。
`destination` は /src を指定します。

```sh
$ docker container run                        \
    --name app                                \
    --rm                                      \
    --detach                                  \
    --interactive                             \
    --tty                                     \
    --mount type=bind,src=$(pwd)/src,dst=/src \
    docker-practice:app                       \
    php -S 0.0.0.0:8000 -t /src
```

### volume と bind の見分け方

どちらも`-v`または`--volume`オプションを利用できるため、この場合は volume と bind のどちらを指定しているのかが分かりづらい
形式としては

- volume の場合は`ボリューム名:マウント先`となる
- bind の場合は`host側のディレクトリ:マウント先`となる

### COPY とバインドマウントの使い分け

`COPY` とバインドマウントはどちらもホストマシンのファイルをコンテナで扱えるようにする機能ですが、用途と反映タイミングを理解しておかないと扱いを間違えやすいので整理しましょう。

#### `COPY`

`COPY` は `image build` をするタイミングでイメージにファイルを含める ため、コンテナが起動すればファイルが存在 します。
また 元ファイルの変更を行ってもコンテナには反映されない ため、image build の再実行が必要 になります。
`COPY` の用途には次のようなものが挙げられるでしょう。

- 設定ファイルなど、コンテナによって変えない かつ 滅多に変更しない ものを配置する場合
- 本番デプロイ時のソースコードなど、即起動できる配布物を作る場合

#### バインド

対してバインドマウントは イメージではなくコンテナに行う ため、同じイメージを使ってもファイルの存在はコンテナ起動のオプションによって異なります。
また ホストマシンでファイルを変更するとコンテナに即時影響します。
バインドマウントの用途には次のようなものが挙げられるでしょう。

- 開発時のソースコードなど、ホストマシンで変更したいがコンテナに随時反映したい ものがある場合
- 初期化クエリなど、イメージを配布する時点では用意できない ものがある場合

２つは イメージに対して行っている か コンテナに対して行っている かが明確に違います。
その点をちゃんと理解しておけば、使い分け も 変更したいならどうすべきか も判断できます。
