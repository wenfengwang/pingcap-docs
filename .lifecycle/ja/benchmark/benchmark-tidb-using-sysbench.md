---
title: Sysbenchを使用してTiDBをテストする方法
aliases: ['/docs/dev/benchmark/benchmark-tidb-using-sysbench/','/docs/dev/benchmark/how-to-run-sysbench/']
---

# Sysbenchを使用してTiDBをテストする方法

[ここからダウンロードできます](https://github.com/akopytov/sysbench/releases/tag/1.0.20)。

## テスト計画

### TiDBの構成

ログレベルが高いほど出力されるログが少なくなり、その結果、TiDBのパフォーマンスにプラスの影響を与えます。具体的には、TiUP構成ファイルに次のコマンドを追加できます:

```yaml
server_configs:
  tidb:
    log.level: "error"
```

また、[`tidb_enable_prepared_plan_cache`](/system-variables.md#tidb_enable_prepared_plan_cache-new-in-v610)が有効になっており、Sysbenchが`--db-ps-mode=auto`を使用してプリペアドステートメントを使用できることを確認することをお勧めします。SQLプリペアド実行計画キャッシュについては、[SQL Prepared Execution Plan Cache](/sql-prepared-plan-cache.md)を参照して、SQLプランキャッシュが何を行い、それをどのように監視するかについてのドキュメントを参照してください。

> **注:**
>
> Sysbenchの異なるバージョンでは、`db-ps-mode`のデフォルト値が異なる場合があります。コマンドで明示的に指定することをお勧めします。

### TiKVの構成

ログレベルが高いほどTiKVのパフォーマンスが向上します。

TiKVクラスタには、デフォルトCF、書き込みCF、ロックCFを含む複数のColumn Familyが主に異なるタイプのデータを保存するために使用されています。Sysbenchテストでは、デフォルトCFと書き込みCFに焦点を当てるだけで十分です。データのインポートに使用されるColumn Familyは、TiDBクラスタ間で一定の比を持っています:

デフォルトCF：書き込みCF＝4：1

TiKV上のRocksDBのブロックキャッシュの構成は、マシンのメモリサイズに基づいて行う必要があり、メモリを最大限に活用するためです。40GB仮想マシンにTiKVクラスタを展開する場合、以下のようにブロックキャッシュを構成することをお勧めします:

```yaml
server_configs:
  tikv:
    log-level: "error"
    rocksdb.defaultcf.block-cache-size: "24GB"
    rocksdb.writecf.block-cache-size: "6GB"
```

また、TiKVをブロックキャッシュを共有するように構成することもできます:

```yaml
server_configs:
  tikv:
    storage.block-cache.capacity: "30GB"
```

TiKVパフォーマンスチューニングの詳細については、[Tune TiKV Performance](/tune-tikv-memory-performance.md)を参照してください。

## テストプロセス

> **注:**
>
> この文書のテストは、HAproxyなどの負荷分散ツールを使用せずに実行しました。Sysbenchテストは個々のTiDBノードで実行し、その結果を合計しました。負荷分散ツールと異なるバージョンのパラメータもパフォーマンスに影響する可能性があります。

### Sysbenchの構成

以下はSysbench構成ファイルの例です:

```txt
mysql-host={TIDB_HOST}
mysql-port=4000
mysql-user=root
mysql-password=password
mysql-db=sbtest
time=600
threads={8, 16, 32, 64, 128, 256}
report-interval=10
db-driver=mysql
```

これらのパラメータは必要に応じて調整できます。その中で`TIDB_HOST`はTiDBサーバのIPアドレスです(複数のアドレスを構成ファイルに含めることはできないため)。`threads`はテスト中の並行接続の数であり、"8, 16, 32, 64, 128, 256"のように調整できます。データのインポート時には、`threads`を8または16に設定することをお勧めします。`threads`を調整した後、**config**という名前のファイルに保存してください。

次の例は、**config**ファイルのサンプルです:

```txt
mysql-host=172.16.30.33
mysql-port=4000
mysql-user=root
mysql-password=password
mysql-db=sbtest
time=600
threads=16
report-interval=10
db-driver=mysql
```

### データのインポート

> **注:**
>
> 楽観的トランザクションモデルを有効にした場合（TiDBはデフォルトで悲観的トランザクションモードを使用します）、TiDBは並行性の競合を見つけたときにトランザクションをロールバックします。`tidb_disable_txn_auto_retry`を`off`に設定すると、トランザクションの競合が発生した後に自動リトライ機構をオンにすることができ、トランザクション競合エラーのためにSysbenchが終了するのを防ぐことができます。

データをインポートする前に、TiDBにいくつかの設定を行う必要があります。MySQLクライアントで次のコマンドを実行します:

{{< copyable "sql" >}}

```sql
set global tidb_disable_txn_auto_retry = off;
```

その後、クライアントを終了します。

MySQLクライアントを再起動し、次のSQLステートメントを実行して`sbtest`というデータベースを作成します:

{{< copyable "sql" >}}

```sql
create database sbtest;
```

Sysbenchスクリプトがインデックスを作成する順序を調整してください。Sysbenchは"テーブルの作成→データの挿入→インデックスの作成"の順にデータをインポートしますが、これによりTiDBがデータのインポートにより多くの時間を要することになります。ユーザーはデータのインポートを高速化するための順序を調整できます。たとえば、Sysbenchのバージョン[1.0.20](https://github.com/akopytov/sysbench/tree/1.0.20)を使用しているとします。以下の2つの方法のいずれかで順序を調整できます:

- [oltp_common.lua](https://raw.githubusercontent.com/pingcap/tidb-bench/master/sysbench/sysbench-patch/oltp_common.lua)を修正されたファイルとしてダウンロードし、`/usr/share/sysbench/oltp_common.lua`ファイルでそれを上書きします。
- `/usr/share/sysbench/oltp_common.lua`で、[235-240行目](https://github.com/akopytov/sysbench/blob/1.0.20/src/lua/oltp_common.lua#L235-L240)を198行目の直後に移動します。

> **注:**
>
> この操作はオプションであり、データのインポートにかかる時間を節約するためのものです。

コマンドラインで次のコマンドを入力してデータのインポートを開始します。構成ファイルは前のステップで構成したものです:

{{< copyable "shell-regular" >}}

```bash
sysbench --config-file=config oltp_point_select --tables=32 --table-size=10000000 prepare
```

### データのウォーミングと統計の収集

データをウォーミングするために、ディスクからデータをメモリのブロックキャッシュに読み込みます。ウォーミングされたデータはシステム全体のパフォーマンスを大幅に向上させます。クラスタを再起動した後に一度データのウォーミングを行うことをお勧めします。

```bash
sysbench --config-file=config oltp_point_select --tables=32 --table-size=10000000 prewarm
```

### ポイント選択テストコマンド

{{< copyable "shell-regular" >}}

```bash
sysbench --config-file=config oltp_point_select --tables=32 --table-size=10000000 --db-ps-mode=auto --rand-type=uniform run
```

### インデックス更新テストコマンド

{{< copyable "shell-regular" >}}

```bash
sysbench --config-file=config oltp_update_index --tables=32 --table-size=10000000 --db-ps-mode=auto --rand-type=uniform run
```

### 読み取り専用テストコマンド

{{< copyable "shell-regular" >}}

```bash
sysbench --config-file=config oltp_read_only --tables=32 --table-size=10000000 --db-ps-mode=auto --rand-type=uniform run
```

## よくある問題

### 高並行性下でTiDBとTiKVが適切に構成されているにもかかわらず、全体のパフォーマンスが低いのはなぜですか？

この問題はしばしばプロキシの使用と関係があります。単一のTiDBサーバに負荷を加え、各結果を合計し、プロキシを使用した場合の合計結果と比較してください。

HAproxyを例に挙げます。`nbproc`パラメータを使用して、最大で開始できるプロセスの数を増やすことができます。後のバージョンのHAproxyでは`nbthread`と`cpu-map`もサポートしています。これらすべてがパフォーマンスに与えるプロキシの使用の負の影響を緩和することができます。

### 高並行性下でTiKVのCPU利用率が低いのはなぜですか？

TiKVの全体的なCPU利用率が低い場合でも、クラスタ内の特定のモジュールのCPU利用率が高い可能性があります。

ストレージリードプール、コプロセッサ、gRPCなどのTiKV上の他のモジュールの最大並行性制限は、TiKV構成ファイルを介して調整できます。

実際のCPU使用状況はGrafanaのTiKVスレッドCPUモニターパネルで観察できます。モジュールにボトルネックがある場合は、モジュールの並行性を増やすことで調整できます。

### 高並行性下でTiKVがまだCPU使用率のボトルネックに達していないのに、TiDBのCPU利用率がまだ低いのはなぜですか？

NUMAアーキテクチャのCPUは、リモートメモリへのクロス-CPUアクセスがパフォーマンスを大幅に低下させる高性能な装置で使用されます。デフォルトでTiDBはサーバのすべてのCPUを使用し、ゴルーチンのスケジューリングは必然的にクロス-CPUメモリアクセスをもたらします。

そのため、NUMAアーキテクチャのサーバ上に*n*個のTiDB（*n*はNUMA CPUの数です）を展開し、同時にTiDBパラメータ`max-procs`をNUMA CPUコアの数と同じ値に設定することをお勧めします。