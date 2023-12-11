---
title: システム変数 `tidb_snapshot` を使用して歴史データを読む
summary: システム変数 `tidb_snapshot` を使用して、TiDB が歴史バージョンからデータを読む方法について説明します。
aliases: ['/docs/dev/read-historical-data/','/docs/dev/how-to/get-started/read-historical-data/']

# システム変数 `tidb_snapshot` を使用して歴史データを読む

このドキュメントでは、システム変数 `tidb_snapshot` を使用して、歴史バージョンからデータを読む方法について、具体的な使用例や歴史データの保存戦略について説明します。

> **ノート:**
>
> [Stale Read](/stale-read.md) 機能を使用して歴史データを読むこともできます。こちらが推奨される方法です。

## 機能の説明

TiDB は、特別なクライアントやドライバを使用せずに、標準の SQL インターフェースを使用して歴史データを読む機能を実装しています。

> **ノート:**
>
> - データが更新または削除されても、SQL インターフェースを使用してその歴史バージョンのデータを読むことができます。
> - 歴史データを読む際、TiDB は、現在のテーブル構造が異なっていても、その古いテーブル構造でデータを返します。

## TiDB が歴史バージョンからデータを読む方法

[`tidb_snapshot`](/system-variables.md#tidb_snapshot) システム変数は、歴史データの読み込みをサポートするために導入されています。 `tidb_snapshot` 変数について:

- この変数は `SESSION` スコープで有効です。
- その値は `SET` ステートメントを使用して変更できます。
- 変数のデータ型はテキストです。
- この変数は TSO (Timestamp Oracle) および日時を受け入れます。TSO は Placement Driver (PD) から取得されるグローバルに一意な時刻サービスです。許容される日時形式は "2016-10-08 16:45:26.999" です。一般的には、日時は秒単位の精度で設定できます。たとえば "2016-10-08 16:45:26" です。
- 変数が設定されると、TiDB はその値をタイムスタンプとして使用してスナップショットを作成しますが、データ構造だけで、オーバーヘッドはありません。その後、すべての `SELECT` 操作はこのスナップショットからデータを読み取ります。

> **ノート:**
>
> TiDB トランザクションのタイムスタンプは Placement Driver (PD) によって割り当てられるため、格納されたデータのバージョンも PD によって割り当てられたタイムスタンプに基づいてマークされます。スナップショットが作成されると、バージョン番号は `tidb_snapshot` 変数の値に基づいています。TiDB サーバのローカル時間と PD サーバの時間に大きな差がある場合は、PD サーバの時間を使用してください。

歴史バージョンからデータを読み込んだ後は、現在のセッションを終了するか、`SET` ステートメントを使用して `tidb_snapshot` 変数の値を "" (空の文字列) に設定することで、最新バージョンからデータを読むことができます。

## TiDB がデータバージョンを管理する方法

TiDB は Multi-Version Concurrency Control (MVCC) を実装してデータバージョンを管理しています。データの歴史バージョンは、更新または削除ごとにデータオブジェクトの新しいバージョンが作成されるため保持されます。ただし、すべてのバージョンが保持されるわけではありません。バージョンが特定の時間よりも古い場合、格納占有率を減らし、過剰な歴史バージョンによって引き起こされるパフォーマンスオーバーヘッドを削減するため、それらは完全に削除されます。

TiDB では、Garbage Collection (GC) が定期的に実行され、古いバージョンのデータが削除されます。GC の詳細については、[TiDB Garbage Collection (GC)](/garbage-collection-overview.md) を参照してください。

以下に特に注意してください:

- [`tidb_gc_life_time`](/system-variables.md#tidb_gc_life_time-new-in-v50): このシステム変数は、早期の変更の保持時間を構成するために使用されます (デフォルト値: `10m0s`)。
- `SELECT * FROM mysql.tidb WHERE variable_name = 'tikv_gc_safe_point'` の出力。これは現在の `safePoint` であり、これまでに読み取ることができる歴史データです。ゴミ収集プロセスが実行されるたびに更新されます。

## 例

1. 最初の段階で、テーブルを作成し、いくつかのデータ行を挿入します:

    ```sql
    mysql> create table t (c int);
    Query OK, 0 rows affected (0.01 sec)

    mysql> insert into t values (1), (2), (3);
    Query OK, 3 rows affected (0.00 sec)
    ```

2. テーブルのデータを表示します:

    ```sql
    mysql> select * from t;
    +------+
    | c    |
    +------+
    |    1 |
    |    2 |
    |    3 |
    +------+
    3 rows in set (0.00 sec)
    ```

3. テーブルのタイムスタンプを表示します:

    ```sql
    mysql> select now();
    +---------------------+
    | now()               |
    +---------------------+
    | 2016-10-08 16:45:26 |
    +---------------------+
    1 row in set (0.00 sec)
    ```

4. 1行のデータを更新します:

    ```sql
    mysql> update t set c=22 where c=2;
    Query OK, 1 row affected (0.00 sec)
    ```

5. データが更新されたことを確認します:

    ```sql
    mysql> select * from t;
    +------+
    | c    |
    +------+
    |    1 |
    |   22 |
    |    3 |
    +------+
    3 rows in set (0.00 sec)
    ```

6. セッションスコープの `tidb_snapshot` 変数を設定します。この変数は、その値の前の最新バージョンを読むように設定されています。

    > **ノート:**
    >
    > この例では、その値を更新操作より前の時間に設定しています。

    ```sql
    mysql> set @@tidb_snapshot="2016-10-08 16:45:26";
    Query OK, 0 rows affected (0.00 sec)
    ```

    > **ノート:**
    >
    > `@` の代わりに `@@` を使用する必要があります。`@@` はシステム変数を示すために使用され、`@` はユーザ変数を示すために使用されます。

    **結果:** 以下のステートメントからの読み取りは、更新操作の前のデータである歴史データです。

    ```sql
    mysql> select * from t;
    +------+
    | c    |
    +------+
    |    1 |
    |    2 |
    |    3 |
    +------+
    3 rows in set (0.00 sec)
    ```

7. `tidb_snapshot` 変数を "" (空の文字列) に設定し、最新バージョンからデータを読むことができます:

    ```sql
    mysql> set @@tidb_snapshot="";
    Query OK, 0 rows affected (0.00 sec)
    ```

    ```sql
    mysql> select * from t;
    +------+
    | c    |
    +------+
    |    1 |
    |   22 |
    |    3 |
    +------+
    3 rows in set (0.00 sec)
    ```

    > **ノート:**
    >
    > `@` の代わりに `@@` を使用する必要があります。`@@` はシステム変数を示すために使用され、`@` はユーザ変数を示すために使用されます。

## 歴史データの復元方法

古いバージョンからデータを復元する前に、ゴミ収集 (GC) が作業中に歴史データをクリアしないようにする必要があります。次の例のように、`tidb_gc_life_time` 変数を設定することでこれができます。復元後に変数を前の値に設定し直すことを忘れないでください。

```sql
SET GLOBAL tidb_gc_life_time="60m";
```

> **ノート:**
>
> デフォルトの 10分から30分以上の GC ライフタイムを増やすと、より多くの行の追加バージョンが保持されるため、より多くのディスク容量が必要になる可能性があります。また、データの読み取り中にこれらの行の余分なバージョンをスキップする必要がある場合、スキャンなどの特定の操作のパフォーマンスに影響を与える可能性があります。

古いバージョンからデータを復元するには、以下のいずれかの方法を使用できます:

- 単純なケースでは、`tidb_snapshot` 変数を設定した後に[`SELECT`](/sql-statements/sql-statement-select.md) を使用して出力をコピーし、または `SELECT ... INTO OUTFILE` を使用してデータをエクスポートし、後で [`LOAD DATA`](/sql-statements/sql-statement-load-data.md) を使用してデータをインポートします。

- [Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview#export-historical-data-snapshots-of-tidb) を使用して、歴史的なスナップショットをエクスポートします。Dumpling は、大量のデータをエクスポートする際に優れたパフォーマンスを発揮します。