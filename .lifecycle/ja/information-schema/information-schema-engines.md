---
title: ENGINES
summary: 「ENGINES」のinformation_schemaテーブルの情報を学びます。

# ENGINES

`ENGINES`テーブルはストレージエンジンに関する情報を提供します。互換性のために、TiDBは常にInnoDBを唯一のサポートされるエンジンとして記述します。さらに、`ENGINES`テーブルの他の列の値も固定値です。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC engines;
```

```sql
+--------------+-------------+------+------+---------+-------+
| Field        | Type        | Null | Key  | Default | Extra |
+--------------+-------------+------+------+---------+-------+
| ENGINE       | varchar(64) | YES  |      | NULL    |       |
| SUPPORT      | varchar(8)  | YES  |      | NULL    |       |
| COMMENT      | varchar(80) | YES  |      | NULL    |       |
| TRANSACTIONS | varchar(3)  | YES  |      | NULL    |       |
| XA           | varchar(3)  | YES  |      | NULL    |       |
| SAVEPOINTS   | varchar(3)  | YES  |      | NULL    |       |
+--------------+-------------+------+------+---------+-------+
6 rows in set (0.00 sec)
```

{{< copyable "sql" >}}

```sql
SELECT * FROM engines;
```

```
+--------+---------+------------------------------------------------------------+--------------+------+------------+
| ENGINE | SUPPORT | COMMENT                                                    | TRANSACTIONS | XA   | SAVEPOINTS |
+--------+---------+------------------------------------------------------------+--------------+------+------------+
| InnoDB | DEFAULT | サポートされているトランザクション、行レベルのロックおよび外部キーをサポートします | YES          | YES  | YES        |
+--------+---------+------------------------------------------------------------+--------------+------+------------+
1 row in set (0.01 sec)
```

`ENGINES`テーブルの列の説明は以下の通りです：

* `ENGINES`: ストレージエンジンの名前。
* `SUPPORT`: サーバーがストレージエンジンをサポートしているレベル。TiDBでは、値は常に`DEFAULT`です。
* `COMMENT`: ストレージエンジンに関する簡単なコメント。
* `TRANSACTIONS`: ストレージエンジンがトランザクションをサポートしているかどうか。
* `XA`: ストレージエンジンがXAトランザクションをサポートしているかどうか。
* `SAVEPOINTS`: ストレージエンジンが`savepoints`をサポートしているかどうか。