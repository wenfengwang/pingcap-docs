---
title: DELETE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースにおけるDELETEの使用概要。
aliases: ['/docs/dev/sql-statements/sql-statement-delete/','/docs/dev/reference/sql/statements/delete/']
---

# DELETE

`DELETE` ステートメントは指定されたテーブルから行を削除します。

## 概要

```ebnf+diagram
DeleteFromStmt ::=
    'DELETE' TableOptimizerHints PriorityOpt QuickOptional IgnoreOptional ( 'FROM' ( TableName TableAsNameOpt IndexHintListOpt WhereClauseOptional OrderByOptional LimitClause | TableAliasRefList 'USING' TableRefs WhereClauseOptional ) | TableAliasRefList 'FROM' TableRefs WhereClauseOptional )
```

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
Query OK, 0 rows affected (0.11 sec)

mysql> INSERT INTO t1 (c1) VALUES (1),(2),(3),(4),(5);
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t1;
+----+----+
| id | c1 |
+----+----+
|  1 |  1 |
|  2 |  2 |
|  3 |  3 |
|  4 |  4 |
|  5 |  5 |
+----+----+
5 rows in set (0.00 sec)

mysql> DELETE FROM t1 WHERE id = 4;
Query OK, 1 row affected (0.02 sec)

mysql> SELECT * FROM t1;
+----+----+
| id | c1 |
+----+----+
|  1 |  1 |
|  2 |  2 |
|  3 |  3 |
|  5 |  5 |
+----+----+
4 rows in set (0.00 sec)
```

## MySQL互換性

TiDBの`DELETE`ステートメントはMySQLと完全に互換性があります。互換性の違いを見つけた場合、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連情報

* [INSERT](/sql-statements/sql-statement-insert.md)
* [SELECT](/sql-statements/sql-statement-select.md)
* [UPDATE](/sql-statements/sql-statement-update.md)
* [REPLACE](/sql-statements/sql-statement-replace.md)