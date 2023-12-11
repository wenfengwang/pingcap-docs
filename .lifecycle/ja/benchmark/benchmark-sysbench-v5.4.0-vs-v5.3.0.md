---
title: TiDB Sysbench Performance Test Report -- v5.4.0 vs. v5.3.0
---

# TiDB Sysbench Performance Test Report -- v5.4.0 vs. v5.3.0

## テストの概要

このテストは、TiDB v5.4.0とTiDB v5.3.0のSysbenchパフォーマンスをオンライントランザクション処理（OLTP）シナリオで比較することを目的としています。結果は、v5.4.0のパフォーマンスが書き込み重視のワークロードで2.59%〜4.85%改善されていることを示しています。

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
| PD        | v5.3.0およびv5.4.0   |
| TiDB      | v5.3.0およびv5.4.0   |
| TiKV      | v5.3.0およびv5.4.0   |
| Sysbench  | 1.1.0-ead2689   |

### パラメータ構成

TiDB v5.4.0とTiDB v5.3.0は同じ構成を使用しています。

#### TiDBパラメータ構成

{{< コピー可能 "" >}}

```yaml
log.level: "error"
performance.max-procs: 20
prepared-plan-cache.enabled: true
tikv-client.max-batch-wait-time: 2000000
```

#### TiKVパラメータ構成

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
```

#### TiDBグローバル変数構成

{{< コピー可能 "sql" >}}

```sql
set global tidb_hashagg_final_concurrency=1;
set global tidb_hashagg_partial_concurrency=1;
set global tidb_enable_async_commit = 1;
set global tidb_enable_1pc = 1;
set global tidb_guarantee_linearizability = 0;
set global tidb_enable_clustered_index = 1;
```

#### HAProxy構成 - haproxy.cfg

TiDBでHAProxyを使用する方法の詳細については、[TiDBでHAProxyを使用するためのベストプラクティス](/best-practices/haproxy-best-practices.md)を参照してください。

{{< コピー可能 "" >}}

```yaml
global                                     # グローバル構成
   chroot      /var/lib/haproxy            # 現在のディレクトリを変更し、セキュリティを向上させるために起動プロセスにスーパーユーザ権限を設定します。
   pidfile     /var/run/haproxy.pid        # HAProxyプロセスのPIDをこのファイルに書き込みます。
   maxconn     4000                        # 単一のHAProxyプロセスに対する同時接続数の最大値。
   user        haproxy                     # UIDパラメータと同じ。
   group       haproxy                     # GIDパラメータと同じ。専用のユーザーグループを推奨します。
   nbproc      64                          # デーモンに移行するときに作成されるプロセス数。複数のプロセスを開始してリクエストを転送する場合は、HAProxyがプロセスをブロックしないように十分に大きな値にする必要があります。
   daemon                                  # プロセスをバックグラウンドにフォークします。これはコマンドラインの"-D"引数と同等です。コマンドラインの"-db"引数で無効にすることができます。
defaults                                   # デフォルト構成
   log global                              # グローバル構成の設定を継承します。
   retries 2                               # アップストリームサーバに接続しようとするリトライの最大回数。接続試行回数がこの値を超えると、バックエンドサーバは利用不可と見なされます。
   timeout connect  2s                     # バックエンドサーバへの接続試行が成功するまでの最大時間。サーバがHAProxyと同じLANにある場合は、短い時間に設定する必要があります。
   timeout client 30000s                   # クライアント側の最大非アクティブ時間。
   timeout server 30000s                   # サーバ側の最大非アクティブ時間。
listen tidb-cluster                        # データベース負荷分散。
   bind 0.0.0.0:3390                       # 浮動IPアドレスとリスニングポート。
   mode tcp                                # HAProxyはレイヤ4、トランスポート層を使用します。
   balance roundrobin                      # 最も少ない接続を持つサーバが接続を受けます。HTTPなどの短いセッションを使用するプロトコルではなく、LDAP、SQL、TSEなど、長いセッションが予想される場合は、「leastconn」が推奨されます。アルゴリズムは動的であり、サーバの重みは必要に応じて動的に調整されます。
   server tidb-1 10.9.18.229:4000 check inter 2000 rise 2 fall 3       # 2000ミリ秒ごとにポート4000を検出します。2回成功した場合、サーバは利用可能と見なされます。3回失敗した場合、サーバは利用不可と見なされます。
   server tidb-2 10.9.39.208:4000 check inter 2000 rise 2 fall 3
   server tidb-3 10.9.64.166:4000 check inter 2000 rise 2 fall 3
```

## テスト計画

1. TiUPを使用してTiDB v5.4.0およびv5.3.0をデプロイします。
2. Sysbenchを使用して、それぞれ1,000万行のデータを持つ16のテーブルをインポートします。
3. 各テーブルに対して`analyze table`ステートメントを実行します。
4. 異なる並行性のテストの前にリストア用のデータをバックアップし、各テストごとにデータの整合性を確保します。
5. Sysbenchクライアントを起動して、`point_select`、`read_write`、`update_index`、および`update_non_index`のテストを実行します。HAProxyを介してTiDBにストレステストを実行します。各ワークロードの各並行性で、テストは20分かかります。
6. 各種類のテストが完了した後、クラスタを停止し、ステップ4のバックアップデータでクラスタを上書きし、クラスタを再起動します。

### テストデータの準備

次のコマンドを実行して、テストデータを準備します。

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

次のコマンドを実行して、テストを実行します。

{{< コピー可能 "shell-regular" >}}

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

### ポイント選択のパフォーマンス

| Threads   | v5.3.0 TPS | v5.4.0 TPS  | v5.3.0 95% レイテンシ（ミリ秒） | v5.4.0 95% レイテンシ（ミリ秒）   | TPS改善率（%）  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|266041.84|264345.73|1.96|2.07|-0.64|
|600|351782.71|348715.98|3.43|3.49|-0.87|
|900|386553.31|399777.11|5.09|4.74|3.42|

v5.4.0はv5.3.0と比較して、ポイント選択のパフォーマンスがわずかに0.64%改善されています。

![ポイント選択](/media/sysbench_v530vsv540_point_select.png)

### インデックスなしの更新パフォーマンス

| Threads   | v5.3.0 TPS | v5.4.0 TPS  | v5.3.0 95% レイテンシ（ミリ秒） | v5.4.0 95% レイテンシ（ミリ秒）   | TPS改善率（%）  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|40804.31|41187.1|11.87|11.87|0.94|
|600|51239.4|53172.03|20.74|19.65|3.77|
|900|57897.56|59666.8|27.66|27.66|3.06|

v5.4.0では、v5.3.0と比較してUpdate Non-indexのパフォーマンスが2.59%向上しました。

![Update Non-index](/media/sysbench_v530vsv540_update_non_index.png)

### Update Indexのパフォーマンス

| Threads   | v5.3.0 TPS | v5.4.0 TPS  | v5.3.0 95% latency (ms) | v5.4.0 95% latency (ms)   | TPS improvement (%)  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|17737.82|18716.5|26.2|24.83|5.52|
|600|21614.39|22670.74|44.98|42.61|4.89|
|900|23933.7|24922.05|62.19|61.08|4.13|

v5.4.0では、v5.3.0と比較してUpdate Indexのパフォーマンスが4.85%向上しました。

![Update Index](/media/sysbench_v530vsv540_update_index.png)

### Read Writeのパフォーマンス

| Threads   | v5.3.0 TPS  | v5.4.0 TPS | v5.3.0 95% latency (ms) | v5.4.0 95% latency (ms)   | TPS improvement (%)  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|3810.78|3929.29|108.68|106.75|3.11|
|600|4514.28|4684.64|193.38|186.54|3.77|
|900|4842.49|4988.49|282.25|277.21|3.01|

v5.4.0では、v5.3.0と比較してRead Writeのパフォーマンスが3.30%向上しました。

![Read Write](/media/sysbench_v530vsv540_read_write.png)