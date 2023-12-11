---
title: FLASHBACK DATABASE
summary: TiDBデータベースでFLASHBACK DATABASEの使用方法を学びます。

# FLASHBACK DATABASE

TiDB v6.4.0では、`FLASHBACK DATABASE`構文が追加されました。`FLASHBACK DATABASE`を使用して、Garbage Collection（GC）寿命内で`DROP`ステートメントによって削除されたデータベースとそのデータを復元できます。

[`tidb_gc_life_time`](/system-variables.md#tidb_gc_life_time-new-in-v50)システム変数を構成することで、履歴データの保持期間を設定できます。デフォルト値は`10m0s`です。次のSQLステートメントを使用して、現在の`safePoint`（つまり、GCが実行された時点）をクエリできます。

```sql
SELECT * FROM mysql.tidb WHERE variable_name = 'tikv_gc_safe_point';
```

`tikv_gc_safe_point`の時間の後に`DROP`によってデータベースが削除される限り、`FLASHBACK DATABASE`を使用してデータベースを復元できます。

## 構文

```sql
FLASHBACK DATABASE データベース名 [TO 新しいDB名]
```

### 概要

```ebnf+diagram
FlashbackDatabaseStmt ::=
    'FLASHBACK' DatabaseSym DBName FlashbackToNewName
FlashbackToNewName ::=
    ( 'TO' Identifier )?
```

## 注記

* データベースが`tikv_gc_safe_point`の時間よりも前に削除された場合、`FLASHBACK DATABASE`ステートメントを使用してデータを復元することはできません。`FLASHBACK DATABASE`ステートメントは、`ERROR 1105 (HY000): Can't find dropped database 'test' in GC safe point 2022-11-06 16:10:10 +0800 CST`のようなエラーを返します。

* `FLASHBACK DATABASE`ステートメントを使用して同じデータベースを複数回復元することはできません。`FLASHBACK DATABASE`によって復元されたデータベースは元のデータベースと同じスキーマIDを持っているため、同じデータベースを複数回復元すると重複したスキーマIDが発生します。TiDBでは、データベースのスキーマIDはグローバルに一意である必要があります。

* TiDB Binlogを有効にしている場合、`FLASHBACK DATABASE`を使用する際には以下の点に注意してください。

    * 下流のセカンダリデータベースは`FLASHBACK DATABASE`をサポートする必要があります。
    * 下流のデータベースのGC寿命は、上流の寿命よりも長くする必要があります。そうでない場合、上流と下流の間の遅延がデータ復元の失敗につながる可能性があります。
    * TiDB Binlogレプリケーションでエラーが発生した場合、TiDB Binlogからデータベースをフィルタリングし、このデータベースの完全なデータを手動でインポートする必要があります。

## 例

- `test`データベースを`DROP`で削除した後に復元する場合：

    ```sql
    DROP DATABASE test;
    ```

    ```sql
    FLASHBACK DATABASE test;
    ```

- `test`データベースを`DROP`で削除し、それを`test1`という名前で復元する場合：

    ```sql
    DROP DATABASE test;
    ```

    ```sql
    FLASHBACK DATABASE test TO test1;
    ```

## MySQL互換性

このステートメントは、MySQL構文へのTiDBの拡張です。