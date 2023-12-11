---
title: 統計に関する概要
summary: 統計は、テーブルレベルとカラムレベルの情報を収集する方法を学びます。
aliases: ['/docs/dev/statistics/','/docs/dev/reference/performance/statistics/']
---

# 統計に関する概要

TiDBは統計を使用して[どのインデックスを選択するか](/choose-index.md)を決定します。

## 統計のバージョン

`tidb_analyze_version`変数は、TiDBによって収集された統計を制御します。現在、`tidb_analyze_version = 1`および`tidb_analyze_version = 2`の2つのバージョンの統計がサポートされています。

- TiDB Self-Hostedでは、この変数のデフォルト値はv5.3.0以降で「1」から「2」に変更されます。
- TiDB Cloudでは、この変数のデフォルト値はv6.5.0以降で「1」から「2」に変更されます。
- クラスタが以前のバージョンからアップグレードされた場合、「tidb_analyze_version」のデフォルト値はアップグレード後に変更されません。

バージョン1と比較して、バージョン2の統計は、データボリュームが非常に大きい場合にハッシュ衝突によって引き起こされる潜在的な不正確さを回避します。また、ほとんどのシナリオで推定の精度を維持します。

この2つのバージョンにはTiDBで異なる情報が含まれています。

| 情報 | バージョン1 | バージョン2 |
| --- | --- | ---|
| テーブル内の総行数 | √ | √ |
| Column Count-Min Sketch | √ | × |
| Index Count-Min Sketch | √ | × |
| Column Top-N | √ | √（メンテナンス方法と精度が改善されました） |
| Index Top-N | √（メンテナンスの精度が不足する可能性があり不正確さを引き起こすことがあります） | √（メンテナンス方法と精度が改善されました） |
| Column histogram | √ | √（ヒストグラムにはTop-Nの値が含まれません） |
| Index histogram | √ | √（ヒストグラムバケツは各バケツ内の異なる値の数を記録し、ヒストグラムはTop-Nの値を含みません。） |
| カラム内の「NULL」の数 | √ | √ |
| インデックス内の「NULL」の数 | √ | √ |
| カラムの平均長 | √ | √ |
| インデックスの平均長 | √ | √ |

`tidb_analyze_version = 2`の場合、`ANALYZE`の実行後にメモリオーバーフローが発生した場合は、`tidb_analyze_version = 1`に設定してバージョン1に戻し、次の操作のいずれかを実行する必要があります。

- 手動で`ANALYZE`文を実行する場合は、分析するテーブルごとに手動で分析する必要があります。

    ```sql
    SELECT DISTINCT(CONCAT('ANALYZE TABLE ', table_schema, '.', table_name, ';')) FROM information_schema.tables, mysql.stats_histograms WHERE stats_ver = 2 AND table_id = tidb_table_id;
    ```

- 自動的にTiDBが`ANALYZE`文を実行する場合は、`DROP STATS`文を生成する次の文を実行する必要があります。

    ```sql
    SELECT DISTINCT(CONCAT('DROP STATS ', table_schema, '.', table_name, ';')) FROM information_schema.tables, mysql.stats_histograms WHERE stats_ver = 2 AND table_id = tidb_table_id;
    ```

- 前述の文の結果がコピー＆ペーストするのに長すぎる場合は、結果を一時的なテキストファイルにエクスポートしてから次のようにファイルから実行することができます。

    ```sql
    SELECT DISTINCT ... INTO OUTFILE '/tmp/sql.txt';
    mysql -h ${TiDB_IP} -u user -P ${TIDB_PORT} ... < '/tmp/sql.txt'
    ```

このドキュメントでは、ヒストグラム、Count-Min Sketch、およびTop-Nについて簡単に紹介し、統計の収集とメンテナンスの詳細を説明します。

## ヒストグラム

ヒストグラムは、データの分布のおおよその表現です。値の全範囲を一連のバケットに分割し、各バケットを記述するために単純なデータを使用します。たとえば、バケット内に落ちる値の数などです。TiDBでは、各テーブルの特定の列に対して等深度のヒストグラムが作成されます。等深度のヒストグラムは、範囲クエリを推定するために使用できます。

ここでの「等深度」とは、各バケットに落ちる値の数が可能な限り均等であることを意味します。たとえば、与えられたセット{1.6, 1.9, 1.9, 2.0, 2.4, 2.6, 2.7, 2.7, 2.8, 2.9, 3.4, 3.5}があるとします。4つのバケットを生成したい場合、等深度のヒストグラムは次のようになります。それは4つのバケット[1.6, 1.9]、[2.0, 2.6]、[2.7, 2.8]、[2.9, 3.5]を含んでいます。バケットの深さは3です。

![等深度ヒストグラムの例](/media/statistics-1.png)

ヒストグラムバケットの上限を決定するパラメータの詳細については、[手動収集](#manual-collection)を参照してください。バケットの数が多いほど、ヒストグラムの精度が高くなりますが、高い精度にはメモリリソースの使用コストがかかります。実際のシナリオに応じてこの数値を適切に調整できます。

## Count-Min Sketch

Count-Min Sketchは、ハッシュ構造です。等価クエリに`a = 1`または`IN`クエリ（たとえば、`a in (1, 2, 3)`）が含まれる場合、TiDBはこのデータ構造を使用して推定します。

Count-Min Sketchはハッシュ構造であるため、ハッシュ衝突が発生する場合があります。等価クエリの推定値が実際の値から大きく逸れる場合、より大きい値とより小さい値がハッシュに含まれていると考えられます。この場合、次のようにハッシュ衝突を回避する方法があります。

- `WITH NUM TOPN`パラメータを変更します。TiDBは、高頻度（上位x）のデータを別々に保存し、その他のデータはCount-Min Sketchに保存します。したがって、より大きい値とより小さい値がハッシュに含まれるのを防ぐには、`WITH NUM TOPN`の値を増やすことができます。デフォルト値は20です。最大値は1024です。このパラメータの詳細については、[手動収集](#manual-collection)を参照してください。
- 2つのパラメータ`WITH NUM CMSKETCH DEPTH`および`WITH NUM CMSKETCH WIDTH`を変更します。両方がハッシュバケツの数と衝突確率に影響します。実際のシナリオに応じて、2つのパラメータの値を適切に増やしてハッシュ衝突の発生確率を減らすことができます。ただし、これには統計のメモリ使用量の増加が伴います。TiDBでは、`WITH NUM CMSKETCH DEPTH`のデフォルト値は5で、`WITH NUM CMSKETCH WIDTH`のデフォルト値は2048です。この2つのパラメータの詳細については、[手動収集](#manual-collection)を参照してください。

## Top-Nの値

Top-Nの値は、列またはインデックス内での上位N回発生する値です。TiDBはTop-Nの値と値の発生回数を記録します。

## 統計情報の収集

### 手動収集

現在、TiDBは統計情報をフルコレクションとして収集します。`ANALYZE TABLE`文を実行して統計情報を収集できます。

> **注意:**
>
> - TiDBでの`ANALYZE TABLE`の実行時間は、MySQLまたはInnoDBでのそれよりも長くなります。InnoDBでは、わずかなページのサンプリングのみが行われますが、TiDBでは包括的な統計情報が完全に再構築されます。MySQL用に書かれたスクリプトは、`ANALYZE TABLE`が短命の操作であると誤って想定しているかもしれません。
> - v7.5.0以降は、統計情報の[高速収集機能（`tidb_enable_fast_analyze`）](/system-variables.md#tidb_enable_fast_analyze)と[増分収集機能](https://docs.pingcap.com/tidb/v7.4/statistics#incremental-collection)は非推奨です。

以下の構文を使用してフルコレクションを実行できます。

+ `TableNameList`のすべてのテーブルの統計情報を収集するには：

    {{< copyable "sql" >}}

    ```sql
    ANALYZE TABLE TableNameList [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
    ```

+ `WITH NUM BUCKETS`は、生成されたヒストグラムの最大バケット数を指定します。
+ `WITH NUM TOPN`は、生成された`TOPN`の最大数を指定します。
+ `WITH NUM CMSKETCH DEPTH`は、CM Sketchの深さを指定します。
+ `WITH NUM CMSKETCH WIDTH`は、CM Sketchの幅を指定します。
+ `WITH NUM SAMPLES`は、サンプルの数を指定します。
+ `WITH FLOAT_NUM SAMPLERATE`は、サンプリングレートを指定します。

`WITH NUM SAMPLES`と`WITH FLOAT_NUM SAMPLERATE`は、異なるアルゴリズムでサンプルを収集する2つの方法に対応しています。

- `WITH NUM SAMPLES`は、TiDBの貯水池サンプリングの中間結果セットに冗長な結果を含むため、大きなテーブルではこのメソッドを使用して統計情報を収集することはお勧めしません。

 これはメモリなどのリソースに追加の圧力をかけるために不要な結果を含む可能性のある中間結果セットを含むことが原因です。
- `WITH FLOAT_NUM SAMPLERATE`は、v5.3.0で導入されたサンプリング方法です。値の範囲は`(0, 1]`で、このパラメータはサンプリング率を指定します。TiDBではベルヌーイ・サンプリングの方法で実装されており、これは大きなテーブルのサンプリングに適しており、収集効率とリソース使用の面で優れた性能を発揮します。

v5.3.0以前、TiDBは統計情報を収集するためにリザーバー・サンプリング手法を使用していました。v5.3.0以降では、TiDBバージョン2の統計情報はデフォルトでベルヌーイ・サンプリング手法を使用して統計情報を収集します。リザーバー・サンプリング手法を再利用するには、`WITH NUM SAMPLES`ステートメントを使用することができます。

現在のサンプリング率は、適応アルゴリズムに基づいて計算されています。[`SHOW STATS_META`](/sql-statements/sql-statement-show-stats-meta.md)を使用してテーブル内の行数を観測できる場合は、この行数を100,000行に対応するサンプリング率を計算するために使用できます。この数値を観測できない場合は、[`TABLE_STORAGE_STATS`](/information-schema/information-schema-table-storage-stats.md)テーブルの`TABLE_KEYS`列をもう一つの参照としてサンプリング率を計算するために使用することができます。

<CustomContent platform="tidb">

> **注意:**
>
> 通常、`STATS_META`は`TABLE_KEYS`よりも信頼できます。ただし、[TiDB Lightning](https://docs.pingcap.com/tidb/stable/tidb-lightning-overview)などの手法でデータをインポートした後、`STATS_META`の結果は`0`となります。このような状況を処理するためには、`STATS_META`の結果が`TABLE_KEYS`の結果よりも遥かに小さい場合は、`TABLE_KEYS`を使用してサンプリング率を計算することができます。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> 通常、`STATS_META`は`TABLE_KEYS`よりも信頼できます。ただし、TiDB Cloudコンソールを通じてデータをインポートした後（[サンプルデータのインポート](/tidb-cloud/import-sample-data.md)を参照）、`STATS_META`の結果は`0`となります。このような状況を処理するためには、`STATS_META`の結果が`TABLE_KEYS`の結果よりも遥かに小さい場合は、`TABLE_KEYS`を使用してサンプリング率を計算することができます。

</CustomContent>

#### いくつかのカラムに対して統計情報を収集する

SQLステートメントを実行する際、最適化プログラムは通常、（`WHERE`、`JOIN`、`ORDER BY`、`GROUP BY`ステートメントなどの）いくつかのカラムに対する統計情報のみを使用します。これらのカラムは`PREDICATE COLUMNS`と呼ばれます。

テーブルに多くのカラムがある場合、すべてのカラムに対して統計情報を収集すると大きなオーバーヘッドが発生することがあります。オーバーヘッドを減らすためには、特定のカラムまたは最適化プログラムで使用される`PREDICATE COLUMNS`のみに統計情報を収集することができます。

> **注記:**
>
> - 一部のカラムに対する統計情報の収集は、[`tidb_analyze_version = 2`](/system-variables.md#tidb_analyze_version-new-in-v510)にのみ適用されます。
> - TiDB v7.2.0から、TiDBでは`ANALYZE`コマンドを実行して統計情報を収集する際に、統計情報収集対象となるカラムの種類を示す[`tidb_analyze_skip_column_types`](/system-variables.md#tidb_analyze_skip_column_types-new-in-v720)システム変数が導入されました。

- 特定のカラムに対して統計情報を収集するには、以下の構文を使用します:

    {{< copyable "sql" >}}

    ```sql
    ANALYZE TABLE テーブル名 COLUMNS カラム名リスト [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
    ```

    この構文において、`カラム名リスト`は対象のカラムの名前リストを指定します。複数のカラムを指定する場合は、カンマ`,`でカラム名を区切ってください。例えば、`ANALYZE table t columns a, b`としてください。指定されたテーブルの特定のカラムに対して統計情報を収集するだけでなく、この構文により同時にそのテーブルのインデックスカラムと全てのインデックスに対する統計情報も収集されます。

    > **注記:**
    >
    > 上記の構文は完全な収集です。例えば、この構文によりカラム`a`および`b`に対する統計情報を収集した後、さらにカラム`c`に対する統計情報も収集したい場合は、`ANALYZE table t columns a, b, c`として、追加のカラム`c`を追加するだけでなく`a`、`b`も再度指定する必要があります。

- `PREDICATE COLUMNS`に対して統計情報を収集するには、次の手順を実行してください:

    > **警告:**
    >
    > 現在、`PREDICATE COLUMNS`の統計情報の収集は実験的な機能です。本番環境で使用しないでください。

    1. [`tidb_enable_column_tracking`](/system-variables.md#tidb_enable_column_tracking-new-in-v540)システム変数の値を`ON`に設定して、TiDBが`PREDICATE COLUMNS`を収集するようにします。

        <CustomContent platform="tidb">

        この設定後、TiDBは100 * [`stats-lease`](/tidb-configuration-file.md#stats-lease)ごとに`mysql.column_stats_usage`システムテーブルに`PREDICATE COLUMNS`情報を書き込みます。

        </CustomContent>

        <CustomContent platform="tidb-cloud">

        この設定後、TiDBは300秒ごとに`mysql.column_stats_usage`システムテーブルに`PREDICATE COLUMNS`情報を書き込みます。

        </CustomContent>

    2. ビジネスのクエリパターンが比較的安定した段階で、以下の構文により`PREDICATE COLUMNS`の統計情報を収集してください:

        {{< copyable "sql" >}}

        ```sql
        ANALYZE TABLE テーブル名 PREDICATE COLUMNS [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
        ```

        特定のテーブルにおける`PREDICATE COLUMNS`の統計情報を収集するだけでなく、この構文により同時にそのテーブルのインデックスカラムと全てのインデックスに対する統計情報も収集されます。

        > **注記:**
        >
        > - もし`mysql.column_stats_usage`システムテーブルに対して`PREDICATE COLUMNS`レコードが一切含まれない場合、前述の構文によりそのテーブルの全てのカラムおよび全てのインデックスに対する統計情報が収集されます。
        > - この構文を使用して統計情報を収集した後、新たな種類のSQLクエリを実行する際には最適化プログラムは一時的に古いまたは擬似的なカラム統計情報を使用することがあり、TiDBは次回以降に使用されたカラムに対する統計情報を収集します。

- 全てのカラムおよびインデックスに対して統計情報を収集するには、以下の構文を使用します:

    {{< copyable "sql" >}}

    ```sql
    ANALYZE TABLE テーブル名 ALL COLUMNS [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
    ```

`ANALYZE`ステートメントにおけるカラム構成（`COLUMNS カラム名リスト`、`PREDICATE COLUMNS`、`ALL COLUMNS`を含む）を永続化したい場合は、`tidb_persist_analyze_options`システム変数の値を`ON`に設定し、[ANALYZE構成の永続化](#persist-analyze-configurations)機能を有効にしてください。ANALYZE構成の永続化機能を有効化した後:

- TiDBが自動的に統計情報を収集する場合や、`ANALYZE`ステートメントを実行して統計情報を収集する際にカラム構成を特に指定しない場合、TiDBは統計情報の収集に先立って以前に永続化された構成を引き続き使用します。
- カラム構成を指定して`ANALYZE`ステートメントを複数回手動で実行する場合、TiDBは新しい構成を指定した最新の`ANALYZE`ステートメントにより前に永続化された構成を上書きします。

`PREDICATE COLUMNS`および統計情報が収集されたカラムを特定するには、以下の構文を使用してください:

{{< copyable "sql" >}}

```sql
SHOW COLUMN_STATS_USAGE [ShowLikeまたはWhere];
```

`SHOW COLUMN_STATS_USAGE`ステートメントは以下の6つの列を返します:

| 列名 | 説明            |
| -------- | ------------- |
| `Db_name`  |  データベース名    |
| `Table_name` | テーブル名 |
| `Partition_name` | パーティション名 |
| `Column_name` | カラム名 |
| `Last_used_at` | クエリ最適化でカラム統計情報が使用された最終時刻 |
| `Last_analyzed_at` | カラム統計情報が収集された最終時刻 |

次の例では、`ANALYZE TABLE t PREDICATE COLUMNS;`を実行した後、TiDBはカラム`b`、`c`、`d`に対する統計情報を収集し、カラム`b`は`PREDICATE COLUMN`であり、カラム`c`と`d`はインデックスカラムです。

{{< copyable "sql" >}}

```sql
SET GLOBAL tidb_enable_column_tracking = ON;
Query OK, 0 rows affected (0.00 sec)

CREATE TABLE t (a INT, b INT, c INT, d INT, INDEX idx_c_d(c, d));
Query OK, 0 rows affected (0.00 sec)

-- `b`カラムに関する統計情報をこのクエリで最適化プログラムが使用します。
SELECT * FROM t WHERE b > 1;
Empty set (0.00 sec)
```
-- 一定の時間（100 * stats-lease）を待った後、TiDBは収集した `PREDICATE COLUMNS` を mysql.column_stats_usage に書き込みます。
-- `PREDICATE COLUMNS` を示すには `last_used_at IS NOT NULL` を指定します。
SHOW COLUMN_STATS_USAGE WHERE db_name = 'test' AND table_name = 't' AND last_used_at IS NOT NULL;
+---------+------------+----------------+-------------+---------------------+------------------+
| Db_name | Table_name | Partition_name | Column_name | Last_used_at        | Last_analyzed_at |
+---------+------------+----------------+-------------+---------------------+------------------+
| test    | t          |                | b           | 2022-01-05 17:21:33 | NULL             |
+---------+------------+----------------+-------------+---------------------+------------------+
1 行がセットされました (0.00 sec)

ANALYZE TABLE t PREDICATE COLUMNS;
クエリ OK, 0 行が変更されました, 1 警告が出力されました (0.03 sec)

-- `last_analyzed_at IS NOT NULL` を指定して、統計が収集された列を表示します。
SHOW COLUMN_STATS_USAGE WHERE db_name = 'test' AND table_name = 't' AND last_analyzed_at IS NOT NULL;
+---------+------------+----------------+-------------+---------------------+---------------------+
| Db_name | Table_name | Partition_name | Column_name | Last_used_at        | Last_analyzed_at    |
+---------+------------+----------------+-------------+---------------------+---------------------+
| test    | t          |                | b           | 2022-01-05 17:21:33 | 2022-01-05 17:23:06 |
| test    | t          |                | c           | NULL                | 2022-01-05 17:23:06 |
| test    | t          |                | d           | NULL                | 2022-01-05 17:23:06 |
+---------+------------+----------------+-------------+---------------------+---------------------+
3 行がセットされました (0.00 sec)

#### インデックスの統計情報を収集

`TableName` の `IndexNameList` に含まれるすべてのインデックスの統計情報を収集するには、次の構文を使用します:

{{< copyable "sql" >}}

```sql
ANALYZE TABLE TableName INDEX [IndexNameList] [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
```

`IndexNameList` が空の場合、この構文は `TableName` のすべてのインデックスの統計情報を収集します。

> **注意:**
>
> 統計情報の収集前後が一貫していることを保証するため、`tidb_analyze_version` が `2` の場合、この構文はインデックスだけでなく、テーブル全体（すべての列とインデックスを含む）の統計情報を収集します。

#### パーティションの統計情報を収集

- `TableName` の `PartitionNameList` に含まれるすべてのパーティションの統計情報を収集するには、次の構文を使用します:

    {{< copyable "sql" >}}

    ```sql
    ANALYZE TABLE TableName PARTITION PartitionNameList [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
    ```

- `TableName` の `PartitionNameList` に含まれるすべてのパーティションのインデックス統計情報を収集するには、次の構文を使用します:

    {{< copyable "sql" >}}

    ```sql
    ANALYZE TABLE TableName PARTITION PartitionNameList INDEX [IndexNameList] [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
    ```

- テーブルの一部のパーティションのみについて [統計情報を収集](/statistics.md#collect-statistics-on-some-columns) する必要がある場合は、次の構文を使用します:

    > **警告:**
    >
    > 現在、`PREDICATE COLUMNS` の統計情報の収集は実験的な機能です。本番環境での使用はお勧めしません。

    {{< copyable "sql" >}}

    ```sql
    ANALYZE TABLE TableName PARTITION PartitionNameList [COLUMNS ColumnNameList|PREDICATE COLUMNS|ALL COLUMNS] [WITH NUM BUCKETS|TOPN|CMSKETCH DEPTH|CMSKETCH WIDTH]|[WITH NUM SAMPLES|WITH FLOATNUM SAMPLERATE];
    ```

#### ダイナミックプルーニングモードでのパーティションテーブルの統計情報の収集

[ダイナミックプルーニングモード](/partitioned-table.md#dynamic-pruning-mode) でパーティションテーブルにアクセスする際、TiDB はテーブルレベルの統計情報である GlobalStats を収集します。現在、GlobalStats はすべてのパーティションの統計情報から集約されます。ダイナミックプルーニングモードでは、どのパーティションテーブルの統計情報更新がトリガーされても GlobalStats が更新されます。

> **注意:**
>
> - GlobalStats 更新がトリガーされ、 [`tidb_skip_missing_partition_stats`](/system-variables.md#tidb_skip_missing_partition_stats-new-in-v730) が `OFF` の場合:
>
>     - 一部のパーティションに統計情報がない場合（つまり、まだ分析されたことのない新しいパーティションなど）、 GlobalStats 生成が中断され、パーティションに統計情報がないという警告メッセージが表示されます。
>     - 特定のパーティションで特定の列の統計情報が存在しない場合（これらのパーティションで分析対象の列が異なる列が指定されている）、これらの列の統計情報が集計されると GlobalStats 生成が中断され、特定のパーティションで特定の列の統計情報が存在しないという警告メッセージが表示されます。
>
> - GlobalStats 更新がトリガーされ、 [`tidb_skip_missing_partition_stats`](/system-variables.md#tidb_skip_missing_partition_stats-new-in-v730) が `ON` の場合:
>
>     一部のパーティションですべてまたは一部の列の統計情報が不足している場合、TiDB はこれらの不足しているパーティション統計情報をスキップして GlobalStats を生成するので、GlobalStats 生成に影響を与えません。
>
> - ダイナミックプルーニングモードでは、パーティションとテーブルの Analyze 設定は同じである必要があります。そのため、`ANALYZE TABLE TableName PARTITION PartitionNameList` 文に続いて `COLUMNS` 設定を指定した場合や、 `WITH` に続いて `OPTIONS` 設定を指定した場合、TiDB はこれらを無視して警告を返します。

### 自動更新

<CustomContent platform="tidb">

`INSERT`、 `DELETE`、または `UPDATE` 文を使用すると、TiDB は行数と変更された行数を自動的に更新します。TiDB はこの情報を定期的に保持し、更新サイクルは 20 * [`stats-lease`](/tidb-configuration-file.md#stats-lease) です。 `stats-lease` のデフォルト値は `3s` です。 値を `0` に設定すると、TiDB は統計情報を自動的に更新しません。

</CustomContent>

<CustomContent platform="tidb-cloud">

`INSERT`、 `DELETE`、または `UPDATE` 文を使用すると、TiDB は行数と変更された行数を自動的に更新します。TiDB はこの情報を定期的に保持し、更新サイクルは 20 * `stats-lease` です。 `stats-lease` のデフォルト値は `3s` です。

</CustomContent>

### 関連システム変数

統計情報の自動更新に関連する 3 つのシステム変数は次の通りです:

|  システム変数 | デフォルト値 | 説明 |
|---|---|---|
| [`tidb_auto_analyze_ratio`](/system-variables.md#tidb_auto_analyze_ratio) | 0.5 | 自動更新の閾値値 |
| [`tidb_auto_analyze_start_time`](/system-variables.md#tidb_auto_analyze_start_time) | `00:00 +0000` | TiDB が自動更新を実行できる一日の開始時刻 |
| [`tidb_auto_analyze_end_time`](/system-variables.md#tidb_auto_analyze_end_time)   | `23:59 +0000` | TiDB が自動更新を実行できる一日の終了時刻 |
| [`tidb_auto_analyze_partition_batch_size`](/system-variables.md#tidb_auto_analyze_partition_batch_size-new-in-v640) | `1` | パーティション化されたテーブルの統計情報を自動的に更新するとき（つまり、パーティション化されたテーブルの統計情報を自動的に更新する場合）、TiDB が自動的に分析するパーティションの数 |

行数が `tbl` の総行数に対する変更された行数の比率が `tidb_auto_analyze_ratio` よりも大きい場合、かつ現在の時刻が `tidb_auto_analyze_start_time` から `tidb_auto_analyze_end_time` の間の場合、TiDB は自動的にこのテーブルの統計情報を更新するためにバックグラウンドで `ANALYZE TABLE tbl` 文を実行します。

小さなテーブルで頻繁に少量のデータを変更する状況が自動的に更新をトリガーすることを回避するため、テーブルの行数が 1000 行未満の場合、TiDB はこのようなデータ変更を自動的に更新しません。 テーブルの行数は `SHOW STATS_META` 文を使用して表示できます。

> **注記:**
>
> 現在、自動更新は手動で `ANALYZE` を実行する際に入力された構成項目を記録しません。したがって、`WITH` 構文を使用して `ANALYZE` の収集動作を制御する場合は、定期的なタスクを手動で設定して統計情報を収集する必要があります。

#### 自動更新の無効化

統計情報の自動更新が過剰なリソースを消費し、オンラインアプリケーションの操作に影響を与えることがわかった場合、[`tidb_enable_auto_analyze`](/system-variables.md#tidb_enable_auto_analyze-new-in-v610) システム変数を使用してそれを無効にすることができます。

#### バックグラウンドでの `ANALYZE` タスクの中止

TiDB v6.0以降、TiDBは`KILL`ステートメントを使用してバックグラウンドで実行されている`ANALYZE`タスクを終了する機能をサポートしています。バックグラウンドで実行されている`ANALYZE`タスクが多くのリソースを消費し、アプリケーションに影響を与えている場合、以下の手順で`ANALYZE`タスクを終了できます。

1. 次のSQLステートメントを実行します。

    {{< copyable "sql" >}}

    ```sql
    SHOW ANALYZE STATUS
    ```

    結果の`instance`列と`process_id`列を確認することで、TiDBインスタンスのアドレスとバックグラウンドでの`ANALYZE`タスクの`ID`を取得できます。

2. バックグラウンドで実行されている`ANALYZE`タスクを終了します。

    <CustomContent platform="tidb">

    - [`enable-global-kill`](/tidb-configuration-file.md#enable-global-kill-new-in-v610)が`true`（デフォルトは`true`）の場合、前の手順で取得したバックグラウンドでの`ANALYZE`タスクの`ID`を使用し、`KILL TIDB ${id};`ステートメントを直接実行できます。
    - `enable-global-kill`が`false`の場合、クライアントを使用してバックエンドで`ANALYZE`タスクを実行しているTiDBインスタンスに接続し、`KILL TIDB ${id};`ステートメントを実行する必要があります。クライアントを使用して他のTiDBインスタンスに接続するか、クライアントとTiDBクラスタの間にプロキシがある場合、`KILL`ステートメントではバックグラウンドでの`ANALYZE`タスクを終了できません。

    </CustomContent>

    <CustomContent platform="tidb-cloud">

    `ANALYZE`タスクを終了するには、前の手順で取得したバックグラウンドでの`ANALYZE`タスクの`ID`を使用して`KILL TIDB ${id};`ステートメントを実行できます。

    </CustomContent>

`KILL`ステートメントの詳細については、[`KILL`](/sql-statements/sql-statement-kill.md)を参照してください。

### `ANALYZE`の並列実行を制御する

`ANALYZE`ステートメントを実行する際、システム変数を使用して並列実行を制御し、システムに与える影響を制御できます。

関連するシステム変数の関係は次の通りです。

![analyze_concurrency](/media/analyze_concurrency.png)

`tidb_build_stats_concurrency`、`tidb_build_sampling_stats_concurrency`、`tidb_analyze_partition_concurrency`は上流と下流の関係にあり、前の図に示されているように、実際の合計並列実行数は次の式で計算できます：`tidb_build_stats_concurrency` * (`tidb_build_sampling_stats_concurrency` + `tidb_analyze_partition_concurrency`)。
これらの変数を変更する際は、それぞれの値を同時に考慮する必要があります。これらの変数の値を変更する際は、`tidb_analyze_partition_concurrency`、`tidb_build_sampling_stats_concurrency`、`tidb_build_stats_concurrency`の順で1つずつ調整することを推奨します。また、これらの3つの変数の値が大きいほど、システムに与えるリソースオーバーヘッドが大きくなります。

#### `tidb_build_stats_concurrency`

`ANALYZE`ステートメントを実行する際、タスクは複数の小さなタスクに分割されます。各タスクは1つのカラムまたはインデックスの統計情報のみを処理します。[`tidb_build_stats_concurrency`](/system-variables.md#tidb_build_stats_concurrency)変数を使用して、同時に実行される小さなタスクの数を制御できます。デフォルト値は`2`です。v7.4.0およびそれ以前のバージョンでは、デフォルト値は`4`です。

#### `tidb_build_sampling_stats_concurrency`

通常のカラムを分析する際、[`tidb_build_sampling_stats_concurrency`](/system-variables.md#tidb_build_sampling_stats_concurrency-new-in-v750)を使用してサンプリングタスクの並列実行を制御できます。デフォルト値は`2`です。

#### `tidb_analyze_partition_concurrency`

`ANALYZE`ステートメントを実行する際、パーティション化されたテーブルの統計情報の読み取りと書き込みの並列実行を制御するために、[`tidb_analyze_partition_concurrency`](/system-variables.md#tidb_analyze_partition_concurrency)を使用できます。デフォルト値は`2`です。v7.4.0およびそれ以前のバージョンでは、デフォルト値は`1`です。

#### `tidb_distsql_scan_concurrency`

通常のカラムを分析する際、[`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency)変数を使用して一度に読み取るリージョンの数を制御できます。デフォルト値は`15`です。値を変更するとクエリのパフォーマンスに影響するため、値を慎重に調整してください。

#### `tidb_index_serial_scan_concurrency`

インデックスカラムを分析する際、[`tidb_index_serial_scan_concurrency`](/system-variables.md#tidb_index_serial_scan_concurrency)変数を使用して一度に読み取るリージョンの数を制御できます。デフォルト値は`1`です。値を変更するとクエリのパフォーマンスに影響するため、値を慎重に調整してください。

### `ANALYZE`構成の永続化

v5.4.0以降、TiDBでは一部の`ANALYZE`構成を永続化する機能をサポートしています。この機能により、既存の構成を将来の統計情報収集に簡単に再利用できます。

以下は永続化をサポートする`ANALYZE`構成です：

| 構成 | 対応するANALYZE構文 |
| --- | --- |
| ヒストグラムのバケット数 | WITH NUM BUCKETS |
| Top-Nの数 | WITH NUM TOPN |
| サンプルの数 | WITH NUM SAMPLES |
| サンプリング率 | WITH FLOATNUM SAMPLERATE |
| `ANALYZE`カラムのタイプ | AnalyzeColumnOption ::= ( 'ALL COLUMNS' \| 'PREDICATE COLUMNS' \| 'COLUMNS' ColumnNameList ) |
| `ANALYZE`カラム | ColumnNameList ::= Identifier ( ',' Identifier )* |

#### `ANALYZE`構成の永続化の有効化

<CustomContent platform="tidb">

`ANALYZE`構成の永続化機能はデフォルトで有効になっています（システム変数`tidb_analyze_version`が`2`であり、`tidb_persist_analyze_options`がデフォルトで`ON`に設定されています）。

</CustomContent>

<CustomContent platform="tidb-cloud">

`ANALYZE`構成の永続化機能はデフォルトで無効になっています。この機能を有効にするには、システム変数`tidb_persist_analyze_options`が`ON`に設定されていることを確認し、システム変数`tidb_analyze_version`を`2`に設定してください。

</CustomContent>

この機能を使用して、`ANALYZE`ステートメントを手動で実行する際に指定された永続化構成を記録できます。記録した後、TiDBは次回統計情報を自動更新するか、あるいは永続化構成を指定せずに手動で統計情報を収集する際に、記録された構成に従って統計情報を収集します。

永続化設定が指定された`ANALYZE`ステートメントを複数回手動で実行すると、TiDBは最新の`ANALYZE`ステートメントで指定された新しい構成で以前に記録した永続化構成を上書きします。

#### `ANALYZE`構成の永続化の無効化

`ANALYZE`構成の永続化機能を無効にするには、`tidb_persist_analyze_options`システム変数を`OFF`に設定してください。`ANALYZE`構成の永続化機能は`tidb_analyze_version = 1`には適用されないため、`tidb_analyze_version = 1`に設定すると、この機能を無効にできます。

`ANALYZE`構成の永続化機能を無効にした後も、TiDBは永続化された構成レコードをクリアしません。したがって、この機能を再度有効にした場合、TiDBは以前に記録した永続化構成を使用して引き続き統計情報を収集します。

> **注意:**
>
> `ANALYZE`構成の永続化機能を再度有効にした場合、以前に記録した永続化構成が最新のデータに適用されなくなった場合は、`ANALYZE`ステートメントを手動で実行し、新しい永続化構成を指定する必要があります。

### 統計情報収集のためのメモリクォータ

> **警告:**
>
> 現在、`ANALYZE`メモリクォータは実験的な機能であり、本番環境でメモリ統計情報が正確でない可能性があります。

TiDB v6.1.0以降、システム変数[`tidb_mem_quota_analyze`](/system-variables.md#tidb_mem_quota_analyze-new-in-v610)を使用してTiDBで統計情報収集のためのメモリクォータを制御できます。

`tidb_mem_quota_analyze`の適切な値を設定するには、クラスタのデータサイズを考慮してください。デフォルトのサンプリング率を使用する場合、主な検討事項はカラムの数、カラム値のサイズ、およびTiDBのメモリ構成です。最大値および最小値を構成する際の以下の提案を考慮してください。

> **注意:**
>
> 以下の提案は参考用です。実際のシナリオに基づいて値を構成する必要があります。

- 最小値：最も多くのカラムを持つテーブルから統計情報を収集する際の最大メモリ使用量よりも大きい必要があります。おおよその参考値：デフォルト構成を使用して20個のカラムを持つテーブルから統計情報を収集するとき、最大メモリ使用量は約800 MiBです。デフォルト構成を使用して160個のカラムを持つテーブルから統計情報を収集するとき、最大メモリ使用量は約5 GiBです。
- 最大値：TiDBが統計情報を収集していないときの利用可能なメモリよりも小さくする必要があります。

### `ANALYZE`ステートの表示
```sql
`ANALYZE` ステートメントを実行すると、以下の SQL ステートメントを使用して `ANALYZE` の現在の状態を表示できます。

{{< copyable "sql" >}}

```sql
SHOW ANALYZE STATUS [ShowLikeOrWhere]
```

このステートメントは `ANALYZE` の状態を返します。`ShowLikeOrWhere` を使用して必要な情報をフィルタリングできます。

現在、`SHOW ANALYZE STATUS` ステートメントは以下の11列を返します：

| 列名 | 説明           |
| :-------- | :------------- |
| table_schema  |  データベース名    |
| table_name | テーブル名 |
| partition_name| パーティション名 |
| job_info | タスク情報。インデックスが分析される場合、この情報にはインデックス名が含まれます。`tidb_analyze_version = 2` の場合、この情報にはサンプル率などの構成項目が含まれます。 |
| processed_rows | 分析された行の数 |
| start_time | タスクが開始された時刻 |
| state | タスクの状態 (`pending`, `running`, `finished`, `failed` を含む) |
| fail_reason | タスクが失敗した理由。実行が成功した場合、値は `NULL` となります。 |
| instance | タスクを実行する TiDB インスタンス |
| process_id | タスクを実行するプロセス ID |

TiDB v6.1.0 から、`SHOW ANALYZE STATUS` ステートメントはクラスタレベルのタスクを表示するようになりました。TiDB を再起動しても、このステートメントを使用して再起動前のタスクレコードを表示できます。TiDB v6.1.0 より前では、`SHOW ANALYZE STATUS` ステートメントはインスタンスレベルのタスクのみを表示し、タスクレコードは TiDB を再起動するとクリアされます。

`SHOW ANALYZE STATUS` は最新のタスクレコードのみを表示します。TiDB v6.1.0 以降、システムテーブル `mysql.analyze_jobs` を使用して直近7日間のタスク履歴を表示できます。

[`tidb_mem_quota_analyze`](/system-variables.md#tidb_mem_quota_analyze-new-in-v610) が設定され、TiDB バックグラウンドで自動的に `ANALYZE` タスクが実行され、この閾値よりも多くのメモリを使用した場合、タスクは再試行されます。`SHOW ANALYZE STATUS` ステートメントの出力には、失敗したタスクおよび再試行したタスクが表示されます。

[`tidb_max_auto_analyze_time`](/system-variables.md#tidb_max_auto_analyze_time-new-in-v610) が0よりも大きい値で設定され、TiDB バックグラウンドで自動的に `ANALYZE` タスクが実行され、この閾値よりも長い時間がかかった場合、タスクは中断されます。

```sql
mysql> SHOW ANALYZE STATUS [ShowLikeOrWhere];
+--------------+------------+----------------+-------------------------------------------------------------------------------------------+----------------+---------------------+---------------------+----------+-------------------------------------------------------------------------------|
| Table_schema | Table_name | Partition_name | Job_info                                                                                  | Processed_rows | Start_time          | End_time            | State    | Fail_reason                                                                   |
+--------------+------------+----------------+-------------------------------------------------------------------------------------------+----------------+---------------------+---------------------+----------+-------------------------------------------------------------------------------|
| test         | sbtest1    |                | retry auto analyze table all columns with 100 topn, 0.055 samplerate                      |        2000000 | 2022-05-07 16:41:09 | 2022-05-07 16:41:20 | finished | NULL                                                                          |
| test         | sbtest1    |                | auto analyze table all columns with 100 topn, 0.5 samplerate                              |              0 | 2022-05-07 16:40:50 | 2022-05-07 16:41:09 | failed   | analyze panic due to memory quota exceeds, please try with smaller samplerate |
```

## 統計情報の表示

以下のステートメントを使用して統計情報の状態を表示できます。

### テーブルのメタデータ

`SHOW STATS_META` ステートメントを使用して合計行数と更新された行数を表示できます。

{{< copyable "sql" >}}

```sql
SHOW STATS_META [ShowLikeOrWhere];
```

`ShowLikeOrWhereOpt` の構文は以下のようになります：

![ShowLikeOrWhereOpt](/media/sqlgram/ShowLikeOrWhereOpt.png)

現在、`SHOW STATS_META` ステートメントは以下の6列を返します：

| 列名 | 説明           |
| :-------- | :------------- |
| `db_name`  |  データベース名    |
| `table_name` | テーブル名 |
| `partition_name`| パーティション名 |
| `update_time` | 更新時刻 |
| `modify_count` | 修正された行数 |
| `row_count` | 合計行数 |

> **注意：**
>
> TiDB が DML ステートメントに応じて合計行数および修正された行数を自動的に更新する場合、`update_time` も更新されます。そのため、`update_time` は必ずしも `ANALYZE` ステートメントが実行された最後の時刻を示すわけではありません。

### テーブルの健全状態

`SHOW STATS_HEALTHY` ステートメントを使用してテーブルの健全状態を確認し、統計情報の精度を大まかに推定できます。`modify_count` >= `row_count` の場合、健全状態は0です。`modify_count` < `row_count` の場合、健全状態は (1 - `modify_count`/`row_count`) * 100 となります。

構文は以下のようになります：

{{< copyable "sql" >}}

```sql
SHOW STATS_HEALTHY [ShowLikeOrWhere];
```

`SHOW STATS_HEALTHY` の概要は以下の通りです：

![ShowStatsHealthy](/media/sqlgram/ShowStatsHealthy.png)

現在、`SHOW STATS_HEALTHY` ステートメントは以下の4列を返します：

| 列名 | 説明           |
| :-------- | :------------- |
| `db_name`  |  データベース名    |
| `table_name` | テーブル名 |
| `partition_name`| パーティション名 |
| `healthy` | テーブルの健全状態 |

### 列のメタデータ

`SHOW STATS_HISTOGRAMS` ステートメントを使用して、すべての列における異なる値の数と `NULL` の数を表示できます。

以下の構文を使用します：

{{< copyable "sql" >}}

```sql
SHOW STATS_HISTOGRAMS [ShowLikeOrWhere]
```

このステートメントはすべての列における異なる値の数と `NULL` の数を返します。`ShowLikeOrWhere` を使用して必要な情報をフィルタリングできます。

現在、`SHOW STATS_HISTOGRAMS` ステートメントは以下の10列を返します：

| 列名 | 説明           |
| :-------- | :------------- |
| `db_name`  |  データベース名    |
| `table_name` | テーブル名 |
| `partition_name`| パーティション名 |
| `column_name` | 列名 (`is_index` が `0` の場合) またはインデックス名 (`is_index` が `1` の場合) |
| `is_index` | インデックス列であるかどうか |
| `update_time` | 更新時刻 |
| `distinct_count` | 異なる値の数 |
| `null_count` | `NULL` の数 |
| `avg_col_size` | 列の平均長 |
| correlation | 列と整数プライマリキーのピアソン相関係数。この値は2つの列間の関連度を示します|

### ヒストグラムのバケット

`SHOW STATS_BUCKETS` ステートメントを使用して、ヒストグラムの各バケットを表示できます。

以下の構文を使用します：

{{< copyable "sql" >}}

```sql
SHOW STATS_BUCKETS [ShowLikeOrWhere]
```

次のダイアグラムが表示されます：

![SHOW STATS_BUCKETS](/media/sqlgram/SHOW_STATS_BUCKETS.png)

このステートメントはすべてのバケットの情報を返します。`ShowLikeOrWhere` を使用して必要な情報をフィルタリングできます。

現在、`SHOW STATS_BUCKETS` ステートメントは以下の11列を返します：

| 列名 | 説明           |
| :-------- | :------------- |
| `db_name`  |  データベース名    |
| `table_name` | テーブル名 |
| `partition_name`| パーティション名 |
| `column_name` | 列名 (`is_index` が `0` の場合) またはインデックス名 (`is_index` が `1` の場合) |
| `is_index` | インデックス列であるかどうか |
| `bucket_id` | バケットの ID |
| `count` | バケットと前のバケットに含まれるすべての値の数 |
| `repeats` | 最大値の発生回数 |
| `lower_bound` | 最小値 |
| `upper_bound` | 最大値 |
| `ndv` | バケット内の異なる値の数。`tidb_analyze_version` = `1` の場合、`ndv` は常に `0` であり、実際の意味を持ちません。 |

### Top-N 情報

`SHOW STATS_TOPN` ステートメントを使用して、TiDB が現在収集している Top-N 情報を表示できます。

以下の構文を使用します：

{{< copyable "sql" >}}

```sql
SHOW STATS_TOPN [ShowLikeOrWhere];
```

現在、`SHOW STATS_TOPN` ステートメントは以下の7列を返します：

| 列名 | 説明 |
| ---- | ----|
| `db_name` | データベース名 |
| `table_name` | テーブル名 |
| `partition_name` | パーティション名 |
| `column_name` | カラム名（`is_index`が`0`の場合）またはインデックス名（`is_index`が`1`の場合）|
| `is_index` | インデックスのカラムであるかどうか |
| `value` | このカラムの値 |
| `count` | 値が出現する回数 |

## 統計情報の削除

`DROP STATS`ステートメントを実行して、統計情報を削除できます。

{{< copyable "sql" >}}

```sql
DROP STATS TableName
```

前述のステートメントは`TableName`のすべての統計情報を削除します。パーティションされたテーブルが指定された場合、このステートメントはこのテーブルのすべてのパーティションの統計情報およびダイナミックプルーニングモードで生成されたGlobalStatsを削除します。

{{< copyable "sql" >}}

```sql
DROP STATS TableName PARTITION PartitionNameList;
```

前述のステートメントは`PartitionNameList`に指定されたパーティションの統計情報のみを削除します。

{{< copyable "sql" >}}

```sql
DROP STATS TableName GLOBAL;
```

前述のステートメントは指定されたテーブルのダイナミックプルーニングモードで生成されたGlobalStatsのみを削除します。

## 統計情報のロード

> **注意:**
>
> 統計情報のロードは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。

デフォルトでは、TiDBはカラムの統計情報のサイズに応じて、以下のように異なる方法で統計情報をロードします。

- メモリを少量使用する統計情報（count、distinctCount、nullCountなど）の場合、カラムデータが更新されると、TiDBは自動的に対応する統計情報をメモリにロードして、SQLの最適化段階で使用します。
- 大量のメモリを使用する統計情報（ヒストグラム、TopN、Count-Min Sketchなど）の場合、SQLの実行パフォーマンスを確保するために、TiDBは必要に応じて統計情報を非同期にロードします。例えば、ヒストグラムの統計情報をカラムに対して使用する際、オプティマイザが対象のカラムのヒストグラム統計情報を使用するときのみ、TiDBはメモリにヒストグラム統計情報をロードします。オンデマンドの非同期統計情報のロードはSQLの実行パフォーマンスに影響を与えませんが、SQLの最適化のために不完全な統計情報が提供されることがあります。

v5.4.0から、TiDBは統計情報を同期的にロードする機能を導入しました。この機能により、SQLステートメントを実行する際に大容量の統計情報（ヒストグラム、TopN、Count-Min Sketchなど）をメモリに同期的にロードできるため、SQLの最適化のための統計情報の完全性が向上します。

この機能を有効にするには、[`tidb_stats_load_sync_wait`](/system-variables.md#tidb_stats_load_sync_wait-new-in-v540)システム変数の値をタイムアウト（ミリ秒単位）に設定します。この変数のデフォルト値は`100`で、この値は機能が有効になっていることを示します。

<CustomContent platform="tidb">

同期的な統計情報のロード機能を有効にした後、次のように機能をさらに構成できます。

- SQLの最適化の待機時間がタイムアウトに達したときにTiDBがどのように動作するかを制御するには、[`tidb_stats_load_pseudo_timeout`](/system-variables.md#tidb_stats_load_pseudo_timeout-new-in-v540)システム変数の値を変更します。この変数のデフォルト値は`ON`で、タイムアウト後、SQLの最適化プロセスはカラムのヒストグラム、TopN、またはCMSketchの統計情報を使用しません。この変数を`OFF`に設定すると、タイムアウト後、SQLの実行が失敗します。
- 同期的な統計情報のロード機能が同時に処理できる最大カラム数を指定するために、TiDB設定ファイルの[`stats-load-concurrency`](/tidb-configuration-file.md#stats-load-concurrency-new-in-v540)オプションの値を変更します。デフォルト値は`5`です。
- 同期的な統計情報のロード機能がキャッシュできる最大カラムリクエスト数を指定するために、TiDB設定ファイルの[`stats-load-queue-size`](/tidb-configuration-file.md#stats-load-queue-size-new-in-v540)オプションの値を変更します。デフォルト値は`1000`です。

TiDBの起動中、初期統計情報の完全なロードが行われる前に実行されるSQLステートメントは、最適な実行プランを持たない場合があり、パフォーマンスの問題を引き起こす可能性があります。このような問題を回避するために、TiDB v7.1.0では[`force-init-stats`](/tidb-configuration-file.md#force-init-stats-new-in-v710)構成パラメータを導入しました。このオプションを使用すると、TiDBは起動中に初期化が完了した統計情報だけが提供されるように制御できます。v7.2.0からは、このパラメータがデフォルトで有効になっています。

v7.1.0から、TiDBは軽量な統計情報の初期化に[`lite-init-stats`](/tidb-configuration-file.md#lite-init-stats-new-in-v710)を導入しました。

- `lite-init-stats`の値が`true`の場合、統計情報の初期化で、インデックスやカラムのヒストグラム、TopN、Count-Min Sketchをメモリにロードしません。
- `lite-init-stats`の値が`false`の場合、統計情報の初期化で、インデックスやプライマリキーのヒストグラム、TopN、Count-Min Sketchをメモリにロードしますが、非プライマリキーのカラムのヒストグラム、TopN、Count-Min Sketchをメモリにロードしません。オプティマイザが特定のインデックスやカラムのヒストグラム、TopN、Count-Min Sketchが必要な場合、必要な統計情報が同期的または非同期的にメモリにロードされます。

`lite-init-stats`のデフォルト値は`true`であり、軽量な統計情報の初期化が有効になっています。`lite-init-stats`を`true`に設定することで、統計情報の初期化が高速化され、不必要な統計情報のロードを避けることにより、TiDBのメモリ使用量が減少します。

</CustomContent>

<CustomContent platform="tidb-cloud">

同期的な統計情報のロード機能を有効にした後、TiDBのSQL最適化の待機時間がタイムアウトに達したときにTiDBがどのように動作するかを、[`tidb_stats_load_pseudo_timeout`](/system-variables.md#tidb_stats_load_pseudo_timeout-new-in-v540)システム変数の値を変更することで制御できます。この変数のデフォルト値は`ON`で、タイムアウト後、SQLの最適化プロセスはカラムのヒストグラム、TopN、またはCMSketchの統計情報を使用しません。この変数を`OFF`に設定すると、タイムアウト後、SQLの実行が失敗します。

</CustomContent>

## 統計情報のエクスポートとインポート

<CustomContent platform="tidb-cloud">

> **注意:**
>
> このセクションはTiDB Cloudには適用されません。

</CustomContent>

### 統計情報のエクスポート

統計情報をエクスポートするためのインターフェースは次の通りです。

+ `${db_name}`データベース内の`${table_name}`テーブルのJSON形式統計情報を取得するために:

    {{< copyable "" >}}

    ```
    http://${tidb-server-ip}:${tidb-server-status-port}/stats/dump/${db_name}/${table_name}
    ```

    例:

    {{< copyable "" >}}

    ```
    curl -s http://127.0.0.1:10080/stats/dump/test/t1 -o /tmp/t1.json
    ```

+ 特定の時刻に`${db_name}`データベース内の`${table_name}`テーブルのJSON形式統計情報を取得するために:

    {{< copyable "" >}}

    ```
    http://${tidb-server-ip}:${tidb-server-status-port}/stats/dump/${db_name}/${table_name}/${yyyyMMddHHmmss}
    ```

### 統計情報のインポート

> **注意:**
>
> MySQLクライアントを起動する際に、`--local-infile=1`オプションを使用してください。

一般的に、インポートされる統計情報とは、エクスポートインターフェースを使用して取得したJSONファイルを指します。

構文:

{{< copyable "sql" >}}

```
LOAD STATS 'file_name'
```

`file_name`はインポートする統計情報のファイル名です。

## 統計情報のロック

> **警告:**
>
> 統計情報のロックは現在のバージョンでの実験的な機能です。本番環境で使用することはお勧めしません。

v6.5.0から、TiDBは統計情報のロックをサポートしています。テーブルまたはパーティションの統計情報がロックされた場合、そのテーブルの統計情報は変更できず、テーブルに対して`ANALYZE`ステートメントを実行できません。例えば:

表`t`を作成し、そこにデータを挿入します。表`t`の統計情報がロックされていない場合、`ANALYZE`ステートメントを正常に実行できます。

```sql
mysql> CREATE TABLE t(a INT, b INT);
Query OK, 0 rows affected (0.03 sec)

mysql> INSERT INTO t VALUES (1,2), (3,4), (5,6), (7,8);
Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> ANALYZE TABLE t;
Query OK, 0 rows affected, 1 warning (0.02 sec)

mysql> SHOW WARNINGS;
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                                                                               |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Note  | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t, reason to use this rate is "Row count in stats_meta is much smaller compared with the row count got by PD, use min(1, 15000/4) as the sample-rate=1" |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

表`t`の統計情報をロックし、`ANALYZE`を実行します。警告メッセージには、`ANALYZE`ステートメントが表`t`をスキップしたことが表示されます。

```sql
mysql> LOCK STATS t;
クエリが実行されました。行に影響を与えません (0.00 秒)

mysql> SHOW STATS_LOCKED;
+---------+------------+----------------+--------+
| Db_name | Table_name | Partition_name | Status |
+---------+------------+----------------+--------+
| test    | t          |                | locked |
+---------+------------+----------------+--------+
1 行をセット (0.01 秒)

mysql> ANALYZE TABLE t;
クエリが実行されました。行に影響を与えません。警告が 2 つあります (0.00 秒)

mysql> SHOW WARNINGS;
+---------+------+-----------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                 |
+---------+------+-----------------------------------------------------------------------------------------------------------------------------------------+
| Note    | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t, reason to use this rate is "use min(1, 110000/8) as the sample-rate=1" |
| Warning | 1105 | skip analyze locked table: test.t                                                                                                       |
+---------+------+-----------------------------------------------------------------------------------------------------------------------------------------+
2 行をセット (0.00 秒)
```

表`t`の統計情報をアンロックし、`ANALYZE`を再度正常に実行できます。

```sql
mysql> UNLOCK STATS t;
クエリが実行されました。行に影響を与えません (0.01 秒)

mysql> ANALYZE TABLE t;
クエリが実行されました。行に影響を与えません。警告が 1 つあります (0.03 秒)

mysql> SHOW WARNINGS;
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                 |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------+
| Note  | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t, reason to use this rate is "use min(1, 110000/8) as the sample-rate=1" |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------+
1 行をセット (0.00 秒)
```

加えて、`LOCK STATS`を使用してパーティションの統計情報をロックすることもできます。例えば：

パーティションテーブル`t`を作成し、データを挿入します。パーティション`p1`の統計情報がロックされていない場合、`ANALYZE`ステートメントを正常に実行できます。

```sql
mysql> CREATE TABLE t(a INT, b INT) PARTITION BY RANGE (a) (PARTITION p0 VALUES LESS THAN (10), PARTITION p1 VALUES LESS THAN (20), PARTITION p2 VALUES LESS THAN (30));
クエリが実行されました。行に影響を与えません (0.03 秒)

mysql> INSERT INTO t VALUES (1,2), (3,4), (5,6), (7,8);
クエリが実行されました。行に影響を与えました (0.00 秒)
レコード: 4  重複: 0  警告: 0

mysql> ANALYZE TABLE t;
クエリが実行されました。行に影響を与えません。警告が 6 つあります (0.02 秒)

mysql> SHOW WARNINGS;
+---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                                                                                                              |
+---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Warning | 1105 | disable dynamic pruning due to t has no global stats                                                                                                                                                                                 |
| Note    | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t's partition p0, reason to use this rate is "Row count in stats_meta is much smaller compared with the row count got by PD, use min(1, 15000/4) as the sample-rate=1" |
| Warning | 1105 | disable dynamic pruning due to t has no global stats                                                                                                                                                                                 |
| Note    | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t's partition p1, reason to use this rate is "TiDB assumes that the table is empty, use sample-rate=1"                                                                 |
| Warning | 1105 | disable dynamic pruning due to t has no global stats                                                                                                                                                                                 |
| Note    | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t's partition p2, reason to use this rate is "TiDB assumes that the table is empty, use sample-rate=1"                                                                 |
+---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
6 行をセット (0.01 秒)
```

パーティション`p1`の統計情報をロックし、`ANALYZE`を実行します。警告メッセージには、`ANALYZE`ステートメントがパーティション`p1`をスキップしたことが表示されます。

```sql
mysql> LOCK STATS t PARTITION p1;
クエリが実行されました。行に影響を与えません (0.00 秒)

mysql> SHOW STATS_LOCKED;
+---------+------------+----------------+--------+
| Db_name | Table_name | Partition_name | Status |
+---------+------------+----------------+--------+
| test    | t          | p1             | locked |
+---------+------------+----------------+--------+
1 行をセット (0.00 秒)

mysql> ANALYZE TABLE t PARTITION p1;
クエリが実行されました。行に影響を与えません。警告が 2 つあります (0.01 秒)

mysql> SHOW WARNINGS;
+---------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                                              |
+---------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Note    | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t's partition p1, reason to use this rate is "TiDB assumes that the table is empty, use sample-rate=1" |
| Warning | 1105 | skip analyze locked table: test.t partition (p1)                                                                                                                     |
+---------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
2 行をセット (0.00 秒)
```

パーティション`p1`の統計情報をアンロックし、`ANALYZE`を再度正常に実行できます。

```sql
mysql> UNLOCK STATS t PARTITION p1;
クエリが実行されました。行に影響を与えません (0.00 秒)

mysql> ANALYZE TABLE t PARTITION p1;
クエリが実行されました。行に影響を与えません。警告が 1 つあります (0.01 秒)

mysql> SHOW WARNINGS;
+-------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                              |
+-------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Note  | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t's partition p1, reason to use this rate is "TiDB assumes that the table is empty, use sample-rate=1" |
+-------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 行をセット (0.00 秒)
```

### 統計情報のロック動作

* パーティションされたテーブルの統計情報をロックすると、パーティションされたテーブルのすべてのパーティションの統計情報がロックされます。
* テーブルまたはパーティションを切り捨てると、テーブルまたはパーティションの統計情報ロックが解除されます。

以下の表は、統計情報をロックする動作を説明しています。

| | テーブル全体を削除 | テーブル全体を切り捨て | パーティションを切り捨て | 新しいパーティションを作成 | パーティションを削除 | パーティションを再構成 | パーティションを交換 |
|----------------------------|------------|----------------------------------------------------------------|----------------------------------------------------------------|----------------|----------------------------------------------|----------------------------------------------|--------------------------|
| パーティションされていないテーブルがロックされている | ロックは無効 | ロックは無効（TiDBは古いテーブルを削除するため、ロック情報も削除される） | / | / | / | / | / |
| パーティションされたテーブルかつテーブル全体がロックされている | ロックは無効 | ロックは無効（TiDBは古いテーブルを削除するため、ロック情報も削除される） | 古いパーティションのロック情報は無効になり、新しいパーティションが自動的にロックされる | 新しいパーティションが自動的にロックされます | 削除されたパーティションのロック情報がクリアされ、テーブル全体のロックが引き続き有効になります | 削除されたパーティションのロック情報がクリアされ、新しいパーティションが自動的にロックされます | ロック情報は交換されたテーブルに移され、新しいパーティションが自動的にロックされます |
| パーティションされたテーブルかつ一部のパーティションのみがロックされている | ロックは無効 | ロックは無効（TiDBは古いテーブルを削除するため、ロック情報も削除される） | ロックは無効（TiDBは古いテーブルを削除するため、ロック情報も削除される） | / | 削除されたパーティションのロック情報がクリアされます | 削除されたパーティションのロック情報がクリアされます | ロック情報は交換されたテーブルに移されます |

## 関連情報

<CustomContent platform="tidb">

* [LOAD STATS](/sql-statements/sql-statement-load-stats.md)
* [DROP STATS](/sql-statements/sql-statement-drop-stats.md)
* [LOCK STATS](/sql-statements/sql-statement-lock-stats.md)
* [UNLOCK STATS](/sql-statements/sql-statement-unlock-stats.md)
* [SHOW STATS_LOCKED](/sql-statements/sql-statement-show-stats-locked.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

* [LOAD STATS](/sql-statements/sql-statement-load-stats.md)
* [LOCK STATS](/sql-statements/sql-statement-lock-stats.md)
* [UNLOCK STATS](/sql-statements/sql-statement-unlock-stats.md)
* [SHOW STATS_LOCKED](/sql-statements/sql-statement-show-stats-locked.md)

</CustomContent>