---
title: パーティション分割
summary: TiDB でのパーティション分割の使用方法を学ぶ
aliases: ['/docs/dev/partitioned-table/','/docs/dev/reference/sql/partitioning/']
---

# パーティション分割

このドキュメントでは、TiDBのパーティション分割の実装について紹介します。

## パーティション分割の種類

このセクションでは、TiDBでサポートされているパーティション分割の種類について紹介します。現在、TiDBは[Range partitioning](#range-partitioning)、[Range COLUMNS partitioning](#range-columns-partitioning)、[List partitioning](#list-partitioning)、[List COLUMNS partitioning](#list-columns-partitioning)、[Hash partitioning](#hash-partitioning)、および[Key partitioning](#key-partitioning)をサポートしています。

- Range partitioning、Range COLUMNS partitioning、List partitioning、およびList COLUMNS partitioningは、アプリケーション内の大量の削除によって引き起こされるパフォーマンスの問題を解決し、パーティションの迅速な削除をサポートしています。
- Hash partitioningおよびKey partitioningは、大量の書き込みが発生するシナリオでデータを分散するために使用されます。Hash partitioningと比較して、Key partitioningは複数の列のデータの分散と非整数の列によるパーティション分割をサポートしています。

### Range partitioning

テーブルをRangeでパーティション分割すると、各パーティションにはパーティション式の値が特定の範囲内にある行が含まれます。範囲は連続していなければなりませんが、重なっていてはいけません。`VALUES LESS THAN`を使用して定義することができます。

次のような人事記録を含むテーブルを作成する必要があるとします：

{{< copyable "sql" >}}

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT NOT NULL
);
```

必要に応じて、さまざまな方法でテーブルをRangeでパーティション分割できます。例えば、`store_id`列を使用してパーティション分割することができます：

{{< copyable "sql" >}}

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT NOT NULL
)

PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    PARTITION p3 VALUES LESS THAN (21)
);
```

このパーティションスキームでは、`store_id`が1から5である従業員に対応するすべての行は`p0`パーティションに格納され、`store_id`が6から10である従業員は`p1`に格納されます。Range partitioningでは、パーティションは最低値から最大値の順に並べる必要があります。

データ行`(72, 'Tom', 'John', '2015-06-25', NULL, NULL, 15)`を挿入すると、`p2`パーティションに入ります。しかし、`store_id`が20より大きいレコードを挿入すると、TiDBはこのレコードをどのパーティションに挿入するかわからないため、エラーが報告されます。この場合、テーブルを作成する際に`MAXVALUE`を使用できます：

{{< copyable "sql" >}}

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT NOT NULL
)

PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

`MAXVALUE`は、他のすべての整数値よりも大きい整数値を表します。今では、`store_id`が16（定義された最大値）以上であるすべてのレコードは`p3`パーティションに格納されます。

従業員のジョブコードでテーブルをパーティション分割することもできます。`job_code`列の値を使用します。2桁のジョブコードは通常の従業員、3桁のコードはオフィスおよびカスタマーサポートスタッフ、4桁のコードは管理職を表すと仮定します。その場合、次のようにパーティション分割されたテーブルを作成できます：

{{< copyable "sql" >}}

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT NOT NULL
)

PARTITION BY RANGE (job_code) (
    PARTITION p0 VALUES LESS THAN (100),
    PARTITION p1 VALUES LESS THAN (1000),
    PARTITION p2 VALUES LESS THAN (10000)
);
```

この例では、通常の従業員に関連するすべての行が`p0`パーティションに、オフィスおよびカスタマーサポートスタッフのすべての行が`p1`パーティションに、管理職のすべての行が`p2`パーティションに格納されます。

`store_id`でテーブルを分割するだけでなく、従業員の雇用年月日でテーブルを分割することもできます：

{{< copyable "sql" >}}

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)

PARTITION BY RANGE ( YEAR(separated) ) (
    PARTITION p0 VALUES LESS THAN (1991),
    PARTITION p1 VALUES LESS THAN (1996),
    PARTITION p2 VALUES LESS THAN (2001),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

Range partitioningでは、`timestamp`列の値に基づいてパーティション分割し、`unix_timestamp()`関数を使用することができます。例えば：

{{< copyable "sql" >}}

```sql
CREATE TABLE quarterly_report_status (
    report_id INT NOT NULL,
    report_status VARCHAR(20) NOT NULL,
    report_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
)

PARTITION BY RANGE ( UNIX_TIMESTAMP(report_updated) ) (
    PARTITION p0 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-01-01 00:00:00') ),
    PARTITION p1 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-04-01 00:00:00') ),
    PARTITION p2 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-07-01 00:00:00') ),
    PARTITION p3 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-10-01 00:00:00') ),
    PARTITION p4 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-01-01 00:00:00') ),
    PARTITION p5 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-04-01 00:00:00') ),
    PARTITION p6 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-07-01 00:00:00') ),
    PARTITION p7 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-10-01 00:00:00') ),
    PARTITION p8 VALUES LESS THAN ( UNIX_TIMESTAMP('2010-01-01 00:00:00') ),
    PARTITION p9 VALUES LESS THAN (MAXVALUE)
);
```

タイムスタンプ列を含む他のパーティショニング式を使用することは許可されていません。

Range partitioningは、次の条件のいずれかまたは複数が満たされている場合に特に有用です：

* 古いデータを削除したい場合。前述の`employees`テーブルを使用する場合、`ALTER TABLE employees DROP PARTITION p0;`という操作を単に実行することで、1991年より前にこの会社を去った従業員のすべてのレコードを削除できます。これは`DELETE FROM employees WHERE YEAR(separated) <= 1990;`操作よりも高速です。
* 時間または日付の値を含む列を使用したい場合、または他のシリーズから生じる値を含む列を使用したい場合。
* パーティショニングに使用される列で頻繁にクエリを実行する必要がある場合。たとえば、`EXPLAIN SELECT COUNT(*) FROM employees WHERE separated BETWEEN '2000-01-01' AND '2000-12-31' GROUP BY store_id;`といったクエリを実行する際、TiDBは`WHERE`条件に一致しないため、他のパーティションをスキャンする必要がないことを迅速に把握できます。

### Range COLUMNS partitioning

Range COLUMNS partitioningはRange partitioningの一種です。1つ以上の列をパーティションキーとして使用することができます。パーティション列のデータ型は整数、文字列（`CHAR`または`VARCHAR`）、`DATE`、および`DATETIME`のいずれかであることができます。他のCOLUMNSパーティショニングと同様に、式の使用はサポートされていません。

名前によってパーティション分割し、古い無効なデータを削除したい場合は次のようにテーブルを作成できます：

```sql
CREATE TABLE t (
  valid_until datetime,
  name varchar(255) CHARACTER SET ascii,
  notes text
)
PARTITION BY RANGE COLUMNS(name, valid_until)
(PARTITION `p2022-g` VALUES LESS THAN ('G','2023-01-01 00:00:00'),
```
```
PARTITION `p2023-g` VALUES LESS THAN ('G','2024-01-01 00:00:00'),
 PARTITION `p2022-m` VALUES LESS THAN ('M','2023-01-01 00:00:00'),
 PARTITION `p2023-m` VALUES LESS THAN ('M','2024-01-01 00:00:00'),
 PARTITION `p2022-s` VALUES LESS THAN ('S','2023-01-01 00:00:00'),
 PARTITION `p2023-s` VALUES LESS THAN ('S','2024-01-01 00:00:00'))
```
前記のSQL文はデータを年ごとと名前ごとに以下の範囲でパーティション分割します： `[ ('', ''), ('G', '2023-01-01 00:00:00') )`, `[ ('G', '2023-01-01 00:00:00'), ('G', '2024-01-01 00:00:00') )`, `[ ('G', '2024-01-01 00:00:00'), ('M', '2023-01-01 00:00:00') )`, `[ ('M', '2023-01-01 00:00:00'), ('M', '2024-01-01 00:00:00') )`, `[ ('M', '2024-01-01 00:00:00'), ('S', '2023-01-01 00:00:00') )`, `[ ('S', '2023-01-01 00:00:00'), ('S', '2024-01-01 00:00:00') )`。これにより、`name`と`valid_until`の両方によるパーティションプルーニングを利用しつつ、無効なデータを簡単に削除できます。この例では、`[,)`は左は閉じており右は開いた範囲を示します。例えば、`[ ('G', '2023-01-01 00:00:00'), ('G', '2024-01-01 00:00:00') )`は、名前が `'G'`であり、年が `2023-01-01 00:00:00`を含み、`2023-01-01 00:00:00`よりも大きく`2024-01-01 00:00:00`より小さいデータの範囲を示します。ただし、`(G, 2024-01-01 00:00:00)`は含まれません。

### 範囲INTERVALパーティション

範囲INTERVALパーティションは、指定された間隔のパーティションを簡単に作成できる範囲パーティションの拡張機能です。v6.3.0以降では、INTERVALパーティションはシンタックスシュガーとしてTiDBに導入されています。

構文は以下のとおりです：

```sql
PARTITION BY RANGE [COLUMNS] (<partitioning expression>)
INTERVAL (<interval expression>)
FIRST PARTITION LESS THAN (<expression>)
LAST PARTITION LESS THAN (<expression>)
[NULL PARTITION]
[MAXVALUE PARTITION]
```

例えば：

```sql
CREATE TABLE employees (
    id int unsigned NOT NULL,
    fname varchar(30),
    lname varchar(30),
    hired date NOT NULL DEFAULT '1970-01-01',
    separated date DEFAULT '9999-12-31',
    job_code int,
    store_id int NOT NULL
) PARTITION BY RANGE (id)
INTERVAL (100) FIRST PARTITION LESS THAN (100) LAST PARTITION LESS THAN (10000) MAXVALUE PARTITION
```

上記の構文は以下の表を作成します：

```sql
CREATE TABLE `employees` (
  `id` int unsigned NOT NULL,
  `fname` varchar(30) DEFAULT NULL,
  `lname` varchar(30) DEFAULT NULL,
  `hired` date NOT NULL DEFAULT '1970-01-01',
  `separated` date DEFAULT '9999-12-31',
  `job_code` int DEFAULT NULL,
  `store_id` int NOT NULL
)
PARTITION BY RANGE (`id`)
(PARTITION `P_LT_100` VALUES LESS THAN (100),
 PARTITION `P_LT_200` VALUES LESS THAN (200),
...
 PARTITION `P_LT_9900` VALUES LESS THAN (9900),
 PARTITION `P_LT_10000` VALUES LESS THAN (10000),
 PARTITION `P_MAXVALUE` VALUES LESS THAN (MAXVALUE))
```

範囲INTERVALパーティションは[Range COLUMNS](#range-columns-partitioning)パーティショニングとも互換性があります。

例えば：

```sql
CREATE TABLE monthly_report_status (
    report_id int NOT NULL,
    report_status varchar(20) NOT NULL,
    report_date date NOT NULL
)
PARTITION BY RANGE COLUMNS (report_date)
INTERVAL (1 MONTH) FIRST PARTITION LESS THAN ('2000-01-01') LAST PARTITION LESS THAN ('2025-01-01')
```

上記の構文は以下の表を作成します：

```sql
CREATE TABLE `monthly_report_status` (
  `report_id` int(11) NOT NULL,
  `report_status` varchar(20) NOT NULL,
  `report_date` date NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
PARTITION BY RANGE COLUMNS(`report_date`)
(PARTITION `P_LT_2000-01-01` VALUES LESS THAN ('2000-01-01'),
 PARTITION `P_LT_2000-02-01` VALUES LESS THAN ('2000-02-01'),
...
 PARTITION `P_LT_2024-11-01` VALUES LESS THAN ('2024-11-01'),
 PARTITION `P_LT_2024-12-01` VALUES LESS THAN ('2024-12-01'),
 PARTITION `P_LT_2025-01-01` VALUES LESS THAN ('2025-01-01'))
```

オプションのパラメータ`NULL PARTITION`は、`PARTITION P_NULL VALUES LESS THAN (<列の型の最小値>)`の定義で、パーティショニング式が`NULL`に評価された場合のみ一致します。`NULL`は他の任意の値よりも小さいと見なされることを説明する[Range partitioningにおけるNULLの取り扱い](#handling-of-null-with-range-partitioning)を参照してください。

オプションのパラメータ`MAXVALUE PARTITION`は、最後のパーティションを`PARTITION P_MAXVALUE VALUES LESS THAN (MAXVALUE)`として作成します。

#### INTERVALパーティショニングのALTER

INTERVALパーティショニングでは、より簡単な構文でのパーティションの追加および削除が可能となります。

次のステートメントは最初のパーティションを変更します。指定された式よりも小さい値のパーティションをすべて削除し、一致したパーティションを新しい最初のパーティションにします。`NULL PARTITION`には影響しません。

```
ALTER TABLE table_name FIRST PARTITION LESS THAN (<expression>)
```

次のステートメントは最後のパーティションを変更します。つまり、より高い範囲の新しいパーティションと新しいデータのための余地を持つように新しいパーティションを追加します。`MAXVALUE PARTITION`が存在する場合は機能しません。なぜならデータの再編成が必要だからです。

```
ALTER TABLE table_name LAST PARTITION LESS THAN (<expression>)
```

#### INTERVALパーティショニングの詳細および制限事項

- INTERVALパーティショニング機能は`CREATE/ALTER TABLE`構文に関係します。メタデータに変更はないため、新しい構文で作成または変更されたテーブルはMySQL互換性が保たれます。
- `SHOW CREATE TABLE`の出力形式には変更がありません。これはMySQL互換性を維持するためです。
- 新しいALTER構文は、INTERVALに準拠する既存のテーブルに適用されます。これらのテーブルを`INTERVAL`構文で作成する必要はありません。
- `RANGE COLUMNS`では、整数、日付、日時のみがサポートされます。

### リストパーティショニング

リストパーティションされたテーブルを作成する前に、次のシステム変数がデフォルトの`ON`で設定されていることを確認してください：

- [`tidb_enable_list_partition`](/system-variables.md#tidb_enable_list_partition-new-in-v50)
- [`tidb_enable_table_partition`](/system-variables.md#tidb_enable_table_partition)

リストパーティショニングは範囲パーティショニングに類似しています。範囲パーティショニングとは異なり、リストパーティショニングでは、各パーティション内のすべての行のパーティション式の値が指定された値セットにあります。各パーティションに定義されたこの値セットは複数の値を持つことができますが、重複した値は持てません。`PARTITION ... VALUES IN (...)`句を使用して値セットを定義できます。

人事レコードのテーブルを作成したいとします。次のようにテーブルを作成できます：

{{< copyable "sql" >}}

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    hired DATE NOT NULL DEFAULT '1970-01-01',
    store_id INT
);
```

次の表に示すように、4つの地区に20の店が分散しているとします：

```
| 地域   | 店舗ID番号        |
| ------- | -------------------- |
| 北      | 1, 2, 3, 4, 5        |
| 東      | 6, 7, 8, 9, 10       |
| 西      | 11, 12, 13, 14, 15   |
| 中央    | 16, 17, 18, 19, 20   |
```

同じ地域の店舗の社員のデータを同じパーティションに保存したい場合、`store_id`に基づくListパーティション化されたテーブルを作成できます：

{{< copyable "sql" >}}

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    hired DATE NOT NULL DEFAULT '1970-01-01',
```
```sql
    store_id INT
)
PARTITION BY LIST (store_id) (
    PARTITION pNorth VALUES IN (1, 2, 3, 4, 5),
    PARTITION pEast VALUES IN (6, 7, 8, 9, 10),
    PARTITION pWest VALUES IN (11, 12, 13, 14, 15),
    PARTITION pCentral VALUES IN (16, 17, 18, 19, 20)
);
```

上記のようにパーティションを作成した後は、テーブル内の特定の地域に関連するレコードを簡単に追加または削除できます。例えば、東部地域（East）のすべての店舗が別の会社に売却されたとします。そしてこの地域の店舗の従業員に関連するすべての行データを`ALTER TABLE employees TRUNCATE PARTITION pEast`を実行することで削除できます。これは、同等の`DELETE FROM employees WHERE store_id IN (6, 7, 8, 9, 10)`文よりも効率が良いです。

また、`ALTER TABLE employees DROP PARTITION pEast`を実行して関連するすべての行を削除することもできますが、この文はまた表定義から`pEast`パーティションも削除します。この場合、表の元のパーティショニングスキームを回復するために`ALTER TABLE ... ADD PARTITION`文を実行する必要があります。

#### デフォルトのListパーティション

v7.3.0から、ListまたはList COLUMNSパーティショニングテーブルにデフォルトパーティションを追加できます。デフォルトパーティションは、パーティションのいずれの値セットにも一致しない行を配置できるフォールバックパーティションとして機能します。

> **注意:**
>
> この機能はMySQL構文のTiDB拡張機能です。デフォルトパーティションを持つListまたはList COLUMNSパーティショニングテーブルのデータは、直接MySQLにレプリケートできません。

次のListパーティショニングテーブルを例に取り上げます：

```sql
CREATE TABLE t (
  a INT,
  b INT
)
PARTITION BY LIST (a) (
  PARTITION p0 VALUES IN (1, 2, 3),
  PARTITION p1 VALUES IN (4, 5, 6)
);
Query OK, 0 rows affected (0.11 sec)
```

次のようにテーブルにデフォルトのリストパーティション`pDef`を追加できます：

```sql
ALTER TABLE t ADD PARTITION (PARTITION pDef DEFAULT);
```

もしくは

```sql
ALTER TABLE t ADD PARTITION (PARTITION pDef VALUES IN (DEFAULT));
```

このようにすることで、パーティションの値セットと一致しない新しく挿入される値は自動的にデフォルトパーティションに配置されます。

```sql
INSERT INTO t VALUES (7, 7);
Query OK, 1 row affected (0.01 sec)
```

また、ListまたはList COLUMNSパーティショニングテーブルを作成する際にもデフォルトパーティションを追加することができます。例えば：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    hired DATE NOT NULL DEFAULT '1970-01-01',
    store_id INT
)
PARTITION BY LIST (store_id) (
    PARTITION pNorth VALUES IN (1, 2, 3, 4, 5),
    PARTITION pEast VALUES IN (6, 7, 8, 9, 10),
    PARTITION pWest VALUES IN (11, 12, 13, 14, 15),
    PARTITION pCentral VALUES IN (16, 17, 18, 19, 20),
    PARTITION pDefault DEFAULT
);
```

デフォルトパーティションのないListまたはList COLUMNSパーティショニングテーブルの場合、`INSERT`文を使用して挿入される値は、表の`PARTITION ... VALUES IN (...)`句で定義された値セットに一致する必要があります。挿入される値がいずれのパーティションの値セットにも一致しない場合、その文は失敗し、エラーが返されます。以下の例で示すように：

```sql
CREATE TABLE t (
  a INT,
  b INT
)
PARTITION BY LIST (a) (
  PARTITION p0 VALUES IN (1, 2, 3),
  PARTITION p1 VALUES IN (4, 5, 6)
);
Query OK, 0 rows affected (0.11 sec)

INSERT INTO t VALUES (7, 7);
ERROR 1525 (HY000): Table has no partition for value 7
```

先のエラーを無視することができるように、`INSERT`文に`IGNORE`キーワードを追加できます。このキーワードが追加されると、`INSERT`文はパーティションの値セットと一致する行のみを挿入し、一致しない行を挿入せずにエラーを返しません：

```sql
TRUNCATE t;
Query OK, 1 row affected (0.00 sec)

INSERT IGNORE INTO t VALUES (1, 1), (7, 7), (8, 8), (3, 3), (5, 5);
Query OK, 3 rows affected, 2 warnings (0.01 sec)
Records: 5  Duplicates: 2  Warnings: 2

select * from t;
+------+------+
| a    | b    |
+------+------+
|    5 |    5 |
|    1 |    1 |
|    3 |    3 |
+------+------+
3 rows in set (0.01 sec)
```

### List COLUMNSパーティショニング

List COLUMNSパーティショニングはListパーティショニングの一部です。複数の列をパーティションキーとして使用できます。整数データ型のほかに、文字列、`DATE`、および`DATETIME`データ型の列もパーティションカラムとして使用できます。

以下の12の都市の店舗従業員を4つの地域に分けたいとします：

```
| 地域 | 都市                           |
| :-- | ------------------------------ |
| 1   | ロサンゼルス、シアトル、ヒューストン |
| 2   | シカゴ、コロンバス、ボストン     |
| 3   | ニューヨーク、ロングアイランド、ボルチモア |
| 4   | アトランタ、ローリー、シンシナティ |
```

次のようにして、List COLUMNSパーティショニングを使用して、従業員の都市に対応するパーティションに各行を格納するテーブルを作成できます：

{{< copyable "sql" >}}

```sql
CREATE TABLE employees_1 (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT,
    city VARCHAR(15)
)
PARTITION BY LIST COLUMNS(city) (
    PARTITION pRegion_1 VALUES IN('ロサンゼルス', 'シアトル', 'ヒューストン'),
    PARTITION pRegion_2 VALUES IN('シカゴ', 'コロンバス', 'ボストン'),
    PARTITION pRegion_3 VALUES IN('ニューヨーク', 'ロングアイランド', 'ボルチモア'),
    PARTITION pRegion_4 VALUES IN('アトランタ', 'ローリー', 'シンシナティ')
);
```

Listパーティショニングとは異なり、List COLUMNSパーティショニングでは`COLUMNS()`句で列の値を整数に変換する式を使用する必要はありません。

また、次の例に示すように、`DATE`および`DATETIME`型の列を使用してもList COLUMNSパーティショニングを実装できます。この例は、前の`employees_1`テーブルと同じ名前と列を使用しますが、`hired`列ベースのList COLUMNSパーティショニングを使用しています：

{{< copyable "sql" >}}

```sql
CREATE TABLE employees_2 (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT,
    city VARCHAR(15)
)
PARTITION BY LIST COLUMNS(hired) (
    PARTITION pWeek_1 VALUES IN('2020-02-01', '2020-02-02', '2020-02-03',
        '2020-02-04', '2020-02-05', '2020-02-06', '2020-02-07'),
    PARTITION pWeek_2 VALUES IN('2020-02-08', '2020-02-09', '2020-02-10',
        '2020-02-11', '2020-02-12', '2020-02-13', '2020-02-14'),
    PARTITION pWeek_3 VALUES IN('2020-02-15', '2020-02-16', '2020-02-17',
        '2020-02-18', '2020-02-19', '2020-02-20', '2020-02-21'),
    PARTITION pWeek_4 VALUES IN('2020-02-22', '2020-02-23', '2020-02-24',
        '2020-02-25', '2020-02-26', '2020-02-27', '2020-02-28')
);
```

さらに、`COLUMNS()`句に複数の列を追加することもできます。例えば：

{{< copyable "sql" >}}

```sql
CREATE TABLE t (
    id int,
    name varchar(10)
)
PARTITION BY LIST COLUMNS(id,name) (
     partition p0 values IN ((1,'a'),(2,'b')),
```
      + パーティション p1 の値 IN ((3,'c'),(4,'d')),
     パーティション p3 の値 IN ((5,'e'),(null,null))
   );

### ハッシュ・パーティション

ハッシュ・パーティショニングは、データが一定数のパーティションに均等に分散されるようにするために使用されます。範囲パーティショニングとは異なり、ハッシュ・パーティショニングを使用する場合は、範囲パーティショニングでは各パーティションの列の値の範囲を指定する必要がありますが、ハッシュ・パーティショニングを使用する場合は単にパーティションの数を指定するだけで済みます。

ハッシュ・パーティション化されたテーブルを作成するには、`CREATE TABLE` 文に `PARTITION BY HASH (expr)` 句を追加する必要があります。`expr` は整数を返す式です。この列の型が整数である場合は列名を使用できます。さらに、`PARTITIONS num` も追加する必要があります。ここで、`num` はテーブルが分割されるパーティションの数を示す正の整数です。

次の操作は、`store_id` によって 4 つのパーティションに分割されたハッシュ・パーティション化されたテーブルを作成します。

{{< コード "sql" >}}

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)

PARTITION BY HASH(store_id)
PARTITIONS 4;
```

`PARTITIONS num` が指定されていない場合、デフォルトのパーティション数は 1 です。

`expr` を返すSQL式を使用することもできます。たとえば、採用年でテーブルをパーティション分割できます。

{{< コード "sql" >}}

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)

PARTITION BY HASH( YEAR(hired) )
PARTITIONS 4;
```

最も効率的なハッシュ・関数は、単一のテーブル列に対して動作し、その値が列の値と一貫して増減する関数です。

たとえば、`date_col` は型が `DATE` の列であり、`TO_DAYS(date_col)` 式の値は `date_col` の値とともに変化します。ただし、`YEAR(date_col)` は `TO_DAYS(date_col)` とは異なります。なぜなら、`date_col` のすべての可能な変更が `YEAR(date_col)` で等価な変更を生じるわけではないからです。

それに対して、型が `INT` の `int_col` 列を考えてみます。たとえば、`POW(5-int_col,3) + 6` という式を考えます。これは適切なハッシュ・関数ではありません。なぜなら、`int_col` の値の変化に伴って、式の結果が比例して変化しないからです。たとえば、`int_col` が 5 から 6 に変化すると、式の結果の変化は -1 です。しかし、`int_col` が 6 から 7 に変化すると、結果の変化は -7 になるかもしれません。

結論として、式が `y = cx` に近い形をしている場合、それはハッシュ・関数に適しています。なぜなら、式が非線形であるほど、データがパーティション間で不均等に分散される傾向があるからです。

理論的には、複数の列値を含む式に対してもプルーニングは可能ですが、そのような式が適切かどうかを判断することは非常に難しく、時間がかかります。このため、複数の列を含むハッシュ式の使用は特に推奨されません。

`PARTITION BY HASH` を使用する場合、TiDB はデータがどのパーティションに格納されるかを、式の結果の剰余に基づいて決定します。つまり、パーティショニング式が `expr` でパーティション数が `num` の場合、`MOD(expr, num)` がデータが格納されるパーティションを決定します。たとえば、`t1` が次のように定義されているとします:

{{< コード "sql" >}}

```sql
CREATE TABLE t1 (col1 INT, col2 CHAR(5), col3 DATE)
    PARTITION BY HASH( YEAR(col3) )
    PARTITIONS 4;
```

`t1` にデータ行を挿入し、`col3` の値が '2005-09-15' の場合、この行はパーティション 1 に挿入されます:

```
MOD(YEAR('2005-09-01'),4)
=  MOD(2005,4)
=  1
```

### キー・パーティショニング

v7.0.0 から、TiDB はキー・パーティショニングをサポートしています。v7.0.0 より前の TiDB のバージョンでは、キー・パーティション化されたテーブルを作成しようとすると、TiDB は非パーティション化されたテーブルを作成して警告を返します。

ハッシュ・パーティショニングとキー・パーティショニングは、どちらも一定数のパーティションにデータを均等に分散させることができます。違いは、ハッシュ・パーティショニングは指定された整数式または整数列に基づいてデータを分散するのに対し、キー・パーティショニングは列リストに基づいてデータを分散することができ、かつキー・パーティショニングのパーティション化列は整数型に限定されていません。また、TiDB のキー・パーティショニングのハッシュ・アルゴリズムは MySQL のものとは異なるため、テーブルデータの分配も異なります。

キー・パーティショニングされたテーブルを作成するには、`CREATE TABLE` 文に `PARTITION BY KEY (columList)` 句を追加する必要があります。`columList` は 1 つ以上の列名で構成される列リストです。リスト内の各列のデータ型は、`BLOB`、`JSON`、`GEOMETRY` (TiDB は `GEOMETRY` をサポートしていないことに注意) を除く任意の型です。さらに、`PARTITIONS num` (ここで `num` はテーブルが分割されるパーティションの数を示す正の整数) を追加する必要があります。または、パーティション名の定義を追加する必要があります。たとえば、`(PARTITION p0, PARTITION p1)` を追加すると、表を `p0` と `p1` という名前の 2 つのパーティションに分割します。

次の操作は、`store_id` によって 4 つのパーティションに分割されたキー・パーティション化されたテーブルを作成します:

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)

PARTITION BY KEY(store_id)
PARTITIONS 4;
```

`PARTITIONS num` が指定されていない場合、デフォルトのパーティション数は 1 です。

`fname` などの整数以外の列に基づいてキー・パーティション化されたテーブルを作成することもできます。

また、`fname` と `store_id` など複数の列に基づいてキー・パーティション化されたテーブルを作成することもできます。

現在、TiDB は `PARTITION BY KEY` で指定されるパーティショニング列リストが空の場合にキー・パーティション化されたテーブルを作成することをサポートしていません。たとえば、次のステートメントを実行した後、TiDB は非パーティション化されたテーブルを作成し、「サポートされていないパーティションタイプ 'KEY'、通常のテーブルとして扱う」という警告を返します。

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)

PARTITION BY KEY()
PARTITIONS 4;
```

#### TiDB がリニア・ハッシュ・パーティションをどのように処理するか

v6.4.0 より前のバージョンでは、TiDB で [MySQL リニア・ハッシュ](https://dev.mysql.com/doc/refman/8.0/en/partitioning-linear-hash.html) パーティションの DDL ステートメントを実行すると、TiDB は非パーティション化されたテーブルのみを作成できました。この場合、TiDB で引き続きパーティション化されたテーブルを使用したい場合は、DDL ステートメントを変更する必要があります。

v6.4.0 以降、TiDB は MySQL の `PARTITION BY LINEAR HASH` 構文を解析することができますが、`LINEAR` キーワードは無視されます。したがって、既存の MySQL リニア・ハッシュ・パーティションの DDL および DML ステートメントがある場合、変更せずに TiDB で実行することができます:

- MySQLのLinear Hashパーティションの `CREATE` ステートメントの場合、TiDBは非線形のHashパーティションテーブルを作成します（なお、TiDBにはLinear Hashパーティションテーブルは存在しません）。パーティションの数が2の累乗である場合、TiDB Hashパーティションテーブルの行はMySQL Linear Hashパーティションテーブルのものと同じように分散されます。それ以外の場合、TiDBにおけるこれらの行の分散はMySQLと異なります。これは、非線形のパーティションテーブルが単純な "パーティションの数の剰余" を使用しているのに対し、線形のパーティションテーブルが "次の2の累乗とパーティション数の間の値を折り返す次数の剰余" を使用しているためです。詳細については、[#38450](https://github.com/pingcap/tidb/issues/38450) を参照してください。

- MySQLのLinear Hashパーティションのその他のステートメントについては、パーティションの数が2の累乗でない場合を除き、TiDBではMySQLと同じように動作しますが、[パーティションの選択](#partition-selection)、`TRUNCATE PARTITION`、および `EXCHANGE PARTITION` において、行が異なるような結果が得られます。

### TiDBがLinear Keyパーティションを取り扱う方法

v7.0.0から、TiDBはMySQLの `PARTITION BY LINEAR KEY` 構文をキーパーティション用に解析することをサポートしています。ただし、TiDBでは `LINEAR` キーワードは無視され、非線形のハッシュアルゴリズムが代わりに使用されます。

v7.0.0以前では、Keyパーティションテーブルを作成しようとすると、TiDBはそれを非パーティションテーブルとして作成し、警告を返します。

### TiDBパーティショニングがNULLを取り扱う方法

TiDBでは、パーティション式の計算結果として `NULL` を使用することが許可されています。

> **注意:**
>
> `NULL` は整数ではありません。TiDBのパーティショニング実装では、`NULL` は他の整数値よりも小さいものとして扱われます。これは `ORDER BY` と同様です。

#### RangeパーティショニングにおけるNULLの取り扱い

Rangeによってパーティション化されたテーブルに行を挿入し、パーティションを決定するために使用される列の値が `NULL` の場合、この行は最も低いパーティションに挿入されます。

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (
    c1 INT,
    c2 VARCHAR(20)
)

PARTITION BY RANGE(c1) (
    PARTITION p0 VALUES LESS THAN (0),
    PARTITION p1 VALUES LESS THAN (10),
    PARTITION p2 VALUES LESS THAN MAXVALUE
);
```

```
クエリは正常に完了しました、影響を受けた行数はありません (0.09 sec)
```

{{< copyable "sql" >}}

```sql
select * from t1 partition(p0);
```

```
+------|--------+
| c1   | c2     |
+------|--------+
| NULL | mothra |
+------|--------+
1 行が選択されました (0.00 sec)
```

{{< copyable "sql" >}}

```sql
select * from t1 partition(p1);
```

```
空のセット (0.00 sec)
```

{{< copyable "sql" >}}

```sql
select * from t1 partition(p2);
```

```
Empty set (0.00 sec)
```

`p0` パーティションを削除して結果を検証します:

{{< copyable "sql" >}}

```sql
alter table t1 drop partition p0;
```

```
クエリは正常に完了しました、影響を受けた行数はありません (0.08 sec)
```

{{< copyable "sql" >}}

```sql
select * from t1;
```

```
空のセット (0.00 sec)
```

#### HashパーティショニングにおけるNULLの取り扱い

Hashによってテーブルをパーティション化する場合、`NULL`値の取り扱い方法が異なります。パーティショニング式の計算結果が `NULL` の場合、それは `0` と見なされます。

{{< copyable "sql" >}}

```sql
CREATE TABLE th (
    c1 INT,
    c2 VARCHAR(20)
)

PARTITION BY HASH(c1)
PARTITIONS 2;
```

```
クエリは正常に完了しました、影響を受けた行数はありません(0.00 sec)
```

{{< copyable "sql" >}}

```sql
INSERT INTO th VALUES (NULL, 'mothra'), (0, 'gigan');
```

```
クエリは正常に完了しました、影響を受けた行数はありません (0.04 sec)
```

{{< copyable "sql" >}}

```sql
select * from th partition (p0);
```

```
+------|--------+
| c1   | c2     |
+------|--------+
| NULL | mothra |
|    0 | gigan  |
+------|--------+
2 行が選択されました (0.00 sec)
```

{{< copyable "sql" >}}

```sql
select * from th partition (p1);
```

```
Empty set (0.00 sec)
```

挿入されたレコード `(NULL, 'mothra')` が `(0, 'gigan')` と同じパーティションに入ることがわかります。

> **注意:**
>
> TiDBにおけるHashパーティションにおける `NULL` 値の取り扱いは、[MySQL Partitioning Handles NULL](https://dev.mysql.com/doc/refman/8.0/en/partitioning-handling-nulls.html) で記述されている方法と同じですが、これはMySQLの実際の動作と一致していないという点で一貫していません。
>
> この場合、TiDBの実際の動作は本文書の記述と一致しています。

#### KeyパーティショニングにおけるNULLの取り扱い

Keyパーティショニングの場合、`NULL` 値の取り扱い方はHashパーティショニングと一貫しています。パーティションフィールドの値が `NULL` の場合、それは `0` として扱われます。

## パーティション管理

`RANGE`、`RANGE COLUMNS`、`LIST`、および `LIST COLUMNS` パーティショニングされたテーブルの場合、次のようにパーティションを管理することができます:

- `ALTER TABLE <テーブル名> ADD PARTITION (<パーティションの定義>)` ステートメントを使用してパーティションを追加します。
- `ALTER TABLE <テーブル名> DROP PARTITION <パーティションのリスト>` ステートメントを使用してパーティションを削除します。
- 指定されたパーティションからすべてのデータを削除するには `ALTER TABLE <テーブル名> TRUNCATE PARTITION <パーティションのリスト>` ステートメントを使用します。 `TRUNCATE PARTITION` のロジックは [`TRUNCATE TABLE`](/sql-statements/sql-statement-truncate.md) に似ていますが、パーティション用です。
- `ALTER TABLE <テーブル名> REORGANIZE PARTITION <パーティションのリスト> INTO (<新しいパーティションの定義>)` ステートメントを使用して、パーティションを統合したり分割したり、他の変更を加えたりすることができます。

`HASH` および `KEY` パーティション化されたテーブルの場合、次のようにパーティションを管理することができます:

- `ALTER TABLE <テーブル名> COALESCE PARTITION <減らすパーティション数>` ステートメントを使用して、パーティション数を減らすことができます。この操作は、テーブル全体をオンラインで新しいパーティション数にコピーすることでパーティションを再構成します。
- `ALTER TABLE <テーブル名> ADD PARTITION <増やすパーティション数 | (追加のパーティションの定義)>` ステートメントを使用して、パーティション数を増やすことができます。この操作は、テーブル全体をオンラインで新しいパーティション数にコピーすることでパーティションを再構成します。
- 指定されたパーティションからすべてのデータを削除するには `ALTER TABLE <テーブル名> TRUNCATE PARTITION <パーティションのリスト>` ステートメントを使用します。 `TRUNCATE PARTITION` のロジックは [`TRUNCATE TABLE`](/sql-statements/sql-statement-truncate.md) に似ていますが、パーティション用です。

`EXCHANGE PARTITION` は、パーティションと非パーティションテーブルを入れ替えることで動作します。`RENAME TABLE t1 TO t1_tmp, t2 TO t1, t1_tmp TO t2` のようにテーブル名を変更する方法と類似しています。

たとえば、`ALTER TABLE partitioned_table EXCHANGE PARTITION p1 WITH TABLE non_partitioned_table` は、`partitioned_table` テーブルの `p1` パーティションを `non_partitioned_table` テーブルと入れ替えます。

パーティションに入れ替えるすべての行がパーティション定義と一致していることを確認してください。そうでない場合、そのステートメントは失敗します。

TiDBには `EXCHANGE PARTITION` に影響を与える特定の機能があります。テーブル構造にこれらの機能が含まれる場合、`EXCHANGE PARTITION` が [MySQLのEXCHANGE PARTITIONの条件](https://dev.mysql.com/doc/refman/8.0/en/partitioning-management-exchange.html) を満たすようにする必要があります。同時に、これらの特定の機能がパーティション化された表と非パーティション化された表の両方で同じように定義されていることを確認してください。これらの特定の機能には次のものが含まれます:

<CustomContent platform="tidb">

* [SQLにおける配置ルール](/placement-rules-in-sql.md)：配置ポリシーが同じです。

</CustomContent>

* [TiFlash](/tikv-overview.md)：TiFlash複製の数が同じです。
* [クラスタ化インデックス](/clustered-indexes.md)：パーティション化された表と非パーティション化された表の両方が `CLUSTERED` または `NONCLUSTERED` です。

さらに、`EXCHANGE PARTITION` と他のコンポーネントとの互換性には制限があります。パーティション化された表と非パーティション化された表の両方は、同じ定義を持っている必要があります。

- TiFlash: パーティション化された表と非パーティション化された表のTiFlashレプリカ定義が異なる場合、`EXCHANGE PARTITION` 操作は実行できません。
- TiCDC：TiCDCは、パーティション化されたテーブルと非パーティション化されたテーブルの両方にプライマリキーまたはユニークキーがある場合にのみ、`EXCHANGE PARTITION`操作を複製します。それ以外の場合、TiCDCはこの操作をレプリケートしません。
- TiDB Lightning および BR：TiDB Lightningを使用してインポートする場合、またはBRを使用してリストアする場合、`EXCHANGE PARTITION`操作は実行されません。

### RANGE、RANGE COLUMNS、LIST、およびLIST COLUMNS パーティションの管理

このセクションでは、下記のSQLステートメントで作成されたパーティション化されたテーブルを例として使用し、RangeとListパーティションを管理する方法を示します。

```sql
CREATE TABLE members (
    id int,
    fname varchar(255),
    lname varchar(255),
    dob date,
    data json
)
PARTITION BY RANGE (YEAR(dob)) (
 PARTITION pBefore1950 VALUES LESS THAN (1950),
 PARTITION p1950 VALUES LESS THAN (1960),
 PARTITION p1960 VALUES LESS THAN (1970),
 PARTITION p1970 VALUES LESS THAN (1980),
 PARTITION p1980 VALUES LESS THAN (1990),
 PARTITION p1990 VALUES LESS THAN (2000));

CREATE TABLE member_level (
 id int,
 level int,
 achievements json
)
PARTITION BY LIST (level) (
 PARTITION l1 VALUES IN (1),
 PARTITION l2 VALUES IN (2),
 PARTITION l3 VALUES IN (3),
 PARTITION l4 VALUES IN (4),
 PARTITION l5 VALUES IN (5));
```

#### パーティションの削除

```sql
ALTER TABLE members DROP PARTITION p1990;

ALTER TABLE member_level DROP PARTITION l5;
```

#### パーティションの切り捨て

```sql
ALTER TABLE members TRUNCATE PARTITION p1980;

ALTER TABLE member_level TRUNCATE PARTITION l4;
```

#### パーティションの追加

```sql
ALTER TABLE members ADD PARTITION (PARTITION `p1990to2010` VALUES LESS THAN (2010));

ALTER TABLE member_level ADD PARTITION (PARTITION l5_6 VALUES IN (5,6));
```

Rangeパーティション化されたテーブルについては、`ADD PARTITION`は既存の最後のパーティションの後に新しいパーティションを追加します。既存のパーティションと比較して、新しいパーティションの`VALUES LESS THAN`で定義される値は大きくなければなりません。そうでない場合、エラーが報告されます:

```sql
ALTER TABLE members ADD PARTITION (PARTITION p1990 VALUES LESS THAN (2000));
```

```
ERROR 1493 (HY000): VALUES LESS THAN value must be strictly increasing for each partition
```

#### パーティションの再編成

パーティションを分割する:

```sql
ALTER TABLE members REORGANIZE PARTITION `p1990to2010` INTO
(PARTITION p1990 VALUES LESS THAN (2000),
 PARTITION p2000 VALUES LESS THAN (2010),
 PARTITION p2010 VALUES LESS THAN (2020),
 PARTITION p2020 VALUES LESS THAN (2030),
 PARTITION pMax VALUES LESS THAN (MAXVALUE));

ALTER TABLE member_level REORGANIZE PARTITION l5_6 INTO
(PARTITION l5 VALUES IN (5),
 PARTITION l6 VALUES IN (6));
```

パーティションを結合する:

```sql
ALTER TABLE members REORGANIZE PARTITION pBefore1950,p1950 INTO (PARTITION pBefore1960 VALUES LESS THAN (1960));

ALTER TABLE member_level REORGANIZE PARTITION l1,l2 INTO (PARTITION l1_2 VALUES IN (1,2));
```

パーティショニングスキームの定義を変更する:

```sql
ALTER TABLE members REORGANIZE PARTITION pBefore1960,p1960,p1970,p1980,p1990,p2000,p2010,p2020,pMax INTO
(PARTITION p1800 VALUES LESS THAN (1900),
 PARTITION p1900 VALUES LESS THAN (2000),
 PARTITION p2000 VALUES LESS THAN (2100));

ALTER TABLE member_level REORGANIZE PARTITION l1_2,l3,l4,l5,l6 INTO
(PARTITION lOdd VALUES IN (1,3,5),
 PARTITION lEven VALUES IN (2,4,6));
```

パーティションの再編成時に、以下の重要な点に注意する必要があります：

- パーティションの再編成（パーティションの結合または分割を含む）により、一覧表示されたパーティションが新しいパーティション定義の新しいセットに変わりますが、パーティショニングのタイプ（たとえば、ListタイプをRangeタイプに変更するか、Range COLUMNSタイプをRangeタイプに変更するか）を変更することはできません。

- Rangeパーティションテーブルについて、隣接するパーティションのみを再編成できます。

    ```sql
    ALTER TABLE members REORGANIZE PARTITION p1800,p2000 INTO (PARTITION p2000 VALUES LESS THAN (2100));
    ```

    ```
    ERROR 8200 (HY000): Unsupported REORGANIZE PARTITION of RANGE; not adjacent partitions
    ```

- Rangeパーティション化されたテーブルについて、範囲の終わりを変更するには、`VALUES LESS THAN`で定義された新しい終わりは、最後のパーティション内の既存の行をカバーしなければなりません。そうでない場合、既存の行にフィットせず、エラーが報告されます:

    ```sql
    INSERT INTO members VALUES (313, "John", "Doe", "2022-11-22", NULL);
    ALTER TABLE members REORGANIZE PARTITION p2000 INTO (PARTITION p2000 VALUES LESS THAN (2050)); -- このステートメントは期待どおりに機能します。なぜなら、2050は既存の行をカバーしています。
    ALTER TABLE members REORGANIZE PARTITION p2000 INTO (PARTITION p2000 VALUES LESS THAN (2020)); -- このステートメントはエラーとなります。なぜなら、2022は新しい範囲に適合しません。
    ```

    ```
    ERROR 1526 (HY000): Table has no partition for value 2022
    ```

- Listパーティション化されたテーブルについて、パーティションで定義された値のセットを変更するには、新しい定義はそのパーティション内の既存の値をカバーしなければなりません。そうでない場合、エラーが報告されます:

    ```sql
    INSERT INTO member_level (id, level) values (313, 6);
    ALTER TABLE member_level REORGANIZE PARTITION lEven INTO (PARTITION lEven VALUES IN (2,4));
    ```

    ```
    ERROR 1526 (HY000): Table has no partition for value 6
    ```

- パーティションを再編成すると、対応するパーティションの統計情報が古くなるため、次の警告が表示されます。この場合は、[`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md)ステートメントを使用して統計情報を更新することができます。

    ```sql
    +---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Level   | Code | Message                                                                                                                                                |
    +---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Warning | 1105 | The statistics of related partitions will be outdated after reorganizing partitions. Please use 'ANALYZE TABLE' statement if you want to update it now |
    +---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
    1 row in set (0.00 sec)
    ```

### ハッシュおよびキー パーティションの管理

このセクションでは、下記のSQLステートメントで作成されたパーティション化されたテーブルを例として使用し、ハッシュパーティションの管理方法を示します。キーパーティションの場合も同じ管理ステートメントを使用できます。

```sql
CREATE TABLE example (
  id INT PRIMARY KEY,
  data VARCHAR(1024)
)
PARTITION BY HASH(id)
PARTITIONS 2;
```

#### パーティションの数を増やす

`example`テーブルのパーティション数を1増やす（2から3に）:

```sql
ALTER TABLE example ADD PARTITION PARTITIONS 1;
```

また、パーティション定義を追加することでパーテイションオプションを指定することもできます。たとえば、次のステートメントを使用して、パーティション数を3から5に増やし、新しく追加されたパーティションの名前を `pExample4` および `pExample5` として指定することができます:

```sql
ALTER TABLE example ADD PARTITION
(PARTITION pExample4 COMMENT = 'not p3, but pExample4 instead',
 PARTITION pExample5 COMMENT = 'not p4, but pExample5 instead');
```

#### パーティションの数を減らす

RangeおよびListパーティショニングとは異なり、ハッシュおよびキーパーティショニングでは `DROP PARTITION` はサポートされていませんが、`COALESCE PARTITION`を使用してパーティション数を減らしたり、`TRUNCATE PARTITION`を使用して特定のパーティションからすべてのデータを削除したりすることができます。

`example`テーブルのパーティション数を1減らす（5から4に）:

```sql
ALTER TABLE example COALESCE PARTITION 1;
```

> **Note:**
>
> ハッシュまたはキーパーティション化されたテーブルのパーティション数を変更するプロセスは、パーティションを新しい数にコピーして再編成するため、ハッシュまたはキーパーティション化されたテーブルのパーティション数を変更した後、古い統計情報に関する次の警告が表示されます。この場合は、[`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md)ステートメントを使用して統計情報を更新することができます。
>
> ```sql
> +---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
> | Level   | Code | Message                                                                                                                                                |
> +---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
> | Warning | 1105 | The statistics of related partitions will be outdated after reorganizing partitions. Please use 'ANALYZE TABLE' statement if you want to update it now |
> +---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
> 1 row in set (0.00 sec)
> ```
`example`テーブルの現在の構造をよりよく理解するためには、次のようにして`example`テーブルを再作成するSQLステートメントを表示できます。

```sql
SHOW CREATE TABLE\G
```

```
*************************** 1. row ***************************
       Table: example
Create Table: CREATE TABLE `example` (
  `id` int(11) NOT NULL,
  `data` varchar(1024) DEFAULT NULL,
  PRIMARY KEY (`id`) /*T![clustered_index] CLUSTERED */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
PARTITION BY HASH (`id`)
(PARTITION `p0`,
 PARTITION `p1`,
 PARTITION `p2`,
 PARTITION `pExample4` COMMENT 'not p3, but pExample4 instead')
1 row in set (0.01 sec)
```

#### パーティションの切り捨て

パーティションからすべてのデータを削除します。

```sql
ALTER TABLE example TRUNCATE PARTITION p0;
```

```
Query OK, 0 rows affected (0.03 sec)
```

### パーティションテーブルを非パーティションテーブルに変換

パーティションテーブルを非パーティションテーブルに変換するには、次のステートメントを使用します。これにより、パーティションが削除され、テーブルのすべての行がコピーされ、テーブルのインデックスがオンラインで再作成されます。

```sql
ALTER TABLE <table_name> REMOVE PARTITIONING
```

たとえば、`members`のパーティションテーブルを非パーティションテーブルに変換するには、次のステートメントを実行できます。

```sql
ALTER TABLE members REMOVE PARTITIONING
```

### 既存のテーブルをパーティション化

既存の非パーティションテーブルをパーティション化したり、既存のパーティションテーブルのパーティションタイプを変更したりするには、次のステートメントを使用します。これにより、すべての行がコピーされ、新しいパーティション定義に応じてテーブルのインデックスがオンラインで再作成されます。

```sql
ALTER TABLE <table_name> PARTITION BY <new partition type and definitions>
```

例：

既存の`members`テーブルをHASHパーティション化して10つのパーティションに変換するには、次のステートメントを実行します。

```sql
ALTER TABLE members PARTITION BY HASH(id) PARTITIONS 10;
```

既存の`member_level`テーブルをRANGEパーティションテーブルに変換するには、次のステートメントを実行します。

```sql
ALTER TABLE member_level PARTITION BY RANGE(level)
(PARTITION pLow VALUES LESS THAN (1),
 PARTITION pMid VALUES LESS THAN (3),
 PARTITION pHigh VALUES LESS THAN (7)
 PARTITION pMax VALUES LESS THAN (MAXVALUE));
```

## パーティション枝刈り

[パーティション枝刈り](/partition-pruning.md)は、非常にシンプルな考え方に基づいた最適化です - 一致しないパーティションをスキャンしないようにします。

パーティション化された`t1`テーブルを作成すると仮定してください：

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (
    fname VARCHAR(50) NOT NULL,
    lname VARCHAR(50) NOT NULL,
    region_code TINYINT UNSIGNED NOT NULL,
    dob DATE NOT NULL
)

PARTITION BY RANGE( region_code ) (
    PARTITION p0 VALUES LESS THAN (64),
    PARTITION p1 VALUES LESS THAN (128),
    PARTITION p2 VALUES LESS THAN (192),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

この`SELECT`ステートメントの結果を取得したい場合：

{{< copyable "sql" >}}

```sql
SELECT fname, lname, region_code, dob
    FROM t1
    WHERE region_code > 125 AND region_code < 130;
```

結果は`p1`または`p2`パーティションに含まれることが明らかであり、`p1`および`p2`で一致する行を検索する必要があります。不要なパーティションを除外することを"枝刈り"と
```sql
セレクト* fromt where dt > '2020-04-18';
```

例外は`floor(unix_timestamp())`です。パーティション式として使用することができます。TiDBはその場合に最適化を行い、パーティションプルーニングがサポートされています。

{{<コピー可能"sql">}}

```sql
create table t (ts timestamp(3) not null default current_timestamp(3))
パーティションごとに範囲を指定する（floor(unix_timestamp(ts))）(
            partition p0 values less than (unix_timestamp('2020-04-01 00:00:00')),
            partition p1 values less than (unix_timestamp('2020-05-01 00:00:00')));
select* from t where ts > '2020-04-18 02:00:42.123';
```

## パーティション選択

`SELECT`ステートメントは`PARTITION`オプションを使用して実装される、パーティション選択をサポートしています。

{{<コピー可能"sql">}}

```sql
SET @@sql_mode = '';

CREATE TABLE employees  (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    fname VARCHAR(25) NOT NULL,
    lname VARCHAR(25) NOT NULL,
    store_id INT NOT NULL,
    department_id INT NOT NULL
)

PARTITION BY RANGE(id)  (
    PARTITION p0 VALUES LESS THAN (5),
    PARTITION p1 VALUES LESS THAN (10),
    PARTITION p2 VALUES LESS THAN (15),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);

INSERT INTO employees VALUES
    ('', 'Bob', 'Taylor', 3, 2), ('', 'Frank', 'Williams', 1, 2),
    ('', 'Ellen', 'Johnson', 3, 4), ('', 'Jim', 'Smith', 2, 4),
    ('', 'Mary', 'Jones', 1, 1), ('', 'Linda', 'Black', 2, 3),
    ('', 'Ed', 'Jones', 2, 1), ('', 'June', 'Wilson', 3, 1),
    ('', 'Andy', 'Smith', 1, 3), ('', 'Lou', 'Waters', 2, 4),
    ('', 'Jill', 'Stone', 1, 4), ('', 'Roger', 'White', 3, 2),
    ('', 'Howard', 'Andrews', 1, 2), ('', 'Fred', 'Goldberg', 3, 3),
    ('', 'Barbara', 'Brown', 2, 3), ('', 'Alice', 'Rogers', 2, 2),
    ('', 'Mark', 'Morgan', 3, 3), ('', 'Karen', 'Cole', 3, 2);
```

`p1`パーティションに格納されている行を表示できます:

{{<コピー可能"sql">}}

```sql
SELECT * FROM employees PARTITION (p1);
```

```
+----|-------|--------|----------|---------------+
| id | fname | lname  | store_id | department_id |
+----|-------|--------|----------|---------------+
|  5 | Mary  | Jones  |        1 |             1 |
|  6 | Linda | Black  |        2 |             3 |
|  7 | Ed    | Jones  |        2 |             1 |
|  8 | June  | Wilson |        3 |             1 |
|  9 | Andy  | Smith  |        1 |             3 |
+----|-------|--------|----------|---------------+
5 rows in set (0.00 sec)
```

複数のパーティション内の行を取得したい場合は、コンマで区切られたパーティション名のリストを使用できます。例えば、`SELECT * FROM employees PARTITION (p1, p2)` は`p1` と `p2` のパーティション内のすべての行を返します。

パーティション選択を使用すると、`WHERE`条件や`ORDER BY`、`LIMIT`などのオプションを使用することができます。`HAVING` や `GROUP BY` などの集計オプションを使用することもサポートされています。

{{<コピー可能"sql">}}

```sql
SELECT * FROM employees PARTITION (p0, p2)
    WHERE lname LIKE 'S%';
```

```
+----|-------|-------|----------|---------------+
| id | fname | lname | store_id | department_id |
+----|-------|-------|----------|---------------+
|  4 | Jim   | Smith |        2 |             4 |
| 11 | Jill  | Stone |        1 |             4 |
+----|-------|-------|----------|---------------+
2 rows in set (0.00 sec)
```

{{<コピー可能"sql">}}

```sql
SELECT id, CONCAT(fname, ' ', lname) AS name
    FROM employees PARTITION (p0) ORDER BY lname;
```

```
+----|----------------+
| id | name           |
+----|----------------+
|  3 | Ellen Johnson  |
|  4 | Jim Smith      |
|  1 | Bob Taylor     |
|  2 | Frank Williams |
+----|----------------+
4 rows in set (0.06 sec)
```

{{<コピー可能"sql">}}

```sql
SELECT store_id, COUNT(department_id) AS c
    FROM employees PARTITION (p1,p2,p3)
    GROUP BY store_id HAVING c > 4;
```

```
+---|----------+
| c | store_id |
+---|----------+
| 5 |        2 |
| 5 |        3 |
+---|----------+
2 rows in set (0.00 sec)
```

パーティション選択は、範囲パーティショニングやハッシュパーティショニングを含む、すべての種類の表パーティションでサポートされています。ハッシュパーティションの場合、パーティション名が指定されていない場合、`p0`、`p1`、`p2`、...、または`pN-1`が自動的にパーティション名として使用されます。

`INSERT...SELECT`内の`SELECT`でもパーティション選択が使用できます。

## パーティションの制約と制限

こちらでは、TiDBにおけるパーティションテーブルの制約と制限について紹介します。

### パーティションキー、主キー、一意キー

このセクションでは、パーティションキーと主キー、一意キーの関係について説明します。この関係に関するルールは次のとおりです: **テーブルのすべての一意キーは、テーブルのパーティション式で使用されているすべての列を使用する必要があります**。これには、テーブルの主キーも含まれます。

例えば、以下のテーブル作成ステートメントは無効です:

{{<コピー可能"sql">}}

```sql
CREATE TABLE t1 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1, col2)
)

PARTITION BY HASH(col3)
PARTITIONS 4;

CREATE TABLE t2 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1),
    UNIQUE KEY (col3)
)

PARTITION BY HASH(col1 + col3)
PARTITIONS 4;
```

各ケースで、提案されたテーブルには、パーティション式で使用されているすべての列を含まない少なくとも1つの一意キーがあります。

以下のステートメントは有効です:

{{<コピー可能"sql">}}

```sql
CREATE TABLE t1 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1, col2, col3)
)

PARTITION BY HASH(col3)
PARTITIONS 4;

CREATE TABLE t2 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1, col3)
)

PARTITION BY HASH(col1 + col3)
PARTITIONS 4;
```

以下の例では、エラーが表示されます:

{{<コピー可能"sql">}}

```sql
CREATE TABLE t3 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1, col2),
    UNIQUE KEY (col3)
)

PARTITION BY HASH(col1 + col3)
PARTITIONS 4;
```

```
ERROR 1491 (HY000): A PRIMARY KEY must include all columns in the table's partitioning function
```

`col1`と`col3`の両方が提案されたパーティションキーに含まれていますが、これらの列のいずれもテーブルのすべての一意キーに含まれていないため、`CREATE TABLE` ステートメントが失敗します。以下の変更後、`CREATE TABLE` ステートメントは有効になります:

{{<コピー可能"sql">}}

```sql
CREATE TABLE t3 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1, col2, col3),
    UNIQUE KEY (col1, col3)
)
PARTITION BY HASH(col1 + col3)
    PARTITIONS 4;
```

以下のテーブルは、ユニークキーに属するカラムをパーティションキーに含める方法がないため、まったくパーティション化することができません。

```sql
CREATE TABLE t4 (
    col1 INT NOT NULL,
    col2 INT NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    UNIQUE KEY (col1, col3),
    UNIQUE KEY (col2, col4)
);
```

すべてのプライマリキーは定義上ユニークキーであるため、次の2つのステートメントは無効です。

```sql
CREATE TABLE t5 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    PRIMARY KEY(col1, col2)
)

PARTITION BY HASH(col3)
PARTITIONS 4;

CREATE TABLE t6 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    PRIMARY KEY(col1, col3),
    UNIQUE KEY(col2)
)

PARTITION BY HASH( YEAR(col2) )
PARTITIONS 4;
```

上記の例では、プライマリキーにはパーティショニング式で参照されるすべてのカラムが含まれていません。必要なカラムをプライマリキーに追加した後、`CREATE TABLE`ステートメントが有効になります。

```sql
CREATE TABLE t5 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    PRIMARY KEY(col1, col2, col3)
)
PARTITION BY HASH(col3)
PARTITIONS 4;
CREATE TABLE t6 (
    col1 INT NOT NULL,
    col2 DATE NOT NULL,
    col3 INT NOT NULL,
    col4 INT NOT NULL,
    PRIMARY KEY(col1, col2, col3),
    UNIQUE KEY(col2)
)
PARTITION BY HASH( YEAR(col2) )
PARTITIONS 4;
```

テーブルにユニークキーもプライマリキーも持たない場合、この制限は適用されません。DDLステートメントを使用してテーブルを変更する際には、一意なインデックスを追加するときにこの制限を考慮する必要があります。たとえば、次のようにパーティション化されたテーブルを作成する場合：

```sql
CREATE TABLE t_no_pk (c1 INT, c2 INT)
    PARTITION BY RANGE(c1) (
        PARTITION p0 VALUES LESS THAN (10),
        PARTITION p1 VALUES LESS THAN (20),
        PARTITION p2 VALUES LESS THAN (30),
        PARTITION p3 VALUES LESS THAN (40)
    );
```

```
Query OK, 0 rows affected (0.12 sec)
```

`ALTER TABLE`ステートメントを使用してユニークでないインデックスを追加できます。ただし、ユニークインデックスを追加する場合、`c1`カラムをユニークインデックスに含める必要があります。

パーティション化されたテーブルを使用する場合、プリフィックスインデックスをユニークな属性として指定することはできません。

```sql
CREATE TABLE t (a varchar(20), b blob,
    UNIQUE INDEX (a(5)))
    PARTITION by range columns (a) (
    PARTITION p0 values less than ('aaaaa'),
    PARTITION p1 values less than ('bbbbb'),
    PARTITION p2 values less than ('ccccc'));
```

```sql
ERROR 1503 (HY000): A UNIQUE INDEX must include all columns in the table's partitioning function
```

### 関数に関連するパーティション化の制限

以下のリストに示されている関数のみが、パーティション化式で許可されています。

```
ABS()
CEILING()
DATEDIFF()
DAY()
DAYOFMONTH()
DAYOFWEEK()
DAYOFYEAR()
EXTRACT() (WEEK指定子を伴うEXTRACT()関数を参照)
FLOOR()
HOUR()
MICROSECOND()
MINUTE()
MOD()
MONTH()
QUARTER()
SECOND()
TIME_TO_SEC()
TO_DAYS()
TO_SECONDS()
UNIX_TIMESTAMP() (TIMESTAMPカラムを伴う)
WEEKDAY()
YEAR()
YEARWEEK()
```

### MySQLとの互換性

現在、TiDBはRangeパーティショニング、Range COLUMNSパーティショニング、Listパーティショニング、List COLUMNSパーティショニング、Hashパーティショニング、およびKeyパーティショニングをサポートしています。TiDBではまだサポートされていないMySQLで利用可能な他のパーティショニングタイプ。

現在、TiDBはキーのパーティション化において空のパーティション列リストを使用することをサポートしていません。

パーティション管理に関して、ボトムの実装でデータの移動を必要とする操作は現在サポートされていません。これには、ハッシュでパーティション化されたテーブルのパーティション数を調整する、Rangeでパーティション化されたテーブルのRangeを変更する、およびパーティションをマージするなどが含まれます。

サポートされていないパーティショニングタイプの場合、TiDBでテーブルを作成すると、パーティショニング情報は無視され、警告が報告されて通常の形式でテーブルが作成されます。

`LOAD DATA`構文は現在TiDBではパーティション選択をサポートしていません。

```sql
create table t (id int, val int) partition by hash(id) partitions 4;
```

通常の`LOAD DATA`操作はサポートされています。

```sql
load local data infile "xxx" into t ...
```

ただし、`LOAD DATA`はパーティション選択をサポートしていません。

```sql
load local data infile "xxx" into t partition (p1)...
```

パーティション化されたテーブルでは、`select * from t`によって返される結果はパーティション間で順序付けられていません。これはMySQLでの結果とは異なります。MySQLではパーティション内で順不同ですが、パーティション間で順序付けられています。

```sql
create table t (id int, val int) partition by range (id) (
    partition p0 values less than (3),
    partition p1 values less than (7),
    partition p2 values less than (11));
```

```
Query OK, 0 rows affected (0.10 sec)
```

```sql
insert into t values (1, 2), (3, 4),(5, 6),(7,8),(9,10);
```

```
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0
```

TiDBでは、異なる結果が返されます。たとえば：

```sql
select * from t;
```

```
+------|------+
| id   | val  |
+------|------+
|    7 |    8 |
|    9 |   10 |
|    1 |    2 |
|    3 |    4 |
|    5 |    6 |
+------|------+
5 rows in set (0.00 sec)
```

これに対して、MySQLでの結果は：

```sql
select * from t;
```

```
+------|------+
| id   | val  |
+------|------+
|    1 |    2 |
|    3 |    4 |
|    5 |    6 |
|    7 |    8 |
|    9 |   10 |
+------|------+
5 rows in set (0.00 sec)
```

`tidb_enable_list_partition`環境変数は、パーティション化されたテーブルの機能を有効にするかどうかを制御します。この変数が`OFF`に設定されている場合、テーブルが通常のテーブルとして作成されるときに、パーティション情報は無視されます。

この変数はテーブルの作成時のみに使用されます。テーブルが作成された後、この変数の値を変更しても効果はありません。詳細については、[system variables](/system-variables.md#tidb_enable_list_partition-new-in-v50)を参照してください。

### 動的プルーニングモード

TiDBは、`dynamic`モードまたは`static`モードでパーティション化されたテーブルにアクセスします。v6.3.0以降では、デフォルトで`dynamic`モードが使用されます。ただし、動的パーティション化は、フルテーブルレベルの統計情報、またはGlobalStatsが収集された後にのみ有効です。GlobalStatsが収集される前に、TiDBは代わりに`static`モードを使用します。GlobalStatsの詳細については、[Collect statistics of partitioned tables in dynamic pruning mode](/statistics.md#collect-statistics-of-partitioned-tables-in-dynamic-pruning-mode)を参照してください。

```sql
set @@session.tidb_partition_prune_mode = 'dynamic'
```

手動のANALYZEおよび通常のクエリは、セッションレベルの`tidb_partition_prune_mode`設定を使用します。バックグラウンドでの`auto-analyze`操作では、グローバルな`tidb_partition_prune_mode`設定が使用されます。

`static`モードでは、パーティション化されたテーブルはパーティションレベルの統計情報を使用します。`dynamic`モードでは、パーティション化されたテーブルはテーブルレベルのGlobalStatsを使用します。

`static`モードから`dynamic`モードに切り替える際には、統計情報をチェックして手動で収集する必要があります。これは、`dynamic`モードに切り替えた後、パーティション化されたテーブルがパーティションレベルの統計情報のみを持ち、テーブルレベルの統計情報を持たなくなるためです。GlobalStatsは、次の`auto-analyze`操作時にのみ収集されます。

```sql
set session tidb_partition_prune_mode = 'dynamic';
show stats_meta where table_name like "t";
```
+---------+------------+----------------+---------------------+--------------+-----------+
| Db_name | Table_name | Partition_name | Update_time         | Modify_count | Row_count |
+---------+------------+----------------+---------------------+--------------+-----------+
| test    | t          | p0             | 2022-05-27 20:23:34 |            1 |         2 |
| test    | t          | p1             | 2022-05-27 20:23:34 |            2 |         4 |
| test    | t          | p2             | 2022-05-27 20:23:34 |            2 |         4 |
+---------+------------+----------------+---------------------+--------------+-----------+
3 rows in set (0.01 sec)

---

SQLステートメントが正しい統計を使用するために、グローバル`dynamic`プルーニングモードを有効にした後は、テーブルまたはテーブルのパーティションに手動で`analyze`をトリガーしてGlobalStatsを取得する必要があります。

{{< copyable "sql" >}}

```sql
analyze table t partition p1;
show stats_meta where table_name like "t";
```

```
+---------+------------+----------------+---------------------+--------------+-----------+
| Db_name | Table_name | Partition_name | Update_time         | Modify_count | Row_count |
+---------+------------+----------------+---------------------+--------------+-----------+
| test    | t          | global         | 2022-05-27 20:50:53 |            0 |         5 |
| test    | t          | p0             | 2022-05-27 20:23:34 |            1 |         2 |
| test    | t          | p1             | 2022-05-27 20:50:52 |            0 |         2 |
| test    | t          | p2             | 2022-05-27 20:50:08 |            0 |         2 |
+---------+------------+----------------+---------------------+--------------+-----------+
4 rows in set (0.00 sec)
```

`analyze`プロセス中に下記の警告が表示される場合は、パーティションの統計情報が整合性がなく、これらのパーティションまたはテーブル全体の統計情報を再収集する必要があります。

```
| Warning | 8244 | Build table: `t` column: `a` global-level stats failed due to missing partition-level column stats, please run analyze table to refresh columns of all partitions
```

また、スクリプトを使用してパーティションされたすべてのテーブルの統計情報を更新することもできます。詳細については、[動的プルーニングモードでパーティションテーブルの統計情報を更新する](#update-statistics-of-partitioned-tables-in-dynamic-pruning-mode)を参照してください。

テーブルレベルの統計情報が整ったら、全体の動的プルーニングモードを有効にして、すべてのSQLステートメントおよび`auto-analyze`操作に効果をもたらします。

{{< copyable "sql" >}}

```sql
set global tidb_partition_prune_mode = dynamic
```

`static`モードでは、TiDBは各パーティションを個別にアクセスし、複数のオペレータを使用して結果をマージします。次の例は、TiDBが`Union`を使用して2つの対応するパーティションの結果をマージする単純な読み取り操作の例です。

{{< copyable "sql" >}}

```sql
mysql> create table t1(id int, age int, key(id)) partition by range(id) (
        partition p0 values less than (100),
        partition p1 values less than (200),
        partition p2 values less than (300),
        partition p3 values less than (400));
Query OK, 0 rows affected (0.01 sec)

mysql> explain select * from t1 where id < 150;
```

```
+------------------------------+----------+-----------+------------------------+--------------------------------+
| id                           | estRows  | task      | access object          | operator info                  |
+------------------------------+----------+-----------+------------------------+--------------------------------+
| PartitionUnion_9             | 6646.67  | root      |                        |                                |
| ├─TableReader_12             | 3323.33  | root      |                        | data:Selection_11              |
| │ └─Selection_11             | 3323.33  | cop[tikv] |                        | lt(test.t1.id, 150)            |
| │   └─TableFullScan_10       | 10000.00 | cop[tikv] | table:t1, partition:p0 | keep order:false, stats:pseudo |
| └─TableReader_18             | 3323.33  | root      |                        | data:Selection_17              |
|   └─Selection_17             | 3323.33  | cop[tikv] |                        | lt(test.t1.id, 150)            |
|     └─TableFullScan_16       | 10000.00 | cop[tikv] | table:t1, partition:p1 | keep order:false, stats:pseudo |
+------------------------------+----------+-----------+------------------------+--------------------------------+
7 rows in set (0.00 sec)
```

`dynamic`モードでは、各オペレータが複数のパーティションに直接アクセスできるようになります。したがって、TiDBはもはや`Union`を使用しません。

{{< copyable "sql" >}}

```sql
mysql> set @@session.tidb_partition_prune_mode = 'dynamic';
Query OK, 0 rows affected (0.00 sec)

mysql> explain select * from t1 where id < 150;
+-------------------------+----------+-----------+-----------------+--------------------------------+
| id                      | estRows  | task      | access object   | operator info                  |
+-------------------------+----------+-----------+-----------------+--------------------------------+
| TableReader_7           | 3323.33  | root      | partition:p0,p1 | data:Selection_6               |
| └─Selection_6           | 3323.33  | cop[tikv] |                 | lt(test.t1.id, 150)            |
|   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t1        | keep order:false, stats:pseudo |
+-------------------------+----------+-----------+-----------------+--------------------------------+
3 rows in set (0.00 sec)
```

上記のクエリ結果から、実行計画内の`Union`オペレータが消えており、またパーティションプルーニングが引き続き効果を持っており、実行計画が`p0`および`p1`のみにアクセスしていることがわかります。

`dynamic`モードは実行計画をよりシンプルかつ明確にし、Unionの同時実行の問題を回避することができます。さらに`dynamic`モードでは、`static`モードでは使用できないIndexJoinを使用した実行計画も可能です。（以下の例を参照）

**例1**：以下の例は、`static`モードでIndexJoinを使用した実行計画でクエリが実行されています。

{{< copyable "sql" >}}

```sql
mysql> create table t1 (id int, age int, key(id)) partition by range(id)
    (partition p0 values less than (100),
     partition p1 values less than (200),
     partition p2 values less than (300),
     partition p3 values less than (400));
Query OK, 0 rows affected (0,08 sec)

mysql> create table t2 (id int, code int);
Query OK, 0 rows affected (0.01 sec)

mysql> set @@tidb_partition_prune_mode = 'static';
Query OK, 0 rows affected (0.00 sec)

mysql> explain select /*+ TIDB_INLJ(t1, t2) */ t1.* from t1, t2 where t2.code = 0 and t2.id = t1.id;
+--------------------------------+----------+-----------+------------------------+------------------------------------------------+
| id                             | estRows  | task      | access object          | operator info                                  |
+--------------------------------+----------+-----------+------------------------+------------------------------------------------+
| HashJoin_13                    | 12.49    | root      |                        | inner join, equal:[eq(test.t1.id, test.t2.id)] |
| ├─TableReader_42(Build)        | 9.99     | root      |                        | data:Selection_41                              |
| │ └─Selection_41               | 9.99     | cop[tikv] |                        | eq(test.t2.code, 0), not(isnull(test.t2.id))   |
| │   └─TableFullScan_40         | 10000.00 | cop[tikv] | table:t2               | keep order:false, stats:pseudo                 |
| └─PartitionUnion_15(Probe)     | 39960.00 | root      |                        |                                                |
|   ├─TableReader_18             | 9990.00  | root      |                        | data:Selection_17                              |
|   │ └─Selection_17             | 9990.00  | cop[tikv] |                        | not(isnull(test.t1.id))                        |
|   │   └─TableFullScan_16       | 10000.00 | cop[tikv] | table:t1, partition:p0 | keep order:false, stats:pseudo                 |
|   ├─TableReader_24             | 9990.00  | root      |                        | data:Selection_23                              |
```
```
| │ └─Selection_23               | 9990.00  | cop[tikv] |                        | not(isnull(test.t1.id))                        |
| │   └─TableFullScan_22         | 10000.00 | cop[tikv] | table:t1, partition:p1       | keep order:false, stats:pseudo                 |
| ├─TableReader_30               | 9990.00  | root      |                        | data:Selection_29                              |
| │ └─Selection_29               | 9990.00  | cop[tikv] |                        | not(isnull(test.t1.id))                        |
| │   └─TableFullScan_28         | 10000.00 | cop[tikv] | table:t1, partition:p2       | keep order:false, stats:pseudo                 |
| └─TableReader_36               | 9990.00  | root      |                        | data:Selection_35                              |
|   └─Selection_35               | 9990.00  | cop[tikv] |                        | not(isnull(test.t1.id))                        |
|     └─TableFullScan_34         | 10000.00 | cop[tikv] | table:t1, partition:p3       | keep order:false, stats:pseudo                 |
+--------------------------------+----------+-----------+------------------------+------------------------------------------------+
```

**Example 2**
以下の例では、`dynamic`モードでクエリを実行し、IndexJoinを使用した実行計画でクエリが実行されます：

```sql
mysql> set @@tidb_partition_prune_mode = 'dynamic';
クエリが実行されました: 0件影響を受けました (0.00 秒)

mysql> explain select /*+ TIDB_INLJ(t1, t2) */ t1.* from t1, t2 where t2.code = 0 and t2.id = t1.id;
+---------------------------------+----------+-----------+------------------------+---------------------------------------------------------------------------------------------------------------------+
| id                              | estRows  | task      | access object          | operator info                                                                                                       |
+---------------------------------+----------+-----------+------------------------+---------------------------------------------------------------------------------------------------------------------+
| IndexJoin_11                    | 12.49    | root      |                        | inner join, inner:IndexLookUp_10, outer key:test.t2.id, inner key:test.t1.id, equal cond:eq(test.t2.id, test.t1.id) |
| ├─TableReader_16(Build)         | 9.99     | root      |                        | data:Selection_15                                                                                                   |
| │ └─Selection_15                | 9.99     | cop[tikv] |                        | eq(test.t2.code, 0), not(isnull(test.t2.id))                                                                        |
| │   └─TableFullScan_14          | 10000.00 | cop[tikv] | table:t2               | keep order:false, stats:pseudo                                                                                      |
| └─IndexLookUp_10(Probe)         | 12.49    | root      | partition:all          |                                                                                                                     |
|   ├─Selection_9(Build)          | 12.49    | cop[tikv] |                        | not(isnull(test.t1.id))                                                                                             |
|   │ └─IndexRangeScan_7          | 12.50    | cop[tikv] | table:t1, index:id(id) | range: decided by [eq(test.t1.id, test.t2.id)], keep order:false, stats:pseudo                                      |
|   └─TableRowIDScan_8(Probe)     | 12.49    | cop[tikv] | table:t1               | keep order:false, stats:pseudo                                                                                      |
+---------------------------------+----------+-----------+------------------------+---------------------------------------------------------------------------------------------------------------------+
8 行が返されました (0.00 秒)
```

例2から、`dynamic`モードでは、クエリを実行する際にIndexJoinを使用した実行計画が選択されます。

現在、`static`または`dynamic`のプルーニングモードは、プリペアドステートメントの実行計画キャッシュをサポートしていません。

#### パーティションテーブルの統計情報を動的プルーニングモードで更新する

1. すべてのパーティションテーブルを特定します：

    ```sql
    SELECT DISTINCT CONCAT(TABLE_SCHEMA,'.', TABLE_NAME)
        FROM information_schema.PARTITIONS
        WHERE TIDB_PARTITION_ID IS NOT NULL
        AND TABLE_SCHEMA NOT IN ('INFORMATION_SCHEMA', 'mysql', 'sys', 'PERFORMANCE_SCHEMA', 'METRICS_SCHEMA');
    ```

    ```
    +-------------------------------------+
    | concat(TABLE_SCHEMA,'.',TABLE_NAME) |
    +-------------------------------------+
    | test.t                              |
    +-------------------------------------+
    1 行が返されました (0.02 秒)
    ```

2. すべてのパーティションテーブルの統計情報を更新するためのステートメントを生成します：

    ```sql
    SELECT DISTINCT CONCAT('ANALYZE TABLE ',TABLE_SCHEMA,'.',TABLE_NAME,' ALL COLUMNS;')
        FROM information_schema.PARTITIONS
        WHERE TIDB_PARTITION_ID IS NOT NULL
        AND TABLE_SCHEMA NOT IN ('INFORMATION_SCHEMA','mysql','sys','PERFORMANCE_SCHEMA','METRICS_SCHEMA');
    ```

    ```
    +----------------------------------------------------------------------+
    | concat('ANALYZE TABLE ',TABLE_SCHEMA,'.',TABLE_NAME,' ALL COLUMNS;') |
    +----------------------------------------------------------------------+
    | ANALYZE TABLE test.t ALL COLUMNS;                                    |
    +----------------------------------------------------------------------+
    1 行が返されました (0.01 秒)
    ```

    必要に応じて`ALL COLUMNS`を必要な列に変更できます。

3. バッチ更新ステートメントをファイルにエクスポートします：

    ```shell
    mysql --host xxxx --port xxxx -u root -p -e "SELECT DISTINCT CONCAT('ANALYZE TABLE ',TABLE_SCHEMA,'.',TABLE_NAME,' ALL COLUMNS;') \
        FROM information_schema.PARTITIONS \
        WHERE TIDB_PARTITION_ID IS NOT NULL \
        AND TABLE_SCHEMA NOT IN ('INFORMATION_SCHEMA','mysql','sys','PERFORMANCE_SCHEMA','METRICS_SCHEMA');" | tee gatherGlobalStats.sql
    ```

4. バッチ更新を実行します：
  
    `source`コマンドを実行する前にSQLステートメントを処理します：

    ```
    sed -i "" '1d' gatherGlobalStats.sql --- mac
    sed -i '1d' gatherGlobalStats.sql --- linux
    ```

    ```sql
    SET session tidb_partition_prune_mode = dynamic;
    source gatherGlobalStats.sql
    ```