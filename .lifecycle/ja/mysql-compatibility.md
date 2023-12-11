---
title: MySQL 互換性
summary: TiDB と MySQL の互換性、サポートされていない機能と異なる機能について学びます。
aliases: ['/docs/dev/mysql-compatibility/','/docs/dev/reference/mysql-compatibility/']
---

# MySQL 互換性

<CustomContent platform="tidb">

TiDB は MySQL プロトコルや MySQL 5.7 および MySQL 8.0 の共通機能や構文と高い互換性を持っています。MySQL のエコシステムツール（PHPMyAdmin、Navicat、MySQL Workbench、DBeaver など）や MySQL クライアントを TiDB でも使用することができます。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDB は MySQL プロトコルや MySQL 5.7 および MySQL 8.0 の共通機能や構文と高い互換性を持っています。MySQL のエコシステムツール（PHPMyAdmin、Navicat、MySQL Workbench、DBeaver など）や MySQL クライアントを TiDB でも使用することができます。

</CustomContent>

ただし、TiDB では MySQL の一部の機能がサポートされていません。これは問題を解決するためのより良い方法が現在存在するため（たとえば XML 関数の代わりに JSON の使用）、または現在の需要と必要な取り組みの欠如のため（ストアドプロシージャや関数など）といった理由があるかもしれません。さらに、いくつかの機能は分散システムで実装するのが難しい場合もあります。

<CustomContent platform="tidb">

TiDB は MySQL 複製プロトコルをサポートしていません。その代わりに、特定のツールが MySQL とデータを複製する機能を提供しています：

- MySQL からのデータ複製：[TiDB データ移行 (DM)](/dm/dm-overview.md) は MySQL や MariaDB から TiDB への完全データ移行および増分データレプリケーションをサポートするツールです。
- MySQL へのデータ複製：[TiCDC](/ticdc/ticdc-overview.md) は TiDB の増分データを TiKV の変更ログを取得して複製するツールです。TiCDC は [MySQL シンク](/ticdc/ticdc-overview.md#replication-consistency) を使用して TiDB の増分データを MySQL へ複製します。

</CustomContent>

<CustomContent platform="tidb">

> **ノート:**
>
> このページでは MySQL と TiDB の一般的な違いについて説明しています。セキュリティや悲観的トランザクションモードに関する MySQL との互換性についての詳細情報については、[セキュリティ](/security-compatibility-with-mysql.md) および [悲観的トランザクションモード](/pessimistic-transaction.md#difference-with-mysql-innodb) の専用ページを参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **ノート:**
>
> MySQL と TiDB のトランザクションの違いについての情報は、[悲観的トランザクションモード](/pessimistic-transaction.md#difference-with-mysql-innodb) を参照してください。

</CustomContent>

TiDB の機能を [TiDB Playground](https://play.tidbcloud.com/?utm_source=docs&utm_medium=mysql_compatibility) で試すことができます。

## サポートされていない機能

+ ストアドプロシージャと関数
+ トリガー
+ イベント
+ ユーザー定義関数
+ `FULLTEXT` 構文およびインデックス [#1793](https://github.com/pingcap/tidb/issues/1793)
+ `SPATIAL`（または `GIS`/`GEOMETRY`）関数、データ型およびインデックス [#6347](https://github.com/pingcap/tidb/issues/6347)
+ `ascii`、`latin1`、`binary`、`utf8`、`utf8mb4`、`gbk` 以外の文字セット。
+ SYS スキーマ
+ オプティマイザトレース
+ XML 関数
+ X-Protocol [#1109](https://github.com/pingcap/tidb/issues/1109)
+ 列レベルの権限 [#9766](https://github.com/pingcap/tidb/issues/9766)
+ `XA` 構文（TiDB は内部的に二段階コミットを使用していますが、これは SQL インターフェースを介して公開されていません）
+ `CREATE TABLE tblName AS SELECT stmt` 構文 [#4754](https://github.com/pingcap/tidb/issues/4754)
+ `CHECK TABLE` 構文 [#4673](https://github.com/pingcap/tidb/issues/4673)
+ `CHECKSUM TABLE` 構文 [#1895](https://github.com/pingcap/tidb/issues/1895)
+ `REPAIR TABLE` 構文
+ `OPTIMIZE TABLE` 構文
+ `HANDLER` ステートメント
+ `CREATE TABLESPACE` ステートメント
+ "Session Tracker: Add GTIDs context to the OK packet"
+ 降順インデックス [#2519](https://github.com/pingcap/tidb/issues/2519)
+ `SKIP LOCKED` 構文 [#18207](https://github.com/pingcap/tidb/issues/18207)
+ Lateral derived tables [#40328](https://github.com/pingcap/tidb/issues/40328)

## MySQL との違い

### オートインクリメント ID

+ TiDB では、オートインクリメント列の値（ID）は個々の TiDB サーバー内でグローバルにユニークであり、増分です。複数の TiDB サーバー間で ID を増分させるには、[`AUTO_INCREMENT` MySQL 互換モード](/auto-increment.md#mysql-compatibility-mode) を使用できます。ただし、ID は必ずしも連続的に割り当てられるわけではないため、「重複エラー」メッセージに遭遇しないように、デフォルト値とカスタム値を混在させるのは避けることをお勧めします。

+ `tidb_allow_remove_auto_inc` システム変数を使用して、`AUTO_INCREMENT` 列属性を削除することを許可または禁止できます。列属性を削除するには、`ALTER TABLE MODIFY` または `ALTER TABLE CHANGE` 構文を使用します。

+ TiDB では `AUTO_INCREMENT` 列属性の追加はサポートされておらず、一旦削除されると復元することはできません。

+ TiDB v6.6.0 およびそれ以前のバージョンでは、TiDB の自動増分列の動作は MySQL InnoDB と同じで、それらが主キーまたはインデックスのプレフィックスであることが要求されていました。v7.0.0 以降、TiDB はこの制限を廃止し、より柔軟なテーブル主キーの定義を可能にしました。[#40580](https://github.com/pingcap/tidb/issues/40580)

詳細については、[`AUTO_INCREMENT`](/auto-increment.md) を参照してください。

> **ノート:**
>
> + テーブルを作成する際に主キーを指定しない場合、TiDB は `_tidb_rowid` を行を識別するために使用します。この値の割り当ては、オートインクリメント列（存在する場合）と共有されています。オートインクリメント列を主キーとして指定した場合は、TiDB はこの列を行を識別するために使用します。この状況では、次のような状況が発生する可能性があります：

```sql
mysql> CREATE TABLE t(id INT UNIQUE KEY AUTO_INCREMENT);
Query OK, 0 rows affected (0.05 sec)

mysql> INSERT INTO t VALUES();
Query OK, 1 rows affected (0.00 sec)

mysql> INSERT INTO t VALUES();
Query OK, 1 rows affected (0.00 sec)

mysql> INSERT INTO t VALUES();
Query OK, 1 rows affected (0.00 sec)

mysql> SELECT _tidb_rowid, id FROM t;
+-------------+------+
| _tidb_rowid | id   |
+-------------+------+
|           2 |    1 |
|           4 |    3 |
|           6 |    5 |
+-------------+------+
3 rows in set (0.01 sec)
```

上記のように、共有された割り当て器のため、`id` は毎回 2 ずつ増分します。この動作は [MySQL 互換モード](/auto-increment.md#mysql-compatibility-mode) では変わり、共有された割り当て器および数値のスキップがないため、このような動作は起こりません。

<CustomContent platform="tidb">

> **ノート:**
>
> `AUTO_INCREMENT` 属性は、本番環境でホットスポットを引き起こす可能性があります。詳細については [ホットスポットの問題をトラブルシューティング](/troubleshoot-hot-spot-issues.md) を参照してください。[`AUTO_RANDOM`](/auto-random.md) を代わりに使用することをお勧めします。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **ノート:**
>
> `AUTO_INCREMENT` 属性は、本番環境でホットスポットを引き起こす可能性があります。詳細については [ホットスポットの問題をトラブルシューティング](https://docs.pingcap.com/tidb/stable/troubleshoot-hot-spot-issues#handle-auto-increment-primary-key-hotspot-tables-using-auto_random) を参照してください。[`AUTO_RANDOM`](/auto-random.md) を代わりに使用することをお勧めします。

</CustomContent>

### パフォーマンススキーマ

<CustomContent platform="tidb">

TiDB では [Prometheus と Grafana](/tidb-monitoring-api.md) を組み合わせてパフォーマンス監視メトリクスを保存およびクエリするために使用しています。TiDB では、パフォーマンススキーマテーブルは空の結果を返します。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDB Cloud でのパフォーマンスメトリクスの確認には、TiDB Cloud コンソールのクラスター概要ページを確認するか、[サードパーティの監視インテグレーション](/tidb-cloud/third-party-monitoring-integrations.md) を使用します。TiDB では、パフォーマンススキーマテーブルは空の結果を返します。

</CustomContent>

### クエリ実行計画

TiDB におけるクエリ実行計画（`EXPLAIN`/`EXPLAIN FOR`）の出力形式、内容、および権限設定は、MySQL とは大きく異なります。
TiDBでは、MySQLのsystem変数`optimizer_switch`は読み取り専用であり、クエリプランに影響を与えません。オプティマイザヒントは、MySQLと似た構文で使用できますが、利用可能なヒントやその実装は異なる場合があります。

詳細については、[クエリ実行計画の理解](/explain-overview.md)を参照してください。

### 組み込み関数

TiDBは、MySQLのほとんどの組み込み関数をサポートしていますが、すべてをサポートしているわけではありません。利用可能な関数のリストを取得するには、`SHOW BUILTINS`ステートメントを使用できます。

詳細については、[TiDB SQL文法](https://pingcap.github.io/sqlgram/#functioncallkeyword)を参照してください。

### DDL操作

TiDBでは、すべてのサポートされているDDL変更をオンラインで実行できます。ただし、MySQLと比較して、TiDBには以下のようないくつかの主なDDL操作の制限があります。

* 1つの`ALTER TABLE`ステートメントを使用して、テーブルの複数のスキーマオブジェクト（列やインデックスなど）を変更する際、同じオブジェクトを複数回指定することはサポートされていません。たとえば、`ALTER TABLE t1 MODIFY COLUMN c1 INT、DROP COLUMN c1`コマンドを実行すると、`Unsupported operate same column/index`エラーが出力されます。
* `TIFLASH REPLICA`、`SHARD_ROW_ID_BITS`、`AUTO_ID_CACHE`などの複数のTiDB固有のスキーマオブジェクトを1つの`ALTER TABLE`ステートメントで変更することはサポートされていません。
* TiDBは、`ALTER TABLE`を使用して一部のデータ型の変更をサポートしていません。たとえば、`DECIMAL`型から`DATE`型への変更はサポートされていません。データ型の変更がサポートされない場合、TiDBは`Unsupported modify column: type %d not match origin %d`エラーを報告します。詳細については、[`ALTER TABLE`](/sql-statements/sql-statement-modify-column.md)を参照してください。
* `ALGORITHM={INSTANT,INPLACE,COPY}`構文は、TiDBではアサーションとして機能し、`ALTER`アルゴリズムを変更しません。詳細については、[`ALTER TABLE`](/sql-statements/sql-statement-alter-table.md)を参照してください。
* `CLUSTERED`タイプのプライマリキーの追加/削除はサポートされていません。`CLUSTERED`タイプのプライマリキーの詳細については、[クラスタ化インデックス](/clustered-indexes.md)を参照してください。
* 異なるタイプのインデックス（`HASH|BTREE|RTREE|FULLTEXT`）はサポートされておらず、指定された場合には解析されて無視されます。
* TiDBは`HASH`、`RANGE`、`LIST`、`KEY`のパーティションタイプをサポートしています。現在、`KEY`パーティションタイプは空のパーティション列リストのパーティションステートメントをサポートしていません。サポートされないパーティションタイプの場合、TiDBは`Warning: Unsupported partition type %s, treat as normal table`と返します（`%s`は特定のサポートされないパーティションタイプです）。
* Range、Range COLUMNS、List、List COLUMNSのパーティションテーブルは`ADD`、`DROP`、`TRUNCATE`、`REORGANIZE`操作をサポートしています。その他のパーティション操作は無視されます。
* Hash、Keyパーティションテーブルは`ADD`、`COALESCE`、`TRUNCATE`操作をサポートしています。その他のパーティション操作は無視されます。
* 次の構文は、パーティションテーブルではサポートされていません。

    - `SUBPARTITION`
    - `{CHECK|OPTIMIZE|REPAIR|IMPORT|DISCARD|REBUILD} PARTITION`

    パーティショニングの詳細については、[パーティショニング](/partitioned-table.md)を参照してください。

### テーブルの解析

TiDBでは、[統計情報の収集](/statistics.md#manual-collection)はMySQLとは異なり、テーブルの統計情報を完全に再構築するため、よりリソース集約型で完了までに時間がかかります。これに対し、MySQL/InnoDBは比較的軽量で短寿命の操作を行います。

詳細については、[`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md)を参照してください。

### `SELECT`構文の制限

TiDBは、以下の`SELECT`構文をサポートしていません。

- `SELECT ... INTO @variable`
- `SELECT .. GROUP BY expr`は、MySQL 5.7と異なり、`GROUP BY expr ORDER BY expr`を含意しません。

詳細については、[`SELECT`](/sql-statements/sql-statement-select.md)ステートメントリファレンスを参照してください。

### `UPDATE`ステートメント

[`UPDATE`](/sql-statements/sql-statement-update.md)ステートメントリファレンスを参照してください。

### ビュー

TiDBのビューは更新可能ではなく、`UPDATE`、`INSERT`、`DELETE`などの書き込み操作をサポートしていません。

### 一時テーブル

詳細については、[TiDBローカル一時テーブルとMySQL一時テーブルの互換性](/temporary-tables.md#compatibility-with-mysql-temporary-tables)を参照してください。

### 文字セットと照合順序

* TiDBでサポートされている文字セットと照合順序については、[文字セットと照合順序の概要](/character-set-and-collation.md)を参照してください。
* MySQL互換性のGBK文字セットの詳細については、[GBKの互換性](/character-set-gbk.md#mysql-compatibility)を参照してください。
* TiDBは、テーブルで使用される文字セットを国立文字セットとして継承します。

### ストレージエンジン

TiDBは、代替ストレージエンジンでテーブルを作成することを許可しています。しかし、TiDBによって記述されたメタデータは、互換性を確保するためにInnoDBストレージエンジンのものです。

<CustomContent platform="tidb">

[`--store`](/command-line-flags-for-tidb-configuration.md#--store)オプションを使用してストレージエンジンを指定するには、TiDBサーバーを起動する必要があります。このストレージエンジンの抽象化機能は、MySQLと類似しています。

</CustomContent>

### SQLモード

TiDBは、ほとんどの[SQLモード](/sql-mode.md)をサポートしています。

- `Oracle`や`PostgreSQL`などの互換モードは解析されますが、無視されます。これらの互換モードはMySQL 5.7で非推奨となり、MySQL 8.0では削除されました。
- `ONLY_FULL_GROUP_BY`モードは、MySQL 5.7とは少し異なる[意味論的な違い](/functions-and-operators/aggregate-group-by-functions.md#differences-from-mysql)があります。
- MySQLの`NO_DIR_IN_CREATE`および`NO_ENGINE_SUBSTITUTION` SQLモードは互換性のために受け入れられますが、TiDBには適用されません。

### デフォルトの違い

TiDBは、MySQL 5.7およびMySQL 8.0と比較して、デフォルトの違いがあります。

- デフォルトの文字セット：
    - TiDBのデフォルト値は`utf8mb4`です。
    - MySQL 5.7のデフォルト値は`latin1`です。
    - MySQL 8.0のデフォルト値は`utf8mb4`です。
- デフォルト照合順序：
    - TiDBのデフォルトの照合順序は`utf8mb4_bin`です。
    - MySQL 5.7のデフォルトの照合順序は`utf8mb4_general_ci`です。
    - MySQL 8.0のデフォルトの照合順序は`utf8mb4_0900_ai_ci`です。
- デフォルトSQLモード：
    - TiDBのデフォルトSQLモードには、以下のモードが含まれます：`ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`。
    - MySQLのデフォルトSQLモード：
        - MySQL 5.7のデフォルトSQLモードはTiDBと同じです。
        - MySQL 8.0のデフォルトSQLモードには、以下のモードが含まれます：`ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION`。
- `lower_case_table_names`のデフォルト値：
    - TiDBのデフォルト値は`2`であり、現在は`2`のみがサポートされています。
    - MySQLのデフォルト値は次のとおりです：
        - Linux：`0`。つまり、テーブルやデータベース名は`CREATE TABLE`または`CREATE DATABASE`ステートメントで指定された文字の大文字と小文字に応じてディスクに格納されます。名前の比較は大文字と小文字を区別します。
        - Windows：`1`。これは、テーブルの名前がディスクに小文字で保存され、名前の比較が大文字と小文字を区別しません。MySQLはストレージと検索時にすべてのテーブル名を小文字に変換します。この動作はデータベース名とテーブルのエイリアスにも適用されます。
        - macOS：`2`。これは、テーブルやデータベース名が`CREATE TABLE`または`CREATE DATABASE`ステートメントで指定された文字の大文字と小文字に応じてディスクに格納されるが、MySQLは検索時にそれらを小文字に変換します。名前の比較は大文字と小文字を区別しません。
- `explicit_defaults_for_timestamp`のデフォルト値：
    - TiDBのデフォルト値は`ON`であり、現在は`ON`のみがサポートされています。
    - MySQLのデフォルト値は次のとおりです：
        - MySQL 5.7：`OFF`。
        - MySQL 8.0：`ON`。

### 日付と時刻

TiDBは、次の点に注意して名前付きタイムゾーンをサポートしています：

+ TiDBは、現在システムにインストールされているすべてのタイムゾーンルール（通常は`tzdata`パッケージ）を使用して計算に使用します。これにより、タイムゾーンテーブルデータをインポートする必要なく、すべてのタイムゾーン名を使用できます。タイムゾーンテーブルデータをインポートしても計算ルールは変わりません。
+ 現在、MySQLはデフォルトでローカルタイムゾーンを使用し、その後計算についてシステムにビルトインされている現在のタイムゾーンルール（たとえば、夏時間が開始されるとき）に依存します。[タイムゾーンテーブルデータをインポート](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html#time-zone-installation)しないと、MySQLはタイムゾーン名で指定することはできません。

### タイプシステムの違い

次のカラムタイプはMySQLでサポートされていますが、TiDBでは**サポートされていません**：
- `SQL_TSI_*`（`SQL_TSI_MONTH`、`SQL_TSI_WEEK`、`SQL_TSI_DAY`、`SQL_TSI_HOUR`、`SQL_TSI_MINUTE`、`SQL_TSI_SECOND`を含むが、`SQL_TSI_YEAR`を除く）

### 廃止された機能による非互換性

TiDBでは、MySQLで廃止された特定の機能が実装されていません。これには次のものが含まれます。

- 浮動小数点型の精度の指定。MySQL 8.0では、[この機能が廃止](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)されており、代わりに`DECIMAL`型を使用することが推奨されています。
- `ZEROFILL`属性。MySQL 8.0では、[この機能が廃止](https://dev.mysql.com/doc/refman/8.0/en/numeric-type-attributes.html)されており、代わりに数値の値をアプリケーション内でパディングすることが推奨されています。

### `CREATE RESOURCE GROUP`、`DROP RESOURCE GROUP`、および`ALTER RESOURCE GROUP`ステートメント

リソースグループの作成、変更、および削除に関する以下のステートメントは、MySQLとは異なるサポートされるパラメータを持っています。詳細については、次のドキュメントを参照してください。

- [`CREATE RESOURCE GROUP`](/sql-statements/sql-statement-create-resource-group.md)
- [`DROP RESOURCE GROUP`](/sql-statements/sql-statement-drop-resource-group.md)
- [`ALTER RESOURCE GROUP`](/sql-statements/sql-statement-alter-resource-group.md)