---
title: INSERT | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのINSERTの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-insert/', '/docs/dev/reference/sql/statements/insert/']
---

# INSERT

このステートメントは新しい行をテーブルに挿入します。

## 概要

```ebnf+diagram
InsertIntoStmt ::=
    'INSERT' TableOptimizerHints PriorityOpt IgnoreOptional IntoOpt TableName PartitionNameListOpt InsertValues OnDuplicateKeyUpdate

TableOptimizerHints ::=
    hintComment?

PriorityOpt ::=
    ( 'LOW_PRIORITY' | 'HIGH_PRIORITY' | 'DELAYED' )?

IgnoreOptional ::=
    'IGNORE'?

IntoOpt  ::= 'INTO'?

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

OnDuplicateKeyUpdate ::=
    ( 'ON' 'DUPLICATE' 'KEY' 'UPDATE' AssignmentList )?
```

## 例

```sql
mysql> CREATE TABLE t1 (a INT);
クエリが正常に完了しました、0 行が影響を受けました(0.11 秒)

mysql> CREATE TABLE t2 LIKE t1;
クエリが正常に完了しました、0 行が影響を受けました(0.11 秒)

mysql> INSERT INTO t1 VALUES (1);
クエリが正常に完了しました、1 行が影響を受けました(0.02 秒)

mysql> INSERT INTO t1 (a) VALUES (1);
クエリが正常に完了しました、1 行が影響を受けました(0.01 秒)

mysql> INSERT INTO t2 SELECT * FROM t1;
クエリが正常に完了しました、2 行が影響を受けました(0.01 秒)
レコード: 2  重複: 0  警告: 0

mysql> SELECT * FROM t1;
+------+
| a    |
+------+
|    1 |
|    1 |
+------+
2 行がセットされました(0.00 秒)

mysql> SELECT * FROM t2;
+------+
| a    |
+------+
|    1 |
|    1 |
+------+
2 行がセットされました(0.00 秒)

mysql> INSERT INTO t2 VALUES (2),(3),(4);
クエリが正常に完了しました、3 行が影響を受けました(0.02 秒)
レコード: 3  重複: 0  警告: 0

mysql> SELECT * FROM t2;
+------+
| a    |
+------+
|    1 |
|    1 |
|    2 |
|    3 |
|    4 |
+------+
5 行がセットされました(0.00 秒)
```

## MySQL互換性

TiDBの`INSERT`ステートメントはMySQLと完全に互換性があります。互換性の違いがある場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)してください。

## 関連項目

* [DELETE](/sql-statements/sql-statement-delete.md)
* [SELECT](/sql-statements/sql-statement-select.md)
* [UPDATE](/sql-statements/sql-statement-update.md)
* [REPLACE](/sql-statements/sql-statement-replace.md)