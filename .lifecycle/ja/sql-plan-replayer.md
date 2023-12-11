---
title: PLAN REPLAYERを使用してクラスターの現地情報を保存および復元する方法
summary: PLAN REPLAYERを使用してクラスターの現地情報を保存および復元する方法について説明します。
---

# PLAN REPLAYERを使用してクラスターの現地情報を保存および復元する方法

TiDBクラスターの問題を特定およびトラブルシューティングする際には、システムおよび実行計画の情報を提供する必要があります。より便利かつ効率的な方法で情報を取得し、クラスターの問題をトラブルシューティングするために、TiDB v5.3.0で`PLAN REPLAYER`コマンドが導入されました。このコマンドを使用すると、クラスターの現地情報を簡単に保存および復元でき、トラブルシューティングの効率が向上し、問題を簡単にアーカイブできます。

`PLAN REPLAYER`の機能は次のとおりです:

- TiDBクラスターの現地トラブルシューティング情報をZIP形式のファイルにエクスポートして保存します。
- 他のTiDBクラスターからエクスポートしたZIP形式のファイルをクラスターにインポートします。このファイルには後述のTiDBクラスターの現地トラブルシューティング情報が含まれています。

## `PLAN REPLAYER`を使用してクラスター情報をエクスポートする

`PLAN REPLAYER`を使用してTiDBクラスターの現地情報を保存できます。エクスポートインターフェースは次のとおりです:

{{< copyable "sql" >}}

```sql
PLAN REPLAYER DUMP EXPLAIN [ANALYZE] [WITH STATS AS OF TIMESTAMP expression] sql-statement;
```

`sql-statement`に基づいて、TiDBは以下の現地情報を整理してエクスポートします:

- TiDBのバージョン
- TiDBの構成
- TiDBのセッション変数
- TiDB SQLのバインディング
- `sql-statement`内のテーブルスキーマ
- `sql-statement`内のテーブルの統計情報
- `EXPLAIN [ANALYZE] sql-statement`の結果
- クエリ最適化のいくつかの内部手順

履歴統計が[有効](/system-variables.md#tidb_enable_historical_stats)になっている場合、`PLAN REPLAYER`ステートメントで特定の時刻を指定して対応する時刻の履歴統計を取得できます。直接時刻と日付を指定するか、タイムスタンプを指定できます。TiDBは指定した時刻よりも前の履歴統計を検索し、その中で最新のものをエクスポートします。

指定した時刻よりも前の履歴統計がない場合、指定した時刻の前から最新の統計がエクスポートされます。また、TiDBはエクスポートされた`ZIP`ファイル内の`errors.txt`ファイルにエラーメッセージを出力します。

> **注意:**
>
> `PLAN REPLAYER`はテーブルデータを**エクスポートしません**。

### クラスター情報のエクスポートの例

{{< copyable "sql" >}}

```sql
use test;
create table t(a int, b int);
insert into t values(1,1), (2, 2), (3, 3);
analyze table t;

plan replayer dump explain select * from t;
plan replayer dump with stats as of timestamp '2023-07-17 12:00:00' explain select * from t;
plan replayer dump with stats as of timestamp '442012134592479233' explain select * from t;
```

`PLAN REPLAYER DUMP`は上記のテーブル情報を`ZIP`ファイルにまとめ、ファイル識別子を実行結果として返します。

> **注意:**
>
> `ZIP`ファイルはTiDBクラスターに最大1時間保存されます。1時間後にTiDBはファイルを削除します。

```sql
MySQL [test]> plan replayer dump explain select * from t;
```

```sql
+------------------------------------------------------------------+
| Dump_link                                                        |
+------------------------------------------------------------------+
| replayer_JOGvpu4t7dssySqJfTtS4A==_1635750890568691080.zip |
+------------------------------------------------------------------+
1 row in set (0.015 sec)
```

または、セッション変数[`tidb_last_plan_replayer_token`](/system-variables.md#tidb_last_plan_replayer_token-new-in-v630)を使用して、前回の`PLAN REPLAYER DUMP`の実行結果を取得できます。

```sql
SELECT @@tidb_last_plan_replayer_token;
```

```sql
+-----------------------------------------------------------+
| @@tidb_last_plan_replayer_token                           |
+-----------------------------------------------------------+
| replayer_Fdamsm3C7ZiPJ-LQqgVjkA==_1663304195885090000.zip |
+-----------------------------------------------------------+
1 row in set (0.00 sec)
```

複数のSQLステートメントがある場合、ファイルを使用して`PLAN REPLAYER DUMP`の実行結果を取得できます。複数のSQLステートメントの実行結果は、このファイル内で`;`で区切られます。

```sql
plan replayer dump explain 'sqls.txt';
```

```sql
Query OK, 0 rows affected (0.03 sec)
```

```sql
SELECT @@tidb_last_plan_replayer_token;
```

```sql
+-----------------------------------------------------------+
| @@tidb_last_plan_replayer_token                           |
+-----------------------------------------------------------+
| replayer_LEDKg8sb-K0u24QesiH8ig==_1663226556509182000.zip |
+-----------------------------------------------------------+
1 row in set (0.00 sec)
```

ファイルはMySQLクライアントではダウンロードできないため、TiDBのHTTPインターフェースとファイル識別子を使用してファイルをダウンロードする必要があります:

{{< copyable "shell-regular" >}}

```shell
http://${tidb-server-ip}:${tidb-server-status-port}/plan_replayer/dump/${file_token}
```

`${tidb-server-ip}:${tidb-server-status-port}`はクラスター内の任意のTiDBサーバーのアドレスです。例:

{{< copyable "shell-regular" >}}

```shell
curl http://127.0.0.1:10080/plan_replayer/dump/replayer_JOGvpu4t7dssySqJfTtS4A==_1635750890568691080.zip > plan_replayer.zip
```

## `PLAN REPLAYER`を使用してクラスター情報をインポートする

> **警告:**
>
> TiDBクラスターの現地情報を他のクラスターにインポートする場合、後者のクラスターのTiDBセッション変数、SQLバインディング、テーブルスキーマおよび統計情報が変更されます。

`PLAN REPLAYER`を使用して既存の`ZIP`ファイルをインポートすることで、クラスターの現地情報を他のTiDBクラスターに復元できます。構文は次のとおりです:

{{< copyable "sql" >}}

```sql
PLAN REPLAYER LOAD 'file_name';
```

上記のステートメントで、`file_name`はエクスポートする`ZIP`ファイルの名前です。

例:

{{< copyable "sql" >}}

```sql
PLAN REPLAYER LOAD 'plan_replayer.zip';
```

クラスター情報がインポートされた後、TiDBクラスターには必要なテーブルスキーマ、統計情報などの情報がロードされます。以下の方法で実行計画を表示したり、統計情報を検証したりできます:

```sql
mysql> desc t;
+-------+---------+------+------+---------+-------+
| Field | Type    | Null | Key  | Default | Extra |
+-------+---------+------+------+---------+-------+
| a     | int(11) | YES  |      | NULL    |       |
| b     | int(11) | YES  |      | NULL    |       |
+-------+---------+------+------+---------+-------+
2 rows in set (0.01 sec)

mysql> explain select * from t where a = 1 or b =1;
+-------------------------+---------+-----------+---------------+--------------------------------------+
| id                      | estRows | task      | access object | operator info                        |
+-------------------------+---------+-----------+---------------+--------------------------------------+
| TableReader_7           | 0.01    | root      |               | data:Selection_6                     |
| └─Selection_6           | 0.01    | cop[tikv] |               | or(eq(test.t.a, 1), eq(test.t.b, 1)) |
|   └─TableFullScan_5     | 6.00    | cop[tikv] | table:t       | keep order:false, stats:pseudo       |
+-------------------------+---------+-----------+---------------+--------------------------------------+
3 rows in set (0.00 sec)

mysql> show stats_meta;
+---------+------------+----------------+---------------------+--------------+-----------+
| Db_name | Table_name | Partition_name | Update_time         | Modify_count | Row_count |
+---------+------------+----------------+---------------------+--------------+-----------+
| test    | t          |                | 2022-08-26 15:52:07 |            3 |         6 |
+---------+------------+----------------+---------------------+--------------+-----------+
1 row in set (0.04 sec)
```

シーンがロードされ、復元された後、クラスターの実行計画を診断および改善できます。

## `PLAN REPLAYER CAPTURE`を使用して対象プランをキャプチャする

特定のシナリオでTiDBの実行計画を特定する際、対象のSQLステートメントと実行計画はクエリ内でのみ偶発的に表示されることがあり、そのため`PLAN REPLAYER`は直接文と計画をキャプチャできないことがあります。そのような場合には、`PLAN REPLAYER CAPTURE`を使用して対象のSQLステートメントと対象の計画の最適化情報をキャプチャすることができます。
```markdown
`PLAN REPLAYER CAPTURE`には、次の主な機能があります:

- ターゲットSQLステートメントとその実行計画のダイジェストを事前にTiDBクラスタに登録し、ターゲットクエリを一致させ始めます。
- ターゲットクエリが正常に一致した場合、その最適化関連情報を直接キャプチャしてZIPファイルとしてエクスポートします。
- 各一致したSQLと実行計画について、情報は1回だけキャプチャされます。
- 進行中の一致タスクと生成されたファイルは、システムテーブルを通じて表示されます。
- 定期的に過去のファイルをクリーンアップします。

### `PLAN REPLAYER CAPTURE`を有効にする

`PLAN REPLAYER CAPTURE`はシステム変数[`tidb_enable_plan_replayer_capture`](/system-variables.md#tidb_enable_plan_replayer_capture)で制御されます。`PLAN REPLAYER CAPTURE`を有効にするには、システム変数の値を`ON`に設定します。

### `PLAN REPLAYER CAPTURE`の使用方法

次のステートメントを使用して、ターゲットSQLステートメントと実行計画のダイジェストをTiDBクラスタに登録できます:

```sql
PLAN REPLAYER CAPTURE 'sql_digest' 'plan_digest';
```

ターゲットSQLステートメントに複数の実行計画が含まれ、すべての実行計画をキャプチャしたい場合は、次のステートメントを使用して一度にすべての実行計画を登録できます:

```sql
PLAN REPLAYER CAPTURE 'sql_digest' '*';
```

### キャプチャタスクを表示する

次のステートメントを使用して、TiDBクラスタで`PLAN REPLAYER CAPTURE`の進行中のキャプチャタスクを表示できます:

```sql
mysql> PLAN PLAYER CAPTURE 'example_sql' 'example_plan';
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM mysql.plan_replayer_task;
+-------------+--------------+---------------------+
| sql_digest  | plan_digest  | update_time         |
+-------------+--------------+---------------------+
| example_sql | example_plan | 2023-01-28 11:58:22 |
+-------------+--------------+---------------------+
1 row in set (0.01 sec)
```

### キャプチャ結果を表示する

`PLAN REPLAYER CAPTURE`が結果を正常にキャプチャした後は、次のSQLステートメントを使用してファイルのダウンロードに使用されるトークンを表示できます:

```sql
mysql> SELECT * FROM mysql.plan_replayer_status;
+------------------------------------------------------------------+------------------------------------------------------------------+------------+-----------------------------------------------------------+---------------------+-------------+-----------------+
| sql_digest                                                       | plan_digest                                                      | origin_sql | token                                                     | update_time         | fail_reason | instance        |
+------------------------------------------------------------------+------------------------------------------------------------------+------------+-----------------------------------------------------------+---------------------+-------------+-----------------+
| 086e3fbd2732f7671c17f299d4320689deeeb87ba031240e1e598a0ca14f808c | 042de2a6652a6d20afc629ff90b8507b7587a1c7e1eb122c3e0b808b1d80cc02 |            | replayer_Utah4nkz2sIEzkks7tIRog==_1668746293523179156.zip | 2022-11-18 12:38:13 | NULL        | 172.16.4.4:4022 |
| b5b38322b7be560edb04f33f15b15a885e7c6209a22b56b0804622e397199b54 | 1770efeb3f91936e095f0344b629562bf1b204f6e46439b7d8f842319297c3b5 |            | replayer_Z2mUXNHDjU_WBmGdWQqifw==_1668746293560115314.zip | 2022-11-18 12:38:13 | NULL        | 172.16.4.4:4022 |
| 96d00c0b3f08795fe94e2d712fa1078ab7809faf4e81d198f276c0dede818cf9 | 8892f74ac2a42c2c6b6152352bc491b5c07c73ac3ed66487b2c990909bae83e8 |            | replayer_RZcRHJB7BaCccxFfOIAhWg==_1668746293578282450.zip | 2022-11-18 12:38:13 | NULL        | 172.16.4.4:4022 |
+------------------------------------------------------------------+------------------------------------------------------------------+------------+-----------------------------------------------------------+---------------------+-------------+-----------------+
3 rows in set (0.00 sec)
```

`PLAN REPLAYER CAPTURE`のファイルのダウンロード方法は、`PLAN REPLAYER`と同じです。詳細については、[クラスタ情報のエクスポートの例](#examples-of-exporting-cluster-information)を参照してください。

> **注意:**
>
> `PLAN REPLAYER CAPTURE`の結果ファイルは、最大1週間TiDBクラスタに保持されます。1週間後に、TiDBはファイルを削除します。

## `PLAN REPLAYER CONTINUOUS CAPTURE`の使用

`PLAN REPLAYER CONTINUOUS CAPTURE`を有効にした後、TiDBは、`SQL DIGEST`と`PLAN DIGEST`に従ってアプリケーションのSQLステートメントを非同期で`PLAN REPLAYER`の方法で記録します。同じDIGESTを共有するSQLステートメントと実行計画については、`PLAN REPLAYER CONTINUOUS CAPTURE`はそれらを繰り返し記録しません。

### `PLAN REPLAYER CONTINUOUS CAPTURE`を有効にする

`PLAN REPLAYER CONTINUOUS CAPTURE`はシステム変数[`tidb_enable_plan_replayer_continuous_capture`](/system-variables.md#tidb_enable_plan_replayer_continuous_capture-new-in-v700)で制御されます。`PLAN REPLAYER CONTINUOUS CAPTURE`を有効にするには、システム変数の値を`ON`に設定します。

### キャプチャ結果を表示する

`PLAN REPLAYER CONTINUOUS CAPTURE`のキャプチャ結果を表示する方法は [Viewing the capture results of `PLAN REPLAYER CAPTURE`](#view-the-capture-results)と同じです。
```