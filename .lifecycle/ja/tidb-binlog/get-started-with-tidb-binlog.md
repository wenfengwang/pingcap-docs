---
title: TiDB Binlog チュートリアル
summary: 単純な TiDB クラスターで TiDB Binlog を展開する方法を学びます。
aliases: ['/docs/dev/get-started-with-tidb-binlog/', '/docs/dev/how-to/get-started/tidb-binlog/']
---

# TiDB Binlog チュートリアル

このチュートリアルは、単一の各コンポーネント（配置ドライバー、TiKVサーバ、TiDBサーバ、Pump、Drainer）のノードで構成された単純なTiDB Binlogデプロイメントから始まり、データをMariaDBサーバインスタンスにプッシュするように設定します。

このチュートリアルは、[TiDBアーキテクチャ](/tidb-architecture.md)にある程度精通しているユーザー、TiDBクラスターのセットアップを既に行ったかもしれないが（必須ではない）、TiDB Binlogを実際に操作してみたいユーザーを対象としています。このチュートリアルは、TiDB Binlogの概念に慣れ親しむための良い方法です。

> **警告:**
>
> このチュートリアルでTiDBを展開する手順は、本番環境または開発環境で使用するべきでは**ありません**。

このチュートリアルでは、x86-64のモダンなLinuxディストリビューションを使用していることを前提としています。このチュートリアルでは、例としてVMwareで実行されている最小限のCentOS 7インストールが使用されています。既存の環境のクセに影響を受けないようにするため、クリーンインストールから開始することをお勧めします。ローカルの仮想化を使用したくない場合は、クラウドサービスを使用して簡単にCentOS 7 VMを起動できます。

## TiDB Binlog の概要

TiDB Binlog は、TiDBからバイナリログデータを収集し、リアルタイムのデータバックアップとレプリケーションを提供するためのソリューションです。TiDBサーバクラスターからの増分データ更新を、ダウンストリームプラットフォームにプッシュします。

TiDB Binlog は、増分バックアップ、1つのTiDBクラスターから別のTiDBクラスターへのデータレプリケーション、またはTiDBの更新をKafkaを介して選択したダウンストリームプラットフォームに送信するために使用できます。

データを MySQL または MariaDB から TiDB に移行する際に TiDB DM（Data Migration）プラットフォームを使用して、MySQL/MariaDB クラスターからデータを TiDB に取り込み、TiDB Binlog を使用して TiDB クラスターとは別のダウンストリームの MySQL/MariaDB インスタンス/クラスターを同期させる場合、TiDB Binlog は特に有用です。TiDB Binlog により、TiDBへのアプリケーショントラフィックをダウンストリームの MySQL または MariaDB インスタンス/クラスターにプッシュすることができます。そのため、TiDBへの移行のリスクが減少し、MySQL または MariaDB にアプリケーションをダウンタイムやデータ損失なしで簡単に戻すことができます。

詳細は[TiDB Binlog クラスター ユーザーガイド](/tidb-binlog/tidb-binlog-overview.md)を参照してください。

## アーキテクチャー

TiDB Binlog は、**Pump** と **Drainer** の2つのコンポーネントで構成されています。 複数のPumpノードがPumpクラスターを構成します。各Pumpノードは、TiDBサーバインスタンスに接続し、クラスタ内の各TiDBサーバインスタンスで行われた更新を受信します。また、DrainerはPumpクラスターに接続し、受信した更新を特定のダウンストリーム先（たとえば、Kafka、別のTiDBクラスターまたはMySQL/MariaDBサーバ）の正しい形式に変換します。

![TiDB-Binlogアーキテクチャー](/media/tidb-binlog-cluster-architecture.png)

Pumpのクラスタードアーキテクチャーにより、新しいTiDBサーバインスタンスがTiDBクラスターに参加したり離脱したりする際に更新が失われないようになっています。

## インストール

この場合は MySQL サーバーの代わりに MariaDB サーバーを使用します。RHEL/CentOS 7にはMariaDBサーバがデフォルトのパッケージリポジトリに含まれているためです。後で使用するためにクライアントとサーバーの両方が必要です。それらをインストールしましょう：

```bash
sudo yum install -y mariadb-server
```

```bash
curl -L https://download.pingcap.org/tidb-community-server-v7.4.0-linux-amd64.tar.gz | tar xzf -
cd tidb-latest-linux-amd64
```

期待される出力：

```
[kolbe@localhost ~]$ curl -LO https://download.pingcap.org/tidb-latest-linux-amd64.tar.gz | tar xzf -
  % Total    % Received    % Xferd    Average Speed    Time    Time     Time  Current
                                 Dload    Upload    Total    Spent    Left    Speed
100  368M  100  368M    0        0   8394k      0  0:00:44  0:00:44 0:00:00 11.1M
[kolbe@localhost ~]$ cd tidb-latest-linux-amd64
[kolbe@localhost tidb-latest-linux-amd64]$
```

## 設定

次に、単純なTiDBクラスターを開始し、`pd-server`、`tikv-server`、`tidb-server`の各1つのインスタンスを使用します。

以下のコマンドを使用して、設定ファイルを構築します：

```bash
printf > pd.toml %s\\n 'log-file="pd.log"' 'data-dir="pd.data"'
printf > tikv.toml %s\\n 'log-file="tikv.log"' '[storage]' 'data-dir="tikv.data"' '[pd]' 'endpoints=["127.0.0.1:2379"]' '[rocksdb]' max-open-files=1024 '[raftdb]' max-open-files=1024
printf > pump.toml %s\\n 'log-file="pump.log"' 'data-dir="pump.data"' 'addr="127.0.0.1:8250"' 'advertise-addr="127.0.0.1:8250"' 'pd-urls="http://127.0.0.1:2379"'
printf > tidb.toml %s\\n 'store="tikv"' 'path="127.0.0.1:2379"' '[log.file]' 'filename="tidb.log"' '[binlog]' 'enable=true'
printf > drainer.toml %s\\n 'log-file="drainer.log"' '[syncer]' 'db-type="mysql"' '[syncer.to]' 'host="127.0.0.1"' 'user="root"' 'password=""' 'port=3306'
```

次のコマンドを使用して、構成の詳細を確認します：

```bash
for f in *.toml; do echo "$f:"; cat "$f"; echo; done
```

期待される出力：

```
drainer.toml:
log-file="drainer.log"
[syncer]
db-type="mysql"
[syncer.to]
host="127.0.0.1"
user="root"
password=""
port=3306

pd.toml:
log-file="pd.log"
data-dir="pd.data"

pump.toml:
log-file="pump.log"
data-dir="pump.data"
addr="127.0.0.1:8250"
advertise-addr="127.0.0.1:8250"
pd-urls="http://127.0.0.1:2379"

tidb.toml:
store="tikv"
path="127.0.0.1:2379"
[log.file]
filename="tidb.log"
[binlog]
enable=true

tikv.toml:
log-file="tikv.log"
[storage]
data-dir="tikv.data"
[pd]
endpoints=["127.0.0.1:2379"]
[rocksdb]
max-open-files=1024
[raftdb]
max-open-files=1024
```

## ブートストラップ

それでは、各コンポーネントを起動できます。特定の順序で行うのがベストです。まず最初に配置ドライバー（PD）、次にTiKVサーバー、その後Pump（なぜならTiDBがバイナリログを送信するためにPumpサービスに接続する必要があるため）、最後にTiDBサーバーです。

次のコマンドを使用して、すべてのサービスが起動するようにします：

```bash
./bin/pd-server --config=pd.toml &>pd.out &
./bin/tikv-server --config=tikv.toml &>tikv.out &
./pump --config=pump.toml &>pump.out &
sleep 3
./bin/tidb-server --config=tidb.toml &>tidb.out &
```

期待される出力:

```
[kolbe@localhost tidb-latest-linux-amd64]$ ./bin/pd-server --config=pd.toml &>pd.out &
[1] 20935
[kolbe@localhost tidb-latest-linux-amd64]$ ./bin/tikv-server --config=tikv.toml &>tikv.out &
[2] 20944
[kolbe@localhost tidb-latest-linux-amd64]$ ./pump --config=pump.toml &>pump.out &
[3] 21050
[kolbe@localhost tidb-latest-linux-amd64]$ sleep 3
[kolbe@localhost tidb-latest-linux-amd64]$ ./bin/tidb-server --config=tidb.toml &>tidb.out &
[4] 21058
```

`jobs`を実行すると、実行中のデーモンのリストが表示されるはずです：

```
[kolbe@localhost tidb-latest-linux-amd64]$ jobs
[1]   Running                 ./bin/pd-server --config=pd.toml &>pd.out &
[2]   Running                 ./bin/tikv-server --config=tikv.toml &>tikv.out &
[3]-  Running                 ./pump --config=pump.toml &>pump.out &
```bash
[4]+ 実行中              ./bin/tidb-server --config=tidb.toml &>tidb.out &
```

サービスのうち1つが起動に失敗した場合（たとえば、「`Running`」の代わりに「`Exit 1`」が表示される場合など）、その個々のサービスを再起動してみてください。

## 接続

TiDBクラスターの4つのコンポーネントがすべて実行されており、MariaDB/MySQLのコマンドラインクライアントを使用してポート4000上のTiDBサーバに接続できるはずです。

```bash
mysql -h 127.0.0.1 -P 4000 -u root -e 'select tidb_version()\G'
```

期待される出力：

```
[kolbe@localhost tidb-latest-linux-amd64]$ mysql -h 127.0.0.1 -P 4000 -u root -e 'select tidb_version()\G'
*************************** 1. row ***************************
tidb_version(): Release Version: v3.0.0-beta.1-154-gd5afff70c
Git Commit Hash: d5afff70cdd825d5fab125c8e52e686cc5fb9a6e
Git Branch: master
UTC Build Time: 2019-04-24 03:10:00
GoVersion: go version go1.12 linux/amd64
Race Enabled: false
TiKV Min Version: 2.1.0-alpha.1-ff3dd160846b7d1aed9079c389fc188f7f5ea13e
Check Table Before Drop: false
```

この時点でTiDBクラスターが実行され、`pump`がクラスターからバイナリログを読み取り、それをリレーログとしてそのデータディレクトリに保存しています。次のステップは、`drainer`が書き込むことができるMariaDBサーバを起動することです。

次のコマンドを使用して`drainer`を起動してください。

```bash
sudo systemctl start mariadb
./drainer --config=drainer.toml &>drainer.out &
```

MySQLサーバーをより簡単にインストールできるオペレーティングシステムを使用している場合は、それでもかまいません。ただし、ポート3306でリッスンするようにしておき、ユーザー「root」で空のパスワードで接続できることを確認するか、必要に応じてdrainer.tomlを調整してください。

```bash
mysql -h 127.0.0.1 -P 3306 -u root
```

```sql
show databases;
```

期待される出力：

```
[kolbe@localhost ~]$ mysql -h 127.0.0.1 -P 3306 -u root
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 20
Server version: 5.5.60-MariaDB MariaDB Server

Copyri
```
```
      + You can use "NodeIDs" with `binlogctl` to control individual nodes. In this case, the NodeID of the drainer is "localhost.localdomain:8249" and the NodeID of the Pump is "localhost.localdomain:8250".

      The main use of `binlogctl` in this tutorial is likely to be in the event of a cluster restart. If you end all processes in the TiDB cluster and try to restart them (not including the downstream MySQL/MariaDB server or Drainer), Pump will refuse to start because it cannot contact Drainer and believe that Drainer is still "online".
      
      There are 3 solutions to this issue:
      
      - Stop Drainer using `binlogctl` instead of killing the process:
      
        ```
        ./binlogctl --pd-urls=http://127.0.0.1:2379 --cmd=drainers
        ./binlogctl --pd-urls=http://127.0.0.1:2379 --cmd=offline-drainer --node-id=localhost.localdomain:8249
        ```
      
      - Start Drainer _before_ starting Pump.
      - Use `binlogctl` after starting PD (but before starting Drainer and Pump) to update the state of the paused Drainer:
      
        ```
        ./binlogctl --pd-urls=http://127.0.0.1:2379 --cmd=update-drainer --node-id=localhost.localdomain:8249 --state=offline
        ```

      ## Cleanup

      To stop the TiDB cluster and TiDB Binlog processes, you can execute `pkill -P $$` in the shell where you started all the processes that form the cluster (pd-server, tikv-server, pump, tidb-server, drainer). To give each component enough time to shut down cleanly, it's helpful to stop them in a particular order:

      ```bash
      for p in tidb-server drainer pump tikv-server pd-server; do pkill "$p"; sleep 1; done
      ```

      Expected output:

      ```
      kolbe@localhost tidb-latest-linux-amd64]$ for p in tidb-server drainer pump tikv-server pd-server; do pkill "$p"; sleep 1; done
      [4]-  Done                    ./bin/tidb-server --config=tidb.toml &>tidb.out
      [5]+  Done                    ./drainer --config=drainer.toml &>drainer.out
      [3]+  Done                    ./pump --config=pump.toml &>pump.out
      [2]+  Done                    ./bin/tikv-server --config=tikv.toml &>tikv.out
      [1]+  Done                    ./bin/pd-server --config=pd.toml &>pd.out
      ```

      If you wish to restart the cluster after all services exit, use the same commands you ran originally to start the services. As discussed in the [`binlogctl`](#binlogctl) section above, you'll need to start `drainer` before `pump`, and `pump` before `tidb-server`.

      ```bash
      ./bin/pd-server --config=pd.toml &>pd.out &
      ./bin/tikv-server --config=tikv.toml &>tikv.out &
      ./drainer --config=drainer.toml &>drainer.out &
      sleep 3
      ./pump --config=pump.toml &>pump.out &
      sleep 3
      ./bin/tidb-server --config=tidb.toml &>tidb.out &
      ```

      If any of the components fail to start, try to restart the failed individual component(s).

      ## Conclusion

      In this tutorial, we've set up TiDB Binlog to replicate from a TiDB cluster to a downstream MariaDB server, using a cluster with a single Pump and a single Drainer. As we've seen, TiDB Binlog is a comprehensive platform for capturing and processing changes to a TiDB cluster.

      In a more robust development, testing, or production deployment, you'd have multiple TiDB servers for high availability and scaling purposes, and you'd use multiple Pump instances to ensure that application traffic to TiDB server instances is unaffected by problems in the Pump cluster. You may also use additional Drainer instances to push updates to different downstream platforms or to implement incremental backups.
```