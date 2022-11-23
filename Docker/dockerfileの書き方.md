# `dockerfile`について

## 参考

- [公式 Dockerfile リファレンス](https://docs.docker.jp/engine/reference/builder.html)

## FROM: ベースイメージを指定する

FROM はベースになるイメージを指定する命令です。

次のように Dockerfile に FROM 命令を追記すると「これから ubuntu:20.04 のレイヤーの上にさらにレイヤーを乗せていくぞ」という意味になります。

```docker
FROM ubuntu:20.04
```

Dockerfile は FROM で始まります。

## RUN: 任意のコマンドを実行する

RUN は Linux のコマンドを実行してその結果をレイヤーとする命令です。
ubuntu:20.04 イメージに vi をインストールするレイヤーを重ねるには、次のように Dockerfile に RUN 命令を追記します。

```docker
FROM ubuntu:20.04

RUN apt update
RUN apt install -y vim
```

RUN により「コンテナを起動するたびに vi をインストールする」という手間を解決できます。

## COPY: ホストマシンのファイルをイメージに追加する

COPY はホストマシンのファイルをイメージに追加する命令です。
ubuntu:20.04 イメージに行番号を表示する設定を記した .vimrc を配置するには、まずホストマシンに .vimrc を作ります。

.vimrc ( Host Machine )

```
set number
```

次に、Dockerfile に COPY 命令を追記します。

```Dockerfile
FROM ubuntu:20.04

RUN apt update
RUN apt install -y vim

COPY .vimrc /root/.vimrc
```

COPY により「コンテナを起動するたびに .vimrc を作成する」という手間を解決できます。

## COPY と ADD の違い

イメージ・サイズの問題のため、 ADD でリモート URL 上のパッケージを取得するのは可能な限り避けてください。その代わりに curl や wget を使うべきです。この方法であれば、展開後に不要となったファイルを削除でき、イメージに余分なレイヤを増やしません。例えば、次のような記述は避けるべきです。
**NG**

```docker
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

そのかわり、次のように記述します。
**OK**

```docker
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

## CMD: デフォルト命令を指定する

CMD はイメージのデフォルト命令を設定する命令です。
bash の起動する汎用イメージではなく、特定のフォーマットで現在時刻を表示するイメージにしたいので、次のように Dockerfile に CMD 命令を追記します。

```Dockerfile
FROM ubuntu:20.04

RUN apt update
RUN apt install -y vim

COPY .vimrc /root/.vimrc

CMD date +"%Y/%m/%d %H:%M:%S ( UTC )"
```

CMD により「container run で date +"%Y/%m/%d %H:%M:%S ( UTC )" という複雑な引数を毎回指定する」という手間を解決できます。

### `RUN`と`CMD`の違い

- RUN - イメージ作成時に実行される
  - image にパッケージをインストールするなどの処理が記述されます。
- CMD - コンテナ実行時に実行される
  - web サーバーの起動などに使用される事が多いです。
  - 逆に言うと RUN でサービスの起動を記載してもコンテナ起動時にサービスが立ち上がってないみたいなかんじになる。

### 具体例

```Dockerfile
FROM alpine

RUN echo "RUN!"
CMD echo "CMD!"
```

#### イメージ作成

`$ docker build . -t example_container`

> (色々な処理)
> RUN!
> (色々な処理)

#### コンテナ実行

`$ docker run example_container`

> CMD!
