---
title: EXPLAIN ANALYZE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースでのEXPLAIN ANALYZEの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-explain-analyze/','/docs/dev/reference/sql/statements/explain-analyze/']
---

# EXPLAIN ANALYZE

`EXPLAIN ANALYZE` ステートメントは、`EXPLAIN`と同様に動作しますが、主な違いは実際にステートメントを実行することです。これにより、クエリの計画の一環として使用される推定値と実際の実行中に遭遇した値を比較することができます。推定値が実際の値と大幅に異なる場合は、影響を受けるテーブルで`ANALYZE TABLE`を実行することを検討する必要があります。

> **注意:**
>
> `DML`ステートメントを実行するときに`EXPLAIN ANALYZE`を使用すると、通常、データの変更が実行されます。現時点では、`DML`ステートメントの実行計画は**まだ**表示できません。

## 概要

```ebnf+diagram
ExplainSym ::=
    'EXPLAIN'
|   'DESCRIBE'
|    'DESC'

ExplainStmt ::=
    ExplainSym ( TableName ColumnName? | 'ANALYZE'? ExplainableStmt | 'FOR' 'CONNECTION' NUM | 'FORMAT' '=' ( stringLit | ExplainFormatType ) ( 'FOR' 'CONNECTION' NUM | ExplainableStmt ) )

ExplainableStmt ::=
    SelectStmt
|   DeleteFromStmt
|   UpdateStmt
|   InsertIntoStmt
|   ReplaceIntoStmt
|   UnionStmt
```

## EXPLAIN ANALYZE 出力フォーマット

`EXPLAIN ANALYZE` は`EXPLAIN`とは異なり、対応するSQL ステートメントを実行し、その実行時情報を記録し、実行計画と一緒に情報を返します。したがって、`EXPLAIN ANALYZE`は`EXPLAIN`ステートメントの拡張と見なすことができます。`EXPLAIN`（クエリのデバッグに使用）と比較して、`EXPLAIN ANALYZE`の戻り結果には、`actRows`、`execution info`、`memory`、`disk`などの情報を含む列情報も含まれます。これらの列の詳細は以下の通りです:

| アトリビュート名 | 説明 |
|:----------------|:---------------------------------|
| actRows       | 演算子によって出力された行数。 |
| execution info  | 演算子の実行情報。`time`は演算子に入った時から演算子を離れるまでの合計の`wall time`を表し、全てのサブ演算子の実行時間を含む。もし演算子が親演算子によって多く呼び出される場合（ループ内で）、その時間は累積時間を意味する。`loops`は現在の演算子が親演算子によって呼び出される回数である。 |
| memory  | 演算子が占めるメモリ領域。 |
| disk  | 演算子が占めるディスク領域。 |

## 例

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
```

```sql
Query OK, 0 rows affected (0.12 sec)
```

{{< copyable "sql" >}}

```sql
INSERT INTO t1 (c1) VALUES (1), (2), (3);
```

```sql
Query OK, 3 rows affected (0.02 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

{{< copyable "sql" >}}

```sql
EXPLAIN ANALYZE SELECT * FROM t1 WHERE id = 1;
```

```sql
+-------------+---------+---------+------+---------------+----------------------------------------------------------------+---------------+--------+------+
| id          | estRows | actRows | task | access object | execution info                                                 | operator info | memory | disk |
+-------------+---------+---------+------+---------------+----------------------------------------------------------------+---------------+--------+------+
| Point_Get_1 | 1.00    | 1       | root | table:t1      | time:757.205µs, loops:2, Get:{num_rpc:1, total_time:697.051µs} | handle:1      | N/A    | N/A  |
+-------------+---------+---------+------+---------------+----------------------------------------------------------------+---------------+--------+------+
1 row in set (0.01 sec)
```

{{< copyable "sql" >}}

```sql
EXPLAIN ANALYZE SELECT * FROM t1;
```

```sql
+-------------------+----------+---------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------+-----------+------+
| id                | estRows  | actRows | task      | access object | execution info                                                                                                                                                                                                                            | operator info                  | memory    | disk |
+-------------------+----------+---------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------+-----------+------+
| TableReader_5     | 10000.00 | 3       | root      |               | time:278.2µs, loops:2, cop_task: {num: 1, max: 437.6µs, proc_keys: 3, rpc_num: 1, rpc_time: 423.9µs, copr_cache_hit_ratio: 0.00}                                                                                                          | data:TableFullScan_4           | 251 Bytes | N/A  |
| └─TableFullScan_4 | 10000.00 | 3       | cop[tikv] | table:t1      | tikv_task:{time:0s, loops:1}, scan_detail: {total_process_keys: 3, total_process_keys_size: 111, total_keys: 4, rocksdb: {delete_skipped_count: 0, key_skipped_count: 3, block: {cache_hit_count: 0, read_count: 0, read_byte: 0 Bytes}}} | keep order:false, stats:pseudo | N/A       | N/A  |
+-------------------+----------+---------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------+-----------+------+
2 rows in set (0.00 sec)
```

## 演算子の実行情報

`time`と`loop`の基本的な実行情報に加えて、`execution info`には演算子固有の実行情報も含まれ、その主な内容は演算子がRPCリクエストを送信するための時間や他の段階の所要時間が含まれます。

### Point_Get

`Point_Get`演算子の実行情報には、通常、次の情報が含まれます:

- `Get:{num_rpc:1, total_time:697.051µs}`: TiKVに送信された`Get` RPCリクエスト（`num_rpc`）の回数とその全体の時間（`total_time`）。
- `ResolveLock:{num_rpc:1, total_time:12.117495ms}`: TiDBがデータを読み取る際にロックに遭遇すると、まずロックを解決する必要があります。通常、これは読み取り書きの競合のシナリオで発生します。この情報は、ロックを解決する時間を示します。

### Batch_Point_Get

`Batch_Point_Get`演算子の実行情報は`Point_Get`演算子と類似していますが、`Batch_Point_Get`は通常、データを読み取るためにTiKVに`BatchGet` RPCリクエストを送信します。

`BatchGet:{num_rpc:2, total_time:83.13µs}`: TiKVに送信された`BatchGet`タイプのRPCリクエスト（`num_rpc`）の回数とそれらの全体の時間（`total_time`）。

### TableReader

`TableReader`演算子の実行情報は、通常、次のようになります:


```
cop_task: {num: 6, max: 1.07587ms, min: 844.312µs, avg: 919.601µs, p95: 1.07587ms, max_proc_keys: 16, p95_proc_keys: 16, tot_proc: 1ms, tot_wait: 1ms, rpc_num: 6, rpc_time: 5.313996 ms, copr_cache_hit_ratio: 0.00}
```

- `cop_task`: `cop`タスクの実行情報が含まれます。例えば:
    - `num`: copタスクの数。
    - `max`、`min`、`avg`、`p95`: copタスクの実行時間の最大値、最小値、平均値、P95値。
    - `max_proc_keys`および`p95_proc_keys`: すべてのcopタスクでTiKVがスキャンした最大値とP95のキー・バリュー数。最大値とP95値の差が大きい場合、データの分布が不均衡である可能性があります。
    - `rpc_num`、`rpc_time`: TiKVに送信された`Cop` RPCリクエストの総数とそれらの総時間。
    - `copr_cache_hit_ratio`: `cop`タスクのリクエストに対するCoprocessor Cacheのヒット率。

- `backoff`: 種々のバックオフのタイプとそのバックオフの総待ち時間が含まれます。

### Insert

`Insert`演算子の実行情報は、通常、次のようになります:
```
```
prepare:109.616µs, check_insert:{total_time:1.431678ms, mem_insert_time:667.878µs, prefetch:763.8µs, rpc:{BatchGet:{num_rpc:1, total_time:699.166µs},Get:{num_rpc:1, total_time:378.276µs }}}
```

- `prepare`: 書き込みの準備にかかる時間です。式、デフォルト値、および自動増分値の計算を含みます。
- `check_insert`: 通常、`insert ignore` および `insert on duplicate` のステートメントで表示される情報で、競合のチェックと TiDB トランザクションキャッシュへのデータ書き込みにかかる時間を含みます。この時間はトランザクションのコミットにかかる時間を含みません。以下の情報を含みます：
    - `total_time`: `check_insert` ステップに費やされた合計時間です。
    - `mem_insert_time`: TiDB トランザクションキャッシュにデータを書き込むのにかかる時間です。
    - `prefetch`: 競合をチェックする必要があるデータの取得にかかる時間です。このステップでは、TiKV に `Batch_Get` RPC 要求を送信してデータを取得します。
    - `rpc`: TiKV に RPC 要求を送信するのにかかる合計時間で、通常は `BatchGet` と `Get` の 2 種類の RPC 時間を含みます。その中には以下があります：
        - `BatchGet` RPC 要求は `prefetch` ステップで送信されます。
        - `Get` RPC 要求は `insert on duplicate` ステートメントが重複更新を実行するときに送信されます。
- `backoff`: 異なるタイプのバックオフとバックオフの総待ち時間を含みます。

### IndexJoin

`IndexJoin` オペレータには 1 つのアウターワーカーと N 個のインナーワーカーがあり、並行して実行されます。結合結果はアウターテーブルの順序を保持します。詳細な実行プロセスは以下のとおりです：

1. アウターワーカーは N 個のアウター行を読み取り、その後タスクにラップし、その結果チャネルおよびインナーワーカーチャネルに送信します。
2. インナーワーカーはタスクを受け取り、そのタスクからキー範囲を構築し、キー範囲に応じてインナー行を取得します。その後、インナー行のハッシュテーブルを構築します。
3. メインの `IndexJoin` スレッドは結果チャネルからタスクを受け取り、インナーワーカーがタスクの処理を完了するのを待ちます。
4. メインの `IndexJoin` スレッドは、それぞれのアウター行をインナー行のハッシュテーブルを参照しながら結合します。

`IndexJoin` オペレータには以下の実行情報が含まれています：

```
inner:{total:4.297515932s, concurrency:5, task:17, construct:97.96291ms, fetch:4.164310088s, build:35.219574ms}, probe:53.574945ms
```

- `Inner`: インナーワーカーの実行情報：
    - `total`: インナーワーカーによって費やされた合計時間です。
    - `concurrency`: 並行インナーワーカーの数です。
    - `task`: インナーワーカーが処理したタスクの総数です。
    - `construct`: インナーワーカーがタスクに対応するインナーテーブル行を読み取る前の準備にかかる時間です。
    - `fetch`: インナーワーカーがインナーテーブル行を読み取るのにかかる合計時間です。
    - `Build`: インナーワーカーが対応するインナーテーブル行のハッシュテーブルを構築するのにかかる合計時間です。
- `probe`: メインの `IndexJoin` スレッドがアウターテーブル行およびインナーテーブル行のハッシュテーブルとの結合操作を行うのに費やされた合計時間です。

### IndexHashJoin

`IndexHashJoin` オペレータの実行プロセスは `IndexJoin` オペレータと類似しています。`IndexHashJoin` オペレータにも 1 つのアウターワーカーと N 個のインナーワーカーが並行して実行されますが、出力順序はアウターテーブルのそれと一致することは保証されません。詳細な実行プロセスは以下のとおりです：

1. アウターワーカーは N 個のアウター行を読み取り、タスクを構築し、その後インナーワーカーチャネルに送信します。
2. インナーワーカーはインナーワーカーチャネルからタスクを受け取り、各タスクに対して順番に以下の 3 つの操作を実行します：
   a. アウター行からハッシュテーブルを構築します。
   b. アウター行からキー範囲を構築し、インナーテーブル行を取得します。
   c. ハッシュテーブルを調査し、結合結果を結果チャネルに送信します。注：a ステップおよび b ステップは並行して実行されます。
3. `IndexHashJoin` のメインスレッドは結果チャネルから結合結果を受け取ります。

`IndexHashJoin` オペレータには以下の実行情報が含まれています：

```sql
inner:{total:4.429220003s, concurrency:5, task:17, construct:96.207725ms, fetch:4.239324006s, build:24.567801ms, join:93.607362ms}
```

- `Inner`: インナーワーカーの実行情報：
    - `total`: インナーワーカーによって費やされた合計時間です。
    - `concurrency`: インナーワーカーの数です。
    - `task`: インナーワーカーが処理したタスクの総数です。
    - `construct`: インナーワーカーがインナーテーブル行を読み取る前の準備に費やされる時間です。
    - `fetch`: インナーワーカーがインナーテーブル行を読み取るのにかかる合計時間です。
    - `Build`: インナーワーカーがアウターテーブル行のハッシュテーブルを構築するのにかかる合計時間です。
    - `join`: インナーワーカーがインナーテーブル行とアウターテーブル行のハッシュテーブルとの結合にかかる合計時間です。

### HashJoin

`HashJoin` オペレータには、インナーワーカー、アウターワーカー、および N 個の結合ワーカーがあります。詳細な実行プロセスは以下のとおりです：

1. インナーワーカーはインナーテーブル行を読み取り、ハッシュテーブルを構築します。
2. アウターワーカーはアウターテーブル行を読み取り、その後タスクにラップし、結合ワーカーに送信します。
3. 結合ワーカーはステップ 1 でのハッシュテーブル構築の完了を待ちます。
4. 結合ワーカーはタスク内のアウターテーブル行およびハッシュテーブルを使用して結合操作を実行し、その後結合結果を結果チャネルに送信します。
5. `HashJoin` のメインスレッドは結果チャネルから結合結果を受け取ります。

`HashJoin` オペレータには以下の実行情報が含まれています：

```
build_hash_table:{total:146.071334ms, fetch:110.338509ms, build:35.732825ms}, probe:{concurrency:5, total:857.162518ms, max:171.48271ms, probe:125.341665ms, fetch:731.820853ms}
```

- `build_hash_table`: インナーテーブルのデータを読み取り、ハッシュテーブルの実行情報を構築します：
    - `total`: 合計時間の消費量です。
    - `fetch`: インナーテーブルデータの読み取りに費やされる合計時間です。
    - `build`: ハッシュテーブルの構築に費やされる合計時間です。
- `probe`: 結合ワーカーの実行情報：
    - `concurrency`: 結合ワーカーの数です。
    - `total`: すべての結合ワーカーによって費やされた合計時間です。
    - `max`: 単一の結合ワーカーが実行するために最も長い時間です。
    - `probe`: アウターテーブル行およびハッシュテーブルとの結合に費やされる合計時間です。
    - `fetch`: 結合ワーカーがアウターテーブル行データを読み取るのを待つ合計時間です。

### TableFullScan (TiFlash)

TiFlash ノードで実行される `TableFullScan` オペレータには以下の実行情報が含まれています：

```sql
tiflash_scan: {
  dtfile: {
    total_scanned_packs: 2, 
    total_skipped_packs: 1, 
    total_scanned_rows: 16000, 
    total_skipped_rows: 8192, 
    total_rough_set_index_load_time: 2ms, 
    total_read_time: 20ms
  }, 
  total_create_snapshot_time: 1ms
}
```

+ `dtfile`: テーブルスキャン中の DTFile（DeltaTree ファイル）に関連する情報で、TiFlash Stable レイヤーのデータスキャンの状態を反映します。
    - `total_scanned_packs`: DTFile でスキャンされたパックの総数です。パックは、TiFlash DTFile で読み取ることができる最小単位です。デフォルトでは、8192 行ごとに 1 つのパックを構成します。
    - `total_skipped_packs`: DTFile でスキャンされたパックの総数です。`WHERE` 句がラフセットインデックスをヒットさせるか、主キーの範囲フィルタリングに一致すると、不要なパックがスキップされます。
    - `total_scanned_rows`: DTFile でスキャンされた行の総数です。MVCC による複数バージョンの更新や削除がある場合、それぞれのバージョンは独立して数えられます。
    - `total_skipped_rows`: DTFile でスキャンされた行の総数です。
    - `total_rs_index_load_time`: DTFile ラフセットインデックスを読み込むのに使われる合計時間です。
    - `total_read_time`: DTFile データを読み取るのに使われる合計時間です。
+ `total_create_snapshot_time`: テーブルスキャン中にスナップショットを作成するのに使われる合計時間です。

### lock_keys 実行情報

悲観的トランザクションで DML ステートメントを実行した場合、オペレータの実行情報には `lock_keys` の実行情報も含まれる場合があります。例：

```
lock_keys: {time:94.096168ms, region:6, keys:8, lock_rpc:274.503214ms, rpc_count:6}
```

- `time`: `lock_keys` オペレーションを実行するための合計時間です。
- `region`: `lock_keys` オペレションを実行する際に関与するリージョンの数です。
- `keys`: `Lock`が必要な`Key`の数。
- `lock_rpc`: TiKVへの`Lock`タイプのRPCリクエストの総時間。複数のRPCリクエストが並行して送信されるため、総RPC時間の消費が`lock_keys`オペレーションの総時間消費よりも大きくなる可能性があります。
- `rpc_count`: TiKVに送信された`Lock`タイプのRPCリクエストの総数。

### commit_txnの実行情報

`autocommit=1`でトランザクション内で書き込み型のDML文が実行される場合、書き込みオペレータの実行情報にはトランザクションのコミットの所要時間も含まれます。例：

```
commit_txn: {prewrite:48.564544ms, wait_prewrite_binlog:47.821579, get_commit_ts:4.277455ms, commit:50.431774ms, region_num:7, write_keys:16, write_byte:536}
```

- `prewrite`: トランザクションの2段階コミットの`prewrite`フェーズで使用される時間。
- `wait_prewrite_binlog:`: `prewrite`バイナリログの書き込みを待機するために使用される時間。
- `get_commit_ts`: トランザクションのコミットタイムスタンプの取得に使用される時間。
- `commit`: トランザクションの2段階コミットの`commit`フェーズで使用される時間。
- `write_keys`: トランザクションで書き込まれた総`keys`数。
- `write_byte`: トランザクションで書き込まれた`key-value`の総バイト数（単位：バイト）。

### RU (Request Unit)の消費

[Request Unit (RU)](/tidb-resource-control.md#what-is-request-unit-ru) は、TiDBリソース制御で定義されたシステムリソースの統一抽象単位です。トップレベルのオペレータの`execution info`には、特定のSQL文の全体的なRU消費が表示されます。

```
RU:273.842670
```

> **注意:**
>
> この値は、この実行によって実際に消費されたRUを示します。同じSQL文が実行されるたびに異なる量のRUが消費される可能性があります。これはキャッシュの影響（たとえば、[coprocessor cache](/coprocessor-cache.md)など）によるものです。

他の値からRUを計算できます。具体的には`EXPLAIN ANALYZE`内の`execution info`列から計算できます。例：

```json
'executeInfo':
   time:2.55ms, 
   loops:2, 
   RU:0.329460, 
   Get:{
       num_rpc:1,
       total_time:2.13ms
   }, 
   total_process_time: 231.5µs,
   total_wait_time: 732.9µs, 
   tikv_wall_time: 995.8µs,
   scan_detail: {
      total_process_keys: 1, 
      total_process_keys_size: 150, 
      total_keys: 1, 
      get_snapshot_time: 691.7µs,
      rocksdb: {
          block: {
              cache_hit_count: 2,
              read_count: 1,
              read_byte: 8.19 KB,
              read_time: 10.3µs
          }
      }
  },
```

基本的なコストは[`tikv/pd`のソースコード](https://github.com/tikv/pd/blob/aeb259335644d65a97285d7e62b38e7e43c6ddca/client/resource_group/controller/config.go#L58C19-L67)で定義され、計算は[`model.go`](https://github.com/tikv/pd/blob/54219d649fb4c8834cd94362a63988f3c074d33e/client/resource_group/controller/model.go#L107)ファイルで実行されます。

TiDB v7.1を使用している場合、計算は`pd/pd-client/model.go`の`BeforeKVRequest()`と`AfterKVRequest()`の合計になります。つまり：

```
キー/値リクエストが処理される前：
      consumption.RRU += float64(kc.ReadBaseCost) -> kv.ReadBaseCost * rpc_nums

キー/値リクエストが処理された後：
      consumption.RRU += float64(kc.ReadBytesCost) * readBytes -> kc.ReadBytesCost * total_process_keys_size
      consumption.RRU += float64(kc.CPUMsCost) * kvCPUMs -> kc.CPUMsCost * total_process_time
```

書き込みとバッチ取得の場合、基本コストが異なるが、計算は同様です。

### その他の一般的な実行情報

Coprocessorオペレータには通常、`cop_task`と`tikv_task`の2つの実行時間情報が含まれています。`cop_task`はTiDBが記録した時間であり、リクエストをサーバーに送信してからレスポンスが受信されるまでの時間です。`tikv_task`はTiKV Coprocessor自体が記録した時間です。両者の間に大きな違いがある場合、応答を待機する時間が長すぎるか、gRPCやネットワークに費やす時間が長すぎることを示す可能性があります。

## MySQL互換性

`EXPLAIN ANALYZE`はMySQL 8.0の機能ですが、TiDBの出力形式と潜在的な実行計画はMySQLと大きく異なります。

## 関連情報

* [クエリ実行計画の理解](/explain-overview.md)
* [EXPLAIN](/sql-statements/sql-statement-explain.md)
* [ANALYZE TABLE](/sql-statements/sql-statement-analyze-table.md)
* [TRACE](/sql-statements/sql-statement-trace.md)