---
title: ALTER INDEX
summary: ALTER INDEXのTiDBデータベースでの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-alter-index/']
---

# ALTER INDEX

`ALTER INDEX`ステートメントは、インデックスの可視性を`Visible`または`Invisible`に変更するために使用されます。Invisibleの場合、インデックスはDMLステートメントによって維持されますが、クエリオプティマイザによって使用されません。これは、永久にインデックスを削除する前に再確認したい場合に便利です。

## 構文

```ebnf+diagram
AlterTableStmt
         ::= 'ALTER' 'IGNORE'? 'TABLE' TableName AlterIndexSpec ( ',' AlterIndexSpec )*

AlterIndexSpec
         ::= 'ALTER' 'INDEX' Identifier ( 'VISIBLE' | 'INVISIBLE' )
```

## 例

`ALTER TABLE ... ALTER INDEX ...`ステートメントを使用して、インデックスの可視性を変更できます。

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (c1 INT, UNIQUE(c1));
ALTER TABLE t1 ALTER INDEX c1 INVISIBLE;
```

```sql
Query OK, 0 rows affected (0.02 sec)
```

{{< copyable "sql" >}}

```sql
SHOW CREATE TABLE t1;
```

```sql
+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table
                                    |
+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| t1    | CREATE TABLE `t1` (
  `c1` int(11) DEFAULT NULL,
  UNIQUE KEY `c1` (`c1`) /*!80000 INVISIBLE */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin |
+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

オプティマイザは、`c1`の**インビジブルインデックス**を使用できません。

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT c1 FROM t1 ORDER BY c1;
```

```sql
+-------------------------+----------+-----------+---------------+--------------------------------+
| id                      | estRows  | task      | access object | operator info                  |
+-------------------------+----------+-----------+---------------+--------------------------------+
| Sort_4                  | 10000.00 | root      |               | test.t1.c1:asc                 |
| └─TableReader_8         | 10000.00 | root      |               | data:TableFullScan_7           |
|   └─TableFullScan_7     | 10000.00 | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+-------------------------+----------+-----------+---------------+--------------------------------+
3 rows in set (0.00 sec)
```

一方、`c2`は**ビジブルインデックス**であり、オプティマイザによって使用できます。

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT c2 FROM t1 ORDER BY c2;
```

```sql
+------------------------+----------+-----------+------------------------+-------------------------------+
| id                     | estRows  | task      | access object          | operator info                 |
+------------------------+----------+-----------+------------------------+-------------------------------+
| IndexReader_13         | 10000.00 | root      |                        | index:IndexFullScan_12        |
| └─IndexFullScan_12     | 10000.00 | cop[tikv] | table:t1, index:c2(c2) | keep order:true, stats:pseudo |
+------------------------+----------+-----------+------------------------+-------------------------------+
2 rows in set (0.00 sec)
```

強制的にインデックスを使用するために`USE INDEX` SQLヒントを使用しても、オプティマイザは依然として非表示のインデックスを使用できません。そうでなければ、エラーが返されます。

{{< copyable "sql" >}}

```sql
SELECT * FROM t1 USE INDEX(c1);
```

```sql
ERROR 1176 (42000): Key 'c1' doesn't exist in table 't1'
```

> **注意:**
>
> ここでの"Invisible"とは、オプティマイザにのみ見えないことを意味します。それでも非表示のインデックスを変更または削除することができます。

{{< copyable "sql" >}}

```sql
ALTER TABLE t1 DROP INDEX c1;
```

```sql
Query OK, 0 rows affected (0.02 sec)
```

## MySQL互換性

* TiDBの非表示インデックスは、MySQL 8.0からの同等の機能をモデル化しています。
* MySQL同様、TiDBでは`PRIMARY KEY`インデックスを非表示にすることはできません。
* MySQLは、オプティマイザスイッチ`use_invisible_indexes=on`を提供して、すべての非表示のインデックスを再び_可視_にする機能を提供しています。この機能はTiDBでは利用できません。

## 関連項目

* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [CREATE INDEX](/sql-statements/sql-statement-create-index.md)
* [ADD INDEX](/sql-statements/sql-statement-add-index.md)
* [DROP INDEX](/sql-statements/sql-statement-drop-index.md)
* [RENAME INDEX](/sql-statements/sql-statement-rename-index.md)