---
title: レイテンシーの分解
summary: TiDBのレイテンシーの詳細と、実際の使用ケースでのレイテンシーの分析方法の紹介
---

# レイテンシーの分解

このドキュメントでは、レイテンシーをメトリクスに分解し、次の観点からユーザーの視点で分析します:

- [一般的なSQLレイヤー](#一般的なSQLレイヤー)
- [読み取りクエリ](#読み取りクエリ)
- [書き込みクエリ](#書き込みクエリ)
- [バッチクライアント](#バッチクライアント)
- [TiKVスナップショット](#TiKVスナップショット)
- [非同期書き込み](#非同期書き込み)

これらの分析により、TiDB SQLクエリ中の時間コストについて深い洞察が得られます。これはTiDBのクリティカルパスの診断のガイドです。また、[診断の使用ケース](#診断の使用ケース)セクションでは、実際の使用ケースでのレイテンシーの分析方法が紹介されています。

このドキュメントを読む前に、[パフォーマンス解析とチューニング](/performance-tuning-methods.md)を読むことをお勧めします。レイテンシーをメトリクスに分解する際には、特定の遅いクエリではなく、期間やレイテンシーの平均値が計算されます。多くのメトリクスはヒストグラムのように表示され、これは期間やレイテンシーの分布です。平均レイテンシーを計算するには、以下のsumとcountのカウンターを使用する必要があります。

```
avg = ${metric_name}_sum / ${metric_name}_count
```

このドキュメントで説明されているメトリクスは、TiDBのPrometheusダッシュボードから直接読み取ることができます。

## 一般的なSQLレイヤー

この一般的なSQLレイヤーのレイテンシーは、TiDBの最上位レベルに存在し、すべてのSQLクエリで共有されます。次の図は、一般的なSQLレイヤー操作の時間コスト図です:

```railroad+diagram
Diagram(
    NonTerminal("Token wait duration"),
    Choice(
        0,
        Comment("Prepared statement"),
        NonTerminal("Parse duration"),
    ),
    OneOrMore(
        Sequence(
        Choice(
            0,
            NonTerminal("Optimize prepared plan duration"),
            Sequence(
            Comment("Plan cache miss"),
            NonTerminal("Compile duration"),
            ),
        ),
        NonTerminal("TSO wait duration"),
        NonTerminal("Execution duration"),
        ),
        Comment("Retry"),
    ),
)
```

一般的なSQLレイヤーのレイテンシーは、`e2e duration`メトリクスとして観測でき、以下のように計算されます:

```text
e2e duration =
    tidb_server_get_token_duration_seconds +
    tidb_session_parse_duration_seconds +
    tidb_session_compile_duration_seconds +
    tidb_session_execute_duration_seconds{type="general"}
```

- `tidb_server_get_token_duration_seconds` は、Tokenの待ち時間の期間を記録します。通常は1ミリ秒未満であり、無視できるほど小さな値です。
- `tidb_session_parse_duration_seconds` は、SQLクエリを抽象構文木（AST）にパースする期間を記録します。これは[`PREPARE/EXECUTE`ステートメント](/develop/dev-guide-optimize-sql-best-practices.md#use-prepare)でスキップできます。
- `tidb_session_compile_duration_seconds` は、ASTを実行計画にコンパイルする期間を記録します。これは[SQLプリペアード実行計画キャッシュ](/sql-prepared-plan-cache.md)でスキップできます。
- `tidb_session_execute_duration_seconds{type="general"}` は実行の期間を記録し、すべての種類のユーザークエリを混合しています。これは、パフォーマンスの問題やボトルネックを分析するために細かい期間に分割する必要があります。

一般的に、オンライントランザクション処理（OLTP）のワークロードは、読み取りクエリと書き込みクエリに分けることができます。これらはいくつかの重要なコードを共有しています。次のセクションでは、異なる方法で実行される[読み取りクエリ](#読み取りクエリ)と[書き込みクエリ](#書き込みクエリ)のレイテンシーについて説明します。

## 読み取りクエリ

読み取りクエリは、単一のプロセス形式のみを持ちます。

### ポイント取得

以下は、[ポイント取得](/glossary.md#point-get)操作の時間コスト図です:

```railroad+diagram
Diagram(
    Choice(
        0,
        NonTerminal("Resolve TSO"),
        Comment("Read by clustered PK in auto-commit-txn mode or snapshot read"),
    ),
    Choice(
        0,
        NonTerminal("Read handle by index key"),
        Comment("Read by clustered PK, encode handle by key"),
    ),
    NonTerminal("Read value by handle"),
)
```

ポイント取得の際、`tidb_session_execute_duration_seconds{type="general"}` の期間は以下のように計算されます:

```text
tidb_session_execute_duration_seconds{type="general"} =
    pd_client_cmd_handle_cmds_duration_seconds{type="wait"} +
    read handle duration +
    read value duration
```

`pd_client_cmd_handle_cmds_duration_seconds{type="wait"}` は、PDから[TSO（タイムスタンプオラクル）](/glossary.md#tso)を取得する期間を記録します。クラスタードプライマリインデックスでのオートコミットトランザクションモードまたはスナップショット読み取り時、その値はゼロになります。

`read handle duration` と `read value duration` は以下のように計算されます:

```text
read handle duration = read value duration =
    tidb_tikvclient_txn_cmd_duration_seconds{type="get"} =
    send request duration =
    tidb_tikvclient_request_seconds{type="Get"} =
    tidb_tikvclient_batch_wait_duration +
    tidb_tikvclient_batch_send_latency +
    tikv_grpc_msg_duration_seconds{type="kv_get"} +
    tidb_tikvclient_rpc_net_latency_seconds{store="?"}
```

`tidb_tikvclient_request_seconds{type="Get"}` は、バッチ処理されたgRPCラッパーを介してTiKVに直接送信される取得リクエストの期間を記録します。`tidb_tikvclient_batch_wait_duration`、`tidb_tikvclient_batch_send_latency`、`tidb_tikvclient_rpc_net_latency_seconds{store="?"}`など、前述のバッチクライアントの期間の詳細については、[バッチクライアント](#バッチクライアント)セクションを参照してください。

`tikv_grpc_msg_duration_seconds{type="kv_get"}` 期間は以下のように計算されます:

```text
tikv_grpc_msg_duration_seconds{type="kv_get"} =
    tikv_storage_engine_async_request_duration_seconds{type="snapshot"} +
    tikv_engine_seek_micro_seconds{type="seek_average"} +
    read value duration +
    read value duration(non-short value)
```

この時点で、リクエストはTiKV内で処理されます。TiKVは、一度のシークと一度または二度のリードアクション（短い値は書き込みカラムファミリにエンコードされ、一度読めば十分です）でゲットリクエストを処理します。リクエストの前にTiKVはスナップショットを取得します。TiKVスナップショットの期間の詳細については、[TiKVスナップショット](#tikvスナップショット)セクションを参照してください。

`read value duration（ディスクから）` は以下のように計算されます:

```text
read value duration(from disk) =
    sum(rate(tikv_storage_rocksdb_perf{metric="block_read_time",req="get/batch_get_command"})) / sum(rate(tikv_storage_rocksdb_perf{metric="block_read_count",req="get/batch_get_command"}))
```

TiKVはそのストレージエンジンとしてRocksDBを使用しています。必要な値がブロックキャッシュにない場合、TiKVはディスクから値をロードする必要があります。`tikv_storage_rocksdb_perf`に対して、getリクエストは`get`または`batch_get_command`のいずれかであることができます。

### バッチポイント取得

以下は、バッチポイント取得操作の時間コスト図です:

```railroad+diagram
Diagram(
  NonTerminal("Resolve TSO"),
  Choice(
    0,
    NonTerminal("Read all handles by index keys"),
    Comment("Read by clustered PK, encode handle by keys"),
  ),
  NonTerminal("Read values by handles"),
)
```

バッチポイント取得中、`tidb_session_execute_duration_seconds{type="general"}` は以下のように計算されます:

```text
tidb_session_execute_duration_seconds{type="general"} =
    pd_client_cmd_handle_cmds_duration_seconds{type="wait"} +
    read handles duration +
    read values duration
```

バッチポイント取得のプロセスは、ポイント取得とほぼ同じですが、バッチポイント取得では複数の値を同時に読み取ります。

`read handles duration` と `read values duration` は以下のように計算されます:

```text
read handles duration = read values duration =
    tidb_tikvclient_txn_cmd_duration_seconds{type="batch_get"} =
    send request duration =
    tidb_tikvclient_request_seconds{type="BatchGet"} =
    tidb_tikvclient_batch_wait_duration(transaction) +
    tidb_tikvclient_batch_send_latency(transaction) +
    tikv_grpc_msg_duration_seconds{type="kv_batch_get"} +
    tidb_tikvclient_rpc_net_latency_seconds{store="?"}(transaction)
```

`tikv_grpc_msg_duration_seconds{type="kv_batch_get"}` 期間は以下のように計算されます:

```text
tikv_grpc_msg_duration_seconds{type="kv_batch_get"} =
    tikv_storage_engine_async_request_duration_seconds{type="snapshot"} +
    n * (
        tikv_engine_seek_micro_seconds{type="seek_max"} +
        read value duration +
        read value duration(non-short value)
    )

read value duration(from disk) =
```
    sum(rate(tikv_storage_rocksdb_perf{metric="block_read_time",req="batch_get"})) / sum(rate(tikv_storage_rocksdb_perf{metric="block_read_count",req="batch_get"}))
```

スナップショットを取得した後、TiKVは同じスナップショットから複数の値を読み取ります。読み取りの期間は[Point get](#point-get)と同じです。TiKVがディスクからデータを読み込むとき、平均期間は`req="batch_get"`を持つ`tikv_storage_rocksdb_perf`によって計算できます。

### テーブルスキャンとインデックススキャン

次の図は、テーブルスキャンとインデックススキャン操作の時間コスト図です：

```railroad+diagram
Diagram(
    Stack(
        NonTerminal("Resolve TSO"),
        NonTerminal("Load region cache for related table/index ranges"),
        OneOrMore(
            NonTerminal("Wait for result"),
            Comment("Next loop: drain the result"),
        ),
    ),
)
```

テーブルスキャンとインデックススキャン中、`tidb_session_execute_duration_seconds{type="general"}`の期間は次のように計算されます:

```text
tidb_session_execute_duration_seconds{type="general"} =
    pd_client_cmd_handle_cmds_duration_seconds{type="wait"} +
    req_per_copr * (
        tidb_distsql_handle_query_duration_seconds{sql_type="general"}
    )
    tidb_distsql_handle_query_duration_seconds{sql_type="general"} <= send request duration
```

テーブルスキャンとインデックススキャンは同じ方法で処理されます。`req_per_copr`は分散タスク数です。コプロセッサ実行とクライアントへのデータ応答は異なるスレッドで行われるため、`tidb_distsql_handle_query_duration_seconds{sql_type="general"}`は待機時間であり、`send request duration`よりも短い時間です。

`send request duration`と`req_per_copr`は次のように計算されます:

```text
send request duration =
    tidb_tikvclient_batch_wait_duration +
    tidb_tikvclient_batch_send_latency +
    tikv_grpc_msg_duration_seconds{type="coprocessor"} +
    tidb_tikvclient_rpc_net_latency_seconds{store="?"}

tikv_grpc_msg_duration_seconds{type="coprocessor"} =
    tikv_coprocessor_request_wait_seconds{type="snapshot"} +
    tikv_coprocessor_request_wait_seconds{type="schedule"} +
    tikv_coprocessor_request_handler_build_seconds{type="index/select"} +
    tikv_coprocessor_request_handle_seconds{type="index/select"}

req_per_copr = rate(tidb_distsql_handle_query_duration_seconds_count) / rate(tidb_distsql_scan_keys_partial_num_count)
```

TiKVでは、テーブルスキャンのタイプは`select`であり、インデックススキャンのタイプは`index`です。`select`と`index`のタイプ期間の詳細は同じです。

### インデックスルックアップ

次の図は、インデックスルックアップ操作の時間コスト図です：

```railroad+diagram
Diagram(
    Stack(
        NonTerminal("Resolve TSO"),
        NonTerminal("Load region cache for related index ranges"),
        OneOrMore(
            Sequence(
                NonTerminal("Wait for index scan result"),
                NonTerminal("Wait for table scan result"),
            ),
        Comment("Next loop: drain the result"),
        ),
    ),
)
```

インデックスルックアップ中、`tidb_session_execute_duration_seconds{type="general"}`の期間は次のように計算されます:

```text
tidb_session_execute_duration_seconds{type="general"} =
    pd_client_cmd_handle_cmds_duration_seconds{type="wait"} +
    req_per_copr * (
        tidb_distsql_handle_query_duration_seconds{sql_type="general"}
    ) +
    req_per_copr * (
        tidb_distsql_handle_query_duration_seconds{sql_type="general"}
    )

req_per_copr = rate(tidb_distsql_handle_query_duration_seconds_count) / rate(tidb_distsql_scan_keys_partial_num_count)
```

インデックスルックアップは、パイプラインで処理されるインデックススキャンとテーブルスキャンを組み合わせます。

## 書き込みクエリ

書き込みクエリは読み取りクエリよりも複雑です。書き込みクエリにはいくつかのバリエーションがあります。次の図は、書き込みクエリ操作の時間コスト図です：

```railroad+diagram
Diagram(
    NonTerminal("Execute write query"),
    Choice(
        0,
        NonTerminal("Pessimistic lock keys"),
        Comment("bypass in optimistic transaction"),
    ),
    Choice(
        0,
        NonTerminal("Auto Commit Transaction"),
        Comment("bypass in non-auto-commit or explicit transaction"),
    ),
)
```

|                 | ペシミスティックトランザクション | オプティミスティックトランザクション |
|-----------------|-------------------------|------------------------|
| オートコミット     | execute + lock + commit | execute + commit       |
| オートコミットでない | execute + lock          | execute                |

書き込みクエリは次の3つのフェーズに分割されます:

- executeフェーズ: TiDBのメモリに操作を実行し、変更を書き込む。
- lockフェーズ: 実行結果のためにペシミスティックロックを獲得する。
- commitフェーズ: 2フェーズコミットプロトコル(2PC)を通じてトランザクションをコミットする。

executeフェーズでは、TiDBはメモリ内のデータを操作し、必要なデータを読み取ることから主な遅延が発生します。更新および削除クエリの場合、TiDBは最初にTiKVからデータを読み取り、次にメモリ内の行を更新または削除します。

例外は、ポイントゲットとバッチポイントゲットを使用するロック時間の読み取り操作(`SELECT FOR UPDATE`)で、これらは1回のリモートプロシージャコール(RPC)での読み取りとロックを行います。

### Lock-time point get

次の図は、ロック時間ポイントゲット操作の時間コスト図です：

```railroad+diagram
Diagram(
    Choice(
        0,
        Sequence(
            NonTerminal("Read handle key by index key"),
            NonTerminal("Lock index key"),
        ),
        Comment("Clustered index"),
    ),
    NonTerminal("Lock handle key"),
    NonTerminal("Read value from pessimistic lock cache"),
)
```

ロック時間ポイントゲット中、`execution(clustered PK)`と`execution(non-clustered PK or UK)`の期間は次のように計算されます:

```text
execution(clustered PK) =
    tidb_tikvclient_txn_cmd_duration_seconds{type="lock_keys"}
execution(non-clustered PK or UK) =
    2 * tidb_tikvclient_txn_cmd_duration_seconds{type="lock_keys"}
```

ロック時間ポイントゲットは、キーをロックし、その値を返します。実行後のロックフェーズと比較して、1ラウンドトリップを節約します。ロック時間ポイントゲットの期間は、[Lock duration](#lock)と同等に扱うことができます。

### Lock-time batch point get

次の図は、ロック時間バッチポイントゲット操作の時間コスト図です：

```railroad+diagram
Diagram(
    Choice(
        0,
        NonTerminal("Read handle keys by index keys"),
        Comment("Clustered index"),
    ),
    NonTerminal("Lock index and handle keys"),
    NonTerminal("Read values from pessimistic lock cache"),
)
```

ロック時間バッチポイントゲット中、`execution(clustered PK)`および`execution(non-clustered PK or UK)`の期間は次のように計算されます:

```text
execution(clustered PK) =
    tidb_tikvclient_txn_cmd_duration_seconds{type="lock_keys"}
execution(non-clustered PK or UK) =
    tidb_tikvclient_txn_cmd_duration_seconds{type="batch_get"} +
    tidb_tikvclient_txn_cmd_duration_seconds{type="lock_keys"}
```

ロック時間バッチポイントゲットの実行は、[Lock-time point get](#lock-time-point-get)と類似しており、複数の値を1回のRPCで読み取ります。`tidb_tikvclient_txn_cmd_duration_seconds{type="batch_get"}`の期間の詳細については、[Batch point get](#batch-point-get)のセクションを参照してください。

### ロック

このセクションでは、ロック期間について説明します。

```text
round = ceil(
    sum(rate(tidb_tikvclient_txn_regions_num_sum{type="2pc_pessimistic_lock"})) /
    sum(rate(tidb_tikvclient_txn_regions_num_count{type="2pc_pessimistic_lock"})) /
    committer-concurrency
)

lock = tidb_tikvclient_txn_cmd_duration_seconds{type="lock_keys"} =
    round * tidb_tikvclient_request_seconds{type="PessimisticLock"}
```

ロックは2PC構造を通じて取得されます。2PCはフロー制御メカニズムを持っており、`committer-concurrency`（デフォルト値は`128`）によって同時に実行中のリクエストが制限されます。単純化すると、フロー制御はリクエストの待機時間（`round`）を増加させると考えることができます。

`tidb_tikvclient_request_seconds{type="PessimisticLock"}`は次のように計算されます:

```text
tidb_tikvclient_request_seconds{type="PessimisticLock"} =
    tidb_tikvclient_batch_wait_duration +
    tidb_tikvclient_batch_send_latency +
    tikv_grpc_msg_duration_seconds{type="kv_pessimistic_lock"} +
    tidb_tikvclient_rpc_net_latency_seconds{store="?"}
```

`tikv_grpc_msg_duration_seconds{type="kv_pessimistic_lock"}`、`tidb_tikvclient_batch_wait_duration`、`tidb_tikvclient_batch_send_latency`、`tidb_tikvclient_rpc_net_latency_seconds{store="?"}`などの前述のバッチクライアント期間の詳細については、[Batch client](#batch-client)のセクションを参照してください。
```
`tikv_grpc_msg_duration_seconds{type="kv_pessimistic_lock"}`の期間は次のように計算されます。

```text
tikv_grpc_msg_duration_seconds{type="kv_pessimistic_lock"} =
    tikv_scheduler_latch_wait_duration_seconds{type="acquire_pessimistic_lock"} +
    tikv_storage_engine_async_request_duration_seconds{type="snapshot"} +
    (lock in-mem key count + lock on-disk key count) * lock read duration +
    lock on-disk key count / (lock in-mem key count + lock on-disk key count) *
    lock write duration
```

- TiDB v6.0以降、TiKVは[デフォルトでインメモリの悲観的ロック](/pessimistic-transaction.md#in-memory-pessimistic-lock)を使用します。インメモリの悲観的ロックは非同期書き込みプロセスをバイパスします。
- `tikv_storage_engine_async_request_duration_seconds{type="snapshot"}`はスナップショットタイプの期間です。詳細については[TiKVスナップショット](#tikv-snapshot)セクションを参照してください。
- `lock in-mem key count`および`lock on-disk key count`は次のように計算されます:

    ```text
    lock in-mem key count =
        sum(rate(tikv_in_memory_pessimistic_locking{result="success"})) /
        sum(rate(tikv_grpc_msg_duration_seconds_count{type="kv_pessimistic_lock"}))

    lock on-disk key count =
        sum(rate(tikv_in_memory_pessimistic_locking{result="full"})) /
        sum(rate(tikv_grpc_msg_duration_seconds_count{type="kv_pessimistic_lock"}))
    ```

    インメモリおよびオンディスクのロックキーの数は、インメモリロックのカウンターによって計算できます。 TiKVはロックを取得する前にキーの値を読み込み、読み込み期間はRocksDBのパフォーマンスコンテキストによって計算できます。

    ```text
    ロックの読み取り期間（ディスクから） =
        sum(rate(tikv_storage_rocksdb_perf{metric="block_read_time",req="acquire_pessimistic_lock"})) / sum(rate(tikv_storage_rocksdb_perf{metric="block_read_count",req="acquire_pessimistic_lock"}))
    ```

- `lock write duration`はディスク上のロックの書き込み期間です。詳細については、[非同期書き込み](#async-write)セクションを参照してください。

### コミット

このセクションでは、コミット期間について説明します。 以下は、コミット操作の時間コスト図です：

```railroad+diagram
Diagram(
    Stack(
        Sequence(
            Choice(
                0,
                Comment("2pcまたは因果一貫性を使用する"),
                NonTerminal("Get min-commit-ts"),
            ),
            Optional("非同期プリライトbinlog"),
            NonTerminal("Prewrite mutations"),
            Optional("プリライトbinlogの結果を待つ"),
        ),
        Sequence(
            Choice(
                1,
                Comment("1pc"),
                Sequence(
                    Comment("2pc"),
                    NonTerminal("Get commit-ts"),
                    NonTerminal("Check schema"),
                    NonTerminal("Commit PK mutation"),
                ),
                Sequence(
                    Comment("非同期コミット"),
                    NonTerminal("Commit mutations asynchronously"),
                ),
            ),
            Choice(
                0,
                Comment("コミット済み"),
                NonTerminal("非同期クリーンアップ"),
            ),
            Optional("コミットbinlog"),
        ),
    ),
)
```

コミット段階の期間は次のように計算されます：

```text
commit =
    Get_latest_ts_time +
    Prewrite_time +
    Get_commit_ts_time +
    Commit_time

Get_latest_ts_time = Get_commit_ts_time =
    pd_client_cmd_handle_cmds_duration_seconds{type="wait"}

prewrite_round = ceil(
    sum(rate(tidb_tikvclient_txn_regions_num_sum{type="2pc_prewrite"})) /
    sum(rate(tidb_tikvclient_txn_regions_num_count{type="2pc_prewrite"})) /
    committer-concurrency
)

commit_round = ceil(
    sum(rate(tidb_tikvclient_txn_regions_num_sum{type="2pc_commit"})) /
    sum(rate(tidb_tikvclient_txn_regions_num_count{type="2pc_commit"})) /
    committer-concurrency
)

Prewrite_time =
    prewrite_round * tidb_tikvclient_request_seconds{type="Prewrite"}

Commit_time =
    commit_round * tidb_tikvclient_request_seconds{type="Commit"}
```

コミット期間は4つのメトリックに分解されます。

- `Get_latest_ts_time`は、非同期コミットまたは単一段階コミット（1PC）トランザクションで最新のTSOを取得する期間を記録します。
- `Prewrite_time`は、プリライト段階の期間を記録します。
- `Get_commit_ts_time`は、通常の2PCトランザクションの期間を記録します。
- `Commit_time`は、コミット段階の期間を記録します。注：非同期コミットまたは1PCトランザクションにはこの段階はありません。

悲観的ロックと同様に、フローコントロールは遅延の増幅（前述の式の`prewrite_round`と`commit_round`）として機能します。

`tidb_tikvclient_request_seconds{type="Prewrite"}`および`tidb_tikvclient_request_seconds{type="Commit"}`の期間は次のように計算されます：

```text
tidb_tikvclient_request_seconds{type="Prewrite"} =
    tidb_tikvclient_batch_wait_duration +
    tidb_tikvclient_batch_send_latency +
    tikv_grpc_msg_duration_seconds{type="kv_prewrite"} +
    tidb_tikvclient_rpc_net_latency_seconds{store="?"}

tidb_tikvclient_request_seconds{type="Commit"} =
    tidb_tikvclient_batch_wait_duration +
    tidb_tikvclient_batch_send_latency +
    tikv_grpc_msg_duration_seconds{type="kv_commit"} +
    tidb_tikvclient_rpc_net_latency_seconds{store="?"}
```

前述のバッチクライアントの期間についての詳細については、`tidb_tikvclient_batch_wait_duration`、`tidb_tikvclient_batch_send_latency`、および`tidb_tikvclient_rpc_net_latency_seconds{store="?"}`など、[バッチクライアント](#batch-client)セクションを参照してください。

`tikv_grpc_msg_duration_seconds{type="kv_prewrite"}`の期間は次のように計算されます：

```text
tikv_grpc_msg_duration_seconds{type="kv_prewrite"} =
    prewrite key count * prewrite read duration +
    prewrite write duration

prewrite key count =
    sum(rate(tikv_scheduler_kv_command_key_write_sum{type="prewrite"})) /
    sum(rate(tikv_scheduler_kv_command_key_write_count{type="prewrite"}))

prewrite read duration(from disk) =
    sum(rate(tikv_storage_rocksdb_perf{metric="block_read_time",req="prewrite"})) / sum(rate(tikv_storage_rocksdb_perf{metric="block_read_count",req="prewrite"}))
```

TiKVのロックと同様に、プリライトは読み取りおよび書き込みフェーズで処理されます。 読み取り期間はRocksDBのパフォーマンスコンテキストから計算できます。 書き込み期間についての詳細については、[非同期書き込み](#async-write)セクションを参照してください。

`tikv_grpc_msg_duration_seconds{type="kv_commit"}`の期間は次のように計算されます：

```text
tikv_grpc_msg_duration_seconds{type="kv_commit"} =
    commit key count * commit read duration +
    commit write duration

commit key count =
    sum(rate(tikv_scheduler_kv_command_key_write_sum{type="commit"})) /
    sum(rate(tikv_scheduler_kv_command_key_write_count{type="commit"}))

commit read duration(from disk) =
    sum(rate(tikv_storage_rocksdb_perf{metric="block_read_time",req="commit"})) / sum(rate(tikv_storage_rocksdb_perf{metric="block_read_count",req="commit"})) (storage)
```

`kv_commit`の期間はほぼ`kv_prewrite`と同様です。 書き込み期間についての詳細については、[非同期書き込み](#async-write)セクションを参照してください。

## バッチクライアント

以下は、バッチクライアントの時間コスト図です：

```railroad+diagram
Diagram(
    NonTerminal("Get conn pool to the target store"),
    Choice(
        0,
        Sequence(
            Comment("バッチ有効"),
                NonTerminal("Push request to channel"),
                NonTerminal("Wait response"),
            ),
            Sequence(
            NonTerminal("Get conn from pool"),
            NonTerminal("Call RPC"),
            Choice(
                0,
                Comment("Unary call"),
                NonTerminal("Recv first"),
            ),
        ),
    ),
)
```

- リクエストの送信全体の期間は、`tidb_tikvclient_request_seconds`として観察されます。
- RPCクライアントは、各ストアへの接続プール（ConnArrayという名前）を維持し、各プールにはバッチリクエスト（送信）チャネルを持つBatchConnがあります。
- ストアがTiKVであり、バッチサイズが正である場合、バッチは有効になります。ほとんどの場合これは当てはまります。
- バッチリクエストチャネルのサイズは[`tikv-client.max-batch-size`](/tidb-configuration-file.md#max-batch-size)（デフォルトは`128`）で、エンキューの期間は`tidb_tikvclient_batch_wait_duration`として観察されます。
- 3種類のストリームリクエストがあります：`CmdBatchCop`、`CmdCopStream`、`CmdMPPConn`は、ストリームから最初の応答を取得するための追加の`recv()`呼び出しを含みます。

まだいくつかの遅延が観察されていますが、`tidb_tikvclient_request_seconds` は以下のようにおおよその計算が可能です:

```text
tidb_tikvclient_request_seconds{type="?"} =
    tidb_tikvclient_batch_wait_duration +
    tidb_tikvclient_batch_send_latency +
    tikv_grpc_msg_duration_seconds{type="kv_?"} +
    tidb_tikvclient_rpc_net_latency_seconds{store="?"}
```

- `tidb_tikvclient_batch_wait_duration` はバッチシステムでの待機時間を記録します。
- `tidb_tikvclient_batch_send_latency` はバッチシステムでのエンコード時間を記録します。
- `tikv_grpc_msg_duration_seconds{type="kv_?"}` は TiKV の処理時間です。
- `tidb_tikvclient_rpc_net_latency_seconds` はネットワークの待機時間を記録します。

## TiKV スナップショット

以下は TiKV スナップショット操作の時間コストダイアグラムです:

```railroad+diagram
Diagram(
    Choice(
        0,
        Comment("ローカル読み取り"),
        Sequence(
            NonTerminal("Propose Wait"),
            NonTerminal("Read index Read Wait"),
        ),
    ),
    NonTerminal("Fetch A Snapshot From KV Engine"),
)
```

TiKV スナップショットの総時間は `tikv_storage_engine_async_request_duration_seconds{type="snapshot"}` として観測され、以下のように計算されます:

```text
tikv_storage_engine_async_request_duration_seconds{type="snapshot"} =
    tikv_coprocessor_request_wait_seconds{type="snapshot"} =
    tikv_raftstore_request_wait_time_duration_secs +
    tikv_raftstore_commit_log_duration_seconds +
    rocksdb からのスナップショットの取得時間
```

リーダーリースが期限切れになると、TiKV は RocksDB からスナップショットを取得する前に読み込みインデックスコマンドを提案します。 `tikv_raftstore_request_wait_time_duration_secs` および `tikv_raftstore_commit_log_duration_seco` は読み込みインデックスコマンドの実行時間です。

通常、RocksDB からスナップショットを取得する操作は高速ですので、 `rocksdb からのスナップショットの取得時間` は無視されます。

## 非同期書き込み

非同期書き込みは、TiKV がコールバックを使用して Raft ベースの複製状態機械にデータを非同期に書き込むプロセスです。

- 非同期 I/O が無効になっている場合の非同期書き込み操作の時間コストダイアグラムは以下の通りです:

    ```railroad+diagram
    Diagram(
        NonTerminal("提案待機"),
        NonTerminal("コマンド処理"),
        Choice(
            0,
            Sequence(
                NonTerminal("現在のバッチ待機"),
                NonTerminal("ログエンジンに書き込む"),
            ),
            Sequence(
                NonTerminal("RaftMsg 送信待機"),
                NonTerminal("コミットログ待機"),
            ),
        ),
        NonTerminal("適用待機"),
        NonTerminal("適用ログ"),
    )
    ```

- 非同期 I/O が有効になっている場合の非同期書き込み操作の時間コストダイアグラムは以下の通りです:

    ```railroad+diagram
    Diagram(
        NonTerminal("提案待機"),
        NonTerminal("コマンド処理"),
        Choice(
            0,
            NonTerminal("書き込みワーカーによって永続化されるまで待機"),
            Sequence(
                NonTerminal("RaftMsg 送信待機"),
                NonTerminal("コミットログ待機"),
            ),
        ),
        NonTerminal("適用待機"),
        NonTerminal("適用ログ"),
    )
    ```

非同期書き込みの所要時間は以下のように計算されます:

```text
async write duration(async io disabled) =
    提案 +
    async io disabled commit +
    tikv_raftstore_apply_wait_time_duration_secs +
    tikv_raftstore_apply_log_duration_seconds

async write duration(async io enabled) =
    提案 +
    async io enabled commit +
    tikv_raftstore_apply_wait_time_duration_secs +
    tikv_raftstore_apply_log_duration_seconds
```

非同期書き込みは以下の3つのフェーズに分解できます:

- 提案
- コミット
- 適用: 前述の式で `tikv_raftstore_apply_wait_time_duration_secs + tikv_raftstore_apply_log_duration_seconds` 

提案フェーズの所要時間は以下のように計算されます:

```text
提案 =
    提案待機時間 +
    提案時間

提案待機時間 =
    tikv_raftstore_store_wf_batch_wait_duration_seconds

提案時間 =
    tikv_raftstore_store_wf_send_to_queue_duration_seconds -
    tikv_raftstore_store_wf_batch_wait_duration_seconds
```

Raft プロセスはウォーターフォールの方法で記録されます。そのため、提案の所要時間は、2つのメトリクスの差から計算されます。

コミットフェーズの所要時間は以下のように計算されます:

```text
async io disabled commit = max(
    その場でログを永続化する時間,
    ログを複製する時間
)

async io enabled commit = max(
    書き込みワーカーによる待機時間,
    ログの複製時間
)
```

v5.3.0 以降、TiKV は Async IO Raft（Raft ログを StoreWriter スレッドプールで書き込む）をサポートしています。 Async IO Raft は [`store-io-pool-size`](/tikv-configuration-file.md#store-io-pool-size-new-in-v530) が正の値に設定されている場合のみ有効になり、コミットプロセスが変更されます。 `その場でログを永続化する時間` および `書き込みワーカーによる待機時間` は、以下のように計算されます:

```text
その場でログを永続化する時間 =
    バッチ待機時間 +
    raft db への書き込み時間

バッチ待機時間 =
    tikv_raftstore_store_wf_before_write_duration_seconds -
    tikv_raftstore_store_wf_send_to_queue_duration_seconds

raft db への書き込み時間 =
    tikv_raftstore_store_wf_write_end_duration_seconds -
    tikv_raftstore_store_wf_before_write_duration_seconds

書き込みワーカーによる待機時間 =
    tikv_raftstore_store_wf_persist_duration_seconds -
    tikv_raftstore_store_wf_send_to_queue_duration_seconds
```

Async IO がある場合、その場でログを永続化する時間の所要時間は、ウォーターフォールメトリクスから直接計算できます（バッチ待機時間をスキップ）。

ログを複製する時間は、クォーラムピアに永続化したログの所要時間を記録しており、RPC の所要時間と大多数でのログの永続化時間が含まれています。 `ログを複製する時間` は以下のように計算されます:

```text
ログを複製する時間 =
    raftmsg 送信待機時間 +
    コミットログ待機時間

raftmsg 送信待機時間 =
    tikv_raftstore_store_wf_send_proposal_duration_seconds -
    tikv_raftstore_store_wf_send_to_queue_duration_seconds

コミットログ待機時間 =
    tikv_raftstore_store_wf_commit_log_duration -
    tikv_raftstore_store_wf_send_proposal_duration_seconds
```

### Raft DB

以下は Raft DB 操作の時間コストダイアグラムです:

```railroad+diagram
Diagram(
    NonTerminal("Writer リーダー待ち"),
    NonTerminal("ログの書き込みと同期"),
    NonTerminal("Memtable へのログの適用"),
)
```

```text
raft db への書き込み時間 = raft db 書き込み時間
コミットログ待機時間 >= raft db 書き込み時間

raft db 書き込み時間(raft engine が有効の場合) =
    raft_engine_write_preprocess_duration_seconds +
    raft_engine_write_leader_duration_seconds +
    raft_engine_write_apply_duration_seconds

raft db 書き込み時間(raft engine が無効の場合) =
    tikv_raftstore_store_perf_context_time_duration_secs{type="write_thread_wait"} +
    tikv_raftstore_store_perf_context_time_duration_secs{type="write_scheduling_flushes_compactions_time"} +
    tikv_raftstore_store_perf_context_time_duration_secs{type="write_wal_time"} +
    tikv_raftstore_store_perf_context_time_duration_secs{type="write_memtable_time"}
```

`コミットログ待機時間` はクォーラムピアの最大の所要時間ですので、 `raft db 書き込み時間` よりも大きい場合があります。

v6.1.0 以降、TiKV はデフォルトのログストレージエンジンとして [Raft Engine](/glossary.md#raft-engine) を使用するため、ログの書き込みプロセスが変更されます。

### KV DB

以下は KV DB 操作の時間コストダイアグラムです:

```railroad+diagram
Diagram(
    NonTerminal("Writer リーダー待ち"),
    NonTerminal("前処理"),
    Choice(
        0,
        Comment("切替不要"),
        NonTerminal("WAL または Memtable の切替"),
    ),
    NonTerminal("ログの書き込みと同期"),
    NonTerminal("Memtable への適用"),
)
```

```text
tikv_raftstore_apply_log_duration_seconds =
    tikv_raftstore_apply_perf_context_time_duration_secs{type="write_thread_wait"} +
    tikv_raftstore_apply_perf_context_time_duration_secs{type="write_scheduling_flushes_compactions_time"} +
    tikv_raftstore_apply_perf_context_time_duration_secs{type="write_wal_time"} +
    tikv_raftstore_apply_perf_context_time_duration_secs{type="write_memtable_time"}
```

非同期書き込みプロセスでは、コミットされたログは KV DB に適用する必要があります。適用にかかる時間は RocksDB のパフォーマンスコンテキストから計算できます。

## 診断ユースケース

前述のセクションでは、クエリ中の時間コストメトリクスの詳細について説明しました。このセクションでは、遅い読み取りまたは書き込みクエリに遭遇した際のメトリクス解析の一般的な手順を紹介します。すべてのメトリクスは、[パフォーマンス概要ダッシュボード](/grafana-performance-overview-dashboard.md)の Database Time パネルで確認できます。

### 遅い読み取りクエリ

`SELECT` ステートメントがデータベースの時間の大部分を占める場合、TiDB が読み取りクエリに遅い可能性があります。
```markdown
スローリードクエリの実行計画は、TiDBダッシュボードの[トップSQLステートメント](/dashboard/dashboard-overview.md#top-sql-statements)パネルで見ることができます。スローリードクエリの時間コストを調査するには、前述の記述に従って、[ポイントゲット](#point-get)、[バッチポイントゲット](#batch-point-get)、およびいくつかの[シンプルなコプロセッサクエリ](#table-scan--index-scan)を分析できます。

### スローライトクエリ

スローライトの調査を行う前に、`tikv_scheduler_latch_wait_duration_seconds_sum{type="acquire_pessimistic_lock"} by (instance)`を確認して、競合の原因をトラブルシューティングする必要があります。

- 特定のTiKVインスタンスでこのメトリクスが高い場合、ホットリージョンでの競合が発生している可能性があります。
- すべてのインスタンスでこのメトリクスが高い場合、アプリケーションで競合が発生している可能性があります。

アプリケーションから競合の原因を確認した後、[Lock](#lock)と[Commit](#commit)の期間を分析して、スローライトクエリを調査できます。
```