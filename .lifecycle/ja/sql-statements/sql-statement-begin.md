---
title: BEGIN | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのBEGINの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-begin/','/docs/dev/reference/sql/statements/begin/']
---

# BEGIN

このステートメントは、TiDB内で新しいトランザクションを開始します。これは、`START TRANSACTION`ステートメントや`SET autocommit=0`ステートメントと類似しています。

`BEGIN`ステートメントがない場合、すべてのステートメントはデフォルトで独自のトランザクションで自動コミットされます。この動作によりMySQL互換性が確保されます。

## 概要

```ebnf+diagram
BeginTransactionStmt ::=
    'BEGIN' ( 'PESSIMISTIC' | 'OPTIMISTIC' )?
|   'START' 'TRANSACTION' ( 'READ' ( 'WRITE' | 'ONLY' ( 'WITH' 'TIMESTAMP' 'BOUND' TimestampBound )? ) | 'WITH' 'CONSISTENT' 'SNAPSHOT' )?
```

## 例

```sql
mysql> CREATE TABLE t1 (a int NOT NULL PRIMARY KEY);
Query OK, 0 rows affected (0.12 sec)

mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t1 VALUES (1);
Query OK, 1 row affected (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.01 sec)
```

## MySQL互換性

TiDBは`BEGIN PESSIMISTIC`または`BEGIN OPTIMISTIC`の構文拡張をサポートしています。これにより、トランザクションのデフォルトのモデルをオーバーライドできます。

## 関連情報

* [COMMIT](/sql-statements/sql-statement-commit.md)
* [ROLLBACK](/sql-statements/sql-statement-rollback.md)
* [START TRANSACTION](/sql-statements/sql-statement-start-transaction.md)
* [TiDB optimistic transaction model](/optimistic-transaction.md)
* [TiDB pessimistic transaction mode](/pessimistic-transaction.md)