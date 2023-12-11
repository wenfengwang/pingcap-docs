---
title: SET TRANSACTION | TiDB SQLステートメントリファレンス
summary: TiDBデータベースでのSET TRANSACTIONの使用の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-set-transaction/','/docs/dev/reference/sql/statements/set-transaction/']
---

# SET TRANSACTION

`SET TRANSACTION`ステートメントは、`GLOBAL`または`SESSION`ベースで現在の分離レベルを変更するために使用できます。この構文は、`SET transaction_isolation='new-value'`という代替構文であり、MySQLとSQL標準の両方と互換性があります。

## 概要

```ebnf+diagram
SetStmt ::=
    'SET' ( VariableAssignmentList |
    'PASSWORD' ('FOR' Username)? '=' PasswordOpt |
    ( 'GLOBAL'| 'SESSION' )? 'TRANSACTION' TransactionChars |
    'CONFIG' ( Identifier | stringLit) ConfigItemName EqOrAssignmentEq SetExpr )

TransactionChars ::=
    ( 'ISOLATION' 'LEVEL' IsolationLevel | 'READ' 'WRITE' | 'READ' 'ONLY' AsOfClause? )

IsolationLevel ::=
    ( 'REPEATABLE' 'READ' | 'READ' ( 'COMMITTED' | 'UNCOMMITTED' ) | 'SERIALIZABLE' )

AsOfClause ::=
    ( 'AS' 'OF' 'TIMESTAMP' Expression)
```

## 例

```sql
mysql> SHOW SESSION VARIABLES LIKE 'transaction_isolation';
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set (0.00 sec)

mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW SESSION VARIABLES LIKE 'transaction_isolation';
+-----------------------+----------------+
| Variable_name         | Value          |
+-----------------------+----------------+
| transaction_isolation | READ-COMMITTED |
+-----------------------+----------------+
1 row in set (0.01 sec)

mysql> SET SESSION transaction_isolation = 'REPEATABLE-READ';
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW SESSION VARIABLES LIKE 'transaction_isolation';
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set (0.00 sec)
```

## MySQLとの互換性

* TiDBは構文上のみでトランザクションを読み取り専用として設定する機能をサポートしています。
* 分離レベル`READ-UNCOMMITTED`および`SERIALIZABLE`はサポートされていません。
* `REPEATABLE-READ`分離レベルは、スナップショット分離技術を使用して達成され、これは一部でMySQLと互換性があります。
* 悲観的トランザクションでは、TiDBはMySQL互換の2つの分離レベル、`REPEATABLE-READ`と`READ-COMMITTED`をサポートしています。詳細な説明については、[分離レベル](/transaction-isolation-levels.md)を参照してください。

## 関連項目

* [`SET [GLOBAL|SESSION] <variable>`](/sql-statements/sql-statement-set-variable.md)
* [分離レベル](/transaction-isolation-levels.md)