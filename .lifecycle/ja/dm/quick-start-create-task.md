---
title: データ移行タスクの作成
summary: DMクラスターのデプロイ後に移行タスクを作成する方法について学びます。
aliases: ['/docs/tidb-data-migration/dev/create-task-and-verify/']
---

# データ移行タスクの作成

このドキュメントでは、DMクラスターが正常にデプロイされた後、簡単なデータ移行タスクを作成する方法について説明します。

## サンプルシナリオ

このサンプルシナリオに基づいてデータ移行タスクを作成するとします：

- バイナリログが有効になっているMySQLインスタンス2つとTiDBインスタンス1つをローカルにデプロイ
- DMクラスターのDM-masterを使用して、クラスターとデータ移行タスクを管理します。

各ノードの情報は次のとおりです。

| インスタンス   | サーバーアドレス  | ポート  |
| :---------- | :----------- | :--- |
| MySQL1     | 127.0.0.1 | 3306 |
| MySQL2     | 127.0.0.1 | 3307 |
| TiDB       | 127.0.0.1 | 4000 |
| DM-master  | 127.0.0.1 | 8261 |

このシナリオに基づいて、以下のセクションでデータ移行タスクの作成方法を説明します。

### 上流のMySQLを開始

実行可能なMySQLインスタンス2つを準備します。また、Dockerを使用してMySQLを素早く起動することもできます。コマンドは次のとおりです。

{{< copyable "shell-regular" >}}

```bash
docker run --rm --name mysql-3306 -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7.22 --log-bin=mysql-bin --port=3306 --bind-address=0.0.0.0 --binlog-format=ROW --server-id=1 --gtid_mode=ON --enforce-gtid-consistency=true > mysql.3306.log 2>&1 &
docker run --rm --name mysql-3307 -p 3307:3307 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql:5.7.22 --log-bin=mysql-bin --port=3307 --bind-address=0.0.0.0 --binlog-format=ROW --server-id=1 --gtid_mode=ON --enforce-gtid-consistency=true > mysql.3307.log 2>&1 &
```

### データの準備

- mysql-3306にサンプルデータを書き込む:

    {{< copyable "sql" >}}

    ```sql
    drop database if exists `sharding1`;
    create database `sharding1`;
    use `sharding1`;
    create table t1 (id bigint, uid int, name varchar(80), info varchar(100), primary key (`id`), unique key(`uid`)) DEFAULT CHARSET=utf8mb4;
    create table t2 (id bigint, uid int, name varchar(80), info varchar(100), primary key (`id`), unique key(`uid`)) DEFAULT CHARSET=utf8mb4;
    insert into t1 (id, uid, name) values (1, 10001, 'Gabriel García Márquez'), (2 ,10002, 'Cien años de soledad');
    insert into t2 (id, uid, name) values (3,20001, 'José Arcadio Buendía'), (4,20002, 'Úrsula Iguarán'), (5,20003, 'José Arcadio');
    ```

- mysql-3307にサンプルデータを書き込む:

    {{< copyable "sql" >}}

    ```sql
    drop database if exists `sharding2`;
    create database `sharding2`;
    use `sharding2`;
    create table t2 (id bigint, uid int, name varchar(80), info varchar(100), primary key (`id`), unique key(`uid`)) DEFAULT CHARSET=utf8mb4;
    create table t3 (id bigint, uid int, name varchar(80), info varchar(100), primary key (`id`), unique key(`uid`)) DEFAULT CHARSET=utf8mb4;
    insert into t2 (id, uid, name, info) values (6, 40000, 'Remedios Moscote', '{}');
    insert into t3 (id, uid, name, info) values (7, 30001, 'Aureliano José', '{}'), (8, 30002, 'Santa Sofía de la Piedad', '{}'), (9, 30003, '17 Aurelianos', NULL);
    ```

### 下流のTiDBを開始

TiDBサーバーを実行するには、次のコマンドを使用します：

{{< copyable "shell-regular" >}}

```bash
wget https://download.pingcap.org/tidb-community-server-v7.4.0-linux-amd64.tar.gz
tar -xzvf tidb-latest-linux-amd64.tar.gz
mv tidb-latest-linux-amd64/bin/tidb-server ./
./tidb-server
```

> **警告:**
>
> このドキュメントでのTiDBのデプロイ方法は、**本番環境や開発環境には適用されません**。

## MySQLデータソースの構成

データ移行タスクを開始する前に、MySQLデータソースを構成する必要があります。

### パスワードの暗号化

> **注意:**
>
> + データベースにパスワードが設定されていない場合は、この手順をスキップできます。
> + DM v1.0.6以降のバージョンでは平文のパスワードを使用してソース情報を構成することができます。

安全性のため、暗号化されたパスワードの構成と使用を推奨します。MySQL/TiDBのパスワードを暗号化するには、dmctlを使用できます。パスワードが "123456" の場合:

{{< copyable "shell-regular" >}}

```bash
./dmctl encrypt "123456"
```

```
fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg=
```

この暗号化された値を保存し、次の手順でMySQLデータソースを作成する際に使用します。

### ソース構成ファイルの編集

以下の構成を `conf/source1.yaml` に書き込みます。

```yaml
# MySQL1の構成.

source-id: "mysql-replica-01"

# GTIDが有効かどうかを示します
enable-gtid: true

from:
  host: "127.0.0.1"
  user: "root"
  password: "fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg="
  port: 3306
```

MySQL2のデータソースには、上記の構成を `conf/source2.yaml` にコピーします。`name` を `mysql-replica-02` に変更し、`password` と `port` を適切な値に変更する必要があります。

### ソースの作成

dmctlを使用してMySQL1のデータソース構成をDMクラスターにロードするには、ターミナルで次のコマンドを実行します:

{{< copyable "shell-regular" >}}

```bash
./dmctl --master-addr=127.0.0.1:8261 operate-source create conf/source1.yaml
```

MySQL2の場合は、上記のコマンドの配置ファイルをMySQL2用に置き換えます。

## データ移行タスクの作成

[準備されたデータ](#prepare-data)をインポートした後、MySQL1とMySQL2の両方にいくつかのシャーディングされたテーブルがあります。これらのテーブルは同じ構造を持ち、「t」で始まるテーブル名の接頭辞が同じである、これらのテーブルが属するデータベースはすべて「sharding」で始まり、主キーまたはユニークキー間での競合はありません（各シャーディングされたテーブルで、主キーまたはユニークキーは他のテーブルと異なります）。

ここで、これらのシャーディングされたテーブルをTiDBの `db_target.t_target` テーブルに移行する必要があるとします。手順は以下の通りです。

1. タスクの構成ファイルを作成します:

    {{< copyable "" >}}

    ```yaml
    ---
    name: test
    task-mode: all
    shard-mode: "pessimistic"
    target-database:
      host: "127.0.0.1"
      port: 4000
      user: "root"
      password: "" # パスワードが空でない場合は、dmctlで暗号化されたパスワードを使用することをお勧めします。

    mysql-instances:
      - source-id: "mysql-replica-01"
        block-allow-list:  "instance"  # この構成はDM v2.0.0-beta.2よりも新しいバージョンに適用されます。それ以外の場合はブラックホワイトリストを使用してください。
        route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
        mydumper-thread: 4
        loader-thread: 16
        syncer-thread: 16
      - source-id: "mysql-replica-02"
        block-allow-list:  "instance"  # この構成はDM v2.0.0-beta.2よりも新しいバージョンに適用されます。それ以外の場合はブラックホワイトリストを使用してください。
        route-rules: ["sharding-route-rules-table", "sharding-route-rules-schema"]
        mydumper-thread: 4
        loader-thread: 16
        syncer-thread: 16
    block-allow-list:  # この構成はDM v2.0.0-beta.2よりも新しいバージョンに適用されます。それ以外の場合はブラックホワイトリストを使用してください。
      instance:
        do-dbs: ["~^sharding[\\d]+"]
        do-tables:
        - db-name: "~^sharding[\\d]+"
          tbl-name: "~^t[\\d]+"
    routes:
      sharding-route-rules-table:
        schema-pattern: sharding*
```
```yaml
        table-pattern: t*
        target-schema: db_target
        target-table: t_target
      sharding-route-rules-schema:
        schema-pattern: sharding*
        target-schema: db_target
    ```

2. `dmctl`を使用してタスクを作成するには、上記の設定を`conf/task.yaml`ファイルに記述します:

    {{< copyable "shell-regular" >}}

    ```bash
    ./dmctl --master-addr 127.0.0.1:8261 start-task conf/task.yaml
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-01",
                "worker": "worker1"
            },
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-02",
                "worker": "worker2"
            }
        ]
    }
    ```

これで、MySQL1とMySQL2のインスタンスからTiDBにシャーディングされたテーブルを移行するタスクが正常に作成されました。

## データの検証

上流のMySQLのシャードテーブルのデータを変更することができます。そして、[sync-diff-inspector](/sync-diff-inspector/shard-diff.md)を使用して上流と下流のデータが一致しているかどうかを確認できます。一致するデータとは、移行タスクが正常に動作していることを意味し、またクラスタが正常に動作していることを示しています。