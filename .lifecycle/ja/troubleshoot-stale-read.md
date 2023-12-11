---
title: TiKVのStale Readとsafe-tsの理解
summary: Stale Readとsafe-tsの原則をTiKVで紹介し、Stale Readに関連する一般的な問題のトラブルシューティングのヒントや例を提供します。

# TiKVにおけるStale Readとsafe-tsの理解

このガイドでは、TiKVでのStale Readとsafe-tsについて学び、Stale Readに関連する一般的な問題の診断方法について説明します。

## Stale Readとsafe-tsの概要

[Stale Read](/stale-read.md) は、TiDBがTiDBに格納されているデータの過去のバージョンを読み取るためのメカニズムです。TiKVにおいて、Stale Readは[safe-ts](/#what-is-safe-ts)に依存しています。リージョンのピアに対する読み取りリクエストのタイムスタンプ（ts）がリージョンのsafe-ts以下であれば、TiDBはそのデータを安全にピアから読み取ることができます。TiKVはこの安全性を保証するために、safe-tsが常に[resolved-ts](#what-is-resolved-ts)以下であることを保証しています。

## safe-tsとresolved-tsの理解

このセクションでは、safe-tsとresolved-tsの概念とメンテナンスについて説明します。

### safe-tsとは？

safe-tsは、リージョン内の各ピアが維持するタイムスタンプです。この値が以下の値より小さいすべてのトランザクションがローカルに適用されていることを保証し、ローカルStale Readを可能にします。

### resolved-tsとは？

resolved-tsは、その値より小さいすべてのトランザクションがリーダーによって適用されていることを保証するタイムスタンプです。safe-tsとは異なり、resolved-tsはリージョンのリーダーのみが維持します。フォロワーはリーダーよりも小さな適用インデックスを持つ場合があるため、resolved-tsはフォロワーにおいて直接safe-tsとして扱うことはできません。

### safe-tsのメンテナンス

`RegionReadProgress`モジュールがsafe-tsを維持します。リージョンのリーダーがresolved-tsを維持し、その後、そのresolved-tsとリージョン自体を`RegionReadProgress`モジュール間のCheckLeader RPCを介してすべてのレプリカに定期的に送信します。

ピアがデータを適用する際、適用インデックスを更新し、保留中のresolved-tsが新しいsafe-tsになれるかを確認します。

### resolved-tsのメンテナンス

リージョンのリーダーは解決者を使用してresolved-tsを管理します。この解決者は、Raftが適用されるときにロックCF（Column Family）内のロックをトラッキングすることで、resolved-tsをトラックします。初期化時には、解決者はロックをトラッキングするためにリージョン全体をスキャンします。

## Stale Readの問題の診断

このセクションでは、Grafana、`tikv-ctl`、およびログを使用してStale Readの問題を診断する方法を紹介します。

### 問題の特定

[Grafana > TiDB ダッシュボード > **KV Request** ダッシュボード](/grafana-tidb-dashboard.md#kv-request)には、Stale Readのヒット率、OPS、トラフィックを示す次のパネルが表示されます。

![Stale Read Hit/Miss OPS](/media/stale-read/metrics-hit-miss.png)

![Stale Read Req OPS](/media/stale-read/metrics-ops.png)

![Stale Read Req Traffic](/media/stale-read/traffic.png)

上記のメトリクスの詳細については、[TiDB監視メトリクス](/grafana-tidb-dashboard.md#kv-request)を参照してください。

Stale Readの問題が発生すると、前述のメトリクスに変化があることに気付くかもしれません。最も直接的な指標は、TiDBからのWARNログです。このログには、`DataIsNotReady`とRegion ID、遭遇した`safe-ts`が記載されています。

### 一般的な原因

Stale Readの有効性に影響を与える最も一般的な原因は次のとおりです。

- コミットに時間がかかるトランザクション。
- コミットする前に長い時間生き続けるトランザクション。
- リーダーからフォロワーへのCheckLeader情報の遅延。

### Grafanaを使用して診断

[**TiKVの詳細** > **Resolved-TS** ダッシュボード](/grafana-tikv-dashboard.md#resolved-ts)では、各TiKVごとに最小のresolved-tsとsafe-tsを持つリージョンを特定できます。これらのタイムスタンプが実際の時間よりも大幅に遅れている場合は、`tikv-ctl`でこれらのリージョンの詳細を確認する必要があります。

### `tikv-ctl`を使用して診断

`tikv-ctl`はリゾルバと`RegionReadProgress`の最新の詳細を提供します。詳細については、[リージョンの`RegionReadProgress`の状態を取得する](/tikv-control.md#get-the-state-of-a-regions-regionreadprogress)を参照してください。

以下は例です。

```bash
./tikv-ctl --host 127.0.0.1:20160 get-region-read-progress -r 14 --log --min-start-ts 0
```

出力は次のとおりです。

```log
リージョンの読み取り進行状況:
    exist: true,
    safe_ts: 0,
    applied_index: 92,
    pending front item (oldest) ts: 0,
    pending front item (oldest) applied index: 0,
    pending back item (latest) ts: 0,
    pending back item (latest) applied index: 0,
    paused: false,
リゾルバ:
    exist: true,
    resolved_ts: 0,
    tracked index: 92,
    number of locks: 0,
    number of transactions: 0,
    stopped: false,
```

前述の出力は以下についての情報を提供します。

- ロックがresolved-tsをブロックしているかどうか。
- 適用インデックスが小さすぎてsafe-tsを更新できないかどうか。
- リーダーがフォロワーピアが存在する場合に十分に更新されたresolved-tsを送信しているかどうか。

### ログを使用して診断

TiKVは10秒ごとに次のメトリクスをチェックします。

- resolved-tsが最小であるリージョンのリーダー
- safe-tsが最小であるリージョンのフォロワー
- resolved-tsが最小であるリージョンのフォロワー

これらのタイムスタンプのいずれかが異常に小さい場合、TiKVはログを出力します。

これらのログは、過去に存在したが現在は存在しない問題を診断する場合に特に有用です。

以下はログの例です。

```log
[2023/08/29 16:48:18.118 +08:00] [INFO] [endpoint.rs:505] ["the max gap of leader resolved-ts is large"] [last_resolve_attempt="Some(LastAttempt { success: false, ts: TimeStamp(443888082736381953), reason: \"lock\", lock: Some(7480000000000000625F728000000002512B5C) })"] [duration_to_last_update_safe_ts=10648ms] [min_memory_lock=None] [txn_num=0] [lock_num=0] [min_lock=None] [safe_ts=443888117326544897] [gap=110705ms] [region_id=291]

[2023/08/29 16:48:18.118 +08:00] [INFO] [endpoint.rs:526] ["the max gap of follower safe-ts is large"] [oldest_candidate=None] [latest_candidate=None] [applied_index=3276] [duration_to_last_consume_leader=11460ms] [resolved_ts=443888117117353985] [safe_ts=443888117117353985] [gap=111503ms] [region_id=273]

[2023/08/29 16:48:18.118 +08:00] [INFO] [endpoint.rs:547] ["the max gap of follower resolved-ts is large; it's the same region that has the min safe-ts"]
```

## トラブルシューティングのヒント

### 遅いトランザクションのコミットを処理する

コミットに時間がかかるトランザクションは、多くの場合、大きなトランザクションです。この遅いトランザクションの事前書き込みフェーズによっていくつかのロックが残りますが、コミットフェーズがロックをクリーンアップするまでに時間がかかりすぎます。この問題のトラブルシューティングには、これらのロックが属するトランザクションを特定し、それらが存在する理由を特定する試みが含まれます。これはログを使用するなどの方法で行うことができます。

次に、実行できるアクションの一部を示します。

- `tikv-ctl`コマンドで`--log`オプションを指定し、TiKVのログから特定のロックとそれらのstart_tsを見つけます。
- start_tsをTiDBとTiKVの両方のログで検索して、トランザクションに関連する問題を特定します。

    クエリが60秒を超えると、SQLステートメントが含まれた`expensive_query`ログが印刷されます。ログをマッチングするためにstart_tsの値を使用できます。以下は例です。

    ```
    [2023/07/17 19:32:09.403 +08:00] [WARN] [expensivequery.go:145] [expensive_query] [cost_time=60.025022732s] [cop_time=0.00346666s] [process_time=8.358409508s] [wait_time=0.013582596s] [request_count=278] [total_keys=9943616] [process_keys=9943360] [num_cop_tasks=278] [process_avg_time=0.030066221s] [process_p90_time=0.045296042s] [process_max_time=0.052828934s] [process_max_addr=192.168.31.244:20160] [wait_avg_time=0.000048858s] [wait_p90_time=0.00006057s] [wait_max_time=0.00040991s] [wait_max_addr=192.168.31.244:20160] [stats=t:442916666913587201] [conn=2826881778407440457] [user=root] [database=test] [table_ids="[100]"] [**txn_start_ts**=442916790435840001] [mem_max="2514229289 Bytes (2.34 GB)"] [sql="update t set b = b + 1"]
    ```

- [`CLUSTER_TIDB_TRX`](/information-schema/information-schema-tidb-trx.md#cluster_tidb_trx) テーブルを使用して、ログから十分な情報が得られない場合は、アクティブなトランザクションを検索します。
- 同じ TiDB サーバーに接続している現在のセッションおよびそれらの現在のステートメントに費やされた時間を表示するには、 [`SHOW PROCESSLIST`](/sql-statements/sql-statement-show-processlist.md) を実行します。ただし、`start_ts` は表示されません。

もしアクティブなトランザクションによってロックが存在する場合は、これらのロックが `resolve-ts` の進行を妨げる可能性があるため、アプリケーションロジックの変更を検討してください。

ロックが進行中のトランザクションに属していない場合は、プリライト後にコーディネータ（TiDB）がクラッシュすることが原因である可能性があります。この場合、TiDB は自動的にロックを解決します。問題が持続しない限り、特別な処置は必要ありません。

### 長期間のトランザクションの処理

長時間アクティブなトランザクションは、最終的に迅速にコミットされることがあっても、`resolve-ts` の前進を妨げる可能性があります。これは、これらの長寿命のトランザクションの `start-ts` が `resolved-ts` の計算に使用されるためです。

この問題に対処するためには:

- トランザクションを特定することから始めます。ロックに関連するトランザクションを特定することは非常に重要です。ログの活用が特に役立ちます。

- アプリケーション・ロジックを検討します。延長されたトランザクションの期間がアプリケーションのロジックの結果である場合は、そのような発生を防ぐためにロジックを改訂することを検討してください。

- 遅いクエリに対処する。トランザクションの期間が遅いクエリによって延長されている場合は、この問題を緩和するためにこれらのクエリを解決することを優先してください。

### CheckLeader の問題に対処する

CheckLeader の問題に対処するには、ネットワークをチェックし、[**TiKV-Details** > **Resolved-TS** ダッシュボード](/grafana-tikv-dashboard.md#resolved-ts) の **Check Leader Duration** メトリクスを確認することができます。

## 例

次のように **Stale Read OPS** のミス率が増加していることが観察される場合:

![Example: Stale Read OPS](/media/stale-read/example-ops.png)

最初に [**TiKV-Details** > **Resolved-TS** ダッシュボード](/grafana-tikv-dashboard.md#resolved-ts) の **Max Resolved TS gap** および **Min Resolved TS Region** メトリクスを確認することができます:

![Example: Max Resolved TS gap](/media/stale-read/example-ts-gap.png)

前述のメトリクスから、リージョン `3121` およびその他のいくつかのリージョンが resolved-ts を時間通りに更新していないことがわかります。

リージョン `3121` の状態についての詳細を取得するには、次のコマンドを実行できます:

```bash
./tikv-ctl --host 127.0.0.1:20160 get-region-read-progress -r 3121 --log
```

出力は次のようになります:

```log
リージョン読み取り進行状況:
    exist: true,
    safe_ts: 442918444145049601,
    applied_index: 2477,
    read_state.ts: 442918444145049601,
    read_state.apply_index: 1532,
    pending front item (oldest) ts: 0,
    pending front item (oldest) applied index: 0,
    pending back item (latest) ts: 0,
    pending back item (latest) applied index: 0,
    paused: false,
    discarding: false,
リゾルバ:
    exist: true,
    resolved_ts: 442918444145049601,
    tracked index: 2477,
    locks の数: 480000,
    トランザクションの数: 1,
    stopped: false,
```

ここで注目すべきなのは、`applied_index` がリゾルバの `tracked index` と等しいことです。したがって、この問題の原因となる可能性があるトランザクションが 1 つ存在し、このリージョンに 480000 のロックを残していることがわかります。

また、このトランザクションの正確な情報と一部のロックの keys を取得するには、TiKV のログをチェックして `locks with` を grep できます。出力は次のようになります:

```log
[2023/07/17 21:16:44.257 +08:00] [INFO] [resolver.rs:213] ["resolver で最小の start_ts を持つロック"] [keys="[74800000000000006A5F7280000000000405F6, ... , 74800000000000006A5F72800000000000EFF6, 74800000000000006A5F7280000000000721D9, 74800000000000006A5F72800000000002F691]"] [start_ts=442918429687808001] [region_id=3121]
```

TiKV のログから、トランザクションの `start_ts` である `442918429687808001` を取得できます。また、これに関連するステートメントとトランザクションの詳細を取得するには、TiDB のログでこのタイムスタンプを grep できます。出力は次のようになります:

```log
[2023/07/17 21:16:18.287 +08:00] [INFO] [2pc.go:685] ["[BIG_TXN]"] [session=2826881778407440457] ["key sample"=74800000000000006a5f728000000000000000] [size=319967171] [keys=10000000] [puts=10000000] [dels=0] [locks=0] [checks=0] [txnStartTS=442918429687808001]

[2023/07/17 21:16:22.703 +08:00] [WARN] [expensivequery.go:145] [expensive_query] [cost_time=60.047172498s] [cop_time=0.004575113s] [process_time=15.356963423s] [wait_time=0.017093811s] [request_count=397] [total_keys=20000398] [process_keys=10000000] [num_cop_tasks=397] [process_avg_time=0.038682527s] [process_p90_time=0.082608262s] [process_max_time=0.116321331s] [process_max_addr=192.168.31.244:20160] [wait_avg_time=0.000043057s] [wait_p90_time=0.00004007s] [wait_max_time=0.00075014s] [wait_max_addr=192.168.31.244:20160] [stats=t:442918428521267201] [conn=2826881778407440457] [user=root] [database=test] [table_ids="[106]"] [txn_start_ts=442918429687808001] [mem_max="2513773983 Bytes (2.34 GB)"] [sql="update t set b = b + 1"]
```

その後、基本的に問題の原因となったステートメントを特定できます。さらに確認するために、[`SHOW PROCESSLIST`](/sql-statements/sql-statement-show-processlist.md) 文を実行できます。出力は次のようになります:

```sql
+---------------------+------+---------------------+--------+---------+------+------------+---------------------------+
| Id                  | User | Host                | db     | Command | Time | State      | Info                      |
+---------------------+------+---------------------+--------+---------+------+------------+---------------------------+
| 2826881778407440457 | root | 192.168.31.43:58641 | test   | Query   | 48   | autocommit | update t set b = b + 1    |
| 2826881778407440613 | root | 127.0.0.1:45952     | test   | Execute | 0    | autocommit | select * from t where a=? |
| 2826881778407440619 | root | 192.168.31.43:60428 | <null> | Query   | 0    | autocommit | show processlist          |
+---------------------+------+---------------------+--------+---------+------+------------+---------------------------+
```
```
The output shows that someone is executing an unexpected `UPDATE` statement (`update t set b = b + 1`), which results in a large transaction and hinders Stale Read.

To resolve this issue, you can stop the application that is running this `UPDATE` statement.
```


```
出力によれば、誰かが予期しない `UPDATE` 文（`update t set b = b + 1`）を実行しており、これにより大規模なトランザクションが発生し、ステールリードが妨げられています。

この問題を解決するためには、この `UPDATE` 文を実行しているアプリケーションを停止することができます。
```