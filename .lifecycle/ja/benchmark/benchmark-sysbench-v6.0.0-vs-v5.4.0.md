---
title: TiDB Sysbench パフォーマンステストレポート -- v6.0.0 vs. v5.4.0
---

# TiDB Sysbench パフォーマンステストレポート -- v6.0.0 vs. v5.4.0

## テストの概要

このテストは、オンライントランザクション処理（OLTP）シナリオにおけるTiDB v6.0.0とTiDB v5.4.0のSysbenchパフォーマンスを比較することを目的としています。結果によれば、読み込み書き込みワークロードにおいてv6.0.0のパフォーマンスが16.17%向上しています。他のワークロードのパフォーマンスは基本的にv5.4.0と同じです。

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
| PD        | v5.4.0 と v6.0.0 |
| TiDB      | v5.4.0 と v6.0.0 |
| TiKV      | v5.4.0 と v6.0.0 |
| Sysbench  | 1.1.0-df89d34   |

### パラメータ構成

TiDB v6.0.0 と TiDB v5.4.0 は同じ構成を使用しています。

#### TiDB パラメータ構成

{{< copyable "" >}}

```yaml
log.level: "error"
prepared-plan-cache.enabled: true
tikv-client.max-batch-wait-time: 2000000
```

#### TiKV パラメータ構成

{{< copyable "" >}}

```yaml
storage.scheduler-worker-pool-size: 5
raftstore.store-pool-size: 3
raftstore.apply-pool-size: 3
rocksdb.max-background-jobs: 8
raftdb.max-background-jobs: 4
raftdb.allow-concurrent-memtable-write: true
server.grpc-concurrency: 6
readpool.storage.normal-concurrency: 10
pessimistic-txn.pipelined: true
```

#### TiDB グローバル変数構成

{{< copyable "sql" >}}

```sql
set global tidb_hashagg_final_concurrency=1;
set global tidb_hashagg_partial_concurrency=1;
set global tidb_enable_async_commit = 1;
set global tidb_enable_1pc = 1;
set global tidb_guarantee_linearizability = 0;
set global tidb_enable_clustered_index = 1;
```

#### HAProxy 構成 - haproxy.cfg

TiDBでHAProxyを使用する方法の詳細については、[TiDBにおけるHAProxyのベストプラクティス](/best-practices/haproxy-best-practices.md)を参照してください。

{{< copyable "" >}}

```yaml
global                                     # グローバル構成。
   pidfile     /var/run/haproxy.pid        # HAProxyプロセスのPIDをこのファイルに書き込みます。
   maxconn     4000                        # 単一のHAProxyプロセスに対する並行接続の最大数。
   user        haproxy                     # UIDパラメータと同じです。
   group       haproxy                     # GIDパラメータと同じです。専用のユーザーグループを推奨します。
   nbproc      64                          # デーモンモードで実行するときに作成されるプロセス数。リクエストを転送するために複数のプロセスを起動する場合、HAProxyがプロセスをブロックしないように十分な値が設定されていることを確認してください。
   daemon                                  # プロセスをバックグラウンドでフォークさせます。これはコマンドラインの"-D"引数と同等です。コマンドラインの"-db"引数で無効にできます。
defaults                                   # デフォルト構成。
   log global                              # グローバル構成の設定を継承します。
   retries 2                               # アップストリームサーバーへの接続の最大リトライ回数。接続試行回数がこの値を超えると、バックエンドサーバーは利用できないと見なされます。
   timeout connect  2s                     # バックエンドサーバーへの接続試行が成功するまでの最大時間。サーバーがHAProxyと同じLANにある場合は、より短い時間に設定すべきです。
   timeout client 30000s                   # クライアント側の最大非アクティビティ時間。
   timeout server 30000s                   # サーバー側の最大非アクティビティ時間。
listen tidb-cluster                        # データベースの負荷分散。
   bind 0.0.0.0:3390                       # フローティングIPアドレスとリッスンポート。
   mode tcp                                # HAProxyはレイヤー4、トランスポートレイヤーを使用します。
   balance leastconn                      # 最も少ない接続を持つサーバーが接続を受けます。長時間のセッションが期待されるLDAP、SQL、TSEなどのプロトコルで推奨されます。HTTPなどの短いセッションを使用するプロトコルではありません。アルゴリズムは動的です。このため、サーバーの重みは、スロースタートなどに対応するために動的に調整される可能性があります。
   server tidb-1 10.9.18.229:4000 check inter 2000 rise 2 fall 3       # 2000ミリ秒ごとにポート4000を検出します。2回成功したら、サーバーは利用可能と見なされます。3回失敗したら、サーバーは利用できないと見なされます。
   server tidb-2 10.9.39.208:4000 check inter 2000 rise 2 fall 3
   server tidb-3 10.9.64.166:4000 check inter 2000 rise 2 fall 3
```

## テスト計画

1. TiUPを使用して、TiDB v6.0.0およびv5.4.0をデプロイします。
2. Sysbenchを使用して、それぞれ1,000万行のデータを持つ16のテーブルをインポートします。
3. 各テーブルに対して`analyze table`ステートメントを実行します。
4. 異なる並行性テストの前にリストアに使用するデータをバックアップし、各テストにおいてデータの整合性を確保します。
5. Sysbenchクライアントを起動して、`point_select`、`read_write`、`update_index`、`update_non_index`のテストを実行します。HAProxyを介してTiDBにストレステストを実行します。各ワークロードの各並行性について、テストに20分かかります。
6. 各種類のテストが完了したら、クラスターを停止し、ステップ4で作成したバックアップデータでクラスターを上書きしてから、クラスターを再起動します。

### テストデータの準備

次のコマンドを実行してテストデータを準備します:

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

次のコマンドを実行してテストを実行します:

{{< copyable "shell-regular" >}}

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

### ポイントセレクトのパフォーマンス

| スレッド数 | v5.4.0 TPS | v6.0.0 TPS  | v5.4.0 95% レイテンシ（ms） | v6.0.0 95% レイテンシ（ms）   | TPS 改善率 (%)  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|260085.19|265207.73|1.82|1.93|1.97|
|600|378098.48|365173.66|2.48|2.61|-3.42|
|900|441294.61|424031.23|3.75|3.49|-3.91|

v5.4.0と比較して、v6.0.0のポイントセレクトのパフォーマンスはわずかに1.79%低下しています。

![Point Select](/media/sysbench_v540vsv600_point_select.png)

### インデックスの更新性能

| スレッド数 | v5.4.0 TPS | v6.0.0 TPS  | v5.4.0 95% レイテンシ（ms） | v6.0.0 95% レイテンシ（ms）   | TPS 改善率 (%)  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|41528.7|40814.23|11.65|11.45|-1.72|
|600|53220.96|51746.21|19.29|20.74|-2.77|
|900|59977.58|59095.34|26.68|28.16|-1.47|

```
      + Compared with v5.4.0, the Update Non-index performance of v6.0.0 is slightly dropped by 1.98%.
      ![Update Non-index](/media/sysbench_v540vsv600_update_non_index.png)

      + ### Update Index performance

      | Threads   | v5.4.0 TPS | v6.0.0 TPS  | v5.4.0 95% latency (ms) | v6.0.0 95% latency (ms)   | TPS improvement (%)  |
      |:----------|:----------|:----------|:----------|:----------|:----------|
      |300|18659.11|18187.54|23.95|25.74|-2.53|
      |600|23195.83|22270.81|40.37|44.17|-3.99|
      |900|25798.31|25118.78|56.84|57.87|-2.63|

      + Compared with v5.4.0, the Update Index performance of v6.0.0 is dropped by 3.05%.
      ![Update Index](/media/sysbench_v540vsv600_update_index.png)

      + ### Read Write performance

      | Threads   | v5.4.0 TPS  | v6.0.0 TPS | v5.4.0 95% latency (ms) | v6.0.0 95% latency (ms)   | TPS improvement (%)  |
      |:----------|:----------|:----------|:----------|:----------|:----------|
      |300|4141.72|4829.01|97.55|82.96|16.59|
      |600|4892.76|5693.12|173.58|153.02|16.36|
      |900|5217.94|6029.95|257.95|235.74|15.56|

      + Compared with v5.4.0, the Read Write performance of v6.0.0 is significantly improved by 16.17%.
      ![Read Write](/media/sysbench_v540vsv600_read_write.png)
```