---
title: WITH | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースでのWITH（Common Table Expression）の使用の概要。
---

# WITH

Common Table Expression（CTE）は、SQLステートメント内で複数回参照される一時的な結果セットであり、ステートメントの可読性と実行効率の向上に役立ちます。Common Table Expressionsを使用するには、`WITH`ステートメントを適用できます。

## 概要

**WithClause:**

```ebnf+diagram
WithClause ::=
        "WITH" WithList
|       "WITH" recursive WithList
```

**WithList:**

```ebnf+diagram
WithList ::=
        WithList ',' CommonTableExpr
|       CommonTableExpr
```

**CommonTableExpr:**

```ebnf+diagram
CommonTableExpr ::=
        Identifier IdentListWithParenOpt "AS" SubSelect
```

**IdentListWithParenOpt:**

```ebnf+diagram
IdentListWithParenOpt ::=
( '(' IdentList ')' )?
```

## 例

非再帰的CTE:

{{< copyable "sql" >}}

```sql
WITH CTE AS (SELECT 1, 2) SELECT * FROM cte t1, cte t2;
```

```
+---+---+---+---+
| 1 | 2 | 1 | 2 |
+---+---+---+---+
| 1 | 2 | 1 | 2 |
+---+---+---+---+
1 row in set (0.00 sec)
```

再帰的CTE:

{{< copyable "sql" >}}

```sql
WITH RECURSIVE cte(a) AS (SELECT 1 UNION SELECT a+1 FROM cte WHERE a < 5) SELECT * FROM cte;
```

```
+---+
| a |
+---+
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |
+---+
5 rows in set (0.00 sec)
```

## MySQL互換性

* 厳密モードでは、データ長が再帰的に計算された場合にTiDBは警告を返し、MySQLはエラーを返します。緩和モードでは、TiDBの動作はMySQLと一貫しています。
* 再帰的CTEのデータ型は、シード部分によって決定されます。シード部分のデータ型は一部の場合（関数など）においてMySQLと完全に一致しないことがあります。
* 複数の`UNION` / `UNION ALL`演算子の場合、MySQLでは`UNION`の後に`UNION ALL`を許可しませんが、TiDBでは許可されます。
* CTEの定義に問題がある場合、TiDBはエラーを報告しますが、MySQLはCTEが参照されない場合は報告しません。

## 関連項目

* [SELECT](/sql-statements/sql-statement-select.md)
* [INSERT](/sql-statements/sql-statement-insert.md)
* [DELETE](/sql-statements/sql-statement-delete.md)
* [UPDATE](/sql-statements/sql-statement-update.md)
* [REPLACE](/sql-statements/sql-statement-replace.md)