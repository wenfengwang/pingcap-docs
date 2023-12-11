---
title: COMMIT | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのCOMMITの利用の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-commit/','/docs/dev/reference/sql/statements/commit/']
---

# COMMIT

このステートメントはTIDBサーバー内でトランザクションをコミットします。

`BEGIN`または`START TRANSACTION`ステートメントがない場合、TiDBのデフォルトの動作は、すべてのステートメントが独自のトランザクションでオートコミットされることです。この動作はMySQLの互換性を確保します。

## 概要

```ebnf+diagram
CommitStmt ::=
    'COMMIT' CompletionTypeWithinTransaction?

CompletionTypeWithinTransaction ::=
    'AND' ( 'CHAIN' ( 'NO' 'RELEASE' )? | 'NO' 'CHAIN' ( 'NO'? 'RELEASE' )? )
|   'NO'? 'RELEASE'
```

## 例

```sql
mysql> CREATE TABLE t1 (a int NOT NULL PRIMARY KEY);
クエリ完了、0行が変更されました (0.12秒)

mysql> START TRANSACTION;
クエリ完了、0行が変更されました (0.00秒)

mysql> INSERT INTO t1 VALUES (1);
クエリ完了、1行が変更されました (0.00秒)

mysql> COMMIT;
クエリ完了、0行が変更されました (0.01秒)
```

## MySQLの互換性

* 現在、TiDBはデフォルトでメタデータロック（MDL）を使用してDDLステートメントがトランザクションで使用されるテーブルを変更することを防いでいます。メタデータロックの動作はTiDBとMySQLで異なります。詳細については、[メタデータロック](/metadata-lock.md)を参照してください。
* デフォルトでは、TiDB 3.0.8以降のバージョンは[悲観的ロック](/pessimistic-transaction.md)を使用します。[楽観的ロック](/optimistic-transaction.md)を使用する場合、`COMMIT`ステートメントが別のトランザクションによって行が修正されたため失敗する可能性があることを考慮することが重要です。
* 楽観的ロックが有効な場合、`UNIQUE`および`PRIMARY KEY`制約のチェックはステートメントがコミットされるまで延期されます。これにより、`COMMIT`ステートメントが失敗する追加の状況が発生します。この動作は`tidb_constraint_check_in_place=ON`を設定することで変更できます。
* TiDBは`syntax ROLLBACK AND [NO] RELEASE`の構文を解析しますが無視します。この機能は、MySQLでトランザクションをコミットした直後にクライアントセッションを切断するために使用されます。TiDBでは、代わりにクライアントドライバーの`mysql_close()`機能を使用することを推奨します。
* TiDBは`syntax ROLLBACK AND [NO] CHAIN`の構文を解析しますが無視します。この機能は、MySQLで現在のトランザクションがコミットされている間に同じ分離レベルで新しいトランザクションを開始するために使用されます。TiDBでは、代わりに新しいトランザクションを開始することを推奨します。

## 関連項目

* [START TRANSACTION](/sql-statements/sql-statement-start-transaction.md)
* [ROLLBACK](/sql-statements/sql-statement-rollback.md)
* [BEGIN](/sql-statements/sql-statement-begin.md)
* [制約の遅延チェック](/transaction-overview.md#lazy-check-of-constraints)