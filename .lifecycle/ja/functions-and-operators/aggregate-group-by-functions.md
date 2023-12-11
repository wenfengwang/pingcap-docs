---
title: 集計（GROUP BY）関数
summary: TiDBでサポートされている集計関数について学びます。
aliases: ['/docs/dev/functions-and-operators/aggregate-group-by-functions/', '/docs/dev/reference/sql/functions-and-operators/aggregate-group-by-functions/']
---

# 集計（GROUP BY）関数

このドキュメントでは、TiDBでサポートされている集計関数についての詳細を説明します。

## サポートされている集計関数

このセクションでは、TiDBでサポートされているMySQLの`GROUP BY`集計関数について説明します。

| 名前                                                                                                 | 説明                                      |
|:------------------------------------------------------------------------------------------------------|:------------------------------------------|
| [`COUNT()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_count)             | 返される行の数をカウント               |
| [`COUNT(DISTINCT)`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_count-distinct) | 異なる値の数をカウント            |
| [`SUM()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_sum)                      | 合計を返します                               |
| [`AVG()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_avg)                     | 引数の平均値を返します                    |
| [`MAX()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_max)                     | 最大値を返します                            |
| [`MIN()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_min)                     | 最小値を返します                            |
| [`GROUP_CONCAT()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_group-concat) | 結合された文字列を返します                 |
| [`VARIANCE()`, `VAR_POP()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_var-pop) | 母集団標準分散を返します |
| [`STD()`, `STDDEV()`, `STDDEV_POP`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_std) | 母集団標準偏差を返します |
| [`VAR_SAMP()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_var-samp) | 標本分散を返します |
| [`STDDEV_SAMP()`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_stddev-samp) | 標本標準偏差を返します |
| [`JSON_OBJECTAGG(key, value)`](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_json-objectagg) | キーと値のペアを含む単一のJSONオブジェクトとして結果セットを返します |

- 特に記載がない限り、グループ関数は`NULL`値を無視します。
- `GROUP BY`句を含まないステートメントでグループ関数を使用すると、すべての行をグループ化するのと同等です。

さらに、TiDBでは以下の集計関数も提供されています:

+ `APPROX_PERCENTILE(expr, constant_integer_expr)`

    この関数は、`expr`のパーセンタイルを返します。`constant_integer_expr`引数は、`[1,100]`の範囲の定数整数でパーセンテージ値を示します。パーセンタイルP<sub>k</sub> (`k`はパーセンテージを表す) は、データセットに少なくとも`k%`の値が存在することを示します。

    この関数は、返される型として [numeric type](/data-type-numeric.md) と [date and time type](/data-type-date-and-time.md) をサポートします。他の返される型の場合、`APPROX_PERCENTILE` は `NULL` を返すだけです。

    次の例では、`INT`列の50パーセンタイルを計算する方法を示しています:

    {{< copyable "sql" >}}

    ```sql
    drop table if exists t;
    create table t(a int);
    insert into t values(1), (2), (3);
    ```

    {{< copyable "sql" >}}

    ```sql
    select approx_percentile(a, 50) from t;
    ```

    ```sql
    +--------------------------+
    | approx_percentile(a, 50) |
    +--------------------------+
    |                        2 |
    +--------------------------+
    1 row in set (0.00 sec)
    ```

`GROUP_CONCAT()` および `APPROX_PERCENTILE()` 関数以外の前述の関数は、すべて [Window functions](/functions-and-operators/window-functions.md) としても機能することができます。

## GROUP BY 修飾子

v7.4.0より、TiDBの`GROUP BY`句は `WITH ROLLUP` 修飾子をサポートします。詳細については、[GROUP BY 修飾子](/functions-and-operators/group-by-modifier.md) を参照してください。

## SQL モードのサポート

TiDBは、SQLモード`ONLY_FULL_GROUP_BY`をサポートしており、有効になっている場合、曖昧な非集計列を含むクエリを拒否します。たとえば、次のクエリは、`ONLY_FULL_GROUP_BY`が有効の場合には不正です。`SELECT`リストの非集計列 "b" が`GROUP BY`句に現れないためです。

```sql
drop table if exists t;
create table t(a bigint, b bigint, c bigint);
insert into t values(1, 2, 3), (2, 2, 3), (3, 2, 3);

mysql> select a, b, sum(c) from t group by a;
+------+------+--------+
| a    | b    | sum(c) |
+------+------+--------+
|    1 |    2 |      3 |
|    2 |    2 |      3 |
|    3 |    2 |      3 |
+------+------+--------+
3 rows in set (0.01 sec)

mysql> set sql_mode = 'ONLY_FULL_GROUP_BY';
Query OK, 0 rows affected (0.00 sec)

mysql> select a, b, sum(c) from t group by a;
ERROR 1055 (42000): Expression #2 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'b' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

現在、TiDBはデフォルトで[`ONLY_FULL_GROUP_BY`](/mysql-compatibility.md#default-differences)モードを有効にしています。

### MySQL との相違点

`ONLY_FULL_GROUP_BY`の現在の実装は、MySQL 5.7のものよりも厳密ではありません。たとえば、以下のクエリを実行したとき、結果は "c" で並べ替えることを期待します。

```sql
drop table if exists t;
create table t(a bigint, b bigint, c bigint);
insert into t values(1, 2, 1), (1, 2, 2), (1, 3, 1), (1, 3, 2);
select distinct a, b from t order by c;
```

結果を順序付けるためには、まず重複を排除する必要があります。それを行うためには、どの行を保持するかを選択する必要があります。この選択は "c" の保持値に影響を及ぼし、それにより順序付けが任意になります。

MySQLでは、`DISTINCT`と`ORDER BY`を持つクエリは次の条件の少なくとも一つを満たさない場合に不正として拒否されます:

- 式が`SELECT`リスト内の式と等しい
- 式に参照されるすべての列が`SELECT`したテーブルの要素である

しかし、TiDBでは、上記のクエリは有効です。より詳細な情報については[#4254](https://github.com/pingcap/tidb/issues/4254)を参照してください。

標準SQLでも`HAVING`句で、`SELECT`リスト内のエイリアス化された式を参照することが許可されていません。しかし、TiDBはこれを許可しています。次のようなクエリは、テーブル "orders" で一度しか発生する "name" 値を返します:

```sql
select name, count(name) from orders
group by name
having count(name) = 1;
```

TiDBの拡張により、`HAVING`句で集約された列のエイリアスを使用することが可能です:

```sql
select name, count(name) as c from orders
group by name
having c = 1;
```

標準SQLでは、`GROUP BY`句には列式のみを許可していますので、このようなステートメントは非正規です。なぜなら "FLOOR(value/100)" は非列式だからです:

```sql
select id, floor(value/100)
from tbl_name
group by id, floor(value/100);
```

TiDBは標準SQLを拡張して、非列式を`GROUP BY`句で許可しており、上記のステートメントを有効と見なします。

標準SQLでは、`GROUP BY`句にはエイリアスを許可していません。しかし、TiDBはこれを許可しており、次のようにクエリを書くこともできます:

```sql
select id, floor(value/100) as val
from tbl_name
group by id, val;
```

## 関連するシステム変数

`group_concat_max_len`変数は `GROUP_CONCAT()`関数の最大アイテム数を設定します。