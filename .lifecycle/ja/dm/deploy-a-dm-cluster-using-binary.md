---
title: DMバイナリを使用してデータ移行をデプロイする
summary: DMバイナリを使用してデータ移行クラスタをデプロイする方法について学びます。
aliases: ['/docs/tidb-data-migration/dev/deploy-a-dm-cluster-using-binary/']
---

# DMバイナリを使用してデータ移行をデプロイする

このドキュメントでは、DMバイナリを使用してData Migration（DM）クラスタを迅速にデプロイする方法について紹介します。

> **注意:**
>
> 本番環境では、[TiUPを使用してDMクラスタをデプロイすることをお勧めします](/dm/deploy-a-dm-cluster-using-tiup.md)。

## DMバイナリのダウンロード

DMバイナリはTiDB Toolkitに含まれています。TiDB Toolkitをダウンロードするには、[TiDBツールをダウンロード](/download-ecosystem-tools.md)してください。

## サンプルシナリオ

次のサンプルシナリオに基づいて、DMクラスタを展開するとします：

2つのDM-workerノードと3つのDM-masterノードが5台のサーバに展開されているとします。

以下に各ノードのアドレスが示されています：

| インスタンス | サーバーアドレス | ポート |
| :---------- | :----------- | :-- |
| DM-master1 | 192.168.0.4 | 8261 |
| DM-master2 | 192.168.0.5 | 8261 |
| DM-master3 | 192.168.0.6 | 8261 |
| DM-worker1 | 192.168.0.7 | 8262 |
| DM-worker2 | 192.168.0.8 | 8262 |

このシナリオに基づいて、以下のセクションでは、DMクラスタの展開方法について説明します。

> **注意:**
>
> - 単一サーバに複数のDM-masterまたはDM-workerのインスタンスを展開する場合、各インスタンスのポートと作業ディレクトリは一意である必要があります。
>
> - DMクラスタの高可用性を確保する必要がない場合は、1つのDM-masterノードのみを展開し、展開されたDM-workerノードの数は、移行する上流のMySQL/MariaDBインスタンスの数より少なくしてください。
>
> - DMクラスタの高可用性を確保するためには、3つのDM-masterノードを展開することをお勧めし、展開されたDM-workerノードの数は、移行する上流のMySQL/MariaDBインスタンスの数より多い（例：DM-workerノードの数は上流のインスタンスの数より2つ多い）ようにしてください。
>
> - 次のコンポーネント間のポートが相互接続されていることを確認してください：
>     - DM-masterノード間の`8291`ポートが相互接続されています。
>     - 各DM-masterノードが、すべてのDM-workerノードの`8262`ポートに接続できます。
>     - 各DM-workerノードが、すべてのDM-masterノードの`8261`ポートに接続できます。

### DM-masterの展開

DM-masterは、[コマンドラインパラメータ](#dm-master-command-line-parameters)または[設定ファイル](#dm-master-configuration-file)を使用して構成することができます。

#### DM-masterコマンドラインパラメータ

以下は、DM-masterコマンドラインパラメータの説明です：

```bash
./dm-master --help
```

```
Usage of dm-master:
  -L string
        log level: debug, info, warn, error, fatal (default "info")
  -V    prints version and exit
  -advertise-addr string
        advertise address for client traffic (default "${master-addr}")
  -advertise-peer-urls string
        advertise URLs for peer traffic (default "${peer-urls}")
  -config string
        path to config file
  -data-dir string
        path to the data directory (default "default.${name}")
  -initial-cluster string
        initial cluster configuration for bootstrapping, e.g. dm-master=http://127.0.0.1:8291
  -join string
        join to an existing cluster (usage: cluster's "${master-addr}" list, e.g. "127.0.0.1:8261,127.0.0.1:18261"
  -log-file string
        log file path
  -master-addr string
        master API server and status addr
  -name string
        human-readable name for this DM-master member
  -peer-urls string
        URLs for peer traffic (default "http://127.0.0.1:8291")
  -print-sample-config
        print sample config file of dm-worker
```

> **注意:**
>
> 一部の状況では、コマンドラインでの一部の設定項目が公開されていないため、DM-masterを構成するために上記の方法を使用できないことがあります。その場合は代わりに設定ファイルを使用してください。

#### DM-master設定ファイル

以下は、DM-masterの設定ファイルです。この方法でDM-masterを構成することをお勧めします。

1. 次の構成を `conf/dm-master1.toml`に書き込んでください：

      ```toml
      # Master Configuration.
      name = "master1"

      # Log configurations.
      log-level = "info"
      log-file = "dm-master.log"

      # DM-masterのリスニングアドレス.
      master-addr = "192.168.0.4:8261"

      # DM-masterのピアURL.
      peer-urls = "192.168.0.4:8291"

      # `initial-cluster`の値は、初期クラスタのすべてのDM-masterノードの`advertise-peer-urls`値の組み合わせです。
      initial-cluster = "master1=http://192.168.0.4:8291,master2=http://192.168.0.5:8291,master3=http://192.168.0.6:8291"
      ```

2. ターミナルで以下のコマンドを実行して、DM-masterを実行します：

      {{< copyable "shell-regular" >}}

      ```bash
      ./dm-master -config conf/dm-master1.toml
      ```

      > **注意:**
      >
      > このコマンドが実行された後、コンソールにはログが出力されません。ランタイムログを表示するには、`tail -f dm-master.log`を実行してください。

3. DM-master2とDM-master3については、設定ファイル内の`name`をそれぞれ`master2`および`master3`に変更し、`peer-urls`をそれぞれ`192.168.0.5:8291`および`192.168.0.6:8291`に変更し、ステップ2を繰り返してください。

### DM-workerの展開

DM-workerは、[コマンドラインパラメータ](#dm-worker-command-line-parameters)または[設定ファイル](#dm-worker-configuration-file)を使用して構成することができます。

#### DM-workerコマンドラインパラメータ

以下は、DM-workerのコマンドラインパラメータの説明です：

```bash
./dm-worker --help
```

```
Usage of worker:
  -L string
        log level: debug, info, warn, error, fatal (default "info")
  -V    prints version and exit
  -advertise-addr string
        advertise address for client traffic (default "${worker-addr}")
  -config string
        path to config file
  -join string
        join to an existing cluster (usage: dm-master cluster's "${master-addr}")
  -keepalive-ttl int
        dm-worker's TTL for keepalive with etcd (in seconds) (default 10)
  -log-file string
        log file path
  -name string
        human-readable name for DM-worker member
  -print-sample-config
        print sample config file of dm-worker
  -worker-addr string
        listen address for client traffic
```

> **注意:**
>
> 一部の状況では、コマンドラインでの一部の設定項目が公開されていないため、DM-workerを構成するために上記の方法を使用できないことがあります。その場合は代わりに設定ファイルを使用してください。

#### DM-worker設定ファイル

以下は、DM-workerの設定ファイルです。この方法でDM-workerを構成することをお勧めします。

1. 次の構成を `conf/dm-worker1.toml`に書き込んでください：

      ```toml
      # Worker Configuration.
      name = "worker1"

      # Log configuration.
      log-level = "info"
      log-file = "dm-worker.log"

      # DM-workerのアドレス.
      worker-addr = ":8262"

      # クラスタ内のDM-masterノードのmaster-addr構成.
      join = "192.168.0.4:8261,192.168.0.5:8261,192.168.0.6:8261"
      ```

2. ターミナルで以下のコマンドを実行して、DM-workerを実行します：

      {{< copyable "shell-regular" >}}

      ```bash
      ./dm-worker -config conf/dm-worker1.toml
      ```

3. DM-worker2については、設定ファイル内の`name`を`worker2`に変更し、ステップ2を繰り返してください。

これで、DMクラスタが正常に展開されました。