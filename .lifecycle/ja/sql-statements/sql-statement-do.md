---
title: DO | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのDOの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-do/','/docs/dev/reference/sql/statements/do/']
---

# DO

`DO`は式を実行しますが、結果を返しません。ほとんどの場合、`DO`は結果を返さない`SELECT expr, ...`と同等です。

> **注意:**
>
> `DO`は式を実行するだけです。`SELECT`が使用できるすべての場合に使用することはできません。たとえば、`DO id FROM t1`は無効です。それはテーブルを参照しているからです。

MySQLでは、一般的なユースケースとして、ストアドプロシージャやトリガーを実行することがあります。ただし、TiDBはストアドプロシージャやトリガーを提供しないため、この機能は限定的に使用されます。

## 概要

```ebnf+diagram
DoStmt   ::= 'DO' ExpressionList

ExpressionList ::=
    Expression ( ',' Expression )*

Expression ::=
    ( singleAtIdentifier assignmentEq | 'NOT' | Expression ( logOr | 'XOR' | logAnd ) ) Expression
|   'MATCH' '(' ColumnNameList ')' 'AGAINST' '(' BitExpr FulltextSearchModifierOpt ')'
|   PredicateExpr ( IsOrNotOp 'NULL' | CompareOp ( ( singleAtIdentifier assignmentEq )? PredicateExpr | AnyOrAll SubSelect ) )* ( IsOrNotOp ( trueKwd | falseKwd | 'UNKNOWN' ) )?
```

## 例

このSELECTステートメントは一時停止しますが、結果セットも生成します。

```sql
mysql> SELECT SLEEP(5);
+----------+
| SLEEP(5) |
+----------+
|        0 |
+----------+
1 row in set (5.00 sec)
```

一方、DOは、結果セットを生成せずに一時停止します。

```sql
mysql> DO SLEEP(5);
Query OK, 0 rows affected (5.00 sec)

mysql> DO SLEEP(1), SLEEP(1.5);
Query OK, 0 rows affected (2.50 sec)
```

## MySQL互換性

TiDBの`DO`ステートメントはMySQLと完全に互換性があります。互換性の違いを見つけた場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [SELECT](/sql-statements/sql-statement-select.md)