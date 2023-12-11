---
title: SHOW TABLE NEXT_ROW_ID
summary: `SHOW TABLE NEXT_ROW_ID`のTiDBでの使用方法を学ぶ
aliases: ['/docs/dev/sql-statements/sql-statement-show-table-next-rowid/']
---

# SHOW TABLE NEXT_ROW_ID

`SHOW TABLE NEXT_ROW_ID`は、テーブルの特定のカラムの詳細を表示するために使用されます。これには次のものが含まれます。

* TiDBによって自動的に作成された`AUTO_INCREMENT`カラム、「_tidb_rowid」カラム。
* ユーザーによって作成された`AUTO_INCREMENT`カラム。
* ユーザーによって作成された[`AUTO_RANDOM`](/auto-random.md)カラム。
* ユーザーによって作成された[`SEQUENCE`](/sql-statements/sql-statement-create-sequence.md)。

## 概要

**ShowTableNextRowIDStmt:**

![ShowTableNextRowIDStmt](/media/sqlgram/ShowTableNextRowIDStmt.png)

**TableName:**

![TableName](/media/sqlgram/TableName.png)

## 例

新しく作成されたテーブルの場合、`NEXT_GLOBAL_ROW_ID`は割り当てられたRow IDがないため`1`となります。

{{< copyable "sql" >}}

```sql
create table t(a int);
Query OK, 0 rows affected (0.06 sec)
```

```sql
show table t next_row_id;
+---------+------------+-------------+--------------------+
| データベース名 | テーブル名   | カラム名      | NEXT_GLOBAL_ROW_ID |
+---------+------------+-------------+--------------------+
| test    | t          | _tidb_rowid |                  1 |
+---------+------------+-------------+--------------------+
1 row in set (0.00 sec)
```

データがテーブルに書き込まれました。データを挿入するTiDBサーバは一度に30000個のIDを割り当ててキャッシュします。そのため、NEXT_GLOBAL_ROW_IDは現在30001です。

```sql
insert into t values (), (), ();
Query OK, 3 rows affected (0.02 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

```sql
show table t next_row_id;
+---------+------------+-------------+--------------------+
| データベース名 | テーブル名   | カラム名      | NEXT_GLOBAL_ROW_ID |
+---------+------------+-------------+--------------------+
| test    | t          | _tidb_rowid |              30001 |
+---------+------------+-------------+--------------------+
1 row in set (0.00 sec)
```

## MySQL互換性

この文は、TiDBのMySQL構文への拡張です。

## 関連項目

* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [AUTO_RANDOM](/auto-random.md)
* [CREATE_SEQUENCE](/sql-statements/sql-statement-create-sequence.md)