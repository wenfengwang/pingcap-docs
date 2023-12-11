---
title: TiDBへの小規模データセットのMySQLシャードのマイグレーションとマージ
summary: MySQLからTiDBへの小規模データセットのシャードの移行とマージ方法を学びましょう。
aliases: ['/tidb/dev/usage-scenario-shard-merge/', '/tidb/dev/usage-scenario-simple-migration/']
---

# TiDBへの小規模データセットのMySQLシャードのマイグレーションとマージ

1 TiB以下のデータを指すこのドキュメントでは、複数のMySQLデータベースインスタンスをTiDBデータベースに移行してマージする方法について説明します。このドキュメントでは、「小規模データセット」は通常、1 TiB前後またはそれ以下のデータを意味します。このドキュメントの例を通じて、移行の手順、注意事項、トラブルシューティングについて学ぶことができます。

このドキュメントは、合計1 TiB未満のMySQLシャードの移行に適用されます。合計1 TiBを超えるMySQLシャードを移行する場合、DMのみを使用して移行すると時間がかかるため、移行を行うには[Migrate and Merge MySQL Shards of Large Datasets to TiDB](/migrate-large-mysql-shards-to-tidb.md)で紹介されている操作に従うことを推奨します。

このドキュメントでは、移行手順を説明するために単純な例を取り上げます。例では、2つのデータソースMySQLインスタンスのMySQLシャードを、下流のTiDBクラスタに移行します。

この例では、MySQLインスタンス1とMySQLインスタンス2の両方に、以下のスキーマとテーブルが含まれています。この例では、両方のインスタンスの`store_01`および`store_02`スキーマの`sale`接頭辞を持つテーブルを、下流の`store`スキーマの`sale`テーブルに移行しマージします。

| スキーマ | テーブル |
|:------|:------|
| store_01 | sale_01, sale_02 |
| store_02 | sale_01, sale_02 |

対象のスキーマおよびテーブル：

| スキーマ | テーブル |
|:------|:------|
| store | sale |

## 前提条件

移行を開始する前に、以下のタスクを完了していることを確認してください:

- [TiUPを使用したDMクラスターのデプロイ](/dm/deploy-a-dm-cluster-using-tiup.md)
- [DM-workerが必要とする特権](/dm/dm-worker-intro.md)

### シャードされたテーブルの衝突をチェック

移行には異なるシャードされたテーブルのデータ統合が含まれる場合、プライマリキーまたはユニークインデックスの衝突が発生する可能性があります。したがって、移行前に現在のシャーディングスキームをビジネス視点から注意深く見直し、衝突を避ける方法を見つける必要があります。詳細については、[複数のシャードされたテーブル間のプライマリキーやユニークインデックスの衝突を処理する](/dm/shard-merge-best-practices.md#handle-conflicts-between-primary-keys-or-unique-indexes-across-multiple-sharded-tables)を参照してください。以下に簡単に説明します。

この例では、`sale_01`および`sale_02`は以下のテーブル構造を持ちます。

{{< copyable "sql" >}}

```sql
CREATE TABLE `sale_01` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `sid` bigint(20) NOT NULL,
  `pid` bigint(20) NOT NULL,
  `comment` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `sid` (`sid`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

`id`列はプライマリキーであり、`sid`列はシャーディングキーです。`id`列は自動増分であり、重複する複数のシャードされたテーブル範囲はデータ衝突を引き起こします。`sid`はインデックスがグローバルに一意であることを保証し、そのため、[自動増分プライマリキーのプライマリキー属性を削除する](/dm/shard-merge-best-practices.md#remove-the-primary-key-attribute-from-the-column)の手順に従って、`id`列をバイパスします。

{{< copyable "sql" >}}

```sql
CREATE TABLE `sale` (
  `id` bigint(20) NOT NULL,
  `sid` bigint(20) NOT NULL,
  `pid` bigint(20) NOT NULL,
  `comment` varchar(255) DEFAULT NULL,
  INDEX (`id`),
  UNIQUE KEY `sid` (`sid`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

## ステップ1. データソースの読み込み

`source1.yaml`という新しいデータソースファイルを作成し、その内容を次のように構成します。

{{< copyable "shell-regular" >}}

```yaml
# 設定
source-id: "mysql-01" # ユニークである必要があります。
# DM-workerがGTID（Global Transaction Identifier）を使用してbinlogを取得するかどうかを指定します。
# 前提条件は、上流のMySQLでGTIDをすでに有効にしている必要があります。
# 上流データベースサービスを異なるノード間で自動的にマスターを切り替えるように構成している場合、GTIDを有効にする必要があります。
enable-gtid: true
from:
  host: "${host}"           # 例: 172.16.10.81
  user: "root"
  password: "${password}"   # 平文のパスワードはサポートされていますが非推奨です。平文のパスワードはdmctl encryptを使用して暗号化することをお勧めします。
  port: ${port}             # 例: 3306
```

ターミナルで次のコマンドを実行します。`tiup dmctl`を使用してデータソースの構成をDMクラスターに読み込みます。

{{< copyable "shell-regular" >}}

```shell
tiup dmctl --master-addr ${advertise-addr} operate-source create source1.yaml
```

パラメータは次のように説明されています。

|パラメータ      | 説明 |
|-              |-            |
|`--master-addr`         | クラスター内のdmctlが接続する任意のDM-masterノードの `{advertise-addr}`。例: 172.16.10.71:8261|
|`operate-source create` | DMクラスターにデータソースをロードします。 |

すべてのデータソースがDMクラスターに追加されるまで、上記の手順を繰り返してください。

## ステップ2. 移行タスクの構成

`task1.yaml`というタスク構成ファイルを作成し、次の内容を記述します。

{{< copyable "shell-regular" >}}

```yaml
name: "shard_merge"               # タスクの名前。グローバルで一意である必要があります。
# タスクモード。次のように設定できます:
# - full: 完全なデータ移行のみを実行します（増分レプリケーションはスキップされます）
# - incremental: バイナリログを使用したリアルタイムの増分レプリケーションのみを実行します（完全なデータ移行はスキップされます）
# - all: 完全なデータ移行と増分レプリケーションの両方を実行します。ここでは、少量から中程度のデータの移行にはこのオプションを使用します。
task-mode: all
# MySQLシャード用に必要です。デフォルトでは、「pessimistic」モードが使用されます。
# 楽観的モードの原理と使用制限についての深い理解がある場合は、「optimistic」モードも使用できます。
# 詳細については、[シャードされたテーブルからのデータのマージと移行](https://docs.pingcap.com/tidb/dev/feature-shard-merge/)を参照してください。
shard-mode: "pessimistic"
meta-schema: "dm_meta"                        # メタデータを格納するために下流データベースにスキーマが作成されます
ignore-checking-items: ["auto_increment_ID"]  # この例では、上流に自動増分のプライマリキーがあるため、この項目をチェックする必要はありません。

target-database:
  host: "${host}"                             # 例: 192.168.0.1
  port: 4000
  user: "root"
  password: "${password}"                     # 平文のパスワードはサポートされていますが非推奨です。平文のパスワードはdmctl encryptを使用して暗号化することをお勧めします。

mysql-instances:
  -
    source-id: "mysql-01"                                    # データソースのID。これはsource1.yaml内のsource-idです。
    route-rules: ["sale-route-rule"]                         # データソースに適用されるテーブルルートルール
    filter-rules: ["store-filter-rule", "sale-filter-rule"]  # データソースに適用されるbinlogイベントフィルタールール
    block-allow-list:  "log-bak-ignored"                     # データソースに適用されるブロックリストおよび許可リストのルール
  -
    source-id: "mysql-02"
    route-rules: ["sale-route-rule"]
    filter-rules: ["store-filter-rule", "sale-filter-rule"]
    block-allow-list:  "log-bak-ignored"

# MySQLシャードのマージ構成
routes:
  sale-route-rule:
    schema-pattern: "store_*"                               # store_01とstore_02のスキーマを下流のstoreスキーマにマージします
    table-pattern: "sale_*"                                 # store_01とstore_02のテーブルsale_01およびsale_02を下流のsaleテーブルにマージします
    target-schema: "store"
    target-table:  "sale"
    # オプション。シャードされたスキーマとテーブルのソース情報を抽出し、抽出した情報を下流のユーザー定義の列に書き込むために使用されます。これらのオプションを構成する場合は、下流にマージされたテーブルを手動で作成する必要があります。詳細については、以下のテーブルルート設定を参照してください。
    # extract-table:                                        # sale_の部分を除いたテーブル名のサフィックスをc-table列に抽出および書き込みます。例えば、shardedテーブルsale_01のc-table列に01を抽出して書き込みます。
    #   table-regexp: "sale_(.*)"
```yaml
    #   target-column: "c_table"
    # extract-schema:                                       # Extracts and writes the schema name suffix without the store_ part to the c_schema column of the merged table. For example, 02 is extracted and written to the c_schema column for the sharded schema store_02.
    #   schema-regexp: "store_(.*)"
    #   target-column: "c_schema"
    # extract-source:                                       # Extracts and writes the source instance information to the c_source column of the merged table. For example, mysql-01 is extracted and written to the c_source column for the data source mysql-01.
    #   source-regexp: "(.*)"
    #   target-column: "c_source"

# Filters out some DDL events.
filters:
  sale-filter-rule:           # Filter name.
    schema-pattern: "store_*" # The binlog events or DDL SQL statements of upstream MySQL instance schemas that match schema-pattern are filtered by the rules below.
    table-pattern: "sale_*"   # The binlog events or DDL SQL statements of upstream MySQL instance tables that match table-pattern are filtered by the rules below.
    events: ["truncate table", "drop table", "delete"]   # The binlog event array.
    action: Ignore                                       # The string (`Do`/`Ignore`). `Do` is the allow list. `Ignore` is the block list.
  store-filter-rule:
    schema-pattern: "store_*"
    events: ["drop database"]
    action: Ignore

# Block and allow list
block-allow-list:           # filter or only migrate all operations of some databases or some tables.
  log-bak-ignored:          # Rule name.
    do-dbs: ["store_*"]     # The allow list of the schemas to be migrated, similar to replicate-do-db in MySQL.
```

上記の例は、マイグレーションタスクを実行するための最小限の構成です。詳細については、[DM Advanced Task Configuration File](/dm/task-configuration-file-full.md) を参照してください。

タスクファイル内の`routes`、`filters`、その他の設定に関する詳細情報については、以下のドキュメントを参照してください。

- [Table routing](/dm/dm-table-routing.md)
- [Block & Allow Table Lists](/dm/dm-block-allow-table-lists.md)
- [Binlog event filter](/filter-binlog-event.md)
- [Filter Certain Row Changes Using SQL Expressions](/filter-dml-event.md)

## ステップ 3. タスクを開始

マイグレーションタスクを開始する前に、`tiup dmctl` の `check-task` サブコマンドを実行して、構成が DM の要件を満たしているかどうかをチェックし、可能なエラーを回避してください。

{{< copyable "shell-regular" >}}

```shell
tiup dmctl --master-addr ${advertise-addr} check-task task.yaml
```

以下のコマンドを `tiup dmctl` で実行して、マイグレーションタスクを開始します。

{{< copyable "shell-regular" >}}

```shell
tiup dmctl --master-addr ${advertise-addr} start-task task.yaml
```

| パラメータ | 説明 |
|-|-|
|`--master-addr`| dmctl が接続するクラスタ内の任意の DM-master ノードの `{advertise-addr}`。 例：172.16.10.71:8261 |
|`start-task`   | データマイグレーションタスクを開始します。 |

マイグレーションタスクを開始できない場合は、エラー情報に従って構成情報を修正し、再度 `start-task task.yaml` を実行してマイグレーションタスクを開始してください。問題が発生した場合は、[エラーの処理](/dm/dm-error-handling.md) および [FAQ](/dm/dm-faq.md) を参照してください。

## ステップ 4. タスクを確認

マイグレーションタスクを開始した後、`dmtcl tiup` を使用して `query-status` を実行して、タスクのステータスを表示できます。

{{< copyable "shell-regular" >}}

```shell
tiup dmctl --master-addr ${advertise-addr} query-status ${task-name}
```

エラーが発生した場合は、`query-status ${task-name}` を使用してより詳細な情報を表示します。`query-status` コマンドのクエリ結果、タスクステータス、およびサブタスクのステータスの詳細については、[TiDB データマイグレーションクエリステータス](/dm/dm-query-status.md) を参照してください。

## ステップ 5. タスクを監視し、ログを確認します（オプション）

Grafana またはログを介して、マイグレーションタスクの履歴や内部の操作メトリクスを表示できます。

- Grafana 経由

    TiUP を使用して DM クラスタを展開する際に PROMetheus、Alertmanager、および Grafana が正しく展開されている場合、Grafana で DM モニタリングメトリクスを表示できます。具体的には、Grafana で IP アドレスと展開時に指定したポートを入力し、DM ダッシュボードを選択します。

- ログを通じて

    DM が実行されている場合、DM-master、DM-worker、および dmctl は、マイグレーションタスクに関する情報を含むログを出力します。各コンポーネントのログディレクトリは次のとおりです。

    - DM-master のログディレクトリ：DM-master プロセスのパラメータ `--log-file` で指定されます。TiUP を使用して DM を展開する場合、ログディレクトリは `/dm-deploy/dm-master-8261/log/` です。
    - DM-worker のログディレクトリ：DM-worker プロセスのパラメータ `--log-file` で指定されます。TiUP を使用して DM を展開する場合、ログディレクトリは `/dm-deploy/dm-worker-8262/log/` です。

## 関連情報

- [大規模データセットの MySQL シャードを TiDB にマイグレーションおよび統合](/migrate-large-mysql-shards-to-tidb.md)
- [分割されたテーブルからデータをマージおよびマイグレーション](/dm/feature-shard-merge.md)
- [シャードマージシナリオでのデータマイグレーションのベストプラクティス](/dm/shard-merge-best-practices.md)
- [エラーの処理](/dm/dm-error-handling.md)
- [パフォーマンスの問題の処理](/dm/dm-handle-performance-issues.md)
- [FAQ](/dm/dm-faq.md)
```