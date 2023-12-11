---
title: TiDB ダッシュボードの診断レポートを使用して問題を特定する
summary: TiDB ダッシュボードの診断レポートを使用して問題を特定する方法について学びます。
aliases: ['/docs/dev/dashboard/dashboard-diagnostics-usage/']
---

# TiDB ダッシュボードの診断レポートを使用して問題を特定する

このドキュメントでは、TiDB ダッシュボードの診断レポートを使用して問題を特定する方法を紹介します。

## 比較診断

このセクションでは、比較診断機能を使用して、大きなクエリまたは書き込みによって引き起こされる QPS のジッタや遅延の増加を診断する方法を示します。

### 例1

![QPS example](/media/dashboard/dashboard-diagnostics-usage1.png)

上の画像は `go-ycsb` の圧力テストの結果を示しています。`2020-03-10 13:24:30` に、QPS が急激に減少し始めました。3 分後、QPS が正常値に戻り始めます。TiDB ダッシュボードの診断レポートを使用して原因を特定できます。

以下の 2 つの時間範囲のシステムを比較するレポートを生成します。

T1： `2020-03-10 13:21:00` から `2020-03-10 13:23:00`。この範囲では、システムは正常です。これを基準範囲と呼びます。

T2： `2020-03-10 13:24:30` から `2020-03-10 13:27:30`。この範囲では、QPS が減少しています。

ジッタの影響範囲が 3 分であるため、上記の 2 つの時間範囲はいずれも 3 分です。また、診断中に一部の監視された平均値が比較に使用されるため、時間範囲が長すぎると平均値の差が無視できるほどになり、問題を正確に特定できなくなります。

レポートが生成されたら、このレポートを **比較診断** ページで表示できます。

![Comparison diagnostics](/media/dashboard/dashboard-diagnostics-usage2.png)

上記の診断結果によると、診断時間中に大きなクエリが存在する可能性があります。上記のレポートの各 **詳細** は次のとおりです。

* `tidb_qps`: QPS が 0.93 倍に減少。
* `tidb_query_duration`: P999 クエリ遅延が 1.54 倍に増加。
* `tidb_cop_duration`: P999 コプロセッサリクエストの処理遅延が 2.48 倍に増加。
* `tidb_kv_write_num`: P999 TiDB トランザクションで書き込まれた KV の数が 7.61 倍に増加。
* `tikv_cop_scan_keys_total_nun`: TiKV コプロセッサによってスキャンされたキー/値の数が 3 つの TiKV インスタンスで大幅に改善されました。
* `pd_operator_step_finish_total_count` では、転送されたリーダーの数が、異常時間範囲でのスケジューリングが通常時間範囲よりも 2.45 倍に増加していることを示します。
* レポートは、遅いクエリが存在する可能性を示し、遅いクエリをクエリするために SQL ステートメントを使用できることを示しています。SQL ステートメントの実行結果は次のようになります。

```sql
SELECT * FROM (SELECT count(*), min(time), sum(query_time) AS sum_query_time, sum(Process_time) AS sum_process_time, sum(Wait_time) AS sum_wait_time, sum(Commit_time), sum(Request_count), sum(process_keys), sum(Write_keys), max(Cop_proc_max), min(query),min(prev_stmt), digest FROM information_schema.CLUSTER_SLOW_QUERY WHERE time >= '2020-03-10 13:24:30' AND time < '2020-03-10 13:27:30' AND Is_internal = false GROUP BY digest) AS t1 WHERE t1.digest NOT IN (SELECT digest FROM information_schema.CLUSTER_SLOW_QUERY WHERE time >= '2020-03-10 13:21:00' AND time < '2020-03-10 13:24:00' GROUP BY digest) ORDER BY t1.sum_query_time DESC limit 10\G
***************************[ 1. row ]***************************
count(*)           | 196
min(time)          | 2020-03-10 13:24:30.204326
sum_query_time     | 46.878509117
sum_process_time   | 265.924
sum_wait_time      | 8.308
sum(Commit_time)   | 0.926820886
sum(Request_count) | 6035
sum(process_keys)  | 201453000
sum(Write_keys)    | 274500
max(Cop_proc_max)  | 0.263
min(query)         | delete from test.tcs2 limit 5000;
min(prev_stmt)     |
digest             | 24bd6d8a9b238086c9b8c3d240ad4ef32f79ce94cf5a468c0b8fe1eb5f8d03df
```

上記の結果から、`13:24:30` 以降、大規模なバッチ削除が行われており、合計で 196 回実行され、各回でデータの 5,000 行が削除され、合計時間は 46.8 秒であることがわかります。

### 例2

大規模なクエリが実行されていない場合、そのクエリは遅いログに記録されません。このような状況でも診断できます。次の例を参照してください。

![QPS results](/media/dashboard/dashboard-diagnostics-usage3.png)

上の画像は別の `go-ycsb` の圧力テストの結果を示しています。`2020-03-08 01:46:30` に、QPS が急激に減少し、回復しなかったことがわかります。

以下の 2 つの時間範囲のシステムを比較するレポートを生成します。

T1： `2020-03-08 01:36:00` から `2020-03-08 01:41:00`。この範囲では、システムは正常です。これを基準範囲と呼びます。

T2： `2020-03-08 01:46:30` から `2020-03-08 01:51:30`。この範囲では、QPS が減少しています。

レポートが生成されたら、このレポートを **比較診断** ページで表示できます。

![Comparison diagnostics](/media/dashboard/dashboard-diagnostics-usage4.png)

診断結果は例1と似ています。上記画像の最後の行は、遅いクエリが存在する可能性があり、TiDB ログで高コストのクエリをクエリすることを示しています。SQL ステートメントの実行結果は次のようになります。

```sql
> SELECT * FROM information_schema.cluster_log WHERE type='tidb' AND time >= '2020-03-08 01:46:30' AND time < '2020-03-08 01:51:30' AND level = 'warn' AND message LIKE '%expensive_query%'\G
TIME     | 2020/03/08 01:47:35.846
TYPE     | tidb
INSTANCE | 172.16.5.40:4009
LEVEL    | WARN
MESSAGE  | [expensivequery.go:167] [expensive_query] [cost_time=60.085949605s] [process_time=2.52s] [wait_time=2.52s] [request_count=9] [total_keys=996009] [process_keys=996000] [num_cop_tasks=9] [process_avg_time=0.28s] [process_p90_time=0.344s] [process_max_time=0.344s] [process_max_addr=172.16.5.40:20150] [wait_avg_time=0.000777777s] [wait_p90_time=0.003s] [wait_max_time=0.003s] [wait_max_addr=172.16.5.40:20150] [stats=t_wide:pseudo] [conn_id=19717] [user=root] [database=test] [table_ids="[80,80]"] [txn_start_ts=415132076148785201] [mem_max="23583169 Bytes (22.490662574768066 MB)"] [sql="select count(*) from t_wide as t1 join t_wide as t2 where t1.c0>t2.c1 and t1.c2>0"]
```

上記のクエリ結果から、この `172.16.5.40:4009` の TiDB インスタンスで、`2020/03/08 01:47:35.846` に 60 秒間実行された高コストのクエリがあり、まだ実行が完了していないことがわかります。このクエリはデカルト積の結合です。

## 比較診断レポートを使用した問題の特定

診断が誤っている可能性があるため、比較レポートを使用すると、 DBA が問題をより迅速に特定できることがあります。次の例を参照してください。

![QPS results](/media/dashboard/dashboard-diagnostics-usage5.png)

上記の画像は `go-ycsb` の圧力テストの結果を示しています。`2020-05-22 22:14:00` に、QPS が急に減少し始めました。3 分後、QPS が正常値に戻り始めます。TiDB ダッシュボードの比較診断レポートを使用して原因を特定できます。

以下の 2 つの時間範囲のシステムを比較するレポートを生成します。

```
T1: `2020-05-22 22:11:00` to `2020-05-22 22:14:00`. In this range, the system is normal, which is called a reference range.

T2: `2020-05-22 22:14:00` `2020-05-22 22:17:00`. In this range, QPS began to decrease.

After generating the comparison report, check the **Max diff item** report. This report compares the monitoring items of the two time ranges above and sorts them according to the difference of the monitoring items. The result of this table is as follows:

![Comparison results](/media/dashboard/dashboard-diagnostics-usage6.png)

From the result above, you can see that the Coprocessor requests in T2 are much more than those in T1. It might be that some large queries appear in T2 that bring more load.

In fact, during the entire time range from T1 to T2, the `go-ycsb` pressure test is running. Then 20 `tpch` queries are running during T2. So it is the `tpch` queries that cause many Coprocessor requests.

If such a large query whose execution time exceeds the threshold of slow log is recorded in the slow log. You can check the `Slow Queries In Time Range t2` report to see whether there is any slow query. However, some queries in T1 might become slow queries in T2, because in T2, other loads cause their executions to slow down.
```