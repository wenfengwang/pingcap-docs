---
title: TiDB Binlog Configuration File
summary: TiDB Binlogの構成項目を学ぶ
aliases: ['/docs/dev/tidb-binlog/tidb-binlog-configuration-file/','/docs/dev/reference/tidb-binlog/config/']
---

# TiDB Binlog Configuration File（TiDB Binlog構成ファイル）

このドキュメントでは、TiDB Binlogの構成項目について紹介します。

## Pump

このセクションでは、Pumpの構成項目について紹介します。完全なPump構成ファイルの例については、[Pump Configuration](https://github.com/pingcap/tidb-binlog/blob/master/cmd/pump/pump.toml)を参照してください。

### addr

- `host:port`形式のHTTP APIのリスニングアドレスを指定します。
- デフォルト値: `127.0.0.1:8250`

### advertise-addr

- 外部からアクセス可能なHTTP APIアドレスを指定します。このアドレスは、`host:port`形式でPDに登録されます。
- デフォルト値: `127.0.0.1:8250`

### socket

- HTTP APIがリスンするUnixソケットアドレスです。
- デフォルト値: ""

### pd-urls

- PD URLのカンマ区切りリストを指定します。複数のアドレスが指定されている場合、PDクライアントが1つのアドレスに接続できないとき、自動的に別のアドレスに接続を試みます。
- デフォルト値: `http://127.0.0.1:2379`

### data-dir

- ローカルにbinlogおよびそのインデックスが保存されるディレクトリを指定します。
- デフォルト値: `data.pump`

### heartbeat-interval

- PDに最新のステータスを報告するハートビート間隔（秒単位）を指定します。
- デフォルト値: `2`

### gen-binlog-interval

- データが偽のbinlogに書き込まれる間隔（秒単位）を指定します。
- デフォルト値: `3`

### gc

- ローカルに保存されたbinlogを保持できる日数（整数）を指定します。指定された日数よりも長く保存されているbinlogは自動的に削除されます。
- デフォルト値: `7`

### log-file

- ログファイルが保存されるパスを指定します。パラメータが空の値に設定されている場合、ログファイルは保存されません。
- デフォルト値: ""

### log-level

- ログレベルを指定します。
- デフォルト値: `info`

### node-id

- PumpノードIDを指定します。このIDにより、このPumpプロセスをクラスター内で識別できます。
- デフォルト値: `hostname:port number`。例: `node-1:8250`

### security

このセクションでは、セキュリティに関連する構成項目が紹介されます。

#### ssl-ca

- 信頼されたSSL証明書リストまたはCAリストのファイルパスを指定します。例: `/path/to/ca.pem`。
- デフォルト値: ""

#### ssl-cert

- PEM形式でエンコードされたX509証明書ファイルのパスを指定します。例: `/path/to/pump.pem`。
- デフォルト値: ""

#### ssl-key

- PEM形式でエンコードされたX509キーファイルのパスを指定します。例: `/path/to/pump-key.pem`。
- デフォルト値: ""

### storage

このセクションでは、ストレージに関連する構成項目が紹介されます。

#### sync-log

- 各バッチのbinlog書き込み後に`fsync`を使用するかどうかを指定します。これによりデータの安全性が確保されます。
- デフォルト値: `true`

#### kv_chan_cap

- Pumpがこれらのリクエストを受け取る前にバッファに格納できる書き込みリクエストの数を指定します。
- デフォルト値: `1048576`（つまり、2の20乗）

#### slow_write_threshold

- 1つのbinlogファイルを書き込むのに指定された閾値よりも長時間かかった場合、その書き込みは遅い書き込みと見なされ、ログに`"take a long time to write binlog"`が出力されます（秒単位での閾値）。
- デフォルト値: `1`

#### stop-write-at-available-space

- 使用可能なストレージ容量がこの指定された値よりも少ない場合、binlog書き込みリクエストはもはや受け付けられません。`900 MB`、`5 GB`、`12 GiB`などの形式を使用してストレージ容量を指定できます。クラスターに複数のPumpノードがある場合、Pumpノードがストレージ容量が不足して書き込みリクエストを拒否した場合、TiDBは自動的に他のPumpノードにbinlogを書き込みます。
- デフォルト値: `10 GiB`

#### kv

Pumpのストレージは現在[GoLevelDB](https://github.com/syndtr/goleveldb)を基に実装されています。`storage`の下には、GoLevel構成を調整するために使用される`kv`サブグループもあります。サポートされている構成項目は以下の通りです:

- block-cache-capacity
- block-restart-interval
- block-size
- compaction-L0-trigger
- compaction-table-size
- compaction-total-size
- compaction-total-size-multiplier
- write-buffer
- write-L0-pause-trigger
- write-L0-slowdown-trigger

上記項目の詳細な説明については、[GoLevelDB Document](https://godoc.org/github.com/syndtr/goleveldb/leveldb/opt#Options)を参照してください。

## Drainer

このセクションでは、Drainerの構成項目について紹介します。完全なDrainer構成ファイルの例については、[Drainer Configuration](https://github.com/pingcap/tidb-binlog/blob/master/cmd/drainer/drainer.toml)を参照してください。

### addr

- `host:port`形式のHTTP APIのリスニングアドレスを指定します。
- デフォルト値: `127.0.0.1:8249`

### advertise-addr

- 外部からアクセス可能なHTTP APIアドレスを指定します。このアドレスは、`host:port`形式でPDに登録されます。
- デフォルト値: `127.0.0.1:8249`

### log-file

- ログファイルが保存されるパスを指定します。パラメータが空の値に設定されている場合、ログファイルは保存されません。
- デフォルト値: ""

### log-level

- ログレベルを指定します。
- デフォルト値: `info`

### node-id

- DrainerノードIDを指定します。このIDにより、このDrainerプロセスをクラスター内で識別できます。
- デフォルト値: `hostname:port number`。例: `node-1:8249`

### data-dir

- Drainer操作中に保存するファイルを保存するディレクトリを指定します。
- デフォルト値: `data.drainer`

### detect-interval

- PDがPump情報を更新する間隔（秒単位）を指定します。
- デフォルト値: `5`

### pd-urls

- PD URLのカンマ区切りリストを指定します。複数のアドレスが指定されている場合、PDクライアントは1つのアドレスに接続中にエラーが発生した場合、自動的に別のアドレスに接続を試みます。
- デフォルト値: `http://127.0.0.1:2379`

### initial-commit-ts

- レプリケーションプロセスの開始時のコミットタイムスタンプ（コミットタイムスタンプ）を指定します。この構成は、初めてレプリケーションプロセスを行うDrainerノードにのみ適用されます。下流で既にチェックポイントが存在する場合、チェックポイントに記録された時間に従ってレプリケーションが行われます。
- コミットタイムスタンプ（commit timestamp）はTiDBにおける[トランザクション](/transaction-overview.md#transactions)のコミットの特定の時間点であり、現在のトランザクションの一意のIDとしてPDから提供されるグローバルに一意で増加するタイムスタンプです。`initial-commit-ts`の構成は以下の典型的な方法で取得できます:
    - BRを使用する場合、BRによってバックアップされたメタデータ（backupmeta）に記録されたバックアップTSから`initial-commit-ts`を取得できます。
    - Dumplingを使用する場合、Dumplingによってバックアップされたメタデータ（metadata）に記録されたPosから`initial-commit-ts`を取得できます。
    - PD Controlを使用する場合、`initial-commit-ts`は`tso`コマンドの出力に含まれています。
- デフォルト値: `-1`。DrainerはPDから開始時の新しいタイムスタンプを取得します。これにより、レプリケーションプロセスは現在時刻から開始されます。

### synced-check-time

- HTTP APIを介して`/status`パスにアクセスし、Drainerレプリケーションのステータスをクエリすることができます。`synced-check-time`は、直近の成功したレプリケーションから何分が`synced`と見なされるかを指定します。
- デフォルト値: `5`

### compressor

- PumpとDrainer間のデータ転送に使用する圧縮アルゴリズムを指定します。現在は`gzip`アルゴリズムのみがサポートされています。
- デフォルト値: ""（圧縮なし）

### security

このセクションでは、セキュリティに関連する構成項目が紹介されます。

#### ssl-ca

- 信頼されたSSL証明書リストまたはCAリストのファイルパスを指定します。例: `/path/to/ca.pem`。
- デフォルト値: ""

#### ssl-cert

- PEM形式でエンコードされたX509証明書ファイルのパスを指定します。例: `/path/to/drainer.pem`。
- デフォルト値: ""

#### ssl-key

- PEM形式でエンコードされたX509キーファイルのパスを指定します。例: `/path/to/pump-key.pem`。
- デフォルト値: ""

### syncer

`syncer`セクションには、下流に関連する構成項目が含まれます。

#### db-type

現在、以下の下流タイプがサポートされています:

- `mysql`
- `tidb`
- `kafka`
- `file`

デフォルト値: `mysql`

#### sql-mode
```yaml
+ 指定当下游为 `mysql` 或 `tidb` 类型时的 SQL 模式。如果有多个模式，请用逗号分隔。
+ 默认值: ""
    + ignore-txn-commit-ts
      + 指定要忽略 binlog 的提交时间戳，例如 `[416815754209656834, 421349811963822081]`。
      + 默认值: `[]`
    + ignored-schemas
      + 指定在复制过程中要忽略的数据库。如果要忽略多个数据库，请用逗号分隔。如果一个 binlog 文件中的所有更改都被过滤，则整个 binlog 文件将被忽略。
      + 默认值: `INFORMATION_SCHEMA,PERFORMANCE_SCHEMA,mysql`
    + ignore-table
      + 在复制过程中忽略指定表的更改。您可以在 `toml` 文件中指定要忽略的多个表。例如:

{{< copyable "" >}}

```toml
[[syncer.ignore-table]]
db-name = "test"
tbl-name = "log"

[[syncer.ignore-table]]
db-name = "test"
tbl-name = "audit"
```
      + 如果一个 binlog 文件中的所有更改都被过滤，则整个 binlog 文件将被忽略。
      + 默认值: `[]`
    + replicate-do-db
      + 指定要复制的数据库。例如，`[db1, db2]`。
      + 默认值: `[]`
    + replicate-do-table
      + 指定要复制的表。例如:

{{< copyable "" >}}

```toml
[[syncer.replicate-do-table]]
db-name ="test"
tbl-name = "log"

[[syncer.replicate-do-table]]
db-name ="test"
tbl-name = "~^a.*"
```
      + 默认值: `[]`
    + txn-batch
      + 当下游为 `mysql` 或 `tidb` 类型时，执行 DML 操作时将其分批处理。此参数指定每个事务中可以包含多少个 DML 操作。
      + 默认值: `20`
    + worker-count
      + 当下游为 `mysql` 或 `tidb` 类型时，执行 DML 操作时将其并发执行。此参数指定 DML 操作的并发数。
      + 默认值: `16`
    + disable-dispatch
      + 禁用并发，并强制将 `worker-count` 设置为 `1`。
      + 默认值: `false`
    + safe-mode
      + 如果启用安全模式，Drainer 将以以下方式修改复制更新:
        + `Insert` 被修改为 `Replace Into` 
        + `Update` 被修改为 `Delete` 加 `Replace Into`
      + 默认值: `false`
  + syncer.to
    + `syncer.to` 部分根据配置类型介绍了与连接下游数据库相关的不同类型的配置项。
      + mysql/tidb
        + 下游数据库连接的以下配置项:
          + `host`: 如果未设置此项，TiDB Binlog 尝试检查默认为 `localhost` 的 `MYSQL_HOST` 环境变量。
          + `port`: 如果未设置此项，TiDB Binlog 尝试检查默认为 `3306` 的 `MYSQL_PORT` 环境变量。
          + `user`: 如果未设置此项，TiDB Binlog 尝试检查默认为 `root` 的 `MYSQL_USER` 环境变量。
          + `password`: 如果未设置此项，TiDB Binlog 尝试检查默认为 `""` 的 `MYSQL_PSWD` 环境变量。
          + `read-timeout`: 指定下游数据库连接的 I/O 读取超时。默认值为 `1m`。如果 Drainer 在执行时间较长的某些 DDL 时一直失败，您可以将此配置设置为更大的值。
      + file
        + `dir`: 指定存储 binlog 文件的目录。如果未设置此项，则使用 `data-dir`。
      + kafka
        + 当下游为 Kafka 时，有效的配置项如下:
          + `zookeeper-addrs`
          + `kafka-addrs`
          + `kafka-version`
          + `kafka-max-messages`
          + `kafka-max-message-size`
          + `topic-name`
  + syncer.to.checkpoint
    + `type`: 指定保存复制进度的方式。当前可用选项为 `mysql`、`tidb` 和 `file`。
        + 此配置项默认与下游类型相同。例如，当下游为 `file` 时，检查点进度保存在本地文件 `<data-dir>/savepoint`；当下游为 `mysql` 时，进度保存在下游数据库中。如果需要显式指定使用 `mysql` 或 `tidb` 来存储进度，请进行以下配置:
    + `schema`: 默认值为 `"tidb_binlog"`。
        > **注意:**
        >
        > 在同一 TiDB 集群中部署多个 Drainer 节点时，需要为每个节点指定不同的检查点模式。否则，两个实例的复制进度会互相覆盖。
    + `host`
    + `user`
    + `password`
    + `port`
```