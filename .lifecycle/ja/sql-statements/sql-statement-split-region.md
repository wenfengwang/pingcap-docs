---
title: リージョン分割
summary: TiDBデータベースのSplit Regionの使用概要。
aliases: ['/docs/dev/sql-statements/sql-statement-split-region/','/docs/dev/reference/sql/statements/split-region/']
---

# リージョン分割

TiDBで新しいテーブルを作成するたびに、1つの[Region](/tidb-storage.md#region)は、デフォルトでこのテーブルのデータを格納するためにセグメント化されます。このデフォルトの動作は、TiDB構成ファイルの`split-table`によって制御されます。このリージョン内のデータがデフォルトのリージョンサイズ制限を超えると、リージョンは2つに分割されます。

上記のケースでは、最初は1つのリージョンしかないため、新しく作成されたテーブルのすべての書き込みリクエストは、リージョンが配置されているTiKVで発生します。新しく作成されたテーブルに大量の書き込みがある場合、ホットスポットが生じます。

上記のシナリオでのホットスポット問題を解決するために、TiDBは指定されたパラメータに従って特定のテーブルのために複数のリージョンを事前に分割し、それらを各TiKVノードに分散させるプリスプリット機能を導入しています。

> **注記:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

## 概要

**SplitRegionStmt:**

![SplitRegionStmt](/media/sqlgram/SplitRegionStmt.png)

**SplitSyntaxOption:**

![SplitSyntaxOption](/media/sqlgram/SplitSyntaxOption.png)

**TableName:**

![TableName](/media/sqlgram/TableName.png)

**PartitionNameListOpt:**

![PartitionNameListOpt](/media/sqlgram/PartitionNameListOpt.png)

**SplitOption:**

![SplitOption](/media/sqlgram/SplitOption.png)

**RowValue:**

![RowValue](/media/sqlgram/RowValue.png)

**Int64Num:**

![Int64Num](/media/sqlgram/Int64Num.png)

## リージョン分割の使用

リージョン分割には2種類の構文があります。

- 均等に分割する構文：

    {{< copyable "sql" >}}

    ```sql
    SPLIT TABLE table_name [INDEX index_name] BETWEEN (lower_value) AND (upper_value) REGIONS region_num
    ```

    `BETWEEN lower_value AND upper_value REGIONS region_num`は、上限、下限、およびリージョンの数を定義します。その後、現在のリージョンは、「region_num」で指定された数のリージョンに均等に分割されます。上限と下限の間のリージョン数が均等に分割されます。

- 不均等に分割する構文：

    {{< copyable "sql" >}}

    ```sql
    SPLIT TABLE table_name [INDEX index_name] BY (value_list) [, (value_list)] ...
    ```

    `BY value_list…`は、手動で一連のポイントを指定し、現在のリージョンが分割される基準とします。データが均等に分散されていないシナリオに適しています。

以下の例は`SPLIT`ステートメントの結果を示しています。

```sql
+--------------------+----------------------+
| TOTAL_SPLIT_REGION | SCATTER_FINISH_RATIO |
+--------------------+----------------------+
| 4                  | 1.0                  |
+--------------------+----------------------+
```

* `TOTAL_SPLIT_REGION`: 新たに分割されたリージョンの数。
* `SCATTER_FINISH_RATIO`: 新たに分割されたリージョンの分散率。 `1.0`はすべてのリージョンが分散されたことを示します。 `0.5`は、半分のリージョンが分散されており、残りが分散中であることを示します。

> **注記:**
>
> 次の2つのセッション変数は、`SPLIT`ステートメントの動作に影響を与える可能性があります。
>
> - `tidb_wait_split_region_finish`: リージョンの分散には時間がかかる場合があります。この期間は、PDのスケジューリングとTiKVの負荷に依存します。この変数は、`SPLIT REGION`ステートメントの実行時にすべてのリージョンが分散されるまで結果をクライアントに返すかどうかを制御するために使用されます。その値が`1`（デフォルト）に設定されている場合、TiDBは分散が完了するまで結果を返します。値が`0`に設定されている場合、TiDBは分散の状態にかかわらず結果を返します。
> - `tidb_wait_split_region_timeout`: `SPLIT REGION`ステートメントの実行タイムアウトを秒単位で設定するための変数です。デフォルト値は300秒です。指定された期間内に`split`操作が完了しない場合、TiDBはタイムアウトエラーを返します。

### テーブルリージョンの分割

各テーブルの行データのキーは `table_id` と `row_id` でエンコードされます。フォーマットは次のとおりです。

```go
t[table_id]_r[row_id]
```

たとえば、`table_id` が 22 で、`row_id` が 11 の場合：

```go
t22_r11
```

同じテーブルの行データは同じ`table_id`を持ちますが、それぞれの行に固有の `row_id` を持っており、リージョンの分割に使用できます。

#### 均等分割

`row_id` は整数値なので、`lower_value`、`upper_value`、`region_num`を指定に応じて、分割対象のキーの値を計算できます。TiDBはまずステップ値を計算します（`step = (upper_value - lower_value)/region_num`）。次に、`lower_value` から `upper_value` までの各「step」ごとに均等に分割され、`region_num`で指定されたリージョン数が生成されます。

たとえば、テーブル`t`のキー範囲 `minInt64`～`maxInt64` から均等に16のリージョンを分割したい場合、次のようなステートメントを使用できます：

{{< copyable "sql" >}}

```sql
SPLIT TABLE t BETWEEN (-9223372036854775808) AND (9223372036854775807) REGIONS 16;
```

このステートメントは、テーブル`t`を、minInt64からmaxInt64までの16のリージョンに分割します。指定された主キー範囲が指定されたものよりも小さい場合、たとえば、0〜1000000000の場合、minInt64およびmaxInt64にそれぞれ0および1000000000を使用してリージョンを分割できます。

{{< copyable "sql" >}}

```sql
SPLIT TABLE t BETWEEN (0) AND (1000000000) REGIONS 16;
```

#### 不均等分割

既知のデータが均等に分布されていない場合、リージョンを、-inf〜10000、10000〜90000、および90000〜+infのキー範囲でそれぞれ分割することができます。以下は指定された固定ポイントを設定して、その例を示しています：

{{< copyable "sql" >}}

```sql
SPLIT TABLE t BY (10000), (90000);
```

### インデックスリージョンの分割

テーブルのインデックスデータのキーは、 `table_id`、`index_id`、およびインデックス列の値でエンコードされます。フォーマットは次のとおりです。

```go
t[table_id]_i[index_id][index_value]
```

たとえば、`table_id` が 22、`index_id` が 5、`index_value` が abc の場合：

```go
t22_i5abc
```

同じテーブルのインデックスデータの`table_id`と`index_id`は同じです。インデックスリージョンを分割するには`index_value`に基づいてリージョンを分割する必要があります。

#### 均等分割

インデックスを均等に分割する方法は、データを均等に分割する方法と同じです。ただし、ステップの値を計算するのはより複雑です。というのも、`index_value`は整数でない場合があるからです。

`upper`および`lower`の値はまずバイト配列にエンコードされます。次に、`lower`および`upper`バイト配列の最も長い共通接頭辞を削除した後、`lower`および`upper`の最初の8バイトがuint64形式に変換されます。その後、`step = (upper - lower)/num`が計算されます。その後、計算されたステップは、`lower`および`upper`バイト配列の最長共通接頭辞に追加されたバイト配列にエンコードされ、これがインデックスの分割のためのデータに追加されます。以下は例です：

`idx`インデックスのカラムが整数型の場合、以下のSQLステートメントを使用してインデックスデータを分割できます：

{{< copyable "sql" >}}

```sql
SPLIT TABLE t INDEX idx BETWEEN (-9223372036854775808) AND (9223372036854775807) REGIONS 16;
```

このステートメントは、テーブルtのidxインデックスのリージョンを`minInt64`から`maxInt64`に16個のリージョンに分割します。

インデックスidx1の列がvarchar型の場合、およびインデックスデータをプレフィックスの文字で分割したい場合。

{{< copyable "sql" >}}

```sql
SPLIT TABLE t INDEX idx1 BETWEEN ("a") AND ("z") REGIONS 25;
```

このステートメントでは、インデックスidx1をa ～ zから25個のリージョンに分割します。リージョン1の範囲は `[minIndexValue, b)`です。リージョン2の範囲は `[b, c)`です。...リージョン25の範囲は`[y, minIndexValue]`です。`idx`インデックスの場合、`a`接頭辞のデータはリージョン1に書き込まれ、`b`接頭辞のデータはリージョン2に書き込まれます。

上記の分割方法では、`y`および`z`の接頭辞のデータは両方ともリージョン25に書き込まれます。なぜならば、上限は `z`ではなく`{`（`z`の次の文字）であるためです。そのため、より正確な分割方法は以下のようになります：

{{< copyable "sql" >}}

```sql
SPLIT TABLE t INDEX idx1 BETWEEN ("a") AND ("{") REGIONS 26;
```

このステートメントは、テーブル`t`のインデックス`idx1`を26のリージョンに分割します。リージョン1の範囲は`[minIndexValue, b)`、リージョン2の範囲は`[b, c)`、...、リージョン25の範囲は`[y, z)`、リージョン26の範囲は`[z, maxIndexValue)`となります。

インデックス`idx2`の列がタイムスタンプ/日時などの時間型の場合で、年によってインデックスリージョンを分割したい場合:

{{< copyable "sql" >}}

```sql
SPLIT TABLE t INDEX idx2 BETWEEN ("2010-01-01 00:00:00") AND ("2020-01-01 00:00:00") REGIONS 10;
```

このステートメントは、テーブル`t`のインデックス`idx2`のリージョンを`2010-01-01 00:00:00`から`2020-01-01 00:00:00`まで10のリージョンに分割します。リージョン1の範囲は`[minIndexValue, 2011-01-01 00:00:00)`、リージョン2の範囲は`[2011-01-01 00:00:00, 2012-01-01 00:00:00)`となります。

インデックスリージョンを日単位で分割したい場合は、以下の例を参照してください:

{{< copyable "sql" >}}

```sql
SPLIT TABLE t INDEX idx2 BETWEEN ("2020-06-01 00:00:00") AND ("2020-07-01 00:00:00") REGIONS 30;
```

このステートメントでは、テーブル`t`の`idex2`のレコードを30のリージョンに分割し、各リージョンが1日を表します。

他のタイプのインデックス列のリージョン分割方法は類似しています。

複合インデックスのデータリージョン分割の場合、異なる点は複数の列値を指定できることです。

例えば、インデックス`idx3 (a, b)`には、列`a`がタイムスタンプ型で列`b`がint型の2つの列が含まれています。列`a`による時間範囲の分割のみを行いたい場合は、単一列の時間インデックスを分割するSQLステートメントを使用できます。この場合、`lower_value`と`upper_velue`に列`b`の値を指定しないでください。

{{< copyable "sql" >}}

```sql
SPLIT TABLE t INDEX idx3 BETWEEN ("2010-01-01 00:00:00") AND ("2020-01-01 00:00:00") REGIONS 10;
```

同じ時間範囲内で、列`b`によるさらなる分割を行いたい場合は、分割時に列bの値を指定してください。

{{< copyable "sql" >}}

```sql
SPLIT TABLE t INDEX idx3 BETWEEN ("2010-01-01 00:00:00", "a") AND ("2010-01-01 00:00:00", "z") REGIONS 10;
```

このステートメントでは、列bの値に応じてa~zの範囲を10のリージョンに分割し、列aと同じ時間プレフィックスを持ちます。列aに指定された値が異なる場合、この場合列bの値は使用されない可能性があります。

テーブルの主キーが[クラスタ化されていないインデックス](/clustered-indexes.md)である場合は、リージョンを分割する際に`PRIMARY`キーワードをエスケープするためにバッククォート ``` ` ``` を使用する必要があります。

例:

```sql
SPLIT TABLE t INDEX `PRIMARY` BETWEEN (-9223372036854775808) AND (9223372036854775807) REGIONS 16;
```

#### 不均等な分割

インデックスデータは指定されたインデックス値で分割することもできます。

例えば、`idx4 (a,b)`には、列`a`がvarchar型で列`b`がタイムスタンプ型が含まれています。

{{< copyable "sql" >}}

```sql
SPLIT TABLE t1 INDEX idx4 BY ("a", "2000-01-01 00:00:01"), ("b", "2019-04-17 14:26:19"), ("c", "");
```

このステートメントは4つのリージョンに分割するために3つの値を指定しています。各リージョンの範囲は以下のようになります:

```
region1  [ minIndexValue               , ("a", "2000-01-01 00:00:01"))
region2  [("a", "2000-01-01 00:00:01") , ("b", "2019-04-17 14:26:19"))
region3  [("b", "2019-04-17 14:26:19") , ("c", "")                   )
region4  [("c", "")                    , maxIndexValue               )
```

### パーティションテーブルのリージョン分割

パーティションテーブルのリージョン分割は通常のテーブルのリージョン分割と同じです。唯一の違いは、同じ分割操作が各パーティションに対して実行されることです。

+ 均等な分割の構文:

    {{< copyable "sql" >}}

    ```sql
    SPLIT [PARTITION] TABLE t [PARTITION] [(partition_name_list...)] [INDEX index_name] BETWEEN (lower_value) AND (upper_value) REGIONS region_num
    ```

+ 不均等な分割の構文:

    {{< copyable "sql" >}}

    ```sql
    SPLIT [PARTITION] TABLE table_name [PARTITION (partition_name_list...)] [INDEX index_name] BY (value_list) [, (value_list)] ...
    ```

#### パーティションテーブルのリージョン分割の例

1. パーティションテーブル`t`を作成します。`a`をハッシュによって2つのパーティションに分割するハッシュテーブルを作成したいとします。例文は以下のようになります:

    {{< copyable "sql" >}}

    ```sql
    create table t (a int,b int,index idx(a)) partition by hash(a) partitions 2;
    ```

    テーブル`t`を作成した後、各パーティションに対してリージョンを分割します。このテーブルのリージョンを表示するために `SHOW TABLE REGIONS` 構文を使用します:

    {{< copyable "sql" >}}

    ```sql
    show table t regions;
    ```

    ```sql
    +-----------+-----------+---------+-----------+-----------------+------------------+------------+---------------+------------+----------------------+------------------+
    | REGION_ID | START_KEY | END_KEY | LEADER_ID | LEADER_STORE_ID | PEERS            | SCATTERING | WRITTEN_BYTES | READ_BYTES | APPROXIMATE_SIZE(MB) | APPROXIMATE_KEYS |
    +-----------+-----------+---------+-----------+-----------------+------------------+------------+---------------+------------+----------------------+------------------+
    | 1978      | t_1400_   | t_1401_ | 1979      | 4               | 1979, 1980, 1981 | 0          | 0             | 0          | 1                    | 0                |
    | 6         | t_1401_   |         | 17        | 4               | 17, 18, 21       | 0          | 223           | 0          | 1                    | 0                |
    +-----------+-----------+---------+-----------+-----------------+------------------+------------+---------------+------------+----------------------+------------------+
    ```

2. 各パーティションに対してリージョンを分割するために `SPLIT` 構文を使用します。各パーティションの`[0,10000]`の範囲のデータを4つのリージョンに分割したいとします。例文は以下のようになります:

    {{< copyable "sql" >}}

    ```sql
    split partition table t between (0) and (10000) regions 4;
    ```

    上記のステートメントでは、`0`と`10000`はそれぞれ希望するホットスポットデータを分散させる下限および上限に対応する`row_id`を表しています。

    > **注意:**
    >
    > この例は、ホットスポットデータが均等に分布しているシナリオにのみ適用されます。指定されたデータ範囲内でホットスポットデータが不均等に分布している場合は、[パーティションテーブルのリージョン分割](#パーティションテーブルのリージョン分割)の不均等分割の構文を参照してください。

3. テーブルのリージョンをもう一度表示するために `SHOW TABLE REGIONS` 構文を使用します。この表は、このテーブルが10のリージョンに含まれており、各パーティションごとに5つのリージョンがあり、そのうちの4つは行データであり、1つはインデックスデータです。

    {{< copyable "sql" >}}

    ```sql
    show table t regions;
    ```

    ```sql
    +-----------+---------------+---------------+-----------+-----------------+------------------+------------+---------------+------------+----------------------+------------------+
    | REGION_ID | START_KEY     | END_KEY       | LEADER_ID | LEADER_STORE_ID | PEERS            | SCATTERING | WRITTEN_BYTES | READ_BYTES | APPROXIMATE_SIZE(MB) | APPROXIMATE_KEYS |
    +-----------+---------------+---------------+-----------+-----------------+------------------+------------+---------------+------------+----------------------+------------------+
    | 1998      | t_1400_r      | t_1400_r_2500 | 2001      | 5               | 2000, 2001, 2015 | 0          | 132           | 0          | 1                    | 0                |
    | 2006      | t_1400_r_2500 | t_1400_r_5000 | 2016      | 1               | 2007, 2016, 2017 | 0          | 35            | 0          | 1                    | 0                |
    | 2010      | t_1400_r_5000 | t_1400_r_7500 | 2012      | 2               | 2011, 2012, 2013 | 0          | 35            | 0          | 1                    | 0                |
    | 1978      | t_1400_r_7500 | t_1401_       | 1979      | 4               | 1979, 1980, 1981 | 0          | 621           | 0          | 1                    | 0                |
    | 1982      | t_1400_       | t_1400_r      | 2014      | 3               | 1983, 1984, 2014 | 0          | 35            | 0          | 1                    | 0                |
    | 1990      | t_1401_r      | t_1401_r_2500 | 1992      | 2               | 1991, 1992, 2020 | 0          | 120           | 0          | 1                    | 0                |
    | 1994      | t_1401_r_2500 | t_1401_r_5000 | 1997      | 5               | 1996, 1997, 2021 | 0          | 129           | 0          | 1                    | 0                |
    | 2002      | t_1401_r_5000 | t_1401_r_7500 | 2003      | 4               | 2003, 2023, 2022 | 0          | 141           | 0          | 1                    | 0                |
    | 6         | t_1401_r_7500 |               | 17        | 4               | 17, 18, 21       | 0          | 601           | 0          | 1                    | 0                |
    | 1986      | t_1401_       | t_1401_r      | 1989      | 5               | 1989, 2018, 2019 | 0          | 123           | 0          | 1                    | 0                |
    +-----------+---------------+---------------+-----------+-----------------+------------------+------------+---------------+------------+----------------------+------------------+
    ```

4. 索引`idx`上的`[1000,10000]`范围也可以进行区分。例如，您可以将`idx`索引的`[1000,10000]`范围分为两个区域。示例如下：

    {{< copyable "sql" >}}

    ```sql
    split partition table t index idx between (1000) and (10000) regions 2;
    ```

#### 单个分区的分区拆分示例

您可以指定要拆分的分区。

1. 创建一个分区表。假设您要创建一个Range分区表，分为三个分区。示例如下：

    {{< copyable "sql" >}}

    ```sql
    create table t ( a int, b int, index idx(b)) partition by range( a ) (
        partition p1 values less than (10000),
        partition p2 values less than (20000),
        partition p3 values less than (MAXVALUE) );
    ```

2. 假设您要将`p1`分区中`[0,10000]`范围内的数据分为两个区域。示例如下：

    {{< copyable "sql" >}}

    ```sql
    split partition table t partition (p1) between (0) and (10000) regions 2;
    ```

3. 假设您要将`p2`分区中`[10000,20000]`范围内的数据分为两个区域。示例如下：

    {{< copyable "sql" >}}

    ```sql
    split partition table t partition (p2) between (10000) and (20000) regions 2;
    ```

4. 您可以使用`SHOW TABLE REGIONS`语法查看此表的区域：

    {{< copyable "sql" >}}

    ```sql
    show table t regions;
    ```

    ```sql
    +-----------+----------------+----------------+-----------+-----------------+------------------+------------+---------------+------------+----------------------+------------------+
    | REGION_ID | START_KEY      | END_KEY        | LEADER_ID | LEADER_STORE_ID | PEERS            | SCATTERING | WRITTEN_BYTES | READ_BYTES | APPROXIMATE_SIZE(MB) | APPROXIMATE_KEYS |
    +-----------+----------------+----------------+-----------+-----------------+------------------+------------+---------------+------------+----------------------+------------------+
    | 2040      | t_1406_        | t_1406_r_5000  | 2045      | 3               | 2043, 2045, 2044 | 0          | 0             | 0          | 1                    | 0                |
    | 2032      | t_1406_r_5000  | t_1407_        | 2033      | 4               | 2033, 2034, 2035 | 0          | 0             | 0          | 1                    | 0                |
    | 2046      | t_1407_        | t_1407_r_15000 | 2048      | 2               | 2047, 2048, 2050 | 0          | 35            | 0          | 1                    | 0                |
    | 2036      | t_1407_r_15000 | t_1408_        | 2037      | 4               | 2037, 2038, 2039 | 0          | 0             | 0          | 1                    | 0                |
    | 6         | t_1408_        |                | 17        | 4               | 17, 18, 21       | 0          | 214           | 0          | 1                    | 0                |
    +-----------+----------------+----------------+-----------+-----------------+------------------+------------+---------------+------------+----------------------+------------------+
    ```

5. 假设您要将`p1`和`p2`分区中`idx`索引的`[0,20000]`范围分为两个区域。示例如下：

    {{< copyable "sql" >}}

    ```sql
    split partition table t partition (p1,p2) index idx between (0) and (20000) regions 2;
    ```

## pre_split_regions

要在创建表时将区域均匀拆分，建议您在`PRE_SPLIT_REGIONS`中使用`SHARD_ROW_ID_BITS`。成功创建表后，`PRE_SPLIT_REGIONS`将表预拆分为`2^(PRE_SPLIT_REGIONS)`所指定的区域数。

> **备注：**
>
> `PRE_SPLIT_REGIONS`的值必须小于或等于`SHARD_ROW_ID_BITS`的值。

`tidb_scatter_region`全局变量会影响`PRE_SPLIT_REGIONS`的行为。此变量控制在创建表后是否等待区域进行预拆分和分散后才返回结果。如果在创建表后进行了大量写操作，则需要将此变量的值设置为`1`，这样TiDB在拆分和散列完成前不会将结果返回给客户端。否则，TiDB会在分散完成前写入数据，这对写入性能会产生显著影响。

### pre_split_regions示例

{{< copyable "sql" >}}

```sql
create table t (a int, b int,index idx1(a)) shard_row_id_bits = 4 pre_split_regions=2;
```

构建表后，此语句为表`t`拆分了`4 + 1`个区域。其中`4 (2^2)`个区域用于保存表行数据，1个区域用于保存`idx1`的索引数据。

4个表区域的范围如下：

```
region1:   [ -inf      ,  1<<61 )
region2:   [ 1<<61     ,  2<<61 )
region3:   [ 2<<61     ,  3<<61 )
region4:   [ 3<<61     ,  +inf  )
```

<CustomContent platform="tidb">

> **备注：**
>
> [Region merge](/best-practices/pd-scheduling-best-practices.md#region-merge) スケジューラーによって Split Region 文によって分割されたリージョンが制御されます。PD が新たに分割されたリージョンをすぐに再マージするのを避けるために、リージョンマージ機能に関連する動的に設定項目を変更する必要があります。

</CustomContent>

## MySQL 互換性

この文は、MySQL 構文への TiDB 拡張機能です。

## 関連情報

* [SHOW TABLE REGIONS](/sql-statements/sql-statement-show-table-regions.md)
* セッション変数: [`tidb_scatter_region`](/system-variables.md#tidb_scatter_region), [`tidb_wait_split_region_finish`](/system-variables.md#tidb_wait_split_region_finish), および [`tidb_wait_split_region_timeout`](/system-variables.md#tidb_wait_split_region_timeout)。