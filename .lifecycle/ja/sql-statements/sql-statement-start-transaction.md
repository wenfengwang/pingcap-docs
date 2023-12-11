---
title: START TRANSACTION | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースにおけるSTART TRANSACTIONの使用法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-start-transaction/','/docs/dev/reference/sql/statements/start-transaction/']
---

# START TRANSACTION

このステートメントは、TiDB内で新しいトランザクションを開始します。これは`BEGIN`ステートメントと類似しています。

`START TRANSACTION`ステートメントがない場合、デフォルトでそれぞれのステートメントが自身のトランザクションで自動コミットされます。この動作はMySQLとの互換性を保証します。

## 概要

**BeginTransactionStmt:**

```ebnf+diagram
BeginTransactionStmt ::=
    'BEGIN' ( 'PESSIMISTIC' | 'OPTIMISTIC' )?
|   'START' 'TRANSACTION' ( 'READ' ( 'WRITE' | 'ONLY' ( ( 'WITH' 'TIMESTAMP' 'BOUND' TimestampBound )? | AsOfClause ) ) | 'WITH' 'CONSISTENT' 'SNAPSHOT' | 'WITH' 'CAUSAL' 'CONSISTENCY' 'ONLY' )?

AsOfClause ::=
    ( 'AS' 'OF' 'TIMESTAMP' Expression)
```

## 例

```sql
mysql> CREATE TABLE t1 (a int NOT NULL PRIMARY KEY);
Query OK, 0 rows affected (0.12 sec)

mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t1 VALUES (1);
Query OK, 1 row affected (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.01 sec)
```

## MySQLとの互換性

* `START TRANSACTION`はTiDB内で即座にトランザクションを開始します。これはMySQLと異なり、`START TRANSACTION`は遅延してトランザクションを作成します。ただし、TiDBの`START TRANSACTION`はMySQLの`START TRANSACTION WITH CONSISTENT SNAPSHOT`と等価です。

* ステートメント`START TRANSACTION READ ONLY`はMySQLとの互換性のために解析されますが、書き込み操作も許可されます。

## 関連項目

* [COMMIT](/sql-statements/sql-statement-commit.md)
* [ROLLBACK](/sql-statements/sql-statement-rollback.md)
* [BEGIN](/sql-statements/sql-statement-begin.md)
* [START TRANSACTION WITH CAUSAL CONSISTENCY ONLY](/transaction-overview.md#causal-consistency)