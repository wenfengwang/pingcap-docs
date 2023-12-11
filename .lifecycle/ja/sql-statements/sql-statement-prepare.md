---
title: PREPARE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのPREPAREの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-prepare/','/docs/dev/reference/sql/statements/prepare/']
---

# PREPARE

`PREPARE` ステートメントは、サーバーサイドのプリペアドステートメントに対するSQLインタフェースを提供します。

## 概要

```ebnf+diagram
PreparedStmt ::=
    'PREPARE' Identifier 'FROM' PrepareSQL

PrepareSQL ::=
    stringLit
|   UserVariable
```

## 例

```sql
mysql> PREPARE mystmt FROM 'SELECT ? as num FROM DUAL';
クエリが実行されました: 行数: 0  (0.00 秒)

mysql> SET @number = 5;
クエリが実行されました: 行数: 0  (0.00 秒)

mysql> EXECUTE mystmt USING @number;
+------+
| num  |
+------+
| 5    |
+------+
1 row in set (0.00 sec)

mysql> DEALLOCATE PREPARE mystmt;
クエリが実行されました: 行数: 0  (0.00 秒)
```

## MySQL互換性

TiDBの`PREPARE`ステートメントはMySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)してください。

## 関連項目

* [EXECUTE](/sql-statements/sql-statement-execute.md)
* [DEALLOCATE](/sql-statements/sql-statement-deallocate.md)