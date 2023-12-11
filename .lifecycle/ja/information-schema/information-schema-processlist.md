```yaml
---
title: PROCESSLIST
summary: 'PROCESSLIST'のinformation_schemaテーブルについて学びます。
---

# PROCESSLIST

`PROCESSLIST`は、`SHOW PROCESSLIST`と同様に、処理中のリクエストを表示するために使用されます。

`PROCESSLIST`テーブルには、`SHOW PROCESSLIST`にはない追加の列があります:

* `SQLステートメントのダイジェストを表示する`DIGEST`列。
* 処理中のリクエストに使用されるメモリをバイト単位で表示する`MEM`列。
* ディスク使用量をバイト単位で表示する`DISK`列。
* トランザクションの開始時間を表示する`TxnStart`列。
* リソースグループ名を表示する`RESOURCE_GROUP`列。

{{< コピー可能 "sql" >}}

```sql
USE information_schema;
DESC processlist;
```

```sql
+---------------------+---------------------+------+------+---------+-------+
| Field               | Type                | Null | Key  | Default | Extra |
+---------------------+---------------------+------+------+---------+-------+
| ID                  | bigint(21) unsigned | NO   |      | 0       |       |
| USER                | varchar(16)         | NO   |      |         |       |
| HOST                | varchar(64)         | NO   |      |         |       |
| DB                  | varchar(64)         | YES  |      | NULL    |       |
| COMMAND             | varchar(16)         | NO   |      |         |       |
| TIME                | int(7)              | NO   |      | 0       |       |
| STATE               | varchar(7)          | YES  |      | NULL    |       |
| INFO                | longtext            | YES  |      | NULL    |       |
| DIGEST              | varchar(64)         | YES  |      |         |       |
| MEM                 | bigint(21) unsigned | YES  |      | NULL    |       |
| DISK                | bigint(21) unsigned | YES  |      | NULL    |       |
| TxnStart            | varchar(64)         | NO   |      |         |       |
| RESOURCE_GROUP      | varchar(32)         | NO   |      |         |       |
+---------------------+---------------------+------+------+---------+-------+
13 行を表示 (0.00 sec)
```

{{< コピー可能 "sql" >}}

```sql
SELECT * FROM processlist\G
```

```sql
*************************** 1. row ***************************
                 ID: 2300033189772525975
               USER: root
               HOST: 127.0.0.1:51289
                 DB: NULL
            COMMAND: Query
               TIME: 0
              STATE: autocommit
               INFO: SELECT * FROM processlist
             DIGEST: dbfaa16980ec628011029f0aaf0d160f4b040885240dfc567bf760d96d374f7e
                MEM: 0
               DISK: 0
           TxnStart:
     RESOURCE_GROUP: rg1
1 行を表示 (0.00 sec)
```

`PROCESSLIST`テーブルのフィールドは以下のように記述されています:

* ID: ユーザーコネクションのID。
* USER: `PROCESS`を実行しているユーザーの名前。
* HOST: ユーザーが接続しているアドレス。
* DB: 現在接続しているデフォルトのデータベースの名前。
* COMMAND: `PROCESS`が実行しているコマンドの種類。
* TIME: `PROCESS`の現在の実行期間（秒単位）。
* STATE: 現在のコネクション状態。
* INFO: 処理中の要求されているステートメント。
* DIGEST: SQLステートメントのダイジェスト。
* MEM: 処理中のリクエストに使用されるメモリ（バイト単位）。
* DISK: ディスク使用量（バイト単位）。
* TxnStart: トランザクションの開始時間。
* RESOURCE_GROUP: リソースグループ名。

## CLUSTER_PROCESSLIST

`CLUSTER_PROCESSLIST`は、`PROCESSLIST`に対応するクラスターシステムテーブルです。クラスター内のすべてのTiDBノードの`PROCESSLIST`情報をクエリするために使用されます。 `CLUSTER_PROCESSLIST`テーブルのスキーマには、`PROCESSLIST`よりも1つの列が追加されており、`INSTANCE`列です。この列は、データのこの行が属するTiDBノードのアドレスを格納します。

{{< コピー可能 "sql" >}}

```sql
SELECT * FROM information_schema.cluster_processlist;
```

```sql
+-----------------+-----+------+----------+------+---------+------+------------+------------------------------------------------------+-----+----------------------------------------+----------------+
| INSTANCE        | ID  | USER | HOST     | DB   | COMMAND | TIME | STATE      | INFO                                                 | MEM | TxnStart                               | RESOURCE_GROUP | 
+-----------------+-----+------+----------+------+---------+------+------------+------------------------------------------------------+-----+----------------------------------------+----------------+
| 10.0.1.22:10080 | 150 | u1   | 10.0.1.1 | test | Query   | 0    | autocommit | select count(*) from usertable                       | 372 | 05-28 03:54:21.230(416976223923077223) | default        |
| 10.0.1.22:10080 | 138 | root | 10.0.1.1 | test | Query   | 0    | autocommit | SELECT * FROM information_schema.cluster_processlist | 0   | 05-28 03:54:21.230(416976223923077220) | rg1            |
| 10.0.1.22:10080 | 151 | u1   | 10.0.1.1 | test | Query   | 0    | autocommit | select count(*) from usertable                       | 372 | 05-28 03:54:21.230(416976223923077224) | rg2            |
| 10.0.1.21:10080 | 15  | u2   | 10.0.1.1 | test | Query   | 0    | autocommit | select max(field0) from usertable                    | 496 | 05-28 03:54:21.230(416976223923077222) | default        |
| 10.0.1.21:10080 | 14  | u2   | 10.0.1.1 | test | Query   | 0    | autocommit | select max(field0) from usertable                    | 496 | 05-28 03:54:21.230(416976223923077225) | default        |
+-----------------+-----+------+----------+------+---------+------+------------+------------------------------------------------------+-----+----------------------------------------+----------------+
```