---
title: SQL FAQs
summary: TiDB SQLに関連するFAQについて学びます。
---

# SQL FAQs

この文書は、TiDBにおけるSQL操作に関連するFAQをまとめたものです。

## TiDBはセカンダリキーをサポートしていますか？

はい。ユニークな[セカンダリインデックス](/develop/dev-guide-create-secondary-indexes.md)を持つ非プライマリキーカラムに`NOT NULL`制約を付けることができます。この場合、そのカラムはセカンダリキーとして動作します。

## 大規模なテーブルへのDDL操作を実行する際のTiDBのパフォーマンスはどうですか？

大規模なテーブルに対するTiDBのDDL操作は通常問題ありません。TiDBはオンラインDDL操作をサポートし、これらのDDL操作はDML操作をブロックしません。

カラムの追加や削除、インデックスの削除などの一部のDDL操作はTiDBが迅速に実行できます。

インデックスの追加などの重いDDL操作の一部に関しては、TiDBはデータを補填する必要があり、これには時間がかかる（テーブルのサイズに応じて異なり、追加のリソースを消費します）場合があります。オンライントラフィックへの影響はチューニング可能です。TiDBは複数のスレッドで補填を行うことができ、消費されるリソースは以下のシステム変数で設定できます:

- [`tidb_ddl_reorg_worker_cnt`](/system-variables.md#tidb_ddl_reorg_worker_cnt)
- [`tidb_ddl_reorg_priority`](/system-variables.md#tidb_ddl_reorg_priority)
- [`tidb_ddl_error_count_limit`](/system-variables.md#tidb_ddl_error_count_limit)
- [`tidb_ddl_reorg_batch_size`](/system-variables.md#tidb_ddl_reorg_batch_size)

## 適切なクエリプランを選択する方法は？ヒントを使用する必要がありますか？またはヒントを使用できますか？

TiDBにはコストベースの最適化機能が含まれています。ほとんどの場合、最適化機能は最適なクエリプランを自動的に選択します。もし最適化機能がうまく機能しない場合は、[最適化ヒント](/optimizer-hints.md)を使用して最適化機能に介入することができます。

さらに、[SQLバインディング](/sql-plan-management.md#sql-binding)を使用して特定のSQLステートメントのクエリプランを固定することもできます。

## 特定のSQLステートメントの実行を防止する方法は？

特定のステートメントの実行時間を小さな値（たとえば1ミリ秒）に制限するために、[`MAX_EXECUTION_TIME`](/optimizer-hints.md#max_execution_timen)ヒントを使用した[SQLバインディング](/sql-plan-management.md#sql-binding)を作成できます。このようにすることで、そのステートメントは閾値によって自動的に終了します。

たとえば、`SELECT * FROM t1, t2 WHERE t1.id = t2.id`の実行を防止する場合は、次のSQLバインディングを使用してステートメントの実行時間を1ミリ秒に制限することができます:

```sql
CREATE GLOBAL BINDING for
    SELECT * FROM t1, t2 WHERE t1.id = t2.id
USING
    SELECT /*+ MAX_EXECUTION_TIME(1) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

> **注意:**
>
> `MAX_EXECUTION_TIME`の精度はおよそ100ミリ秒です。TiDBがSQLステートメントを終了する前に、TiKVのタスクが開始される場合があります。そのような場合のTiKVのリソース消費を削減するために、[`tidb_enable_paging`](/system-variables.md#tidb_enable_paging-new-in-v540)を`ON`に設定することをお勧めします。

このSQLバインディングを削除すると、制限が解除されます。

```sql
DROP GLOBAL BINDING for
    SELECT * FROM t1, t2 WHERE t1.id = t2.id;
```

## TiDBがサポートするMySQL変数は何ですか？

[System Variables](/system-variables.md)を参照してください。

## `ORDER BY`が省略された場合、MySQLと結果の順序が異なる理由は？

これはバグではありません。デフォルトでのレコードの順序は一貫性を保証するものではありません。

MySQLにおける結果の順序が安定して見えるのは、クエリが単一のスレッドで実行されるためです。しかしながら、新しいバージョンにアップグレードする際にクエリプランが変更されることがあるため、結果が安定するとは限りません。結果の順序が望ましい場合は、常に`ORDER BY`を使用することをお勧めします。

次のような参照は[ISO/IEC 9075:1992, Database Language SQL- July 30, 1992](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt)で見つけることができます:

> `<order by clause>`が指定されていない場合、`<cursor specification>`によって指定されたテーブルはTであり、Tの行の順序は実装依存です。

次の2つのクエリの結果はともに正当なものと見なされます:
```sql
> select * from t;
+------+------+
| a    | b    |
+------+------+
|    1 |    1 |
|    2 |    2 |
+------+------+
2 rows in set (0.00 sec)
```

```sql
> select * from t; -- 結果の順序は保証されません
+------+------+
| a    | b    |
+------+------+
|    2 |    2 |
|    1 |    1 |
+------+------+
2 rows in set (0.00 sec)
```

`ORDER BY`で使用される列のリストがユニークでない場合、そのステートメントは非決定的と見なされます。次の例では、列`a`には重複する値があるため、`ORDER BY a, b`のみが確実に決定的と見なされます:
```sql
> select * from t order by a;
+------+------+
| a    | b    |
+------+------+
|    1 |    1 |
|    2 |    1 |
|    2 |    2 |
+------+------+
3 rows in set (0.00 sec)
```

次のステートメントでは、列`a`の順序は保証されていますが、列`b`の順序は保証されていません。
```sql
> select * from t order by a;
+------+------+
| a    | b    |
+------+------+
|    1 |    1 |
|    2 |    2 |
|    2 |    1 |
+------+------+
3 rows in set (0.00 sec)
```

TiDBでは、システム変数[`tidb_enable_ordered_result_mode`](/system-variables.md#tidb_enable_ordered_result_mode)を使用して最終的な出力結果を自動的にソートすることもできます。

## TiDBは`SELECT FOR UPDATE`をサポートしていますか？

はい。悲観的ロックを使用している場合（TiDB v3.0.8以降のデフォルト値）`SELECT FOR UPDATE`の実行はMySQLと同様に動作します。

楽観的ロックを使用している場合、`SELECT FOR UPDATE`はトランザクション開始時にデータをロックしませんが、トランザクションコミット時に競合を確認します。確認の結果、競合が発生すると、コミット中のトランザクションはロールバックされます。

詳細については、[SELECTの構文要素の説明](/sql-statements/sql-statement-select.md#description-of-the-syntax-elements)を参照してください。

## TiDBのコーデックはUTF-8文字列をmemcomparableに保証できますか？UTF-8をサポートする必要がある場合のコーディングの提案はありますか？

TiDBはデフォルトでUTF-8文字セットを使用し、現在はUTF-8のみをサポートしています。TiDBの文字列はmemcomparable形式を使用します。

## トランザクション内の最大ステートメント数は何ですか？

トランザクション内の最大ステートメント数はデフォルトで5000です。

楽観的トランザクションモードでは、トランザクションの再試行が有効になっている場合、デフォルトの上限は5000です。`stmt-count-limit`パラメータを使用してこの上限を調整することができます。

## TiDBで後から挿入されたデータの自動インクリメントIDが先に挿入されたデータよりも小さい理由は何ですか？

TiDBの自動インクリメントID機能は、自動的に増加してユニークであることが保証されているが、連続して割り当てられることが保証されているわけではありません。現在、TiDBはIDをバッチで割り当てています。複数のTiDBサーバーに同時にデータを挿入すると、割り当てられるIDは連続的ではありません。複数のスレッドが複数の`tidb-server`インスタンスにデータを同時に挿入する場合、後から挿入されたデータの自動インクリメントIDは小さくなる可能性があります。TiDBでは整数フィールドに`AUTO_INCREMENT`を指定することができますが、単一のテーブルに対しては1つの`AUTO_INCREMENT`フィールドしか許可されません。詳細については、[Auto-increment ID](/mysql-compatibility.md#auto-increment-id)と[the AUTO_INCREMENT attribute](/auto-increment.md)を参照してください。

## TiDBで`sql_mode`をどのように変更できますか？

TiDBは、[`sql_mode`](/system-variables.md#sql_mode)システム変数をSESSIONまたはGLOBALの基準で変更することができます。

- `GLOBAL`スコープの変数を変更すると、変更内容はクラスターの他のサーバーに伝播し、再起動しても`sql_mode`の値を各TiDBサーバーで変更する必要がありません。
- `SESSION`スコープの変数の変更は現在のクライアントセッションにのみ影響します。サーバーを再起動すると、変更内容は失われます。

## Sqoopを使用してデータをTiDBにバッチで書き込む際に`java.sql.BatchUpdateExecption:statement count 5001 exceeds the transaction limitation`エラーが発生する

Sqoopでは、`--batch`はそれぞれ100ステートメントをコミットし、デフォルトで1つのステートメントに100個のSQLステートメントが含まれます。そのため100 * 100 = 10000のSQLステートメントが含まれますが、これは単一のTiDBトランザクションで許可される最大ステートメント数である5000を超えています。

2 つの解決策：

`-Dsqoop.export.records.per.statement=10` オプションを次のように追加してください：

{{< copyable "shell-regular" >}}

```bash
sqoop export \
    -Dsqoop.export.records.per.statement=10 \
    --connect jdbc:mysql://mysql.example.com/sqoop \
    --username sqoop ${user} \
    --password ${passwd} \
    --table ${tab_name} \
    --export-dir ${dir} \
    --batch
```

また、単一の TiDB トランザクションにおけるステートメント数制限を増やすことも可能ですが、これにはより多くのメモリが必要となります。詳細については、[SQL ステートメントの制限](/tidb-limitations.md#limitations-on-sql-statements)を参照してください。

TiDB には Oracle の Flashback Query のような機能がありますか？DDL はサポートされていますか？

はい、あります。そして DDL もサポートされています。詳細については、[`AS OF TIMESTAMP` 句を使用した過去データの読み取り](/as-of-timestamp.md)を参照してください。

データを削除した後、TiDB はすぐにスペースを解放しますか？

`DELETE`、`TRUNCATE`、`DROP` のいずれも、データを直ちに解放しません。`TRUNCATE` および `DROP` の操作では、TiDB の GC（Garbage Collection）タイム（デフォルトでは 10 分）の後にデータが削除され、スペースが解放されます。`DELETE` の操作では、データは削除されますが、スペースはすぐに解放されず、コンパクションが実行されるまで解放されません。

データを削除した後、クエリのスピードが遅くなるのはなぜですか？

大量のデータを削除すると、多くの無駄なキーが残り、クエリの効率に影響を与えます。この問題を解決するためには、[リージョン マージ](/best-practices/massive-regions-best-practices.md#method-3-enable-region-merge)機能を使用できます。詳細については、TiDB Best Practices の[データの削除セクション](https://en.pingcap.com/blog/tidb-best-practice/#write)を参照してください。

データを削除した後、ストレージスペースの回収が遅くなった場合は、どうすればいいですか？

TiDB はマルチバージョン同時実行制御（MVCC）を使用しているため、古いデータが新しいデータで上書きされると、古いデータは置き換えられずに新しいデータと共に保持されます。タイムスタンプがデータバージョンを識別するために使用されます。データを削除しても、すぐにスペースは回収されません。並行トランザクションが行なわれるため古いバージョンの行を見ることができるようゴミの収集が遅れます。これは[`tidb_gc_life_time`](/system-variables.md#tidb_gc_life_time-new-in-v50)（デフォルト：`10m0s`）システム変数を介して構成できます。

`SHOW PROCESSLIST` にはシステムプロセス ID が表示されますか？

TiDB の `SHOW PROCESSLIST` の表示内容は、ほぼ MySQL の `SHOW PROCESSLIST` と同じです。TiDB の `SHOW PROCESSLIST` はシステムプロセス ID を表示しません。それが表示するのは、現在のセッション ID です。TiDB の `SHOW PROCESSLIST` と MySQL の `SHOW PROCESSLIST` の違いは次のようになります。

- TiDB は分散データベースであるため、`tidb-server` インスタンスは SQL ステートメントを解析および実行するためのステートレス エンジンです（詳細については、[TiDB アーキテクチャ](/tidb-architecture.md)を参照してください）。`SHOW PROCESSLIST` は、MySQL クライアントから `tidb-server` インスタンスにログインしたユーザが実行したセッションリストを表示し、クラスタで実行されているすべてのセッションのリストではないことに注意してください。一方、MySQL はスタンドアロン データベースであり、その `SHOW PROCESSLIST` は MySQL で実行されたすべての SQL ステートメントを表示します。
- TiDB では、クエリの実行中、`State` 列が連続して更新されません。TiDB は並列クエリをサポートしているため、各ステートメントは一度に複数の _状態_ になる可能性があり、単一の値に単純化することが難しいためです。

SQL コミットの実行優先度を制御したり変更したりするにはどうすればいいですか？

TiDB は[グローバル](/system-variables.md#tidb_force_priority)または個々のステートメントにおける優先度の変更をサポートしています。優先度は以下のような意味を持ちます。

- `HIGH_PRIORITY` : このステートメントは高い優先度を持ち、つまり、TiDB はこのステートメントを優先して最初に実行します。
- `LOW_PRIORITY` : このステートメントは低い優先度を持ち、つまり、TiDB はこのステートメントの実行中に優先度を下げます。
- `DELAYED` : このステートメントは通常の優先度を持ち、`tidb_force_priority` の `NO_PRIORITY` 設定と同じです。

上記の 2 つのパラメータを DML of TiDB と組み合わせて使用することができます。例えば：

1. データベース内に SQL ステートメントを書いて優先度を調整します：

{{< copyable "sql" >}}

```sql
SELECT HIGH_PRIORITY | LOW_PRIORITY | DELAYED COUNT(*) FROM table_name;
INSERT HIGH_PRIORITY | LOW_PRIORITY | DELAYED INTO table_name insert_values;
DELETE HIGH_PRIORITY | LOW_PRIORITY | DELAYED FROM table_name;
UPDATE HIGH_PRIORITY | LOW_PRIORITY | DELAYED table_reference SET assignment_list WHERE where_condition;
REPLACE HIGH_PRIORITY | LOW_PRIORITY | DELAYED INTO table_name;
```

2. フルテーブルスキャン ステートメントは自動的に低優先度に調整されます。[`ANALYZE`](/sql-statements/sql-statement-analyze-table.md) は、デフォルトで低優先度です。

`auto analyze` のトリガー戦略はどのようになっていますか？

トリガー戦略：新しいテーブル内の行数が 1000 に達し、このテーブルに 1 分以内に書き込み操作がない場合、`auto analyze` が自動的にトリガーされます。

変更された行数 / 現在の総行数 の比率が `tidb_auto_analyze_ratio` よりも大きい場合、`analyze` ステートメントが自動的にトリガーされます。`tidb_auto_analyze_ratio` のデフォルト値は 0.5 で、この機能がデフォルトで有効になっていることを示しています。安全性を確保するため、この機能が有効の場合、その最小値は 0.3 であり、`pseudo-estimate-ratio` のデフォルト値である 0.8 よりも小さくなければ疑似統計情報が一時的に使用されます。`tidb_auto_analyze_ratio` は 0.5 に設定することを推奨します。

自動分析を無効にするには、`tidb_enable_auto_analyze` システム変数を使用してください。

オプティマイザのヒントを使用して、オプティマイザの動作を上書きすることはできますか？

TiDB は、[ヒント](/optimizer-hints.md)および[SQL プラン管理](/sql-plan-management.md)を含む、デフォルトのクエリオプティマイザの動作を上書きする複数の方法をサポートしています。基本的な使用法は MySQL と類似していますが、いくつかの TiDB 固有の拡張があります：

```sql
SELECT column_name FROM table_name USE INDEX（index_name）WHERE where_condition;
```

DDL 実行

このセクションは、DDL ステートメントの実行に関連する問題をリストしています。DDL 実行原則の詳細な説明については、[DDL ステートメントの実行原則とベストプラクティス](/ddl-introduction.md)を参照してください。

各種 DDL 操作の実行にはどのくらい時間がかかりますか？

DDL 操作がブロックされていないと仮定し、各 TiDB サーバがスキーマ バージョンを通常に更新し、DDL Owner ノードが正常に実行されている場合、各種 DDL 操作の見積もられる時間は次のとおりです。

| DDL 操作の種類 | 見積もられる時間 |
|:----------|:-----------|
| `ADD INDEX`、`MODIFY COLUMN` などの Reorg DDL（再構成タイプのデータ変更） | データ量、システム負荷、および DDL パラメータ設定によって異なります。 |
| 一般 DDL（Reorg 以外の DDL タイプ）、`CREATE DATABASE`、`CREATE TABLE`、`DROP DATABASE`、`DROP TABLE`、`TRUNCATE TABLE`、`ALTER TABLE ADD`、`ALTER TABLE DROP`、`MODIFY COLUMN`（メタデータのみの変更）、`DROP INDEX` | 約 1 秒 |

> **注意:**
>
> 上記は操作の見積もられる時間です。実際の時間は異なる可能性があります。

DDL 実行が遅い可能性のある理由

- ユーザセッション内で、DDL ステートメントの前に自動コミットでない DML ステートメントがある場合、および非自動コミット DML ステートメントのコミット操作が遅い場合、DDL ステートメントの実行が遅くなる可能性があります。つまり、TiDB は DDL ステートメントを実行する前にコミットされていない DML ステートメントをコミットします。
- 複数の DDL ステートメントが一緒に実行された場合、後の DDL ステートメントの実行が遅くなる場合があります。これは、キューイングの場合に発生します。キューイングのシナリオには次のようなものがあります。
    - 同じ種類の DDL ステートメントをキューに入れる必要がある場合。例えば、`CREATE TABLE` および `CREATE DATABASE` の両方が一般的な DDL ステートメントであるため、両方の操作が同時に実行された場合、それらをキューに入れる必要があります。TiDB v6.2.0 からは、並列 DDL ステートメントがサポートされていますが、DDL の実行が TiDB のコンピューティング リソースを取りすぎないようにするため、並列度制限があります。DDL が並列度制限を超えるとキューが発生します。
    - 同じテーブルに対して実行された DDL 操作間には依存関係がある。後の DDL ステートメントは、前の DDL 操作が完了するのを待つ必要があります。
- クラスタが通常に開始された後、最初の DDL 操作の実行時間が比較的長くなる場合があります。これは、DDL モジュールが DDL Owner を選出しているためです。
- TiDB が終了してしまうことで、PD と正常に通信できなくなった場合（電源が切れた場合を含む）。または、`kill -9` コマンドによって TiDB が終了してしまい、PD からの登録データのクリアがタイムリーに行われなくなった場合。
- クラスタ内の特定の TiDB ノードと PD または TiKV の間で通信問題が発生し、TiDB がタイムリーに最新のバージョン情報を取得できない場合。

`Information schema is changed` エラーがトリガーされる原因は何ですか？
SQLステートメントを実行する際、TiDBは隔離レベルに基づいてオブジェクトのスキーマバージョンを決定し、SQLステートメントを適切に処理します。TiDBはオンライン非同期DDL変更もサポートしています。DMLステートメントを実行する際、同時に実行中のDDLステートメントがある可能性があり、各SQLステートメントが同じスキーマ上で実行されることを確認する必要があります。したがって、DMLを実行する際、DDL操作が進行中の場合、TiDBは`Information schema is changed`エラーを報告する可能性があります。

v6.4.0からTiDBに[メタデータロックメカニズム](/metadata-lock.md)が実装されており、DMLステートメントとDDLスキーマ変更の協調実行を可能にし、`Information schema is changed`エラーを回避します。

現在も、次のようないくつかの原因によりこのエラーが報告される可能性があります:

+ 原因1: DML操作に関与する一部のテーブルが、実行中のDDL操作に関与する同じテーブルである。実行中のDDL操作を確認するには、`ADMIN SHOW DDL`ステートメントを使用します。
+ 原因2: DML操作が長時間続く。この期間中に多くのDDLステートメントが実行され、1024以上の`schema`バージョン変更が発生する。このデフォルト値は`tidb_max_delta_schema_count`変数の変更によって変更できます。
+ 原因3: DMLリクエストを受け入れるTiDBサーバーが、長時間`スキーマ情報`を読み込めない（TiDBとPDまたはTiKVの接続障害による可能性があります）。この期間中に多数のDDLステートメントが実行され、100以上の`schema`バージョン変更が発生します。
+ 原因4: TiDBが再起動され、最初のDDL操作が実行される前にDML操作が実行され、その後最初のDDL操作に遭遇します（つまり、最初のDDL操作が実行される前に、DMLに対応するトランザクションが開始され、最初の`schema`バージョンのDDLが変更された後、DMLに対応するトランザクションがコミットされる）。この場合、DML操作はこのエラーを報告します。

前述の原因のうち、原因1のみがテーブルに関連しています。原因1および原因2はアプリケーションに影響を与えませんが、関連するDML操作は失敗した後に再試行されます。原因3では、TiDBとTiKV/PD間のネットワークを確認する必要があります。

> **注意:**
>
> + 現時点では、TiDBはすべての`schema`バージョン変更をキャッシュしていません。
> + 各DDL操作に対して、`schema`バージョン変更の数は、対応する`schema state`バージョン変更の数と同じです。
> + 異なるDDL操作は異なる数の`schema`バージョン変更を引き起こします。たとえば、`CREATE TABLE`ステートメントは1つの`schema`バージョン変更を引き起こし、`ADD COLUMN`ステートメントは4つのバージョン変更を引き起こします。

### "Information schema is out of date"エラーの原因は何ですか？

TiDB v6.5.0以前では、DMLステートメントを実行する際に、TiDBがDDLリース（デフォルトで45秒）の最新のスキーマをロードできない場合、`Information schema is out of date`エラーが発生する可能性があります。考えられる原因は次のとおりです:

- このDMLを実行したTiDBインスタンスが終了され、このDMLステートメントに対応するトランザクションの実行がDDLリースよりも長い時間かかった。トランザクションがコミットされた際にエラーが発生しました。
- TiDBがこのDMLステートメントの実行中にPDまたはTiKVに接続できなかった。その結果、TiDBはDDLリース内でスキーマをロードできず、またはPDとの接続がkeepalive設定によって切断されました。

### 高並行性下でDDLステートメントの実行時にエラーが報告されるのはなぜですか？

高い並行性下でDDLステートメント（たとえば、テーブルをバッチで作成する）を実行する場合、これらのステートメントのごく一部が同時実行中にキーの競合によって失敗する可能性があります。

並行して実行されるDDLステートメントの数を20未満に保つことをお勧めします。それ以外の場合、クライアントから失敗したステートメントを再試行する必要があります。

### DDL実行がブロックされるのはなぜですか？

TiDB v6.2.0以前では、TiDBはDDLステートメントのタイプに基づいて、再構成DDLを再構成キューに、一般DDLを一般キューに優先順位付けして割り当てていました。具体的には、Reorg DDLがReorgキューに入り、一般DDLが一般キューに入ります。FIFOの制限と同じテーブル上でのDDLステートメントの直列実行の必要性により、複数のDDLステートメントが実行中にブロックされる可能性があります。

たとえば、次のDDLステートメントを考えてみます:

- DDL 1: `CREATE INDEX idx on t(a int);`
- DDL 2: `ALTER TABLE t ADD COLUMN b int;`
- DDL 3: `CREATE TABLE t1(a int);`

FIFOキューの制限により、DDL 3はDDL 2の実行を待たなければなりません。また、同じテーブルのDDLステートメントは直列で実行する必要があるため、DDL 2はDDL 1の実行を待たなければなりません。したがって、DDL 3は別のテーブル上で動作しているにもかかわらず、DDL 1の実行をまず待たなければなりません。

TiDB v6.2.0からは、TiDBのDDLモジュールは並行フレームワークを使用しています。並行フレームワークでは、FIFOキューの制限は効力を持ちません。代わりに、TiDBはすべてのDDLタスクから実行可能なDDLタスクを選択します。また、Reorgワーカーの数は、ノードあたりおおよそ`CPU/4`に拡張されています。これにより、TiDBは並行フレームワークで複数のテーブルに対して同時にインデックスを構築することができます。

クラスタが新しいクラスタであるか、以前のバージョンからのアップグレードであるかにかかわらず、TiDBはTiDB v6.2以降のバージョンで自動的に並行フレームワークを使用します。手動での調整は必要ありません。

### スタックしたDDL実行の原因を特定する

1. DDLステートメントの実行を遅くするその他の原因を排除します。
2. 次のいずれかの方法を使用して、DDLオーナーノードを特定します:
    - `curl http://{TiDBIP}:10080/info/all`を使用して、現在のクラスタのオーナーを取得します。
    - 監視ダッシュボードの**DDL** > **DDL META OPM**から、特定の期間中のオーナーを表示します。

- オーナーが存在しない場合、`curl -X POST http://{TiDBIP}:10080/ddl/owner/resign`でオーナー選出を手動でトリガーします。
- オーナーが存在する場合は、Goroutineスタックをエクスポートし、スタックした場所を確認します。

## SQL最適化

### TiDB実行計画の説明

[クエリ実行計画の理解](/explain-overview.md)を参照してください。

### 統計情報収集

[統計情報の紹介](/statistics.md)を参照してください。

### `select count(1)`を最適化する方法は？

`count(1)`ステートメントはテーブル内の総行数を数えます。並列度を改善することで、処理速度を大幅に向上させることができます。並行性を改善するためには、[`tidb_distsql_scan_concurrency`ドキュメント](/system-variables.md#tidb_distsql_scan_concurrency)を参照してください。ただし、これはCPUとI/Oリソースに依存します。TiDBはクエリごとにTiKVにアクセスします。データが少量であればすべてのMySQLはメモリ内にあり、TiDBはネットワークアクセスを実行する必要があります。

お勧め:

- ハードウェア構成を改善してください。[ソフトウェアおよびハードウェア要件](/hardware-and-software-requirements.md)を参照してください。
- 並行性を改善してください。デフォルト値は10です。これを50に改善し、試してみてください。ただし、デフォルト値の2-4倍の改善が一般的です。
- 大量のデータの場合の`count`をテストしてください。
- TiKVの構成を最適化してください。[TiKVスレッドパフォーマンスの調整](/tune-tikv-thread-performance.md)および[TiKVメモリパフォーマンスの調整](/tune-tikv-memory-performance.md)を参照してください。
- [Coprocessor Cache](/coprocessor-cache.md)を有効にしてください。

### 現在のDDLジョブの進捗状況を表示する方法は？

`ADMIN SHOW DDL`を使用して、現在のDDLジョブの進捗状況を表示できます。操作手順は次のとおりです:

```sql
ADMIN SHOW DDL;
```

```
*************************** 1. row ***************************
  SCHEMA_VER: 140
       OWNER: 1a1c4174-0fcd-4ba0-add9-12d08c4077dc
RUNNING_JOBS: ID:121, Type:add index, State:running, SchemaState:write reorganization, SchemaID:1, TableID:118, RowCount:77312, ArgLen:0, start time: 2018-12-05 16:26:10.652 +0800 CST, Err:<nil>, ErrCount:0, SnapshotVersion:404749908941733890
     SELF_ID: 1a1c4174-0fcd-4ba0-add9-12d08c4077dc
```

上記の結果から、現在`ADD INDEX`操作が処理中であることが分かります。また、`RUNNING_JOBS`列の`RowCount`フィールドから、現在`ADD INDEX`操作がインデックスの77312行を追加したことが分かります。

### DDLジョブを表示する方法は？

- `ADMIN SHOW DDL`: 実行中のDDLジョブを表示するため
- `ADMIN SHOW DDL JOBS`: 現在のDDLジョブキューにあるすべての結果（実行中および実行を待っているタスクを含む）と、完了したDDLジョブキューの最後の10の結果を表示するため
- `ADMIN SHOW DDL JOBS QUERIES 'job_id' [, 'job_id'] ...`: 'job_id'に対応するDDLタスクの元のSQLステートメントを表示するには。 'job_id'は実行中のDDLジョブとDDL履歴ジョブキューの最後の10件の結果を検索します。

### TiDBはCBO（コストベース最適化）をサポートしていますか？ もしサポートしている場合、どの程度までですか？

はい。TiDBはコストベースの最適化を使用しています。コストモデルと統計は常に最適化されています。TiDBはhash joinやsort-merge joinなどの結合アルゴリズムもサポートしています。

### テーブルに`analyze`を実行する必要があるかどうかをどのように判断しますか？

`SHOW STATS_HEALTHY`を使用して`Healthy`フィールドを表示し、一般的にフィールド値が60未満の場合はテーブルに`ANALYZE`を実行する必要があります。

### クエリプランがツリーとして表示される際のIDのルールは何ですか？ このツリーの実行順序は何ですか？

これらのIDにはルールは存在しませんが、IDは一意です。IDが生成される際、カウンタが動作し、1つのプランが生成されるごとに1を追加します。実行順序はIDとは関係ありません。クエリプラン全体はツリーであり、実行プロセスは根ノードから開始し、データは連続して上位レベルに返されます。クエリプランの詳細については、[TiDBクエリ実行計画の理解](/explain-overview.md)を参照してください。

### TiDBのクエリプランで、`cop`タスクは同じルートにあります。それらは同時に実行されますか？

現在、TiDBの計算タスクは`cop task`と`root task`の2つの異なるタイプのタスクに属します。

`cop task`は、分散実行のためにKVエンドにプッシュダウンされる計算タスクであり、`root task`はTiDBエンドでの単一ポイント実行の計算タスクです。

一般的に`root task`の入力データは`cop task`から来ます；`root task`がデータを処理するとき、TiKVの`cop task`は同時にデータを処理し、TiDBの`root task`のプルを待ちます。したがって、`cop`タスクは`root task`と同時に実行されると見なすことができます。しかし、それらのデータには上流と下流の関係があります。実行中は一部の時間において同時に実行されます。例えば、最初の`cop task`がデータを[100、200]で処理し、2番目の`cop task`がデータを[1、100]で処理しているとします。詳細については、[TiDBクエリ計画の理解](/explain-overview.md)を参照してください。

## データベースの最適化

### TiDBオプションの編集

[TiDBコマンドオプション](/command-line-flags-for-tidb-configuration.md)を参照してください。

### ホットスポットの問題を回避し、負荷分散を実現する方法は？

ホットスポットを引き起こすシナリオについては、[一般的なホットスポット](/troubleshoot-hot-spot-issues.md#common-hotspots)を参照してください。以下のTiDB機能は、ホットスポットの問題解決を支援するために設計されています。

- [`SHARD_ROW_ID_BITS`](/troubleshoot-hot-spot-issues.md#use-shard_row_id_bits-to-process-hotspots)属性。この属性を設定すると、行IDが分散され、複数のリージョンに書き込まれるため、書き込みホットスポットの問題が緩和されます。
- [`AUTO_RANDOM`](/troubleshoot-hot-spot-issues.md#handle-auto-increment-primary-key-hotspot-tables-using-auto_random)属性は、オートインクリメントプライマリキーによって引き起こされるホットスポットを解決するのに役立ちます。
- [Coprocessor Cache](/coprocessor-cache.md)：小さな表の読み取りホットスポットに対応。
- [Load Base Split](/configure-load-base-split.md)：リージョン間の不均衡なアクセスによるホットスポットに対応し、小さな表の完全なスキャンなど。
- [Cached tables](/cached-tables.md)：頻繁にアクセスされるが滅多に更新されない小さなホットスポットテーブル。

ホットスポットによるパフォーマンスの問題がある場合は、[ホットスポットのトラブルシューティング](/troubleshoot-hot-spot-issues.md)を参照して解決方法を得てください。

### TiKVのパフォーマンスの調整

[Tune TiKV Thread Performance](/tune-tikv-thread-performance.md)および[Tune TiKV Memory Performance](/tune-tikv-memory-performance.md)を参照してください。