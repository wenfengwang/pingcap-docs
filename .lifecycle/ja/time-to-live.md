---
title: 定期的にTTL (Time to Live) を使用してデータを削除する
summary: Time to Live (TTL) は、TiDBデータの寿命を行レベルで管理する機能です。このドキュメントでは、TTLを使用して古いデータを自動的に期限切れにし、削除する方法を学ぶことができます。
---

# TTL (Time to Live) を使用して期限切れのデータを定期的に削除する

Time to Live (TTL) は、TiDBデータの寿命を行レベルで管理する機能です。TTL属性があるテーブルでは、TiDBがデータの寿命を自動的に確認し、行レベルで期限切れのデータを削除します。この機能は、特定のシナリオでストレージスペースを効果的に節約し、パフォーマンスを向上させることができます。

以下は、TTLの一般的な利用シナリオです：

- 定期的に検証コードや短縮URLを削除する。
- 不要な歴史的注文を定期的に削除する。
- 計算の中間結果を自動的に削除する。

TTLは、オンラインの読み取りおよび書き込みワークロードに影響を与えることなく、定期的かつタイムリーに不要なデータを整理するために設計されています。TTLは異なるTiDBノードに異なるジョブを同時にディスパッチし、テーブルの単位でデータを並列に削除します。TTLは、すべての期限切れのデータがすぐに削除されることを保証しません。これは、一部のデータが期限切れであっても、背景でTTLジョブによってそのデータが削除されるまで、クライアントがそのデータを期限切れ後のある時点まで読み取る可能性があることを意味します。

## 構文

[`CREATE TABLE`](/sql-statements/sql-statement-create-table.md)または[`ALTER TABLE`](/sql-statements/sql-statement-alter-table.md)ステートメントを使用して、テーブルのTTL属性を構成できます。

### TTL属性を持つテーブルを作成する

- TTL属性を持つテーブルを作成する：

    ```sql
    CREATE TABLE t1 (
        id int PRIMARY KEY,
        created_at TIMESTAMP
    ) TTL = `created_at` + INTERVAL 3 MONTH;
    ```

    上記の例では、`created_at`をTTLタイムスタンプ列として指定し、データの作成時間を示します。また、`INTERVAL 3 MONTH`を使用してテーブル内の行の最長生存時間を3か月に設定します。この値を超えるデータは後で削除されます。

- `TTL_ENABLE`属性を設定して期限切れのデータのクリーンアップ機能を有効または無効にする：

    ```sql
    CREATE TABLE t1 (
        id int PRIMARY KEY,
        created_at TIMESTAMP
    ) TTL = `created_at` + INTERVAL 3 MONTH TTL_ENABLE = 'OFF';
    ```

    `TTL_ENABLE`が`OFF`に設定されている場合、他のTTLオプションが設定されていても、TiDBはこのテーブルの期限切れのデータを自動的にクリーンアップしません。TTL属性を持つテーブルの場合、`TTL_ENABLE`はデフォルトで`ON`になります。

- MySQLとの互換性を保つために、コメントを使用してTTL属性を設定できます：

    ```sql
    CREATE TABLE t1 (
        id int PRIMARY KEY,
        created_at TIMESTAMP
    ) /*T![ttl] TTL = `created_at` + INTERVAL 3 MONTH TTL_ENABLE = 'OFF'*/;
    ```

    TiDBでは、テーブルのTTL属性を使用したり、コメントを使用してTTLを構成したりすることは同等です。MySQLでは、コメントは無視され、通常のテーブルが作成されます。

### テーブルのTTL属性を変更する

- テーブルのTTL属性を変更する：

    ```sql
    ALTER TABLE t1 TTL = `created_at` + INTERVAL 1 MONTH;
    ```

    上記のステートメントを使用して、既存のTTL属性のあるテーブルを変更したり、TTL属性のないテーブルにTTL属性を追加したりできます。

- TTL属性を持つテーブルの`TTL_ENABLE`の値を変更する：

    ```sql
    ALTER TABLE t1 TTL_ENABLE = 'OFF';
    ```

- テーブルのすべてのTTL属性を削除する：

    ```sql
    ALTER TABLE t1 REMOVE TTL;
    ```

### TTLとデータ型のデフォルト値

TTLを[データ型のデフォルト値](/data-type-default-values.md)と一緒に使用できます。以下は、2つの一般的な使用例です：

- `DEFAULT CURRENT_TIMESTAMP`を使用して、列のデフォルト値を現在の作成時間として指定し、この列をTTLタイムスタンプ列として使用します。3か月前に作成されたレコードが期限切れです：

    ```sql
    CREATE TABLE t1 (
        id int PRIMARY KEY,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    ) TTL = `created_at` + INTERVAL 3 MONTH;
    ```

- 列のデフォルト値を作成時間または最新の更新時間として指定し、この列をTTLタイムスタンプ列として使用します。3か月間更新されていないレコードが期限切れです：

    ```sql
    CREATE TABLE t1 (
        id int PRIMARY KEY,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    ) TTL = `created_at` + INTERVAL 3 MONTH;
    ```

### TTLと生成列

[TTLと生成列](/generated-columns.md)を使用して、複雑な期限切れルールを構成できます。例：

```sql
CREATE TABLE message (
    id int PRIMARY KEY,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    image bool,
    expire_at TIMESTAMP AS (IF(image,
            created_at + INTERVAL 5 DAY,
            created_at + INTERVAL 30 DAY
    ))
) TTL = `expire_at` + INTERVAL 0 DAY;
```

上記のステートメントでは、`expire_at`列をTTLタイムスタンプ列として使用し、メッセージのタイプに応じて有効期限を設定しています。メッセージが画像の場合、5日後に期限切れになります。それ以外の場合は30日後に期限切れになります。

[JSON型](/data-type-json.md)とTTLを一緒に使用できます。例：

```sql
CREATE TABLE orders (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    order_info JSON,
    created_at DATE AS (JSON_EXTRACT(order_info, '$.created_at')) VIRTUAL
) TTL = `created_at` + INTERVAL 3 month;
```

## TTLジョブ

TTL属性の付いた各テーブルについて、TiDBは内部で期限切れデータをクリーンアップするバックグラウンドジョブをスケジュールします。これらのジョブの実行期間を`TTL_JOB_INTERVAL`属性を設定することでカスタマイズできます。以下の例では、テーブル`orders`のバックグラウンドクリーンアップジョブを24時間ごとに実行するように設定しています：

```sql
ALTER TABLE orders TTL_JOB_INTERVAL = '24h';
```

`TTL_JOB_INTERVAL`はデフォルトで`1h`に設定されています。

TTLジョブを実行する際、TiDBはテーブルを最大で64タスクに分割し、Regionが最小単位となります。これらのタスクは分散して実行されます。クラスタ全体での同時TTLタスクの数を制限するために、システム変数[`tidb_ttl_running_tasks`](/system-variables.md#tidb_ttl_running_tasks-new-in-v700)を設定できます。ただし、すべての種類のテーブルのTTLジョブがタスクに分割できるわけではありません。どの種類のテーブルのTTLジョブがタスクに分割できないかの詳細については、[制限事項](#limitations)セクションを参照してください。

TTLジョブの実行を無効にするには、`TTL_ENABLE='OFF'`のテーブルオプションを設定するだけでなく、[`tidb_ttl_job_enable`](/system-variables.md#tidb_ttl_job_enable-new-in-v650)グローバル変数を設定することで、クラスタ全体でTTLジョブの実行を無効にすることもできます。

一部のシナリオでは、TTLジョブを特定の時間枠でのみ実行するようにすることが望ましい場合があります。この場合、[`tidb_ttl_job_schedule_window_start_time`](/system-variables.md#tidb_ttl_job_schedule_window_start_time-new-in-v650)と[`tidb_ttl_job_schedule_window_end_time`](/system-variables.md#tidb_ttl_job_schedule_window_end_time-new-in-v650)グローバル変数を使用して、実行時間のウィンドウを指定できます。例：

```sql
SET @@global.tidb_ttl_job_schedule_window_start_time = '01:00 +0000';
SET @@global.tidb_ttl_job_schedule_window_end_time = '05:00 +0000';
```

上記のステートメントでは、TTLジョブをUTCの1:00から5:00までの間のみスケジュールするように設定しています。デフォルトでは、時間枠は`00:00 +0000`から`23:59 +0000`に設定されており、ジョブはいつでもスケジュールできます。

## 観測

<CustomContent platform="tidb-cloud">

> **注：**
>
> このセクションはTiDB Cloudにのみ適用されます。現在、TiDB CloudではTTLメトリクスは提供されていません。

</CustomContent>

TiDBは定期的にTTLに関する実行情報を収集し、これらのメトリクスの視覚化チャートをGrafanaで提供します。GrafanaのTiDB -> TTLパネルでこれらのメトリクスを確認できます。

<CustomContent platform="tidb">

メトリクスの詳細については、[TiDB監視メトリクス](/grafana-tidb-dashboard.md)のTTLセクションを参照してください。

</CustomContent>

さらに、TiDBはTTLジョブに関する情報を取得するための3つのテーブルを提供しています：

+ `mysql.tidb_ttl_table_status`テーブルには、すべてのTTLテーブルについて、以前に実行されたTTLジョブや現在実行中のTTLジョブに関する情報が含まれています

    ```sql
    MySQL [(none)]> SELECT * FROM mysql.tidb_ttl_table_status LIMIT 1\G;
    *************************** 1. row ***************************
                          table_id: 85
                  parent_table_id: 85
                  table_statistics: NULL
                      last_job_id: 0b4a6d50-3041-4664-9516-5525ee6d9f90
              last_job_start_time: 2023-02-15 20:43:46
              last_job_finish_time: 2023-02-15 20:44:46
              last_job_ttl_expire: 2023-02-15 19:43:46
                  last_job_summary: {"total_rows":4369519,"success_rows":4369519,"error_rows":0,"total_scan_task":64,"scheduled_scan_task":64,"finished_scan_task":64}
                    current_job_id: NULL
              current_job_owner_id: NULL
            current_job_owner_addr: NULL
        current_job_owner_hb_time: NULL
            current_job_start_time: NULL
            current_job_ttl_expire: NULL
                current_job_state: NULL
                current_job_status: NULL
    current_job_status_update_time: NULL
    1 行が返されました (0.040 秒)

    カラム`table_id`は分割テーブルのIDであり、`parent_table_id`は`infomation_schema.tables`内のIDと対応しています。テーブルが分割テーブルでない場合、これら2つのIDは同じです。

    カラム`{last, current}_job_{start_time, finish_time, ttl_expire}`はそれぞれ、直近または現在の実行のTTLジョブによって使用される開始時刻、終了時刻、有効期限時刻を示しています。`last_job_summary`カラムは、直近のTTLタスクの実行状況を説明し、総行数、成功した行数、失敗した行数などが含まれています。

+ `mysql.tidb_ttl_task`テーブルには進行中のTTLサブタスクに関する情報が含まれています。TTLジョブは多くのサブタスクに分割され、このテーブルは現在実行されているサブタスクを記録しています。
+ `mysql.tidb_ttl_job_history`テーブルには実行されたTTLジョブに関する情報が含まれています。TTLジョブの実行履歴は90日間保持されます。

    ```sql
    MySQL [(none)]> SELECT * FROM mysql.tidb_ttl_job_history LIMIT 1\G;
    *************************** 1. row ***************************
              job_id: f221620c-ab84-4a28-9d24-b47ca2b5a301
            table_id: 85
      parent_table_id: 85
        table_schema: test_schema
          table_name: TestTable
      partition_name: NULL
          create_time: 2023-02-15 17:43:46
          finish_time: 2023-02-15 17:45:46
          ttl_expire: 2023-02-15 16:43:46
        summary_text: {"total_rows":9588419,"success_rows":9588419,"error_rows":0,"total_scan_task":63,"scheduled_scan_task":63,"finished_scan_task":63}
        expired_rows: 9588419
        deleted_rows: 9588419
    error_delete_rows: 0
              status: finished
    ```

    カラム`table_id`は分割テーブルのIDであり、`parent_table_id`は`infomation_schema.tables`内のIDと対応しています。`table_schema`、`table_name`、`partition_name`はそれぞれ、データベース、テーブル名、パーティション名に対応しています。`create_time`、`finish_time`、`ttl_expire` はTTLタスクの作成時刻、終了時刻、有効期限時刻を示します。`expired_rows` と `deleted_rows` はそれぞれ有効期限切れの行数、正常に削除された行数を示します。

## TiDBツールとの互換性

TTLは他のTiDBの移行、バックアップ、リカバリツールと併用することができます。

| ツール名 | サポートされる最低バージョン | 説明 |
| --- | --- | --- |
| バックアップ＆リストア（BR） | v6.6.0 | BRを使用してデータをリストアした後、テーブルの `TTL_ENABLE` 属性が `OFF` に設定されます。これにより、バックアップおよびリストア後に期限切れデータがすぐに削除されなくなります。各テーブルで `TTL_ENABLE` 属性を手動で有効にする必要があります。 |
| TiDBライトニング | v6.6.0 | TiDBライトニングを使用してデータをインポートした後、インポートされたテーブルの `TTL_ENABLE` 属性が `OFF` に設定されます。これにより、インポート後に期限切れデータがすぐに削除されなくなります。各テーブルで `TTL_ENABLE` 属性を手動で有効にする必要があります。 |
| TiCDC | v7.0.0 | 下流の`TTL_ENABLE` 属性は自動的に `OFF` に設定されます。上流のTTL削除は下流に同期されます。そのため、重複した削除を防ぐために、下流テーブルの `TTL_ENABLE` 属性は強制的に `OFF` に設定されます。 |

## SQLとの互換性

| 機能名 | 説明 |
| :-- | :---- |
| [`FLASHBACK TABLE`](/sql-statements/sql-statement-flashback-table.md) | `FLASHBACK TABLE` はテーブルの `TTL_ENABLE` 属性を `OFF` に設定します。これにより、フラッシュバック後に期限切れデータがすぐに削除されなくなります。各テーブルで `TTL_ENABLE` 属性を手動で有効にする必要があります。 |
| [`FLASHBACK DATABASE`](/sql-statements/sql-statement-flashback-database.md) | `FLASHBACK DATABASE` はテーブルの `TTL_ENABLE` 属性を `OFF` に設定し、`TTL_ENABLE` 属性は変更されません。これにより、フラッシュバック後に期限切れデータがすぐに削除されなくなります。各テーブルで `TTL_ENABLE` 属性を手動で有効にする必要があります。 |
| [`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md) | `FLASHBACK CLUSTER TO TIMESTAMP` は、システム変数 [`TIDB_TTL_JOB_ENABLE`](/system-variables.md#tidb_ttl_job_enable-new-in-v650) を `OFF` に設定し、`TTL_ENABLE` 属性の値は変更されません。 |

## 制限事項

現在、TTL機能には以下の制限があります:

* TTL属性は一時テーブルに対して設定できません。これにはローカル一時テーブルやグローバル一時テーブルが含まれます。
* TTL属性を持つテーブルは、他のテーブルから外部キー制約のプライマリテーブルとして参照されることをサポートしていません。
* 期限切れデータがすぐに削除されることは保証されません。期限切れデータの削除時刻は、バックグラウンドクリーンアップジョブのスケジューリング間隔およびスケジューリングウィンドウに依存します。
* [クラスタ化インデックス](/clustered-indexes.md)を使用するテーブルの場合、プライマリキーが整数型またはバイナリ文字列型でない場合、TTLジョブを複数のタスクに分割することができません。これにより、TTLジョブが単一のTiDBノードで順次実行されます。テーブルに大量のデータが含まれる場合、TTLジョブの実行が遅くなる可能性があります。
* TTLは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)には使用できません。

## FAQs

<CustomContent platform="tidb">

- データ削除が相対的に安定した状態を保つために削除が十分に迅速であるかどうかをどのように判断できますか？

    [Grafana `TiDB` ダッシュボード](/grafana-tidb-dashboard.md)では、`TTL Insert Rows Per Hour` パネルが前の1時間に挿入された行の合計数を記録しています。これに対応する `TTL Delete Rows Per Hour` は、TTLタスクによって前の1時間に削除された行の合計数を記録しています。長期間にわたって `TTL Insert Rows Per Hour` が `TTL Delete Rows Per Hour` よりも大きい場合、挿入速度が削除速度よりも高いことを意味し、データの総量が増加することを意味します。例:

    ![insert fast example](/media/ttl/insert-fast.png)

    なお、TTLは期限切れの行がすぐに削除されることを保証しないため、現在挿入されている行が将来のTTLタスクで削除されるため、短期間の削除速度が挿入速度よりも低い場合でも、必ずしもTTLの速度が遅いとは限りません。状況を文脈で考慮する必要があります。

- TTLタスクのボトルネックがスキャンまたは削除にあるかどうかをどのように判断できますか？

    `TTL Scan Worker Time By Phase` と `TTL Delete Worker Time By Phase` パネルを見てください。スキャンワーカーが大部分の時間を `dispatch` フェーズで待機しており、削除ワーカーが滅多に `idle` フェーズでない場合、スキャンワーカーは削除ワーカーの削除が終了するのを待っています。この段階でクラスターリソースがまだ空いている場合、`tidb_ttl_ delete_worker_count` を増やして削除ワーカーの数を増やすことを検討することができます。例:

    ![scan fast example](/media/ttl/scan-fast.png)

    対照的に、スキャンワーカーが `dispatch` フェーズで滅多になく、削除ワーカーが長時間にわたり `idle` フェーズにある場合、スキャンワーカーは比較的忙しいことを示します。例:

    ![delete fast example](/media/ttl/delete-fast.png)

    TTLジョブでのスキャンと削除の割合は、マシン構成やデータ配布に関連するため、各瞬間のモニタリングデータは実行されているTTLジョブを代表するものです。特定の時点で実行されているTTLジョブとその対応するテーブルを判断するために、`mysql.tidb_ttl_job_history` テーブルを参照することができます。

- `tidb_ttl_scan_worker_count` と `tidb_ttl_delete_worker_count` を適切に設定する方法はありますか？

```yaml
      + How to configure `tidb_ttl_scan_worker_count` and `tidb_ttl_delete_worker_count` properly?
      + If the number of TiKV nodes is high, increase the value of `tidb_ttl_scan_worker_count` can make the TTL task workload more balanced.
      + But too many TTL workers will cause a lot of pressure, you need to evaluate the CPU level of TiDB and the disk and CPU usage of TiKV together. Depending on different scenarios and needs (whether you need to speed up TTL as much as possible, or to reduce the impact of TTL on other queries), you can adjust the value of `tidb_ttl_scan_worker_count` and `tidb_ttl_delete_worker_count` to improve the speed of TTL scanning and deleting or reduce the performance impact brought by TTL tasks.
```
```yaml
      + `tidb_ttl_scan_worker_count` および `tidb_ttl_delete_worker_count` を適切に構成する方法は？

      + TiKV ノードの数が多い場合、`tidb_ttl_scan_worker_count` の値を増やすと TTL タスクのワークロードがより均等になります。
      
      + ただし、TTL ワーカーが多すぎると多くの圧力を引き起こしますので、TiDB の CPU レベルおよび TiKV のディスクおよび CPU 使用状況を総合的に評価する必要があります。異なるシナリオとニーズに応じて（TTL を可能な限りスピードアップする必要があるか、または他のクエリに対する TTL の影響を軽減する必要があるか）、`tidb_ttl_scan_worker_count` および `tidb_ttl_delete_worker_count` の値を調整して、TTL のスキャンと削除のスピードを向上させるか、TTL タスクによるパフォーマンスへの影響を軽減することができます。
```