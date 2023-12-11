---
title: CLUSTER_LOG
summary: `CLUSTER_LOG`情報スキーマテーブルについて学ぶ。
aliases: ['/docs/dev/system-tables/system-table-cluster-log/','/docs/dev/reference/system-databases/cluster-log/','/tidb/dev/system-table-cluster-log/']
---

# CLUSTER_LOG

`CLUSTER_LOG`クラスターログテーブルでクラスターログをクエリできます。クエリ条件をそれぞれのインスタンスにプッシュダウンすることで、クエリがクラスターパフォーマンスに与える影響は`grep`コマンドよりも少なくなります。

> **注意:**
>
> このテーブルはTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

TiDBクラスターのログをv4.0より前で取得するためには、それぞれのインスタンスにログインしてログを要約する必要があります。この4.0のクラスターログテーブルは、グローバルで時間順に並んだログ検索結果を提供し、これにより完全なリンクイベントを追跡しやすくなります。例えば、`region id`に基づいてログを検索することで、このリージョンのライフサイクル全体のログをクエリできます。同様に、遅いログの`txn id`に基づいてフルリンクログを検索することで、このトランザクションによって各インスタンスでスキャンされるキーのフローと数をクエリできます。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC cluster_log;
```

```sql
+----------+------------------+------+------+---------+-------+
| Field    | Type             | Null | Key  | Default | Extra |
+----------+------------------+------+------+---------+-------+
| TIME     | varchar(32)      | YES  |      | NULL    |       |
| TYPE     | varchar(64)      | YES  |      | NULL    |       |
| INSTANCE | varchar(64)      | YES  |      | NULL    |       |
| LEVEL    | varchar(8)       | YES  |      | NULL    |       |
| MESSAGE  | var_string(1024) | YES  |      | NULL    |       |
+----------+------------------+------+------+---------+-------+
5 rows in set (0.00 sec)
```

フィールドの説明:

* `TIME`: ログの表示時刻。
* `TYPE`: インスタンスのタイプ。オプション値は`tidb`、`pd`、`tikv`です。
* `INSTANCE`: インスタンスのサービスアドレス。
* `LEVEL`: ログレベル。
* `MESSAGE`: ログの内容。

> **注意:**
>
> + クラスターログテーブルのすべてのフィールドは、対応するインスタンスに実行されるため、クラスターログテーブルを使用するオーバーヘッドを減らすために、検索に使用するキーワード、時間範囲、およびできるだけ多くの条件を指定する必要があります。例えば、`select * from cluster_log where message like '%ddl%' and time > '2020-05-18 20:40:00' and time<'2020-05-18 21:40:00' and type='tidb'`。
>
> + `message`フィールドは`like`および`regexp`正規表現をサポートし、対応するパターンは`regexp`としてエンコードされます。複数の`message`条件を指定することは、`grep`コマンドの`pipeline`形式と同等です。例えば、`select * from cluster_log where message like 'coprocessor%' and message regexp '.*slow.*' and time > '2020-05-18 20:40:00' and time<'2020-05-18 21:40:00'`文は、すべてのクラスターインスタンスで`grep 'coprocessor' xxx.log | grep -E '.*slow.*'`を実行するのと同等です。

次の例では、`CLUSTER_LOG`テーブルを使用してDDL文の実行プロセスをクエリする方法を示しています:

{{< copyable "sql" >}}

```sql
SELECT time,instance,left(message,150) FROM cluster_log WHERE message LIKE '%ddl%job%ID.80%' AND type='tidb' AND time > '2020-05-18 20:40:00' AND time < '2020-05-18 21:40:00'
```

```sql
+-------------------------+----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| time                    | instance       | left(message,150)                                                                                                                                      |
+-------------------------+----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| 2020/05/18 21:37:54.784 | 127.0.0.1:4002 | [ddl_worker.go:261] ["[ddl] add DDL jobs"] ["batch count"=1] [jobs="ID:80, Type=create table, State:none, SchemaState:none, SchemaID:1, TableID:79, Ro |
| 2020/05/18 21:37:54.784 | 127.0.0.1:4002 | [ddl.go:477] ["[ddl] start DDL job"] [job="ID:80, Type=create table, State:none, SchemaState:none, SchemaID:1, TableID:79, RowCount:0, ArgLen:1, start |
| 2020/05/18 21:37:55.327 | 127.0.0.1:4000 | [ddl_worker.go:568] ["[ddl] run DDL job"] [worker="worker 1, tp general"] [job="ID:80, Type=create table, State:none, SchemaState:none, SchemaID:1, Ta |
| 2020/05/18 21:37:55.381 | 127.0.0.1:4000 | [ddl_worker.go:763] ["[ddl] wait latest schema version changed"] [worker="worker 1, tp general"] [ver=70] ["take time"=50.809848ms] [job="ID:80, Type: |
| 2020/05/18 21:37:55.382 | 127.0.0.1:4000 | [ddl_worker.go:359] ["[ddl] finish DDL job"] [worker="worker 1, tp general"] [job="ID:80, Type=create table, State:synced, SchemaState:public, SchemaI |
| 2020/05/18 21:37:55.786 | 127.0.0.1:4002 | [ddl.go:509] ["[ddl] DDL job is finished"] [jobID=80]                                                                                                  |
+-------------------------+----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
```

上記のクエリ結果は、DDL文の実行プロセスを示しています:

1. `80`のDDL JOB IDを持つリクエストが`127.0.0.1:4002` TiDBインスタンスに送信されます。
2. `127.0.0.1:4000` TiDBインスタンスがこのDDLリクエストを処理し、その時点で`127.0.0.1:4000`インスタンスがDDLオーナーであることを示しています。
3. `80`のDDL JOB IDを持つリクエストが処理されました。