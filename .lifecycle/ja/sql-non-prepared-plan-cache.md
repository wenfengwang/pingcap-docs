---
title: SQL非準備実行プランキャッシュ
summary: TiDBにおけるSQL非準備実行プランキャッシュの原則、使用法、および例について学びます。

# SQL非準備実行プランキャッシュ

TiDBは、[`Prepare`/`Execute`ステートメント](/sql-prepared-plan-cache.md)と同様、一部の非`PREPARE`ステートメントに対する実行プランのキャッシングをサポートしています。この機能により、これらのステートメントは最適化フェーズをスキップし、パフォーマンスを向上させることができます。

非準備プランキャッシュを有効にすると、追加のメモリおよびCPUオーバーヘッドが発生し、すべてのシナリオに適しているわけではありません。この機能を有効にするかどうかを決定するためには、[パフォーマンスの利点](#performance-benefits)と[メモリの監視](#monitoring)セクションを参照してください。

## 原則

非準備プランキャッシュはセッションレベルの機能であり、[準備プランキャッシュ](/sql-prepared-plan-cache.md)とキャッシュを共有します。非準備プランキャッシュの基本的な原則は次のとおりです。

1. 非準備プランキャッシュを有効にすると、TiDBはまずクエリを抽象構文木（AST）に基づいてパラメータ化します。例えば、`SELECT * FROM t WHERE b < 10 AND a = 1`は、`SELECT * FROM t WHERE b < ? and a = ?`としてパラメータ化されます。
2. 次に、TiDBはパラメータ化されたクエリを使用してプランキャッシュを検索します。
3. 再利用可能なプランが見つかった場合、それが直接使用され、最適化フェーズがスキップされます。
4. それ以外の場合、オプティマイザは新しいプランを生成し、それをキャッシュに追加して以降のクエリで再利用します。

## 使用法

非準備プランキャッシュを有効または無効にするには、[`tidb_enable_non_prepared_plan_cache`](/system-variables.md#tidb_enable_non_prepared_plan_cache)システム変数を設定することができます。また、非準備プランキャッシュのサイズを制御するには、[`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710)システム変数を使用できます。キャッシュされたプランの数が`tidb_session_plan_cache_size`を超えると、TiDBは最も最近使用された（LRU）の戦略でプランを削除します。

v7.1.0からは、システム変数[`tidb_plan_cache_max_plan_size`](/system-variables.md#tidb_plan_cache_max_plan_size-new-in-v710)を使用して、キャッシュできるプランの最大サイズを制御することができます。デフォルト値は2 MBです。プランのサイズがこの値を超えると、そのプランはキャッシュされません。

> **注記:**
>
> `tidb_session_plan_cache_size`で指定されたメモリは、準備済みおよび非準備プランキャッシュと共有されます。現在のクラスタで準備済みプランキャッシュを有効にしている場合、非準備プランキャッシュを有効にすると、元の準備プランキャッシュの命中率が低下する可能性があります。

## 例

次の例では、非準備プランキャッシュの使用方法を示しています。

1. テスト用のテーブル`t`を作成します。

    ```sql
    CREATE TABLE t (a INT, b INT, KEY(b));
    ```

2. 非準備プランキャッシュを有効にします。

    ```sql
    SET tidb_enable_non_prepared_plan_cache = ON;
    ```

3. 次の2つのクエリを実行します。

    ```sql
    SELECT * FROM t WHERE b < 10 AND a = 1;
    SELECT * FROM t WHERE b < 5 AND a = 2;
    ```

4. 2番目のクエリがキャッシュをヒットするかどうかを確認します。

    ```sql
    SELECT @@last_plan_from_cache;
    ```

    出力の`last_plan_from_cache`の値が`1`であれば、2番目のクエリの実行プランがキャッシュから来ていることを意味します。

    ```sql
    +------------------------+
    | @@last_plan_from_cache |
    +------------------------+
    |                      1 |
    +------------------------+
    1 row in set (0.00 sec)
    ```

## 制限

### 非最適なプランのキャッシング

TiDBは、パラメータ化されたクエリに対して1つのプランのみをキャッシュします。例えば、クエリ`SELECT * FROM t WHERE a < 1`と`SELECT * FROM t WHERE a < 100000`は同じパラメータ化された形`SELECT * FROM t WHERE a < ?`を共有し、したがって同じプランを共有します。

これがパフォーマンスの問題を引き起こす場合、`ignore_plan_cache()`ヒントを使用してキャッシュ内のプランを無視し、オプティマイザが毎回新しい実行プランを生成するようにすることができます。SQLを変更できない場合は、問題を解決するためにバインディングを作成することができます。たとえば、`CREATE BINDING FOR SELECT ... USING SELECT /*+ ignore_plan_cache() */ ...`とします。

### 使用制限

前述のリスクおよび実行プランキャッシュが単純なクエリにのみ重要な利点を提供することから（クエリが複雑で実行に時間がかかる場合、実行プランキャッシュを使用してもあまり役立たないことがあります）、TiDBは非準備プランキャッシュの適用範囲に厳しい制限を設けています。これらの制限は次のとおりです。

- [準備プランキャッシュ](/sql-prepared-plan-cache.md)でサポートされていないクエリやプランも、非準備プランキャッシュではサポートされません。
- `Window`や`Having`などの複雑な演算子を含むクエリはサポートされません。
- 3つ以上の`Join`テーブルやサブクエリを含むクエリはサポートされません。
- `ORDER BY`や`GROUP BY`の直後に数値や式が直接指定されているクエリはサポートされません。たとえば、`ORDER BY 1`や`GROUP BY a+1`はサポートされません。`ORDER BY column_name`や`GROUP BY column_name`のみがサポートされます。
- `JSON`、`ENUM`、`SET`、または`BIT`型の列でフィルタリングされているクエリはサポートされません。たとえば、`SELECT * FROM t WHERE json_col = '{}'`。
- `NULL`値でフィルタリングされているクエリはサポートされません。たとえば、`SELECT * FROM t WHERE a is NULL`。
- パラメータ化後のパラメータが200以上含まれるクエリはデフォルトでサポートされません。たとえば、`SELECT * FROM t WHERE a in (1, 2, 3, ... 201)`。v7.3.0からは、システム変数[`44823`](/optimizer-fix-controls.md#44823-new-in-v730)および[`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v710)を設定することでこの制限を変更できます。
- パーティション分けされたテーブル、仮想列、一時テーブル、ビュー、またはメモリーテーブルにアクセスするクエリはサポートされません。たとえば、`SELECT * FROM INFORMATION_SCHEMA.COLUMNS`のように、`COLUMNS`がTiDBメモリーテーブルである場合。
- ヒントやバインディングを使用するクエリはサポートされません。
- デフォルトでは、DMLステートメントや`FOR UPDATE`句を含む`SELECT`ステートメントはサポートされません。この制限を解除するには、`SET tidb_enable_non_prepared_plan_cache_for_dml = ON`を実行してください。

この機能を有効にすると、オプティマイザはクエリを迅速に評価します。非準備プランキャッシュのサポート条件を満たさない場合、クエリは通常の最適化プロセスにフォールバックします。

## パフォーマンスの利点

内部テストでは、非準備プランキャッシュ機能を有効にすると、ほとんどのTPシナリオで著しいパフォーマンスの利点が得られます。たとえば、TPC-Cテストでは約4%、一部の銀行業務では10%以上、Sysbench RangeScanでは15%のパフォーマンス向上があります。

ただし、この機能はクエリがサポートされるかどうかを判断し、クエリをパラメータ化し、キャッシュ内のプランを検索することなど、追加のメモリおよびCPUオーバーヘッドをもたらします。キャッシュがワークロードの大部分のクエリをヒットできない場合、その機能を有効にすると実際にはパフォーマンスが悪化する可能性があります。

この場合、Grafanaの**Queries Using Plan Cache OPS**パネルでの`non-prepared`メトリクスおよび**Plan Cache Miss OPS**パネルでの`non-prepared-unsupported`メトリクスを観察する必要があります。ほとんどのクエリがサポートされない場合、またはわずかなクエリのみがプランキャッシュをヒットする場合、この機能を無効にすることができます。

![non-prepared-unsupported](/media/non-prepapred-plan-cache-unsupprot.png)

## 診断

非準備プランキャッシュを有効にした後、`EXPLAIN FORMAT='plan_cache' SELECT ...`ステートメントを実行して、クエリがキャッシュをヒットできるかどうかを確認することができます。キャッシュをヒットできないクエリについては、システムは警告を返します。

`FORMAT='plan_cache'`を追加しないと、`EXPLAIN`ステートメントは決してキャッシュをヒットしません。

クエリがキャッシュをヒットするかどうかを確認するには、次の`EXPLAIN FORMAT='plan_cache'`ステートメントを実行してください。

```sql
EXPLAIN FORMAT='plan_cache' SELECT * FROM (SELECT a+1 FROM t1) t;
```

出力は次のとおりです。

```sql
3 rows in set, 1 warning (0.00 sec)
```

キャッシュをヒットしないクエリを表示するには、`SHOW warnings;`を実行してください。

```sql
SHOW warnings;
```

出力は次のとおりです。

```sql
+---------+------+-------------------------------------------------------------------------------+
| Level   | Code | Message                                                                       |
+---------+------+-------------------------------------------------------------------------------+
| Warning | 1105 | skip non-prepared plan-cache: queries that have sub-queries are not supported |
+---------+------+-------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

上記の例では、`+`演算がサポートされないため、クエリがキャッシュをヒットできません。

## モニタリング
非プリペアードプランキャッシュを有効にした後は、次のペインでメモリ使用量、キャッシュ内のプラン数、およびキャッシュヒット率を監視できます。

![non-prepare-plan-cache](/media/tidb-non-prepared-plan-cache-metrics.png)

また、`statements_summary` テーブルと遅いクエリログでもキャッシュヒット率を監視できます。以下は、`statements_summary` テーブルでキャッシュヒット率を表示する方法です。

1. テーブル `t` を作成します：

    ```sql
    CREATE TABLE t (a int);
    ```

2. 非プリペアードプランキャッシュを有効にします：

    ```sql
    SET @@tidb_enable_non_prepared_plan_cache=ON;
    ```

3. 次の3つのクエリを実行します：

    ```sql
    SELECT * FROM t WHERE a<1;
    SELECT * FROM t WHERE a<2;
    SELECT * FROM t WHERE a<3;
    ```

4. `statements_summary` テーブルをクエリして、キャッシュヒット率を表示します：

    ```sql
    SELECT digest_text, query_sample_text, exec_count, plan_in_cache, plan_cache_hits FROM INFORMATION_SCHEMA.STATEMENTS_SUMMARY WHERE query_sample_text LIKE '%SELECT * FROM %';
    ```

    出力は次のようになります：

    ```sql
    +---------------------------------+------------------------------------------+------------+---------------+-----------------+
    | digest_text                     | query_sample_text                        | exec_count | plan_in_cache | plan_cache_hits |
    +---------------------------------+------------------------------------------+------------+---------------+-----------------+
    | SELECT * FROM `t` WHERE `a` < ? | SELECT * FROM t WHERE a<1                |          3 |             1 |               2 |
    +---------------------------------+------------------------------------------+------------+---------------+-----------------+
    1 row in set (0.01 sec)
    ```

    出力から、クエリが3回実行され、キャッシュが2回ヒットしたことがわかります。