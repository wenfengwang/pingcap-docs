---
title: FLASHBACK TABLE
summary: `FLASHBACK TABLE` ステートメントを使用して、テーブルを復元する方法を学びます。
aliases: ['/docs/dev/sql-statements/sql-statement-flashback-table/','/docs/dev/reference/sql/statements/flashback-table/']
---

# FLASHBACK TABLE

TiDB 4.0 から導入された `FLASHBACK TABLE` 構文を使用して、`DROP` または `TRUNCATE` 操作によって削除されたテーブルおよびデータを Garbage Collection（GC）寿命内で復元できます。

システム変数 [`tidb_gc_life_time`](/system-variables.md#tidb_gc_life_time-new-in-v50)（デフォルト値: `10m0s`）は、以前の行のバージョンの保持期間を定義します。ガベージコレクションが実行された現在の `safePoint` は、次のクエリで取得できます：

{{< copyable "sql" >}}

```sql
SELECT * FROM mysql.tidb WHERE variable_name = 'tikv_gc_safe_point';
```

`DROP` または `TRUNCATE` ステートメントによってテーブルが `tikv_gc_safe_point` 時間以降に削除された場合は、`FLASHBACK TABLE` ステートメントを使用してテーブルを復元できます。

## 構文

{{< copyable "sql" >}}

```sql
FLASHBACK TABLE table_name [TO other_table_name]
```

## 概要

```ebnf+diagram
FlashbackTableStmt ::=
    'FLASHBACK' 'TABLE' TableName FlashbackToNewName

TableName ::=
    Identifier ( '.' Identifier )?

FlashbackToNewName ::=
    ( 'TO' Identifier )?
```

## メモ

テーブルが削除され、GC 寿命が経過した場合、`FLASHBACK TABLE` ステートメントを使用して削除されたデータを回復することはできません。それ以外の場合は、`Can't find dropped / truncated table 't' in GC safe point 2020-03-16 16:34:52 +0800 CST` のようなエラーが返されます。

TiDB バイナリログを有効にして `FLASHBACK TABLE` ステートメントを使用する際に注視すべき条件と要件に注意してください：

* 下流のセカンダリ クラスターも `FLASHBACK TABLE` をサポートする必要があります。
* 下流のクラスターの GC 寿命がプライマリ クラスターのそれよりも長くなければなりません。
* 上流と下流のレプリケーションの遅延がデータを下流に回復できない原因となる場合があります。
* TiDB バイナリログがテーブルをレプリケートしてエラーが発生した場合は、TiDB バイナリログでそのテーブルをフィルタリングし、そのテーブルのすべてのデータを手動でインポートする必要があります。

## 例

- `DROP` 操作によって削除されたテーブルのデータを回復する：

    {{< copyable "sql" >}}

    ```sql
    DROP TABLE t;
    ```

    {{< copyable "sql" >}}

    ```sql
    FLASHBACK TABLE t;
    ```

- `TRUNCATE` 操作によって削除されたテーブルのデータを回復する。`t` という切り捨てられたテーブルがまだ存在しているため、回復するためにテーブル `t` の名前を変更する必要があります。そうしないと、テーブル `t` が既に存在しているため、エラーが返されます。

    {{< copyable "sql" >}}

    ```sql
    TRUNCATE TABLE t;
    ```

    {{< copyable "sql" >}}

    ```sql
    FLASHBACK TABLE t TO t1;
    ```

## 実装原理

テーブルを削除する場合、TiDB はテーブルのメタデータのみを削除し、削除するテーブルデータ（行データおよび索引データ）を `mysql.gc_delete_range` テーブルに書き込みます。TiDB バックグラウンドの GC Worker は定期的に `mysql.gc_delete_range` テーブルから GC 寿命を超えるキーを削除します。

したがって、テーブルを回復するには、テーブルのメタデータを回復し、GC Worker がテーブルデータを削除する前に `mysql.gc_delete_range` テーブルの対応する行レコードを削除するだけです。テーブルメタデータを回復するには、TiDB のスナップショット読み取りを使用するだけです。スナップショット読み取りの詳細については、[履歴データの読み取り](/read-historical-data.md) を参照してください。

次は、`FLASHBACK TABLE t TO t1` の動作プロセスです：

1. TiDB は最近の DDL 履歴ジョブを検索し、テーブル `t` に対する最初の `DROP TABLE` 操作または `truncate table` タイプの DDL 操作を特定します。TiDB がこれを特定できない場合、エラーが返されます。
2. TiDB は、DDL ジョブの開始時間が `tikv_gc_safe_point` より前かどうかを確認します。開始時間が `tikv_gc_safe_point` より前であれば、`DROP` または `TRUNCATE` 操作によって削除されたテーブルが GC によってクリーンアップされていることを意味し、エラーが返されます。
3. TiDB は、DDL ジョブの開始時間をスナップショットとして使用して、履歴データを読み取り、テーブルメタデータを読み取ります。
4. TiDB は、`mysql.gc_delete_range` で `t` テーブルに関連する GC タスクを削除します。
5. TiDB は、テーブルのメタデータで `name` を `t1` に変更し、このメタデータを使用して新しいテーブルを作成します。テーブル名が変更されるだけで、テーブル ID は変更されません。テーブル ID は以前に削除されたテーブル `t` と同じです。

上記のプロセスから、TiDB が常にテーブルのメタデータで操作し、テーブルのユーザーデータが変更されたことはないことがわかります。回復されたテーブル `t1` の ID は以前に削除されたテーブル `t` の ID と同じですので、`t1` は `t` のユーザーデータを読み取ることができます。

> **注：**
>
> 同じ削除されたテーブルを複数回回復する場合は、`FLASHBACK` ステートメントを使用できません。回復されたテーブルの ID は削除されたテーブルの同じ ID である必要があるため、TiDB はすべての既存のテーブルにグローバルに一意なテーブル ID がある必要があります。

`FLASHBACK TABLE` の操作は、TiDB がスナップショット読み取りを介してテーブルメタデータを取得し、`CREATE TABLE` と同様のテーブル作成プロセスを経て行うことで実行されます。そのため、`FLASHBACK TABLE` は本質的には種類の一種の DDL 操作です。

## MySQL 互換性

このステートメントは、MySQL 構文への TiDB 拡張です。