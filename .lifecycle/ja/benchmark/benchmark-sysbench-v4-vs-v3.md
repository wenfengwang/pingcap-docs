---
title: TiDB Sysbench パフォーマンステストレポート -- v4.0 vs. v3.0
summary: TiDB 4.0とTiDB 3.0のSysbenchパフォーマンスを比較します。
aliases: ['/docs/dev/benchmark/benchmark-sysbench-v4-vs-v3/']
---

# TiDB Sysbench パフォーマンステストレポート -- v4.0 vs. v3.0

## テスト目的

このテストは、TiDB 4.0とTiDB 3.0のOnline Transactional Processing (OLTP)シナリオにおけるSysbenchパフォーマンスを比較することを目的としています。

## テスト環境（AWS EC2）

### ハードウェア構成

| サービスタイプ         | EC2タイプ     | インスタンス数 |
|:----------|:----------|:----------|
| PD        | m5.xlarge |     3     |
| TiKV      | i3.4xlarge|     3     |
| TiDB      | c5.4xlarge|     3     |
| Sysbench  | m5.4xlarge|     1     |

### ソフトウェアバージョン

| サービスタイプ   | ソフトウェアバージョン    |
|:----------|:-----------|
| PD        | 3.0 and 4.0   |
| TiDB      | 3.0 and 4.0   |
| TiKV      | 3.0 and 4.0   |
| Sysbench  | 1.0.20     |

### パラメータ構成

#### TiDB v3.0 構成

{{< copyable "" >}}

```yaml
log.level: "error"
performance.max-procs: 20
prepared-plan-cache.enabled: true
tikv-client.max-batch-wait-time: 2000000
```

#### TiKV v3.0 構成

{{< copyable "" >}}

```yaml
storage.scheduler-worker-pool-size: 5
raftstore.store-pool-size: 3
raftstore.apply-pool-size: 3
rocksdb.max-background-jobs: 3
raftdb.max-background-jobs: 3
raftdb.allow-concurrent-memtable-write: true
server.grpc-concurrency: 6
readpool.storage.normal-concurrency: 10
readpool.coprocessor.normal-concurrency: 5
```

#### TiDB v4.0 構成

{{< copyable "" >}}

```yaml
log.level: "error"
performance.max-procs: 20
prepared-plan-cache.enabled: true
tikv-client.max-batch-wait-time: 2000000
```

#### TiKV v4.0 構成

{{< copyable "" >}}

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

#### グローバル変数構成

{{< copyable "sql" >}}

```sql
set global tidb_hashagg_final_concurrency=1;
set global tidb_hashagg_partial_concurrency=1;
set global tidb_disable_txn_auto_retry=0;
```

## テスト計画

1. TiUPを使用してTiDB v4.0およびv3.0をデプロイします。
2. Sysbenchを使用してデータ行数が1,000万行のテーブルを16個インポートします。
3. 各テーブルに対して`analyze table`ステートメントを実行します。
4. 異なる並列度のテストの前に、リストア用のデータをバックアップして、各テストでデータの一貫性を確保します。
5. Sysbenchクライアントを開始し、`point_select`、`read_write`、`update_index`、および`update_non_index`のテストを実行します。AWS NLBを介してTiDBにストレステストを実行します。各テストタイプでは、ウォームアップに1分、テストに5分がかかります。
6. 各テストタイプが完了したら、クラスターを停止し、ステップ4でバックアップしたデータでクラスターを上書きし、クラスターを再起動します。

### テストデータの準備

次のコマンドを実行してテストデータを準備します。

{{< copyable "shell-regular" >}}

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

{{< copyable "shell-regular" >}}

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

| スレッド数   | v3.0 QPS   | v3.0 95% レイテンシー（ms）   | v4.0 QPS   | v4.0 95% レイテンシー（ms）   | QPS改善率 |
|:----------|:----------|:----------|:----------|:----------|:----------|
| 150        | 117085.701 |     1.667     | 118165.1357        | 1.608 |     0.92%     |
| 300      | 200621.4471|     2.615     | 207774.0859        | 2.032 |     3.57%     |
| 600      | 283928.9323|     4.569     | 320673.342        | 3.304 |     12.94%     |
| 900  | 343218.2624|     6.686     | 383913.3855        | 4.652 |     11.86%     |
| 1200  | 347200.2366|     8.092     | 408929.4372        | 6.318 |     17.78%     |
| 1500  | 366406.2767|     10.562     | 418268.8856        | 7.985 |     14.15%     |

TiDB v4.0は、v3.0と比較して、ポイントセレクトのパフォーマンスが14%向上しました。

![Point Select](/media/sysbench-v4vsv3-point-select.png)

### インデックスの更新パフォーマンス

| Threads   | v3.0 QPS   | v3.0 95% レイテンシー（ms）   | v4.0 QPS   | v4.0 95% レイテンシー（ms）   | QPS改善率 |
|:----------|:----------|:----------|:----------|:----------|:----------|
| 150        | 15446.41024 |     11.446     | 16954.39971        | 10.844 |     9.76%     |
| 300      | 22276.15572|     17.319     | 24364.44689        | 16.706 |     9.37%     |
| 600      | 28784.88353|     29.194     | 31635.70833        | 28.162 |     9.90%    |
| 900  | 32194.15548|     42.611     | 35787.66078        | 38.942 |     11.16%     |
| 1200  | 33954.69114|     58.923     | 38552.63158        | 51.018 |     13.54%     |
| 1500  | 35412.0032|     74.464     | 40859.63755        | 62.193 |     15.38%     |

TiDB v4.0は、v3.0と比較して、更新インデックスのパフォーマンスが15%向上しました。

![Update Non-index](/media/sysbench-v4vsv3-update-non-index.png)

### インデックスの更新パフォーマンス

| Threads   | v3.0 QPS   | v3.0 95% レイテンシー（ms）   | v4.0 QPS   | v4.0 95% レイテンシー（ms）   | QPS改善率 |
|:----------|:----------|:----------|:----------|:----------|:----------|
| 150        | 11164.40571 |     16.706     | 11954.73635        | 16.408 |     7.08%     |
| 300      | 14460.98057|     28.162     | 15243.40899        | 28.162 |     5.41%     |
| 600      | 17112.73036|     53.85     | 18535.07515        | 50.107 |     8.31%    |
| 900  | 18233.83426|     86.002     | 20339.6901        | 70.548 |     11.55%     |
| 1200  | 18622.50283|     127.805     | 21390.25122        | 94.104 |     14.86%     |
| 1500  | 18980.34447|     170.479     | 22359.996        | 114.717 |     17.81%     |

v3.0と比較して、TiDB v4.0のUpdate Indexの性能が17%向上しました。

![Update Index](/media/sysbench-v4vsv3-update-index.png)

### 読み書き性能

| スレッド数   | v3.0 QPS   | v3.0 95% レイテンシ（ms）   | v4.0 QPS   | v4.0 95% レイテンシ（ms）   | QPS改善率 |
|:----------|:----------|:----------|:----------|:----------|:----------|
| 150        | 43768.33633 |     71.83     | 53912.63705        | 59.993 |     23.18%     |
| 300      | 55655.63589|     121.085     | 71327.21336        | 97.555 |     28.16%     |
| 600      | 64642.96992|     223.344     | 84487.75483        | 176.731 |     30.70%    |
| 900  | 68947.25293|     325.984     | 90177.94612        | 257.95 |     30.79%     |
| 1200  | 71334.80099|     434.829     | 92779.71507        | 344.078 |     30.06%     |
| 1500  | 72069.9115|     580.017     | 95088.50812        | 434.829 |     31.94%     |

v3.0と比較して、TiDB v4.0の読み書き性能が31%向上しました。

![Read Write](/media/sysbench-v4vsv3-read-write.png)