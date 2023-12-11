---
title: TopNおよびLimit演算子プッシュダウン
summary: TopNおよびLimit演算子プッシュダウンの実装について学ぶ
---

# TopNおよびLimit演算子プッシュダウン

この文書では、TopNおよびLimit演算子プッシュダウンの実装について説明します。

TiDBの実行計画ツリーでは、SQLの`LIMIT`句はLimit演算子ノードに、`ORDER BY`句はSort演算子ノードに対応します。隣接するLimit演算子とSort演算子はTopN演算子ノードとして結合され、つまり特定のソートルールに従って上位N件のレコードが返されることを意味します。つまり、Limit演算子は、ソートルールがnullであるTopN演算子ノードと等価です。

プッシュダウンは、述語プッシュダウンと同様に、実行計画ツリーでできるだけデータソースに近い位置にTopNおよびLimitをプッシュダウンし、必要なデータを早い段階でフィルタリングするようにします。このようにして、プッシュダウンはデータの転送と計算のオーバーヘッドを大幅に削減します。

このルールを無効にするには、[最適化ルールおよび式プッシュダウンのブロックリスト](/blocklist-control-plan.md)を参照してください。

## 例

このセクションでは、いくつかの例を通じてTopNプッシュダウンを説明します。

### 例1：ストレージレイヤーのコプロセッサにプッシュダウン

{{< copyable "sql" >}}

```sql
create table t(id int primary key, a int not null);
explain select * from t order by a limit 10;
```

```
+----------------------------+----------+-----------+---------------+--------------------------------+
| id                         | estRows  | task      | access object | operator info                  |
+----------------------------+----------+-----------+---------------+--------------------------------+
| TopN_7                     | 10.00    | root      |               | test.t.a, offset:0, count:10   |
| └─TableReader_15           | 10.00    | root      |               | data:TopN_14                   |
|   └─TopN_14                | 10.00    | cop[tikv] |               | test.t.a, offset:0, count:10   |
|     └─TableFullScan_13     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo |
+----------------------------+----------+-----------+---------------+--------------------------------+
4 rows in set (0.00 sec)
```

このクエリでは、TopN演算子ノードがTiKVにプッシュダウンされてデータのフィルタリングが行われ、各コプロセッサがTiDBに対して10件のレコードのみを返します。TiDBがデータを集計した後に最終的なフィルタリングが実行されます。

### 例2：TopNはJoinにプッシュダウンできる（ソーティングルールは外部テーブルの列にのみ依存）

{{< copyable "sql" >}}

```sql
create table t(id int primary key, a int not null);
create table s(id int primary key, a int not null);
explain select * from t left join s on t.a = s.a order by t.a limit 10;
```

```
+----------------------------------+----------+-----------+---------------+-------------------------------------------------+
| id                               | estRows  | task      | access object | operator info                                   |
+----------------------------------+----------+-----------+---------------+-------------------------------------------------+
| TopN_12                          | 10.00    | root      |               | test.t.a, offset:0, count:10                    |
| └─HashJoin_17                    | 12.50    | root      |               | left outer join, equal:[eq(test.t.a, test.s.a)] |
|   ├─TopN_18(Build)               | 10.00    | root      |               | test.t.a, offset:0, count:10                    |
|   │ └─TableReader_26             | 10.00    | root      |               | data:TopN_25                                    |
|   │   └─TopN_25                  | 10.00    | cop[tikv] |               | test.t.a, offset:0, count:10                    |
|   │     └─TableFullScan_24       | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                  |
|   └─TableReader_30(Probe)        | 10000.00 | root      |               | data:TableFullScan_29                           |
|     └─TableFullScan_29           | 10000.00 | cop[tikv] | table:s       | keep order:false, stats:pseudo                  |
+----------------------------------+----------+-----------+---------------+-------------------------------------------------+
8 rows in set (0.01 sec)
```

このクエリでは、TopN演算子のソーティングルールが外部テーブル`t`の列にのみ依存するため、Joinにプッシュダウンする前に計算を行い、Join演算の計算コストを削減することができます。また、TiDBはTopNをストレージレイヤーにもプッシュダウンします。

### 例3：Join前にTopNはプッシュダウンできない

{{< copyable "sql" >}}

```sql
create table t(id int primary key, a int not null);
create table s(id int primary key, a int not null);
explain select * from t join s on t.a = s.a order by t.id limit 10;
```

```
+-------------------------------+----------+-----------+---------------+--------------------------------------------+
| id                            | estRows  | task      | access object | operator info                              |
+-------------------------------+----------+-----------+---------------+--------------------------------------------+
| TopN_12                       | 10.00    | root      |               | test.t.id, offset:0, count:10              |
| └─HashJoin_16                 | 12500.00 | root      |               | inner join, equal:[eq(test.t.a, test.s.a)] |
|   ├─TableReader_21(Build)     | 10000.00 | root      |               | data:TableFullScan_20                      |
|   │ └─TableFullScan_20        | 10000.00 | cop[tikv] | table:s       | keep order:false, stats:pseudo             |
|   └─TableReader_19(Probe)     | 10000.00 | root      |               | data:TableFullScan_18                      |
|     └─TableFullScan_18        | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo             |
+-------------------------------+----------+-----------+---------------+--------------------------------------------+
6 rows in set (0.00 sec)
```

TopNは`Inner Join`の前にプッシュダウンできません。上記のクエリを例にとると、Join後に100件のレコードが得られるとすると、TopN後に10件のみが残ります。しかし、最初にTopNを行うと、Join後に5件しか残らなくなります。このようなケースでは、プッシュダウンによって結果が異なります。

同様に、TopNはOuter Joinの内部テーブルにプッシュダウンすることもできず、TopNのソートルールが複数のテーブルの列に関連する場合（例：`t.a+s.a`など）、TopNをプッシュダウンすることはできません。TopNがソートルールに独占的に依存する場合のみ、TopNをプッシュダウンすることができます。

### 例4：TopNをLimitに変換する

{{< copyable "sql" >}}

```sql
create table t(id int primary key, a int not null);
create table s(id int primary key, a int not null);
explain select * from t left join s on t.a = s.a order by t.id limit 10;
```

```
+----------------------------------+----------+-----------+---------------+-------------------------------------------------+
| id                               | estRows  | task      | access object | operator info                                   |
+----------------------------------+----------+-----------+---------------+-------------------------------------------------+
| TopN_12                          | 10.00    | root      |               | test.t.id, offset:0, count:10                   |
| └─HashJoin_17                    | 12.50    | root      |               | left outer join, equal:[eq(test.t.a, test.s.a)] |
|   ├─Limit_21(Build)              | 10.00    | root      |               | offset:0, count:10                              |
|   │ └─TableReader_31             | 10.00    | root      |               | data:Limit_30                                   |
|   │   └─Limit_30                 | 10.00    | cop[tikv] |               | offset:0, count:10                              |
|   │     └─TableFullScan_29       | 10.00    | cop[tikv] | table:t       | keep order:true, stats:pseudo                   |
|   └─TableReader_35(Probe)        | 10000.00 | root      |               | data:TableFullScan_34                           |
|     └─TableFullScan_34           | 10000.00 | cop[tikv] | table:s       | keep order:false, stats:pseudo                  |
+----------------------------------+----------+-----------+---------------+-------------------------------------------------+
8 rows in set (0.00 sec)
```

上記のクエリでは、TopNが最初に外部テーブル`t`にプッシュダウンされます。TopNは`keep order: true`で直接順序で読むことができるため（主キーである`t.id`によって）、TopNで追加のソートなしに直接読めます。したがって、TopNはLimitとして簡略化されます。