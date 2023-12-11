---
title: TABLE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースでのTABLEの使用方法の概要。
---

# TABLE

`TABLE`ステートメントは、集計や複雑なフィルタリングが不要な場合に、`SELECT * FROM`の代わりに使用できます。

## 構文

```ebnf+diagram
TableStmt ::=
    "TABLE" Table ( "ORDER BY" Column )? ( "LIMIT" NUM )?
```

## 例

`t1`テーブルを作成する：

```sql
CREATE TABLE t1(id INT PRIMARY KEY);
```

`t1`にいくつかのデータを挿入する：

```sql
INSERT INTO t1 VALUES (1),(2),(3);
```

`t1`テーブルのデータを表示する：

```sql
TABLE t1;
```

```sql
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
+----+
3 rows in set (0.01 sec)
```

`t1`をクエリし、結果を`id`フィールドで降順に並べ替える：

```sql
TABLE t1 ORDER BY id DESC;
```

```sql
+----+
| id |
+----+
|  3 |
|  2 |
|  1 |
+----+
3 rows in set (0.01 sec)
```

`t1`の最初のレコードをクエリする：

```sql
TABLE t1 LIMIT 1;
```

```sql
+----+
| id |
+----+
|  1 |
+----+
1 row in set (0.01 sec)
```

## MySQL互換性

`TABLE`ステートメントは、MySQL 8.0.19で導入されました。

## 関連項目

- [`SELECT`](/sql-statements/sql-statement-select.md)
- [MySQLにおける`TABLE`ステートメント](https://dev.mysql.com/doc/refman/8.0/en/table.html)