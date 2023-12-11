---
title: SAVEPOINT | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのSAVEPOINTの使用概要。

# SAVEPOINT

`SAVEPOINT`はTiDB v6.2.0で導入された機能です。構文は次のとおりです。

```sql
SAVEPOINT 識別子
ROLLBACK TO [SAVEPOINT] 識別子
RELEASE SAVEPOINT 識別子
```

> **警告:**
>
> - TiDB Binlogが有効な場合、`SAVEPOINT`は使用できません。
> - [`tidb_constraint_check_in_place_pessimistic`が無効な場合、`SAVEPOINT`を悲観的トランザクションで使用できません](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630)。

- `SAVEPOINT`は、現在のトランザクションに指定された名前のセーブポイントを設定するために使用されます。同じ名前のセーブポイントがすでに存在する場合、それが削除され、同じ名前の新しいセーブポイントが設定されます。

- `ROLLBACK TO SAVEPOINT`は、指定された名前のセーブポイントにトランザクションをロールバックし、トランザクションを終了しません。セーブポイントの後に行われたテーブルデータのデータ変更は、ロールバックで元に戻され、セーブポイントの後のすべてのセーブポイントが削除されます。悲観的トランザクションでは、トランザクションが保持するロックはロールバックされません。代わりに、トランザクションが終了するとロックが解放されます。

    `ROLLBACK TO SAVEPOINT`ステートメントで指定されたセーブポイントが存在しない場合、次のエラーが返されます。

    ```
    ERROR 1305 (42000): SAVEPOINT identifier does not exist
    ```

- `RELEASE SAVEPOINT`ステートメントは、指定された名前のセーブポイントとその後の**すべてのセーブポイント**を現在のトランザクションから削除し、現在のトランザクションをコミットまたはロールバックしません。指定された名前のセーブポイントが存在しない場合、次のエラーが返されます。

    ```
    ERROR 1305 (42000): SAVEPOINT identifier does not exist
    ```

    トランザクションがコミットまたはロールバックされた後、トランザクション内のすべてのセーブポイントが削除されます。

## 例

テーブル`t1`を作成します。

```sql
CREATE TABLE t1 (a INT NOT NULL PRIMARY KEY);
```

```sql
Query OK, 0 rows affected (0.12 sec)
```

現在のトランザクションを開始します。

```sql
BEGIN;
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

テーブルにデータを挿入し、セーブポイント`sp1`を設定します。

```sql
INSERT INTO t1 VALUES (1);
```

```sql
Query OK, 1 row affected (0.00 sec)
```

```sql
SAVEPOINT sp1;
```

```sql
Query OK, 0 rows affected (0.01 sec)
```

テーブルに再度データを挿入し、セーブポイント`sp2`を設定します。

```sql
INSERT INTO t1 VALUES (2);
```

```sql
Query OK, 1 row affected (0.00 sec)
```

```sql
SAVEPOINT sp2;
```

```sql
Query OK, 0 rows affected (0.01 sec)
```

セーブポイント`sp2`を解放します。

```sql
RELEASE SAVEPOINT sp2;
```

```sql
Query OK, 0 rows affected (0.01 sec)
```

セーブポイント`sp1`にロールバックします。

```sql
ROLLBACK TO SAVEPOINT sp1;
```

```sql
Query OK, 0 rows affected (0.01 sec)
```

トランザクションをコミットし、テーブルをクエリします。`sp1`の前に挿入されたデータのみが返されます。

```sql
COMMIT;
```

```sql
Query OK, 0 rows affected (0.01 sec)
```

```sql
SELECT * FROM t1;
```

```sql
+---+
| a |
+---+
| 1 |
+---+
1 row in set
```

## MySQL互換性

`ROLLBACK TO SAVEPOINT`を使用してトランザクションを指定されたセーブポイントにロールバックする場合、MySQLは指定されたセーブポイント以降に保持されたロックのみを解放しますが、TiDBの悲観的トランザクションでは、指定されたセーブポイント以降に保持したすべてのロックをすぐに解放しません。代わりに、TiDBはトランザクションがコミットまたはロールバックされたときにすべてのロックを解放します。

## 関連項目

* [COMMIT](/sql-statements/sql-statement-commit.md)
* [ROLLBACK](/sql-statements/sql-statement-rollback.md)
* [START TRANSACTION](/sql-statements/sql-statement-start-transaction.md)
* [TiDBオプティミスティックトランザクションモード](/optimistic-transaction.md)
* [TiDB悲観的トランザクションモード](/pessimistic-transaction.md)