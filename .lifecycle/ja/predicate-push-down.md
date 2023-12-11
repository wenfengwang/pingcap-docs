---
title: プレディケート プッシュ ダウン
summary: TiDB の論理最適化ルールの一つであるプレディケート プッシュ ダウン (PPD) を紹介します。
aliases: ['/tidb/dev/predicates-push-down']

# プレディケート プッシュ ダウン (PPD)

このドキュメントは、TiDB の論理最適化ルールの一つであるプレディケート プッシュ ダウン (PPD) を紹介します。このルールは、プレディケート プッシュ ダウンについて理解し、適用可能なシナリオと非適用可能なシナリオを把握するのに役立ちます。

PPD は、選択演算子をデータソースにできるだけ早く、データのフィルタリングをできるだけ早く完了するようにプッシュダウンし、データの送信や計算コストを大幅に削減します。

## 例

以下のケースは PPD の最適化を説明しています。ケース 1、2、3 は PPD が適用されるシナリオであり、ケース 4、5、6 は PPD が適用されないシナリオです。

### ケース 1: ストレージ層にプレディケートをプッシュ

```sql
create table t(id int primary key, a int);
explain select * from t where a < 1;
+-------------------------+----------+-----------+---------------+--------------------------------+
| id                      | estRows  | task      | access object | operator info                  |
+-------------------------+----------+-----------+---------------+--------------------------------+
| TableReader_7           | 3323.33  | root      |               | data:Selection_6               |
| └─Selection_6           | 3323.33  | cop[tikv] |               | lt(test.t.a, 1)                |
|   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo |
+-------------------------+----------+-----------+---------------+--------------------------------+
3 rows in set (0.00 sec)
```

このクエリでは、述語 `a < 1` を TiKV レイヤにプッシュダウンしてデータをフィルタリングすることで、ネットワークの転送コストを削減できます。

### ケース 2: ストレージ層にプレディケートをプッシュ

```sql
create table t(id int primary key, a int not null);
explain select * from t where a < substring('123', 1, 1);
+-------------------------+----------+-----------+---------------+--------------------------------+
| id                      | estRows  | task      | access object | operator info                  |
+-------------------------+----------+-----------+---------------+--------------------------------+
| TableReader_7           | 3323.33  | root      |               | data:Selection_6               |
| └─Selection_6           | 3323.33  | cop[tikv] |               | lt(test.t.a, 1)                |
|   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo |
+-------------------------+----------+-----------+---------------+--------------------------------+
```

このクエリはケース 1 のクエリと同じ実行計画を持っています。これは、述語 `a < substring('123', 1, 1)` の `substring` の入力パラメータが定数であるため、事前に計算できるためです。その後、述語が等価な述語 `a < 1` に単純化されます。その後、TiDB は `a < 1` を TiKV にプッシュダウンできます。

### ケース 3: 結合演算子の下にプレディケートをプッシュ

```sql
create table t(id int primary key, a int not null);
create table s(id int primary key, a int not null);
explain select * from t join s on t.a = s.a where t.a < 1;
+------------------------------+----------+-----------+---------------+--------------------------------------------+
| id                           | estRows  | task      | access object | operator info                              |
+------------------------------+----------+-----------+---------------+--------------------------------------------+
| HashJoin_8                   | 4154.17  | root      |               | inner join, equal:[eq(test.t.a, test.s.a)] |
| ├─TableReader_15(Build)      | 3323.33  | root      |               | data:Selection_14                          |
| │ └─Selection_14             | 3323.33  | cop[tikv] |               | lt(test.s.a, 1)                            |
| │   └─TableFullScan_13       | 10000.00 | cop[tikv] | table:s       | keep order:false, stats:pseudo             |
| └─TableReader_12(Probe)      | 3323.33  | root      |               | data:Selection_11                          |
|   └─Selection_11             | 3323.33  | cop[tikv] |               | lt(test.t.a, 1)                            |
|     └─TableFullScan_10       | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo             |
+------------------------------+----------+-----------+---------------+--------------------------------------------+
7 rows in set (0.00 sec)
```

このクエリでは、述語 `t.a < 1` が結合演算子の下にプッシュされ、事前にフィルタリングされるため、結合の計算オーバーヘッドを削減できます。

また、この SQL ステートメントには内部結合が実行されており、`ON` 条件は `t.a = s.a` です。述語 `s.a <1` は `t.a < 1` から導出でき、内部結合演算子の下に `s` テーブルにプッシュダウンできます。`s` テーブルをフィルタリングすることで、結合の計算オーバーヘッドをさらに削減できます。

### ケース 4: ストレージ層でサポートされていない述語はプッシュダウンできない

```sql
create table t(id int primary key, a varchar(10) not null);
desc select * from t where truncate(a, " ") = '1';
+-------------------------+----------+-----------+---------------+---------------------------------------------------+
| id                      | estRows  | task      | access object | operator info                                     |
+-------------------------+----------+-----------+---------------+---------------------------------------------------+
| Selection_5             | 8000.00  | root      |               | eq(truncate(cast(test.t.a, double BINARY), 0), 1) |
| └─TableReader_7         | 10000.00 | root      |               | data:TableFullScan_6                              |
|   └─TableFullScan_6     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                    |
+-------------------------+----------+-----------+---------------+---------------------------------------------------+
```

このクエリでは、述語 `truncate(a, " ") = '1'` があります。

`explain` の結果から、述語が TiKV にプッシュダウンされて計算されていないことがわかります。これは、TiKV 共同処理者が組込み関数 `truncate` をサポートしていないためです。

### ケース 5: 外部結合の内部テーブルの述語はプッシュダウンできない

```sql
create table t(id int primary key, a int not null);
create table s(id int primary key, a int not null);
explain select * from t left join s on t.a = s.a where s.a is null;
+-------------------------------+----------+-----------+---------------+-------------------------------------------------+
| id                            | estRows  | task      | access object | operator info                                   |
+-------------------------------+----------+-----------+---------------+-------------------------------------------------+
| Selection_7                   | 10000.00 | root      |               | isnull(test.s.a)                                |
| └─HashJoin_8                  | 12500.00 | root      |               | left outer join, equal:[eq(test.t.a, test.s.a)] |
|   ├─TableReader_13(Build)     | 10000.00 | root      |               | data:TableFullScan_12                           |
|   │ └─TableFullScan_12        | 10000.00 | cop[tikv] | table:s       | keep order:false, stats:pseudo                  |
|   └─TableReader_11(Probe)     | 10000.00 | root      |               | data:TableFullScan_10                           |
|     └─TableFullScan_10        | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                  |
+-------------------------------+----------+-----------+---------------+-------------------------------------------------+
6 rows in set (0.00 sec)
```

このクエリでは、内部テーブル `s` の述語 `s.a is null` があります。

`explain` の結果から、この述語は結合演算子の下にプッシュされていないことがわかります。これは、外部結合が `on` 条件を満たさない場合に内部テーブルを `NULL` 値で埋め、述語 `s.a is null` は結合後の結果をフィルタリングするためです。内部テーブルの下にプッシュダウンすると、実行計画が元のものと等しくないためです。

### ケース 6: ユーザ変数を含む述語はプッシュダウンできない

```sql
create table t(id int primary key, a char);
set @a = 1;
explain select * from t where a < @a;
+-------------------------+----------+-----------+---------------+--------------------------------+
| id                      | estRows  | task      | access object | operator info                  |
+-------------------------+----------+-----------+---------------+--------------------------------+
| Selection_5             | 8000.00  | root      |               | lt(test.t.a, getvar("a"))      |
| └─TableReader_7         | 10000.00 | root      |               | data:TableFullScan_6           |
|   └─TableFullScan_6     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo |
+-------------------------+----------+-----------+---------------+--------------------------------+
3 行がセットされました (0.00 秒)


このクエリでは、テーブル `t` の列 `a` に対する述語 `a < @a` があります。述語の `@a` はユーザー変数です。

`explain` 結果から分かるように、述語は簡略化されて `a < 1` になり、TiKV にプッシュダウンされるというケースはありません。これはユーザー変数 `@a` の値が計算中に変わる可能性があり、TiKV が変更を把握していないためです。そのため、TiDB は `@a` を `1` に置き換えず、TiKV にプッシュダウンしません。

理解を助けるための例を以下に示します:

```sql
create table t(id int primary key, a int);
insert into t values(1, 1), (2,2);
set @a = 1;
select id, a, @a:=@a+1 from t where a = @a;
+----+------+----------+
| id | a    | @a:=@a+1 |
+----+------+----------+
|  1 |    1 | 2        |
|  2 |    2 | 3        |
+----+------+----------+
2 行がセットされました (0.00 秒)
```

このクエリから分かるように、`@a` の値はクエリの間に変更されます。したがって、`a = @a` を `a = 1` に置き換えて TiKV にプッシュダウンすると、同等の実行計画にはなりません。