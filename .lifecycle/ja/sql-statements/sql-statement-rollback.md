---
title: ROLLBACK | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのROLLBACKの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-rollback/','/docs/dev/reference/sql/statements/rollback/']
---

# ROLLBACK

このステートメントは、TIDB内の現在のトランザクション内でのすべての変更を取り消します。これは `COMMIT` ステートメントの反対です。

## 概要

```ebnf+diagram
RollbackStmt ::=
    'ROLLBACK' CompletionTypeWithinTransaction?

CompletionTypeWithinTransaction ::=
    'AND' ( 'CHAIN' ( 'NO' 'RELEASE' )? | 'NO' 'CHAIN' ( 'NO'? 'RELEASE' )? )
|   'NO'? 'RELEASE'
```

## 例

```sql
mysql> CREATE TABLE t1 (a INT NOT NULL PRIMARY KEY);
Query OK, 0 rows affected (0.12 sec)

mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t1 VALUES (1);
Query OK, 1 row affected (0.00 sec)

mysql> ROLLBACK;
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT * FROM t1;
Empty set (0.01 sec)
```

## MySQL互換性

* TiDBは`ROLLBACK AND [NO] RELEASE`構文を解析しますが無視します。この機能は、MySQLでトランザクションをロールバックした直後にクライアントセッションを切断するために使用されます。TiDBでは、代わりにクライアントドライバーの`mysql_close()`機能を使用することをお勧めします。
* TiDBは`ROLLBACK AND [NO] CHAIN`構文を解析しますが無視します。この機能は、MySQLで現在のトランザクションがロールバックされる間に同じ分離レベルで新しいトランザクションをすぐに開始するために使用されます。TiDBでは、代わりに新しいトランザクションを開始することをお勧めします。

## 関連項目

* [SAVEPOINT](/sql-statements/sql-statement-savepoint.md)
* [COMMIT](/sql-statements/sql-statement-commit.md)
* [BEGIN](/sql-statements/sql-statement-begin.md)
* [START TRANSACTION](/sql-statements/sql-statement-start-transaction.md)