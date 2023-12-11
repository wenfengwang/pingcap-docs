---
title: HTAP クラスターの使用
summary: TiDB Cloud で HTAP クラスターを使用する方法について学びます。

# HTAP クラスターの使用
[HTAP](https://en.wikipedia.org/wiki/Hybrid_transactional/analytical_processing) はハイブリッドトランザクショナル/解析処理を意味します。TiDB Cloud のHTAPクラスターは、トランザクション処理向けに設計された行ベースのストレージエンジンである[TiKV](https://tikv.org)と、解析処理向けに設計された列ベースのストレージである[TiFlash](https://docs.pingcap.com/tidb/stable/tiflash-overview)から構成されています。アプリケーションデータはまずTiKVに格納され、その後Raftコンセンサスアルゴリズムを介してTiFlashにレプリケートされます。つまり、行ベースのストアから列ベースのストアへのリアルタイムレプリケーションが行われます。

TiDB Cloudを使用すると、HTAPクラスターを簡単に作成できます。HTAPワークロードに応じて1つ以上のTiFlashノードを指定して、HTAPクラスターを作成できます。クラスターを作成する際にTiFlashノードの数を指定しないか、さらにTiFlashノードを追加したい場合は、[クラスターのスケーリング](/tidb-cloud/scale-tidb-cluster.md)によってノード数を変更できます。

> **注意:**
>
> TiDB Serverlessクラスターでは、TiFlashは常に有効です。無効にすることはできません。

デフォルトではTiKVデータはTiFlashにレプリケートされません。次のSQLステートメントを使用して、TiFlashにレプリケートするテーブルを選択できます。

{{< copyable "sql" >}}

```sql
ALTER TABLE table_name SET TIFLASH REPLICA 1;
```

レプリカ数はTiFlashノードの数以下でなければなりません。レプリカの数を `0` に設定すると、TiFlashのレプリカが削除されます。


レプリケーションの進行状況を確認するには、次のコマンドを使用します。

{{< copyable "sql" >}}

```sql
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = '<db_name>' and TABLE_NAME = '<table_name>';
```

## TiDBを使用してTiFlashレプリカを読む

データがTiFlashにレプリケートされた後、次の3つの方法のいずれかを使用して、TiFlashレプリカを読み取って解析処理を高速化できます。

### スマート選択

TiFlashレプリカがあるテーブルの場合、TiDBのオプティマイザーは自動的にコスト推定に基づいてTiFlashレプリカを使用するかどうかを決定します。例:

{{< copyable "sql" >}}

```sql
explain analyze select count(*) from test.t;
```

```sql
+--------------------------+---------+---------+--------------+---------------+----------------------------------------------------------------------+--------------------------------+-----------+------+
| id                       | estRows | actRows | task         | access object | execution info                                                       | operator info                  | memory    | disk |
+--------------------------+---------+---------+--------------+---------------+----------------------------------------------------------------------+--------------------------------+-----------+------+
| StreamAgg_9              | 1.00    | 1       | root         |               | time:83.8372ms, loops:2                                              | funcs:count(1)->Column#4       | 372 Bytes | N/A  |
| └─TableReader_17         | 1.00    | 1       | root         |               | time:83.7776ms, loops:2, rpc num: 1, rpc time:83.5701ms, proc keys:0 | data:TableFullScan_16          | 152 Bytes | N/A  |
|   └─TableFullScan_16     | 1.00    | 1       | cop[tiflash] | table:t       | time:43ms, loops:1                                                   | keep order:false, stats:pseudo | N/A       | N/A  |
+--------------------------+---------+---------+--------------+---------------+----------------------------------------------------------------------+--------------------------------+-----------+------+
```

`cop[tiflash]` は、このタスクがTiFlashに送信されることを意味します。クエリでTiFlashレプリカが選択されていない場合は、`analyze table` ステートメントを使用して統計情報を更新し、その後`explain analyze` ステートメントを使用して結果を確認してください。

### エンジンの分離

エンジンの分離は、`tidb_isolation_read_engines`変数を構成することで、すべてのクエリが指定されたエンジンのレプリカを使用するようにすることです。オプションのエンジンは、"tikv"、"tidb"（TiDBの内部メモリテーブルエリアであり、TiDBシステムテーブルを保持し、ユーザーが積極的に使用できない）、「tiflash」です。

{{< copyable "sql" >}}

```sql
set @@session.tidb_isolation_read_engines = "エンジンリストをカンマで区切って入力";
```

### マニュアルヒント

マニュアルヒントは、エンジンの分離を満たす前提で、TiDBに特定のテーブルの1つ以上の特定のレプリカを使用するように強制します。以下はマニュアルヒントを使用する例です。

{{< copyable "sql" >}}

```sql
select /*+ read_from_storage(tiflash[table_name]) */ ... from table_name;
```

TiFlashについてさらに詳しく学ぶには、[こちら](https://docs.pingcap.com/tidb/stable/tiflash-overview/)のドキュメントを参照してください。
