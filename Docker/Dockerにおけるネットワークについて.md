# Docker におけるネットワークについて

## ポートについて

`--publish`, `-p`
公開ポートは `--publish host-machine:container` の形式でマッピングします。
このとき host-machine 側のポートは起動する人が自由に決めます が、1024 未満のポートは特権ポートと呼ばれるあらかじめ用途が決められているポートなるので避けておくのが無難です。

```sh
$ docker container run                        \
    --name app                                \
    --rm                                      \
    --detach                                  \
    --interactive                             \
    --tty                                     \
    --mount type=bind,src=$(pwd)/src,dst=/src \
    --publish 18000:8000                      \
    docker-practice:app                       \
    php -S 0.0.0.0:8000 -t /src
```

- ホストマシンからアクセス したい場合は、コンテナのポートを公開する
- コンテナ側のポートは、起動しているサービスに合わせる
- ホストマシン側のポートは、衝突しないように自分で決める

## ネットワークドライバ

- Docker のコンテナは**ネットワークドライバ**というもので Docker ネットワークに接続されます。
- ネットワークドライバはデフォルトでいくつか用意されており、たとえば**ブリッジネットワーク**や**オーバーレイネットワーク**というものがあります。

### ブリッジネットワーク

- ネットワークドライバを特に指定しなかった場合の**デフォルト**である
- **同一の Docker Engine 上のコンテナ** が互いに通信をする場合に利用する

### オーバーレイットワーク

- **異なる Docker Engine 上のコンテナ** が互いに通信をする場合に利用する

![](https://res.cloudinary.com/zenn/image/fetch/s--krSxoat5--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/13ca7d98474dad769c53b5ff.jpeg%3Fsha%3Ddc5db27cb68f28636f7a0634baa595bd6acb4566)

## デフォルトブリッジネットワーク

コンテナを起動する際にネットワークドライバについて一切の指定を行わないと、デフォルトブリッジネットワークが自動的に生成され、コンテナはこのネットワークに接続されます。このデフォルトブリッジネットワークには次のような特徴があります。

- コンテナが通信するためには、全てのコンテナ間をリンクする操作が必要になる
- コンテナ間の通信は IP アドレスで行う
- Docker Engine 上の全てのコンテナ ( たとえば別プロジェクト ) に接続できてしまう

![](https://res.cloudinary.com/zenn/image/fetch/s--nlJx_Fgl--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/35913ea864711aa63fb02130.jpeg%3Fsha%3D5b35cf21e5702b324681ef60077b9653739f54dc)

## ユーザ定義ブリッジネットワーク

これに対し、自分でブリッジネットワークを作成すると、デフォルトネットワークと比べて次のような利点のあるネットワークに接続できます。

- 相互通信をできるようにするには同じネットワークを割り当てるだけでよい
- コンテナ間で自動的に DNS 解決を行える
- 通信できるコンテナが同一ネットワーク上のコンテナに限られ、隔離度があがる
