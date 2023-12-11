---
title: テーブルの作成
summary: テーブル作成における定義、ルール、およびガイドラインを学びます。

# テーブルの作成

このドキュメントでは、SQLステートメントと関連するベストプラクティスを使用してテーブルを作成する方法について紹介します。[書店](/develop/dev-guide-bookshop-schema-design.md)アプリケーションを使用したTiDBベースの実例が、ベストプラクティスを示しています。

## 開始前に

このドキュメントを読む前に、以下のタスクが完了していることを確認してください。

- [TiDB Serverless Clusterの構築](/develop/dev-guide-build-cluster-in-cloud.md)。
- [スキーマ設計概要](/develop/dev-guide-schema-design-overview.md)の読み込み。
- [データベースの作成](/develop/dev-guide-create-database.md)。

## テーブルとは

[テーブル](/develop/dev-guide-schema-design-overview.md#table)とは、[データベース](/develop/dev-guide-schema-design-overview.md#database)に属する、TiDBクラスタ内の論理オブジェクトです。SQLステートメントから送信されたデータを保存するために使用されます。テーブルはデータを行と列の形式で保存します。テーブルには少なくとも1つの列が必要です。`n`個の列を定義した場合、各データ行は`n`個の列と同じフィールドを持ちます。

## テーブルの名前を付ける

テーブルを作成する最初のステップは、テーブルに名前を付けることです。将来的に自分自身や同僚に大きな苦痛をもたらす意味のない名前は使用しないでください。企業や組織のテーブル命名規則に従うことをお勧めします。

通常、`CREATE TABLE`ステートメントは以下の形式を取ります:

```sql
CREATE TABLE {table_name} ( {elements} );
```

**パラメータの説明**

- `{table_name}`: 作成するテーブルの名前。
- `{elements}`: 列定義や主キー定義などの列要素のコンマ区切りリスト。

例えば`bookshop`データベース内にユーザ情報を保存するためのテーブルを作成する必要があるとします。

以下のSQLステートメントは、まだ1つの列も追加されていないため、実行できません。

```sql
CREATE TABLE `bookshop`.`users` (
);
```

## 列の定義

**列**はテーブルに属しています。各テーブルには少なくとも1つの列が必要です。列は各行の値を単一のデータ型の小さなセルに分割することで、テーブルに構造を提供します。

列の定義は、通常以下の形式を取ります。

```
{column_name} {data_type} {column_qualification}
```

**パラメータの説明**

- `{column_name}`: 列の名前。
- `{data_type}`: [データ型](/data-type-overview.md)の列。
- `{column_qualification}`: **列レベルの制約**や[生成列](/generated-columns.md)のクローズなど、列の修飾の仕方。

`users`テーブルにいくつかの列を追加する場合、一意の識別子`id`、`balance`、`nickname`などの列を追加できます。

```sql
CREATE TABLE `bookshop`.`users` (
  `id` bigint,
  `nickname` varchar(100),
  `balance` decimal(15,2)
);
```

上記のステートメントでは、`id`という名前のフィールドが`bigint`タイプで定義されており、これはユニークなユーザ識別子を表します。つまり、すべてのユーザ識別子は`bigint`タイプである必要があります。

次に`nickname`という名前のフィールドを定義し、これは長さが100文字以内の[varchar](/data-type-string.md#varchar-type)タイプです。これはユーザのニックネームが`varchar`タイプであり、100文字を超えないことを意味します。

最後に`balance`という名前のフィールドを追加しています。これは[decimal](/data-type-numeric.md#decimal-type)タイプで、**精度**が`15`、**スケール**が`2`です。**精度**はフィールド内の総桁数を表し、**スケール**は小数点以下の桁数を表します。たとえば、`decimal(5,2)`は精度が`5`、スケールが`2`で、範囲は`-999.99`から`999.99`です。`decimal(6,1)`は精度が`6`、スケールが`1`で、範囲は`-99999.9`から`99999.9`です。**decimal**は[固定小数点型](/data-type-numeric.md#fixed-point-types)で、正確な数値を格納するために使用できます。正確な数値が必要なシナリオ（たとえば、ユーザ関連のプロパティ）では、**decimal**タイプを使用することを確認してください。

TiDBは他にも多くの列のデータ型をサポートしており、[整数型](/data-type-numeric.md#integer-types)、[浮動小数点型](/data-type-numeric.md#floating-point-types)、[固定小数点型](/data-type-numeric.md#fixed-point-types)、[日付と時刻の型](/data-type-date-and-time.md)、および[enum型](/data-type-string.md#enum-type)などを含む[データ型](/data-type-overview.md)のサポートもあります。データベースに保存したいデータに適した**データ型**を使用してください。

少し複雑にするために、`bookshop`の中核となる`books`テーブルに`books`のid、タイトル、タイプ（例: 雑誌、小説、ライフ、アート）、在庫、価格、および発行日のフィールドを定義できます。

```sql
CREATE TABLE `bookshop`.`books` (
  `id` bigint NOT NULL,
  `title` varchar(100),
  `type` enum('Magazine', 'Novel', 'Life', 'Arts', 'Comics', 'Education & Reference', 'Humanities & Social Sciences', 'Science & Technology', 'Kids', 'Sports'),
  `published_at` datetime,
  `stock` int,
  `price` decimal(15,2)
);
```

このテーブルには`users`テーブルよりも多くのデータ型が含まれています。

- [int](/data-type-numeric.md#integer-types): ディスク使用量が過剰になったりパフォーマンスに影響を与えたりするのを避けるために、適切なサイズのタイプを使用することをお勧めします（タイプ範囲が大きすぎる場合やデータのオーバーフローが発生する場合など）。
- [datetime](/data-type-date-and-time.md): **datetime**タイプは時間値を保存するために使用できます。
- [enum](/data-type-string.md#enum-type): enum型は限られた選択の値を保存するために使用できます。

## 主キーの選択

[主キー](/constraints.md#primary-key)とは、テーブル内の行を一意に識別する列または列のセットです。

> **注:**
>
> TiDBにおける**主キー**のデフォルトの定義は、[InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html)(MySQLの一般的なストレージエンジン)とは異なります。
>
> - **InnoDB**: **主キー**は一意であり、**空でない**です。そして**クラスター化インデックス**です。
>
> - TiDB: **主キー**は一意であり、**空でない**です。ただし、**主キー**が**クラスター化インデックス**であるかどうかは保証されていません。代わりにキーワード`CLUSTERED` / `NONCLUSTERED`の別のセットが使用され、**主キー**が**クラスターイングを含む**かどうかを制御します。キーワードが指定されていない場合、システム変数`@@global.tidb_enable_clustered_index`によって制御されます。これについては[クラスターインデックス](https://docs.pingcap.com/zh/tidb/stable/clustered-indexes)で説明されています。

**主キー**は`CREATE TABLE`ステートメント内で定義されます。[主キー制約](/constraints.md#primary-key)には、すべての制約された列が**NULLでない**値を含むことが必要です。

テーブルは**主キー**なしで作成することも、非整数の**主キー**を持つように作成することもできます。この場合、TiDBは`_tidb_rowid`を**暗黙の主キー**として作成します。**暗黙の主キー**`_tidb_rowid`は、単調増加する性質により、書き込み集中型のシナリオでホットスポットを引き起こす可能性があります。したがって、アプリケーションが書き込み中心である場合は、[`SHARD_ROW_ID_BITS`](/shard-row-id-bits.md)および[`PRE_SPLIT_REGIONS`](/sql-statements/sql-statement-split-region.md#pre_split_regions)パラメータを使用してデータをシャーディングすることを検討してください。ただし、これにより読み取りの増幅が発生する可能性がありますので、独自のトレードオフを行う必要があります。

テーブルの**主キー**が[整数型](/data-type-numeric.md#integer-types)であり、`AUTO_INCREMENT`が使用されている場合、`SHARD_ROW_ID_BITS`を使用してホットスポットを回避することはできません。ホットスポットを回避し、連続的かつ増分の主キーが必要でない場合は、`AUTO_INCREMENT`の代わりに[`AUTO_RANDOM`](/auto-random.md)を使用して行IDの連続性を排除することができます。

<CustomContent platform="tidb">

ホットスポット問題の対処方法の詳細については、[ホットスポットの問題のトラブルシューティング](/troubleshoot-hot-spot-issues.md)を参照してください。

</CustomContent>

**主キー**の選択に関する[ガイドラインに従うためのガイドライン](#guidelines-to-follow-when-selecting-primary-key)に従うと、以下の例では、`users`テーブルで`AUTO_RANDOM`主キーが定義されています。

```sql
CREATE TABLE `bookshop`.`users` (
  `id` bigint AUTO_RANDOM,
  `balance` decimal(15,2),
  `nickname` varchar(100),
  PRIMARY KEY (`id`)
);
```
## クラスタリングされているかどうか

TiDBはv5.0以降、[クラスタ化インデックス](/clustered-indexes.md)機能をサポートしています。この機能により、プライマリキーを含むテーブルのデータの格納方法を制御できます。TiDBには特定のクエリのパフォーマンスを向上させる方法でテーブルを整理する能力が提供されます。

この文脈での「クラスタ化」の用語は、データの格納方法を指し、一緒に作業するデータベースサーバーのグループではありません。一部のデータベース管理システムでは、クラスタ化されたインデックスをインデックスで組織化されたテーブル（IOT）として参照します。

TiDBの現在のテーブル**内のプライマリキー**は、次の2つのカテゴリに分かれています。

- `非クラスタ化`: テーブルのプライマリキーは非クラスタ化されたインデックスです。非クラスタ化されたインデックスを持つテーブルでは、行データのキーはTiDBによって暗黙的に割り当てられた内部の`_tidb_rowid`から構成されます。プライマリキーは本質的にユニークなインデックスであるため、非クラスタ化されたインデックスを持つ表には、行を格納するために少なくとも2つのキー値ペアが必要です。
    - `_tidb_rowid`（キー） - 行データ（値）
    - プライマリキーデータ（キー） - `_tidb_rowid`（値）
- `クラスタ化`: テーブルのプライマリキーはクラスタ化されたインデックスです。クラスタ化されたインデックスを持つテーブルでは、行データのキーはユーザーによって与えられたプライマリキーデータから構成されます。したがって、クラスタ化されたインデックスを持つ表には、行を格納するために1つのキー値ペアのみが必要です。
    - プライマリキーデータ（キー） - 行データ（値）

[プライマリキーの選択](#select-primary-key)で説明されているように、TiDBでは`CLUSTERED`および`NONCLUSTERED`というキーワードを使用して**クラスタ化されたインデックス**を制御します。

> **注記:**
>
> TiDBはクラスタ化をテーブルの`PRIMARY KEY`のみでサポートしています。クラスタ化されたインデックスが有効になっている場合、`PRIMARY KEY`および`クラスタ化されたインデックス`という用語が同義に使用される場合があります。`PRIMARY KEY`は制約（論理的なプロパティ）を指し、クラスタ化されたインデックスはデータの格納方法を物理的に記述します。

[クラスタ化されたインデックスの選択のガイドライン](#guidelines-to-follow-when-selecting-clustered-index)に従って、次の例では、`books`と`users`の間の関連付けを表す`ratings`の表を作成し、`book_id`と`user_id`を使用して複合プライマリキーを構築し、その**プライマリキー**に対して**クラスタ化されたインデックス**を作成します。

```sql
CREATE TABLE `bookshop`.`ratings` (
  `book_id` bigint,
  `user_id` bigint,
  `score` tinyint,
  `rated_at` datetime,
  PRIMARY KEY (`book_id`,`user_id`) CLUSTERED
);
```

## カラム制約の追加

[プライマリキー制約](#select-primary-key)に加えて、TiDBは[NOT NULL](/constraints.md#not-null)制約、[UNIQUE KEY](/constraints.md#unique-key)制約、および`DEFAULT`などの他の**カラム制約**をサポートしています。完全な制約については、[TiDBの制約](/constraints.md)ドキュメントを参照してください。

### デフォルト値の設定

列にデフォルト値を設定するには、`DEFAULT`制約を使用します。デフォルト値を使用すると、各列の値を指定せずにデータを挿入できます。

`DEFAULT`を[supported SQL functions](/functions-and-operators/functions-and-operators-overview.md)と併用して使用することで、デフォルトの計算をアプリケーションレイヤーから外に移動し、アプリケーションレイヤーのリソースを節約できます。計算によって消費されるリソースは消失せず、TiDB クラスタに移動します。通常、デフォルトの時間でデータを挿入できます。次の例は、`ratings`表でデフォルト値を設定する方法を示しています。

```sql
CREATE TABLE `bookshop`.`ratings` (
  `book_id` bigint,
  `user_id` bigint,
  `score` tinyint,
  `rated_at` datetime DEFAULT NOW(),
  PRIMARY KEY (`book_id`,`user_id`) CLUSTERED
);
```

クラスターに**TiFlash**ノードが含まれていない場合、このSQLステートメントはエラーを報告します: `1105 - the tiflash replica count: 1 should be less than the total tiflash server count: 0`。[Build a TiDB Serverless Cluster](/develop/dev-guide-build-cluster-in-cloud.md#step-1-create-a-tidb-serverless-cluster)を使用して、**TiFlash**を含むTiDB Serverlessクラスターを作成できます。

次に、以下のクエリを実行できます:

```sql
SELECT HOUR(`rated_at`), AVG(`score`) FROM `bookshop`.`ratings` GROUP BY HOUR(`rated_at`);
```

また、[`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md)ステートメントを実行して、このステートメントが**TiFlash**を使用しているかどうかを確認できます:

```sql
EXPLAIN ANALYZE SELECT HOUR(`rated_at`), AVG(`score`) FROM `bookshop`.`ratings` GROUP BY HOUR(`rated_at`);
```

実行結果:

```sql
+-----------------------------+-----------+---------+--------------+---------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------+----------+------+
| id                          | estRows   | actRows | task         | access object | execution info                                                                                                                                                                                                                                                                                                                                                       | operator info                                                                                                                                  | memory   | disk |
+-----------------------------+-----------+---------+--------------+---------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------+----------+------+
| Projection_4                | 299821.99 | 24      | root         |               | time:60.8ms, loops:6, Concurrency:5                                                                                                                                                                                                                                                                                                                                  | hour(cast(bookshop.ratings.rated_at, time))->Column#6, Column#5                                                                                | 17.7 KB  | N/A  |
| └─HashAgg_5                 | 299821.99 | 24      | root         |               | time:60.7ms, loops:6, partial_worker:{wall_time:60.660079ms, concurrency:5, task_num:293, tot_wait:262.536669ms, tot_exec:40.171833ms, tot_time:302.827753ms, max:60.636886ms, p95:60.636886ms}, final_worker:{wall_time:60.701437ms, concurrency:5, task_num:25, tot_wait:303.114278ms, tot_exec:176.564µs, tot_time:303.297475ms, max:60.69326ms, p95:60.69326ms}  | group by:Column#10, funcs:avg(Column#8)->Column#5, funcs:firstrow(Column#9)->bookshop.ratings.rated_at                                         | 714.0 KB | N/A  |
|   └─Projection_15           | 300000.00 | 300000  | root         |               | time:58.5ms, loops:294, Concurrency:5                                                                                                                                                                                                                                                                                                                                | cast(bookshop.ratings.score, decimal(8,4) BINARY)->Column#8, bookshop.ratings.rated_at, hour(cast(bookshop.ratings.rated_at, time))->Column#10 | 366.2 KB | N/A  |
|     └─TableReader_10        | 300000.00 | 300000  | root         |               | time:43.5ms, loops:294, cop_task: {num: 1, max: 43.1ms, proc_keys: 0, rpc_num: 1, rpc_time: 43ms, copr_cache_hit_ratio: 0.00}                                                                                                                                                                                                                                        | data:TableFullScan_9                                                                                                                           | 4.58 MB  | N/A  |
|       └─TableFullScan_9     | 300000.00 | 300000  | cop[tiflash] | table:ratings | tiflash_task:{time:5.98ms, loops:8, threads:1}, tiflash_scan:{dtfile:{total_scanned_packs:45, total_skipped_packs:1, total_scanned_rows:368640, total_skipped_rows:8192, total_rs_index_load_time: 1ms, total_read_time: 1ms},total_create_snapshot_time:1ms}                                                                                                        | keep order:false                                                                                                                               | N/A      | N/A  |
+-----------------------------+-----------+---------+--------------+---------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------+----------+------+
```

`cop[tiflash]`のフィールドが現れる場合、そのタスクが**TiFlash**に送信されて処理されていることを意味します。

## `CREATE TABLE`ステートメントを実行する

上記のルールに従ってすべてのテーブルを作成した後、[データベース初期化](/develop/dev-guide-bookshop-schema-design.md#database-initialization-script-dbinitsql)スクリプトは次のようになるはずです。詳細な表情報を確認する必要がある場合は、[テーブルの説明](/develop/dev-guide-bookshop-schema-design.md#description-of-the-tables)を参照してください。

データベース初期化スクリプトを`init.sql`という名前で保存し、次のステートメントを実行してデータベースを初期化できます。

```shell
mysql
    -u root \
    -h {host} \
    -P {port} \
    -p {password} \
    < init.sql
```

`bookshop`データベースのすべてのテーブルを表示するには、[`SHOW TABLES`](/sql-statements/sql-statement-show-tables.md#show-full-tables)ステートメントを使用します。

```sql
SHOW TABLES IN `bookshop`;
```

実行結果:

```
+--------------------+
| Tables_in_bookshop |
+--------------------+
| authors            |
| book_authors       |
| books              |
| orders             |
| ratings            |
| users              |
+--------------------+
```

## テーブルを作成する際のガイドライン

このセクションでは、テーブルを作成する際に必要なガイドラインを提供します。

### テーブルの命名に従うべきガイドライン

- **完全修飾**されたテーブル名を使用します（たとえば、`CREATE TABLE {database_name}.{table_name}`）。データベース名を指定しない場合、TiDBは**SQLセッション**での現在のデータベースを使用します。**SQLセッション**で`USE {databasename};`を使用してデータベースを指定しないと、TiDBはエラーを返します。
- 意味のあるテーブル名を使用します。たとえば、ユーザーテーブルを作成する場合、以下のような名前を使用できます: `user`, `t_user`, `users`。または、会社や組織の命名規則に従うことができます。会社や組織に命名規則がない場合は、[テーブルの命名規則](/develop/dev-guide-object-naming-guidelines.md#table-naming-convention)を参照してください。`t1`、`table1`などのテーブル名は使用しないでください。
- 複数の単語はアンダースコアで区切り、名前は32文字を超えないことが推奨されます。
- 異なるビジネスモジュールのために別々の`DATABASE`を作成し、適切なコメントを追加します。

### 列を定義する際のガイドライン

- 列でサポートされている[データ型](/data-type-overview.md)を確認し、データタイプの制限に従ってデータを整理します。列に格納するデータに適切なタイプを選択してください。
- [主キーを選択する際のガイドライン](#guidelines-to-follow-when-selecting-primary-key)を確認し、主キー列を使用するかどうかを決定してください。
- [クラスター化インデックスを選択する際のガイドライン](#guidelines-to-follow-when-selecting-clustered-index)を確認し、**クラスター化インデックス**を指定するかどうかを決定してください。
- [列に制約を追加](#add-column-constraints)するかどうかを決定してください。
- 意味のある列名を使用してください。会社や組織のテーブル命名規則に従うことが推奨されます。会社や組織に対応する命名規則がない場合は、[列の命名規則](/develop/dev-guide-object-naming-guidelines.md#column-naming-convention)を参照してください。

### 主キーを選択する際のガイドライン

- テーブル内で**主キー**または**一意なインデックス**を定義します。
- 意味のある**列**を**主キー**として選択するようにしてください。
- パフォーマンス上の理由から、余分な広いテーブルを保存しないようにしてください。テーブルのフィールド数が`60`を超えることや、1つの行の合計データサイズが`64K`を超えることは推奨されません。データ長が長いフィールドは別のテーブルに分割することをお勧めします。
- 複雑なデータ型の使用は推奨されません。
- 結合するフィールドについて、データ型が一貫していることを確認し、暗黙的な変換を避けてください。
- 単調なデータ列に**主キー**を定義することは推奨されません。単調なデータ列（たとえば、`AUTO_INCREMENT`属性を持つ列）を使用して**主キー**を定義すると、書き込みパフォーマンスに影響を与える可能性があります。可能な限り、`AUTO_RANDOM`を使用することをお勧めします。これは、主キーの連続性と増加性属性を破棄します。
- 書き込みが集中するシナリオで単調なデータ列にインデックスを作成する場合、その単調なデータ列を**主キー**として定義する代わりに、`AUTO_RANDOM`を使用してそのテーブルの**主キー**を作成するか、[`SHARD_ROW_ID_BITS`](/shard-row-id-bits.md)と[`PRE_SPLIT_REGIONS`](/sql-statements/sql-statement-split-region.md#pre_split_regions)を使用して`_tidb_rowid`を分割することを検討してください。

### クラスター化インデックスを選択する際のガイドライン
- **クラスター化されたインデックス**を構築するには、[プライマリキーの選択ガイドライン](#guidelines-to-follow-when-selecting-primary-key)に従ってください。
- 非クラスター化されたインデックスを持つテーブルと比較して、クラスター化されたインデックスを持つテーブルは以下のシナリオでより高いパフォーマンスとスループットの利点を提供します：
    - データが挿入されると、クラスター化されたインデックスによりネットワークからのインデックスデータの書き込みが1回削減されます。
    - 同等の条件を持つクエリがプライマリキーのみを対象とする場合、クラスター化されたインデックスによりネットワークからのインデックスデータの読み込みが1回削減されます。
    - 範囲条件を持つクエリがプライマリキーのみを対象とする場合、クラスター化されたインデックスによりネットワークからのインデックスデータの複数読み取りが削減されます。
    - 同等または範囲の条件を持つクエリがプライマリキーの接頭辞のみを対象とする場合、クラスター化されたインデックスによりネットワークからのインデックスデータの複数読み取りが削減されます。
- 一方で、クラスター化されたインデックスを持つテーブルには以下の問題が発生する可能性があります：
    - 近い値を持つ大量のプライマリキーを挿入する場合、書き込みのホットスポットの問題が発生する可能性があります。[プライマリキーの選択ガイドライン](#guidelines-to-follow-when-selecting-primary-key)に従ってください。
    - プライマリキーのデータ型が64ビットより大きい場合、特に複数の2次インデックスがある場合、テーブルデータがより大きなストレージ容量を占有します。

- [クラスター化されたインデックスを使用するデフォルトの動作を制御](/clustered-indexes.md#create-a-table-with-clustered-indexes)するには、`@@global.tidb_enable_clustered_index` システム変数と `alter-primary-key` 設定の代わりに、クラスター化されたインデックスの使用を明示的に指定することができます。

### `CREATE TABLE` ステートメントを実行する際のガイドライン

- データベーススキーマの変更を行う際には、クライアント側のドライバーやORMを使用することは推奨されません。[MySQLクライアント](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)を使用するか、GUIクライアントを使用してデータベーススキーマの変更を行うことを推奨します。このドキュメントでは、ほとんどのシナリオでSQLファイルを渡してデータベーススキーマの変更を行う際には、**MySQLクライアント**が使用されます。
- SQL開発の[テーブルの作成と削除の仕様](/develop/dev-guide-sql-development-specification.md#create-and-delete-tables)に従ってください。作成および削除ステートメントをビジネスアプリケーション内にラップし、判断ロジックを追加することを推奨します。

## もう一つのステップ

本文書で作成されたすべてのテーブルには、2次インデックスが含まれていません。2次インデックスを追加するためのガイドについては、[2次インデックスの作成](/develop/dev-guide-create-secondary-indexes.md)を参照してください。