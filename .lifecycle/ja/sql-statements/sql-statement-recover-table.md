---
title: テーブルのリカバリー
summary: TiDBデータベースでのRECOVER TABLEの使用方法の概要
aliases: ['/docs/dev/sql-statements/sql-statement-recover-table/','/docs/dev/reference/sql/statements/recover-table/']
---

# RECOVER TABLE

`RECOVER TABLE`は、`DROP TABLE`ステートメントの実行後のGC（Garbage Collection）ライフタイム内で、削除されたテーブルとそのデータを復元するために使用されます。

## 構文

{{< copyable "sql" >}}

```sql
RECOVER TABLE テーブル名;
```

{{< copyable "sql" >}}

```sql
RECOVER TABLE BY JOB JOB_ID;
```

## 概要

```ebnf+diagram
RecoverTableStmt ::=
    'RECOVER' 'TABLE' ( 'BY' 'JOB' Int64Num | TableName Int64Num? )

TableName ::=
    Identifier ( '.' Identifier )?

Int64Num ::= NUM

NUM ::= intLit
```

> **注意:**
>
> + テーブルが削除され、GCライフタイムが切れている場合、`RECOVER TABLE`でテーブルを回復することはできません。このシナリオでの`RECOVER TABLE`実行は、`snapshot is older than GC safe point 2019-07-10 13:45:57 +0800 CST`といったエラーを返します。
>
> + TiDBのバージョンが3.0.0以降の場合、TiDB Binlogを使用することは推奨されません。
>
> + `RECOVER TABLE`はBinlogバージョン3.0.1でサポートされているため、次の3つの状況で`RECOVER TABLE`を使用できます。
>
>     - Binlogのバージョンが3.0.1以降である。
>     - 上流クラスタと下流クラスタの両方でTiDB 3.0が使用されている。
>     - 下流のGCライフタイムが上流のGCライフタイムよりも長い必要があります。ただし、上流と下流のデータレプリケーション中に遅延が発生するため、下流でのデータの回復に失敗する可能性があります。

<CustomContent platform="tidb">

**TiDB Binlog Replication中のエラーのトラブルシューティング**

TiDB Binlogレプリケーション中に、TiDB Binlogが次の3つの状況で中断される可能性があります:

+ 下流のデータベースが`RECOVER TABLE`ステートメントをサポートしていない。エラー例: `check the manual that corresponds to your MySQL server version for the right syntax to use near 'RECOVER TABLE table_name'`。

+ 上流データベースと下流データベースのGCライフタイムが一致していない。エラー例: `snapshot is older than GC safe point 2019-07-10 13:45:57 +0800 CST`。

+ 上流と下流のデータベース間でのレプリケーション中に遅延が発生している。エラー例: `snapshot is older than GC safe point 2019-07-10 13:45:57 +0800 CST`。

上記の3つの状況については、[削除されたテーブルのフルインポート](/ecosystem-tool-user-guide.md#backup-and-restore---backup--restore-br)を使用して、TiDB Binlogからデータレプリケーションを再開することができます。

</CustomContent>

## 例

+ テーブル名に基づいて削除されたテーブルを回復する。

    {{< copyable "sql" >}}

    ```sql
    DROP TABLE t;
    ```

    {{< copyable "sql" >}}

    ```sql
    RECOVER TABLE t;
    ```

    この方法は、最新のDDLジョブ履歴を検索し、`DROP TABLE`タイプの最初のDDL操作を探し出し、その後`RECOVER TABLE`ステートメントで指定されたテーブル名と同じ名前の削除されたテーブルを回復します。

+ テーブルの`DDL JOB ID`に基づいて削除されたテーブルを回復する。

    `t`を削除し、別の`t`を作成し、再度新しく作成した`t`を削除した場合、最初に削除された`t`を回復したい場合は`DDL JOB ID`を指定する方法を使用する必要があります。

    {{< copyable "sql" >}}

    ```sql
    DROP TABLE t;
    ```

    {{< copyable "sql" >}}

    ```sql
    ADMIN SHOW DDL JOBS 1;
    ```

    上記の2番目のステートメントは、`t`を削除するためのテーブルの`DDL JOB ID`を検索するために使用されます。次の例では、そのIDは`53`です。

    ```
    +--------+---------+------------+------------+--------------+-----------+----------+-----------+-----------------------------------+--------+
    | JOB_ID | DB_NAME | TABLE_NAME | JOB_TYPE   | SCHEMA_STATE | SCHEMA_ID | TABLE_ID | ROW_COUNT | START_TIME                        | STATE  |
    +--------+---------+------------+------------+--------------+-----------+----------+-----------+-----------------------------------+--------+
    | 53     | test    |            | drop table | none         | 1         | 41       | 0         | 2019-07-10 13:23:18.277 +0800 CST | synced |
    +--------+---------+------------+------------+--------------+-----------+----------+-----------+-----------------------------------+--------+
    ```

    {{< copyable "sql" >}}

    ```sql
    RECOVER TABLE BY JOB 53;
    ```

    この方法は、`DDL JOB ID`経由で削除されたテーブルを回復します。対応するDDLジョブが`DROP TABLE`タイプでない場合、エラーが発生します。

## 実装原理

テーブルを削除する際、TiDBはテーブルのメタデータを削除し、削除されるテーブルデータ（行データとインデックスデータ）を`mysql.gc_delete_range`テーブルに書き込みます。TiDBバックグラウンドのGC Workerは定期的に`mysql.gc_delete_range`テーブルからGCライフタイムを超えるキーを削除します。

したがって、テーブルを回復するには、テーブルのメタデータを回復し、GC Workerがテーブルデータを削除する前に、`mysql.gc_delete_range`テーブルの対応する行レコードを削除する必要があります。テーブルのメタデータを回復するためにTiDBスナップショットリードを使用します。詳細は[履歴データの読み取り](/read-historical-data.md)を参照してください。

テーブルの回復は、TiDBがスナップショットリードを使用してテーブルのメタデータを取得し、その後`CREATE TABLE`に類似したテーブル作成プロセスを経て行われます。したがって、`RECOVER TABLE`自体が本質的には種類のDDL操作です。

## MySQL互換性

このステートメントはMySQL構文へのTiDBの拡張です。