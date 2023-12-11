---
title: 高コストなクエリの特定
aliases: ['/docs/dev/identify-expensive-queries/','/docs/dev/how-to/maintain/identify-abnormal-queries/identify-expensive-queries/']
---

# 高コストなクエリの特定

TiDBはSQL実行中に高コストなクエリを特定できるようにします。これにより、SQL実行のパフォーマンスを診断し、改善することができます。具体的には、TiDBは実行時間が[`tidb_expensive_query_time_threshold`](/system-variables.md#tidb_expensive_query_time_threshold)（デフォルトでは60秒）を超えたステートメントやメモリ使用量が[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)（デフォルトでは1GB）を超えたステートメントの情報を [tidb-server log file](/tidb-configuration-file.md#logfile)（デフォルトでは "tidb.log"）に出力します。

> **注意:**
>
> 高コストなクエリログは[遅いクエリログ](/identify-slow-queries.md)と異なります。TiDBはリソースの使用量（実行時間またはメモリ使用量）の閾値を超える**直後に**高コストなクエリログにステートメントの情報を出力しますが、遅いクエリログにステートメントの情報をステートメント実行**後に**出力します。

## 高コストなクエリログの例

```sql
[2020/02/05 15:32:25.096 +08:00] [WARN] [expensivequery.go:167] [expensive_query] [cost_time=60.008338935s] [wait_time=0s] [request_count=1] [total_keys=70] [process_keys=65] [num_cop_tasks=1] [process_avg_time=0s] [process_p90_time=0s] [process_max_time=0s] [process_max_addr=10.0.1.9:20160] [wait_avg_time=0.002s] [wait_p90_time=0.002s] [wait_max_time=0.002s] [wait_max_addr=10.0.1.9:20160] [stats=t:pseudo] [conn_id=60026] [user=root] [database=test] [table_ids="[122]"] [txn_start_ts=414420273735139329] [mem_max="1035 Bytes (1.0107421875 KB)"] [sql="insert into t select sleep(1) from t"]
```

## Fields description

基本的なフィールド:

* `cost_time`: ログが印刷された時のステートメントの実行時間。
* `stats`: ステートメントに関与するテーブルまたはインデックスの統計のバージョン。値が`pseudo`である場合、利用可能な統計情報がないことを意味します。この場合、テーブルまたはインデックスを解析する必要があります。
* `table_ids`: ステートメントに関与するテーブルのID。
* `txn_start_ts`: トランザクションの開始タイムスタンプと一意のID。この値を使用して、トランザクションに関連するログを検索できます。
* `sql`: SQLステートメント。

メモリ使用量関連のフィールド:

* `mem_max`: ログが印刷された時のステートメントのメモリ使用量。このフィールドにはメモリ使用量を測定するための2種類の単位があります：バイトおよび他の読みやすく適応可能な単位（MBやGBなど）。

ユーザ関連のフィールド:

* `user`: ステートメントを実行するユーザの名前。
* `conn_id`: 接続ID（セッションID）。例えば、セッションIDが`60026`であるログを検索するために`con:60026`キーワードを使用できます。
* `database`: ステートメントが実行されるデータベース。

TiKV Coprocessorタスク関連のフィールド:

* `wait_time`: TiKVにおけるステートメントのすべてのコプロセッサリクエストの総待ち時間。TiKVのコプロセッサは限られた数のスレッドで動作するため、すべてのコプロセッサのスレッドが動作しているときにリクエストがキューイングされることがあります。キュー内のリクエストが処理に長い時間を要すると、後続のリクエストの待ち時間が増加します。
* `request_count`: ステートメントが送信するコプロセッサリクエストの数。
* `total_keys`: コプロセッサがスキャンしたキーの数。
* `processed_keys`: コプロセッサが処理したキーの数。`total_keys`と比較して、`processed_keys`はMVCCの古いバージョンを含みません。`processed_keys`と`total_keys`の大きな差は多くの古いバージョンが存在することを示します。
* `num_cop_tasks`: ステートメントが送信するコプロセッサリクエストの数。
* `process_avg_time`: コプロセッサタスクの平均実行時間。
* `process_p90_time`: コプロセッサタスクのP90実行時間。
* `process_max_time`: コプロセッサタスクの最大実行時間。
* `process_max_addr`: 最も長い実行時間を持つコプロセッサタスクのアドレス。
* `wait_avg_time`: コプロセッサタスクの平均待ち時間。
* `wait_p90_time`: コプロセッサタスクのP90待ち時間。
* `wait_max_time`: コプロセッサタスクの最大待ち時間。
* `wait_max_addr`: 最も長い待ち時間を持つコプロセッサタスクのアドレス。