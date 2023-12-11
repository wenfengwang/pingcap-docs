---
title: CREATE INDEX | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのCREATE INDEXの使用についての概要。
aliases: ['/docs/dev/sql-statements/sql-statement-create-index/','/docs/dev/reference/sql/statements/create-index/']
---

# CREATE INDEX

このステートメントは、既存のテーブルに新しいインデックスを追加します。これは`ALTER TABLE .. ADD INDEX`の代替構文であり、MySQLとの互換性のために含まれています。

## 構文

```ebnf+diagram
CreateIndexStmt ::=
    'CREATE' IndexKeyTypeOpt 'INDEX' IfNotExists Identifier IndexTypeOpt 'ON' TableName '(' IndexPartSpecificationList ')' IndexOptionList IndexLockAndAlgorithmOpt

IndexKeyTypeOpt ::=
    ( 'UNIQUE' | 'SPATIAL' | 'FULLTEXT' )?

IfNotExists ::=
    ( 'IF' 'NOT' 'EXISTS' )?

IndexTypeOpt ::=
    IndexType?

IndexPartSpecificationList ::=
    IndexPartSpecification ( ',' IndexPartSpecification )*

IndexOptionList ::=
    IndexOption*

IndexLockAndAlgorithmOpt ::=
    ( LockClause AlgorithmClause? | AlgorithmClause LockClause? )?

IndexType ::=
    ( 'USING' | 'TYPE' ) IndexTypeName

IndexPartSpecification ::=
    ( ColumnName OptFieldLen | '(' Expression ')' ) Order

IndexOption ::=
    'KEY_BLOCK_SIZE' '='? LengthNum
|   IndexType
|   'WITH' 'PARSER' Identifier
|   'COMMENT' stringLit
|   IndexInvisible

IndexTypeName ::=
    'BTREE'
|   'HASH'
|   'RTREE'

ColumnName ::=
    Identifier ( '.' Identifier ( '.' Identifier )? )?

OptFieldLen ::=
    FieldLen?

IndexNameList ::=
    ( Identifier | 'PRIMARY' )? ( ',' ( Identifier | 'PRIMARY' ) )*

KeyOrIndex ::=
    'Key' | 'Index'
```

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
Query OK, 0 rows affected (0.10 sec)

mysql> INSERT INTO t1 (c1) VALUES (1),(2),(3),(4),(5);
Query OK, 5 rows affected (0.02 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> EXPLAIN SELECT * FROM t1 WHERE c1 = 3;
+-------------------------+----------+-----------+---------------+--------------------------------+
| id                      | estRows  | task      | access object | operator info                  |
+-------------------------+----------+-----------+---------------+--------------------------------+
| TableReader_7           | 10.00    | root      |               | data:Selection_6               |
| └─Selection_6           | 10.00    | cop[tikv] |               | eq(test.t1.c1, 3)              |
|   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+-------------------------+----------+-----------+---------------+--------------------------------+
3 rows in set (0.00 sec)

mysql> CREATE INDEX c1 ON t1 (c1);
Query OK, 0 rows affected (0.30 sec)

mysql> EXPLAIN SELECT * FROM t1 WHERE c1 = 3;
+------------------------+---------+-----------+------------------------+---------------------------------------------+
| id                     | estRows | task      | access object          | operator info                               |
+------------------------+---------+-----------+------------------------+---------------------------------------------+
| IndexReader_6          | 10.00   | root      |                        | index:IndexRangeScan_5                      |
| └─IndexRangeScan_5     | 10.00   | cop[tikv] | table:t1, index:c1(c1) | range:[3,3], keep order:false, stats:pseudo |
+------------------------+---------+-----------+------------------------+---------------------------------------------+
2 rows in set (0.00 sec)

mysql> ALTER TABLE t1 DROP INDEX c1;
Query OK, 0 rows affected (0.30 sec)

mysql> CREATE UNIQUE INDEX c1 ON t1 (c1);
Query OK, 0 rows affected (0.31 sec)
```

## 式インデックス

特定のシナリオでは、クエリのフィルタリング条件が特定の式に基づいています。これらのシナリオでは、通常のインデックスが効果を発揮できないため、クエリはテーブル全体をスキャンするしかありません。式インデックスは、式の上に作成できる特別なインデックスの一種です。式インデックスが作成されると、TiDBは式ベースのクエリにそのインデックスを使用でき、クエリのパフォーマンスが大幅に向上します。

たとえば、`lower(col1)`に基づいてインデックスを作成したい場合は、次のSQLステートメントを実行します：

{{< copyable "sql" >}}

```sql
CREATE INDEX idx1 ON t1 ((lower(col1)));
```

または、次の同等のステートメントを実行できます：

{{< copyable "sql" >}}

```sql
ALTER TABLE t1 ADD INDEX idx1((lower(col1)));
```

テーブルを作成する際にも、式インデックスを指定できます：

{{< copyable "sql" >}}

```sql
CREATE TABLE t1(col1 char(10), col2 char(10), index((lower(col1))));
```

> **注意：**
>
> 式インデックスの式は、`(`および`)`で囲む必要があります。それ以外の場合は、構文エラーが報告されます。

式インデックスは、さまざまな種類の式を含みます。正確性を確保するため、本番環境では完全にテストされた一部の関数のみが式インデックスの作成に許可されます。これにより、本番環境ではこれらの関数のみが式に許可されます。これらの関数は、`tidb_allow_function_for_expression_index`変数をクエリすることで取得できます。現在、許可される関数は次のとおりです：

```
json_array, json_array_append, json_array_insert, json_contains, json_contains_path, json_depth, json_extract, json_insert, json_keys, json_length, json_merge_patch, json_merge_preserve, json_object, json_pretty, json_quote, json_remove, json_replace, json_search, json_set, json_storage_size, json_type, json_unquote, json_valid, lower, md5, reverse, tidb_shard, upper, vitess_hash
```

上記のリストに含まれていない関数については、それらの関数が完全にテストされていないため、本番環境では推奨されず、実験的と見なされます。演算子、`cast`、`case when`などのその他の式も実験的であり、本番環境では推奨されません。

<CustomContent platform="tidb">

それでもこれらの式を使用したい場合は、[TiDB設定ファイル](/tidb-configuration-file.md#allow-expression-index-new-in-v400)で次の構成を行うことができます。

{{< copyable "sql" >}}

```sql
allow-expression-index = true
```

</CustomContent>

> **注意：**
>
> 式インデックスは主キーに作成できません。
>
> 式インデックスの式には、次のコンテンツを含めることができません：
>
> - `rand()`や`now()`などの不安定な関数。
> - システム変数およびユーザ変数。
> - サブクエリ。
> - `AUTO_INCREMENT`列。`tidb_enable_auto_increment_in_generated`（システム変数）の値を`true`に設定することで、この制限を解除できます。
> - ウィンドウ関数。
> - `create table t (j json, key k (((j,j))));`のようなROW関数。
> - 集約関数。
>
> 式インデックスは、暗黙的に名前を占有します（たとえば、`_V$_{index_name}_{index_offset}`）。既存の列と同じ名前で新しい式インデックスを作成しようとすると、エラーが発生します。また、同じ名前の新しい列を追加しようとすると、同様にエラーが発生します。
>
> 式インデックスの式の関数パラメータの数が正しいことを確認してください。
>
> インデックスの式に文字列関連の関数が含まれている場合、返されるタイプと長さに影響を受け、式インデックスの作成に失敗する場合があります。この状況では、返されるタイプと長さを明示的に指定するために、`cast()`関数を使用できます。たとえば、`repeat(a, 3)`式に基づいて式インデックスを作成するには、この式を`cast(repeat(a, 3) as char(20))`に変更する必要があります。

クエリステートメントの式に式インデックスの式が一致する場合、オプティマイザはクエリに対して式インデックスを選択できます。一部の場合、オプティマイザは統計情報に依存して式インデックスを選択しないことがあります。この状況では、オプティマイザヒントを使用して、オプティマイザに式インデックスを選択させることができます。

次の例では、`lower(col1)`に基づいて式インデックス`idx`を作成したと仮定します：

クエリステートメントの結果が同じ式である場合、式インデックスが適用されます。次のステートメントを例に取ります：

{{< copyable "sql" >}}

```sql
SELECT lower(col1) FROM t;
```

フィルタリング条件に同じ式が含まれている場合、式インデックスが適用されます。次のステートメントを例に取ります：

{{< copyable "sql" >}}

```sql```
```
SELECT * FROM t WHERE lower(col1) = "a";
SELECT * FROM t WHERE lower(col1) > "a";
SELECT * FROM t WHERE lower(col1) BETWEEN "a" AND "b";
SELECT * FROM t WHERE lower(col1) in ("a", "b");
SELECT * FROM t WHERE lower(col1) > "a" AND lower(col1) < "b";
SELECT * FROM t WHERE lower(col1) > "b" OR lower(col1) < "a";
```

クエリを同じ式でソートすると、式のインデックスが適用されます。次のステートメントを例にとると：

{{< copyable "sql" >}}

```sql
SELECT * FROM t ORDER BY lower(col1);
```

同じ式が集約関数 (`GROUP BY`) に含まれる場合も、式のインデックスが適用されます。次のステートメントを例にとると：

{{< copyable "sql" >}}

```sql
SELECT max(lower(col1)) FROM t;
SELECT min(col1) FROM t GROUP BY lower(col1);
```

式のインデックスに対応する式を確認するには、`show index` を実行するか、システムテーブル `information_schema.tidb_indexes` およびテーブル `information_schema.STATISTICS` を確認します。出力の `Expression` 列には、対応する式が示されます。非式インデックスの場合、列には `NULL` が表示されます。

式インデックスの維持コストは他のインデックスの維持コストよりも高いため、行が挿入されるたびに式の値を計算する必要があります。式の値は既にインデックスに格納されているため、オプティマイザが式のインデックスを選択する際には、この値を再計算する必要はありません。

したがって、クエリのパフォーマンスが挿入および更新のパフォーマンスを上回る場合は、式にインデックスを付けることを検討できます。

式インデックスは、MySQL での同じ構文と制限を持っています。これらは不可視の生成仮想列にインデックスを作成して実装されるため、サポートされている式はすべて[生成された仮想列の制限](/generated-columns.md#limitations)を引き継ぎます。

## 複数値のインデックス

複数値のインデックスは、配列列に定義された二次インデックスの一種です。通常のインデックスでは、1 つのインデックスレコードが 1 つのデータレコードに対応します (1:1)。複数値のインデックスでは、複数のインデックスレコードが 1 つのデータレコードに対応します (N:1)。複数値のインデックスは、JSON 配列にインデックスを付けるために使用されます。たとえば、`zipcode` フィールドに定義された複数値のインデックスは、`zipcode` 配列内の各要素に対して 1 つのインデックスレコードを生成します。

```json
{
    "user":"Bob",
    "user_id":31,
    "zipcode":[94477,94536]
}
```

### 複数値のインデックスの作成

`CAST(... AS ... ARRAY)` 式を使用して、式インデックスの定義と同様に複数値のインデックスを作成できます。

```sql
mysql> CREATE TABLE customers (
    id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name CHAR(10),
    custinfo JSON,
    INDEX zips((CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)))
);
```

複数値のインデックスを一意インデックスとして定義できます。

```sql
mysql> CREATE TABLE customers (
    id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name CHAR(10),
    custinfo JSON,
    UNIQUE INDEX zips( (CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)))
);
```

複数値のインデックスが一意インデックスとして定義されている場合は、重複するデータを挿入しようとするとエラーが発生します。

```sql
mysql> INSERT INTO customers VALUES (1, 'pingcap', '{"zipcode": [1,2]}');
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO customers VALUES (1, 'pingcap', '{"zipcode": [2,3]}');
ERROR 1062 (23000): Duplicate entry '2' for key 'customers.zips'
```

同じレコードに重複した値を持つことができますが、異なるレコードに重複した値を持つとエラーが発生します。

```sql
-- 挿入成功
mysql> INSERT INTO t1 VALUES('[1,1,2]');
mysql> INSERT INTO t1 VALUES('[3,3,3,4,4,4]');

-- 挿入失敗
mysql> INSERT INTO t1 VALUES('[1,2]');
mysql> INSERT INTO t1 VALUES('[2,3]');
```

複数値のインデックスを複合インデックスとして定義することもできます。

```sql
mysql> CREATE TABLE customers (
    id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name CHAR(10),
    custinfo JSON,
    INDEX zips(name, (CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)))
);
```

複数値のインデックスが複合インデックスとして定義されている場合、複数値の部分はどの位置にも表示されることができますが、 1 回しか表示されません。

```sql
mysql> CREATE TABLE customers (
    id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name CHAR(10),
    custinfo JSON,
    INDEX zips(name, (CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)), (CAST(custinfo->'$.zipcode' AS UNSIGNED ARRAY)))
);
ERROR 1235 (42000): This version of TiDB doesn't yet support 'more than one multi-valued key part per index'.
```

書き込まれるデータは、複数値のインデックスで定義されたタイプと完全に一致していなければならず、データの書き込みに失敗します。

```sql
-- zipcode フィールド内のすべての要素は UNSIGNED 型でなければならない。
mysql> INSERT INTO customers VALUES (1, 'pingcap', '{"zipcode": [-1]}');
ERROR 3752 (HY000): Value is out of range for expression index 'zips' at row 1

mysql> INSERT INTO customers VALUES (1, 'pingcap', '{"zipcode": ["1"]}'); -- Incompatible with MySQL
ERROR 3903 (HY000): Invalid JSON value for CAST for expression index 'zips'

mysql> INSERT INTO customers VALUES (1, 'pingcap', '{"zipcode": [1]}');
Query OK, 1 row affected (0.00 sec)
```

### 複数値のインデックスの使用

詳細については、複数値のインデックスを使用する - [インデックスの選択](/choose-index.md#use-multi-valued-indexes) を参照してください。

### 制限

- 空の JSON 配列の場合、対応するインデックスレコードは生成されません。
- `CAST(... AS ... ARRAY)` 内のターゲット型は、`BINARY`、`JSON`、`YEAR`、`FLOAT`、`DECIMAL` のいずれでもない必要があります。ソース型は JSON でなければなりません。
- ソートに複数値のインデックスを使用することはできません。
- JSON 配列にのみ複数値のインデックスを作成できます。
- 複数値のインデックスはプライマリキーまたは外部キーにできません。
- 複数値のインデックスが使用する追加のストレージスペース = 行あたりの平均配列要素数 * 通常の二次インデックスの使用スペース
- 通常のインデックスと比較して、複数値のインデックスの DML 操作では、より多くのインデックスレコードが変更されるため、複数値のインデックスは通常のインデックスよりも大きなパフォーマンスの影響を受けます。
- 複数値のインデックスは特別なタイプの式インデックスであるため、複数値のインデックスは式インデックスと同じ制限を持ちます。
- テーブルで複数値のインデックスを使用する場合、TiDB v6.6.0 より前のTiDBクラスタに対して BR、TiCDC、または TiDB Lightning を使用してテーブルをバックアップしたり、レプリケーション、またはインポートすることはできません。
- 複数値のインデックスの収集された統計がないため、複数値のインデックスの選択率は現在固定の仮定に基づいています。クエリが複数の複数値のインデックスにヒットすると、TiDB は最適なインデックスを選択できないことがあります。そのような場合は、固定の実行プランを強制するために [`use_index_merge`](/optimizer-hints.md#use_index_merget1_name-idx1_name--idx2_name-) オプティマイザヒントを使用することをお勧めします。
- 複雑な条件でクエリを実行する場合、TiDB は複数値のインデックスを選択できないことがあります。複数値のインデックスでサポートされている条件パターンに関する詳細は、[複数値のインデックスを使用する](/choose-index.md#use-multi-valued-indexes) を参照してください。

## 不可視インデックス

不可視インデックスは、クエリオプティマイザーによって無視されるインデックスです。

```sql
CREATE TABLE t1 (c1 INT, c2 INT, UNIQUE(c2));
CREATE UNIQUE INDEX c1 ON t1 (c1) INVISIBLE;
```

詳細については、[`ALTER INDEX`](/sql-statements/sql-statement-alter-index.md) を参照してください。

## 関連するシステム変数

`CREATE INDEX` ステートメントに関連付けられたシステム変数には、`tidb_ddl_enable_fast_reorg`、`tidb_ddl_reorg_worker_cnt`、`tidb_ddl_reorg_batch_size`、`tidb_enable_auto_increment_in_generated`、および `tidb_ddl_reorg_priority` があります。詳細については[システム変数](/system-variables.md#tidb_ddl_reorg_worker_cnt) を参照してください。

## MySQL 互換性

* TiDB は `FULLTEXT` および `SPATIAL` 構文の解析をサポートしていますが、`FULLTEXT`、`HASH`、`SPATIAL` インデックスの使用はサポートされていません。
* 降順インデックスはサポートされていません（MySQL 5.7 と同様）。
* テーブルに `CLUSTERED` タイプの主キーを追加することはサポートされていません。`CLUSTERED` タイプの主キーの詳細については、[クラスター化インデックス](/clustered-indexes.md) を参照してください。
* 式インデックスはビューと互換性がありません。ビューを使用してクエリを実行する場合、同時に式インデックスを使用することはできません。
* 式インデックスにはバインディングとの互換性の問題があります。式インデックスの式に定数があると、対応するクエリのバインディングがその範囲を拡大します。例えば、式インデックスの式が `a+1` であり、対応するクエリ条件が `a+1 > 2` の場合、作成されるバインディングは `a+? > ?` となります。これは、`a+2 > 2` などの条件を持つクエリも式インデックスを強制的に使用させ、実行計画が悪化することを意味します。さらに、これはSQLプラン管理（SPM）におけるベースラインのキャプチャおよびベースラインの進化にも影響します。
* 複数値インデックスで書き込まれたデータは、定義されたデータ型と完全に一致する必要があります。そうでない場合、データの書き込みが失敗します。詳細については、[複数値インデックスの作成](/sql-statements/sql-statement-create-index.md#create-multi-valued-indexes)を参照してください。

## 関連項目

* [インデックスの選択](/choose-index.md)
* [誤ったインデックスの解決方法](/wrong-index-solution.md)
* [ADD INDEX](/sql-statements/sql-statement-add-index.md)
* [DROP INDEX](/sql-statements/sql-statement-drop-index.md)
* [RENAME INDEX](/sql-statements/sql-statement-rename-index.md)
* [ALTER INDEX](/sql-statements/sql-statement-alter-index.md)
* [ADD COLUMN](/sql-statements/sql-statement-add-column.md)
* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [EXPLAIN](/sql-statements/sql-statement-explain.md)