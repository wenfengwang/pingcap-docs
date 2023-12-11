---
title: SHOW [GLOBAL|SESSION] BINDINGS
summary: TiDBデータベースでのSHOW BINDINGSバインディングの使用方法。
aliases: ['/docs/dev/sql-statements/sql-statement-show-bindings/']
---

# SHOW [GLOBAL|SESSION] BINDINGS

`SHOW BINDINGS`ステートメントは、作成されたSQLバインディングに関する情報を表示するために使用されます。`BINDING`には、`GLOBAL`または`SESSION`のいずれかの基準があります。デフォルトは`SESSION`です。

## 概要

**ShowStmt:**

![ShowStmt](/media/sqlgram/ShowStmt.png)

**ShowTargetFilterable:**

![ShowTargetFilterable](/media/sqlgram/ShowTargetFilterable.png)

**GlobalScope:**

![GlobalScope](/media/sqlgram/GlobalScope.png)

**ShowLikeOrWhereOpt**

![ShowLikeOrWhereOpt](/media/sqlgram/ShowLikeOrWhereOpt.png)

## 構文の説明

{{< copyable "sql" >}}

```sql
SHOW [GLOBAL | SESSION] BINDINGS [ShowLikeOrWhereOpt];
```

このステートメントは、GLOBALまたはSESSIONレベルでの実行計画のバインディングを出力します。デフォルトのスコープはSESSIONです。現在、`SHOW BINDINGS`は以下のように8つの列を出力します。

| 列名 | 説明 |
| :---------- | :------------- |
| original_sql  |  パラメーター化後の元のSQLステートメント |
| bind_sql | ヒントを含むバインドされたSQLステートメント |
| default_db | デフォルトのデータベース |
| status | 'Using'、'Deleted'、'Invalid'、'Rejected'、'Pending verification'を含むステータス|
| create_time | 作成時刻 |
| update_time | 更新時刻 |
| charset | 文字セット |
| collation | ソート規則 |
| source | バインディングが作成された方法。`manual` (SQLステートメント`create [global] binding`によって作成された)、`capture` (TiDBによって自動的にキャプチャされた)、`evolve` (TiDBによって自動的に進化した) |

## 例

{{< copyable "sql" >}}

```sql
mysql> CREATE TABLE t1 (
          id INT NOT NULL PRIMARY KEY auto_increment,
          b INT NOT NULL,
          pad VARBINARY(255),
          INDEX(b)
         );
Query OK, 0 rows affected (0.07 sec)

mysql> INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM dual;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
Query OK, 1 row affected (0.00 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
Query OK, 8 rows affected (0.00 sec)
Records: 8  Duplicates: 0  Warnings: 0

mysql> INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
Query OK, 1000 rows affected (0.04 sec)
Records: 1000  Duplicates: 0  Warnings: 0

mysql> INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
Query OK, 100000 rows affected (1.74 sec)
Records: 100000  Duplicates: 0  Warnings: 0

mysql> INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
Query OK, 100000 rows affected (2.15 sec)
Records: 100000  Duplicates: 0  Warnings: 0

mysql> INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
Query OK, 100000 rows affected (2.64 sec)
Records: 100000  Duplicates: 0  Warnings: 0

mysql> SELECT SLEEP(1);
+----------+
| SLEEP(1) |
+----------+
|        0 |
+----------+
1 row in set (1.00 sec)

mysql> ANALYZE TABLE t1;
Query OK, 0 rows affected (1.33 sec)

mysql> EXPLAIN ANALYZE SELECT * FROM t1 WHERE b = 123;
+-------------------------------+---------+---------+-----------+----------------------+---------------------------------------------------------------------------+-----------------------------------+----------------+------+
| id                            | estRows | actRows | task      | access object        | execution info                                                            | operator info                     | memory         | disk |
+-------------------------------+---------+---------+-----------+----------------------+---------------------------------------------------------------------------+-----------------------------------+----------------+------+
| IndexLookUp_10                | 583.00  | 297     | root      |                      | time:10.545072ms, loops:2, rpc num: 1, rpc time:398.359µs, proc keys:297  |                                   | 109.1484375 KB | N/A  |
| ├─IndexRangeScan_8(Build)     | 583.00  | 297     | cop[tikv] | table:t1, index:b(b) | time:0s, loops:4                                                          | range:[123,123], keep order:false | N/A            | N/A  |
| └─TableRowIDScan_9(Probe)     | 583.00  | 297     | cop[tikv] | table:t1             | time:12ms, loops:4                                                        | keep order:false                  | N/A            | N/A  |
+-------------------------------+---------+---------+-----------+----------------------+---------------------------------------------------------------------------+-----------------------------------+----------------+------+
3 rows in set (0.02 sec)

mysql> CREATE SESSION BINDING FOR
         SELECT * FROM t1 WHERE b = 123
        USING
         SELECT * FROM t1 IGNORE INDEX (b) WHERE b = 123;
Query OK, 0 rows affected (0.00 sec)

mysql> EXPLAIN ANALYZE SELECT * FROM t1 WHERE b = 123;
+-------------------------+-----------+---------+-----------+---------------+--------------------------------------------------------------------------------+--------------------+---------------+------+
| id                      | estRows   | actRows | task      | access object | execution info                                                                 | operator info      | memory        | disk |
+-------------------------+-----------+---------+-----------+---------------+--------------------------------------------------------------------------------+--------------------+---------------+------+
| TableReader_7           | 583.00    | 297     | root      |               | time:222.32506ms, loops:2, rpc num: 1, rpc time:222.078952ms, proc keys:301010 | data:Selection_6   | 88.6640625 KB | N/A  |
| └─Selection_6           | 583.00    | 297     | cop[tikv] |               | time:224ms, loops:298                                                          | eq(test.t1.b, 123) | N/A           | N/A  |
|   └─TableFullScan_5     | 301010.00 | 301010  | cop[tikv] | table:t1      | time:220ms, loops:298                                                          | keep order:false   | N/A           | N/A  |
+-------------------------+-----------+---------+-----------+---------------+--------------------------------------------------------------------------------+--------------------+---------------+------+
3 rows in set (0.22 sec)

mysql> SHOW SESSION BINDINGS\G
*************************** 1. row ***************************
Original_sql: select * from t1 where b = ?
    Bind_sql: SELECT * FROM t1 IGNORE INDEX (b) WHERE b = 123
  Default_db: test
      Status: using
 Create_time: 2020-05-22 14:38:03.456
 Update_time: 2020-05-22 14:38:03.456
     Charset: utf8mb4
   Collation: utf8mb4_0900_ai_ci
1 row in set (0.00 sec)
```

## MySQL互換性

このステートメントはMySQL構文へのTiDBの拡張です。

## 関連情報

* [CREATE [GLOBAL|SESSION] BINDING](/sql-statements/sql-statement-create-binding.md)
* [DROP [GLOBAL|SESSION] BINDING](/sql-statements/sql-statement-drop-binding.md)
* [ANALYZE TABLE](/sql-statements/sql-statement-analyze-table.md)
* [Optimizer Hints](/optimizer-hints.md)
* [SQL Plan Management](/sql-plan-management.md)