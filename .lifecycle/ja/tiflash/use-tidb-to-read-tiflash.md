---
title: TiFlash レプリカを読むために TiDB を使用する
summary: TiDB を使用して TiFlash レプリカを読む方法を学ぶ
---

# TiDB を使用して TiFlash レプリカを読む

このドキュメントでは、TiDB を使用して TiFlash レプリカを読む方法について紹介します。

TiDB では、TiFlash レプリカを読むための3つの方法が提供されています。エンジン構成を行わないで TiFlash レプリカを追加した場合、CBO (コストベース最適化) モードがデフォルトで使用されます。

## スマート選択

TiFlash レプリカがあるテーブルに対して、TiDB オプティマイザはコスト推定に基づいて自動的にTiFlash レプリカを使用するかどうかを決定します。`desc` や `explain analyze` ステートメントを使用して、TiFlash レプリカが選択されたかどうかを確認できます。たとえば：

{{< copyable "sql" >}}

```sql
desc select count(*) from test.t;
```

```
+--------------------------+---------+--------------+---------------+--------------------------------+
| id                       | estRows | task         | access object | operator info                  |
+--------------------------+---------+--------------+---------------+--------------------------------+
| StreamAgg_9              | 1.00    | root         |               | funcs:count(1)->Column#4       |
| └─TableReader_17         | 1.00    | root         |               | data:TableFullScan_16          |
|   └─TableFullScan_16     | 1.00    | cop[tiflash] | table:t       | keep order:false, stats:pseudo |
+--------------------------+---------+--------------+---------------+--------------------------------+
3 rows in set (0.00 sec)
```

{{< copyable "sql" >}}

```sql
explain analyze select count(*) from test.t;
```

```
+--------------------------+---------+---------+--------------+---------------+----------------------------------------------------------------------+--------------------------------+-----------+------+
| id                       | estRows | actRows | task         | access object | execution info                                                       | operator info                  | memory    | disk |
+--------------------------+---------+---------+--------------+---------------+----------------------------------------------------------------------+--------------------------------+-----------+------+
| StreamAgg_9              | 1.00    | 1       | root         |               | time:83.8372ms, loops:2                                              | funcs:count(1)->Column#4       | 372 Bytes | N/A  |
| └─TableReader_17         | 1.00    | 1       | root         |               | time:83.7776ms, loops:2, rpc num: 1, rpc time:83.5701ms, proc keys:0 | data:TableFullScan_16          | 152 Bytes | N/A  |
|   └─TableFullScan_16     | 1.00    | 1       | cop[tiflash] | table:t       | tiflash_task:{time:43ms, loops:1, threads:1}, tiflash_scan:{...}     | keep order:false, stats:pseudo | N/A       | N/A  |
+--------------------------+---------+---------+--------------+---------------+----------------------------------------------------------------------+--------------------------------+-----------+------+
```

`cop[tiflash]` はタスクが処理のために TiFlash に送信されることを意味します。TiFlash レプリカを選択していない場合は、`analyze table` ステートメントを使用して統計情報を更新し、その後 `explain analyze` ステートメントを使用して結果を確認できます。

1つの TiFlash レプリカしか持たないテーブルがあり、関連ノードがサービスを提供できない場合、CBO モードのクエリは繰り返しリトライを行います。この状況では、エンジンを指定するか、TiKV レプリカからデータを読むために手動ヒントを使用する必要があります。 

## エンジンの分離

エンジンの分離は、対応する変数を構成することで、すべてのクエリが指定されたエンジンのレプリカを使用するよう指定するものです。オプションとしては "tikv"、"tidb"（TiDB の内部メモリーテーブル領域であり、ユーザーによってアクティブに使用されないいくつかの TiDB システムテーブルが格納されています）、そして "tiflash" が利用可能です。

<CustomContent platform="tidb">

次の2つの構成レベルでエンジンを指定できます：

* TiDB インスタンスレベル（INSTANCE レベル）。TiDB 構成ファイルに次の構成項目を追加します：

    ```
    [isolation-read]
    engines = ["tikv", "tidb", "tiflash"]
    ```

    **INSTANCE レベルのデフォルト構成は `["tikv", "tidb", "tiflash"]` です。**

* SESSION レベル。次のステートメントを使用して構成します：

    {{< copyable "sql" >}}

    ```sql
    set @@session.tidb_isolation_read_engines = "カンマで区切られたエンジンのリスト";
    ```

    または

    {{< copyable "sql" >}}

    ```sql
    set SESSION tidb_isolation_read_engines = "カンマで区切られたエンジンのリスト";
    ```

    SESSION レベルのデフォルト構成は TiDB INSTANCE レベルの構成から継承されます。

最終的なエンジン構成はセッションレベルの構成です。つまり、セッションレベルの構成がインスタンスレベルの構成をオーバーライドします。たとえば、INSTANCE レベルで "tikv" を構成し、SESSION レベルで "tiflash" を構成した場合、TiFlash レプリカが読み込まれます。最終的なエンジン構成が "tikv" ＋ "tiflash" の場合、TiKV と TiFlash レプリカの両方を読み込み、オプティマイザが自動的により良いエンジンを選択します。

> **注意:**
>
> [TiDB Dashboard](/dashboard/dashboard-intro.md) など、他のコンポーネントは TiDB メモリーテーブル領域に格納されているいくつかのシステムテーブルを読み取る必要があるため、インスタンスレベルのエンジン構成に常に "tidb" エンジンを追加することを推奨します。

</CustomContent>

<CustomContent platform="tidb-cloud">

次のステートメントを使用してエンジンを指定できます：

```sql
set @@session.tidb_isolation_read_engines = "カンマで区切られたエンジンのリスト";
```

または

```sql
set SESSION tidb_isolation_read_engines = "カンマで区切られたエンジンのリスト";
```

</CustomContent>

指定されたテーブルが指定されたエンジンのレプリカを持っていない場合（たとえばエンジンが "tiflash" に構成されているが、テーブルが TiFlash レプリカを持っていない場合）、クエリはエラーを返します。

## 手動ヒント

手動ヒントを使用すると、エンジンの分離の前提を満たしつつ、特定のテーブルの指定したレプリカを TiDB に使用させることができます。手動ヒントの使用例を以下に示します：

{{< copyable "sql" >}}

```sql
select /*+ read_from_storage(tiflash[table_name]) */ ... from table_name;
```

クエリステートメントでテーブルにエイリアスを設定する場合、ヒントでエイリアスを使用する必要があります。たとえば：

{{< copyable "sql" >}}

```sql
select /*+ read_from_storage(tiflash[alias_a,alias_b]) */ ... from table_name_1 as alias_a, table_name_2 as alias_b where alias_a.column_1 = alias_b.column_2;
```

上記のステートメントでは、`tiflash[]` はオプティマイザに TiFlash レプリカを読み込むよう指示します。必要に応じて `tikv[]` を使用して TiKV レプリカを読み込むこともできます。ヒント構文の詳細については、[READ_FROM_STORAGE](/optimizer-hints.md#read_from_storaget1_name--t1_name--t2_name--t2_name-)を参照してください。

ヒントで指定されたテーブルに指定されたエンジンのレプリカがない場合、ヒントは無視され、警告が報告されます。また、ヒントはエンジンの分離の前提条件に基づいてのみ有効です。ヒントで指定されたエンジンがエンジン分離リストに含まれていない場合、ヒントは無視され、警告が報告されます。

> **注意:**
>
> 5.7.7 やそれ以前のバージョンの MySQL クライアントでは、デフォルトでオプティマイザヒントがクリアされます。これらの古いバージョンでヒント構文を使用するには、たとえば `mysql -h 127.0.0.1 -P 4000 -uroot --comments` のようにクライアントを `--comments` オプションで起動する必要があります。

## スマート選択、エンジンの分離、および手動ヒントの関係

前述の3つの TiFlash レプリカを読む方法では、エンジンの分離がエンジンの利用可能なレプリカの全体範囲を指定し、その範囲内で手動ヒントがもっと細かいレベルでのステートメントとテーブルのエンジン選択を提供します。最終的に CBO が決定し、指定されたエンジンリスト内でコスト推定に基づいてエンジンのレプリカを選択します。

> **注意:**
>
> v4.0.3 より前では、読み取り専用でない SQL ステートメント（たとえば `INSERT INTO ... SELECT`、`SELECT ... FOR UPDATE`、`UPDATE ...`、`DELETE ...`など）における TiFlash レプリカからの読み取り動作は未定義でした。v4.0.3 以降では、内部的に TiDB はデータの正確性を保証するために、読み取り専用でない SQL ステートメントのTiFlash レプリカの読み込みを無視します。つまり、[スマート選択](#smart-selection) では自動的に TiFlash レプリカ以外を選択します。TiFlash レプリカのみを指定する [エンジンの分離](#engine-isolation) では、TiDB はエラーを報告します。そして、[手動ヒント](#manual-hint) では、TiDB はヒントを無視します。