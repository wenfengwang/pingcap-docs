---
title: CREATE TABLE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのCREATE TABLEの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-create-table/','/docs/dev/reference/sql/statements/create-table/']
---

# CREATE TABLE

このステートメントは現在選択されているデータベースに新しいテーブルを作成します。これはMySQLの`CREATE TABLE`ステートメントと同様の動作をします。

## 概要

```ebnf+diagram
CreateTableStmt ::=
    'CREATE' OptTemporary 'TABLE' IfNotExists TableName ( TableElementListOpt CreateTableOptionListOpt PartitionOpt DuplicateOpt AsOpt CreateTableSelectOpt | LikeTableWithOrWithoutParen ) OnCommitOpt

OptTemporary ::=
    ( 'TEMPORARY' | ('GLOBAL' 'TEMPORARY') )?

IfNotExists ::=
    ('IF' 'NOT' 'EXISTS')?

TableName ::=
    Identifier ('.' Identifier)?

TableElementListOpt ::=
    ( '(' TableElementList ')' )?

TableElementList ::=
    TableElement ( ',' TableElement )*

TableElement ::=
    ColumnDef
|   Constraint

ColumnDef ::=
    ColumnName ( Type | 'SERIAL' ) ColumnOptionListOpt

ColumnOptionListOpt ::=
    ColumnOption*

ColumnOptionList ::=
    ColumnOption*

ColumnOption ::=
    'NOT'? 'NULL'
|   'AUTO_INCREMENT'
|   PrimaryOpt 'KEY'
|   'UNIQUE' 'KEY'?
|   'DEFAULT' DefaultValueExpr
|   'SERIAL' 'DEFAULT' 'VALUE'
|   'ON' 'UPDATE' NowSymOptionFraction
|   'COMMENT' stringLit
|   ConstraintKeywordOpt 'CHECK' '(' Expression ')' EnforcedOrNotOrNotNullOpt
|   GeneratedAlways 'AS' '(' Expression ')' VirtualOrStored
|   ReferDef
|   'COLLATE' CollationName
|   'COLUMN_FORMAT' ColumnFormat
|   'STORAGE' StorageMedia
|   'AUTO_RANDOM' OptFieldLen

Constraint ::=
    IndexDef
|   ForeignKeyDef

IndexDef ::=
    ( 'INDEX' | 'KEY' ) IndexName? '(' KeyPartList ')' IndexOption?

KeyPartList ::=
    KeyPart ( ',' KeyPart )*

KeyPart ::=
    ColumnName ( '(' Length ')')? ( 'ASC' | 'DESC' )?
|   '(' Expression ')' ( 'ASC' | 'DESC' )?

IndexOption ::=
    'COMMENT' String
|   ( 'VISIBLE' | 'INVISIBLE' )

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

CreateTableOptionListOpt ::=
    TableOptionList?

PartitionOpt ::=
    ( 'PARTITION' 'BY' PartitionMethod PartitionNumOpt SubPartitionOpt PartitionDefinitionListOpt )?

DuplicateOpt ::=
    ( 'IGNORE' | 'REPLACE' )?

TableOptionList ::=
    TableOption ( ','? TableOption )*

TableOption ::=
    PartDefOption
|   DefaultKwdOpt ( CharsetKw EqOpt CharsetName | 'COLLATE' EqOpt CollationName )
|   ( 'AUTO_INCREMENT' | 'AUTO_ID_CACHE' | 'AUTO_RANDOM_BASE' | 'AVG_ROW_LENGTH' | 'CHECKSUM' | 'TABLE_CHECKSUM' | 'KEY_BLOCK_SIZE' | 'DELAY_KEY_WRITE' | 'SHARD_ROW_ID_BITS' | 'PRE_SPLIT_REGIONS' ) EqOpt LengthNum
|   ( 'CONNECTION' | 'PASSWORD' | 'COMPRESSION' ) EqOpt stringLit
|   RowFormat
|   ( 'STATS_PERSISTENT' | 'PACK_KEYS' ) EqOpt StatsPersistentVal
|   ( 'STATS_AUTO_RECALC' | 'STATS_SAMPLE_PAGES' ) EqOpt ( LengthNum | 'DEFAULT' )
|   'STORAGE' ( 'MEMORY' | 'DISK' )
|   'SECONDARY_ENGINE' EqOpt ( 'NULL' | StringName )
|   'UNION' EqOpt '(' TableNameListOpt ')'
|   'ENCRYPTION' EqOpt EncryptionOpt
|    'TTL' EqOpt TimeColumnName '+' 'INTERVAL' Expression TimeUnit (TTLEnable EqOpt ( 'ON' | 'OFF' ))? (TTLJobInterval EqOpt stringLit)?
|   PlacementPolicyOption

OnCommitOpt ::=
    ('ON' 'COMMIT' 'DELETE' 'ROWS')?

PlacementPolicyOption ::=
    "PLACEMENT" "POLICY" EqOpt PolicyName
|   "PLACEMENT" "POLICY" (EqOpt | "SET") "DEFAULT"
```

以下の*table_options*はサポートされています。`AVG_ROW_LENGTH`、`CHECKSUM`、`COMPRESSION`、`CONNECTION`、`DELAY_KEY_WRITE`、`ENGINE`、`KEY_BLOCK_SIZE`、`MAX_ROWS`、`MIN_ROWS`、`ROW_FORMAT`、`STATS_PERSISTENT`などのその他のオプションは解析されますが無視されます。

| オプション | 説明 | 例 |
| ---------- | ---------- | ------- |
| `AUTO_INCREMENT` | インクリメントフィールドの初期値 | `AUTO_INCREMENT` = 5 |
| [`SHARD_ROW_ID_BITS`](/shard-row-id-bits.md)| 暗黙の`_tidb_rowid`のシャードのビット数を設定する |`SHARD_ROW_ID_BITS` = 4|
|`PRE_SPLIT_REGIONS`| テーブルを作成する際に`2^(PRE_SPLIT_REGIONS)`リージョンを事前に分割する |`PRE_SPLIT_REGIONS` = 4|
|`AUTO_ID_CACHE`| TiDBインスタンスの自動IDキャッシュサイズを設定する。デフォルトでは、TiDBは自動IDの割り当て速度に応じてこのサイズを自動的に変更します |`AUTO_ID_CACHE` = 200。このオプションは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。|
|`AUTO_RANDOM_BASE`| auto_randomの初期増分部分の値を設定する。このオプションは内部インタフェースの一部と見なすことができます。ユーザーはこのパラメータを無視できます |`AUTO_RANDOM_BASE` = 0|
| `CHARACTER SET` | テーブルの[文字セット](/character-set-and-collation.md)を指定する | `CHARACTER SET` =  'utf8mb4' |
| `COMMENT` | コメント情報 | `COMMENT` = 'コメント情報' |

<CustomContent platform="tidb">

> **注意:**
>
> `split-table`構成オプションはデフォルトで有効になっています。有効になっていると、新しく作成されたテーブルごとに別々のリージョンが作成されます。詳細は、[TiDB構成ファイル](/tidb-configuration-file.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> TiDBでは、新しく作成されたテーブルごとに別々のリージョンが作成されます。

</CustomContent>

## 例

単純なテーブルを作成し、1行を挿入する例:

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (a int);
DESC t1;
SHOW CREATE TABLE t1\G
INSERT INTO t1 (a) VALUES (1);
SELECT * FROM t1;
```

```
mysql> drop table if exists t1;
クエリが実行されました: 0件の行が影響を受けました (0.23秒)

mysql> CREATE TABLE t1 (a int);
クエリが実行されました: 0件の行が影響を受けました (0.09秒)

mysql> DESC t1;
+-------+---------+------+------+---------+-------+
| Field | Type    | Null | Key  | Default | Extra |
+-------+---------+------+------+---------+-------+
| a     | int(11) | YES  |      | NULL    |       |
+-------+---------+------+------+---------+-------+
1行が表示されました (0.00秒)

mysql> SHOW CREATE TABLE t1\G
*************************** 1. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `a` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
1行が表示されました (0.00秒)

mysql> INSERT INTO t1 (a) VALUES (1);
クエリが実行されました: 1件の行が影響を受けました (0.03秒)

mysql> SELECT * FROM t1;
+------+
| a    |
+------+
|    1 |
+------+
1行が表示されました (0.00秒)
```

存在する場合はテーブルを削除し、存在しない場合にテーブルを条件付きで作成する例:

{{< copyable "sql" >}}

```sql
DROP TABLE IF EXISTS t1;
CREATE TABLE IF NOT EXISTS t1 (
 id BIGINT NOT NULL PRIMARY KEY auto_increment,
 b VARCHAR(200) NOT NULL
);
DESC t1;
```

```sql
mysql> DROP TABLE IF EXISTS t1;
クエリが実行されました: 0件の行が影響を受けました (0.22秒)

mysql> CREATE TABLE IF NOT EXISTS t1 (
         id BIGINT NOT NULL PRIMARY KEY auto_increment,
         b VARCHAR(200) NOT NULL
        );
```
```mysql
      +-------+--------------+------+------+---------+----------------+ {T}
      | Field | Type         | Null | Key  | Default | Extra          | {T}
      +-------+--------------+------+------+---------+----------------+ {T}
      | id    | bigint(20)   | NO   | PRI  | NULL    | auto_increment | {T}
      | b     | varchar(200) | NO   |      | NULL    |                | {T}
      +-------+--------------+------+------+---------+----------------+ {T}
2 rows in set (0.00 sec) {T}

## MySQLの互換性 {T}

* 空間型を除くすべてのデータ型がサポートされています。 {T}
* `FULLTEXT`、`HASH`、`SPATIAL` インデックスはサポートされていません。 {T}

<CustomContent platform="tidb">

* 互換性のため、デフォルトで最大長制限が 3072 バイトの `index_col_name` 属性は、長さのオプションをサポートします。長さの制限は `max-index-length` 構成オプションで変更できます。詳細については、「[TiDB 構成ファイル](/tidb-configuration-file.md#max-index-length)」を参照してください。 {T}

</CustomContent>

<CustomContent platform="tidb-cloud">

* 互換性のため、デフォルトで最大長制限が 3072 バイトの `index_col_name` 属性は、長さのオプションをサポートします。 {T}

</CustomContent>

* `index_col_name` 内の `[ASC | DESC]` は現在パースされていますが、無視されます（MySQL 5.7 互換の動作）。 {T}
* `COMMENT` 属性は `WITH PARSER` オプションをサポートしていません。 {T}
* TiDB はデフォルトで単一テーブルに 1017 列をサポートし、最大で 4096 列をサポートします。InnoDB の対応する列数制限は 1017 列であり、MySQL のハード制限は 4096 列です。詳細については、「[TiDB 制限事項](/tidb-limitations.md)」を参照してください。 {T}
* パーティションテーブルについては、Range、Hash、Range Columns（単一列）のみがサポートされています。詳細については、「[パーティションテーブル](/partitioned-table.md)」を参照してください。 {T}

## 関連項目 {T}

* [データ型](/data-type-overview.md) {T}
* [DROP TABLE](/sql-statements/sql-statement-drop-table.md) {T}
* [CREATE TABLE LIKE](/sql-statements/sql-statement-create-table-like.md) {T}
* [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md) {T}
```