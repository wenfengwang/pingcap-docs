---
title: CREATE TABLE LIKE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのCREATE TABLE LIKEの使用法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-create-table-like/','/docs/dev/reference/sql/statements/create-table-like/']
---

# CREATE TABLE LIKE

このステートメントは、データをコピーせずに既存のテーブルの定義をコピーします。

## 構文

```ebnf+diagram
CreateTableLikeStmt ::=
    'CREATE' OptTemporary 'TABLE' IfNotExists TableName LikeTableWithOrWithoutParen OnCommitOpt

OptTemporary ::=
    ( 'TEMPORARY' | ('GLOBAL' 'TEMPORARY') )?

LikeTableWithOrWithoutParen ::=
    'LIKE' TableName
|   '(' 'LIKE' TableName ')'

OnCommitOpt ::=
    ('ON' 'COMMIT' 'DELETE' 'ROWS')?
```

## 例

```sql
mysql> CREATE TABLE t1 (a INT NOT NULL);
クエリは成功しました。0件のレコードが影響を受けました（0.13秒）

mysql> INSERT INTO t1 VALUES (1),(2),(3),(4),(5);
クエリは成功しました。5件のレコードが影響を受けました（0.02秒）
レコード: 5  重複: 0  警告: 0

mysql> SELECT * FROM t1;
+---+
| a |
+---+
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |
+---+
5 rows in set (0.00 sec)

mysql> CREATE TABLE t2 LIKE t1;
クエリは成功しました。0件のレコードが影響を受けました（0.10秒）

mysql> SELECT * FROM t2;
空のセット（0.00秒）
```

## プリスプリット領域

コピーされるテーブルが`PRE_SPLIT_REGIONS`属性で定義されている場合、`CREATE TABLE LIKE`ステートメントを使用して作成されたテーブルはこの属性を継承し、新しいテーブルのRegionは分割されます。`PRE_SPLIT_REGIONS`の詳細については、[`CREATE TABLE`ステートメント](/sql-statements/sql-statement-create-table.md)を参照してください。

## MySQL互換性

TiDBの`CREATE TABLE LIKE`ステートメントは、MySQLと完全に互換性があります。互換性の違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連情報

* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md)