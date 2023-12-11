---
title: SHOW TABLE STATUS | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのSHOW TABLE STATUSの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-show-table-status/', '/docs/dev/reference/sql/statements/show-table-status/']
---

# SHOW TABLE STATUS

このステートメントは、TiDBのテーブルに関するさまざまな統計情報を表示します。統計情報が古くなっている場合は、[`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md)を実行することをお勧めします。

## シノプシス

**ShowTableStatusStmt:**

![ShowTableStatusStmt](/media/sqlgram/ShowTableStatusStmt.png)

**FromOrIn:**

![FromOrIn](/media/sqlgram/FromOrIn.png)

**StatusTableName:**

![StatusTableName](/media/sqlgram/StatusTableName.png)

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
Query OK, 0 rows affected (0.11 sec)

mysql> INSERT INTO t1 (c1) VALUES (1),(2),(3),(4),(5);
Query OK, 5 rows affected (0.02 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW TABLE STATUS LIKE 't1'\G
*************************** 1. row ***************************
           Name: t1
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 0
 Avg_row_length: 0
    Data_length: 0
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 30001
    Create_time: 2019-04-19 08:32:06
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_bin
       Checksum:
 Create_options:
        Comment:
1 row in set (0.00 sec)

mysql> analyze table t1;
Query OK, 0 rows affected (0.12 sec)

mysql> SHOW TABLE STATUS LIKE 't1'\G
*************************** 1. row ***************************
           Name: t1
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 5
 Avg_row_length: 16
    Data_length: 80
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 30001
    Create_time: 2019-04-19 08:32:06
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_bin
       Checksum:
 Create_options:
        Comment:
1 row in set (0.00 sec)
```

## MySQL互換性

TiDBの`SHOW TABLE STATUS`ステートメントはMySQLと完全に互換性があります。互換性の違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [SHOW TABLES](/sql-statements/sql-statement-show-tables.md)
* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [DROP TABLE](/sql-statements/sql-statement-drop-table.md)
* [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md)