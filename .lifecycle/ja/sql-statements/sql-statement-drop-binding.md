---
title: DROP [GLOBAL|SESSION] BINDING
summary: TiDBデータベースでのDROP BINDINGの使用方法
aliases: ['/docs/dev/sql-statements/sql-statement-drop-binding/']
---

# DROP [GLOBAL|SESSION] BINDING

このステートメントは、特定のSQLステートメントからバインディングを削除します。バインディングは、基になるクエリを変更せずにステートメントにヒントをインジェクトするために使用できます。

`BINDING` は `GLOBAL` または `SESSION` のいずれかの基になります。デフォルトは `SESSION` です。

## 構文

```ebnf+diagram
DropBindingStmt ::=
    'DROP' GlobalScope 'BINDING' 'FOR' ( BindableStmt ( 'USING' BindableStmt )?
|   'SQL' 'DIGEST' SqlDigest)

GlobalScope ::=
    ( 'GLOBAL' | 'SESSION' )?

BindableStmt ::=
    ( SelectStmt | UpdateStmt | InsertIntoStmt | ReplaceIntoStmt | DeleteStmt )
```

## 例

SQLステートメントまたは `sql_digest` に従ってバインディングを削除できます。

次の例は、SQLステートメントに従ってバインディングを削除する方法を示しています。

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
...

次の例は、`sql_digest` に従ってバインディングを削除する方法を示しています。
```sql
mysql> CREATE TABLE t(id INT PRIMARY KEY , a INT, KEY(a));
クエリが成功しました。0行が影響を受けました (0.06 秒)

mysql> SELECT /*+ IGNORE_INDEX(t, a) */ * FROM t WHERE a = 1;
空のセット (0.01 秒)

mysql> SELECT plan_digest FROM INFORMATION_SCHEMA.STATEMENTS_SUMMARY WHERE QUERY_SAMPLE_TEXT = 'SELECT /*+ IGNORE_INDEX(t, a) */ * FROM t WHERE a = 1';
+------------------------------------------------------------------+
| plan_digest                                                      |
+------------------------------------------------------------------+
| 4e3159169cc63c14b139a4e7d72eae1759875c9a9581f94bb2079aae961189cb |
+------------------------------------------------------------------+
1 行が返されました (0.01 秒)

mysql> CREATE BINDING FROM HISTORY USING PLAN DIGEST '4e3159169cc63c14b139a4e7d72eae1759875c9a9581f94bb2079aae961189cb';
クエリが成功しました。0行が影響を受けました (0.02 秒)

mysql> SELECT * FROM t WHERE a = 1;
空のセット (0.01 秒)

mysql> SELECT @@LAST_PLAN_FROM_BINDING;
+--------------------------+
| @@LAST_PLAN_FROM_BINDING |
+--------------------------+
|                        1 |
+--------------------------+
1 行が返されました (0.01 秒)

mysql> SHOW BINDINGS\G;
*************************** 1. 行目 ***************************
Original_sql: select * from `test` . `t` where `a` = ?
    Bind_sql: SELECT /*+ use_index(@`sel_1` `test`.`t` ) ignore_index(`t` `a`)*/ * FROM `test`.`t` WHERE `a` = 1
  Default_db: test
      Status: enabled
 Create_time: 2022-12-14 15:26:22.277
 Update_time: 2022-12-14 15:26:22.277
     Charset: utf8mb4
   Collation: utf8mb4_general_ci
      Source: history
  Sql_digest: 6909a1bbce5f64ade0a532d7058dd77b6ad5d5068aee22a531304280de48349f
 Plan_digest: 4e3159169cc63c14b139a4e7d72eae1759875c9a9581f94bb2079aae961189cb
1 行が返されました (0.02 秒)

ERROR:
クエリが指定されていません

mysql> DROP BINDING FOR SQL DIGEST '6909a1bbce5f64ade0a532d7058dd77b6ad5d5068aee22a531304280de48349f';
クエリが成功しました。0行が影響を受けました (0.00 秒)

mysql> SHOW BINDINGS\G;
空のセット (0.01 秒)

ERROR:
クエリが指定されていません
```

## MySQL 互換性

このステートメントは、MySQL構文へのTiDB拡張です。

## 関連情報

* [CREATE [GLOBAL|SESSION] BINDING](/sql-statements/sql-statement-create-binding.md)
* [SHOW [GLOBAL|SESSION] BINDINGS](/sql-statements/sql-statement-show-bindings.md)
* [ANALYZE TABLE](/sql-statements/sql-statement-analyze-table.md)
* [Optimizer Hints](/optimizer-hints.md)
* [SQL Plan Management](/sql-plan-management.md)