---
title: SHOW ERRORS | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースでのSHOW ERRORSの利用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-errors/','/docs/dev/reference/sql/statements/show-errors/']
---

# SHOW ERRORS

このステートメントは以前に実行されたステートメントからエラーを表示します。エラーバッファはステートメントが成功裏に実行されるとすぐにクリアされます。その場合、`SHOW ERRORS`は空のセットを返します。

どのステートメントがエラーや警告を生成するかの振る舞いは、現在の `sql_mode` に大きく影響を受けます。

## 概要

**ShowErrorsStmt:**

![ShowErrorsStmt](/media/sqlgram/ShowErrorsStmt.png)

## 例

```sql
mysql> select invalid;
ERROR 1054 (42S22): Unknown column 'invalid' in 'field list'
mysql> create invalid;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your TiDB version for the right syntax to use line 1 column 14 near "invalid"
mysql> SHOW ERRORS;
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                   |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Error | 1054 | Unknown column 'invalid' in 'field list'                                                                                                                  |
| Error | 1064 | You have an error in your SQL syntax; check the manual that corresponds to your TiDB version for the right syntax to use line 1 column 14 near "invalid"  |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> CREATE invalid2;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your TiDB version for the right syntax to use line 1 column 15 near "invalid2"
mysql> SELECT 1;
+------+
| 1    |
+------+
|    1 |
+------+
1 row in set (0.00 sec)

mysql> SHOW ERRORS;
Empty set (0.00 sec)
```

## MySQL 互換性

TiDBの`SHOW ERRORS`ステートメントはMySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [SHOW WARNINGS](/sql-statements/sql-statement-show-warnings.md)