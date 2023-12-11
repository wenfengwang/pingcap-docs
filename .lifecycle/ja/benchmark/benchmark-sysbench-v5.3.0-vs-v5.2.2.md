---
title: TiDB Sysbenchパフォーマンステストレポート -- v5.3.0 vs. v5.2.2
---

# TiDB Sysbenchパフォーマンステストレポート -- v5.3.0 vs. v5.2.2

## テスト概要

このテストは、TiDB v5.3.0とTiDB v5.2.2のSysbenchパフォーマンスをオンライントランザクション処理（OLTP）シナリオで比較することを目的としています。その結果、v5.3.0のパフォーマンスはv5.2.2とほぼ同等であることが示されました。

## テスト環境（AWS EC2）

### ハードウェア構成

| サービスタイプ         | EC2タイプ     | インスタンス数 |
|:----------|:----------|:----------|
| PD        | m5.xlarge |     3     |
| TiKV      | i3.4xlarge|     3     |
| TiDB      | c5.4xlarge|     3     |
| Sysbench  | c5.9xlarge|     1     |

### ソフトウェアバージョン

| サービスタイプ   | ソフトウェアバージョン    |
|:----------|:-----------|
| PD        | v5.2.2とv5.3.0   |
| TiDB      | v5.2.2とv5.3.0   |
| TiKV      | v5.2.2とv5.3.0   |
| Sysbench  | 1.1.0-ead2689   |

### パラメータ構成

TiDB v5.3.0とTiDB v5.2.2は同じ構成を使用しています。

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

```yaml
global                                     # グローバル設定。
   chroot      /var/lib/haproxy            # 現在のディレクトリを変更し、セキュリティを向上させるために起動プロセスにスーパーユーザ特権を設定します。
   pidfile     /var/run/haproxy.pid        # HAProxyプロセスのPIDをこのファイルに書き込みます。
   maxconn     4000                        # 1つのHAProxyプロセスに対する同時接続の最大数。
   user        haproxy                     # UIDパラメータと同じです。
   group       haproxy                     # GIDパラメータと同じです。専用のユーザグループが推奨されます。
   nbproc      64                          # プロセスがデーモンになるときに作成されるプロセス数。複数のプロセスを開始してリクエストを転送する場合、HAProxyがプロセスをブロックしないようにするために十分な値に設定してください。
   daemon                                  # プロセスをバックグラウンドにフォークします。これはコマンドラインの"-D"引数と同等です。コマンドラインの"-db"引数で無効にできます。

defaults                                   # デフォルト設定。
   log global                              # グローバル設定の設定を継承します。
   retries 2                               # アップストリームサーバに接続する最大リトライ回数。接続試行回数がこの値を超えると、バックエンドサーバは利用できないと見なされます。
   timeout connect  2s                     # バックエンドサーバへの接続試行が成功するまでの最大待機時間。サーバーがHAProxyと同じLANにある場合は、より短い時間に設定する必要があります。
   timeout client 30000s                   # クライアント側の最大非アクティブ時間。
   timeout server 30000s                   # サーバ側の最大非アクティブ時間。

listen tidb-cluster                        # データベースの負荷分散。
   bind 0.0.0.0:3390                       # Floating IPアドレスとリスニングポート。
   mode tcp                                # HAProxyはレイヤ4、トランスポートレイヤを使用します。
   balance roundrobin                      # 最も少ない接続を持つサーバが接続を受けます。HTTPなどの短いセッションを使用するプロトコルではなく、長時間のセッションが予想されるLDAP、SQL、TSEなどのプロトコルでは、「leastconn」が推奨されます。アルゴリズムはダイナミックであり、サーバーの重みがランタイムで調整されるため、スタートが遅い場合など、動的です。
   server tidb-1 10.9.18.229:4000 check inter 2000 rise 2 fall 3       # 2000ミリ秒ごとにポート4000を検出します。2回成功するとサーバが利用可能と見なされます。3回失敗するとサーバが利用できないと見なされます。
   server tidb-2 10.9.39.208:4000 check inter 2000 rise 2 fall 3
   server tidb-3 10.9.64.166:4000 check inter 2000 rise 2 fall 3
```

## テストプラン

1. TiUPを使用してTiDB v5.3.0およびv5.2.2を展開します。
2. Sysbenchを使用して16のテーブルをインポートし、各テーブルに1,000万行のデータを含めます。
3. 各テーブルに`analyze table`ステートメントを実行します。
4. 異なる同時性テストの前に復元用に使用されるデータをバックアップし、各テストのデータの整合性を確保します。
5. Sysbenchクライアントを起動し、`point_select`、`read_write`、`update_index`、`update_non_index`テストを実行します。各ワークロードごとの各同時性で、テストは20分間続きます。
6. 各タイプのテストが完了した後、クラスターを停止し、ステップ4でのバックアップデータでクラスターを上書きし、クラスターを再起動します。

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

### ポイント選択パフォーマンス

| Threads   | v5.2.2 TPS | v5.3.0 TPS | v5.2.2 95%レイテンシ（ms） | v5.3.0 95%レイテンシ（ms） | TPS改善率（%） |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|267673.17|267516.77|1.76|1.67|-0.06|
|600|369820.29|361672.56|2.91|2.97|-2.20|
|900|417143.31|416479.47|4.1|4.18|-0.16|

v5.2.2と比較して、v5.3.0のポイント選択パフォーマンスはわずかに0.81%低下しました。

![ポイント選択](/media/sysbench_v522vsv530_point_select.png)

### インデックスなしの更新パフォーマンス

| Threads   | v5.2.2 TPS | v5.3.0 TPS  | v5.2.2 95%レイテンシ（ms） | v5.3.0 95%レイテンシ（ms）   | TPS改善率（%）  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|39715.31|40041.03|11.87|12.08|0.82|
|600|50239.42|51110.04|20.74|20.37|1.73|
```
| 900 | 57073.97 | 57252.74 | 28.16 | 27.66 | 0.31 |

v5.3.0のUpdate Non-indexパフォーマンスは、v5.2.2と比較してわずかに0.95%改善されています。

![Update Non-index](/media/sysbench_v522vsv530_update_non_index.png)

### Update Indexパフォーマンス

| Threads   | v5.2.2 TPS | v5.3.0 TPS  | v5.2.2 95% レイテンシ (ms) | v5.3.0 95% レイテンシ (ms)   | TPS 改善率 (%)  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|17634.03|17821.1|25.74|25.74|1.06|
|600|20998.59|21534.13|46.63|45.79|2.55|
|900|23420.75|23859.64|64.47|62.19|1.87|

v5.3.0のUpdate Indexパフォーマンスは、v5.2.2と比較してわずかに1.83%改善されています。

![Update Index](/media/sysbench_v522vsv530_update_index.png)

### Read Writeパフォーマンス

| Threads   | v5.2.2 TPS  | v5.3.0 TPS | v5.2.2 95% レイテンシ (ms) | v5.3.0 95% レイテンシ (ms)   | TPS 改善率 (%)  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|3872.01|3848.63|106.75|106.75|-0.60|
|600|4514.17|4471.77|200.47|196.89|-0.94|
|900|4877.05|4861.45|287.38|282.25|-0.32|

v5.3.0のRead Writeパフォーマンスは、v5.2.2と比較してわずかに0.62%低下しています。

![Read Write](/media/sysbench_v522vsv530_read_write.png)
```