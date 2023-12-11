---
title: TiDBのHTAPを素早く始める
summary: TiDBのHTAPを素早く始める方法を学びます。

# TiDB HTAPのクイックスタートガイド

このガイドでは、Hybrid Transactional and Analytical Processing（HTAP）のTiDBのワンストップソリューションを素早く始めるための最も簡単な方法について説明します。

> **注意:**
>
> このガイドで提供される手順はテスト環境での素早い開始のみです。本番環境では、[HTAPを探索](/explore-htap.md)することを推奨します。

## 基本的な概念

TiDB HTAPを使用する前に、TiDBオンライントランザクション処理（OLTP）向けの行ベースのストレージエンジンである[TiKV](/tikv-overview.md)と、TiDBオンライン解析処理（OLAP）向けの列ベースのストレージエンジンである[TiFlash](/tiflash/tiflash-overview.md)について基本的な知識が必要です。

- HTAPのストレージエンジン: 行ベースのストレージエンジンと列ベースのストレージエンジンが共存し、両方のストレージエンジンはデータを自動的にレプリケートしてデータの強力な整合性を保ちます。行ベースのストレージエンジンはOLTPパフォーマンスを最適化し、列ベースのストレージエンジンはOLAPパフォーマンスを最適化します。
- HTAPのデータ整合性: 分散型のトランザクショナルなキー値データベースであるTiKVは、ACID準拠のトランザクションインターフェースを提供し、[Raftコンセンサスアルゴリズム](https://raft.github.io/raft.pdf)の実装により複数のレプリカ間でのデータ整合性と高い可用性を保証します。TiKVの列ベースのストレージ拡張であるTiFlashは、Raft Learnerコンセンサスアルゴリズムに従ってTiKVからデータをリアルタイムで複製するため、データがTiKVとTiFlashの間で強力に整合的であることを保証します。
- HTAPのデータ隔離: TiKVとTiFlashはHTAPリソースの隔離の問題を解決するために必要に応じて異なるマシンに展開することができます。
- MPPコンピューティングエンジン: [MPP](/tiflash/use-tiflash-mpp-mode.md#control-whether-to-select-the-mpp-mode)はTiDB 5.0以降からTiFlashエンジンが提供する分散コンピューティングフレームワークであり、ノード間でデータの交換を可能にし、高性能で高スループットなSQLアルゴリズムを提供します。MPPモードでは、解析クエリの実行時間を大幅に短縮できます。

## 手順

このドキュメントでは、[TPC-H](http://www.tpc.org/tpch/)データセットの例のテーブルをクエリして、TiDB HTAPの利便性と高いパフォーマンスを体験できます。TPC-Hはビジネスに関連するアドホッククエリのスイートであり、大量のデータと高度な複雑さを備えている一般的な意思決定サポートベンチマークです。22の完全なSQLクエリを使用してTPC-Hを体験するには、[tidb-benchリポジトリ](https://github.com/pingcap/tidb-bench/tree/master/tpch/queries)を参照するか、クエリ文とデータを生成する手順については[TPC-H](http://www.tpc.org/tpch/)を参照してください。

### 手順1. ローカルテスト環境を展開する

TiDB HTAPを使用する前に、[TiDBデータベースプラットフォームのクイックスタートガイド](/quick-start-with-tidb.md)の手順に従ってローカルテスト環境を準備し、以下のコマンドを実行してTiDBクラスターを展開します。

{{< copyable "shell-regular" >}}

```shell
tiup playground
```

> **注意:**
>
> `tiup playground` コマンドは素早い開始専用であり、本番環境では使用しないでください。

### 手順2. テストデータを準備する

以下の手順では、TiDB HTAPで使用するための[TPC-H](http://www.tpc.org/tpch/)データセットを作成します。TPC-Hに興味がある場合は、[一般的な実装ガイドライン](http://tpc.org/tpc_documents_current_versions/pdf/tpc-h_v3.0.0.pdf)を参照してください。

> **注意:**
>
> 解析クエリで既存のデータを使用する場合は、[TiDBにデータをマイグレーション](/migration-overview.md)することができます。独自のテストデータを設計および作成する場合は、SQLステートメントを実行するか関連するツールを使用して作成できます。

1. 次のコマンドを実行して、テストデータ生成ツールをインストールします。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup install bench
    ```

2. 次のコマンドを実行して、テストデータを生成します。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup bench tpch --sf=1 prepare
    ```

    このコマンドの出力が「Finished」であれば、データが作成されていることを示します。

3. 次のSQLステートメントを実行して、生成されたデータを表示します。

    {{< copyable "sql" >}}

    ```sql
    SELECT
      CONCAT(table_schema,'.',table_name) AS 'テーブル名',
      table_rows AS '行数',
      FORMAT_BYTES(data_length) AS 'データサイズ',
      FORMAT_BYTES(index_length) AS 'インデックスサイズ',
      FORMAT_BYTES(data_length+index_length) AS '合計'
    FROM
      information_schema.TABLES
    WHERE
      table_schema='test';
    ```

    出力から、合計8つのテーブルが作成されており、最大のテーブルには650万行のデータが含まれていることがわかります（データは実際のSQLクエリの結果に依存するため、生成される行数はランダムです）。

    ```sql
    +---------------+----------------+-----------+------------+-----------+
    |  テーブル名   | 行数           | データサイズ | インデックスサイズ | 合計      |
    +---------------+----------------+-----------+------------+-----------+
    | test.nation   |             25 | 2.44 KiB  | 0 bytes    | 2.44 KiB  |
    | test.region   |              5 | 416 bytes | 0 bytes    | 416 bytes |
    | test.part     |         200000 | 25.07 MiB | 0 bytes    | 25.07 MiB |
    | test.supplier |          10000 | 1.45 MiB  | 0 bytes    | 1.45 MiB  |
    | test.partsupp |         800000 | 120.17 MiB| 12.21 MiB  | 132.38 MiB|
    | test.customer |         150000 | 24.77 MiB | 0 bytes    | 24.77 MiB |
    | test.orders   |        1527648 | 174.40 MiB| 0 bytes    | 174.40 MiB|
    | test.lineitem |        6491711 | 849.07 MiB| 99.06 MiB  | 948.13 MiB|
    +---------------+----------------+-----------+------------+-----------+
    8 rows in set (0.06 sec)
    ```

    これは商業注文システムのデータベースです。`test.nation`テーブルは国に関する情報を、`test.region`テーブルは地域に関する情報を、`test.part`テーブルはパーツに関する情報を、`test.supplier`テーブルはサプライヤに関する情報を、`test.partsupp`テーブルはサプライヤのパーツに関する情報を、`test.customer`テーブルは顧客に関する情報を、`test.orders`テーブルは注文に関する情報を、`test.lineitem`テーブルはオンラインアイテムに関する情報を表します。

### 手順3. 行ベースのストレージエンジンを使用してデータをクエリする

行ベースのストレージエンジンのみを使用した場合のTiDBのパフォーマンスを把握するために、次のSQLステートメントを実行します。

{{< copyable "sql" >}}

```sql
SELECT
    l_orderkey,
    SUM(
        l_extendedprice * (1 - l_discount)
    ) AS revenue,
    o_orderdate,
    o_shippriority
FROM
    customer,
    orders,
    lineitem
WHERE
    c_mktsegment = 'BUILDING'
AND c_custkey = o_custkey
AND l_orderkey = o_orderkey
AND o_orderdate < DATE '1996-01-01'
AND l_shipdate > DATE '1996-02-01'
GROUP BY
    l_orderkey,
    o_orderdate,
    o_shippriority
ORDER BY
    revenue DESC,
    o_orderdate
limit 10;
```

これは出荷優先度クエリであり、指定された日付より前に出荷されていない最高収益の注文の優先度と潜在的な収益を提供します。潜在的な収益は、`l_extendedprice * (1-l_discount)`の合計と定義されます。注文は収益の降順でリストされます。この例では、このクエリはトップ10に潜在的な収益がある未出荷注文をリストします。

### 手順4. テストデータを列ベースのストレージエンジンにレプリケートする

TiFlashが展開された後、TiKVはTiFlashにすぐにデータをレプリケートしません。TiDBでMySQLクライアントから以下のDDLステートメントを実行して、レプリケートする必要のあるテーブルを指定します。その後、TiDBはTiFlashに指定されたレプリカを作成します。

{{< copyable "sql" >}}

```sql
ALTER TABLE test.customer SET TIFLASH REPLICA 1;
ALTER TABLE test.orders SET TIFLASH REPLICA 1;
ALTER TABLE test.lineitem SET TIFLASH REPLICA 1;
```

特定のテーブルのレプリケーションステータスを確認するには、次のステートメントを実行します。

{{< copyable "sql" >}}

```sql
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = 'test' and TABLE_NAME = 'customer';
```
```sql
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = 'test' and TABLE_NAME = 'orders';
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = 'test' and TABLE_NAME = 'lineitem';
```

上記のステートメントの結果では、以下のことが示されます：

- `AVAILABLE` は、特定のテーブルの TiFlash レプリカが利用可能かどうかを示します。`1` は利用可能を表し、`0` は利用不可を示します。 `AVAILABLE` フィールドが `1` になると、このステータスは変更されません。
- `PROGRESS` はレプリケーションの進捗を意味します。値は 0.0 から 1.0 の間の範囲です。1 は、TiFlash レプリカのレプリケーション進捗が完了したことを示します。

### ステップ 5. HTAP を使用してデータをより速く分析する

[ステップ 3](#step-3-query-data-with-the-row-based-storage-engine) で SQL ステートメントを実行し、TiDB HTAP のパフォーマンスを確認できます。

TiFlash レプリカを持つテーブルでは、TiDB オプティマイザーはコスト推定に基づいて自動的に TiFlash レプリカを使用するかどうかを判断します。TiFlash レプリカが選択されたかどうかを確認するには、`desc` または `explain analyze` ステートメントを使用できます。例：

{{< copyable "sql" >}}

```sql
explain analyze SELECT
    l_orderkey,
    SUM(
        l_extendedprice * (1 - l_discount)
    ) AS revenue,
    o_orderdate,
    o_shippriority
FROM
    customer,
    orders,
    lineitem
WHERE
    c_mktsegment = 'BUILDING'
AND c_custkey = o_custkey
AND l_orderkey = o_orderkey
AND o_orderdate < DATE '1996-01-01'
AND l_shipdate > DATE '1996-02-01'
GROUP BY
    l_orderkey,
    o_orderdate,
    o_shippriority
ORDER BY
    revenue DESC,
    o_orderdate
limit 10;
```

`EXPLAIN` ステートメントの結果に `ExchangeSender` と `ExchangeReceiver` 演算子が表示される場合、MPP モードが有効になっていることを示します。

さらに、クエリ全体のそれぞれの部分を TiFlash エンジンのみを使用して計算するように指定することができます。詳細については、[Use TiDB to read TiFlash replicas](/tiflash/use-tidb-to-read-tiflash.md) を参照してください。

この2つの方法のクエリ結果とクエリのパフォーマンスを比較できます。

## 次は何ですか

- [TiDB HTAP のアーキテクチャ](/tiflash/tiflash-overview.md#architecture)
- [HTAP を探索する](/explore-htap.md)
- [TiFlash の使用](/tiflash/tiflash-overview.md#use-tiflash)
```