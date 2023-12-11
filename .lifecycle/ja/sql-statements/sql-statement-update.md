---
title: UPDATE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースに対するUPDATEの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-update/','/docs/dev/reference/sql/statements/update/']
---

# UPDATE

`UPDATE`ステートメントは指定されたテーブル内のデータを修正するために使用されます。

## 概要

**UpdateStmt:**

![UpdateStmt](/media/sqlgram/UpdateStmt.png)

**PriorityOpt:**

![PriorityOpt](/media/sqlgram/PriorityOpt.png)

**TableRef:**

![TableRef](/media/sqlgram/TableRef.png)

**TableRefs:**

![TableRefs](/media/sqlgram/TableRefs.png)

**AssignmentList:**

![AssignmentList](/media/sqlgram/AssignmentList.png)

**WhereClauseOptional:**

![WhereClauseOptional](/media/sqlgram/WhereClauseOptional.png)

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
Query OK, 0 rows affected (0.11 sec)

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

mysql> UPDATE t1 SET c1=5 WHERE c1=3;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT * FROM t1;
+----+----+
| id | c1 |
+----+----+
|  1 |  1 |
|  2 |  2 |
|  3 |  5 |
+----+----+
3 rows in set (0.00 sec)
```

## MySQL互換性

TiDBは常に式を評価する際にカラムの元の値を使用します。例えば:

```sql
CREATE TABLE t (a int, b int);
INSERT INTO t VALUES (1,2);
UPDATE t SET a = a+1,b=a;
```

MySQLでは、カラム`b`は`a`の値に設定されているため、2に更新されます。同じステートメント内で`a`の値（1）が`a+1`（2）に更新されます。

TiDBはより標準的なSQLの動作に従い、`b`を1に更新します。

## 関連情報

* [INSERT](/sql-statements/sql-statement-insert.md)
* [SELECT](/sql-statements/sql-statement-select.md)
* [DELETE](/sql-statements/sql-statement-delete.md)
* [REPLACE](/sql-statements/sql-statement-replace.md)