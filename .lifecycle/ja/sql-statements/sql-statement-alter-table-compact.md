---
title: ALTER TABLE ... COMPACT
summary: ALTER TABLE ... COMPACTのTiDBデータベースでの使用法の概要
---

# ALTER TABLE ... COMPACT

TiDBでは、読み取りパフォーマンスを向上させ、ディスク使用量を削減するために、バックグラウンドでストレージノード上のデータのコンパクションを自動的にスケジュールします。コンパクション中、ストレージノードは物理データを書き換え、削除された行をクリーンアップし、更新によって引き起こされた複数のデータバージョンをマージします。`ALTER TABLE ... COMPACT` ステートメントを使用すると、バックグラウンドでコンパクションがトリガされるのを待たずに、特定のテーブルのコンパクションを即座に開始することができます。

このステートメントの実行は既存のSQLステートメントをブロックせず、トランザクション、DDL、およびGCなどのTiDBの機能にも影響を与えません。SQLステートメントによって選択できるデータも変更されません。このステートメントの実行には、いくらかのIOおよびCPUリソースが消費されます。ビジネスに負荷を与えないように、リソースが利用可能なタイミングなど、実行する適切なタイミングを選択するよう注意してください。

コンパクションステートメントは、テーブルのすべてのレプリカがコンパクションされると終了し、返されます。実行プロセス中は、[`KILL`](/sql-statements/sql-statement-kill.md)ステートメントを実行することでコンパクションを安全に中断することができます。コンパクションを中断しても、データの整合性が壊れたり、データの損失が発生したりすることはありません。その後の手動またはバックグラウンドのコンパクションにも影響しません。

このデータコンパクションステートメントは、TiFlashのレプリカのみをサポートしており、TiKVのレプリカはサポートしていません。

## 構文

```ebnf+diagram
AlterTableCompactStmt ::=
    'ALTER' 'TABLE' TableName 'COMPACT' ( 'PARTITION' PartitionNameList )? ( 'TIFLASH' 'REPLICA' )?
```

v6.2.0以降、構文の`TIFLASH REPLICA`部分は省略可能になりました。省略した場合、ステートメントのセマンティクスは変わらず、TiFlashにのみ影響します。

## 例

### テーブル内のTiFlashレプリカをコンパクト化

次の例は、2つのTiFlashレプリカを持つ`employees`テーブルを取り上げます。

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    hired DATE NOT NULL DEFAULT '1970-01-01',
    store_id INT
)
PARTITION BY LIST (store_id) (
    PARTITION pNorth VALUES IN (1, 2, 3, 4, 5),
    PARTITION pEast VALUES IN (6, 7, 8, 9, 10),
    PARTITION pWest VALUES IN (11, 12, 13, 14, 15),
    PARTITION pCentral VALUES IN (16, 17, 18, 19, 20)
);
ALTER TABLE employees SET TIFLASH REPLICA 2;
```

以下のステートメントを実行して、`employees`テーブルのすべてのパーティションの2つのTiFlashレプリカについて、コンパクションを即座に開始できます:

{{< copyable "sql" >}}

```sql
ALTER TABLE employees COMPACT TIFLASH REPLICA;
```

### テーブル内の指定されたパーティションのTiFlashレプリカをコンパクト化

次の例は、2つのTiFlashレプリカを持つ`employees`テーブルを取り上げます。

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    hired DATE NOT NULL DEFAULT '1970-01-01',
    store_id INT
)
PARTITION BY LIST (store_id) (
    PARTITION pNorth VALUES IN (1, 2, 3, 4, 5),
    PARTITION pEast VALUES IN (6, 7, 8, 9, 10),
    PARTITION pWest VALUES IN (11, 12, 13, 14, 15),
    PARTITION pCentral VALUES IN (16, 17, 18, 19, 20)
);

ALTER TABLE employees SET TIFLASH REPLICA 2;
```

以下のステートメントを実行して、`employees`テーブルの`pNorth`および`pEast`パーティションの2つのTiFlashレプリカについて、コンパクションを即座に開始できます:

```sql
ALTER TABLE employees COMPACT PARTITION pNorth, pEast TIFLASH REPLICA;
```

## 並行性

`ALTER TABLE ... COMPACT` ステートメントは、テーブル内のすべてのレプリカを同時にコンパクションします。

オンラインビジネスに大きな影響を与えないようにするために、デフォルトでは、各TiFlashインスタンスは1度に1つのテーブルのデータのみをコンパクションします(バックグラウンドでトリガされるコンパクションを除く)。これはつまり、複数のテーブルで同時に`ALTER TABLE ... COMPACT`ステートメントを実行した場合、それらの実行は同じTiFlashインスタンスでキューイングされることになり、同時に実行されるわけではありません。

<CustomContent platform="tidb">

より高いリソース使用量でより高いテーブルレベルの並行性を得るためには、TiFlashの構成[`manual_compact_pool_size`](/tiflash/tiflash-configuration.md)を変更することができます。例えば、`manual_compact_pool_size`が2に設定されている場合、2つのテーブルのコンパクションを同時に処理できます。

</CustomContent>

## データコンパクションの進捗状況の確認

`INFORMATION_SCHEMA.TIFLASH_TABLES` テーブルの `TOTAL_DELTA_ROWS` 列を確認することで、データコンパクションの進捗状況を観察したり、テーブルのコンパクションを開始するかどうかを決定することができます。`TOTAL_DELTA_ROWS` の値が大きいほど、コンパクションできるデータが多いことを示します。`TOTAL_DELTA_ROWS` が `0` の場合、テーブルのすべてのデータが最適な状態にあり、コンパクションの必要はありません。

<details>
  <summary>例: パーティションされていないテーブルのコンパクション状態をチェック</summary>

```sql
USE test;

CREATE TABLE foo(id INT);

ALTER TABLE foo SET TIFLASH REPLICA 1;

SELECT TOTAL_DELTA_ROWS, TOTAL_STABLE_ROWS FROM INFORMATION_SCHEMA.TIFLASH_TABLES
    WHERE IS_TOMBSTONE = 0 AND
    `TIDB_DATABASE` = "test" AND `TIDB_TABLE` = "foo";
+------------------+-------------------+
| TOTAL_DELTA_ROWS | TOTAL_STABLE_ROWS |
+------------------+-------------------+
|                0 |                 0 |
+------------------+-------------------+

INSERT INTO foo VALUES (1), (3), (7);

SELECT TOTAL_DELTA_ROWS, TOTAL_STABLE_ROWS FROM INFORMATION_SCHEMA.TIFLASH_TABLES
    WHERE IS_TOMBSTONE = 0 AND
    `TIDB_DATABASE` = "test" AND `TIDB_TABLE` = "foo";
+------------------+-------------------+
| TOTAL_DELTA_ROWS | TOTAL_STABLE_ROWS |
+------------------+-------------------+
|                3 |                 0 |
+------------------+-------------------+
-- 新たに書き込まれたデータはコンパクションできます

ALTER TABLE foo COMPACT TIFLASH REPLICA;

SELECT TOTAL_DELTA_ROWS, TOTAL_STABLE_ROWS FROM INFORMATION_SCHEMA.TIFLASH_TABLES
    WHERE IS_TOMBSTONE = 0 AND
    `TIDB_DATABASE` = "test" AND `TIDB_TABLE` = "foo";
+------------------+-------------------+
| TOTAL_DELTA_ROWS | TOTAL_STABLE_ROWS |
+------------------+-------------------+
|                0 |                 3 |
+------------------+-------------------+
-- すべてのデータが最適な状態であり、コンパクションは必要ありません
```

</details>

<details>
  <summary>例: パーティションされたテーブルのコンパクション状態をチェック</summary>

```sql
USE test;

CREATE TABLE employees
    (id INT NOT NULL, store_id INT)
    PARTITION BY LIST (store_id) (
        PARTITION pNorth VALUES IN (1, 2, 3, 4, 5),
        PARTITION pEast VALUES IN (6, 7, 8, 9, 10),
        PARTITION pWest VALUES IN (11, 12, 13, 14, 15),
        PARTITION pCentral VALUES IN (16, 17, 18, 19, 20)
    );

ALTER TABLE employees SET TIFLASH REPLICA 1;

INSERT INTO employees VALUES (1, 1), (6, 6), (10, 10);

SELECT PARTITION_NAME, TOTAL_DELTA_ROWS, TOTAL_STABLE_ROWS
    FROM INFORMATION_SCHEMA.TIFLASH_TABLES t, INFORMATION_SCHEMA.PARTITIONS p
    WHERE t.IS_TOMBSTONE = 0 AND t.TABLE_ID = p.TIDB_PARTITION_ID AND
    p.TABLE_SCHEMA = "test" AND p.TABLE_NAME = "employees";
+----------------+------------------+-------------------+
| PARTITION_NAME | TOTAL_DELTA_ROWS | TOTAL_STABLE_ROWS |
+----------------+------------------+-------------------+
| pNorth         |                1 |                 0 |
| pEast          |                2 |                 0 |
| pWest          |                0 |                 0 |
| pCentral       |                0 |                 0 |
+----------------+------------------+-------------------+
-- いくつかのパーティションはコンパクションできます

ALTER TABLE employees COMPACT TIFLASH REPLICA;

SELECT PARTITION_NAME, TOTAL_DELTA_ROWS, TOTAL_STABLE_ROWS
    FROM INFORMATION_SCHEMA.TIFLASH_TABLES t, INFORMATION_SCHEMA.PARTITIONS p
    WHERE t.IS_TOMBSTONE = 0 AND t.TABLE_ID = p.TIDB_PARTITION_ID AND
    p.TABLE_SCHEMA = "test" AND p.TABLE_NAME = "employees";
+----------------+------------------+-------------------+
| PARTITION_NAME | TOTAL_DELTA_ROWS | TOTAL_STABLE_ROWS |
+----------------+------------------+-------------------+
| pNorth         |                0 |                 1 |
| pEast          |                0 |                 2 |
| pWest          |                0 |                 0 |
| pCentral       |                0 |                 0 |
+----------------+------------------+-------------------+
-- すべてのパーティションのデータが最良の状態にあり、コンパクションは必要ありません
```

</details>

> **注意:**
>
> - コンパクション中にデータが更新された場合、`TOTAL_DELTA_ROWS` はコンパクションが完了した後でもゼロでない値のままになることがあります。これは正常であり、これらの更新がまだコンパクトされていないことを示します。これらの更新をコンパクトするには、`ALTER TABLE ... COMPACT` ステートメントを再度実行してください。
>
> - `TOTAL_DELTA_ROWS` はデータのバージョンを示し、行数ではありません。たとえば、行を挿入してから削除すると、`TOTAL_DELTA_ROWS` は 2 増加します。

## 互換性

### MySQL 互換性

`ALTER TABLE ... COMPACT` 構文は TiDB 固有のものであり、標準SQL構文の拡張です。MySQL 互換の構文は存在しませんが、MySQLクライアントやMySQLプロトコルをサポートするさまざまなデータベースドライバを使用して、このステートメントを実行できます。

### TiDB バイナリログおよび TiCDC 互換性

`ALTER TABLE ... COMPACT` ステートメントは論理的なデータ変更を引き起こさず、したがって TiDB バイナリログや TiCDC によってダウンストリームにレプリケートされません。

## 関連情報

- [ALTER TABLE](/sql-statements/sql-statement-alter-table.md)
- [KILL TIDB](/sql-statements/sql-statement-kill.md)