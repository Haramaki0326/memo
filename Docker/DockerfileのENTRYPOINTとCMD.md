# Dockerfile の ENTRYPOINT と CMD

## 参考

- [Dockerfile の ENTRYPOINT と CMD](https://architecting.hateblo.jp/entry/2020/08/13/171732)

## `CMD`

### 記述例

```docker
CMD [ "echo",  "hi" ]
```

#### 実行例１

下記のようにコンテナを実行するとコンテナは「hi」と表示して終了する。

```sh
$ docker run ubuntu
hi
$
```

#### 実行例２

`docker run` でコマンドを指定すると `CMD` の内容を上書きする。

下記の例では docker run で指定した「echo bye」が実行されている。

```sh
$ docker run ubuntu echo bye
bye
$
```

## `ENTRYPOINT`

次に ENTRYPOINT を見てみよう。

### 記述例

以下に ENTRYPOINT の記述例を示す。

```docker
ENTRYPOINT [ "echo", "hi" ]
```

#### 実行例１

下記のようにコンテナを実行するとコンテナは「hi」と表示して終了する。

```sh
$ docker run ubuntu
hi
$
```

#### 実行例２

`CMD` と異なり `docker run` でコマンドを指定しても `ENTRYPOINT` の内容は上書きされない。`ENTRYPOINT` の内容に追加される。

下記の例では `ENTRYPOINT` で指定した「echo hi」と docker run で指定した「echo bye」が結合された「echo hi echo bye」が実行されている。

```sh
$ docker run ubuntu echo bye
hi echo bye
$
```

#### 実行例３

実行例２の動作を利用して以下のように、`ENTRYPOINT` で「echo」を、docker run で表示する文字列をそれぞれ指定することができる。

下記の例では `ENTRYPOINT` で指定した「echo」と docker run で指定した「bye」が結合された「echo bye」が実行されている。

```docker
ENTRYPOINT [ "echo" ]
```

```sh
$ docker run ubuntu bye
bye
$
```

## まとめ

以上のことをまとめると `ENTRYPOINT` と `CMD` の Best Practice は以下の通りとなる。

- 絶対に実行したいプロセスを `ENTRYPOINT` で指定する。
- デフォルト引数を `CMD` で指定する。
- デフォルト以外の引数をプロセスに渡したい場合は `docker run` で指定する。

# RUN、ENTRYPOINT、CMD の exec 表記と shell 表記

RUN、ENTRYPOINT、CMD には exec 表記と shell 表記の 2 つの記法がある。

## shell 表記

shell 表記で記述した例を示す。

```docker
CMD echo hi
```

- shell 表記を使用すると`/bin/sh -c`が起動し、そのシェル上で指定したコマンド（echo hi）が実行される
- シェル機能を利用できることがメリット
  - 環境変数の置き換え（$HOME → /home/user1）
  - パイプやリダイレクト
- 「`/bin/sh -c`」が Root Process になる
  - 指定したコマンドは PID 2 になる

#### `/bin/sh -c`について

`-c string`

- `-c` オプションが指定されると、コマンドが string から読み込まれます。
  - [bash のｃオプションの使いどころが知りたい](https://teratail.com/questions/366124)

## exec 表記

exec 表記で記述した例を示す。

```docker
CMD [ "echo", "hi" ]
```

- exec 表記を使用すると「`/bin/sh -c`」は起動せず、直接、指定したコマンド- （echo hi）が実行される
- exec 表記ではシェル機能は利用できない
- `[]`は JSON 配列である
- 配列の要素はダブルクオテーションで囲わなくてはならない
- 配列の最初の要素は Executable でなくてはならない
  - `CMD [ "echo hi" ]`は NG
- 指定したコマンド（echo hi）が Root Process になる

## 結論

- 基本的には exec 表記を使用する
- シェルの機能が必須であれば shell 表記を使用する

## exec 表記が推奨される理由

- exec 表記はシェルを実行しないのでリソースを節約できる
- exec 表記で指定したプロセスは Root Process になる（注）

## 注

Root Process は以下のような特性があるためどのプロセスが Root Process になるかは重要である。

- Root Process は docker stop 時にシグナルを受け取る。
  - 終了時の処理を記述したプログラムがあればそのプログラムを Root Process にしたい。
- Root Process が終了するとコンテナが終了する。
  - 本来実行したいことが完了する前にコンテナが終了しては困る。
