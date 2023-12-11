---
title: `CREATE [GLOBAL|SESSION] BINDING`の作成
summary: TiDBデータベースにおけるCREATE BINDINGの使用方法。
aliases: ['/docs/dev/sql-statements/sql-statement-create-binding/']
---

# `CREATE [GLOBAL|SESSION] BINDING`

このステートメントは、TiDBにおいて新しい実行計画バインディングを作成します。バインディングは、基になるクエリを変更することなく、ステートメントにヒントを注入するために使用できます。

`BINDING`は`GLOBAL`または`SESSION`のいずれかの基準で設定できます。デフォルトは`SESSION`です。

バインドされたSQLステートメントはパラメーター化され、システムテーブルに保存されます。SQLクエリを処理する際に、パラメーター化されたSQLステートメントとシステムテーブル内のバインドされたステートメントが一致し、システム変数`tidb_use_plan_baselines`が`ON`に設定されている場合（デフォルト）、対応するオプティマイザーヒントが利用可能になります。複数の実行計画が利用可能な場合、オプティマイザーは最小コストの計画をバインドします。詳細については、[バインディングの作成](/sql-plan-management.md#create-a-binding)を参照してください。

## 構文

```ebnf+diagram
CreateBindingStmt ::=
    'CREATE' GlobalScope 'BINDING' ( 'FOR' BindableStmt 'USING' BindableStmt
|   'FROM' 'HISTORY' 'USING' 'PLAN' 'DIGEST' PlanDigest )

GlobalScope ::=
    ( 'GLOBAL' | 'SESSION' )?

BindableStmt ::=
    ( SelectStmt | UpdateStmt | InsertIntoStmt | ReplaceIntoStmt | DeleteStmt )
```

****

## 例

SQLステートメントまたは過去の実行計画に応じてバインディングを作成できます。

下記の例は、SQLステートメントに応じてバインディングを作成する方法を示しています。

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

mysql> DROP SESSION BINDING FOR SELECT * FROM t1 WHERE b = 123;
Query OK, 0 rows affected (0.00 sec)

mysql> EXPLAIN ANALYZE SELECT * FROM t1 WHERE b = 123;
+-------------------------------+---------+---------+-----------+----------------------+-------------------------------------------------------------------------+-----------------------------------+----------------+------+
| id                            | estRows | actRows | task      | access object        | execution info                                                          | operator info                     | memory         | disk |
+-------------------------------+---------+---------+-----------+----------------------+-------------------------------------------------------------------------+-----------------------------------+----------------+------+
| IndexLookUp_10                | 583.00  | 297     | root      |                      | time:5.31206ms, loops:2, rpc num: 1, rpc time:665.927µs, proc keys:297  |                                   | 109.1484375 KB | N/A  |
| ├─IndexRangeScan_8(Build)     | 583.00  | 297     | cop[tikv] | table:t1, index:b(b) | time:0s, loops:4                                                        | range:[123,123], keep order:false | N/A            | N/A  |
| └─TableRowIDScan_9(Probe)     | 583.00  | 297     | cop[tikv] | table:t1             | time:0s, loops:4                                                        | keep order:false                  | N/A            | N/A  |
+-------------------------------+---------+---------+-----------+----------------------+-------------------------------------------------------------------------+-----------------------------------+----------------+------+
3 行をセットしました (0.01 秒)

```

次の例では、過去の実行計画に基づいてバインディングを作成する方法を示しています。

```sql
mysql> CREATE TABLE t(id INT PRIMARY KEY , a INT, KEY(a));
クエリが実行され、行数は 0 です (0.06 秒)

mysql> SELECT /*+ IGNORE_INDEX(t, a) */ * FROM t WHERE a = 1;
空のセット (0.01 秒)

mysql> SELECT plan_digest FROM INFORMATION_SCHEMA.STATEMENTS_SUMMARY WHERE QUERY_SAMPLE_TEXT = 'SELECT /*+ IGNORE_INDEX(t, a) */ * FROM t WHERE a = 1';
+------------------------------------------------------------------+
| plan_digest                                                      |
+------------------------------------------------------------------+
| 4e3159169cc63c14b139a4e7d72eae1759875c9a9581f94bb2079aae961189cb |
+------------------------------------------------------------------+
1 行をセット (0.01 秒)

mysql> CREATE BINDING FROM HISTORY USING PLAN DIGEST '4e3159169cc63c14b139a4e7d72eae1759875c9a9581f94bb2079aae961189cb';
クエリが実行され、行数は 0 です (0.02 秒)

mysql> SELECT * FROM t WHERE a = 1;
空のセット (0.01 秒)

mysql> SELECT @@LAST_PLAN_FROM_BINDING;
+--------------------------+
| @@LAST_PLAN_FROM_BINDING |
+--------------------------+
|                        1 |
+--------------------------+
1 行をセット (0.01 秒)

```

## MySQL 互換性

このステートメントは、MySQL構文のTiDB拡張機能です。

## 関連情報

* [DROP [GLOBAL|SESSION] BINDING](/sql-statements/sql-statement-drop-binding.md)
* [SHOW [GLOBAL|SESSION] BINDINGS](/sql-statements/sql-statement-show-bindings.md)
* [ANALYZE TABLE](/sql-statements/sql-statement-analyze-table.md)
* [Optimizer Hints](/optimizer-hints.md)
* [SQL Plan Management](/sql-plan-management.md)