---
title: TiDB Cloud HTAP クイックスタート
summary: TiDB Cloud で HTAP を開始する方法を学びます。
aliases: ['/tidbcloud/use-htap-cluster']
---

# TiDB Cloud HTAP クイックスタート

[HTAP](https://en.wikipedia.org/wiki/Hybrid_transactional/analytical_processing) は、Hybrid Transactional and Analytical Processingの略です。TiDB CloudのHTAPクラスターは、[TiKV](https://tikv.org) という、トランザクション処理用に設計された行ベースのストレージエンジンと、[TiFlash](https://docs.pingcap.com/tidb/stable/tiflash-overview) という、解析処理用に設計された列ベースのストレージから構成されています。アプリケーションデータは、まずTiKVに保存され、その後Raftコンセンサスアルゴリズムを使用してTiFlashにレプリケートされます。したがって、これは行ベースのストレージから列ベースのストレージへのリアルタイムレプリケーションです。

このチュートリアルでは、TiDB CloudのHybrid Transactional and Analytical Processing（HTAP）機能を体験する簡単な方法をご案内します。内容は、どのようにしてテーブルをTiFlashにレプリケートし、TiFlashでクエリを実行し、パフォーマンス向上を体験するかを含んでいます。

## 開始する前に

HTAP機能を体験する前に、[TiDB Cloudクイックスタート](/tidb-cloud/tidb-cloud-quickstart.md)に従って、TiFlashノードを持つクラスターを作成し、TiDBクラスターに接続し、Capital Bikeshareのサンプルデータをクラスターにインポートしてください。

## 手順

### ステップ1. テーブルのサンプルデータを列ベースのストレージにレプリケートする

TiFlashノードを持つクラスターが作成された後、TiKVはデフォルトではデータをTiFlashにレプリケートしません。TiDBのMySQLクライアントでDDLステートメントを実行して、レプリケートするテーブルを指定する必要があります。その後、TiDBがTiFlashに指定されたテーブルのレプリカを作成します。

たとえば、`trips`テーブル（Capital Bikeshareのサンプルデータ）をTiFlashにレプリケートするには、次のステートメントを実行します:

```sql
USE bikeshare;
```

```sql
ALTER TABLE trips SET TIFLASH REPLICA 1;
```

レプリケーションの進行状況を確認するには、次のステートメントを実行します：

```sql
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = 'bikeshare' and TABLE_NAME = 'trips';
```

```sql
+--------------+------------+----------+---------------+-----------------+-----------+----------+------------+
| TABLE_SCHEMA | TABLE_NAME | TABLE_ID | REPLICA_COUNT | LOCATION_LABELS | AVAILABLE | PROGRESS | TABLE_MODE |
+--------------+------------+----------+---------------+-----------------+-----------+----------+------------+
| bikeshare    | trips      |       88 |             1 |                 |         1 |        1 | NORMAL     |
+--------------+------------+----------+---------------+-----------------+-----------+----------+------------+
1 row in set (0.20 sec)
```

前述のステートメントの結果では、以下のことが示されています：

- `AVAILABLE`によって、特定のテーブルのTiFlashレプリカが利用可能かどうかが示されます。`1`は利用可能を、`0`は利用不可を意味します。レプリカが利用可能になると、この状態はもはや変わりません。
- `PROGRESS`によって、レプリケーションの進捗が示されます。値は`0`から`1`の間です。`1`は少なくとも1つのレプリカがレプリケートされていることを意味します。

### ステップ2. HTAPを使用したデータのクエリ

レプリケーションのプロセスが完了したら、いくつかのクエリを実行できます。

たとえば、異なる出発駅と到着駅ごとのトリップ数を確認できます：

```sql
SELECT start_station_name, end_station_name, COUNT(ride_id) as count from `trips`
GROUP BY start_station_name, end_station_name
ORDER BY count ASC;
```

### ステップ3. 行ベースのストレージと列ベースのストレージのクエリパフォーマンスを比較する

このステップでは、TiKV（行ベースのストレージ）とTiFlash（列ベースのストレージ）を使用したクエリの実行統計を比較できます。

- このクエリをTiKVを使用して実行した際の実行統計を取得するには、次のステートメントを実行します：

    ```sql
    EXPLAIN ANALYZE SELECT /*+ READ_FROM_STORAGE(TIKV[trips]) */ start_station_name, end_station_name, COUNT(ride_id) as count from `trips`
    GROUP BY start_station_name, end_station_name
    ORDER BY count ASC;
    ```

    TiFlashレプリカを持つテーブルの場合、TiDBオプティマイザーはコスト推定に基づいてTiKVまたはTiFlashのレプリカのいずれを使用するかを自動的に決定します。前述の`EXPLAIN ANALYZE`ステートメントでは、`HINT /*+ READ_FROM_STORAGE(TIKV[trips]) */`が使用されており、オプティマイザーにTiKVを選択するよう強制するため、TiKVの実行統計を確認できます。

    > **注記:**
    >
    > MySQLコマンドラインクライアントのバージョンが5.7.7より前の場合、デフォルトでオプティマイザーヒントを削除します。これらの古いバージョンを使用している場合は、クライアントを起動する際に `--comments`オプションを追加してください。たとえば：`mysql -h 127.0.0.1 -P 4000 -uroot --comments`。

    出力から、`execution info`列から実行時間を取得できます。

    ```sql
    id                         | estRows   | actRows | task      | access object | execution info                            | operator info                                | memory  | disk
    ---------------------------+-----------+---------+-----------+---------------+-------------------------------------------+-----------------------------------------------+---------+---------
    Sort_5                     | 633.00    | 73633   | root      |               | time:1.62s, loops:73                      | Column#15                                    | 6.88 MB | 0 Bytes
    └─Projection_7             | 633.00    | 73633   | root      |               | time:1.57s, loops:76, Concurrency:OFF...  | bikeshare.trips.start_station_name...        | 6.20 MB | N/A                                                                                                                                        | 6.20 MB | N/A
      └─HashAgg_15             | 633.00    | 73633   | root      |               | time:1.57s, loops:76, partial_worker:...  | group by:bikeshare.trips.end_station_name... | 58.0 MB | N/A
        └─TableReader_16       | 633.00    | 111679  | root      |               | time:1.34s, loops:3, cop_task: {num: ...  | data:HashAgg_8                               | 7.55 MB | N/A
          └─HashAgg_8          | 633.00    | 111679  | cop[tikv] |               | tikv_task:{proc max:830ms, min:470ms,...  | group by:bikeshare.trips.end_station_name... | N/A     | N/A
            └─TableFullScan_14 | 816090.00 | 816090  | cop[tikv] | table:trips   | tikv_task:{proc max:490ms, min:310ms,...  | keep order:false                             | N/A     | N/A
    (6 rows)
    ```

- このクエリをTiFlashを使用して実行した際の実行統計を取得するには、次のステートメントを実行します：

    ```sql
    EXPLAIN ANALYZE SELECT start_station_name, end_station_name, COUNT(ride_id) as count from `trips`
    GROUP BY start_station_name, end_station_name
    ORDER BY count ASC;
    ```

    出力から、`execution info`列から実行時間を取得できます。

    ```sql
    id                                 | estRows   | actRows | task         | access object | execution info                            | operator info                      | memory  | disk
    -----------------------------------+-----------+---------+--------------+---------------+-------------------------------------------+------------------------------------+---------+---------
    Sort_5                             | 633.00    | 73633   | root         |               | time:420.2ms, loops:73                    | Column#15                          | 5.61 MB | 0 Bytes
    └─Projection_7                     | 633.00    | 73633   | root         |               | time:368.7ms, loops:73, Concurrency:OFF   | bikeshare.trips.start_station_...  | 4.94 MB | N/A
      └─TableReader_34                 | 633.00    | 73633   | root         |               | time:368.6ms, loops:73, cop_task: {num... | data:ExchangeSender_33             | N/A     | N/A
        └─ExchangeSender_33            | 633.00    | 73633   | mpp[tiflash] |               | tiflash_task:{time:360.7ms, loops:1,...   | ExchangeType: PassThrough          | N/A     | N/A
          └─Projection_29              | 633.00    | 73633   | mpp[tiflash] |               | tiflash_task:{time:330.7ms, loops:1,...   | Column#15, bikeshare.trips.star... | N/A     | N/A
```
      └─HashAgg_30               | 633.00    | 73633   | mpp[tiflash] |               | tiflash_task:{time:330.7ms, loops:1,...   | group by:bikeshare.trips.end_st... | N/A     | N/A
        └─ExchangeReceiver_32    | 633.00    | 73633   | mpp[tiflash] |               | tiflash_task:{time:280.7ms, loops:12,...  |                                    | N/A     | N/A
          └─ExchangeSender_31    | 633.00    | 73633   | mpp[tiflash] |               | tiflash_task:{time:272.3ms, loops:256,... | ExchangeType: HashPartition, Ha... | N/A     | N/A
            └─HashAgg_12         | 633.00    | 73633   | mpp[tiflash] |               | tiflash_task:{time:252.3ms, loops:256,... | group by:bikeshare.trips.end_st... | N/A     | N/A
              └─TableFullScan_28 | 816090.00 | 816090  | mpp[tiflash] | table:trips   | tiflash_task:{time:92.3ms, loops:16,...   | keep order:false                   | N/A     | N/A
    (10 rows)
    ```

> **Note:**
>
> サンプルデータのサイズが小さく、このドキュメントのクエリが非常に単純であるため、すでに最適化プログラムに対してこのクエリをTiKVを選択するように強制した場合、同じクエリを再度実行すると、TiKVはそのキャッシュを再利用するため、クエリがはるかに高速になることがあります。 データが頻繁に更新される場合、キャッシュはミスを起こす可能性があります。

## 詳細情報

- [TiFlash の概要](/tiflash/tiflash-overview.md)
- [TiFlash レプリカの作成](/tiflash/create-tiflash-replicas.md)
- [TiFlash からデータを読む](/tiflash/use-tidb-to-read-tiflash.md)
- [MPP（大規模並列処理）モードの使用](/tiflash/use-tiflash-mpp-mode.md)
- [サポートされるプッシュダウン計算](/tiflash/tiflash-supported-pushdown-calculations.md)
```