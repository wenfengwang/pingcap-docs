---
title: TiDB 5.0の新機能
---

# TiDB 5.0の新機能

リリース日: 2021年4月7日

TiDBバージョン: 5.0.0

v5.0では、PingCAPは企業がTiDBをベースとしたアプリケーションを迅速に構築し、データベースのパフォーマンス、パフォーマンスの揺らぎ、セキュリティ、高可用性、障害回復、SQLパフォーマンスのトラブルシューティングなどを気にする必要がないよう支援することを目指しています。

v5.0での主な新機能や改良点は次の通りです：

+ TiFlashノードを介してMassively Parallel Processing (MPP)アーキテクチャを導入し、大規模な結合クエリの実行ワークロードをTiFlashノード間で共有します。MPPモードが有効になると、コストに基づいてTiDBは計算を実行するためにMPPフレームワークを使用するかどうかを決定します。MPPモードでは、`Exchange`操作を介して結合キーが再配置され、計算中に計算圧力が各TiFlashノードに分散され、計算が高速化されます。ベンチマークによると、同じクラスターリソースを使用した場合、TiDB 5.0のMPPは、Greenplum 6.15.0およびApache Spark 3.1.1に比べて2～3倍のスピードアップを示し、一部のクエリは8倍の性能向上を達成しています。
+ データベースのパフォーマンスを向上させるためにクラスター化インデックス機能を導入します。例えば、TPC-C tpmCテストでは、クラスター化インデックスが有効になっている場合、TiDBのパフォーマンスが39%向上します。
+ 書き込み遅延を減少させるために非同期コミット機能を有効にします。例えば、64スレッドのSysbenchテストでは、非同期コミットが有効になっている場合、インデックスの更新の平均遅延が12.04 msから7.01 msに減少します。
+ 揺れを減らす。これは、最適化安定性の改善とI/O、ネットワーク、CPU、メモリリソースのシステムタスクの使用制限によって実現されます。例えば、8時間のパフォーマンステストでは、TPC-C tpmCの標準偏差が2％を超えません。
+ スケジューリングの改善と実行計画の安定性を可能な限り高めることで、システムの安定性を向上させます。
+ リージョンのメンバーシップ変更時にシステムの可用性を確保するためにRaft Joint Consensusアルゴリズムを導入します。
+ Database Administrators（DBA）がSQLステートメントをより効率的にデバッグできるように、`EXPLAIN`機能と不可視インデックスを最適化します。
+ 企業データの信頼性を保証します。TiDBからAmazon S3ストレージやGoogle Cloud GCSにデータをバックアップしたり、これらのクラウドストレージプラットフォームからデータを復元したりすることができます。
+ Amazon S3ストレージやTiDB/MySQLからのデータのインポートまたはエクスポートのパフォーマンスを向上させ、クラウド上でアプリケーションを迅速に構築します。例えば、TPC-Cテストでは、1 TiBのデータのインポートパフォーマンスが254 GiB/hから366 GiB/hに40%向上します。

## 互換性の変更

### システム変数

+ 複数のオペレータの並列実行を制御するための[`tidb_executor_concurrency`](/system-variables.md#tidb_executor_concurrency-new-in-v50)システム変数を追加します。以前の`tidb_*_concurrency`設定（例: `tidb_projection_concurrency`）は引き続き有効ですが、使用する際に警告が表示されます。
+ `tidb_skip_ascii_check`システム変数を追加して、ASCII文字セットが書き込まれた際にASCII検証チェックをスキップするかどうかを指定します。デフォルト値は`OFF`です。
+ `tidb_enable_strict_double_type_check`システム変数を追加して、`double(N)`などの構文がテーブルスキーマで定義できるかどうかを決定します。デフォルト値は`OFF`です。
+ [`tidb_dml_batch_size`](/system-variables.md#tidb_dml_batch_size)のデフォルト値を`20000`から`0`に変更します。これにより、`LOAD`や`INSERT INTO SELECT ...`でデフォルトでバッチDMLステートメントが使用されなくなり、代わりに厳密なACIDセマンティクスに準拠するために大規模トランザクションが使用されます。

    > **注意:**
    >
    > 変数のスコープがセッションからグローバルに変更され、デフォルト値が`20000`から`0`に変更されました。元のデフォルト値にアプリケーションが依存している場合は、アップグレード後に`set global`ステートメントを使用して変数を元の値に変更する必要があります。

+ [`tidb_enable_noop_functions`](/system-variables.md#tidb_enable_noop_functions-new-in-v40)システム変数を使用して一時テーブルの構文の互換性を制御します。この変数の値が`OFF`の場合、`CREATE TEMPORARY TABLE`構文はエラーを返します。
+ ガベージコレクション関連のパラメータを直接制御するために次のシステム変数を追加します：
    - [`tidb_gc_concurrency`](/system-variables.md#tidb_gc_concurrency-new-in-v50)
    - [`tidb_gc_enable`](/system-variables.md#tidb_gc_enable-new-in-v50)
    - [`tidb_gc_life_time`](/system-variables.md#tidb_gc_life_time-new-in-v50)
    - [`tidb_gc_run_interval`](/system-variables.md#tidb_gc_run_interval-new-in-v50)
    - [`tidb_gc_scan_lock_mode`](/system-variables.md#tidb_gc_scan_lock_mode-new-in-v50)
+ [`enable-joint-consensus`](/pd-configuration-file.md#enable-joint-consensus-new-in-v50)のデフォルト値を`false`から`true`に変更し、Joint Consensus機能をデフォルトで有効にします。
+ `tidb_enable_amend_pessimistic_txn`の値を`0`または`1`から`ON`または`OFF`に変更します。
+ [`tidb_enable_clustered_index`](/system-variables.md#tidb_enable_clustered_index-new-in-v50)のデフォルト値を`OFF`から`INT_ONLY`に変更し、以下の新しい意味を追加します：
    + `ON`: クラスター化インデックスが有効になり、非クラスター化インデックスの追加や削除がサポートされます。
    + `OFF`: クラスター化インデックスが無効になり、非クラスター化インデックスの追加や削除がサポートされます。
    + `INT_ONLY`: デフォルト値。挙動はv5.0以前と一貫しています。`alter-primary-key = false`と一緒にINT型のクラスター化インデックスを有効にするかどうかを制御できます。

    > **注意:**
    >
    > v5.0 GAの`tidb_enable_clustered_index`の`INT_ONLY`の値は、v5.0 RCの`OFF`の値と同じ意味を持っています。`OFF`の設定であるv5.0 RCクラスターからv5.0 GAにアップグレードすると、「INT_ONLY」として表示されます。

### 設定ファイルのパラメータ

+ TiDBのための[`index-limit`](/tidb-configuration-file.md#index-limit-new-in-v50)構成項目を追加します。その値はデフォルトで`64`であり、`[64,512]`の範囲です。MySQLテーブルは最大で64のインデックスをサポートします。このデフォルト設定を超えてテーブルに64以上のインデックスが作成された場合、テーブルスキーマがMySQLに再インポートされるとエラーが発生します。
+ TiDBのために[`enable-enum-length-limit`](/tidb-configuration-file.md#enable-enum-length-limit-new-in-v50)構成項目を追加して、MySQLのENUM/SET長さ（ENUM長さ < 255）と互換性を確保します。デフォルト値は`true`です。
+ `pessimistic-txn.enable`構成項目を[`tidb_txn_mode`](/system-variables.md#tidb_txn_mode)環境変数に置き換えます。
+ `performance.max-memory`構成項目を[`performance.server-memory-quota`](/tidb-configuration-file.md#server-memory-quota-new-in-v409)に置き換えます。
+ `tikv-client.copr-cache.enable`構成項目を[`tikv-client.copr-cache.capacity-mb`](/tidb-configuration-file.md#capacity-mb)に置き換えます。その値が`0.0`の場合、この機能は無効になります。その値が`0.0`より大きい場合、この機能が有効になります。デフォルト値は`1000.0`です。
+ `rocksdb.auto-tuned`構成項目を[`rocksdb.rate-limiter-auto-tuned`](/tikv-configuration-file.md#rate-limiter-auto-tuned-new-in-v50)に置き換えます。
+ `raftstore.sync-log`構成項目を削除します。書き込まれたデータは強制的にディスクにスパイルされます。v5.0以前では明示的に`raftstore.sync-log`を無効にできましたが、v5.0以降は構成値が強制的に`true`に設定されます。
+ `gc.enable-compaction-filter`構成項目のデフォルト値を`false`から`true`に変更します。
+ `enable-cross-table-merge`構成項目のデフォルト値を`false`から`true`に変更します。
+ [`rate-limiter-auto-tuned`](/tikv-configuration-file.md#rate-limiter-auto-tuned-new-in-v50)構成項目のデフォルト値を`false`から`true`に変更します。

### その他

+ アップグレード前にTiDB構成の[`feedback-probability`](https://docs.pingcap.com/tidb/v5.0/tidb-configuration-file#feedback-probability)の値を確認してください。値が0でない場合、アップグレード後に「recoverable goroutine内のパニック」エラーが発生しますが、このエラーはアップグレードに影響しません。
+ データの正確性の問題を避けるために、カラムタイプを変更する際の`VARCHAR`型と`CHAR`型の変換を禁止します。

## 新機能
### SQL

#### リストパーティショニング（**実験的**）

[ユーザードキュメント](/partitioned-table.md#list-partitioning)

リストパーティショニング機能を使用すると、大量のデータを持つテーブルを効果的にクエリおよびメンテナンスできます。

この機能を有効にすると、`PARTITION BY LIST(expr) PARTITION part_name VALUES IN (...)` 式に従って、パーティションおよびパーティション間でのデータの分散方法が定義されます。パーティションテーブルのデータセットは最大1024の異なる整数値をサポートします。`PARTITION ... VALUES IN (...)` 句を使用して値を定義できます。

リストパーティショニングを有効にするには、セッション変数[`tidb_enable_list_partition`](/system-variables.md#tidb_enable_list_partition-new-in-v50) を `ON` に設定します。

#### 列リストパーティショニング（**実験的**）

[ユーザードキュメント](/partitioned-table.md#list-columns-partitioning)

列リストパーティショニングはリストパーティショニングの変種です。複数の列をパーティションキーとして使用できます。整数データ型に加えて、文字列、`DATE`、`DATETIME`データ型の列をパーティション列として使用することもできます。

リスト列パーティショニングを有効にするには、セッション変数[`tidb_enable_list_partition`](/system-variables.md#tidb_enable_list_partition-new-in-v50) を `ON` に設定します。

#### 不可視インデックス

[ユーザードキュメント](/sql-statements/sql-statement-alter-index.md), [#9246](https://github.com/pingcap/tidb/issues/9246)

パフォーマンスを調整したり最適なインデックスを選択する場合、SQLステートメントを使用してインデックスを`Visible`または`Invisible`に設定できます。この設定により、`DROP INDEX`や`ADD INDEX`などのリソース消費量の多い操作を回避できます。

インデックスの可視性を変更するには、`ALTER INDEX` ステートメントを使用します。変更後に、オプティマイザはインデックスの可視性に基づいてこのインデックスをインデックスリストに追加するかどうかを決定します。

#### `EXCEPT` および `INTERSECT` 演算子

[ユーザードキュメント](/functions-and-operators/set-operators.md), [#18031](https://github.com/pingcap/tidb/issues/18031)

`INTERSECT` 演算子は複数のクエリの結果セットの積集合を返すセット演算子です。ある意味では、`Inner Join` 演算子の代替となります。

`EXCEPT` 演算子は、2つのクエリの結果セットを結合し、最初のクエリ結果には含まれているが2番目には含まれていない要素を返します。

### トランザクション

[#18005](https://github.com/pingcap/tidb/issues/18005)

悲観的なトランザクションモードでは、トランザクションに関与するテーブルに同時DDL操作または `SCHEMA VERSION` の変更が含まれている場合、システムはトランザクションの `SCHEMA VERSION` を最新のものに自動的に更新し、トランザクションのコミット成功を確保し、クライアントがDDL操作または `SCHEMA VERSION` の変更によってトランザクションが中断された際に `情報スキーマが変更されました` エラーを受け取らないようにします。

この機能はデフォルトで無効になっています。機能を有効にするには、`tidb_enable_amend_pessimistic_txn` システム変数の値を変更します。この機能はv4.0.7で導入され、v5.0で以下の問題が修正されました：

+ TiDB Binlogが`Add Column`操作を実行する際に発生する互換性の問題
+ この機能を一意のインデックスと一緒に使用した際に発生するデータの不整合の問題
+ この機能を追加されたインデックスと一緒に使用した際に発生するデータの不整合の問題

現在、この機能には以下の互換性の問題が存在します：

+ 同時トランザクションが存在する場合、トランザクションのセマンティクスが変わる可能性があります
+ この機能をTiDB Binlogと一緒に使用した際に発生する既知の互換性の問題
+ `Change Column` との互換性がありません

### 文字セットと照合順序

- `utf8mb4_unicode_ci` および `utf8_unicode_ci` 照合順序をサポート。[ユーザードキュメント](/character-set-and-collation.md#new-framework-for-collations), [#17596](https://github.com/pingcap/tidb/issues/17596)
- 照合順序の大文字小文字を区別しない比較をサポート

### セキュリティ

[ユーザードキュメント](/log-redaction.md), [#18566](https://github.com/pingcap/tidb/issues/18566)

セキュリティ準拠要件（例: *一般データ保護規則（GDPR）*など）を満たすために、システムは、出力エラーメッセージおよびログの情報（IDやクレジットカード番号など）を非表示化する機能をサポートしています。これにより、機密情報の漏洩を回避できます。

TiDBは、出力ログ情報の非表示をサポートしています。この機能を有効にするには、次のスイッチを使用します：

+ グローバル変数 [`tidb_redact_log`](/system-variables.md#tidb_redact_log)。デフォルト値は `0` であり、非表示化が無効になっています。tidb-serverログの非表示を有効にするには、変数値を `1` に設定します。
+ 構成項目 `security.redact-info-log`。デフォルト値は `false` であり、非表示化が無効になっています。tikv-serverログの非表示を有効にするには、変数値を `true` に設定します。
+ tiflash-serverおよびtiflash-learner用の構成項目 `security.redact_info_log` および `security.redact-info-log`。デフォルト値はどちらも `false` であり、非表示化が無効になっています。tiflash-serverおよびtiflash-learnerログの非表示を有効にするには、両変数の値を `true` に設定します。

この機能はv5.0で導入されました。この機能を使用するには、上記のシステム変数およびすべての構成項目を有効にする必要があります。

## パフォーマンス最適化

### MPPアーキテクチャ

[ユーザードキュメント](/tiflash/use-tiflash-mpp-mode.md)

TiDBはTiFlashノードを介してMPPアーキテクチャを導入しています。このアーキテクチャにより、複数のTiFlashノードが大規模な結合クエリの実行ワークロードを共有できます。

MPPモードがオンの場合、TiDBは計算コストに基づいてクエリをMPPエンジンに送信するかどうかを決定します。MPPモードでは、テーブル結合の計算を各実行中のTiFlashノードに分散することで結合キーを再配布することでデータ計算中に計算を高速化します。さらに、TiFlashがすでにサポートしている集計計算機能により、TiDBはクエリの計算をTiFlash MPPクラスタにプッシュダウンできます。その後、分散環境が全体の実行プロセスの加速し、解析クエリの速度を劇的に向上させます。

TPC-H 100ベンチマークテストでは、TiFlash MPPは従来のアナリティクスデータベースやHadoop上のSQLのアナリティクスエンジンよりも処理速度が顕著に向上します。このアーキテクチャを使用することで、従来のオフラインアナリティクスソリューションよりも高いパフォーマンスで直近のトランザクションデータで大規模な分析クエリを実行できます。ベンチマークによると、同じクラスタリソースを使用した場合、TiDB 5.0 MPPはGreenplum 6.15.0およびApache Spark 3.1.1に比べて2〜3倍の速度向上を示しており、一部のクエリでは8倍のパフォーマンス向上が見られます。

現在、MPPモードがサポートしていない主要な機能は以下の通りです（詳細は[Use TiFlash](/tiflash/use-tiflash-mpp-mode.md)を参照）：

+ テーブルのパーティション
+ ウィンドウ関数
+ 照合順序
+ 一部のビルトイン関数
+ TiKVからのデータ読み取り
+ OOMスピル
+ Union
+ 完全外部結合

### クラスタ化インデックス

[ユーザードキュメント](/clustered-indexes.md), [#4841](https://github.com/pingcap/tidb/issues/4841)

テーブル構造を設計したり、データベースの動作を分析する際、特定の主キーを持つ列が頻繁にグループ化およびソートされ、これらの列に対するクエリが特定のデータ範囲またはさまざまな値の少量のデータを返し、対応するデータが読み取りまたは書き込みのホットスポットの問題を引き起こさない場合は、クラスタ化インデックス機能を使用することをお勧めします。

クラスタ化インデックス、または一部のデータベース管理システムでは *インデックス構成テーブル* として知られており、テーブルのデータと関連付けられたストレージ構造です。クラスタ化インデックスを作成すると、テーブルから1つ以上の列をインデックスのキーとして指定できます。TiDBはこれらのキーを特定の構造に保存し、これによりTiDBはキーに関連付けられた行を迅速かつ効率的に見つけることができ、クエリおよびデータの書き込みのパフォーマンスが向上します。

クラスタ化インデックス機能が有効になっている場合、TiDBのパフォーマンスは以下の場合で顕著に向上します（例えば、TPC-C tpmCテストでは、クラスタ化インデックスが有効な場合、TiDBのパフォーマンスが39%向上します）：

+ データが挿入される際、クラスタ化インデックスはネットワークからインデックスデータの1つの書き込みを削減します。
+ 等価条件を持つクエリが主キーに関与する場合、クラスタ化インデックスはネットワークからインデックスデータの1つの読み取りを削減します。
+ 範囲条件を持つクエリが主キーに関与する場合、クラスタ化インデックスはネットワークから複数のインデックスデータの読み取りを削減します。
+ 等価条件または範囲条件を持つクエリが主キーのプレフィックスに関与する場合、クラスタ化インデックスはネットワークから複数のインデックスデータの読み取りを削減します。

各テーブルはクラスタ化または非クラスタ化されたインデックスを使用してデータをソートおよび格納できます。これら2つのストレージ構造の違いは以下の通りです：
+ クラスタードインデックスを作成する際には、テーブル内の1つ以上の列をインデックスのキー値として指定できます。クラスタードインデックスは、テーブルのデータをキー値に従ってソートして格納します。各テーブルにはクラスタードインデックスは1つだけです。テーブルがクラスタードインデックスを持っている場合、クラスタードインデックス テーブルと呼ばれます。そうでない場合は、ノンクラスタードインデックス テーブルと呼ばれます。

+ ノンクラスタードインデックスを作成する際には、テーブルのデータが順不同構造で格納されます。ノンクラスタードインデックスのキー値を明示的に指定する必要はありません。TiDBは自動的にユニークなROWIDをデータの各行に割り当てます。クエリ時には、ROWID が対応する行を特定するために使用されます。クエリやデータの挿入時には少なくとも2つのネットワーク I/O 操作が発生するため、パフォーマンスはクラスタードインデックスに比べて低下します。

テーブルデータが変更されると、データベース システムが自動的にクラスタードインデックスとノンクラスタードインデックスを維持します。

すべてのプライマリキーはデフォルトでノンクラスタードインデックスとして作成されます。プライマリキーは、クラスタードインデックスまたはノンクラスタードインデックスのいずれかの方法で作成できます。

+ テーブルを作成する際にステートメントにキーワード `CLUSTERED | NONCLUSTERED` を指定すると、システムは指定された方法でテーブルを作成します。構文は次のとおりです。

```sql
CREATE TABLE `t` (`a` VARCHAR(255), `b` INT, PRIMARY KEY (`a`, `b`) CLUSTERED);
```

または

```sql
CREATE TABLE `t` (`a` VARCHAR(255) PRIMARY KEY CLUSTERED, `b` INT);
```

`SHOW INDEX FROM tbl-name` ステートメントを実行して、テーブルにクラスタードインデックスがあるかどうかを検索できます。

+ システム変数 `tidb_enable_clustered_index` を構成してクラスタードインデックスの機能を制御できます。サポートされる値は `ON`、`OFF`、`INT_ONLY` です。
  + `ON`: すべての種類のプライマリキーに対してクラスタードインデックスの機能が有効になります。ノンクラスタードインデックスの追加と削除がサポートされます。
  + `OFF`: すべての種類のプライマリキーに対してクラスタードインデックスの機能が無効になります。ノンクラスタードインデックスの追加と削除がサポートされます。
  + `INT_ONLY`: デフォルト値です。変数が `INT_ONLY` に設定され、`alter-primary-key` が `false` に設定されている場合、単一の整数列からなるプライマリキーはデフォルトでクラスタードインデックスとして作成されます。この動作は TiDB v5.0 およびそれ以前のバージョンと一致しています。

`CREATE TABLE` ステートメントにキーワード `CLUSTERED | NONCLUSTERED` が含まれている場合、そのステートメントはシステム変数および構成項目の構成を上書きします。

指定するキーワード `CLUSTERED | NONCLUSTERED` を使用してクラスタードインデックスの機能を推奨します。これにより、TiDB が必要に応じてシステム全体でクラスタードおよびノンクラスタードインデックスのすべてのデータ型を使用する柔軟性が高まります。

`tidb_enable_clustered_index = INT_ONLY` を使用することは推奨されません。`INT_ONLY` はこの機能を互換性のために一時的に使用するものであり、将来的に廃止される予定です。

クラスタードインデックスの制限には、以下があります。

+ クラスタードインデックスからノンクラスタードインデックスへの相互変換はサポートされていません。
+ クラスタードインデックスの削除はサポートされていません。
+ `ALTER TABLE` ステートメントを使用してクラスタードインデックスの追加、削除、および変更はサポートされていません。
+ クラスタードインデックスの再編成および再作成はサポートされていません。
+ インデックスの有効化または無効化はサポートされていないため、クラスタードインデックスではインデックスの非表示機能が有効になりません。
+ `UNIQUE KEY` をクラスタードインデックスとして作成することはできません。
+ クラスタードインデックス機能を TiDB ビンログと併用することはサポートされていません。TiDB ビンログが有効になっている場合、TiDB では単一の整数型のプライマリキーのみをクラスタードインデックスとして作成できます。既存のクラスタードインデックスを持つテーブルのデータ変更を下流に複製しません。
+ `SHARD_ROW_ID_BITS` および `PRE_SPLIT_REGIONS` と一緒にクラスタードインデックス機能を使用することはサポートされていません。
+ クラスターが後のバージョンにアップグレードされた後にロールバックする場合、ロールバック前に新たに追加されたテーブルのデータをエクスポートし、ロールバック後にデータをインポートする必要があります。その他のテーブルには影響がありません。

### 非同期コミット

[ユーザードキュメント](/system-variables.md#tidb_enable_async_commit-new-in-v50)、[#8316](https://github.com/tikv/tikv/issues/8316)

データベースのクライアントは、データベース システムがコミットを2つの段階（2PC）で同期的に完了するのを待ちます。最初の段階のコミットが成功した後、トランザクションはクライアントに結果を返し、システムはバックグラウンドで2番目の段階のコミット操作を実行してトランザクションのコミット遅延を減らします。トランザクションの書き込みが1つのリージョンのみに関連する場合、第2段階は直接省略され、トランザクションは1段階のコミットになります。

非同期コミット機能を有効にした後は、同じハードウェアと構成で Sysbench を使用して Update インデックスを64スレッドでテストすると、平均レイテンシが12.04ms から 7.01ms に41.7% 減少します。

非同期コミット機能を有効にした後は、1つのネットワーク相互作用レイテンシを減らし、データ書き込みのパフォーマンスを向上させるために、データベース アプリケーション開発者には、トランザクションの整合性をリニア整合性から[因果整合性](/transaction-overview.md#causal-consistency)に低下させることを考慮することをお勧めします。因果整合性を有効にする SQL ステートメントは `START TRANSACTION WITH CAUSAL CONSISTENCY` です。

因果整合性を有効にした後は、同じハードウェアと構成で Sysbench を使用して oltp_write_only を64スレッドでテストすると、平均レイテンシが11.86ms から 11.19ms に5.6% 減少します。

トランザクションの整合性がリニア整合性から因果整合性に低下した場合、アプリケーション内の複数のトランザクション間に依存関係がない場合、トランザクションにはグローバルで整合的な順序がありません。

**非同期コミット機能は、新しく作成された v5.0 クラスターではデフォルトで有効になっています。**

この機能は、以前のバージョンから v5.0 にアップグレードされたクラスターではデフォルトで無効になっています。この機能を有効にするには、`set global tidb_enable_async_commit = ON;` および `set global tidb_enable_1pc = ON;` ステートメントを実行できます。

非同期コミット機能の制限は以下の通りです。

+ 直接ダウングレードすることはできません。

### コプロセッサーキャッシュ機能をデフォルトで有効にする

[ユーザードキュメント](/tidb-configuration-file.md#tikv-clientcopr-cache-new-in-v400)、[#18028](https://github.com/pingcap/tidb/issues/18028)

5.0 GA では、コプロセッサーキャッシュ機能がデフォルトで有効になっています。この機能を有効にした後、TiDB は tikv-server にプッシュされた演算子の計算結果をキャッシュして、データの読み取りレイテンシを減らします。

コプロセッサーキャッシュ機能を無効にするには、`tikv-client.copr-cache` の `capacity-mb` 構成項目を `0.0` に変更できます。

### `delete from table where id <? Limit ?` ステートメントの実行パフォーマンスを改善

[#18028](https://github.com/pingcap/tidb/issues/18028)

`delete from table where id <? limit ?` ステートメントの p99 パフォーマンスが4倍に改善されました。

### データの分割ができないいくつかの小さなテーブルホットスポット読み取りシナリオでのパフォーマンス問題を解決するために、ベース分割戦略を最適化

[#18005](https://github.com/pingcap/tidb/issues/18005)

## 安定性の向上

### スケジュールの不完全な非安定性の問題を解消するためのパフォーマンスジッターの最適化

[#18005](https://github.com/pingcap/tidb/issues/18005)

TiDB のスケジュールプロセスは、I/O、ネットワーク、CPU、メモリなどのリソースを占有します。TiDB がスケジュールタスクを制御していない場合、予約リソースを奪うことによって QPS および遅延がパフォーマンスジッターを引き起こす可能性があります。

以下の最適化を行った結果、8時間のパフォーマンステストでは TPC-C tpmC の標準偏差が2%を超えませんでした。

#### 不要なスケジュールおよびパフォーマンスジッターを減らすための新しいスケジュール計算式の導入

ノード容量が常にシステムで設定されたウォーターライン近くにある場合、または `store-limit` が大きすぎる場合、容量負荷を均衡させるため、システムは頻繁にリージョンを他のノードにスケジュールしたり、リージョンを元のノードにスケジュールしたりします。このようなスケジュールは、I/O、ネットワーク、CPU、メモリなどのリソースを占有し、パフォーマンスジッターを引き起こすため、不要です。

この問題を緩和するために、PD は新しいデフォルトのスケジュール計算式を導入します。`region-score-formula-version = v1` を構成することで、古い計算式に切り替えることができます。

#### テーブル間リージョンマージ機能がデフォルトで有効になる

[ユーザードキュメント](/pd-configuration-file.md#enable-cross-table-merge)

v5.0 以前には、TiDB ではテーブル間リージョンマージ機能がデフォルトで無効になっていました。v5.0 からは、この機能がデフォルトで有効になり、空のリージョンの数とネットワーク、メモリ、CPU のオーバーヘッドを減らすために使用されます。この機能を無効にするには、`schedule.enable-cross-table-merge` 構成項目を変更できます。

#### システムが背景タスクと前景の読み取りおよび書き込みの I/O リソースの競合を均衡させるために、データの圧縮速度を自動的に調整する機能をデフォルトで有効にする

[ユーザードキュメント](/tikv-configuration-file.md#rate-limiter-auto-tuned-new-in-v50)
v5.0以前、バックグラウンドタスクとフォアグラウンドの読み書き間でI/Oリソースの競合を調整する機能はデフォルトでは無効になっています。v5.0以降、TiDBはデフォルトでこの機能を有効にし、アルゴリズムを最適化してレイテンシのジッタを大幅に減らしています。

この機能を無効にするには、`rate-limiter-auto-tuned`の設定項目を変更します。

#### GC Compaction Filter機能をデフォルトで有効にして、GCがCPUとI/Oリソースを消費するのを削減する

[ユーザードキュメント](/garbage-collection-configuration.md#gc-in-compaction-filter), [#18009](https://github.com/pingcap/tidb/issues/18009)

TiDBがガベージコレクション（GC）とデータ圧縮を実行する際、パーティションはCPUとI/Oリソースを占有します。これらの2つのタスクの実行時にはオーバーラップするデータが存在します。

GCが消費するCPUとI/Oリソースを削減するために、GC Compaction Filter機能はこれらの2つのタスクを組み合わせて1つのタスクで実行します。この機能はデフォルトで有効になっています。`gc.enable-compaction-filter = false`を設定することで無効にすることができます。

#### TiFlashはI/Oリソースの使用状況を制限します（**実験的な機能**）

この機能により、バックグラウンドタスクとフォアグラウンドの読み書きの間のI/Oリソースの競合が軽減されます。

この機能はデフォルトでは無効になっています。`bg_task_io_rate_limit`の設定項目を変更することで、この機能を有効にすることができます。

#### スケジューリング制約のチェックパフォーマンスと大規模なクラスタでの不健全なリージョンの修正パフォーマンスを改善する

### パフォーマンスのジッタを避けるために実行計画が変更されないようにする

[ユーザードキュメント](/sql-plan-management.md)

#### SQLバインディングは`INSERT`、`REPLACE`、`UPDATE`、`DELETE`ステートメントをサポートします

パフォーマンスの調整やデータベースのメンテナンスを行う際に、実行計画が安定しないことによりシステムのパフォーマンスに影響がある場合、判断や`EXPLAIN ANALYZE`でテストされた最適化されたSQL文を選択し、アプリケーションコードで実行されるSQL文にバインドすることでパフォーマンスの安定性を確保することができます。

SQLバインディングを使用してSQL文を手動でバインドする場合、最適化されたSQL文が元のSQL文と同じ構文を持っていることを確認する必要があります。

バインドされた実行計画情報は、`SHOW {GLOBAL | SESSION} BINDINGS`コマンドを実行することで表示することができます。出力はv5.0より前のバージョンと同じです。

#### 自動的に実行計画をキャプチャしバインドする

TiDBをアップグレードする際には、パフォーマンスのジッタを回避するためにベースラインキャプチャ機能を有効にして、システムが最新の実行計画を自動的にキャプチャしバインドし、システムテーブルに保存することができます。TiDBをアップグレードした後、`SHOW GLOBAL BINDING`コマンドを実行してバインドされた実行計画をエクスポートし、これらの計画を削除するかどうかを決定することができます。

この機能はデフォルトでは無効です。`tidb_capture_plan_baselines`グローバルシステム変数を変更するか、サーバーの設定を変更して有効にすることができます。この機能が有効になると、システムはステートメントのサマリから少なくとも2回以上出現するSQL文を`bind-info-lease`（デフォルト値は`3s`）ごとに収集し、自動的にこれらのSQL文をキャプチャしバインドします。

### TiFlashクエリの安定性を向上させる

TiFlashクエリが失敗した場合、クエリをTiKVにフォールバックするためのシステム変数[`tidb_allow_fallback_to_tikv`](/system-variables.md#tidb_allow_fallback_to_tikv-new-in-v50)を追加します。デフォルト値は`OFF`です。

### TiCDCの安定性を向上し、過剰な増分データのレプリケートによるOOM問題を緩和する

[TiCDCユーザードキュメント](/ticdc/ticdc-manage-changefeed.md#unified-sorter), [#1150](https://github.com/pingcap/tiflow/issues/1150)

TiCDC v4.0.9以前のバージョンでは、大量のデータ変更をレプリケートするとOOMが発生する場合があります。v5.0ではUnified Sorter機能をデフォルトで有効にし、次のシナリオで引き起こされるOOM問題を緩和することができます。

- TiCDCのデータレプリケートタスクが長時間停止し、大量の増分データが蓄積され、レプリケートする必要がある場合。
- データレプリケートタスクが初期タイムスタンプから開始され、大量の増分データをレプリケートする必要がある場合。

Unified Sorterは、以前のバージョンの`memory`/`file`ソートエンジンオプションと統合されています。手動で設定する必要はありません。

制限事項:

- 増分データの量に応じて十分なディスク容量を用意する必要があります。空き容量が128GB以上あるSSDを使用することを推奨します。

## 高可用性と災害復旧

### リージョンのメンバーシップ変更中のシステムの可用性を向上する

[ユーザードキュメント](/pd-configuration-file.md#enable-joint-consensus-new-in-v50), [#18079](https://github.com/pingcap/tidb/issues/18079), [#7587](https://github.com/tikv/tikv/issues/7587), [#2860](https://github.com/tikv/pd/issues/2860)

リージョンのメンバーシップの変更中、"メンバーの追加"と"メンバーの削除"は2つのステップで行われる2つの操作です。メンバーシップの変更が完了した時点で障害が発生した場合、リージョンは利用できなくなり、フォアグラウンドのアプリケーションにエラーが返されます。

導入されたRaft Joint Consensusアルゴリズムは、リージョンのメンバーシップ変更中のシステムの可用性を向上させることができます。メンバーシップの変更中に"メンバーの追加"と"メンバーの削除"操作を1つの操作に統合し、すべてのメンバーに送信します。変更プロセス中は、リージョンは中間状態にあります。変更されたメンバーが障害を起こしても、システムは利用可能です。

この機能はデフォルトで有効になっています。`pd-ctl config set enable-joint-consensus`コマンドを実行して、`enable-joint-consensus`の値を`false`に設定することで無効にすることができます。

### メモリ管理モジュールを最適化してシステムのOOMリスクを削減する

集約関数のメモリ使用量をトラックします。この機能はデフォルトで有効になっています。集約関数を含むSQL文が実行されると、現在のクエリの総メモリ使用量が`mem-quota-query`で設定された閾値を超える場合、システムは自動的に`oom-action`で定義された操作を実行します。

### ネットワーク分断中のシステムの可用性を向上させる

## データマイグレーション

### S3/AuroraからTiDBへのデータマイグレーション

TiDBデータマイグレーションツールは、Amazon S3（および他のS3互換ストレージサービス）をデータマイグレーションの中間データとして使用し、Auroraのスナップショットデータを直接TiDBに初期化することができる機能をサポートしています。これにより、Amazon S3/AuroraからTiDBへのデータマイグレーションにさまざまなオプションを提供することができます。

この機能の使用方法については、以下のドキュメントを参照してください：

- [データをAmazon S3クラウドストレージにエクスポートする](/dumpling-overview.md#export-data-to-amazon-s3-cloud-storage), [#8](https://github.com/pingcap/dumpling/issues/8)
- [Amazon Aurora MySQLからTiDB Lightningを使用してマイグレーションする](/migrate-aurora-to-tidb.md), [#266](https://github.com/pingcap/tidb-lightning/issues/266)

### TiDB Cloudのデータインポートのパフォーマンスを最適化する

TiDB Lightningは、特にTiDB CloudのAWS T1.standard構成（または同等の構成）のデータインポートパフォーマンスを最適化しています。テスト結果によれば、TiDB LightningはTPC-Cの1TBのデータをTiDBにインポートする速度を254 GiB/hから366 GiB/hに向上させることができます。

## データ共有とサブスクリプション

### TiCDCを使用してTiDBをKafka Connect（Confluent Platform）に統合する（**実験的な機能**）

[ユーザードキュメント](/ticdc/integrate-confluent-using-ticdc.md), [#660](https://github.com/pingcap/tiflow/issues/660)

TiDBのデータを他のシステムにストリーミングするビジネス要件をサポートするために、この機能はTiDBのデータをKafka、Hadoop、Oracleなどのシステムにストリーミングすることができます。

Confluentプラットフォームが提供するKafkaコネクタプロトコルは、コミュニティで広く使用されており、異なるプロトコルでリレーショナルまたは非リレーショナルデータベースにデータを転送することができます。TiCDCをConfluentプラットフォームのKafka Connectに統合することで、TiDBはTiDBデータを他の異種データベースやシステムにストリーミングする能力を拡張します。

## 診断

[ユーザードキュメント](/sql-statements/sql-statement-explain.md#explain)

SQLのパフォーマンスの問題のトラブルシューティング中には、パフォーマンスの問題の原因を特定するために詳細な診断情報が必要です。TiDB v5.0以前では、`EXPLAIN`ステートメントで収集された情報は十分に詳細ではありませんでした。問題の根本原因は、ログ情報やモニタリング情報、さらには推測に基づいてのみ特定される場合があり、非効率なことがありました。

TiDB v5.0では、次の改善が行われ、パフォーマンスの問題のトラブルシューティングをより効率的に行うことができるようになりました。

+ `EXPLAIN ANALYZE`ステートメントを使用して、すべてのDMLステートメントを分析し、実際のパフォーマンス計画と各オペレータの実行情報を表示することができます。[#18056](https://github.com/pingcap/tidb/issues/18056)
+ `EXPLAIN FOR CONNECTION`ステートメントを使用して、実行中のすべてのSQLステートメントのリアルタイムの状態を確認することができます。例えば、各オペレータの実行時間や処理された行数を確認するために使用することができます。[#18233](https://github.com/pingcap/tidb/issues/18233)
+ `EXPLAIN ANALYZE` ステートメントの出力で、オペレータの実行に関する詳細を提供します。これには、オペレータが送信した RPC リクエストの数、ロックの競合を解決するための時間、ネットワークの待ち時間、RocksDB で削除されたデータのスキャンされた量、および RocksDB キャッシュのヒット率が含まれます。[#18663](https://github.com/pingcap/tidb/issues/18663)

+ SQL ステートメントの詳細な実行情報をスローログに自動的に記録するサポートを提供します。スローログ中の実行情報は、`EXPLAIN ANALYZE` ステートメントの出力情報と一貫しており、各オペレータによって消費された時間、処理された行数、および送信された RPC リクエストの数が含まれています。[#15009](https://github.com/pingcap/tidb/issues/15009)

## デプロイとメンテナンス

### クラスターデプロイメント操作のロジックを最適化し、DBA が標準的な TiDB プロダクションクラスターをより迅速に展開できるように支援します

[ユーザドキュメント](/production-deployment-using-tiup.md)

以前の TiDB バージョンでは、TiUP を使用して TiDB クラスターを展開する DBA は、環境の初期化が複雑で、チェックサムの構成が過剰で、クラスタートポロジファイルの編集が困難であることがわかりました。これらの問題がすべて DBA の展開効率の低下につながっています。TiDB v5.0 では、DBA の TiUP を使用した TiDB デプロイの効率が以下の項目を通じて向上しています。

+ TiUP クラスターは、より包括的なワンクリック環境チェックを実行し、修復の推奨事項を提供するための `check topo.yaml` コマンドをサポートします。
+ TiUP クラスターは、環境チェック中に見つかった環境問題を自動的に修復するための `check topo.yaml --apply` コマンドをサポートします。
+ TiUP クラスターは、DBA が編集するためのクラスタートポロジテンプレートファイルを取得し、グローバルノードパラメータを変更するのをサポートするための `template` コマンドをサポートします。
+ `edit-config` コマンドを使用して `remote_config` パラメータを編集し、リモートプロメテウスを構成する。
+ `edit-config` コマンドを使用して異なる AlertManager を構成するための `external_alertmanagers` パラメータを編集します。
+ tiup-cluster でトポロジファイルを編集する際に、構成項目の値のデータ型を変更できます。

### アップグレードの安定性を向上させる

TiUP v1.4.0 以前では、tiup-cluster を使用して TiDB クラスターをアップグレードすると、クラスターの SQL 応答が長時間不安定になり、PD オンラインローリングアップグレード中に、クラスターの QPS が 10秒から30秒の間不安定になりました。

TiUP v1.4.0 では、ロジックを調整し、以下の最適化を行っています。

+ PD ノードのアップグレード中に、TiUP は自動的に再起動した PD ノードのステータスをチェックし、ステータスが準備完了であることを確認した後、次の PD ノードのアップグレードをローリングで行います。
+ TiUP は PD の役割を自動的に識別し、まずフォロワー役割の PD ノードをアップグレードし、最後に PD リーダー ノードをアップグレードします。

### アップグレード時間の最適化

TiUP v1.4.0 以前は、tiup-cluster を使用して TiDB クラスターをアップグレードすると、ノード数が多いクラスターの場合、総アップグレード時間が長く、特定のユーザのアップグレード時間ウィンドウの要件を満たすことができませんでした。

v1.4.0 からは、TiUP が以下の項目を最適化しています。

+ `tiup cluster upgrade --offline` サブコマンドを使用して、高速なオフラインアップグレードをサポートします。
+ デフォルトでローリングアップグレードを使用するユーザ向けに、アップグレード中にリージョン リーダーの再配置を高速化し、それにより TiKV のローリングアップグレードの時間を短縮します。
+ ローリングアップグレードを実行する前に、`check` サブコマンドを使用してリージョンモニタのステータスをチェックします。アップグレード前にクラスターが正常な状態であることを確認し、それによりアップグレードの失敗の確率を低減します。

### ブレークポイント機能のサポート

TiUP v1.4.0 以前では、tiup-cluster を使用して TiDB クラスターをアップグレードする場合、コマンドの実行が中断された場合、すべてのアップグレード操作を最初から実行する必要がありました。

TiUP v1.4.0 では、中断されたアップグレード操作をブレークポイントから再試行するための tiup-cluster `replay` サブコマンドをサポートし、アップグレードの中断後にすべての操作を再実行することを避けます。

### メンテナンスおよび運用機能の拡張

TiUP v1.4.0 では、TiDB クラスターの運用とメンテナンス機能をさらに強化しています。

+ ダウンタイムの TiDB および DM クラスターに対するアップグレードまたはパッチ操作をサポートし、さらに多様な使用シナリオに適応します。
+ `display` サブコマンドの `--version` パラメータを追加し、クラスターバージョンを取得します。
+ モニタリング構成の更新を実行しないように、スケールアウトされているノードに Promethus しか含まれていない場合に操作を実行しないようにし、Prometheus ノードの不在によるスケールアウトの失敗を回避します。
+ 入力された TiUP コマンドの結果が不正な場合に、エラーメッセージにユーザー入力を追加し、問題の原因を迅速に特定できるようにします。

## テレメトリ

TiDB は、クラスターの使用状況メトリクスをテレメトリに追加します。これには、データテーブルの数、クエリの数、および新機能の有効化の有無が含まれます。

詳細およびこの動作を無効にする方法については、[telemetry](/telemetry.md) を参照してください。