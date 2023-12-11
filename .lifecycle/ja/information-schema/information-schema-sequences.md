---
title: シーケンス
summary: `SEQUENCES` INFORMATION_SCHEMA テーブルを学びます。
---

# シーケンス

`SEQUENCES` テーブルはシーケンスに関する情報を提供します。[シーケンス機能](/sql-statements/sql-statement-create-sequence.md) は、MariaDB の類似機能に基づいています。

```sql
USE INFORMATION_SCHEMA;
DESC SEQUENCES;
```

出力は次のようになります。

```sql
+-----------------+--------------+------+------+---------+-------+
| Field           | Type         | Null | Key  | Default | Extra |
+-----------------+--------------+------+------+---------+-------+
| TABLE_CATALOG   | varchar(512) | NO   |      | NULL    |       |
| SEQUENCE_SCHEMA | varchar(64)  | NO   |      | NULL    |       |
| SEQUENCE_NAME   | varchar(64)  | NO   |      | NULL    |       |
| CACHE           | tinyint(0)   | NO   |      | NULL    |       |
| CACHE_VALUE     | bigint(21)   | YES  |      | NULL    |       |
| CYCLE           | tinyint(0)   | NO   |      | NULL    |       |
| INCREMENT       | bigint(21)   | NO   |      | NULL    |       |
| MAX_VALUE       | bigint(21)   | YES  |      | NULL    |       |
| MIN_VALUE       | bigint(21)   | YES  |      | NULL    |       |
| START           | bigint(21)   | YES  |      | NULL    |       |
| COMMENT         | varchar(64)  | YES  |      | NULL    |       |
+-----------------+--------------+------+------+---------+-------+
11 行がセットされました (0.00 sec)
```

`test.seq` というシーケンスを作成し、シーケンスの次の値をクエリします。

```sql
CREATE SEQUENCE test.seq;
SELECT nextval(test.seq);
SELECT * FROM sequences\G
```

出力は次のようになります。

```sql
+-------------------+
| nextval(test.seq) |
+-------------------+
|                 1 |
+-------------------+
1 行がセットされました (0.01 sec)
```

すべてのシーケンスを表示します。

```sql
SELECT * FROM SEQUENCES\G
```

出力は次のようになります。

```sql
*************************** 1. row ***************************
  TABLE_CATALOG: def
SEQUENCE_SCHEMA: test
  SEQUENCE_NAME: seq
          CACHE: 1
    CACHE_VALUE: 1000
          CYCLE: 0
      INCREMENT: 1
      MAX_VALUE: 9223372036854775806
      MIN_VALUE: 1
          START: 1
        COMMENT:
1 行がセットされました (0.00 sec)
```