---
title: 生成されたカラム
summary: 生成されたカラムの使用方法を学ぶ
aliases: ['/docs/dev/generated-columns/','/docs/dev/reference/sql/generated-columns/']
---

# 生成されたカラム

このドキュメントでは、生成されたカラムの概念と使用方法について紹介します。

## 基本的な概念

一般のカラムとは異なり、生成されたカラムの値はカラムの定義内の式によって計算されます。生成されたカラムに値を割り当てることはできず、`DEFAULT`のみを使用できます。

生成されたカラムには、仮想的なものと格納されたものの2種類があります。仮想生成されたカラムはストレージを占有せず、読み取られる際に計算されます。格納された生成されたカラムは書き込まれる（挿入または更新される）際に計算され、ストレージを占有します。仮想生成されたカラムよりも格納された生成されたカラムの方が読み取りパフォーマンスが向上しますが、より多くのディスク容量を占有します。

生成されたカラムには、それが仮想であろうと格納されたであろうと、インデックスを作成することができます。

## 使用法

生成されたカラムの主な使用法の1つは、JSONデータ型からデータを抽出し、そのデータにインデックスを付けることです。

MySQL 5.7およびTiDBの両方で、JSON型のカラムには直接インデックスを付けることはできません。つまり、以下のテーブル定義は**サポートされていません**：

{{< copyable "sql" >}}

```sql
CREATE TABLE person (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address_info JSON,
    KEY (address_info)
);
```

JSONカラムにインデックスを付けるには、まず生成されたカラムとしてそれを抽出する必要があります。

`address_info`内の`city`フィールドを例として使用すると、仮想生成されたカラムを作成し、それにインデックスを追加できます:

{{< copyable "sql" >}}

```sql
CREATE TABLE person (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address_info JSON,
    city VARCHAR(64) AS (JSON_UNQUOTE(JSON_EXTRACT(address_info, '$.city'))), -- 仮想生成されたカラム
    -- city VARCHAR(64) AS (JSON_UNQUOTE(JSON_EXTRACT(address_info, '$.city'))) VIRTUAL, -- 仮想生成されたカラム
    -- city VARCHAR(64) AS (JSON_UNQUOTE(JSON_EXTRACT(address_info, '$.city'))) STORED, -- 格納された生成されたカラム
    KEY (city)
);
```

このテーブルでは、`city`カラムは仮想生成されたカラムであり、インデックスがあります。次のクエリでは、そのインデックスを使用して実行が高速化されます:

{{< copyable "sql" >}}

```sql
SELECT name, id FROM person WHERE city = 'Beijing';
```

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT name, id FROM person WHERE city = 'Beijing';
```

```sql
+---------------------------------+---------+-----------+--------------------------------+-------------------------------------------------------------+
| id                              | estRows | task      | access object                  | operator info                                               |
+---------------------------------+---------+-----------+--------------------------------+-------------------------------------------------------------+
| Projection_4                    | 10.00   | root      |                                | test.person.name, test.person.id                            |
| └─IndexLookUp_10                | 10.00   | root      |                                |                                                             |
|   ├─IndexRangeScan_8(Build)     | 10.00   | cop[tikv] | table:person, index:city(city) | range:["Beijing","Beijing"], keep order:false, stats:pseudo |
|   └─TableRowIDScan_9(Probe)     | 10.00   | cop[tikv] | table:person                   | keep order:false, stats:pseudo                              |
+---------------------------------+---------+-----------+--------------------------------+-------------------------------------------------------------+
```

クエリの実行計画から、`city`インデックスが`city ='Beijing'`条件を満たす行の`HANDLE`を読み取り、その`HANDLE`を使用して行のデータを読み取ることがわかります。

`$.city`にパスにデータが存在しない場合、`JSON_EXTRACT`は`NULL`を返します。`city`が`NOT NULL`であることを強制したい場合は、次のように仮想生成されたカラムを定義できます:

{{< copyable "sql" >}}

```sql
CREATE TABLE person (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    address_info JSON,
    city VARCHAR(64) AS (JSON_UNQUOTE(JSON_EXTRACT(address_info, '$.city'))) NOT NULL,
    KEY (city)
);
```

## 生成されたカラムの検証

`INSERT`および`UPDATE`文によって、仮想カラムの定義が検査されます。検証に合格しない行はエラーを返します:

{{< copyable "sql" >}}

```sql
mysql> INSERT INTO person (name, address_info) VALUES ('Morgan', JSON_OBJECT('Country', 'Canada'));
ERROR 1048 (23000): Column 'city' cannot be null
```

## 生成されたカラムのインデックス置換ルール

クエリ内の式がインデックスを持つ生成されたカラムと厳密に等しい場合、TiDBはその式を対応する生成されたカラムで置換し、そのインデックスを実行計画構築の際に考慮できるようにします。

次の例では、式`a+1`のための生成されたカラムとインデックスが作成されます。`a`のカラム型はintであり、`a+1`のカラム型はbigintです。生成されたカラムの型がintに設定されている場合、置換は行われません。型変換のルールについては、[式評価の型変換](/functions-and-operators/type-conversion-in-expression-evaluation.md)を参照してください。

```sql
create table t(a int);
desc select a+1 from t where a+1=3;
```

```sql
+---------------------------+----------+-----------+---------------+--------------------------------+
| id                        | estRows  | task      | access object | operator info                  |
+---------------------------+----------+-----------+---------------+--------------------------------+
| Projection_4              | 8000.00  | root      |               | plus(test.t.a, 1)->Column#3    |
| └─TableReader_7           | 8000.00  | root      |               | data:Selection_6               |
|   └─Selection_6           | 8000.00  | cop[tikv] |               | eq(plus(test.t.a, 1), 3)       |
|     └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo |
+---------------------------+----------+-----------+---------------+--------------------------------+
4 rows in set (0.00 sec)
```

```sql
alter table t add column b bigint as (a+1) virtual;
alter table t add index idx_b(b);
desc select a+1 from t where a+1=3;
```

```sql
+------------------------+---------+-----------+-------------------------+---------------------------------------------+
| id                     | estRows | task      | access object           | operator info                               |
+------------------------+---------+-----------+-------------------------+---------------------------------------------+
| IndexReader_6          | 10.00   | root      |                         | index:IndexRangeScan_5                      |
| └─IndexRangeScan_5     | 10.00   | cop[tikv] | table:t, index:idx_b(b) | range:[3,3], keep order:false, stats:pseudo |
+------------------------+---------+-----------+-------------------------+---------------------------------------------+
2 rows in set (0.01 sec)
```

> **注意:**
>
> 置換される式と生成されたカラムが共に文字列型であっても、長さが異なる場合は、システム変数[`tidb_enable_unsafe_substitute`](/system-variables.md#tidb_enable_unsafe_substitute-new-in-v630)を`ON`に設定することで、式を置換することができます。このシステム変数を構成する際には、生成されたカラムによって厳密に生成されたカラムの定義が満たされることを確実にしてください。そうでない場合、長さの違いによってデータが切り捨てられ、不正な結果が生じる可能性があります。GitHub issue [#35490](https://github.com/pingcap/tidb/issues/35490#issuecomment-1211658886)を参照してください。

## 制限事項

JSONと生成されたカラムの現在の制限事項は以下の通りです:

- `ALTER TABLE`を使用して格納された生成されたカラムを追加することはできません。
- `ALTER TABLE`文を使用して、格納された生成されたカラムを通常のカラムに変換すること、または通常のカラムを格納された生成されたカラムに変換することはできません。
- `ALTER TABLE`文を使用して格納された生成されたカラムの式を変更することはできません。
- すべての[JSON関数](/functions-and-operators/json-functions.md)がサポートされていません;
- 現在、生成されたカラムのインデックスの置換ルールは、生成されたカラムが仮想生成されたカラムの場合にのみ有効です。格納された生成されたカラムでは有効ではありませんが、生成されたカラム自体を直接使用することでインデックスを使用できます。