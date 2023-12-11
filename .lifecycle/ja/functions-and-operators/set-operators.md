---
title: 集合演算
summary: TiDBでサポートされている集合演算を学びます。

# 集合演算

TiDBでは、UNION、EXCEPT、INTERSECT演算子を使用した3つの集合演算がサポートされています。集合の最小単位は[`SELECT`ステートメント](/sql-statements/sql-statement-select.md)です。

## UNION演算子

数学では、2つの集合AとBの和集合は、AまたはBにあるすべての要素で構成されています。例：

```sql
SELECT 1 UNION SELECT 2;
+---+
| 1 |
+---+
| 2 |
| 1 |
+---+
2 rows in set (0.00 sec)
```

TiDBでは、`UNION DISTINCT`と`UNION ALL`演算子の両方がサポートされています。`UNION DISTINCT`は結果セットから重複するレコードを削除し、`UNION ALL`は重複を含むすべてのレコードを保持します。TiDBではデフォルトで`UNION DISTINCT`が使用されます。

```sql
CREATE TABLE t1 (a int);
CREATE TABLE t2 (a int);
INSERT INTO t1 VALUES (1),(2);
INSERT INTO t2 VALUES (1),(3);
```

`UNION DISTINCT`および`UNION ALL`クエリの例はそれぞれ次の通りです：

```sql
SELECT * FROM t1 UNION DISTINCT SELECT * FROM t2;
+---+
| a |
+---+
| 1 |
| 2 |
| 3 |
+---+
3 rows in set (0.00 sec)

SELECT * FROM t1 UNION ALL SELECT * FROM t2;
+---+
| a |
+---+
| 1 |
| 2 |
| 1 |
| 3 |
+---+
4 rows in set (0.00 sec)
```

## EXCEPT演算子

AとBが2つの集合である場合、EXCEPTはAとBの差集合を返し、AにあるがBにない要素で構成されます。

```sql
SELECT * FROM t1 EXCEPT SELECT * FROM t2;
+---+
| a |
+---+
| 2 |
+---+
1 rows in set (0.00 sec)
```

`EXCEPT ALL`演算子はまだサポートされていません。

## INTERSECT演算子

数学では、2つの集合AとBの共通部分は、AとBの両方にあるすべての要素で構成され、他の要素はありません。

```sql
SELECT * FROM t1 INTERSECT SELECT * FROM t2;
+---+
| a |
+---+
| 1 |
+---+
1 rows in set (0.00 sec)
```

`INTERSECT ALL`演算子はまだサポートされていません。INTERSECT演算子はEXCEPTおよびUNION演算子よりも優先順位が高いです。

```sql
SELECT * FROM t1 UNION ALL SELECT * FROM t1 INTERSECT SELECT * FROM t2;
+---+
| a |
+---+
| 1 |
| 1 |
| 2 |
+---+
3 rows in set (0.00 sec)
```

## パーレンテシス

TiDBでは、集合演算の優先順位を指定するためにカッコを使用することができます。カッコ内の式が最初に処理されます。

```sql
(SELECT * FROM t1 UNION ALL SELECT * FROM t1) INTERSECT SELECT * FROM t2;
+---+
| a |
+---+
| 1 |
+---+
1 rows in set (0.00 sec)
```

## `ORDER BY`と`LIMIT`の使用

TiDBでは、集合演算で[`ORDER BY`](/media/sqlgram/OrderByOptional.png)または[`LIMIT`](/media/sqlgram/LimitClause.png)句を使用することができます。これらの2つの句は、ステートメント全体の最後に配置する必要があります。

```sql
(SELECT * FROM t1 UNION ALL SELECT * FROM t1 INTERSECT SELECT * FROM t2) ORDER BY a LIMIT 2;
+---+
| a |
+---+
| 1 |
| 1 |
+---+
2 rows in set (0.00 sec)
```