---
title: DEALLOCATE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのDEALLOCATEの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-deallocate/','/docs/dev/reference/sql/statements/deallocate/']
---

# DEALLOCATE

`DEALLOCATE`ステートメントは、サーバーサイドのプリペアドステートメントへのSQLインタフェースを提供します。

## 概要

```ebnf+diagram
DeallocateStmt ::=
    DeallocateSym 'PREPARE' Identifier

DeallocateSym ::=
    'DEALLOCATE'
|   'DROP'

Identifier ::=
    identifier
|   UnReservedKeyword
|   NotKeywordToken
|   TiDBKeyword
```

## 例

```sql
mysql> PREPARE mystmt FROM 'SELECT ? as num FROM DUAL';
クエリが実行されました。 0件影響を受けました (0.00 秒)

mysql> SET @number = 5;
クエリが実行されました。 0件影響を受けました (0.00 秒)

mysql> EXECUTE mystmt USING @number;
+------+
| num  |
+------+
| 5    |
+------+
1 行がセットされました (0.00 秒)

mysql> DEALLOCATE PREPARE mystmt;
クエリが実行されました。 0件影響を受けました (0.00 秒)
```

## MySQL互換性

TiDBにおける`DEALLOCATE`ステートメントはMySQLと完全に互換性があります。互換性の違いを見つけた場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [PREPARE](/sql-statements/sql-statement-prepare.md)
* [EXECUTE](/sql-statements/sql-statement-execute.md)