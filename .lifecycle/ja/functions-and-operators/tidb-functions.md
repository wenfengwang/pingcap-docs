---
title: TiDB 特定の関数
summary: TiDB 特有の関数の使用方法について学ぶ
---

# TiDB 特定の関数

以下の関数はTiDBの拡張機能であり、MySQLには存在しません。

<CustomContent platform="tidb">

| 関数名 | 関数の説明 |
| :-------------- | :------------------------------------- |
| `TIDB_BOUNDED_STALENESS()` | `TIDB_BOUNDED_STALENESS` 関数は、指定した時間範囲内で可能な限り新しいデータを読み取るようTiDBに指示します。また、[AS OF TIMESTAMP句を使用した過去のデータの読み取り](/as-of-timestamp.md)も参照してください。 |
| [`TIDB_DECODE_KEY(str)`](#tidb_decode_key) | `TIDB_DECODE_KEY` 関数は、TiDBでエンコードされたキーエントリを`_tidb_rowid`と`table_id`を含むJSON構造にデコードするために使用できます。これらのエンコードされたキーは、一部のシステムテーブルやログ出力で見つけることができます。 |
| [`TIDB_DECODE_PLAN(str)`](#tidb_decode_plan) | `TIDB_DECODE_PLAN` 関数は、TiDBの実行計画をデコードするために使用できます。 |
| `TIDB_IS_DDL_OWNER()` | `TIDB_IS_DDL_OWNER` 関数は、接続しているTiDBインスタンスがDDLオーナーであるかどうかをチェックするために使用できます。DDLオーナーはクラスタ内のすべての他のノードを代表してDDLステートメントを実行するTiDBインスタンスです。 |
| [`TIDB_PARSE_TSO(num)`](#tidb_parse_tso) | `TIDB_PARSE_TSO` 関数は、TiDB TSOタイムスタンプから物理的なタイムスタンプを抽出するために使用できます。また、[`tidb_current_ts`](/system-variables.md#tidb_current_ts)も参照してください。 |
| `TIDB_PARSE_TSO_LOGICAL(num)` | `TIDB_PARSE_TSO_LOGICAL` 関数は、TiDB TSOタイムスタンプから論理的なタイムスタンプを抽出するために使用できます。 |
| [`TIDB_VERSION()`](#tidb_version) | `TIDB_VERSION` 関数は、TiDBのバージョンと追加のビルド情報を返します。 |
| [`TIDB_DECODE_SQL_DIGESTS(digests, stmtTruncateLength)`](#tidb_decode_sql_digests) | `TIDB_DECODE_SQL_DIGESTS()` 関数は、クラスタ内のSQLダイジェストに対応する正規化されたSQLステートメント（フォーマットや引数のない形式）をクエリするために使用されます。 |
| `VITESS_HASH(str)` | `VITESS_HASH` 関数は、Vitessの`HASH`関数と互換性のある文字列のハッシュを返します。これはVitessからのデータ移行をサポートするためのものです。 |
| `TIDB_SHARD()` | `TIDB_SHARD` 関数は、インデックスホットスポットを分散させるためのシャードインデックスを作成するために使用できます。シャードインデックスは、プレフィックスとして`TIDB_SHARD`関数を持つ式インデックスです。|
| `TIDB_ROW_CHECKSUM()` | `TIDB_ROW_CHECKSUM` 関数は、行のチェックサム値をクエリするために使用されます。この関数は`SELECT`ステートメント内のFastPlanプロセスでのみ使用できます。つまり、`SELECT TIDB_ROW_CHECKSUM() FROM t WHERE id = ?`や`SELECT TIDB_ROW_CHECKSUM() FROM t WHERE id IN (?, ?, ...)`などのステートメントを介してクエリできます。また、[単一行データのデータ整合性検証](https://docs.pingcap.com/tidb/stable/ticdc-integrity-check)も参照してください。 |
| `CURRENT_RESOURCE_GROUP()`  | `CURRENT_RESOURCE_GROUP` 関数は、現在のセッションがバインドされているリソースグループ名を返すために使用されます。また、[リソース制御を使用してリソースの分離を実現する](/tidb-resource-control.md)も参照してください。 |

</CustomContent>

<CustomContent platform="tidb-cloud">

| 関数名 | 関数の説明 |
| :-------------- | :------------------------------------- |
| `TIDB_BOUNDED_STALENESS()` | `TIDB_BOUNDED_STALENESS` 関数は、指定した時間範囲内で可能な限り新しいデータを読み取るようTiDBに指示します。また、[AS OF TIMESTAMP句を使用した過去のデータの読み取り](/as-of-timestamp.md)も参照してください。 |
| [`TIDB_DECODE_KEY(str)`](#tidb_decode_key) | `TIDB_DECODE_KEY` 関数は、TiDBでエンコードされたキーエントリを`_tidb_rowid`と`table_id`を含むJSON構造にデコードするために使用できます。これらのエンコードされたキーは、一部のシステムテーブルやログ出力で見つけることができます。 |
| [`TIDB_DECODE_PLAN(str)`](#tidb_decode_plan) | `TIDB_DECODE_PLAN` 関数は、TiDBの実行計画をデコードするために使用できます。 |
| `TIDB_IS_DDL_OWNER()` | `TIDB_IS_DDL_OWNER` 関数は、接続しているTiDBインスタンスがDDLオーナーであるかどうかをチェックするために使用できます。DDLオーナーはクラスタ内のすべての他のノードを代表してDDLステートメントを実行するTiDBインスタンスです。 |
| [`TIDB_PARSE_TSO(num)`](#tidb_parse_tso) | `TIDB_PARSE_TSO` 関数は、TiDB TSOタイムスタンプから物理的なタイムスタンプを抽出するために使用できます。また、[`tidb_current_ts`](/system-variables.md#tidb_current_ts)も参照してください。 |
| `TIDB_PARSE_TSO_LOGICAL(num)` | `TIDB_PARSE_TSO_LOGICAL` 関数は、TiDB TSOタイムスタンプから論理的なタイムスタンプを抽出するために使用できます。 |
| [`TIDB_VERSION()`](#tidb_version) | `TIDB_VERSION` 関数は、TiDBのバージョンと追加のビルド情報を返します。 |
| [`TIDB_DECODE_SQL_DIGESTS(digests, stmtTruncateLength)`](#tidb_decode_sql_digests) | `TIDB_DECODE_SQL_DIGESTS()` 関数は、クラスタ内のSQLダイジェストに対応する正規化されたSQLステートメント（フォーマットや引数のない形式）をクエリするために使用されます。 |
| `VITESS_HASH(str)` | `VITESS_HASH` 関数は、Vitessの`HASH`関数と互換性のある文字列のハッシュを返します。これはVitessからのデータ移行をサポートするためのものです。 |
| `TIDB_SHARD()` | `TIDB_SHARD` 関数は、インデックスホットスポットを分散させるためのシャードインデックスを作成するために使用できます。シャードインデックスは、プレフィックスとして`TIDB_SHARD`関数を持つ式インデックスです。|
| `TIDB_ROW_CHECKSUM()` | `TIDB_ROW_CHECKSUM` 関数は、行のチェックサム値をクエリするために使用されます。この関数は`SELECT`ステートメント内のFastPlanプロセスでのみ使用できます。つまり、`SELECT TIDB_ROW_CHECKSUM() FROM t WHERE id = ?`や`SELECT TIDB_ROW_CHECKSUM() FROM t WHERE id IN (?, ?, ...)`などのステートメントを介してクエリできます。また、[単一行データのデータ整合性検証](https://docs.pingcap.com/tidb/stable/ticdc-integrity-check)も参照してください。 |
| `CURRENT_RESOURCE_GROUP()`  | `CURRENT_RESOURCE_GROUP` 関数は、現在のセッションがバインドされているリソースグループ名を返すために使用されます。また、[リソース制御を使用してリソースの分離を実現する](/tidb-resource-control.md)も参照してください。 |

</CustomContent>

## 例

このセクションでは、上記の一部の関数について例を提供します。

### TIDB_DECODE_KEY

次の例では、テーブル`t1`にはTiDBによって生成された非表示の`rowid`があります。ステートメントで`TIDB_DECODE_KEY`が使用されています。結果から、非表示の`rowid`がデコードされて出力されていることがわかります。これは非クラスタ化プライマリキーの典型的な結果です。

{{< copyable "sql" >}}

```sql
SELECT START_KEY, TIDB_DECODE_KEY(START_KEY) FROM information_schema.tikv_region_status WHERE table_name='t1' AND REGION_ID=2\G
```

```sql
*************************** 1. row ***************************
                 START_KEY: 7480000000000000FF3B5F728000000000FF1DE3F10000000000FA
TIDB_DECODE_KEY(START_KEY): {"_tidb_rowid":1958897,"table_id":"59"}
1 row in set (0.00 sec)
```

次の例では、テーブル`t2`には複合クラスタ化プライマリキーがあります。JSON出力から、プライマリキーの構成要素である両方の列の名前と値を含む`handle`が見えます。

{{< copyable "sql" >}}

```sql
show create table t2\G
```

```sql
*************************** 1. row ***************************
       Table: t2
Create Table: CREATE TABLE `t2` (
  `id` binary(36) NOT NULL,
  `a` tinyint(3) unsigned NOT NULL,
  `v` varchar(512) DEFAULT NULL,
  PRIMARY KEY (`a`,`id`) /*T![clustered_index] CLUSTERED */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
1 row in set (0.001 sec)
```

{{< copyable "sql" >}}

```sql
select * from information_schema.tikv_region_status where table_name='t2' limit 1\G
```

```sql
*************************** 1. 行目 ***************************
                REGION_ID: 48
                START_KEY: 7480000000000000FF3E5F720400000000FF0000000601633430FF3338646232FF2D64FF3531632D3131FF65FF622D386337352DFFFF3830653635303138FFFF61396265000000FF00FB000000000000F9
                  END_KEY:
                 TABLE_ID: 62
                  DB_NAME: test
               TABLE_NAME: t2
                 IS_INDEX: 0
                 INDEX_ID: NULL
               INDEX_NAME: NULL
           EPOCH_CONF_VER: 1
            EPOCH_VERSION: 38
            WRITTEN_BYTES: 0
               READ_BYTES: 0
         APPROXIMATE_SIZE: 136
         APPROXIMATE_KEYS: 479905
  REPLICATIONSTATUS_STATE: NULL
REPLICATIONSTATUS_STATEID: NULL
1 行を取得 (0.005 sec)

{{< copiable "sql" >}}

```sql
select tidb_decode_key('7480000000000000FF3E5F720400000000FF0000000601633430FF3338646232FF2D64FF3531632D3131FF65FF622D386337352DFFFF3830653635303138FFFF61396265000000FF00FB000000000000F9');
```

```sql
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tidb_decode_key('7480000000000000FF3E5F720400000000FF0000000601633430FF3338646232FF2D64FF3531632D3131FF65FF622D386337352DFFFF3830653635303138FFFF61396265000000FF00FB000000000000F9') |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| {"handle":{"a":"6","id":"c4038db2-d51c-11eb-8c75-80e65018a9be"},"table_id":62}                                                                                                        |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 行を取得 (0.001 sec)
```

### TIDB_DECODE_PLAN

TiDBの実行計画をエンコード形式でスロークエリログで確認できます。`TIDB_DECODE_PLAN()` 関数を使用して、エンコードされた計画を人間が読める形式にデコードすることができます。

この関数は、計画がステートメントが実行された時点でキャプチャされます。`EXPLAIN` でステートメントを再実行しても、データの分布と統計が時間とともに変化するため、異なる結果が得られる可能性があります。

```sql
SELECT tidb_decode_plan('8QIYMAkzMV83CQEH8E85LjA0CWRhdGE6U2VsZWN0aW9uXzYJOTYwCXRpbWU6NzEzLjHCtXMsIGxvb3BzOjIsIGNvcF90YXNrOiB7bnVtOiAxLCBtYXg6IDU2OC41wgErRHByb2Nfa2V5czogMCwgcnBjXxEpAQwFWBAgNTQ5LglZyGNvcHJfY2FjaGVfaGl0X3JhdGlvOiAwLjAwfQkzLjk5IEtCCU4vQQoxCTFfNgkxXzAJMwm2SGx0KHRlc3QudC5hLCAxMDAwMCkNuQRrdgmiAHsFbBQzMTMuOMIBmQnEDDk2MH0BUgEEGAoyCTQzXzUVwX1oGFibGU6dCwga2VlcCBvcmRlcjpmYWxzZSwgc3RhdHM6cHNldWRvCTk2ISE2aAAIMTUzXmYA')\G
```

```sql
*************************** 1. 行目 ***************************
  tidb_decode_plan('8QIYMAkzMV83CQEH8E85LjA0CWRhdGE6U2VsZWN0aW9uXzYJOTYwCXRpbWU6NzEzLjHCtXMsIGxvb3BzOjIsIGNvcF90YXNrOiB7bnVtOiAxLCBtYXg6IDU2OC41wgErRHByb2Nfa2V5czogMCwgcnBjXxEpAQwFWBAgNTQ5LglZyGNvcHJfY2FjaGVfaGl0X3JhdGlvOiAWLjAwfQkzLjk5IEtCCU4vQQoxCTFfNgkxXz:     id                     task         estRows    operator info                              actRows    execution info                                                                                                                         memory     disk
    TableReader_7          root         319.04     data:Selection_6                           960        time:713.1µs, loops:2, cop_task: {num: 1, max: 568.5µs, proc_keys: 0, rpc_num: 1, rpc_time: 549.1µs, copr_cache_hit_ratio: 0.00}    3.99 KB    N/A
    └─Selection_6          cop[tikv]    319.04     lt(test.t.a, 10000)                        960        tikv_task:{time:313.8µs, loops:960}                                                                                                   N/A        N/A
      └─TableFullScan_5    cop[tikv]    960        table:t, keep order:false, stats:pseudo    960        tikv_task:{time:153µs, loops:960}                                                                                                     N/A        N/A
```

### TIDB_PARSE_TSO

`TIDB_PARSE_TSO` 関数を使用して、TiDB TSO タイムスタンプから物理タイムスタンプを抽出することができます。TSO は Time Stamp Oracle の略で、PD（Placement Driver）によってトランザクションごとに提供される単調増加するタイムスタンプです。

TSO は以下の2つの部分から構成される数値です：

- 物理タイムスタンプ
- 論理カウンタ

```sql
BEGIN;
SELECT TIDB_PARSE_TSO(@@tidb_current_ts);
ROLLBACK;
```

```sql
+-----------------------------------+
| TIDB_PARSE_TSO(@@tidb_current_ts) |
+-----------------------------------+
| 2021-05-26 11:33:37.776000        |
+-----------------------------------+
1 行を取得 (0.0012 sec)
```

ここで、 `TIDB_PARSE_TSO` は、`tidb_current_ts` セッション変数で利用可能なタイムスタンプ番号から物理タイムスタンプを抽出するために使用されています。タイムスタンプはトランザクションごとに提供されるため、この関数はトランザクション内で実行されています。

### TIDB_VERSION

`TIDB_VERSION` 関数を使用すると、接続している TiDB サーバーのバージョンおよびビルドの詳細を取得できます。GitHub で問題を報告する際にこの関数を使用できます。

```sql
SELECT TIDB_VERSION()\G
```

```sql
*************************** 1. 行目 ***************************
TIDB_VERSION(): リリースバージョン: v5.1.0-alpha-13-gd5e0ed0aa-dirty
エディション: Community
Git コミットハッシュ: d5e0ed0aaed72d2f2dfe24e9deec31cb6cb5fdf0
Git ブランチ: master
UTC ビルド時間: 2021-05-24 14:39:20
GoVersion: go1.13
Race Enabled: false
TiKV Min Version: v3.0.0-60965b006877ca7234adaced7890d7b029ed1306
Check Table Before Drop: false
1 行を取得 (0.00 sec)
```

### TIDB_DECODE_SQL_DIGESTS

`TIDB_DECODE_SQL_DIGESTS()` 関数は、クラスター内の SQL ダイジェストに対応する正規化された SQL 文（フォーマットや引数のない形式）をクエリするために使用されます。この関数は 1 つまたは 2 つの引数を受け入れます：

* `digests`: 文字列。このパラメータは JSON 文字列の配列形式で、配列内の各文字列が SQL ダイジェストです。
* `stmtTruncateLength`: 整数（オプション）。返される結果の各 SQL 文の長さを制限するために使用されます。指定した長さを超える SQL 文は切り捨てられます。 `0` は長さが制限されていないことを意味します。

この関数は、JSON 文字列の配列形式である文字列を返します。配列内の i 番目の項目は、 `digests` パラメータ内の i 番目の要素に対応する正規化された SQL 文です。 `digests` パラメータ内の要素が有効な SQL ダイジェストでないか、システムが対応する SQL 文を見つけられない場合、返される結果の対応する項目は `null` です。切り捨ての長さが指定されている場合（`stmtTruncateLength > 0`）、返される結果の各文がこの長さを超えている場合、最初の `stmtTruncateLength` 文字が保持され、切り捨てを示すために最後に `"..."` が追加されます。 `digests` パラメータが `NULL` の場合、関数の返り値も `NULL` です。

> **注記:**
>
```sql
+--------------------------+
| CURRENT_RESOURCE_GROUP() |
+--------------------------+
| rg1                      |
+--------------------------+
```
```sql
+--------------------------+
| rg1                      |
+--------------------------+
1 行がセットされました (0.00 秒)

```

現在のセッションのリソースグループを`rg2`に設定するには、`SET RESOURCE GROUP`を実行し、その後現在のユーザーにバインドされたリソースグループを表示します。

```sql
SET RESOURCE GROUP `rg2`;
SELECT CURRENT_RESOURCE_GROUP();
```

```
+--------------------------+
| CURRENT_RESOURCE_GROUP() |
+--------------------------+
| rg2                      |
+--------------------------+
1 行がセットされました (0.00 秒)
```