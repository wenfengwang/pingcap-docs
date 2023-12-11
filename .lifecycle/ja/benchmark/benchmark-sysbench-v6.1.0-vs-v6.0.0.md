---
title: TiDB Sysbench パフォーマンステストレポート -- v6.1.0 vs. v6.0.0
---

# TiDB Sysbench パフォーマンステストレポート -- v6.1.0 vs. v6.0.0

## テスト概要

このテストは、TiDB v6.1.0とTiDB v6.0.0のSysbenchパフォーマンスを、オンライントランザクション処理（OLTP）シナリオで比較することを目的としています。その結果、v6.1.0のパフォーマンスは書き込みの作業量において改善されています。書き込みが中心の作業量では、パフォーマンスが2.33%～4.61%向上しています。

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
| PD        | v6.0.0 および v6.1.0 |
| TiDB      | v6.0.0 および v6.1.0 |
| TiKV      | v6.0.0 および v6.1.0 |
| Sysbench  | 1.1.0-df89d34   |

### パラメータ設定

TiDB v6.1.0およびTiDB v6.0.0は、同じ構成を使用しています。

#### TiDBパラメータ設定

{{< copyable "" >}}

```yaml
log.level: "error"
prepared-plan-cache.enabled: true
tikv-client.max-batch-wait-time: 2000000
```

#### TiKVパラメータ設定

{{< copyable "" >}}

```yaml
storage.scheduler-worker-pool-size: 5
raftstore.store-pool-size: 3
raftstore.apply-pool-size: 3
rocksdb.max-background-jobs: 8
server.grpc-concurrency: 6
readpool.storage.normal-concurrency: 10
```

#### TiDBグローバル変数設定

{{< copyable "sql" >}}

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

TiDBでHAProxyを使用する方法の詳細については、[TiDBでHAProxyを使用するベストプラクティス](/best-practices/haproxy-best-practices.md)を参照してください。

{{< copyable "" >}}

```yaml
global                                     # Global configuration.
   pidfile     /var/run/haproxy.pid        # HAProxyプロセスのPIDをこのファイルに書き込みます。
   maxconn     4000                        # 単一のHAProxyプロセスに対する同時接続数の最大値。
   user        haproxy                     # UIDパラメータと同じ。
   group       haproxy                     # GIDパラメータと同じ。専用のユーザーグループが推奨されます。
   nbproc      64                          # デーモン化時に作成されるプロセス数。複数のプロセスを開始してリクエストを転送する場合、HAProxyがプロセスをブロックしないように十分な値にする。
   daemon                                  # プロセスをバックグラウンドでフォークします。これはコマンドラインの「-D」引数と同等です。コマンドラインの「-db」引数で無効にすることができます。

defaults                                   # デフォルトの設定。
   log global                              # グローバル設定の継承。
   retries 2                               # アップストリームサーバーに接続する最大リトライ回数。接続試行回数がこの値を超えると、バックエンドサーバーは利用不可と見なされます。
   timeout connect  2s                     # バックエンドサーバーへの接続試行が成功するまでの最大待ち時間。サーバーがHAProxyと同じLANにある場合、短い時間に設定する必要があります。
   timeout client 30000s                   # クライアント側の最大非アクティブ時間。
   timeout server 30000s                   # サーバー側の最大非アクティブ時間。

listen tidb-cluster                        # データベースの負荷分散。
   bind 0.0.0.0:3390                       # フローティングIPアドレスとリスニングポート。
   mode tcp                                # HAProxyはレイヤー4、トランスポート層を使用します。
   balance leastconn                      # 最も少ない接続を持つサーバーが接続を受けます。長時間のセッションが期待される場合（例: LDAP、SQL、TSEなど）、"leastconn"が推奨されます。状況に応じてサーバーのウェイトが動的に調整されるため、例えば遅れた起動に対応することができます。
   server tidb-1 10.9.18.229:4000 check inter 2000 rise 2 fall 3       # 2000ミリ秒ごとにポート4000を検出します。成功した検出が2回あれば、サーバーは利用可能; 失敗が3回あれば、サーバーは利用不可と見なされます。
   server tidb-2 10.9.39.208:4000 check inter 2000 rise 2 fall 3
   server tidb-3 10.9.64.166:4000 check inter 2000 rise 2 fall 3
```

## テスト計画

1. TiUPを使用して、TiDB v6.1.0およびv6.0.0をデプロイします。
2. Sysbenchを使用して、それぞれ1,000万行のデータを持つ16個のテーブルをインポートします。
3. 各テーブルに対して`analyze table`ステートメントを実行します。
4. 異なる並行性テストの前に、リストア用にデータをバックアップします。これにより、各テストのデータ整合性が確保されます。
5. Sysbenchクライアントを起動して、`point_select`、`read_write`、`update_index`、および`update_non_index`テストを実行します。さらに、HAProxyを介してTiDBにストレステストを実施します。各ワークロードの各並行性に対して、テスト時間は20分です。
6. 各種類のテストが完了した後、クラスターを停止し、手順4でバックアップしたデータでクラスターを上書きし、クラスターを再起動します。

### テストデータの準備

次のコマンドを実行して、テストデータを準備します：

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

### テストの実施

次のコマンドを実行して、テストを実施します：

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

| スレッド数  | v6.0.0 TPS | v6.1.0 TPS  | v6.0.0 95%レイテンシ（ms） | v6.1.0 95%レイテンシ（ms）   | TPS改善率 (%)  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|268934.84|265353.15|1.89|1.96|-1.33|
|600|365217.96|358976.94|2.57|2.66|-1.71|
|900|420799.64|407625.11|3.68|3.82|-3.13|

v6.0.0と比較して、v6.1.0のポイントセレクトのパフォーマンスはわずかに2.1%低下しています。

![ポイントセレクト](/media/sysbench_v600vsv610_point_select.png)

### 非インデックス更新のパフォーマンス

| スレッド数  | v6.0.0 TPS | v6.1.0 TPS  | v6.0.0 95%レイテンシ（ms） | v6.1.0 95%レイテンシ（ms）   | TPS改善率 (%)  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|41778.95|42991.9|11.24|11.45|2.90 |
|600|52045.39|54099.58|20.74|20.37|3.95|
|900|59243.35|62084.65|27.66|26.68|4.80|

v6.0.0と比較して、v6.1.0の非インデックス更新のパフォーマンスは3.88%改善されています。
![非インデックスの更新](/media/sysbench_v600vsv610_update_non_index.png)

### インデックスの更新パフォーマンス

| スレッド   | v6.0.0 TPS | v6.1.0 TPS  | v6.0.0 95% レイテンシ（ms） | v6.1.0 95% レイテンシ（ms）   | TPS 改善率（％）  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|18085.79|19198.89|25.28|23.95|6.15|
|600|22210.8|22877.58|42.61|41.85|3.00|
|900|25249.81|26431.12|55.82|53.85|4.68|

v6.1.0はv6.0.0と比較して、更新インデックスのパフォーマンスが4.61％改善されています。

![更新インデックス](/media/sysbench_v600vsv610_update_index.png)

### 読み書きパフォーマンス

| スレッド   | v6.0.0 TPS  | v6.1.0 TPS | v6.0.0 95% レイテンシ（ms） | v6.1.0 95% レイテンシ（ms）   | TPS 改善率（％）  |
|:----------|:----------|:----------|:----------|:----------|:----------|
|300|4856.23|4914.11|84.47|82.96|1.19|
|600|5676.46|5848.09|161.51|150.29|3.02|
|900|6072.97|6223.95|240.02|223.34|2.49|

v6.1.0はv6.0.0と比較して、読み書きパフォーマンスが2.23％改善されています。

![読み書き](/media/sysbench_v600vsv610_read_write.png)