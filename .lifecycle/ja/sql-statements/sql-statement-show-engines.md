---
title: SHOW ENGINES | TiDB SQL文のリファレンス
summary: TiDBデータベースのSHOW ENGINESの使用法の概要
aliases: ['/docs/dev/sql-statements/sql-statement-show-engines/','/docs/dev/reference/sql/statements/show-engines/']
---

# SHOW ENGINES

この文は、すべてのサポートされるストレージエンジンをリストするために使用されます。構文は、MySQLとの互換性のためにのみ含まれています。

## 概要

**ShowEnginesStmt:**

![ShowEnginesStmt](/media/sqlgram/ShowEnginesStmt.png)

```sql
SHOW ENGINES;
```

## 例

```sql
mysql> SHOW ENGINES;
+--------+---------+------------------------------------------------------------+--------------+------+------------+
| Engine | Support | Comment                                                    | Transactions | XA   | Savepoints |
+--------+---------+------------------------------------------------------------+--------------+------+------------+
| InnoDB | DEFAULT | トランザクション、行レベルのロック、および外部キーをサポート | YES          | YES  | YES        |
+--------+---------+------------------------------------------------------------+--------------+------+------------+
1 row in set (0.00 sec)
```

## MySQL互換性

* この文は常にサポートされるエンジンとしてInnoDBのみを返します。内部的には、TiDBは通常、ストレージエンジンとしてTiKVを使用します。