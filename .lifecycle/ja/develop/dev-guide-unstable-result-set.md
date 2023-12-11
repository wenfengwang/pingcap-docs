---
title: 不安定な結果セット
summary: 不安定な結果セットのエラーを処理する方法を学びます。

# 不安定な結果セット

この文書では、不安定な結果セットのエラーを解決する方法について説明します。

## GROUP BY

利便性のため、MySQLは`GROUP BY`構文を"拡張"して、`SELECT`句が`GROUP BY`句で宣言されていない非集約フィールドを参照できるようにします。つまり、`NON-FULL GROUP BY`構文です。他のデータベースでは、これは不安定な結果セットを引き起こすため、構文**_ERROR_**と見なされます。

例えば、次の2つのテーブルがあるとします。

- `stu_info`は学生の情報を格納します
- `stu_score`は学生のテストの得点を格納します

すると、次のようなSQLクエリ文を書くことができます。

```sql
SELECT
    `a`.`class`,
    `a`.`stuname`,
    max( `b`.`courscore` )
FROM
    `stu_info` `a`
    JOIN `stu_score` `b` ON `a`.`stuno` = `b`.`stuno`
GROUP BY
    `a`.`class`,
    `a`.`stuname`
ORDER BY
    `a`.`class`,
    `a`.`stuname`;
```

結果：

```sql
+------------+--------------+------------------+
| class      | stuname      | max(b.courscore) |
+------------+--------------+------------------+
| 2018_CS_01 | MonkeyDLuffy |             95.5 |
| 2018_CS_03 | PatrickStar  |             99.0 |
| 2018_CS_03 | SpongeBob    |             95.0 |
+------------+--------------+------------------+
3 rows in set (0.00 sec)
```

`a`.`class`と`a`.`stuname`フィールドは`GROUP BY`句で指定されており、選択された列は`a`.`class`、`a`.`stuname`、`b`.`courscore`です。`GROUP BY`条件にない唯一の列である`b`.`courscore`も`max()`関数を使用して一意の値で指定されています。このSQL文を満たす結果は**_ONLY ONE_**であり、曖昧さがないため、これは`FULL GROUP BY`構文と呼ばれます。

反例は`NON-FULL GROUP BY`構文です。例えば、これらの2つのテーブルで、次のSQLクエリを書きます（`GROUP BY`で`a`.`stuname`を削除）。

```sql
SELECT
    `a`.`class`,
    `a`.`stuname`,
    max( `b`.`courscore` )
FROM
    `stu_info` `a`
    JOIN `stu_score` `b` ON `a`.`stuno` = `b`.`stuno`
GROUP BY
    `a`.`class`
ORDER BY
    `a`.`class`,
    `a`.`stuname`;
```

その結果、このSQLに一致する2つの値が返されます。

最初に返される値：

```sql
+------------+--------------+------------------------+
| class      | stuname      | max( `b`.`courscore` ) |
+------------+--------------+------------------------+
| 2018_CS_01 | MonkeyDLuffy |                   95.5 |
| 2018_CS_03 | PatrickStar  |                   99.0 |
+------------+--------------+------------------------+
```

2番目に返される値：

```sql
+------------+--------------+------------------+
| class      | stuname      | max(b.courscore) |
+------------+--------------+------------------+
| 2018_CS_01 | MonkeyDLuffy |             95.5 |
| 2018_CS_03 | SpongeBob    |             99.0 |
+------------+--------------+------------------+
```

`a`.`stuname`フィールドの値をSQLでどのように取得するかを指定していないため、2つの結果があり、SQLのセマンティクスによって両方が満たされるため、不安定な結果セットになります。そのため、`GROUP BY`句の結果セットの安定性を保証したい場合は、`FULL GROUP BY`構文を使用してください。

MySQLは`FULL GROUP BY`構文をチェックするかどうかを制御する`sql_mode`スイッチ`ONLY_FULL_GROUP_BY`を提供しています。TiDBもこの`sql_mode`スイッチに対応しています。

```sql
mysql> select a.class, a.stuname, max(b.courscore) from stu_info a join stu_score b on a.stuno=b.stuno group by a.class order by a.class, a.stuname;
+------------+--------------+------------------+
| class      | stuname      | max(b.courscore) |
+------------+--------------+------------------+
| 2018_CS_01 | MonkeyDLuffy |             95.5 |
| 2018_CS_03 | PatrickStar  |             99.0 |
+------------+--------------+------------------+
2 rows in set (0.01 sec)

mysql> set @@sql_mode='STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ONLY_FULL_GROUP_BY';
Query OK, 0 rows affected (0.01 sec)

mysql> select a.class, a.stuname, max(b.courscore) from stu_info a join stu_score b on a.stuno=b.stuno group by a.class order by a.class, a.stuname;
ERROR 1055 (42000): Expression #2 of ORDER BY is not in GROUP BY clause and contains nonaggregated column '' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

**実行結果**: 上記の例は`sql_mode`に`ONLY_FULL_GROUP_BY`を設定した場合の効果を示しています。

## ORDER BY

SQLの意味論では、`ORDER BY`構文を使用しない場合、結果セットは順不同で出力されます。単一インスタンスのデータベースでは、データが1つのサーバーに格納されているため、複数回の実行結果はデータの再構成なしでも安定することがしばしばあります。一部のデータベース（特にMySQL InnoDBストレージエンジン）は、主キーまたはインデックスの順に結果セットを出力できることさえあります。

分散データベースであるTiDBは、データを複数のサーバーに格納します。さらに、TiDBレイヤーはデータページをキャッシュしないため、`ORDER BY`を使用しないSQL文の結果セットの順序は不安定として知覚されます。連続した結果セットを出力するには、`ORDER BY`句に明示的に順序フィールドを追加する必要があり、これはSQLの意味に準拠します。

次の例では、`ORDER BY`句に1つのフィールドのみが追加され、TiDBはその1つのフィールドで結果を並べ替えます。

```sql
mysql> select a.class, a.stuname, b.course, b.courscore from stu_info a join stu_score b on a.stuno=b.stuno order by a.class;
+------------+--------------+-------------------------+-----------+
| class      | stuname      | course                  | courscore |
+------------+--------------+-------------------------+-----------+
| 2018_CS_01 | MonkeyDLuffy | PrinciplesofDatabase    |      60.5 |
| 2018_CS_01 | MonkeyDLuffy | English                 |      43.0 |
| 2018_CS_01 | MonkeyDLuffy | OpSwimming              |      67.0 |
| 2018_CS_01 | MonkeyDLuffy | OpFencing               |      76.0 |
| 2018_CS_01 | MonkeyDLuffy | FundamentalsofCompiling |      88.0 |
| 2018_CS_01 | MonkeyDLuffy | OperatingSystem         |      90.5 |
| 2018_CS_01 | MonkeyDLuffy | PrincipleofStatistics   |      69.0 |
| 2018_CS_01 | MonkeyDLuffy | ProbabilityTheory       |      76.0 |
| 2018_CS_01 | MonkeyDLuffy | Physics                 |      63.5 |
| 2018_CS_01 | MonkeyDLuffy | AdvancedMathematics     |      95.5 |
| 2018_CS_01 | MonkeyDLuffy | LinearAlgebra           |      92.5 |
| 2018_CS_01 | MonkeyDLuffy | DiscreteMathematics     |      89.0 |
| 2018_CS_03 | SpongeBob    | PrinciplesofDatabase    |      88.0 |
| 2018_CS_03 | SpongeBob    | English                 |      79.0 |
| 2018_CS_03 | SpongeBob    | OpBasketball            |      92.0 |
| 2018_CS_03 | SpongeBob    | OpTennis                |      94.0 |
| 2018_CS_03 | PatrickStar  | LinearAlgebra           |       6.5 |
| 2018_CS_03 | PatrickStar  | AdvancedMathematics     |       5.0 |
| 2018_CS_03 | SpongeBob    | DiscreteMathematics     |      72.0 |
| 2018_CS_03 | PatrickStar  | ProbabilityTheory       |      12.0 |
| 2018_CS_03 | PatrickStar  | PrincipleofStatistics   |      20.0 |
| 2018_CS_03 | PatrickStar  | OperatingSystem         |      36.0 |
| 2018_CS_03 | PatrickStar  | FundamentalsofCompiling |       2.0 |
| 2018_CS_03 | PatrickStar  | DiscreteMathematics     |      14.0 |
```
```
      | 2018_CS_03 | PatrickStar  | PrinciplesofDatabase    |       9.0 |
      | 2018_CS_03 | PatrickStar  | English                 |      60.0 |
      | 2018_CS_03 | PatrickStar  | OpTableTennis           |      12.0 |
      | 2018_CS_03 | PatrickStar  | OpPiano                 |      99.0 |
      | 2018_CS_03 | SpongeBob    | FundamentalsofCompiling |      43.0 |
      | 2018_CS_03 | SpongeBob    | OperatingSystem         |      95.0 |
      | 2018_CS_03 | SpongeBob    | PrincipleofStatistics   |      90.0 |
      | 2018_CS_03 | SpongeBob    | ProbabilityTheory       |      87.0 |
      | 2018_CS_03 | SpongeBob    | Physics                 |      65.0 |
      | 2018_CS_03 | SpongeBob    | AdvancedMathematics     |      55.0 |
      | 2018_CS_03 | SpongeBob    | LinearAlgebra           |      60.5 |
      | 2018_CS_03 | PatrickStar  | Physics                 |       6.0 |
+------------+--------------+-------------------------+-----------+
36 rows in set (0.01 sec)

```

結果が安定しないのは、 `ORDER BY` の値が同じ場合には結果が不安定になるためです。ランダム性を減らすためには、 `ORDER BY` の値をユニークにする必要があります。ユニーク性を保証できない場合は、`ORDER BY` フィールドを追加して、`ORDER BY` フィールドの組み合わせがユニークになるようにする必要があります。そのために結果が安定化します。

## 結果セットが不安定な理由は、 `GROUP_CONCAT()` で `ORDER BY` が使用されていないためです

結果セットが不安定なのは、TiDBがストレージレイヤーからデータを並列で読み取るため、 `ORDER BY` なしの `GROUP_CONCAT()` によって返される結果セットの順序が不安定に見えるからです。

`GROUP_CONCAT()` が結果セットの出力を順序通りに取得するようにするには、 `ORDER BY` 句に並べ替えフィールドを追加する必要があります。これはSQLのセマンティクスに準拠しています。次の例では、 `ORDER BY` が含まれていない `customer_id` を連結する `GROUP_CONCAT()` によって不安定な結果セットが発生します。

1. `ORDER BY` が除外されています。

    最初のクエリ:

    {{< copyable "sql" >}}

    ```sql
    mysql>  select GROUP_CONCAT( customer_id SEPARATOR ',' ) FROM customer where customer_id like '200002%';
    +-------------------------------------------------------------------------+
    | GROUP_CONCAT(customer_id  SEPARATOR ',')                                |
    +-------------------------------------------------------------------------+
    | 20000200992,20000200993,20000200994,20000200995,20000200996,20000200... |
    +-------------------------------------------------------------------------+
    ```

    2番目のクエリ:

    {{< copyable "sql" >}}

    ```sql
    mysql>  select GROUP_CONCAT( customer_id SEPARATOR ',' ) FROM customer where customer_id like '200002%';
    +-------------------------------------------------------------------------+
    | GROUP_CONCAT(customer_id  SEPARATOR ',')                                |
    +-------------------------------------------------------------------------+
    | 20000203040,20000203041,20000203042,20000203043,20000203044,20000203... |
    +-------------------------------------------------------------------------+
    ```

2. `ORDER BY` が含まれています。

    最初のクエリ:

    {{< copyable "sql" >}}

    ```sql
    mysql>  select GROUP_CONCAT( customer_id order by customer_id SEPARATOR ',' ) FROM customer where customer_id like '200002%';
    +-------------------------------------------------------------------------+
    | GROUP_CONCAT(customer_id  SEPARATOR ',')                                |
    +-------------------------------------------------------------------------+
    | 20000200000,20000200001,20000200002,20000200003,20000200004,20000200... |
    +-------------------------------------------------------------------------+
    ```

    2番目のクエリ:

    {{< copyable "sql" >}}

    ```sql
    mysql>  select GROUP_CONCAT( customer_id order by customer_id SEPARATOR ',' ) FROM customer where customer_id like '200002%';
    +-------------------------------------------------------------------------+
    | GROUP_CONCAT(customer_id  SEPARATOR ',')                                |
    +-------------------------------------------------------------------------+
    | 20000200000,20000200001,20000200002,20000200003,20000200004,20000200... |
    +-------------------------------------------------------------------------+
    ```

## `SELECT * FROM T LIMIT N` における不安定な結果

返される結果は、ストレージノード（TiKV）上のデータの分布に関連しています。複数のクエリが実行されると、ストレージノード（TiKV）の異なるストレージユニット（リージョン）が異なる速度で結果を返し、不安定な結果を引き起こすことがあります。