---
title: FOREIGN KEY 制約
summary: TiDB データベースの FOREIGN KEY 制約の使用の概要。
---

# FOREIGN KEY 制約

v6.6.0 から、TiDB は外部キー機能をサポートし、関連データのクロステーブル参照およびデータ整合性を維持するための外部キー制約を可能にします。

> **警告:**
>
> - 現在、外部キー機能は実験的な機能です。本番環境での使用は推奨されません。この機能は予告なく変更または削除される場合があります。バグを見つけた場合は、GitHub で [issue](https://github.com/pingcap/tidb/issues) を報告できます。
> - 通常、外部キー機能は小〜中規模のデータ量の整合性および一貫性制約チェックを提供するために使用されます。ただし、分散データベースシステムの大規模なデータ量の場合、外部キーの使用は重大なパフォーマンスの問題を引き起こし、システムに予測不可能な影響を与える可能性があります。外部キーを使用する場合は、まず徹底的な検証を行い、慎重に使用してください。

外部キーは子テーブルで定義されます。構文は次のようになります：

```ebnf+diagram
ForeignKeyDef
         ::= ( 'CONSTRAINT' Identifier )? 'FOREIGN' 'KEY'
             Identifier? '(' ColumnName ( ',' ColumnName )* ')'
             'REFERENCES' TableName '(' ColumnName ( ',' ColumnName )* ')'
             ( 'ON' 'DELETE' ReferenceOption )?
             ( 'ON' 'UPDATE' ReferenceOption )?

ReferenceOption
         ::= 'RESTRICT'
           | 'CASCADE'
           | 'SET' 'NULL'
           | 'SET' 'DEFAULT'
           | 'NO' 'ACTION'
```

## 命名

外部キーの命名は次の規則に従います：

- `CONSTRAINT identifier` で名前が指定されている場合、指定された名前が使用されます。
- `CONSTRAINT identifier` に名前が指定されていないが、`FOREIGN KEY identifier` で名前が指定されている場合、`FOREIGN KEY identifier` で指定された名前が使用されます。
- `CONSTRAINT identifier` と `FOREIGN KEY identifier` のいずれも名前が指定されていない場合、`fk_1`、`fk_2`、`fk_3` などが自動的に生成されます。
- 外部キー名は現在のテーブルで一意でなければなりません。それ以外の場合、外部キーが作成されたときに`ERROR 1826: Duplicate foreign key constraint name 'fk'` というエラーが報告されます。

## 制限

外部キーを作成する際は、次の条件を満たす必要があります：

- 親テーブルも子テーブルも一時テーブルではありません。
- ユーザーは親テーブルに対する `REFERENCES` 権限を持っています。
- 外部キーによって参照される親テーブルと子テーブルのカラムは、データ型が同じであり、サイズ、精度、長さ、文字セット、照合規則も同じでなければなりません。
- 外部キーのカラムは、それ自体を参照してはいけません。
- 外部キーのカラムと参照される親テーブルのカラムは同じインデックスを持ち、そのインデックス内のカラムの順序が外部キー内の順序と一致している必要があります。これにより、外部キー制約チェック時の完全なテーブルスキャンを回避するためにインデックスを使用できます。

    - 親テーブルに対応する外部キーインデックスが存在しない場合、`ERROR 1822: Failed to add the foreign key constraint. Missing index for constraint 'fk' in the referenced table 't'` というエラーが報告されます。
    - 子テーブルに対応する外部キーインデックスが存在しない場合、外部キーと同じ名前のインデックスが自動的に作成されます。

- `BLOB` や `TEXT` 型のカラムに外部キーを作成することはサポートされていません。
- パーティションテーブルに外部キーを作成することはサポートされていません。
- 仮想生成カラムに外部キーを作成することはサポートされていません。

## 参照操作

`UPDATE` または `DELETE` 操作が親テーブルの外部キーを影響する場合、外部キーの定義で `ON UPDATE` または `ON DELETE` 句で定義された参照操作によって、子テーブルの対応する外部キー値が決まります。参照操作には次のものがあります：

- `CASCADE`: 親テーブルの `UPDATE` または `DELETE` 操作が子テーブルの一致する行に影響を与えると、対応する行を自動的に更新または削除します。カスケード操作は深さ優先で実行されます。
- `SET NULL`: 親テーブルの `UPDATE` または `DELETE` 操作が子テーブルの一致する外部キーカラムを `NULL` に自動的に設定します。
- `RESTRICT`: 子テーブルに一致する行が含まれている場合、`UPDATE` または `DELETE` 操作を拒否します。
- `NO ACTION`: `RESTRICT` と同じです。
- `SET DEFAULT`: `RESTRICT` と同じです。

親テーブルに対応する外部キー値が存在しない場合、子テーブルの `INSERT` または `UPDATE` 操作は拒否されます。

外部キーの定義で `ON DELETE` または `ON UPDATE` を指定しない場合、デフォルトの動作は `NO ACTION` です。

外部キーが `STORED GENERATED COLUMN` に定義されている場合、`CASCADE`、`SET NULL`、`SET DEFAULT` の参照はサポートされません。

## 外部キーの使用例

次の例では、単一列の外部キーを使用して親テーブルと子テーブルを関連付けます：

```sql
CREATE TABLE parent (
    id INT KEY
);

CREATE TABLE child (
    id INT,
    pid INT,
    INDEX idx_pid (pid),
    FOREIGN KEY (pid) REFERENCES parent(id) ON DELETE CASCADE
);
```

次の例は、`product_order` テーブルが他の2つのテーブルを参照する2つの外部キーを持つより複雑な例です。 1 つの外部キーは `product` テーブルの 2 つのインデックスを参照し、もう 1 つの外部キーは `customer` テーブルの単一のインデックスを参照しています：

```sql
CREATE TABLE product (
    category INT NOT NULL,
    id INT NOT NULL,
    price DECIMAL(20,10),
    PRIMARY KEY(category, id)
);

CREATE TABLE customer (
    id INT KEY
);

CREATE TABLE product_order (
    id INT NOT NULL AUTO_INCREMENT,
    product_category INT NOT NULL,
    product_id INT NOT NULL,
    customer_id INT NOT NULL,

    PRIMARY KEY(id),
    INDEX (product_category, product_id),
    INDEX (customer_id),

    FOREIGN KEY (product_category, product_id)
      REFERENCES product(category, id)
      ON UPDATE CASCADE ON DELETE RESTRICT,

    FOREIGN KEY (customer_id)
      REFERENCES customer(id)
);
```

## 外部キー制約の作成

外部キー制約を作成するには、次の `ALTER TABLE` ステートメントを使用できます：

```sql
ALTER TABLE table_name
    ADD [CONSTRAINT [identifier]] FOREIGN KEY
    [identifier] (col_name, ...)
    REFERENCES tbl_name (col_name,...)
    [ON DELETE reference_option]
    [ON UPDATE reference_option]
```

外部キーは自己参照、つまり同じテーブルを参照できます。`ALTER TABLE` を使用してテーブルに外部キー制約を追加する場合は、最初に外部キーが参照する親テーブルのカラムにインデックスを作成する必要があります。

## 外部キー制約の削除

外部キー制約を削除するには、次の `ALTER TABLE` ステートメントを使用できます：

```sql
ALTER TABLE table_name DROP FOREIGN KEY fk_identifier;
```

外部キー制約の作成時に名前が付けられている場合は、名前を参照して外部キー制約を削除できます。それ以外の場合は、自動的に生成された制約名を使用して制約を削除する必要があります。外部キー名を表示するには、 `SHOW CREATE TABLE` を使用できます：

```sql
mysql> SHOW CREATE TABLE child\G
*************************** 1. row ***************************
       Table: child
Create Table: CREATE TABLE `child` (
  `id` int(11) DEFAULT NULL,
  `pid` int(11) DEFAULT NULL,
  KEY `idx_pid` (`pid`),
  CONSTRAINT `fk_1` FOREIGN KEY (`pid`) REFERENCES `test`.`parent` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```

## 外部キー制約のチェック

TiDB は外部キー制約チェックをサポートしており、これはシステム変数 [`foreign_key_checks`](/system-variables.md#foreign_key_checks) で制御されます。この変数はデフォルトで `ON` に設定されており、つまり外部キー制約チェックが有効になっています。この変数には `GLOBAL` および `SESSION` の 2 つのスコープがあります。この変数を有効にしておくことで、外部キー参照関係の整合性を確保できます。

外部キー制約チェックを無効にすると次のような影響があります：

- 外部キーで参照されている親テーブルを削除する場合、外部キー制約チェックが無効になっている場合のみ削除が成功します。
- データをデータベースにインポートする際、テーブルの作成の順序が外部キーの依存順序と異なる場合、テーブルの作成に失敗する可能性があります。外部キー制約チェックが無効になっている場合のみ、テーブルを正常に作成できます。また、外部キー制約チェックを無効にすることで、データのインポートを高速化できます。
- データをデータベースにインポートする際、子テーブルのデータが最初にインポートされると、エラーが報告されます。外部キー制約チェックが無効になっている場合のみ、子テーブルのデータを正常にインポートできます。
- 実行する `ALTER TABLE` 操作に外部キーの変更が含まれる場合、外部キー制約チェックが無効になっている場合のみ、この操作が成功します。

外部キー制約チェックが無効になっている場合、外部キー制約チェックと参照操作は実行されません。ただし、次のシナリオでは、`ALTER TABLE` の実行が外部キーの定義を間違ったものにする可能性がある場合には、実行中にエラーが報告されます。
- 外部キーに必要なインデックスを削除する場合、外部キーを最初に削除する必要があります。それ以外の場合、エラーが報告されます。
- 関連する条件や制約を満たさない状態で外部キーを作成すると、エラーが報告されます。

## ロック

子テーブルに`INSERT`または`UPDATE`を行う際、外部キー制約は親テーブルで対応する外部キー値が存在するかをチェックし、他の操作によって外部キー制約が違反されることなく、親テーブルの行をロックします。このロックの挙動は、親テーブル内の外部キー値が存在する行に対して`SELECT FOR UPDATE`操作を実行することと同等です。

TiDBは現在`LOCK IN SHARE MODE`をサポートしていないため、子テーブルが多数の同時書き込みを受け入れ、参照される外部キー値の大部分が同一である場合、重大なロックの競合が発生する可能性があります。子テーブルデータを大量に書き込む場合は、[`foreign_key_checks`](/system-variables.md#foreign_key_checks)を無効にすることを推奨します。

## 外部キーの定義とメタデータ

外部キー制約の定義を表示するには、[`SHOW CREATE TABLE`](/sql-statements/sql-statement-show-create-table.md)ステートメントを実行します：

```sql
mysql> SHOW CREATE TABLE child\G
*************************** 1. row ***************************
       Table: child
Create Table: CREATE TABLE `child` (
  `id` int(11) DEFAULT NULL,
  `pid` int(11) DEFAULT NULL,
  KEY `idx_pid` (`pid`),
  CONSTRAINT `fk_1` FOREIGN KEY (`pid`) REFERENCES `test`.`parent` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```

また、次のシステムテーブルを使用して外部キーに関する情報を取得することもできます：

- [`INFORMATION_SCHEMA.KEY_COLUMN_USAGE`](/information-schema/information-schema-key-column-usage.md)
- [`INFORMATION_SCHEMA.TABLE_CONSTRAINTS`](/information-schema/information-schema-table-constraints.md)
- [`INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS`](/information-schema/information-schema-referential-constraints.md)

以下は例です：

`INFORMATION_SCHEMA.KEY_COLUMN_USAGE`システムテーブルから外部キーに関する情報を取得する：

```sql
mysql> SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, CONSTRAINT_NAME FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE WHERE REFERENCED_TABLE_SCHEMA IS NOT NULL;
+--------------+---------------+------------------+-----------------+
| TABLE_SCHEMA | TABLE_NAME    | COLUMN_NAME      | CONSTRAINT_NAME |
+--------------+---------------+------------------+-----------------+
| test         | child         | pid              | fk_1            |
| test         | product_order | product_category | fk_1            |
| test         | product_order | product_id       | fk_1            |
| test         | product_order | customer_id      | fk_2            |
+--------------+---------------+------------------+-----------------+
```

`INFORMATION_SCHEMA.TABLE_CONSTRAINTS`システムテーブルから外部キーに関する情報を取得する：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS WHERE CONSTRAINT_TYPE='FOREIGN KEY'\G
***************************[ 1. row ]***************************
CONSTRAINT_CATALOG | def
CONSTRAINT_SCHEMA  | test
CONSTRAINT_NAME    | fk_1
TABLE_SCHEMA       | test
TABLE_NAME         | child
CONSTRAINT_TYPE    | FOREIGN KEY
```

`INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS`システムテーブルから外部キーに関する情報を取得する：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS\G
***************************[ 1. row ]***************************
CONSTRAINT_CATALOG        | def
CONSTRAINT_SCHEMA         | test
CONSTRAINT_NAME           | fk_1
UNIQUE_CONSTRAINT_CATALOG | def
UNIQUE_CONSTRAINT_SCHEMA  | test
UNIQUE_CONSTRAINT_NAME    | PRIMARY
MATCH_OPTION              | NONE
UPDATE_RULE               | NO ACTION
DELETE_RULE               | CASCADE
TABLE_NAME                | child
REFERENCED_TABLE_NAME     | parent
```

## 外部キーを使用した実行計画の表示

`EXPLAIN`ステートメントを使用して実行計画を表示できます。`Foreign_Key_Check`演算子は実行されるDMLステートメントの外部キー制約チェックを行います。

```sql
mysql> explain insert into child values (1,1);
+-----------------------+---------+------+---------------+-------------------------------+
| id                    | estRows | task | access object | operator info                 |
+-----------------------+---------+------+---------------+-------------------------------+
| Insert_1              | N/A     | root |               | N/A                           |
| └─Foreign_Key_Check_3 | 0.00    | root | table:parent  | foreign_key:fk_1, check_exist |
+-----------------------+---------+------+---------------+-------------------------------+
```

`EXPLAIN ANALYZE`ステートメントを使用して外部キー参照の実行を表示できます。`Foreign_Key_Cascade`演算子は実行されるDMLステートメントの外部キー参照を行います。

```sql
mysql> explain analyze delete from parent where id = 1;
+----------------------------------+---------+---------+-----------+---------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------+-----------+------+
| id                               | estRows | actRows | task      | access object                   | execution info                                                                                                                                                                               | operator info                               | memory    | disk |
+----------------------------------+---------+---------+-----------+---------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------+-----------+------+
| Delete_2                         | N/A     | 0       | root      |                                 | time:117.3µs, loops:1                                                                                                                                                                        | N/A                                         | 380 Bytes | N/A  |
| ├─Point_Get_1                    | 1.00    | 1       | root      | table:parent                    | time:63.6µs, loops:2, Get:{num_rpc:1, total_time:29.9µs}                                                                                                                                     | handle:1                                    | N/A       | N/A  |
| └─Foreign_Key_Cascade_3          | 0.00    | 0       | root      | table:child, index:idx_pid      | total:1.28ms, foreign_keys:1                                                                                                                                                                 | foreign_key:fk_1, on_delete:CASCADE         | N/A       | N/A  |
|   └─Delete_7                     | N/A     | 0       | root      |                                 | time:904.8µs, loops:1                                                                                                                                                                        | N/A                                         | 1.11 KB   | N/A  |
|     └─IndexLookUp_11             | 10.00   | 1       | root      |                                 | time:869.5µs, loops:2, index_task: {total_time: 371.1µs, fetch_handle: 357.3µs, build: 1.25µs, wait: 12.5µs}, table_task: {total_time: 382.6µs, num: 1, concurrency: 5}                      |                                             | 9.13 KB   | N/A  |
|       ├─IndexRangeScan_9(Build)  | 10.00   | 1       | cop[tikv] | table:child, index:idx_pid(pid) | time:351.2µs, loops:3, cop_task: {num: 1, max: 282.3µs, proc_keys: 0, rpc_num: 1, rpc_time: 263µs, copr_cache_hit_ratio: 0.00, distsql_concurrency: 15}, tikv_task:{time:220.2µs, loops:0}   | range:[1,1], keep order:false, stats:pseudo | N/A       | N/A  |
|       └─TableRowIDScan_10(Probe) | 10.00   | 1       | cop[tikv] | table:child                     | time:223.9µs, loops:2, cop_task: {num: 1, max: 168.8µs, proc_keys: 0, rpc_num: 1, rpc_time: 154.5µs, copr_cache_hit_ratio: 0.00, distsql_concurrency: 15}, tikv_task:{time:145.6µs, loops:0} | keep order:false, stats:pseudo              | N/A       | N/A  |
+----------------------------------+---------+---------+-----------+---------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------+-----------+------+
```

## 互換性

### TiDBバージョン間の互換性

v6.6.0以前のバージョンでは、TiDBは外部キーを作成する構文をサポートしていますが、作成された外部キーは無効です。v6.6.0以前に作成されたTiDBクラスタをv6.6.0以降にアップグレードすると、アップグレード前に作成された外部キーは引き続き無効です。v6.6.0以降のバージョンで作成された外部キーのみが有効です。無効な外部キーを削除し、新しい外部キーを作成して外部キー制約を有効にすることができます。外部キーが有効かどうかを確認するには、`SHOW CREATE TABLE`ステートメントを使用します。無効な外部キーには`/* FOREIGN KEY INVALID */`というコメントが付いています。

```sql
mysql> SHOW CREATE TABLE child\G
***************************[ 1. row ]***************************
Table        | child
Create Table | CREATE TABLE `child` (
  `id` int(11) DEFAULT NULL,
  `pid` int(11) DEFAULT NULL,
  KEY `idx_pid` (`pid`),
```
CONSTRAINT `fk_1` FOREIGN KEY (`pid`) REFERENCES `test`.`parent` (`id`) ON DELETE CASCADE /* FOREIGN KEY INVALID */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin

### TiDBツールとの互換性

<CustomContent platform="tidb">

- [TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md)は外部キーをサポートしていません。
- [DM](/dm/dm-overview.md)は外部キーをサポートしていません。DM v6.6.0では、TiDBへのデータレプリケーション時にダウンストリームTiDBの[`foreign_key_checks`](/system-variables.md#foreign_key_checks)が無効になります。したがって、外部キーによる連鎖的操作は、上流から下流へレプリケーションされず、データの不整合を引き起こす可能性があります。この動作は以前のDMバージョンと一致しています。
- [TiCDC](/ticdc/ticdc-overview.md) v6.6.0は外部キーと互換性があります。以前のバージョンのTiCDCは、外部キーを持つテーブルのレプリケーション時にエラーが発生することがあります。TiCDCのv6.6.0より前のバージョンを使用する場合は、データベースの下流TiDBクラスタの`foreign_key_checks`を無効にすることをお勧めします。
- [BR](/br/backup-and-restore-overview.md) v6.6.0は外部キーと互換性があります。以前のBRバージョンは、外部キーを持つテーブルをv6.6.0以降のクラスタに復元する際にエラーが発生する場合があります。BRをv6.6.0よりも前のバージョンを使用する場合は、クラスタを復元する前に下流TiDBクラスタの`foreign_key_checks`を無効にすることをお勧めします。
- [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)を使用する場合は、データをインポートする前に、下流TiDBクラスタの`foreign_key_checks`を無効にすることをお勧めします。

</CustomContent>

- [Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview)は外部キーと互換性があります。

<CustomContent platform="tidb">

- [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md)を使用して上流と下流のデータを比較する場合、データベースのバージョンが異なる場合や下流TiDBに[無効な外部キー](#compatibility-between-tidb-versions)がある場合、sync-diff-inspectorはテーブルスキーマの不一致エラーを報告することがあります。これはTiDB v6.6.0が無効な外部キーに`/* FOREIGN KEY INVALID */`コメントを追加するためです。

</CustomContent>

### MySQLとの互換性

外部キーを名前を指定せずに作成すると、TiDBで生成される名前はMySQLで生成される名前とは異なります。例えば、TiDBで生成される外部キーの名前は`fk_1`、 `fk_2`、 `fk_3` ですが、MySQLで生成される外部キーの名前は `table_name_ibfk_1`、 `table_name_ibfk_2`、 `table_name_ibfk_3`です。

MySQLとTiDBの両方が、「inline `REFERENCES` specifications」を解析しますが無視します。`FOREIGN KEY`の定義の一部である `REFERENCES` の仕様のみがチェックおよび強制されます。次の例は、`REFERENCES`句を使用して外部キー制約を作成します：

```sql
CREATE TABLE parent (
    id INT PRIMARY KEY
);

CREATE TABLE child (
    id INT,
    pid INT,
    FOREIGN KEY (pid) REFERENCES parent(id)
);

SHOW CREATE TABLE child;
```

出力では、`child`テーブルに外部キーが含まれていないことが示されます：

```sql
+-------+-------------------------------------------------------------+
| Table | Create Table                                                |
+-------+-------------------------------------------------------------+
| child | CREATE TABLE `child` (                                      |
|       |   `id` int(11) DEFAULT NULL,                                |
|       |   `pid` int(11) DEFAULT NULL,                               |
|       |   FOREIGN KEY (pid) REFERENCES parent(id)                    |
|       | ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin |
+-------+-------------------------------------------------------------+
```