---
title: EXECUTE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのEXECUTEの使用概要。
aliases: ['/docs/dev/sql-statements/sql-statement-execute/','/docs/dev/reference/sql/statements/execute/']
---

# EXECUTE

`EXECUTE`ステートメントは、サーバーサイドのプリペアドステートメントに対するSQLインタフェースを提供します。

## 構文

```ebnf+diagram
ExecuteStmt ::=
    'EXECUTE' 識別子 ( 'USING' ユーザ変数 ( ',' ユーザ変数 )* )?
```

## 例

```sql
mysql> PREPARE mystmt FROM 'SELECT ? as num FROM DUAL';
クエリOK、0件の行が変更されました (0.00 秒)

mysql> SET @number = 5;
クエリOK、0件の行が変更されました (0.00 秒)

mysql> EXECUTE mystmt USING @number;
+------+
| num  |
+------+
| 5    |
+------+
1 row in set (0.00 秒)

mysql> DEALLOCATE PREPARE mystmt;
クエリOK、0件の行が変更されました (0.00 秒)
```

## MySQL互換性

TiDBの`EXECUTE`ステートメントはMySQLと完全に互換性があります。互換性の違いを見つけた場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [PREPARE](/sql-statements/sql-statement-prepare.md)
* [DEALLOCATE](/sql-statements/sql-statement-deallocate.md)