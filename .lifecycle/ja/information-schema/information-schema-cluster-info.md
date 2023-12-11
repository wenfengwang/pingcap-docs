---
title: CLUSTER_INFO
summary: `CLUSTER_INFO`クラスタトポロジ情報表の使用方法について学ぶ。
aliases: ['/docs/dev/system-tables/system-table-cluster-info/','/docs/dev/reference/system-databases/cluster-info/','/tidb/dev/system-table-cluster-info/']
---

# CLUSTER_INFO

`CLUSTER_INFO`クラスタトポロジ表は、クラスタの現在のトポロジ情報、各インスタンスのバージョン情報、インスタンスバージョンに対応するGitハッシュ、各インスタンスの開始時間、および各インスタンスの実行時間を提供します。

> **注：**
>
> この表は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。

{{< copyable "sql" >}}

```sql
USE information_schema;
desc cluster_info;
```

```sql
+----------------+-------------+------+------+---------+-------+
| Field          | Type        | Null | Key  | Default | Extra |
+----------------+-------------+------+------+---------+-------+
| TYPE           | varchar(64) | YES  |      | NULL    |       |
| INSTANCE       | varchar(64) | YES  |      | NULL    |       |
| STATUS_ADDRESS | varchar(64) | YES  |      | NULL    |       |
| VERSION        | varchar(64) | YES  |      | NULL    |       |
| GIT_HASH       | varchar(64) | YES  |      | NULL    |       |
| START_TIME     | varchar(32) | YES  |      | NULL    |       |
| UPTIME         | varchar(32) | YES  |      | NULL    |       |
| SERVER_ID      | bigint(21)  | YES  |      | NULL    |       |
+----------------+-------------+------+------+---------+-------+
8 行が返されました (0.01 sec)
```

フィールドの説明：

* `TYPE`: インスタンスのタイプ。`tidb`、`pd`、`tikv`のいずれかのオプション値です。
* `INSTANCE`: インスタンスのアドレスで、`IP:PORT`の形式の文字列です。
* `STATUS_ADDRESS`: HTTP APIのサービスアドレス。`tikv-ctl`、`pd-ctl`、または`tidb-ctl`のコマンドの中にはこのAPIとこのアドレスを使用するものがあります。このアドレスを通じてより多くのクラスタ情報を取得することもできます。詳細については、[TiDB HTTP APIドキュメント](https://github.com/pingcap/tidb/blob/master/docs/tidb_http_api.md)を参照してください。
* `VERSION`: 対応するインスタンスのセマンティックバージョン番号。MySQLバージョン番号と互換性を保つため、TiDBバージョンは`${mysql-version}-${tidb-version}`の形式で表示されます。
* `GIT_HASH`: インスタンスバージョンをコンパイルする際のGitコミットハッシュであり、2つのインスタンスが完全に一致するバージョンであるかどうかを識別するために使用されます。
* `START_TIME`: 対応するインスタンスの開始時間。
* `UPTIME`: 対応するインスタンスの稼働時間。
* `SERVER_ID`: 対応するインスタンスのサーバーID。

{{< copyable "sql" >}}

```sql
SELECT * FROM cluster_info;
```

```sql
+------+-----------------+-----------------+--------------+------------------------------------------+---------------------------+---------------------+
| TYPE | INSTANCE        | STATUS_ADDRESS  | VERSION      | GIT_HASH                                 | START_TIME                | UPTIME              |
+------+-----------------+-----------------+--------------+------------------------------------------+---------------------------+---------------------+
| tidb | 0.0.0.0:4000    | 0.0.0.0:10080   | 4.0.0-beta.2 | 0df3b74f55f8f8fbde39bbd5d471783f49dc10f7 | 2020-07-05T09:25:53-06:00 | 26h39m4.352862693s  |
| pd   | 127.0.0.1:2379  | 127.0.0.1:2379  | 4.1.0-alpha  | 1ad59bcbf36d87082c79a1fffa3b0895234ac862 | 2020-07-05T09:25:47-06:00 | 26h39m10.352868103s |
| tikv | 127.0.0.1:20165 | 127.0.0.1:20180 | 4.1.0-alpha  | b45e052df8fb5d66aa8b3a77b5c992ddbfbb79df | 2020-07-05T09:25:50-06:00 | 26h39m7.352869963s  |
+------+-----------------+-----------------+--------------+------------------------------------------+---------------------------+---------------------+
3 行が返されました (0.00 sec)
```