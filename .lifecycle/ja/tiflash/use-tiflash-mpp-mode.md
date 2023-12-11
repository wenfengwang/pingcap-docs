---
title: TiFlashのMPPモードの使用方法
summary: TiFlashのMPPモードとその使用方法について学びます。

# TiFlashのMPPモードの使用方法

<CustomContent platform="tidb">

このドキュメントでは、TiFlashの[Massively Parallel Processing (MPP)](/glossary.md#mpp)モードとその使用方法について紹介します。

</CustomContent>

<CustomContent platform="tidb-cloud">

このドキュメントでは、TiFlashの[Massively Parallel Processing (MPP)](/tidb-cloud/tidb-cloud-glossary.md#mpp)モードとその使用方法について紹介します。

</CustomContent>

TiFlashは、クエリの実行にMPPモードを使用でき、これによりクロスノードデータの交換（データのシャッフルプロセス）が計算に導入されます。 TiDBは、オプティマイザのコスト推定を使用してMPPモードを選択するかどうかを自動的に判断します。 [`tidb_allow_mpp`](/system-variables.md#tidb_allow_mpp-new-in-v50)および[`tidb_enforce_mpp`](/system-variables.md#tidb_enforce_mpp-new-in-v51)の値を変更することで選択戦略を変更できます。

次の図は、MPPモードの動作を示しています。

![mpp-mode](/media/tiflash/tiflash-mpp.png)

## MPPモードを選択するかどうかの制御

`tidb_allow_mpp`変数は、TiDBがMPPモードを選択してクエリを実行できるかどうかを制御します。 `tidb_enforce_mpp`変数は、オプティマイザのコスト推定を無視し、TiFlashのMPPモードを強制してクエリを実行するかどうかを制御します。

これら2つの変数のすべての値に対応する結果は次のとおりです。

|                        | tidb_allow_mpp=off | tidb_allow_mpp=on (デフォルト)          |
| ---------------------- | -------------------- | -------------------------------- |
| tidb_enforce_mpp=off (デフォルト) | MPPモードは使用されません。 | オプティマイザはデフォルトでコスト推定に基づいてMPPモードを選択します。 |
| tidb_enforce_mpp=on  | MPPモードは使用されません。   | TiDBはコスト推定を無視し、MPPモードを選択します。      |

たとえば、MPPモードを使用したくない場合は、次のステートメントを実行できます。

{{< copyable "sql" >}}

```sql
set @@session.tidb_allow_mpp=0;
```

TiDBのコストベースのオプティマイザにMPPモードを自動的に選択させたい場合（デフォルトで）、次のステートメントを実行できます。

{{< copyable "sql" >}}

```sql
set @@session.tidb_allow_mpp=1;
set @@session.tidb_enforce_mpp=0;
```

TiDBにオプティマイザのコスト推定を無視し、MPPモードを強制して選択させたい場合は、次のステートメントを実行できます。

{{< copyable "sql" >}}

```sql
set @@session.tidb_allow_mpp=1;
set @@session.tidb_enforce_mpp=1;
```

<CustomContent platform="tidb">

`tidb_enforce_mpp`セッション変数の初期値は、このtidb-serverインスタンスの[`enforce-mpp`](/tidb-configuration-file.md#enforce-mpp)構成値と等しい（デフォルトでは`false`）です。 TiDBクラスタ内の複数のtidb-serverインスタンスが解析クエリのみを実行し、これらのインスタンスでMPPモードが使用されることを保証したい場合は、これらのインスタンスの[`enforce-mpp`](/tidb-configuration-file.md#enforce-mpp)構成値を`true`に変更できます。

</CustomContent>

> **注記：**
>
> `tidb_enforce_mpp=1`が有効になった場合、TiDBオプティマイザはコスト推定を無視してMPPモードを選択します。ただし、MPPモードを妨げる他の要因がある場合は、TiDBはMPPモードを選択しません。これらの要因には、TiFlashレプリカの不在、TiFlashレプリカの未完成のレプリケーション、およびMPPモードでサポートされていない演算子や関数を含むステートメントが含まれます。
>
> コスト推定以外の理由でTiDBオプティマイザがMPPモードを選択できない場合、実行計画を確認するために`EXPLAIN`ステートメントを使用すると、理由を説明する警告が返されます。たとえば：

> ```sql
> set @@session.tidb_enforce_mpp=1;
> create table t(a int);
> explain select count(*) from t;
> show warnings;
> ```

> ```
> +---------+------+-----------------------------------------------------------------------------+
> | Level   | Code | Message                                                                     |
> +---------+------+-----------------------------------------------------------------------------+
> | Warning | 1105 | MPP mode may be blocked because there aren't tiflash replicas of table `t`. |
> +---------+------+-----------------------------------------------------------------------------+
> ```

## MPPモードの物理アルゴリズムのサポート

MPPモードは次の物理アルゴリズムをサポートしています: ブロードキャストハッシュ結合、シャッフルハッシュ結合、シャッフルハッシュ集約、全てのユニオン、TopN、およびLimit。オプティマイザはクエリで使用するアルゴリズムを自動的に決定します。具体的なクエリ実行計画を確認するには、`EXPLAIN`ステートメントを実行できます。`EXPLAIN`ステートメントの結果に`ExchangeSender`および`ExchangeReceiver`演算子が表示される場合、MPPモードが有効になっていることを示します。

次のステートメントは、TPC-Hテストセット内のテーブル構造を示しています。

```sql
explain select count(*) from customer c join nation n on c.c_nationkey=n.n_nationkey;
+------------------------------------------+------------+--------------+---------------+----------------------------------------------------------------------------+
| id                                       | estRows    | task         | access object | operator info                                                              |
+------------------------------------------+------------+--------------+---------------+----------------------------------------------------------------------------+
| HashAgg_23                               | 1.00       | root         |               | funcs:count(Column#16)->Column#15                                          |
| └─TableReader_25                         | 1.00       | root         |               | data:ExchangeSender_24                                                     |
|   └─ExchangeSender_24                    | 1.00       | mpp[tiflash] |               | ExchangeType: PassThrough                                                  |
|     └─HashAgg_12                         | 1.00       | mpp[tiflash] |               | funcs:count(1)->Column#16                                                  |
|       └─HashJoin_17                      | 3000000.00 | mpp[tiflash] |               | inner join, equal:[eq(tpch.nation.n_nationkey, tpch.customer.c_nationkey)] |
|         ├─ExchangeReceiver_21(Build)     | 25.00      | mpp[tiflash] |               |                                                                            |
|         │ └─ExchangeSender_20            | 25.00      | mpp[tiflash] |               | ExchangeType: Broadcast                                                    |
|         │   └─TableFullScan_18           | 25.00      | mpp[tiflash] | table:n       | keep order:false                                                           |
|         └─TableFullScan_22(Probe)        | 3000000.00 | mpp[tiflash] | table:c       | keep order:false                                                           |
+------------------------------------------+------------+--------------+---------------+----------------------------------------------------------------------------+
9 rows in set (0.00 sec)
```

上記の実行計画の例では、`ExchangeReceiver`および`ExchangeSender`演算子が含まれています。実行計画は、`nation`テーブルが読み取られた後、`ExchangeSender`演算子が各ノードに対してテーブルをブロードキャストし、`HashJoin`および`HashAgg`演算が`nation`テーブルと`customer`テーブルで実行され、結果がTiDBに返されることを示しています。

TiFlashは、ブロードキャストハッシュ結合を使用するかどうかを制御するために次の3つのグローバル/セッション変数を提供します:

- [`tidb_broadcast_join_threshold_size`](/system-variables.md#tidb_broadcast_join_threshold_size-new-in-v50): 値の単位はバイトです。テーブルサイズが変数の値未満の場合（バイト単位）、ブロードキャストハッシュ結合アルゴリズムが使用されます。それ以外の場合、シャッフルハッシュ結合アルゴリズムが使用されます。
- [`tidb_broadcast_join_threshold_count`](/system-variables.md#tidb_broadcast_join_threshold_count-new-in-v50): 値の単位は行です。結合操作のオブジェクトがサブクエリに属する場合、オプティマイザはサブクエリ結果セットのサイズを推定できませんので、サイズは結果セットの行数で決まります。サブクエリの推定行数がこの変数の値未満の場合、ブロードキャストハッシュ結合アルゴリズムが使用されます。それ以外の場合、シャッフルハッシュ結合アルゴリズムが使用されます。
- [`tidb_prefer_broadcast_join_by_exchange_data_size`](/system-variables.md#tidb_prefer_broadcast_join_by_exchange_data_size-new-in-v710): ネットワーク伝送のオーバーヘッドが最小限に抑えられるアルゴリズムを使用するかどうかを制御します。この変数が有効になると、TiDBは`Broadcast Hash Join`および`Shuffled Hash Join`それぞれのデータのサイズを推定し、小さいサイズのものを選択します。この変数が有効になると、[`tidb_broadcast_join_threshold_count`](/system-variables.md#tidb_broadcast_join_threshold_count-new-in-v50)および[`tidb_broadcast_join_threshold_size`](/system-variables.md#tidb_broadcast_join_threshold_size-new-in-v50)は無効になります。

## MPPモードでパーティションテーブルにアクセスする

MPP（Massively Parallel Processing）モードで分割テーブルにアクセスするためには、まず[ダイナミックプルーニングモード](https://docs.pingcap.com/tidb/stable/partitioned-table#dynamic-pruning-mode)を有効にする必要があります。

例：

```sql
mysql> DROP TABLE if exists test.employees;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> CREATE TABLE test.employees
(id int(11) NOT NULL,
 fname varchar(30) DEFAULT NULL,
 lname varchar(30) DEFAULT NULL,
 hired date NOT NULL DEFAULT '1970-01-01',
 separated date DEFAULT '9999-12-31',
 job_code int DEFAULT NULL,
 store_id int NOT NULL) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
PARTITION BY RANGE (store_id)
(PARTITION p0 VALUES LESS THAN (6),
 PARTITION p1 VALUES LESS THAN (11),
 PARTITION p2 VALUES LESS THAN (16),
 PARTITION p3 VALUES LESS THAN (MAXVALUE));
Query OK, 0 rows affected (0.10 sec)

mysql> ALTER table test.employees SET tiflash replica 1;
Query OK, 0 rows affected (0.09 sec)

mysql> SET tidb_partition_prune_mode=static;
Query OK, 0 rows affected (0.00 sec)

mysql> explain SELECT count(*) FROM test.employees;
+----------------------------------+----------+-------------------+-------------------------------+-----------------------------------+
| id                               | estRows  | task              | access object                 | operator info                     |
+----------------------------------+----------+-------------------+-------------------------------+-----------------------------------+
| HashAgg_18                       | 1.00     | root              |                               | funcs:count(Column#10)->Column#9  |
| └─PartitionUnion_20              | 4.00     | root              |                               |                                   |
|   ├─StreamAgg_35                 | 1.00     | root              |                               | funcs:count(Column#12)->Column#10 |
|   │ └─TableReader_36             | 1.00     | root              |                               | data:StreamAgg_26                 |
|   │   └─StreamAgg_26             | 1.00     | batchCop[tiflash] |                               | funcs:count(1)->Column#12         |
|   │     └─TableFullScan_34       | 10000.00 | batchCop[tiflash] | table:employees, partition:p0 | keep order:false, stats:pseudo    |
|   ├─StreamAgg_52                 | 1.00     | root              |                               | funcs:count(Column#14)->Column#10 |
|   │ └─TableReader_53             | 1.00     | root              |                               | data:StreamAgg_43                 |
|   │   └─StreamAgg_43             | 1.00     | batchCop[tiflash] |                               | funcs:count(1)->Column#14         |
|   │     └─TableFullScan_51       | 10000.00 | batchCop[tiflash] | table:employees, partition:p1 | keep order:false, stats:pseudo    |
|   ├─StreamAgg_69                 | 1.00     | root              |                               | funcs:count(Column#16)->Column#10 |
|   │ └─TableReader_70             | 1.00     | root              |                               | data:StreamAgg_60                 |
|   │   └─StreamAgg_60             | 1.00     | batchCop[tiflash] |                               | funcs:count(1)->Column#16         |
|   │     └─TableFullScan_68       | 10000.00 | batchCop[tiflash] | table:employees, partition:p2 | keep order:false, stats:pseudo    |
|   └─StreamAgg_86                 | 1.00     | root              |                               | funcs:count(Column#18)->Column#10 |
|     └─TableReader_87             | 1.00     | root              |                               | data:StreamAgg_77                 |
|       └─StreamAgg_77             | 1.00     | batchCop[tiflash] |                               | funcs:count(1)->Column#18         |
|         └─TableFullScan_85       | 10000.00 | batchCop[tiflash] | table:employees, partition:p3 | keep order:false, stats:pseudo    |
+----------------------------------+----------+-------------------+-------------------------------+-----------------------------------+
18 rows in set (0,00 sec)

mysql> SET tidb_partition_prune_mode=dynamic;
Query OK, 0 rows affected (0.00 sec)

mysql> explain SELECT count(*) FROM test.employees;
+------------------------------+----------+--------------+-----------------+---------------------------------------------------------+
| id                           | estRows  | task         | access object   | operator info                                           |
+------------------------------+----------+--------------+-----------------+---------------------------------------------------------+
| HashAgg_17                   | 1.00     | root         |                 | funcs:count(Column#11)->Column#9                        |
| └─TableReader_19             | 1.00     | root         | partition:all   | data:ExchangeSender_18                                  |
|   └─ExchangeSender_18        | 1.00     | mpp[tiflash] |                 | ExchangeType: PassThrough                               |
|     └─HashAgg_8              | 1.00     | mpp[tiflash] |                 | funcs:count(1)->Column#11                               |
|       └─TableFullScan_16     | 10000.00 | mpp[tiflash] | table:employees | keep order:false, stats:pseudo, PartitionTableScan:true |
+------------------------------+----------+--------------+-----------------+---------------------------------------------------------+
5 rows in set (0,00 sec)
```