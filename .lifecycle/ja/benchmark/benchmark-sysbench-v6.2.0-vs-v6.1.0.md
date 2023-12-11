---
title: TiDB Sysbench パフォーマンステストレポート -- v6.2.0 vs. v6.1.0
---

# TiDB Sysbench パフォーマンステストレポート -- v6.2.0 vs. v6.1.0

## テストの概要

このテストは、オンライントランザクション処理（OLTP）シナリオにおけるTiDB v6.2.0とTiDB v6.1.0のSysbenchパフォーマンスを比較することを目的としています。結果から、v6.2.0のパフォーマンスは基本的にv6.1.0と同じであり、Point Selectのパフォーマンスはわずか3.58%低下しています。

## テスト環境（AWS EC2）

### ハードウェア構成

| サービスタイプ | EC2タイプ | インスタンス数 |
|:----------|:----------|:----------|
| PD        | m5.xlarge |     3     |
| TiKV      | i3.4xlarge|     3     |
| TiDB      | c5.4xlarge|     3     |
| Sysbench  | c5.9xlarge|     1     |

### ソフトウェアバージョン

| サービスタイプ | ソフトウェアバージョン |
|:----------|:-----------|
| PD        | v6.1.0 および v6.2.0 |
| TiDB      | v6.1.0 および v6.2.0 |
| TiKV      | v6.1.0 および v6.2.0 |
| Sysbench  | 1.1.0-df89d34   |

### パラメータ設定

TiDB v6.2.0とTiDB v6.1.0は同じ設定を使用しています。

#### TiDBパラメータ設定

{{＜コピー可能 ""＞}}

```yaml
log.level: "error"
prepared-plan-cache.enabled: true
tikv-client.max-batch-wait-time: 2000000
```

#### TiKVパラメータ設定

{{＜コピー可能 ""＞}}

```yaml
storage.scheduler-worker-pool-size: 5
raftstore.store-pool-size: 3
raftstore.apply-pool-size: 3
rocksdb.max-background-jobs: 8
server.grpc-concurrency: 6
readpool.unified.max-thread-count: 10
```

#### TiDBグローバル変数設定

{{＜コピー可能 "sql"＞}}

```sql
set global tidb_hashagg_final_concurrency=1;
set global tidb_hashagg_partial_concurrency=1;
set global tidb_enable_async_commit = 1;
set global tidb_enable_1pc = 1;
set global tidb_guarantee_linearizability = 0;
set global tidb_enable_clustered_index = 1;
set global tidb_prepared_plan_cache_size=1000;
```

#### HAProxy設定 - haproxy.cfg

TiDBでHAProxyを使用する方法の詳細については、[TiDBでHAProxyのベストプラクティス](/best-practices/haproxy-best-practices.md)を参照してください。

{{＜コピー可能 ""＞}}

```yaml
global                                     # グローバル構成。
   pidfile     /var/run/haproxy.pid        # HAProxyプロセスのPIDをこのファイルに書き込みます。
   maxconn     4000                        # 単一のHAProxyプロセスに対する最大同時接続数。
   user        haproxy                     # UIDパラメータと同じ。
   group       haproxy                     # GIDパラメータと同じ。専用のユーザーグループが推奨されます。
   nbproc      64                          # デーモンになる際に作成されるプロセスの数。複数のプロセスを開始してリクエストを転送する場合は、HAProxyがプロセスをブロックしないように十分大きな値にする必要があります。
   daemon                                  # プロセスをバックグラウンドでフォークします。これはコマンドラインの"-D"引数と同等です。コマンドラインの"-db"引数によって無効にすることができます。

defaults                                   # デフォルトの構成。
   log global                              # グローバル構成の設定を継承します。
   retries 2                               # アップストリームサーバへの接続をリトライする最大回数。接続試行回数がこの値を超えると、バックエンドサーバは利用不可と見なされます。
   timeout connect  2s                     # バックエンドサーバへの接続試行が成功するまでの最大時間。サーバがHAProxyと同じLAN上にある場合は、より短い時間に設定する必要があります。
   timeout client 30000s                   # クライアント側の非アクティブ時間の最大値。
   timeout server 30000s                   # サーバ側の非アクティブ時間の最大値。

listen tidb-cluster                        # データベース負荷分散。
   bind 0.0.0.0:3390                       # 浮動IPアドレスとリッスンポート。
   mode tcp                                # HAProxyはレイヤー4、トランスポートレイヤーを使用します。
   balance leastconn                      # 最も少ない接続を持つサーバが接続を受け取ります。LDAP、SQL、TSEなど、長時間のセッションが予想される場合には「leastconn」が推奨されます。重みづけのアルゴリズムは動的であり、たとえば遅い開始のためにサーバの重みがリクエストごとに調整される可能性があります。
   server tidb-1 10.9.18.229:4000 check inter 2000 rise 2 fall 3       # 2000ミリ秒ごとにポート4000を検出します。2回成功したらサーバは利用可能と見なされます。3回失敗したらサーバは利用不可と見なされます。
   server tidb-2 10.9.39.208:4000 check inter 2000 rise 2 fall 3
   server tidb-3 10.9.64.166:4000 check inter 2000 rise 2 fall 3
```

## テスト計画

1. TiUPを使用してTiDB v6.2.0およびv6.1.0をデプロイします。
2. Sysbenchを使用して、それぞれ1,000万行のデータを持つ16のテーブルをインポートします。
3. 各テーブルに対して`analyze table`ステートメントを実行します。
4. 異なる並行性テストの前にリストア用に使用されるデータのバックアップを取得します。これにより、各テストでデータの整合性が確保されます。
5. Sysbenchクライアントを起動して`point_select`、`read_write`、`update_index`、および`update_non_index`のテストを実行します。HAProxyを介してTiDB上でストレステストを実行します。各ワークロードの各並行性について、テストには20分かかります。
6. 各タイプのテストが完了した後、クラスタを停止し、ステップ4でのバックアップデータでクラスタを上書きし、クラスタを再起動します。

### テストデータの準備

次のコマンドを実行してテストデータの準備を行います：

{{＜コピー可能 "shell-regular"＞}}

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

次のコマンドを実行してテストを実行します：

{{＜コピー可能 "shell-regular"＞}}

```bash
sysbench $testname \
    --threads=$threads \
    --time=1200 \
    --report-interval=1 \
    --rand-type=uniform \
    --db-driver=mysql \
    --mysql-db=sbtest \
    --mysql-host=$aws_nlb_host \
    --mysql-port=$aws_nlb_port \
    run --tables=16 --table-size=10000000
```

## テスト結果

### Point Select パフォーマンス

| スレッド数 | v6.1.0 TPS | v6.2.0 TPS | v6.1.0 95%レイテンシ（ms） | v6.2.0 95%レイテンシ（ms） | TPS改善率（%） |
| :------ | :--------- | :--------- | :---------------------- | :---------------------- | :----------- |
| 300     | 243530.01  | 236885.24  | 1.93                    | 2.07                    | -2.73        |
| 600     | 304121.47  | 291395.84  | 3.68                    | 4.03                    | -4.18        |
| 900     | 327301.23  | 314720.02  | 5                       | 5.47                    | -3.84        |

v6.2.0のPoint Selectパフォーマンスは、v6.1.0と比較してわずか3.58%低下しています。

![Point Select](/media/sysbench_v610vsv620_point_select.png)

### Update Non-index パフォーマンス

| スレッド数 | v6.1.0 TPS | v6.2.0 TPS | v6.1.0 95%レイテンシ（ms） | v6.2.0 95%レイテンシ（ms） | TPS改善率（%）  |
| :------ | :--------- | :--------- | :---------------------- | :---------------------- | :----------- |
| 300     | 42608.8    | 42372.82   | 11.45                   | 11.24                   | -0.55        |
| 600     | 54264.47   | 53672.69   | 18.95                   | 18.95                   | -1.09        |
| 900     | 60667.47   | 60116.14   | 26.2                    | 26.68                   | -0.91        |

v6.2.0のUpdate Non-indexパフォーマンスは、v6.1.0と比較して基本的に変わらず、0.85%減少しました。

![Update Non-index](/media/sysbench_v610vsv620_update_non_index.png)

### Update Indexパフォーマンス

| Threads | v6.1.0 TPS | v6.2.0 TPS | v6.1.0 95%レイテンシ（ms） | v6.2.0 95%レイテンシ（ms） | TPS向上率（%） |
| :------ | :--------- | :--------- | :---------------------- | :---------------------- | :----------- |
| 300     | 19384.75   | 19353.58   | 23.52                   | 23.52                   | -0.16        |
| 600     | 24144.78   | 24007.57   | 38.25                   | 37.56                   | -0.57        |
| 900     | 26770.9    | 26589.84   | 51.94                   | 52.89                   | -0.68        |

v6.2.0のUpdate Indexパフォーマンスは、v6.1.0と比較して基本的に変わらず、0.47%減少しました。

![Update Index](/media/sysbench_v610vsv620_update_index.png)

### Read Writeパフォーマンス

| Threads | v6.1.0 TPS | v6.2.0 TPS | v6.1.0 95%レイテンシ（ms） | v6.2.0 95%レイテンシ（ms） | TPS向上率（%） |
| :------ | :--------- | :--------- | :---------------------- | :---------------------- | :----------- |
| 300     | 4849.67    | 4797.59    | 86                      | 84.47                   | -1.07        |
| 600     | 5643.89    | 5565.17    | 161.51                  | 161.51                  | -1.39        |
| 900     | 5954.91    | 5885.22    | 235.74                  | 235.74                  | -1.17        |

v6.2.0のRead Writeパフォーマンスは、v6.1.0と比較して1.21%減少しました。

![Read Write](/media/sysbench_v610vsv620_read_write.png)