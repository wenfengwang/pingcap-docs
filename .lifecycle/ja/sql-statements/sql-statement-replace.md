---
title: REPLACE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースでのREPLACEの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-replace/','/docs/dev/reference/sql/statements/replace/']
---

# REPLACE

`REPLACE`ステートメントは、意味的には`DELETE`+`INSERT`ステートメントの組み合わせです。アプリケーションコードを簡略化するために使用できます。

## 構文

```ebnf+diagram
ReplaceIntoStmt ::=
    'REPLACE' PriorityOpt IntoOpt TableName PartitionNameListOpt InsertValues

PriorityOpt ::=
    ( 'LOW_PRIORITY' | 'HIGH_PRIORITY' | 'DELAYED' )?

IntoOpt ::= 'INTO'?

TableName ::=
    Identifier ( '.' Identifier )?

PartitionNameListOpt ::=
    ( 'PARTITION' '(' Identifier ( ',' Identifier )* ')' )?

InsertValues ::=
    '(' ( ColumnNameListOpt ')' ( ValueSym ValuesList | SelectStmt | '(' SelectStmt ')' | UnionStmt ) | SelectStmt ')' )
|   ValueSym ValuesList
|   SelectStmt
|   UnionStmt
|   'SET' ColumnSetValue? ( ',' ColumnSetValue )*
```

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
Query OK, 0 rows affected (0.12 sec)

mysql> INSERT INTO t1 (c1) VALUES (1), (2), (3);
Query OK, 3 rows affected (0.02 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t1;
+----+----+
| id | c1 |
+----+----+
|  1 |  1 |
|  2 |  2 |
|  3 |  3 |
+----+----+
3 rows in set (0.00 sec)

mysql> REPLACE INTO t1 (id, c1) VALUES(3, 99);
Query OK, 2 rows affected (0.01 sec)

mysql> SELECT * FROM t1;
+----+----+
| id | c1 |
+----+----+
|  1 |  1 |
|  2 |  2 |
|  3 | 99 |
+----+----+
3 rows in set (0.00 sec)
```

## MySQL互換性

TiDBの`REPLACE`ステートメントはMySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連情報

* [DELETE](/sql-statements/sql-statement-delete.md)
* [INSERT](/sql-statements/sql-statement-insert.md)
* [SELECT](/sql-statements/sql-statement-select.md)
* [UPDATE](/sql-statements/sql-statement-update.md)