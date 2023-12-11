---
title: クラスター化インデックス
summary: クラスター化インデックスの概念、ユーザシナリオ、使用法、制限事項、および互換性について学びます。

# クラスター化インデックス

TiDBはv5.0からクラスター化インデックス機能をサポートしています。この機能は、プライマリキーを含むテーブルのデータの格納方法を制御します。これにより、特定のクエリのパフォーマンスを向上させる方法でテーブルを組織化することができます。

この文脈における「クラスター化」という用語は、「データの格納方法の組織化」を意味し、「データベースサーバーのグループ化」を意味しません。いくつかのデータベース管理システムはクラスター化インデックスを「インデックスによって組織化されたテーブル」（IOT）として参照します。

現在、TiDBのプライマリキーを含むテーブルは、次の2つのカテゴリに分けることができます：

- `NONCLUSTERED`: テーブルのプライマリキーはノンクラスター化インデックスです。ノンクラスター化インデックスのあるテーブルでは、行データのキーはTiDBによって暗黙的に割り当てられた内部の `_tidb_rowid` で構成されます。プライマリキーは基本的に一意のインデックスであるため、ノンクラスター化インデックスのあるテーブルには行を格納するために少なくとも2つのキー値ペアが必要です：
    - `_tidb_rowid`（キー） - 行データ（値）
    - プライマリキーデータ（キー） - `_tidb_rowid`（値）
- `CLUSTERED`: テーブルのプライマリキーはクラスター化インデックスです。クラスター化インデックスのあるテーブルでは、行データのキーはユーザーが指定したプライマリキーデータで構成されます。したがって、クラスター化インデックスのあるテーブルには行を格納するために1つのキー値ペアのみが必要です：
    - プライマリキーデータ（キー） - 行データ（値）

> **注：**
>
> TiDBはクラスター化を表の `PRIMARY KEY` によってのみサポートしています。クラスター化インデックスが有効になっている場合、用語「THE `PRIMARY KEY`」および「THE CLUSTERED INDEX」は同義として使用されるかもしれません。`PRIMARY KEY` は制約（論理的なプロパティ）を指し、クラスター化インデックスはデータの格納方法の物理的な実装を示します。

## ユーザシナリオ

ノンクラスター化インデックスのあるテーブルと比較して、クラスター化インデックスのあるテーブルは次のシナリオでパフォーマンスとスループットの利点を提供します：

+ データが挿入される際、クラスター化インデックスはネットワークからのインデックスデータの書き込みを1つ減らします。
+ 条件が等価なクエリがプライマリキーのみを対象とする場合、クラスター化インデックスはネットワークからのインデックスデータの読み取りを1つ減らします。
+ 等価または範囲条件を持つクエリがプライマリキーのみを対象とする場合、クラスター化インデックスはネットワークからの複数のインデックスデータの読み取りを減らします。
+ 等価または範囲条件を持つクエリがプライマリキーのプレフィックスのみを対象とする場合、クラスター化インデックスはネットワークからの複数のインデックスデータの読み取りを減らします。

一方で、クラスター化インデックスのあるテーブルにはいくつかの欠点があります。以下を参照してください：

- 近い値を持つ大量のプライマリキーを挿入する場合、書き込みホットスポットの問題が発生する可能性があります。
- プライマリキーのデータ型が64ビットよりも大きい場合、特に複数のセカンダリインデックスがある場合、テーブルデータはより多くのストレージスペースを占有します。

## 使用法

## クラスター化インデックスを持つテーブルを作成する

TiDB v5.0以降、`CREATE TABLE` 文の `PRIMARY KEY` の後に予約されていないキーワード `CLUSTERED` または `NONCLUSTERED` を追加して、テーブルのプライマリキーがクラスター化インデックスかどうかを指定できます。例：

```sql
CREATE TABLE t (a BIGINT PRIMARY KEY CLUSTERED, b VARCHAR(255));
CREATE TABLE t (a BIGINT PRIMARY KEY NONCLUSTERED, b VARCHAR(255));
CREATE TABLE t (a BIGINT KEY CLUSTERED, b VARCHAR(255));
CREATE TABLE t (a BIGINT KEY NONCLUSTERED, b VARCHAR(255));
CREATE TABLE t (a BIGINT, b VARCHAR(255), PRIMARY KEY(a, b) CLUSTERED);
CREATE TABLE t (a BIGINT, b VARCHAR(255), PRIMARY KEY(a, b) NONCLUSTERED);
```

`KEY` と `PRIMARY KEY` は列定義で同じ意味を持ちます。

また、TiDBでは[コメント構文](/comment-syntax.md)を使用して、プライマリキーのタイプを指定することもできます。例：

```sql
CREATE TABLE t (a BIGINT PRIMARY KEY /*T![clustered_index] CLUSTERED */, b VARCHAR(255));
CREATE TABLE t (a BIGINT PRIMARY KEY /*T![clustered_index] NONCLUSTERED */, b VARCHAR(255));
CREATE TABLE t (a BIGINT, b VARCHAR(255), PRIMARY KEY(a, b) /*T![clustered_index] CLUSTERED */);
CREATE TABLE t (a BIGINT, b VARCHAR(255), PRIMARY KEY(a, b) /*T![clustered_index] NONCLUSTERED */);
```

`CLUSTERED`/`NONCLUSTERED` キーワードを明示的に指定しない場合、デフォルトの動作はシステム変数 [`@@global.tidb_enable_clustered_index`](/system-variables.md#tidb_enable_clustered_index-new-in-v50) によって制御されます。この変数のサポートされている値は次のとおりです：

- `OFF` は、デフォルトでプライマリキーがノンクラスター化インデックスとして作成されることを示します。
- `ON` は、デフォルトでプライマリキーがクラスター化インデックスとして作成されることを示します。
- `INT_ONLY` は、動作は `alter-primary-key` 構成項目によって制御されます。`alter-primary-key` が `true` に設定されている場合、デフォルトでプライマリキーがノンクラスター化インデックスとして作成されます。`false` に設定されている場合、整数列で構成されるプライマリキーのみがクラスター化インデックスとして作成されます。

`@@global.tidb_enable_clustered_index` のデフォルト値は `ON` です。

### クラスター化インデックスの追加と削除

TiDBは、テーブルが作成された後にクラスター化インデックスを追加したり削除したりすることをサポートしていません。また、クラスター化インデックスとノンクラスター化インデックスの相互変換もサポートしていません。例：

```sql
ALTER TABLE t ADD PRIMARY KEY(b, a) CLUSTERED; -- 現在はサポートされていません。
ALTER TABLE t DROP PRIMARY KEY;     -- プライマリキーがクラスター化インデックスの場合、サポートされていません。
ALTER TABLE t DROP INDEX `PRIMARY`; -- プライマリキーがクラスター化インデックスの場合、サポートされていません。
```

### ノンクラスター化インデックスの追加と削除

TiDBは、テーブルが作成された後でノンクラスター化インデックスを追加したり削除したりすることをサポートしています。`NONCLUSTERED` キーワードを明示的に指定したり、省略することができます。例：

```sql
ALTER TABLE t ADD PRIMARY KEY(b, a) NONCLUSTERED;
ALTER TABLE t ADD PRIMARY KEY(b, a); -- キーワードを省略した場合、デフォルトでプライマリキーはノンクラスター化インデックスです。
ALTER TABLE t DROP PRIMARY KEY;
ALTER TABLE t DROP INDEX `PRIMARY`;
```

### プライマリキーがクラスター化インデックスかどうかを確認する

次のいずれかの方法を使用して、テーブルのプライマリキーがクラスター化インデックスかどうかを確認できます：

- `SHOW CREATE TABLE` コマンドを実行します。
- `SHOW INDEX FROM` コマンドを実行します。
- `information_schema.tables` システムテーブルの `TIDB_PK_TYPE` 列をクエリします。

`SHOW CREATE TABLE` コマンドを実行することで、`PRIMARY KEY` の属性が `CLUSTERED` または `NONCLUSTERED` かどうかを確認できます。例：

```sql
mysql> SHOW CREATE TABLE t;
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                      |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `a` bigint(20) NOT NULL,
  `b` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`a`) /*T![clustered_index] CLUSTERED */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```

`SHOW INDEX FROM` コマンドを実行することで、`Clustered` 列の結果が `YES` または `NO` かどうかを確認できます。例：

```sql
mysql> SHOW INDEX FROM t;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+-----------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression | Clustered |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+-----------+
| t     |          0 | PRIMARY  |            1 | a           | A         |           0 |     NULL | NULL   |      | BTREE      |         |               | YES     | NULL       | YES       |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+-----------+
1 row in set (0.01 sec)
```

`information_schema.tables` システムテーブルの `TIDB_PK_TYPE` 列をクエリすることで、結果が `CLUSTERED` または `NONCLUSTERED` かどうかを確認できます。例：

```sql
mysql> SELECT TIDB_PK_TYPE FROM information_schema.tables WHERE table_schema = 'test' AND table_name = 't';
+--------------+
| TIDB_PK_TYPE |
+--------------+
| CLUSTERED    |
+--------------+
1 row in set (0.03 sec)
```

## 制限事項

現在、クラスター化インデックス機能にはいくつかの異なる制限事項があります。以下を参照してください：

- サポートされていない状況とサポート計画外の状況：
- クラスター化されたインデックスを使用して、[`SHARD_ROW_ID_BITS`](/shard-row-id-bits.md)属性を併用することはサポートされていません。また、クラスター化されたインデックスを持つテーブルで、[`PRE_SPLIT_REGIONS`](/sql-statements/sql-statement-split-region.md#pre_split_regions)属性は効果がありません。
- クラスター化されたインデックスを持つテーブルのダウングレードはサポートされていません。そのようなテーブルをダウングレードする必要がある場合は、データを移行するために論理バックアップツールを使用してください。
- まだサポートされていないが、今後のサポート計画内の状況:
    - `ALTER TABLE`ステートメントを使用してクラスター化されたインデックスを追加、削除、変更することはサポートされていません。
- 特定バージョンの制限:
    - v5.0では、TiDB Binlogとクラスター化インデックス機能を併用することはサポートされていません。TiDB Binlogが有効になると、TiDBは主キーのクラスター化インデックスとして単一の整数列のみを作成することを許可します。 TiDB Binlogは、既存のクラスター化されたインデックスを持つテーブルのデータ変更（挿入、削除、更新など）をダウンストリームに複製しません。クラスター化されたインデックスを持つテーブルをダウンストリームに複製する必要がある場合は、クラスターをv5.1にアップグレードするか、代わりに[ **TiCDC**](https://docs.pingcap.com/tidb/stable/ticdc-overview)を使用して複製してください。

TiDB Binlogが有効になると、作成するクラスター化されたインデックスが単一の整数プライマリキーでない場合、TiDBは次のエラーを返します：

```sql
mysql> CREATE TABLE t (a VARCHAR(255) PRIMARY KEY CLUSTERED);
ERROR 8200 (HY000): Cannot create clustered index table when the binlog is ON
```

クラスター化されたインデックスと`SHARD_ROW_ID_BITS`属性を併用した場合、TiDBは次のエラーを報告します：

```sql
mysql> CREATE TABLE t (a VARCHAR(255) PRIMARY KEY CLUSTERED) SHARD_ROW_ID_BITS = 3;
ERROR 8200 (HY000): Unsupported shard_row_id_bits for table with primary key as row id
```

## 互換性

### 以前および以降のTiDBバージョンとの互換性

TiDBは、クラスター化されたインデックスを持つテーブルをアップグレードすることはサポートしていますが、そのようなテーブルをダウングレードすることはサポートしていません。つまり、後のTiDBバージョンでクラスター化されたインデックスを持つテーブルのデータは、以前のバージョンでは使用できません。

クラスター化されたインデックスの機能は、TiDB v3.0およびv4.0で部分的にサポートされています。次の要件をすべて満たした場合にデフォルトで有効になります。

- テーブルに`PRIMARY KEY`が含まれている。
- `PRIMARY KEY`が1つの列のみで構成されている。
- `PRIMARY KEY`が`INTEGER`である。

TiDB v5.0以降、クラスター化されたインデックスの機能はすべてのタイプのプライマリキーに完全にサポートされていますが、デフォルトの動作はTiDB v3.0およびv4.0と一貫しています。デフォルトの動作を変更するには、システム変数`@@tidb_enable_clustered_index`を`ON`または`OFF`に構成できます。詳細については、[クラスター化されたインデックスを持つテーブルを作成する](#create-a-table-with-clustered-indexes)を参照してください。

### MySQLとの互換性

TiDB専用のコメント構文は、キーワード`CLUSTERED`および`NONCLUSTERED`をコメントで囲むことをサポートしています。`SHOW CREATE TABLE`の結果にはTiDB専用のSQLコメントが含まれています。MySQLデータベースと以前のバージョンのTiDBデータベースは、これらのコメントを無視します。

### TiDB移行ツールとの互換性

クラスター化されたインデックス機能は、v5.0および以降のバージョンでの次の移行ツールとのみ互換性があります。

- バックアップとリストアツール: BR、Dumpling、およびTiDB Lightning。
- データ移行および複製ツール: DMおよびTiCDC。

ただし、v5.0のBRツールを使用してクラスター化されたインデックスを持たないインデックスを持つテーブルをクラスター化されたインデックスを持つテーブルに変換することはできません。その逆も同様です。

### その他のTiDBの機能との互換性

結合されたプライマリキーまたは単一の非整数プライマリキーを持つテーブルの場合、プライマリキーを非クラスター化されたインデックスからクラスター化されたインデックスに変更すると、その行データのキーも変更されます。したがって、TiDB v5.0以降のバージョンで実行可能であった`SPLIT TABLE BY/BETWEEN`ステートメントは、その後も動作しなくなります。クラスター化されたインデックスを持つテーブルを`SPLIT TABLE BY/BETWEEN`を使用して分割する場合、整数値を指定するのではなく、プライマリキー列の値を提供する必要があります。次の例を参照してください：

```sql
mysql> create table t (a int, b varchar(255), primary key(a, b) clustered);
Query OK, 0 rows affected (0.01 sec)
mysql> split table t between (0) and (1000000) regions 5;
ERROR 1105 (HY000): Split table region lower value count should be 2
mysql> split table t by (0), (50000), (100000);
ERROR 1136 (21S01): Column count doesn't match value count at row 0
mysql> split table t between (0, 'aaa') and (1000000, 'zzz') regions 5;
+--------------------+----------------------+
| TOTAL_SPLIT_REGION | SCATTER_FINISH_RATIO |
+--------------------+----------------------+
|                  4 |                    1 |
+--------------------+----------------------+
1 row in set (0.00 sec)
mysql> split table t by (0, ''), (50000, ''), (100000, '');
+--------------------+----------------------+
| TOTAL_SPLIT_REGION | SCATTER_FINISH_RATIO |
+--------------------+----------------------+
|                  3 |                    1 |
+--------------------+----------------------+
1 row in set (0.01 sec)
```

属性[`AUTO_RANDOM`](/auto-random.md)は、クラスター化されたインデックスでのみ使用できます。それ以外の場合、TiDBは次のエラーを返します：

```sql
mysql> create table t (a bigint primary key nonclustered auto_random);
ERROR 8216 (HY000): Invalid auto random: column a is not the integer primary key, or the primary key is nonclustered
```