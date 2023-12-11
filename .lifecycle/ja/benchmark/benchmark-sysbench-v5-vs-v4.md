---
title: TiDB Sysbenchテストのパフォーマンスレポート -- v5.0 対 v4.0
---

# TiDB Sysbenchテストのパフォーマンスレポート -- v5.0 対 v4.0

## テスト目的

このテストは、TiDB v5.0とTiDB v4.0のSysbenchパフォーマンスをオンライントランザクション処理（OLTP）シナリオで比較することを目的としています。

## テスト環境（AWS EC2）

### ハードウェア構成

| サービス種別        | EC2種別     | インスタンス数 |
|:----------|:----------|:----------|
| PD        | m5.xlarge |     3     |
| TiKV      | i3.4xlarge|     3     |
| TiDB      | c5.4xlarge|     3     |
| Sysbench  | c5.9xlarge|     1     |

### ソフトウェアバージョン

| サービス種別   | ソフトウェアバージョン    |
|:----------|:-----------|
| PD        | 4.0 と 5.0   |
| TiDB      | 4.0 と 5.0   |
| TiKV      | 4.0 と 5.0   |
| Sysbench  | 1.0.20     |

### パラメータ構成

#### TiDB v4.0 構成

{{< コピー可能 "" >}}

```yaml
log.level: "error"
performance.max-procs: 20
prepared-plan-cache.enabled: true
tikv-client.max-batch-wait-time: 2000000
```

#### TiKV v4.0 構成

{{< コピー可能 "" >}}

```yaml
storage.scheduler-worker-pool-size: 5
raftstore.store-pool-size: 3
raftstore.apply-pool-size: 3
rocksdb.max-background-jobs: 3
raftdb.max-background-jobs: 3
raftdb.allow-concurrent-memtable-write: true
server.grpc-concurrency: 6
readpool.unified.min-thread-count: 5
readpool.unified.max-thread-count: 20
readpool.storage.normal-concurrency: 10
pessimistic-txn.pipelined: true
```

#### TiDB v5.0 構成

{{< コピー可能 "" >}}

```yaml
log.level: "error"
performance.max-procs: 20
prepared-plan-cache.enabled: true
tikv-client.max-batch-wait-time: 2000000
```

#### TiKV v5.0 構成

{{< コピー可能 "" >}}

```yaml
storage.scheduler-worker-pool-size: 5
raftstore.store-pool-size: 3
raftstore.apply-pool-size: 3
rocksdb.max-background-jobs: 8
raftdb.max-background-jobs: 4
raftdb.allow-concurrent-memtable-write: true
server.grpc-concurrency: 6
readpool.unified.min-thread-count: 5
readpool.unified.max-thread-count: 20
readpool.storage.normal-concurrency: 10
pessimistic-txn.pipelined: true
server.enable-request-batch: false
```

#### TiDB v4.0 グローバル変数構成

{{< コピー可能 "sql" >}}

```sql
set global tidb_hashagg_final_concurrency=1;
set global tidb_hashagg_partial_concurrency=1;
```

#### TiDB v5.0 グローバル変数構成

{{< コピー可能 "sql" >}}

```sql
set global tidb_hashagg_final_concurrency=1;
set global tidb_hashagg_partial_concurrency=1;
set global tidb_enable_async_commit = 1;
set global tidb_enable_1pc = 1;
set global tidb_guarantee_linearizability = 0;
set global tidb_enable_clustered_index = 1;
```

## テスト計画

1. TiUPを使用してTiDB v5.0およびv4.0を展開する。
2. Sysbenchを使用して16のテーブルをインポートし、各テーブルに1,000万行のデータを作成する。
3. 各テーブルに対して`analyze table`ステートメントを実行する。
4. 異なる並行性テストの前にリストア用のデータをバックアップすることで、各テストのデータ整合性を確保する。
5. Sysbenchクライアントを開始して、`point_select`、`read_write`、`update_index`、`update_non_index`のテストを実行する。AWS NLBを介してTiDB上でストレステストを実行する。各テストタイプでウォームアップに1分、テストに5分かかる。
6. 各種テストが完了した後、クラスターを停止し、ステップ4でバックアップしたデータでクラスターを上書きしてから、クラスターを再起動する。

### テストデータの準備

次のコマンドを実行してテストデータを準備します。

{{< コピー可能 "shell-regular" >}}

```bash
sysbench oltp_common \
    --threads=16 \
    --rand-type=uniform \
    --db-driver=mysql \
    --mysql-db=sbtest \
    --mysql-host=$aws_nlb_host \
    --mysql-port=$aws_nlb_port \
    --mysql-user=root \
    --mysql-password=password \
    prepare --tables=16 --table-size=10000000
```

### テストの実行

次のコマンドを実行してテストを実行します。

{{< コピー可能 "shell-regular" >}}

```bash
sysbench $testname \
    --threads=$threads \
    --time=300 \
    --report-interval=1 \
    --rand-type=uniform \
    --db-driver=mysql \
    --mysql-db=sbtest \
    --mysql-host=$aws_nlb_host \
    --mysql-port=$aws_nlb_port \
    run --tables=16 --table-size=10000000
```

## テスト結果

### ポイントセレクトのパフォーマンス

| Threads   | v4.0 QPS   | v4.0 95% レイテンシ（ms）   | v5.0 QPS   | v5.0 95% レイテンシ（ms）   | QPS改善率   |
|:----------|:----------|:----------|:----------|:----------|:----------|
| 150        | 159451.19        | 1.32        | 177876.25        | 1.23        | 11.56%     |
| 300        | 244790.38        | 1.96        | 252675.03        | 1.82        | 3.22%     |
| 600        | 322929.05        | 3.75        | 331956.84        | 3.36        | 2.80%     |
| 900        | 364840.05        | 5.67        | 365655.04        | 5.09        | 0.22%     |
| 1200        | 376529.18        | 7.98        | 366507.47        | 7.04        | -2.66%     |
| 1500        | 368390.52        | 10.84        | 372476.35        | 8.90        | 1.11%     |

TiDB v5.0のポイントセレクトのパフォーマンスは、v4.0と比較して2.7%向上しています。

![ポイントセレクト](/media/sysbench_v5vsv4_point_select.png)

### インデックス無し更新のパフォーマンス

| Threads   | v4.0 QPS   | v4.0 95% レイテンシ（ms）   | v5.0 QPS   | v5.0 95% レイテンシ（ms）   | QPS改善率   |
|:----------|:----------|:----------|:----------|:----------|:----------|
| 150        | 17243.78        | 11.04        | 30866.23        | 6.91        | 79.00%     |
| 300        | 25397.06        | 15.83        | 45915.39        | 9.73        | 80.79%     |
| 600        | 33388.08        | 25.28        | 60098.52        | 16.41        | 80.00%     |
| 900        | 38291.75        | 36.89        | 70317.41        | 21.89        | 83.64%     |
| 1200        | 41003.46        | 55.82        | 76376.22        | 28.67        | 86.27%     |
| 1500        | 44702.84        | 62.19        | 80234.58        | 34.95        | 79.48%     |

TiDB v5.0のインデックス無し更新のパフォーマンスは、v4.0と比較して81%向上しています。

![インデックス無し更新](/media/sysbench_v5vsv4_update_non_index.png)

### インデックス更新のパフォーマンス

| Threads   | v4.0 QPS   | v4.0 95% レイテンシ（ms）   | v5.0 QPS   | v5.0 95% レイテンシ（ms）   | QPS改善率   |
|:----------|:----------|:----------|:----------|:----------|:----------|
| 150        | 11736.21        | 17.01        | 15631.34        | 17.01        | 33.19%     |
| 300        | 15435.95        | 28.67        | 19957.06        | 22.69        | 29.29%     |
| 600        | 18983.21        | 49.21        | 23218.14        | 41.85        | 22.31%     |
| 900        | 20855.29        | 74.46        | 26226.76        | 53.85        | 25.76%     |
| 1200        | 21887.64        | 102.97        | 28505.41        | 69.29        | 30.24%     |
| 1500        | 23621.15        | 110.66        | 30341.06        | 82.96        | 28.45%     |

TiDB v5.0のUpdate Indexパフォーマンスは、v4.0と比較して28%向上しました。

![Update Index](/media/sysbench_v5vsv4_update_index.png)

### Read Write performance

| Threads   | v4.0 QPS   | v4.0 95% latency (ms)   | v5.0 QPS   | v5.0 95% latency (ms)   | QPS improvement   |
|:----------|:----------|:----------|:----------|:----------|:----------|
| 150        | 59979.91        | 61.08        | 66098.57        | 55.82        | 10.20%     |
| 300        | 77118.32        | 102.97        | 84639.48        | 90.78        | 9.75%     |
| 600        | 90619.52        | 183.21        | 101477.46        | 167.44        | 11.98%     |
| 900        | 97085.57        | 267.41        | 109463.46        | 240.02        | 12.75%     |
| 1200        | 106521.61        | 331.91        | 115416.05        | 320.17        | 8.35%     |
| 1500        | 116278.96        | 363.18        | 118807.5        | 411.96        | 2.17%     |

TiDB v5.0のread-writeパフォーマンスは、v4.0と比較して9%向上しました。

![Read Write](/media/sysbench_v5vsv4_read_write.png)