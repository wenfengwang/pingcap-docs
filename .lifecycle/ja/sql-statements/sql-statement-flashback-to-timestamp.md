---
title: FLASHBACK CLUSTER TO TIMESTAMP
summary: FLASHBACK CLUSTER TO TIMESTAMPのTiDBデータベースでの使用方法を学びます。

# FLASHBACK CLUSTER TO TIMESTAMP

TiDB v6.4.0では`FLASHBACK CLUSTER TO TIMESTAMP`構文が導入されました。これを使用して、クラスタを特定の時点に復元できます。

> **警告：**
>
> `FLASHBACK CLUSTER TO TIMESTAMP`構文は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタには適用されません。予期しない結果を避けるために、このステートメントをTiDB Serverlessクラスタで実行しないでください。

<CustomContent platform="tidb">

> **警告：**
>
> TiDB v7.1.0でこの機能を使用すると、FLASHBACK操作が完了した後も、一部のリージョンがFLASHBACKプロセスに残る場合があります。v7.1.0でこの機能を使用しないことをお勧めします。詳細については、issue [#44292](https://github.com/pingcap/tidb/issues/44292)を参照してください。
>
> この問題に遭遇した場合は、[TiDBスナップショットバックアップとリストア](/br/br-snapshot-guide.md)機能を使用してデータを復元できます。

</CustomContent>

> ** 注意：**
>
> `FLASHBACK CLUSTER TO TIMESTAMP`の作業原理は、特定の時点の古いデータを最新のタイムスタンプで書き込み、現在のデータを削除しないことです。したがって、この機能を使用する前に、古いデータと現在のデータのために十分なストレージスペースがあることを確認する必要があります。

## 構文

```sql
FLASHBACK CLUSTER TO TIMESTAMP '2022-09-21 16:02:50';
```

### 概要

```ebnf+diagram
FlashbackToTimestampStmt ::=
    "FLASHBACK" "CLUSTER" "TO" "TIMESTAMP" stringLit
```

## ノート

* `FLASHBACK`ステートメントで指定された時刻は、Garbage Collection (GC)寿命内である必要があります。システム変数[`tidb_gc_life_time`](/system-variables.md#tidb_gc_life_time-new-in-v50)（デフォルト：`10m0s`）は、行の以前のバージョンの保持時間を定義します。ガベージコレクションが実行された最新の`safePoint`は次のクエリで取得できます：

    ```sql
    SELECT * FROM mysql.tidb WHERE variable_name = 'tikv_gc_safe_point';
    ```

<CustomContent platform='tidb'>

* `FLASHBACK CLUSTER` SQLステートメントを実行できるのは`SUPER`権限を持つユーザーのみです。
* `FLASHBACK CLUSTER`は、`ALTER TABLE ATTRIBUTE`、`ALTER TABLE REPLICA`、`CREATE PLACEMENT POLICY`など、PD関連情報を変更するDDLステートメントのロールバックをサポートしません。
* `FLASHBACK`ステートメントで指定された時刻に、完全に実行されていないDDLステートメントが存在してはいけません。そのようなDDLが存在する場合、TiDBはそれを拒否します。
* `FLASHBACK CLUSTER TO TIMESTAMP`を実行する前に、TiDBはすべて関連する接続を切断し、これらのテーブルでの読み書き操作を禁止します。その後でないと、`FLASHBACK CLUSTER`ステートメントが完了するまでです。
* `FLASHBACK CLUSTER TO TIMESTAMP`ステートメントを実行した後はキャンセルできません。TiDBは成功するまでリトライし続けます。
* `FLASHBACK CLUSTER`の実行中にデータのバックアップが必要な場合は、[バックアップ＆リストア](/br/br-snapshot-guide.md)を使用し、`FLASHBACK CLUSTER`の開始時間よりも前の`BackupTS`を指定する必要があります。さらに、`FLASHBACK CLUSTER`の実行中には、[ログバックアップ](/br/br-pitr-guide.md)を有効にすることはできません。したがって、`FLASHBACK CLUSTER`が完了した後にログバックアップを有効にしてください。
* `FLASHBACK CLUSTER`ステートメントがメタデータ（テーブル構造、データベース構造）のロールバックを引き起こす場合、関連する変更はTiCDCによって**レプリケートされません**。そのため、タスクを手動で一時停止し、`FLASHBACK CLUSTER`の完了を待ち、上流と下流のスキーマ定義を手動で一致させる必要があります。その後、TiCDC changefeedを再作成する必要があります。

</CustomContent>

<CustomContent platform='tidb-cloud'>

* `FLASHBACK CLUSTER` SQLステートメントを実行できるのは`SUPER`権限を持つユーザーのみです。
* `FLASHBACK CLUSTER`は、`ALTER TABLE ATTRIBUTE`、`ALTER TABLE REPLICA`、`CREATE PLACEMENT POLICY`など、PD関連情報を変更するDDLステートメントのロールバックをサポートしません。
* `FLASHBACK`ステートメントで指定された時刻に、完全に実行されていないDDLステートメントが存在してはいけません。そのようなDDLが存在する場合、TiDBはそれを拒否します。
* `FLASHBACK CLUSTER TO TIMESTAMP`を実行する前に、TiDBはすべて関連する接続を切断し、これらのテーブルでの読み書き操作を禁止します。その後でないと、`FLASHBACK CLUSTER`ステートメントが完了するまでです。
* `FLASHBACK CLUSTER TO TIMESTAMP`ステートメントの実行後はキャンセルできません。TiDBは成功するまでリトライし続けます。
* `FLASHBACK CLUSTER`ステートメントがメタデータ（テーブル構造、データベース構造）のロールバックを引き起こす場合、関連する変更はTiCDCによって**レプリケートされません**。そのため、タスクを手動で一時停止し、`FLASHBACK CLUSTER`の完了を待ち、上流と下流のスキーマ定義を手動で一致させる必要があります。その後、TiCDC changefeedを再作成する必要があります。

</CustomContent>

## 例

以下の例は、新しく挿入されたデータを復元する方法を示しています：

```sql
mysql> CREATE TABLE t(a INT);
Query OK, 0 rows affected (0.09 sec)

mysql> SELECT * FROM t;
Empty set (0.01 sec)

mysql> SELECT now();
+---------------------+
| now()               |
+---------------------+
| 2022-09-28 17:24:16 |
+---------------------+
1 row in set (0.02 sec)

mysql> INSERT INTO t VALUES (1);
Query OK, 1 row affected (0.02 sec)

mysql> SELECT * FROM t;
+------+
| a    |
+------+
|    1 |
+------+
1 row in set (0.01 sec)

mysql> FLASHBACK CLUSTER TO TIMESTAMP '2022-09-28 17:24:16';
Query OK, 0 rows affected (0.20 sec)

mysql> SELECT * FROM t;
Empty set (0.00 sec)
```

`FLASHBACK`ステートメントで指定された時点で完全に実行されていないDDLステートメントがある場合、`FLASHBACK`ステートメントは失敗します：

```sql
mysql> ALTER TABLE t ADD INDEX k(a);
Query OK, 0 rows affected (0.56 sec)

mysql> ADMIN SHOW DDL JOBS 1;
+--------+---------+-----------------------+------------------------+--------------+-----------+----------+-----------+---------------------+---------------------+---------------------+--------+
| JOB_ID | DB_NAME | TABLE_NAME            | JOB_TYPE               | SCHEMA_STATE | SCHEMA_ID | TABLE_ID | ROW_COUNT | CREATE_TIME         | START_TIME          | END_TIME            | STATE  |
+--------+---------+-----------------------+------------------------+--------------+-----------+----------+-----------+---------------------+---------------------+---------------------+--------+
|     84 | test    | t                     | add index /* ingest */ | public       |         2 |       82 |         0 | 2023-01-29 14:33:11 | 2023-01-29 14:33:11 | 2023-01-29 14:33:12 | synced |
+--------+---------+-----------------------+------------------------+--------------+-----------+----------+-----------+---------------------+---------------------+---------------------+--------+
1 rows in set (0.01 sec)

mysql> FLASHBACK CLUSTER TO TIMESTAMP '2023-01-29 14:33:12';
ERROR 1105 (HY000): Detected another DDL job at 2023-01-29 14:33:12 +0800 CST, can't do flashback
```

ログを通じて`FLASHBACK`の実行進捗を取得することができます。次のはその例です：

```
[2022/10/09 17:25:59.316 +08:00] [INFO] [cluster.go:463] ["flashback cluster stats"] ["complete regions"=9] ["total regions"=10] []
```

## MySQL互換性

このステートメントはMySQL構文のTiDB拡張です。