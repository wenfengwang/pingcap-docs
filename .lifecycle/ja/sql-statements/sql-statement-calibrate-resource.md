---
title: リソースのキャリブレーション
summary: TiDBデータベースのCALIBRATE RESOURCEの使用概要。

---

# `CALIBRATE RESOURCE`

`CALIBRATE RESOURCE`ステートメントは、現在のクラスタのリクエストユニット（RU）の容量を推定して出力するために使用されます。 ['リクエストユニット（RU）`の容量プロット](/tidb-resource-control#what-is-request-unit-ru)

> **注意:**
>
> この機能は、TiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

## 概要

```ebnf+diagram
CalibrateResourceStmt ::= 'CALIBRATE' 'RESOURCE' WorkloadOption

WorkloadOption ::=
( 'WORKLOAD' ('TPCC' | 'OLTP_READ_WRITE' | 'OLTP_READ_ONLY' | 'OLTP_WRITE_ONLY') )
| ( 'START_TIME' 'TIMESTAMP' ('DURATION' stringLit | 'END_TIME' 'TIMESTAMP')?)?

```

## 権限

このコマンドを実行するには、次の要件を満たしていることを確認してください。

- [`tidb_enable_resource_control`](/system-variables.md#tidb_enable_resource_control-new-in-v660)を有効にしていること。
- ユーザが`SUPER`または`RESOURCE_GROUP_ADMIN`権限を持っていること。
- `METRICS_SCHEMA`スキーマ内のすべてのテーブルに対して`SELECT`権限を持っていること。

## 容量の推定方法

TiDBでは、次の2つの方法で容量を推定できます。

### 実際のワークロードに基づく容量の推定

アプリケーションが既に本番環境で実行されているか、実際のビジネステストを実行できる場合は、推定のために一定期間の実際のワークロードを使用することを推奨します。推定の精度を向上させるために、次の制約に注意してください。

- `START_TIME`パラメータを使用して、推定が開始される時点の時刻を`2006-01-02 15:04:05`の形式で指定します。デフォルトの推定終了時刻は現在時刻です。
- `START_TIME`パラメータを指定した後は、`END_TIME`パラメータを使用して推定終了時刻を指定するか、`START_TIME`からの推定時間ウィンドウを指定するために`DURATION`パラメータを使用できます。
- 時間ウィンドウは、10分から24時間までの範囲です。
- 指定された時間ウィンドウ内でTiDBとTiKVのCPU利用率が低すぎる場合、容量を推定できません。

> **注意:**
>
> TiKVはmacOSでのCPU使用率メトリックスを監視しません。macOSでは、実際のワークロードに基づく容量の推定をサポートしません。

### ハードウェアデプロイメントに基づく容量の推定

この方法は主に、現在のクラスタ構成と異なるワークロードに対する経験則的な値に基づいて容量を推定します。さまざまなタイプのワークロードには異なるハードウェアの比率が必要なため、同じハードウェア構成の出力容量は異なる場合があります。こちらの`WORKLOAD`パラメーターは以下のワークロードの異なるタイプを受け入れます。デフォルト値は`TPCC`です。

- `TPCC`: ヘビーなデータ書き込みを行うワークロードに適用されます。`TPC-C`に似たワークロードモデルを基に推定されます。
- `OLTP_WRITE_ONLY`: ヘビーなデータ書き込みを行うワークロードに適用されます。`sysbench oltp_write_only`に似たワークロードモデルを基に推定されます。
- `OLTP_READ_WRITE`: データの読み書きが均等なワークロードに適用されます。`sysbench oltp_read_write`に似たワークロードモデルを基に推定されます。
- `OLTP_READ_ONLY`: ヘビーなデータ読み取りを行うワークロードに適用されます。`sysbench oltp_read_only`に似たワークロードモデルを基に推定されます。
- `TPCH_10`: APクエリに適用されます。`TPCH-10G`からの22のクエリに基づいて推定されます。

> **注意:**
>
> クラスタのRU容量は、クラスタのトポロジー、各コンポーネントのハードウェアおよびソフトウェア構成と異なります。各クラスタが提供できる実際のRUは実際のワークロードと関係があります。ハードウェアデプロイメントに基づいて推定された値は参考値であり、実際の最大値と異なる場合があります。[実際のワークロードに基づく容量を推定](#estimate-capacity-based-on-actual-workload)することをお勧めします。

## 例

開始時間`START_TIME`と時間ウィンドウ`DURATION`を指定して、実際のワークロードに応じたRU容量を表示します。

```sql
CALIBRATE RESOURCE START_TIME '2023-04-18 08:00:00' DURATION '20m';
+-------+
| QUOTA |
+-------+
| 27969 |
+-------+
1 行が返されました（0.01 秒）

```

開始時間 `START_TIME`と終了時間 `END_TIME`を指定して、実際のワークロードに応じたRU容量を表示します。

```sql
CALIBRATE RESOURCE START_TIME '2023-04-18 08:00:00' END_TIME '2023-04-18 08:20:00';
+-------+
| QUOTA |
+-------+
| 27969 |
+-------+
1 行が返されました（0.01 秒）
```

時間ウィンドウの範囲 `DURATION` が10分から24時間の間にない場合は、エラーが発生します。

```sql
CALIBRATE RESOURCE START_TIME '2023-04-18 08:00:00' DURATION '25h';
ERROR 1105 (HY000): the duration of calibration is too long, which could lead to inaccurate output. Please make the duration between 10m0s and 24h0m0s
CALIBRATE RESOURCE START_TIME '2023-04-18 08:00:00' DURATION '9m';
ERROR 1105 (HY000): the duration of calibration is too short, which could lead to inaccurate output. Please make the duration between 10m0s and 24h0m0s
```

[実際のワークロードに基づく容量の推定](#estimate-capacity-based-on-actual-workload)のモニタリングメトリックスには、`tikv_cpu_quota`、`tidb_server_maxprocs`、`resource_manager_resource_unit`、`process_cpu_usage`、`tiflash_cpu_quota`、`tiflash_resource_manager_resource_unit`、`tiflash_process_cpu_usage`が含まれています。CPUクォータのモニタリングデータが空の場合、次の例のように対応するモニタリングメトリックス名にエラーが表示されます。

```sql
CALIBRATE RESOURCE START_TIME '2023-04-18 08:00:00' DURATION '60m';
Error 1105 (HY000): There is no CPU quota metrics, metrics 'tikv_cpu_quota' is empty
```

時間窓のワークロードが低すぎる、または `resource_manager_resource_unit` と `process_cpu_usage` のモニタリングデータが欠落している場合、次のエラーが報告されます。また、TiKVはmacOSでCPU使用率を監視しないため、実際のワークロードに基づく容量推定をサポートせず、このエラーも報告されます。

```sql
CALIBRATE RESOURCE START_TIME '2023-04-18 08:00:00' DURATION '60m';
ERROR 1105 (HY000): The workload in selected time window is too low, with which TiDB is unable to reach a capacity estimation; please select another time window with higher workload, or calibrate resource by hardware instead
```

`WORKLOAD`を指定してRU容量を表示します。デフォルト値は`TPCC`です。

```sql
CALIBRATE RESOURCE;
+-------+
| QUOTA |
+-------+
| 190470 |
+-------+
1 行が返されました（0.01 秒）

CALIBRATE RESOURCE WORKLOAD OLTP_WRITE_ONLY;
+-------+
| QUOTA |
+-------+
| 27444 |
+-------+
1 行が返されました（0.01 秒）
```