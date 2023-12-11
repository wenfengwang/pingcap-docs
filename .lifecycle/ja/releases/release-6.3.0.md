---
title: TiDB 6.3.0 リリースノート
---

# TiDB 6.3.0 リリースノート

リリース日: 2022年9月30日

TiDB バージョン: 6.3.0-DMR

> **注意:**
>
> TiDB 6.3.0-DMR のドキュメントは[こちら](https://docs-archive.pingcap.com/tidb/v6.3/)でアーカイブされています。PingCAP では TiDB データベースの[最新 LTS バージョン](https://docs.pingcap.com/tidb/stable)のご使用をお勧めしています。

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.3/quick-start-with-tidb) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.3.0#version-list)

v6.3.0-DMR における主な新機能と改善点は以下の通りです:

- TiKV は SM4 アルゴリズムを使用した暗号化をサポートします。
- TiDB は SM3 アルゴリズムを使用した認証をサポートします。
- `CREATE USER` および `ALTER USER` ステートメントが `ACCOUNT LOCK/UNLOCK` オプションをサポートします。
- JSON データ型と関数が一般的に利用可能（GA）になります。
- TiDB は null-aware アンチジョインをサポートします。
- TiDB はより細かい粒度で実行時間メトリクスを提供します。
- 新しい構文糖が追加され、Range パーティションの定義が簡略化されます。
- Range COLUMNS パーティション化では複数の列を定義することができます。
- インデックスの追加のパフォーマンスが3倍に向上しました。
- リソース消費クエリが軽量クエリの応答時間に及ぼす影響が50%以上削減されました。

## 新機能

### SQL

* 新しい構文糖（Range INTERVAL パーティション化）が追加され、Range パーティションの定義が簡略化されます（実験的） [#35683](https://github.com/pingcap/tidb/issues/35683) @[mjonss](https://github.com/mjonss)

    TiDB は [INTERVAL パーティション化](/partitioned-table.md#range-interval-partitioning) を新しいRangeパーティションの定義方法として提供します。すべてのパーティションを列挙する必要はなく、これにより切り詰められた Range パーティションの DDL ステートメントが大幅に削減されます。構文は元の Range パーティション化のそれと同等です。

* Range COLUMNS パーティション化では、複数の列を定義することができます [#36636](https://github.com/pingcap/tidb/issues/36636) @[mjonss](https://github.com/mjonss)

    TiDB は [PARTITION BY RANGE COLUMNS (column_list)](/partitioned-table.md#range-columns-partitioning) をサポートします。`column_list` は単一の列に限定されなくなりました。基本的な機能は MySQL と同じです。

* [EXCHANGE PARTITION](/partitioned-table.md#partition-management) がGAになりました [#35996](https://github.com/pingcap/tidb/issues/35996) @[ymkzpx](https://github.com/ymkzpx)

* 2つのウィンドウ関数のプッシュダウンがTiFlashへサポートされます [#5579](https://github.com/pingcap/tiflash/issues/5579) @[SeaRise](https://github.com/SeaRise)

    * `LEAD()`
    * `LAG()`

* DDL の変更時に DML 成功率を向上させる軽量なメタデータロックが提供されます（実験的） [#37275](https://github.com/pingcap/tidb/issues/37275) @[wjhuang2016](https://github.com/wjhuang2016)

    TiDB はオンライン非同期スキーマ変更アルゴリズムを使用してメタデータオブジェクトの変更をサポートします。トランザクション実行時、トランザクション開始時に対応するメタデータのスナップショットを取得します。トランザクション中にメタデータが変更されると、データの整合性を確保するため、TiDB は「情報スキーマが変更されました」とエラーを返し、トランザクションはコミットできなくなります。この問題を解決するため、TiDB v6.3.0 ではオンライン DDL アルゴリズムに[メタデータロック](/metadata-lock.md)を導入しています。可能な限り DML エラーを回避するために、TiDB はテーブルメタデータの変更中に DDL を実行し、古いメタデータを持つDMLを待機させます。

* インデックスの追加のパフォーマンスを向上させ、DML トランザクションへの影響を減らします（実験的） [#35983](https://github.com/pingcap/tidb/issues/35983) @[benjamin2037](https://github.com/benjamin2037)

    インデックスを作成する際のバックフィル速度を向上させるために、TiDB v6.3.0 は [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630) システム変数が有効化されている場合、`ADD INDEX` および `CREATE INDEX` の DDL操作を高速化します。この機能が有効な場合、インデックスの追加のパフォーマンスが約3倍に向上します。

### セキュリティ

* TiKV は安息時の暗号化に SM4 アルゴリズムをサポートします [#13041](https://github.com/tikv/tikv/issues/13041) @[jiayang-zheng](https://github.com/jiayang-zheng)

    TiKV の安息時暗号化に [SM4 アルゴリズム](/encryption-at-rest.md)が追加されました。安息時暗号化を構成する際には、`data-encryption-method` 設定の値を `sm4-ctr` に設定することで、SM4 暗号化機能を有効にできます。

* TiDB は SM3 アルゴリズムによる認証をサポートします [#36192](https://github.com/pingcap/tidb/issues/36192) @[CbcWestwolf](https://github.com/CbcWestwolf)

    TiDB はSM3アルゴリズムに基づく認証プラグイン [`tidb_sm3_password`](/security-compatibility-with-mysql.md) を追加しました。このプラグインが有効になると、ユーザーのパスワードはSM3アルゴリズムを使用して暗号化され、検証されます。

* TiDB JDBC はSM3アルゴリズムによる認証をサポートします [#25](https://github.com/pingcap/mysql-connector-j/issues/25) @[lastincisor](https://github.com/lastincisor)

    ユーザーのパスワードの認証にはクライアントサイドのサポートが必要です。現在、[JDBCがSM3アルゴリズムをサポート](/develop/dev-guide-choose-driver-or-orm.md#java-drivers)しているため、TiDB-JDBC経由でSM3認証を使用してTiDBに接続できます。

### 可観測性

* TiDB はSQLクエリの実行時間の粒度の細かいメトリクスを提供します [#34106](https://github.com/pingcap/tidb/issues/34106) @[cfzjywxk](https://github.com/cfzjywxk)

    TiDB v6.3.0 は[実行時間の詳細な観測に向けたメトリクスを提供します](/latency-breakdown.md)。完全でセグメント化されたメトリクスを通じて、SQLクエリの主要な時間消費を明確に把握し、すばやく重要な問題を見つけ、トラブルシューティングに時間を節約することができます。

* スローログと`TRACE`ステートメントの出力が強化されました [#34106](https://github.com/pingcap/tidb/issues/34106) @[cfzjywxk](https://github.com/cfzjywxk)

    TiDB v6.3.0 はスローログと`TRACE`の出力を強化しました。TiDBのパーシングからKV RocksDBへのディスク書き込みまでの[全リンクの期間](/latency-breakdown.md)を観測できるようになり、さらに診断機能が強化されます。

* TiDB ダッシュボードはデッドロックの履歴情報を提供します [#34106](https://github.com/pingcap/tidb/issues/34106) @[cfzjywxk](https://github.com/cfzjywxk)

    v6.3.0 から、TiDBダッシュボードはデッドロックの履歴情報を提供します。TiDBダッシュボードのスローログを確認して、いくつかのSQLステートメントのロック待ち時間が過剰に長いと判断された場合、デッドロックの履歴を確認し、ルート原因を特定することが可能となり、診断を容易にします。

### パフォーマンス

* TiFlashはFastScanの使用方法を変更しました（実験的） [#5252](https://github.com/pingcap/tiflash/issues/5252) @[hongyunyan](https://github.com/hongyunyan)

    v6.2.0 では、TiFlashは期待されるパフォーマンス改善をもたらすFastScan機能を導入しましたが、使用方法には柔軟性が不足していました。そのため、v6.3.0 では、FastScanの有効化および無効化に`ALTER TABLE ... SET TIFLASH MODE ...`構文は廃止されました。代わりに、システム変数 [`tiflash_fastscan`](/system-variables.md#tiflash_fastscan-new-in-v630) を使用して簡単にFastScanの有効化/無効化を制御できるようになりました。

    v6.2.0 から v6.3.0 へのアップグレード時、v6.2.0でのすべてのFastScan設定は無効になりますが、データの正常な読み取りには影響しません。`tiflash_fastscan`変数を設定する必要があります。v6.2.0またはそれ以前のバージョンからv6.3.0にアップグレードすると、デフォルトですべてのセッションでFastScan機能は有効化されませんが、これはデータの整合性を維持するためです。

* TiFlashは複数の並行タスクのシナリオでのデータスキャンパフォーマンスを最適化しました [#5376](https://github.com/pingcap/tiflash/issues/5376) @[JinheLin](https://github.com/JinheLin)

    TiFlashは同じデータの読み取り操作を結合することで、同じデータの重複した読み込みを軽減します。これにより、リソースのオーバーヘッドが最適化され、[同時タスクのデータスキャンのパフォーマンスが改善](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)されます。複数の同時タスクの場合、各タスクが同じデータを別々に読み取る状況を避け、同じデータを複数回同時に読み取る可能性も回避します。

    この機能はv6.2.0で実験的なものとなり、v6.3.0でGAとなります。

* TiFlashはデータ複製のパフォーマンスを改善します [#5237](https://github.com/pingcap/tiflash/issues/5237) @[breezewish](https://github.com/breezewish)

    TiFlashはRaftプロトコルを使用して、TiKVからデータを複製します。v6.3.0以前では、大量のレプリカデータの複製にはしばしば長い時間がかかりました。TiDB v6.3.0により、TiFlashのデータ複製メカニズムが最適化され、複製速度が大幅に向上します。BRを使用してデータを回復したり、TiDB Lightningを使用してデータをインポートしたり、新しいTiFlashレプリカを追加する際には、TiFlashレプリカをより迅速に複製することができます。TiFlashを使用して、より迅速にクエリを実行できます。また、TiFlashレプリカは、スケールアップ、スケールダウン、またはTiFlashレプリカの数を変更した場合に、より安全でバランスの取れた状態にも迅速に到達します。

* TiFlashは個々の`COUNT(DISTINCT)`の3段階の集計をサポートします [#37202](https://github.com/pingcap/tidb/issues/37202) @[fixdb](https://github.com/fixdb)

    TiFlashは、単一の`COUNT(DISTINCT)`のみを含むクエリを、[3段階の集計](/system-variables.md#tidb_opt_three_stage_distinct_agg-new-in-v630)に書き換えることをサポートしています。これにより並行性とパフォーマンスが向上します。

* TiKVはログのリサイクルをサポートします [#214](https://github.com/tikv/raft-engine/issues/214) @[LykxSassinator](https://github.com/LykxSassinator)

    TiKVはRaft Engineで[ログファイルのリサイクル](/tikv-configuration-file.md#enable-log-recycle-new-in-v630)をサポートします。これにより、Raftログの追加中のネットワークディスクでの長い待ち時間が削減され、書き込みワークロードにおけるパフォーマンスが向上します。

* TiDBはヌルに対応したアンチ結合をサポートします [#37525](https://github.com/pingcap/tidb/issues/37525) @[Arenatlx](https://github.com/Arenatlx)

    TiDB v6.3.0では、新しい結合タイプである[ヌルに対応したアンチ結合 (NAAJ)](/explain-subqueries.md#null-aware-anti-semi-join-not-in-and--all-subqueries)が導入されます。NAAJは、コレクション操作を処理する際に、コレクションが空であるか`NULL`であるかを認識できます。これにより、`IN`や`= ANY`などの操作の実行効率が最適化され、SQLのパフォーマンスが向上します。

* ハッシュ結合のビルドエンドを制御するために最適化ヒントを追加する [#35439](https://github.com/pingcap/tidb/issues/35439) @[Reminiscent](https://github.com/Reminiscent)

    v6.3.0では、TiDBオプティマイザに`HASH_JOIN_BUILD()`および`HASH_JOIN_PROBE()`の2つのヒントが導入され、ハッシュ結合およびそのプローブエンド、ビルドエンドを指定することができます。オプティマイザが最適な実行プランを選択できない場合、これらのヒントを使用してプランに介入することができます。

* セッションレベルでの共通テーブル式(CTE)インラインをサポートする [#36514](https://github.com/pingcap/tidb/issues/36514) @[elsa0520](https://github.com/elsa0520)

    TiDB v6.2.0では、CTEインラインを許可するために`MERGE`ヒントがオプティマイザに導入され、これによりTiFlashでCTEクエリ結果のコンシューマが並行して実行できます。v6.3.0では、セッション変数[`tidb_opt_force_inline_cte`](/system-variables.md#tidb_opt_force_inline_cte-new-in-v630)が導入され、セッションでCTEインラインを許可することができます。これにより、利便性が大幅に向上します。

### トランザクション

* 悲観的トランザクションにおける一意制約のチェックの遅延をサポートする [#36579](https://github.com/pingcap/tidb/issues/36579) @[ekexium](https://github.com/ekexium)

    [`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630)システム変数を使用すると、悲観的トランザクションにおける[一意制約のチェック](/constraints.md#pessimistic-transactions)をどのタイミングで行うかを制御できます。この変数はデフォルトで無効になっています。変数を有効にする場合（`ON`に設定する場合）、TiDBは悲観的トランザクションにおけるロック操作と一意制約のチェックを必要な時まで延期するため、大量のDML操作のパフォーマンスが向上します。

* Read-Committed分離レベルでのTSOの取得方法を最適化する [#36812](https://github.com/pingcap/tidb/issues/36812) @[TonsnakeLin](https://github.com/TonsnakeLin)

    Read-Committed分離レベルでは、システム変数[`tidb_rc_write_check_ts`](/system-variables.md#tidb_rc_write_check_ts-new-in-v630)が導入され、TSOの取得方法を制御できるようになります。Plan Cacheがヒットした場合、TiDBはTSOの取得頻度を減らすことでバッチでのDMLステートメントの実行効率を向上し、バッチで実行されるタスクの実行時間を短縮します。

### 安定性

* リソース消費が大きいクエリが軽量クエリの応答時間に与える影響を軽減する [#13313](https://github.com/tikv/tikv/issues/13313) @[glorv](https://github.com/glorv)

    リソース消費が大きいクエリと軽量クエリが同時に実行される場合、軽量クエリの応答時間に影響が及ぶことがあります。この場合、トランザクションサービスの品質を確保するために、軽量クエリが先にTiDBで処理されることが期待されます。v6.3.0では、TiKVは読み取りリクエストのスケジューリングメカニズムを最適化し、各ラウンドでリソース消費が大きいクエリの実行時間が期待値に達するようにします。このことにより、リソース消費が大きいクエリが軽量クエリの応答時間に与える影響が大幅に削減され、混合ワークロードシナリオにおけるP99レイテンシが50%以上削減されます。

* 統計情報が古くなった場合の統計のロードポリシーを変更する [#27601](https://github.com/pingcap/tidb/issues/27601) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)

    v5.3.0では、TiDBは統計が古くなった場合のオプティマイザの挙動を制御するために[`tidb_enable_pseudo_for_outdated_stats`](/system-variables.md#tidb_enable_pseudo_for_outdated_stats-new-in-v530)システム変数を導入しました。デフォルト値は`ON`で、統計情報が古くなった場合、オプティマイザはその統計情報（テーブルの行数以外の統計情報）を信頼できなくなったと見なし、代わりに擬似統計情報を使用します。実際のユーザーシナリオのテストと分析の結果、v6.3.0以降のデフォルト値は`OFF`に変更されています。統計情報が古くなった場合でも、オプティマイザはテーブルの統計情報を引き続き使用します。

* Titanの無効化がGAとなります @[tabokie](https://github.com/tabokie)

    オンラインTiKVノードで[Titanを無効にする](/storage-engine/titan-configuration.md#disable-titan)ことができます。

* GlobalStatsが準備ができていない場合の`static`パーティションプルーニングの使用 [#37535](https://github.com/pingcap/tidb/issues/37535) @[Yisaer](https://github.com/Yisaer)

    [`dynamic pruning`](/partitioned-table.md#dynamic-pruning-mode)が有効になっている場合、オプティマイザは[GlobalStats](/statistics.md#collect-statistics-of-partitioned-tables-in-dynamic-pruning-mode)に基づいて実行プランを選択します。GlobalStatsが完全に収集されるまで、擬似統計情報を使用することでパフォーマンスが低下する可能性があります。v6.3.0では、この問題に対処し、GlobalStatsが収集されるまで`static`モードを維持することでパフォーマンスの安定性を確保します。パーティションプルーニング設定を変更する際に、TiDBはGlobalStatsが収集されるまで`static`モードに留まります。

### 使いやすさ

* SQLベースのデータ配置ルールとTiFlashレプリカの競合を解消する [#37171](https://github.com/pingcap/tidb/issues/37171) @[lcwangchao](https://github.com/lcwangchao)

    TiDB v6.0.0は[SQLベースのデータ配置ルール](/placement-rules-in-sql.md)を提供していますが、実装の問題によりTiFlashレプリカと競合することがあります。TiDB v6.3.0では、実装メカニズムを最適化し、SQLベースのデータ配置ルールとTiFlashの競合を解消します。

### MySQL互換性

* MySQL 8.0の互換性を向上させ、4つの正規表現関数`REGEXP_INSTR()`、`REGEXP_LIKE()`、`REGEXP_REPLACE()`、`REGEXP_SUBSTR()`をサポートすることで改善 [#23881](https://github.com/pingcap/tidb/issues/23881) @[windtalker](https://github.com/windtalker)
    MySQLとの互換性の詳細については、[MySQLとの正規表現互換性](/functions-and-operators/string-functions.md#regular-expression-compatibility-with-mysql) を参照してください。

* `CREATE USER`および`ALTER USER`ステートメントは、`ACCOUNT LOCK/UNLOCK`オプションをサポートします[#37051](https://github.com/pingcap/tidb/issues/37051) @[CbcWestwolf](https://github.com/CbcWestwolf)

    [`CREATE USER`](/sql-statements/sql-statement-create-user.md) ステートメントを使用してユーザーを作成する場合、`ACCOUNT LOCK/UNLOCK`オプションを使用して作成されたユーザーがロックされているかどうかを指定できます。 ロックされたユーザーはデータベースにログインできません。

    [`ALTER USER`](/sql-statements/sql-statement-alter-user.md) ステートメントで、既存のユーザーのロック状態を変更できます。

* JSONデータタイプとJSON関数がGAになりました [#36993](https://github.com/pingcap/tidb/issues/36993) @[xiongjiwei](https://github.com/xiongjiwei)

    JSONは多くのプログラムで採用されている人気のあるデータ形式です。 TiDBは、以前のバージョンから実験的な機能としてMySQLのJSONデータ型および一部のJSON関数と互換性のある[JSONサポート](/data-type-json.md)を導入しています。

    TiDB v6.3.0では、JSONデータ型および関数がGAになり、TiDBのデータ型を充実させ、[式インデックス](/sql-statements/sql-statement-create-index.md#expression-index)および[生成列](/generated-columns.md)でJSON関数を使用できるようになり、さらにMySQLとの互換性が向上しています。

### バックアップとリストア

* PITRは[GCSとAzure Blob Storage](/br/backup-and-restore-storages.md)をバックアップストレージとしてサポートします @[joccau](https://github.com/joccau)

    TiDBクラスターがGoogle CloudまたはAzureに展開されている場合、クラスターをv6.3.0にアップグレードした後、PITR機能を使用できます。

* BRはAWS S3 Object Lockをサポートします [#13442](https://github.com/tikv/tikv/issues/13442) @[3pointer](https://github.com/3pointer)

    [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)を有効にすることで、AWS上のバックアップデータを不正な操作や削除から保護できます。

### データ移行

* TiDB Lightningは、Apache Hiveでエクスポートされた[ParquetファイルをTiDBにインポートできるようになりました](/tidb-lightning/tidb-lightning-data-source.md#parquet) [#37536](https://github.com/pingcap/tidb/issues/37536) @[buchuitoudegou](https://github.com/buchuitoudegou)

* DMに新しい構成項目`safe-mode-duration`が追加されました [#6224](https://github.com/pingcap/tiflow/issues/6224) @[okJiang](https://github.com/okJiang)

    この構成項目は[タスク構成ファイル](/dm/task-configuration-file-full.md)に追加されます。 DMが異常終了した後の自動セーフモードの期間を調整できます。デフォルト値は60秒です。`safe-mode-duration`が`"0s"`に設定されている場合、DMが異常な再起動後にセーフモードに入ろうとするとエラーが報告されます。

### TiDBデータ共有サブスクリプション

* TiCDCは、複数の地理的に分散したデータソースからデータをレプリケートできるデプロイメントトポロジをサポートします [#5301](https://github.com/pingcap/tiflow/issues/5301) @[sdojjy](https://github.com/sdojjy)

    単一のTiDBクラスターから複数の地理的に分散したデータシステムにデータをレプリケートするために、v6.3.0から[TiCDCを複数のIDCに展開](/ticdc/deploy-ticdc.md)して、各IDCにデータをレプリケートすることができるようになりました。 この機能により、地理的に分散したデータのレプリケーションとデプロイメントトポロジの機能が提供されます。

* TiCDCは、上流と下流（同期ポイント）間でスナップショットを一貫させることをサポートします [#6977](https://github.com/pingcap/tiflow/issues/6977) @[asddongmen](https://github.com/asddongmen)

    ディザスタリカバリのためのデータレプリケーションシナリオでは、TiCDCは[定期的に下流データスナップショットを維持](/ticdc/ticdc-upstream-downstream-check.md)し、下流スナップショットを上流スナップショットと一貫させます。 この機能により、読み取りと書き込みが分離されたシナリオをより良くサポートし、コストを低減できます。

* TiCDCは、優雅なアップグレードをサポートします [#4757](https://github.com/pingcap/tiflow/issues/4757) @[overvenus](https://github.com/overvenus) @[3AceShowHand](https://github.com/3AceShowHand)

    TiCDCは、[TiUP](/ticdc/deploy-ticdc.md#upgrade-cautions) (>=v1.11.0)または[TiDB Operator](https://docs.pingcap.com/tidb-in-kubernetes/v1.3/configure-a-tidb-cluster#configure-graceful-upgrade-for-ticdc-cluster) (>=v1.3.8)を使用してデプロイされている場合、TiCDCクラスターを優雅にアップグレードできます。 アップグレード中は、データレプリケーションの遅延が30秒以下に抑えられます。 これにより、安定性が向上し、遅延に敏感なアプリケーションをより良くサポートできます。

## 互換性変更

### システム変数

| 変数名 | 変更タイプ | 説明 |
| --- | --- | --- |
| [`default_authentication_plugin`](/system-variables.md#default_authentication_plugin) | 変更済み | `tidb_sm3_password`という新しいオプションが追加されました。この変数が`tidb_sm3_password`に設定されている場合、暗号化アルゴリズムとしてSM3が使用されます。 |
| [`sql_require_primary_key`](/system-variables.md#sql_require_primary_key-new-in-v630)   | 新規追加  | テーブルにプライマリキーが必要であるという要件を強制するかどうかを制御します。 この変数を有効にした後、プライマリキーなしでテーブルを作成または変更しようとするとエラーが発生します。 |
| [`tidb_adaptive_closest_read_threshold`](/system-variables.md#tidb_adaptive_closest_read_threshold-new-in-v630) | 新規追加 | [`tidb_replica_read`](/system-variables.md#tidb_replica_read-new-in-v40)が`closest-adaptive`に設定されている場合、TiDBサーバーが同じリージョンのレプリカに読み取りリクエストを送信する閾値を制御します。 |
| [`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630) | 新規追加 | TiDBが[悲観的トランザクション](/constraints.md#pessimistic-transactions)で一意制約をチェックするタイミングを制御します。 |
| [`tidb_ddl_disk_quota`](/system-variables.md#tidb_ddl_disk_quota-new-in-v630) | 新規追加 | [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)が有効になっている場合にのみ効果があります。 インデックスの作成時のバックフィリング中のローカルストレージの使用制限を設定します。 |
| [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630) | 新規追加 | `ADD INDEX`および`CREATE INDEX`DDL操作の加速を有効にするかどうかを制御します。 |
| [`tidb_ddl_flashback_concurrency`](/system-variables.md#tidb_ddl_flashback_concurrency-new-in-v630) | 新規追加 | `flashback cluster`の並行処理を制御します。 この変数で制御される機能は、TiDB v6.3.0では完全に機能していません。 デフォルト値を変更しないでください。 |
| [`tidb_enable_exchange_partition`](/system-variables.md#tidb_enable_exchange_partition) | 廃止予定 | [`exchange partitions with tables`](/partitioned-table.md#partition-management)機能を有効にするかどうかを制御します。 デフォルト値は`ON`で、つまり`exchange partitions with tables`がデフォルトで有効になっています。 |
| [`tidb_enable_foreign_key`](/system-variables.md#tidb_enable_foreign_key-new-in-v630) | 新規追加 | `FOREIGN KEY`機能を有効にするかどうかを制御します。 この変数で制御される機能は、TiDB v6.3.0では完全に機能していません。 デフォルト値を変更しないでください。 |
| `tidb_enable_general_plan_cache` | 新規追加 | 一般プランキャッシュ機能を有効にするかどうかを制御します。 この変数で制御される機能は、TiDB v6.3.0では完全に機能していません。 デフォルト値を変更しないでください。 |
| [`tidb_enable_metadata_lock`](/system-variables.md#tidb_enable_metadata_lock-new-in-v630) | 新規追加 | [Metadata lock](/metadata-lock.md)機能を有効にするかどうかを指定します。 |
| [`tidb_enable_null_aware_anti_join`](/system-variables.md#tidb_enable_null_aware_anti_join-new-in-v630) | 新規追加 | 特殊なセット演算子`NOT IN`および`!= ALL`によって生成されたアンチジョインに対してNull-Aware Hash Joinを適用するかどうかを制御します。 |
| [`tidb_enable_pseudo_for_outdated_stats`](/system-variables.md#tidb_enable_pseudo_for_outdated_stats-new-in-v530) | 変更済み | 統計情報が古くなっている場合に、テーブルの統計情報をオプティマイザが使用するかどうかを制御します。デフォルト値は`ON`から`OFF`に変更され、つまりテーブルの統計情報が古くなっていても、オプティマイザは引き続きそのテーブルの統計情報を使用し続けます。 |
| [`tidb_enable_rate_limit_action`](/system-variables.md#tidb_enable_rate_limit_action) | 変更済み | データを読み取る演算子のための動的メモリ制御機能を有効にするかどうかを制御します。この変数が`ON`に設定されている場合、メモリ使用量は[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)の制御下にない場合があります。そのため、デフォルト値は`ON`から`OFF`に変更されました。 |
| [`tidb_enable_tiflash_read_for_write_stmt`](/system-variables.md#tidb_enable_tiflash_read_for_write_stmt-new-in-v630) | 新たに追加 | SQL書き込み文の読み取りリクエストをTiFlashにプッシュダウンするかどうかを制御します。この変数で制御される機能はTiDB v6.3.0では完全に機能していません。デフォルト値を変更しないでください。 |
| [`tidb_enable_unsafe_substitute`](/system-variables.md#tidb_enable_unsafe_substitute-new-in-v630) | 新たに追加 | 安全でない方法で生成列の式を置換するかどうかを制御します。 |
| `tidb_general_plan_cache_size` | 新たに追加 | 一般的なプランキャッシュによってキャッシュされる実行プランの最大数を制御します。この変数で制御される機能はTiDB v6.3.0では完全に機能していません。デフォルト値を変更しないでください。 |
| [`tidb_last_plan_replayer_token`](/system-variables.md#tidb_enable_unsafe_substitute-new-in-v630) | 新たに追加 | 読み取り専用で、現在のセッションで最後に`PLAN REPLAYER DUMP`を実行した結果を取得するために使用されます。 |
| [tidb_max_paging_size](/system-variables.md#tidb_max_paging_size-new-in-v630) | 新たに追加 | コプロセッサのページングリクエスト処理中における最小行数を設定するために使用されます。 |
| [`tidb_opt_force_inline_cte`](/system-variables.md#tidb_opt_force_inline_cte-new-in-v630) | 新たに追加 | セッション全体で共通テーブル式(CTE)をインライン展開するかどうかを制御します。デフォルト値は`OFF`で、つまりデフォルトではCTEのインライン展開は強制されません。 |
| [`tidb_opt_three_stage_distinct_agg`](/system-variables.md#tidb_opt_three_stage_distinct_agg-new-in-v630) | 新たに追加 | MPPモードで`COUNT(DISTINCT)`集約を3段階の集約に書き換えるかどうかを指定します。デフォルト値は`ON`です。 |
| [`tidb_partition_prune_mode`](/system-variables.md#tidb_partition_prune_mode-new-in-v51) | 変更済み | 動的プルーニングを有効にするかどうかを指定します。v6.3.0以降、デフォルト値は`dynamic`に変更されました。 |
| [`tidb_rc_read_check_ts`](/system-variables.md#tidb_rc_read_check_ts-new-in-v600) | 変更済み | タイムスタンプの取得を最適化するために使用され、読み取りコミットされた状態の分離レベルで、読み書きの競合がまれなシナリオに適しています。この機能は特定のサービスワークロード向けであり、他のシナリオでパフォーマンスの低下を引き起こす可能性があります。このため、v6.3.0以降、この変数のスコープは`GLOBAL \| SESSION`から`INSTANCE`に変更されました。つまり、この機能を特定のTiDBインスタンスに対して有効にすることができます。 |
| [`tidb_rc_write_check_ts`](/system-variables.md#tidb_rc_write_check_ts-new-in-v630)  | 新たに追加 | 数が少ないポイント書き込みの競合があるRC分離レベルの悲観的トランザクションの実行中に、タイムスタンプの取得を最適化するために使用されます。この変数を有効にすると、グローバルなタイムスタンプの取得によって引き起こされる待ち時間とオーバーヘッドを回避することができます。 |
| [`tiflash_fastscan`](/system-variables.md#tiflash_fastscan-new-in-v630) | 新たに追加 | FastScanを有効にするかどうかを制御します。FastScanが有効になっている場合（`ON`に設定）、TiFlashはより効率的なクエリパフォーマンスを提供しますが、クエリ結果やデータの一貫性は保証されません。 |

### 設定ファイルのパラメータ

| 設定ファイル | 設定 | 変更の種類 | 説明 |
| --- | --- | --- | --- |
| TiDB | [`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630) | 新たに追加 | TiDBが一時データを格納するために使用するファイルシステムの場所を指定します。機能によってTiDBノードでローカルストレージが必要な場合、TiDBはこの場所に対応する一時データを格納します。デフォルト値は`/tmp/tidb`です。 |
| TiKV | [`auto-adjust-pool-size`](/tikv-configuration-file.md#auto-adjust-pool-size-new-in-v630) | 新たに追加 | スレッドプールのサイズを自動的に調整するかどうかを制御します。有効にすると、TiKVの読み取りパフォーマンスが現在のCPU使用状況に基づいてUnifyReadPoolスレッドプールのサイズを自動的に調整することで最適化されます。 |
| TiKV | [`data-encryption-method`](/tikv-configuration-file.md#data-encryption-method) | 変更済み | 新しい`sm4-ctr`オプションの値を導入します。この設定項目が`sm4-ctr`に設定されている場合、データは格納される前にSM4を使用して暗号化されます。 |
| TiKV | [`enable-log-recycle`](/tikv-configuration-file.md#enable-log-recycle-new-in-v630) | 新たに追加 | Raft Engineで古いログファイルをリサイクルするかどうかを決定します。有効にすると、論理的にパージされたログファイルはリサイクル用に保持されます。これにより、書き込みワークロードにおける長時間の待ち時間を減らすことができます。この設定項目は[format-version](/tikv-configuration-file.md#format-version-new-in-v630)が>= 2の場合にのみ使用可能です。 |
| TiKV | [`format-version`](/tikv-configuration-file.md#format-version-new-in-v630) | 新たに追加 | Raft Engineのログファイルのバージョンを指定します。TiKV v6.3.0より前のデフォルトログファイルバージョンは`1`です。このログファイルはTiKV v6.1.0以上で読むことができます。TiKV v6.3.0以降のデフォルトログファイルバージョンは`2`です。TiKV v6.3.0以降はこのログファイルを読むことができます。 |
| TiKV | [`log-backup.enable`](/tikv-configuration-file.md#enable-new-in-v620) | 変更済み | v6.3.0以降、デフォルト値が`false`から`true`に変更されました。 |
| TiKV | [`log-backup.max-flush-interval`](/tikv-configuration-file.md#max-flush-interval-new-in-v620) | 変更済み | v6.3.0以降、デフォルト値が`5min`から`3min`に変更されました。 |
| PD | [enable-diagnostic](/pd-configuration-file.md#enable-diagnostic-new-in-v630) | 新たに追加 | 診断機能を有効にするかどうかを制御します。デフォルト値は`false`です。 |
| TiFlash | [`dt_enable_read_thread`](/tiflash/tiflash-configuration.md#configure-the-tiflash-learnertoml-file) | 廃止予定 | この設定項目はv6.3.0以降、廃止されています。スレッドプールはデフォルトでストレージエンジンからの読み取りリクエストを処理するために使用され、無効にすることはできません。 |
| DM | [`safe-mode-duration`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced) | 新たに追加 | 自動セーフモードの期間を指定します。 |
| TiCDC | [`enable-sync-point`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 新たに追加 | Syncpoint機能を有効にするかどうかを指定します。 |
| TiCDC | [`sync-point-interval`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 新たに追加 | Syncpointが上流と下流のスナップショットを整合させる間隔を指定します。 |
| TiCDC | [`sync-point-retention`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 新たに追加 | Syncpointによって下流のテーブルでデータが保持される期間を指定します。この期間を超過するとデータがクリーンアップされます。 |
| TiCDC | [`sink-uri.memory`](/ticdc/ticdc-changefeed-config.md#changefeed-cli-parameters) | 廃止予定 | `memory`排序は廃止されており、どの状況でも使用することはお勧めしません。 |

### その他

* ログバックアップはバックアップストレージとしてGCSおよびAzure Blob Storageをサポートしています。
* ログバックアップは`exchange partition` DDLと互換性があります。
* 以前には、[fastscan](/tiflash/use-fastscan.md)を有効にするために使用されていた`ALTER TABLE ...SET TiFLASH MODE ...`のSQLステートメントは廃止され、システム変数[`tiflash_fastscan`](/system-variables.md#tiflash_fastscan-new-in-v630)に置き換えられました。v6.2.0からv6.3.0にアップグレードすると、v6.2.0のすべてのFastScan設定が無効になりますが、データの通常の読み取りには影響しません。この場合、変数[`tiflash_fastscan`](/system-variables.md#tiflash_fastscan-new-in-v630)を構成して、FastScanを有効または無効にする必要があります。以前のバージョンからv6.3.0にアップグレードする場合、データの一貫性を保つため、すべてのセッションでFastScan機能はデフォルトで有効になっていません。

* Linux AMD64アーキテクチャでTiFlashを展開するには、CPUはAVX2命令セットをサポートする必要があります。`cat /proc/cpuinfo | grep avx2`に出力があることを確認してください。Linux ARM64アーキテクチャでTiFlashを展開するには、CPUはARMv8命令セットアーキテクチャをサポートする必要があります。`cat /proc/cpuinfo | grep 'crc32' | grep 'asimd'`に出力があることを確認してください。命令セット拡張を使用することで、TiFlashのベクトル化エンジンがより優れたパフォーマンスを提供できます。

* TiDBで動作するHAProxyの最小バージョンは現在v1.5です。v1.5からv2.1までのHAProxyバージョンには、`mysql-check`で`post-41`構成オプションを設定する必要があります。HAProxy v2.2以降を使用することをお勧めします。

## 削除された機能

v6.3.0以降、TiCDCはPulsarシンクの構成をサポートしなくなりました。代替として、StreamNativeが提供する[kop](https://github.com/streamnative/kop)を使用できます。

## 改善点

+ TiDB

    - TiDBは、テーブルの存在をチェックする際に、対象テーブル名に対して大文字と小文字を区別しなくなりました [#34610](https://github.com/pingcap/tidb/issues/34610) @[tiancaiamao](https://github.com/tiancaiamao)
    - `init_connect`の値を設定する際にパースチェックを追加することで、MySQL互換性を向上させました [#35324](https://github.com/pingcap/tidb/issues/35324) @[CbcWestwolf](https://github.com/CbcWestwolf)
    - 新しい接続に対して生成されるログ警告を改善しました [#34964](https://github.com/pingcap/tidb/issues/34964) @[xiongjiwei](https://github.com/xiongjiwei)
    - DDL履歴ジョブをクエリするためのHTTP APIを最適化し、`start_job_id`パラメータをサポートしました [#35838](https://github.com/pingcap/tidb/issues/35838) @[tiancaiamao](https://github.com/tiancaiamao)
    - JSONパスが誤った構文を持つ場合にエラーを報告するようにしました [#22525](https://github.com/pingcap/tidb/issues/22525) [#34959](https://github.com/pingcap/tidb/issues/34959) @[xiongjiwei](https://github.com/xiongjiwei)
    - 偽の共有問題を修正することで、JOIN操作のパフォーマンスを向上させました [#37641](https://github.com/pingcap/tidb/issues/37641) @[gengliqi](https://github.com/gengliqi)
    - [`PLAN REPLAYER`](/sql-plan-replayer.md)を使用して複数のSQLステートメントの実行計画情報を一度にエクスポートすることをサポートし、トラブルシューティングを効率的に行うことができるようにしました [#37798](https://github.com/pingcap/tidb/issues/37798) @[Yisaer](https://github.com/Yisaer)

+ TiKV

    - `unreachable_backoff`アイテムの構成をサポートし、1つのピアが到達不能になるとRaftstoreが多くのメッセージをブロードキャストすることを避けるようにしました [#13054](https://github.com/tikv/tikv/issues/13054) @[5kbpers](https://github.com/5kbpers)
    - TSOサービスの耐障害性を向上させました [#12794](https://github.com/tikv/tikv/issues/12794) @[pingyu](https://github.com/pingyu)
    - ロックスDBで同時に行われるサブコンパクション操作の数を動的に変更することをサポートしました（`rocksdb.max-sub-compactions`） [#13145](https://github.com/tikv/tikv/issues/13145) @[ethercflow](https://github.com/ethercflow)
    - 空のリージョンのマージのパフォーマンスを最適化しました [#12421](https://github.com/tikv/tikv/issues/12421) @[tabokie](https://github.com/tabokie)
    - より多くの正規表現関数をサポートしました [#13483](https://github.com/tikv/tikv/issues/13483) @[gengliqi](https://github.com/gengliqi)
    - CPU使用率に基づいてスレッドプールサイズを自動的に調整することをサポートしました [#13313](https://github.com/tikv/tikv/issues/13313) @[glorv](https://github.com/glorv)

+ PD

    - TiDBダッシュボードでTiKV IO MBpsメトリクスをクエリする処理を改善しました [#5366](https://github.com/tikv/pd/issues/5366) @[YiniXu9506](https://github.com/YiniXu9506)
    - TiDBダッシュボードのURLを`metrics`から`monitoring`に変更しました [#5366](https://github.com/tikv/pd/issues/5366) @[YiniXu9506](https://github.com/YiniXu9506)

+ TiFlash

    - `elt`関数をTiFlashにプッシュダウンすることをサポートしました [#5104](https://github.com/pingcap/tiflash/issues/5104) @[Willendless](https://github.com/Willendless)
    - `leftShift`関数をTiFlashにプッシュダウンすることをサポートしました [#5099](https://github.com/pingcap/tiflash/issues/5099) @[AnnieoftheStars](https://github.com/AnnieoftheStars)
    - `castTimeAsDuration`関数をTiFlashにプッシュダウンすることをサポートしました [#5306](https://github.com/pingcap/tiflash/issues/5306) @[AntiTopQuark](https://github.com/AntiTopQuark)
    - `HexIntArg/HexStrArg`関数をTiFlashにプッシュダウンすることをサポートしました [#5107](https://github.com/pingcap/tiflash/issues/5107) @[YangKeao](https://github.com/YangKeao)
    - TiFlashのインタプリタをリファクタリングし、新しいインタプリタPlannerをサポートしました [#4739](https://github.com/pingcap/tiflash/issues/4739) @[SeaRise](https://github.com/SeaRise)
    - TiFlashのメモリトラッカーの精度を向上させました [#5609](https://github.com/pingcap/tiflash/issues/5609) @[bestwoody](https://github.com/bestwoody)
    - `UTF8_BIN/ASCII_BIN/LATIN1_BIN/UTF8MB4_BIN`照合を持つ文字列カラムのパフォーマンスを向上させました [#5294](https://github.com/pingcap/tiflash/issues/5294) @[solotzg](https://github.com/solotzg)
    - 読み込み制限器により、バックグラウンドでI/Oスループットを計算することをサポートしました [#5401](https://github.com/pingcap/tiflash/issues/5401), [#5091](https://github.com/pingcap/tiflash/issues/5091) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)

+ ツール

    + バックアップ＆リストア（BR）

        - PITRは、ログバックアップで生成された小さいファイルをマージし、バックアップファイルの数を大幅に削減しました [#13232](https://github.com/tikv/tikv/issues/13232) @[Leavrth](https://github.com/Leavrth)
        - PITRは、復元後にアップストリームクラスター構成に基づいてTiFlashレプリカの数を自動的に設定することをサポートしました [#37208](https://github.com/pingcap/tidb/issues/37208) @[YuJuncen](https://github.com/YuJuncen)

    + TiCDC

        - TiCDCの並行DDLフレームワークとの互換性を向上させました [#6506](https://github.com/pingcap/tiflow/issues/6506) @[lance6716](https://github.com/lance6716)
        - MySQLシンクでエラーが発生した場合に、DMLステートメントの`start ts`をロギングするサポートを追加しました [#6460](https://github.com/pingcap/tiflow/issues/6460) @[overvenus](https://github.com/overvenus)
        - TiCDCクラスターのより正確なヘルス状態を返すために、`api/v1/health` APIを強化しました [#4757](https://github.com/pingcap/tiflow/issues/4757) @[overvenus](https://github.com/overvenus)
        - シンクスループットを向上させるために、MQシンクとMySQLシンクを非同期モードで実装しました [#5928](https://github.com/pingcap/tiflow/issues/5928) @[hicqu](https://github.com/hicqu) @[hi-rustin](https://github.com/hi-rustin)
        - 廃止されたPulsarシンクを削除しました [#7087](https://github.com/pingcap/tiflow/issues/7087) @[hi-rustin](https://github.com/hi-rustin)
- チェンジフィードに関連しないDDLステートメントを破棄することでレプリケーションパフォーマンスを向上させる [#6447](https://github.com/pingcap/tiflow/issues/6447) @[asddongmen](https://github.com/asddongmen)

    + TiDBデータ移行（DM）

        - MySQL 8.0との互換性を向上させるためのデータソースとしての機能を改善する [#6448](https://github.com/pingcap/tiflow/issues/6448) @[lance6716](https://github.com/lance6716)
        - "無効な接続"が発生した場合にDDLを非同期で実行してDDLを最適化する [#4689](https://github.com/pingcap/tiflow/issues/4689) @[lyzx2001](https://github.com/lyzx2001)

    + TiDB Lightning

        - 指定されたロールを仮定して別のアカウントのS3データにアクセスするためのS3外部ストレージURL用のクエリパラメータを追加する [#36891](https://github.com/pingcap/tidb/issues/36891) @[dsdashun](https://github.com/dsdashun)

## バグ修正

+ TiDB

    - `PREPARE`ステートメントの権限チェックがスキップされる問題を修正する [#35784](https://github.com/pingcap/tidb/issues/35784) @[lcwangchao](https://github.com/lcwangchao)
    - システム変数`tidb_enable_noop_variable`を`WARN`に設定できる問題を修正する [#36647](https://github.com/pingcap/tidb/issues/36647) @[lcwangchao](https://github.com/lcwangchao)
    - 式インデックスが定義されている場合に`INFORMATION_SCHEMA.COLUMNS`テーブルの`ORDINAL_POSITION`列が正しくない可能性がある問題を修正する [#31200](https://github.com/pingcap/tidb/issues/31200) @[bb7133](https://github.com/bb7133)
    - TiDBが`MAXINT32`よりも大きいタイムスタンプにエラーを報告しない問題を修正する [#31585](https://github.com/pingcap/tidb/issues/31585) @[bb7133](https://github.com/bb7133)
    - エンタープライズプラグインを使用する際にTiDBサーバーを起動できない問題を修正する [#37319](https://github.com/pingcap/tidb/issues/37319) @[xhebox](https://github.com/xhebox)
    - `SHOW CREATE PLACEMENT POLICY`の出力が正しくない問題を修正する [#37526](https://github.com/pingcap/tidb/issues/37526) @[xhebox](https://github.com/xhebox)
    - 一時テーブルに対する`EXCHANGE PARTITION`操作の予期しない動作を修正する [#37201](https://github.com/pingcap/tidb/issues/37201) @[lcwangchao](https://github.com/lcwangchao)
    - `INFORMATION_SCHEMA.TIKV_REGION_STATUS`のクエリが正しい結果を返さない問題を修正する @[zimulala](https://github.com/zimulala)
    - ビューに対する`EXPLAIN`クエリが権限をチェックしない問題を修正する [#34326](https://github.com/pingcap/tidb/issues/34326) @[hawkingrei](https://github.com/hawkingrei)
    - JSONの`null`を`NULL`に更新できない問題を修正する [#37852](https://github.com/pingcap/tidb/issues/37852) @[YangKeao](https://github.com/YangKeao)
    - DDLジョブの`row_count`が正確でない問題を修正する [#25968](https://github.com/pingcap/tidb/issues/25968) @[Defined2014](https://github.com/Defined2014)
    - `FLASHBACK TABLE`が正しく機能しない問題を修正する [#37386](https://github.com/pingcap/tidb/issues/37386) @[tiancaiamao](https://github.com/tiancaiamao)
    - 典型的なMySQLプロトコルでの`prepared`ステートメントフラグの処理に失敗する問題を修正する [#36731](https://github.com/pingcap/tidb/issues/36731) @[dveeden](https://github.com/dveeden)
    - 一部の極端な状況で起動時に正しくないTiDBステータスが発生する問題を修正する [#36791](https://github.com/pingcap/tidb/issues/36791) @[xhebox](https://github.com/xhebox)
    - `INFORMATION_SCHEMA.VARIABLES_INFO`がセキュリティ強化モード（SEM）に準拠しない問題を修正する [#37586](https://github.com/pingcap/tidb/issues/37586) @[CbcWestwolf](https://github.com/CbcWestwolf)
    - `UNION`において文字列を文字列にキャストする際にクエリが誤る問題を修正する [#31678](https://github.com/pingcap/tidb/issues/31678) @[cbcwestwolf](https://github.com/cbcwestwolf)
    - パーティションテーブルのダイナミックモードを有効にした場合に間違った結果が返る問題を修正する [#37254](https://github.com/pingcap/tidb/issues/37254) @[wshwsh12](https://github.com/wshwsh12)
    - TiDBでのバイナリ文字列とJSONの間のキャストおよび比較がMySQLと互換性がない問題を修正する [#31918](https://github.com/pingcap/tidb/issues/31918) [#25053](https://github.com/pingcap/tidb/issues/25053) @[YangKeao](https://github.com/YangKeao)
    - TiDBでの`JSON_OBJECTAGG`および`JSON_ARRAYAGG`がMySQLとバイナリ値で互換性がない問題を修正する [#25053](https://github.com/pingcap/tidb/issues/25053) @[YangKeao](https://github.com/YangKeao)
    - JSON不透明値の比較がパニックを引き起こす問題を修正する [#37315](https://github.com/pingcap/tidb/issues/37315) @[YangKeao](https://github.com/YangKeao)
    - 単精度浮動小数点がJSON集約関数で使用できない問題を修正する [#37287](https://github.com/pingcap/tidb/issues/37287) @[YangKeao](https://github.com/YangKeao)
    - `UNION`演算子が予期しない空の結果を返す可能性がある問題を修正する [#36903](https://github.com/pingcap/tidb/issues/36903) @[tiancaiamao](https://github.com/tiancaiamao)
    - `castRealAsTime`式の結果がMySQLと一致しない問題を修正する [#37462](https://github.com/pingcap/tidb/issues/37462) @[mengxin9014](https://github.com/mengxin9014)
    - 悲観的なDML操作がユニークでないインデックスキーをロックする問題を修正する [#36235](https://github.com/pingcap/tidb/issues/36235) @[ekexium](https://github.com/ekexium)
    - `auto-commit`の変更がトランザクションのコミット動作に影響を与える問題を修正する [#36581](https://github.com/pingcap/tidb/issues/36581) @[cfzjywxk](https://github.com/cfzjywxk)
    - DML実行者を含む`EXPLAIN ANALYZE`ステートメントがトランザクションのコミットが終了する前に結果を返す問題を修正する [#37373](https://github.com/pingcap/tidb/issues/37373) @[cfzjywxk](https://github.com/cfzjywxk)
    - `UPDATE`ステートメントでの投影の除外が正しく行われない場合に`Can't find column`エラーが発生する問題を修正する  [#37568](https://github.com/pingcap/tidb/issues/37568) @[AilinKid](https://github.com/AilinKid)
    - Join Reorder操作が誤ってアウタージョイン条件を押し下げる問題を修正する [#37238](https://github.com/pingcap/tidb/issues/37238) @[AilinKid](https://github.com/AilinKid)
    - 一部のパターンで`IN`および`NOT IN`サブクエリが`Can't find column`エラーを報告する問題を修正する [#37032](https://github.com/pingcap/tidb/issues/37032) @[AilinKid](https://github.com/AilinKid)
    - `UPDATE`ステートメントに共通テーブル式（CTE）が含まれる場合に`Can't find column`が報告される問題を修正する [#35758](https://github.com/pingcap/tidb/issues/35758) @[AilinKid](https://github.com/AilinKid)
    - 正しくない`PromQL`を修正する [#35856](https://github.com/pingcap/tidb/issues/35856) @[Defined2014](https://github.com/Defined2014)

+ TiKV

    - Regionのハートビートが中断された後にPDがTiKVに再接続しない問題を修正する [#12934](https://github.com/tikv/tikv/issues/12934) @[bufferflies](https://github.com/bufferflies)
    - Raftstoreがビジーな場合にRegionが重複する可能性がある問題を修正する [#13160](https://github.com/tikv/tikv/issues/13160) @[5kbpers](https://github.com/5kbpers)
- PD

    - PDクライアントがデッドロックを引き起こす可能性がある問題を修正する[#13191](https://github.com/tikv/tikv/issues/13191) @[bufferflies](https://github.com/bufferflies) [#12933](https://github.com/tikv/tikv/issues/12933) @[BurtonQin](https://github.com/BurtonQin)
    - 暗号化が無効になっている場合に TiKV がパニックを起こす可能性がある問題を修正する[#13081](https://github.com/tikv/tikv/issues/13081) @[jiayang-zheng](https://github.com/jiayang-zheng)
    - ダッシュボードで `Unified Read Pool CPU` の表現が間違っている問題を修正する[#13086](https://github.com/tikv/tikv/issues/13086) @[glorv](https://github.com/glorv)
    - TiKV インスタンスが隔離されたネットワーク環境にある場合、TiKVサービスが数分間利用できなくなる問題を修正する[#12966](https://github.com/tikv/tikv/issues/12966) @[cosven](https://github.com/cosven)
    - TiKV が誤って `PessimisticLockNotFound` エラーを報告する問題を修正する[#13425](https://github.com/tikv/tikv/issues/13425) @[sticnarf](https://github.com/sticnarf)
    - 特定の状況で PITR がデータ損失を引き起こす可能性がある問題を修正する[#13281](https://github.com/tikv/tikv/issues/13281) @[YuJuncen](https://github.com/YuJuncen)
    - いくつかの長時間の悲観的なトランザクションが存在する場合、チェックポイントが進まなくなる問題を修正する[#13304](https://github.com/tikv/tikv/issues/13304) @[YuJuncen](https://github.com/YuJuncen)
    - TiKV が JSON内の日付時刻型（`DATETIME`、`DATE`、`TIMESTAMP`、`TIME`）と `STRING` 型を区別しない問題を修正する[#13417](https://github.com/tikv/tikv/issues/13417) @[YangKeao](https://github.com/YangKeao)
    - MySQLとの互換性に関する JSONブール値と他のJSONの値の比較に関する非互換性を修正する[#13386](https://github.com/tikv/tikv/issues/13386) [#37481](https://github.com/pingcap/tidb/issues/37481) @[YangKeao](https://github.com/YangKeao)

+ TiFlash

    - クエリがキャンセルされたときにウィンドウ関数がTiFlashをクラッシュさせる可能性がある問題を修正する [#5814](https://github.com/pingcap/tiflash/issues/5814) @[SeaRise](https://github.com/SeaRise)
    - `CAST(value AS DATETIME)`の間違ったデータ入力により高いTiFlashシステムCPUが引き起こされる問題を修正する[#5097](https://github.com/pingcap/tiflash/issues/5097) @[xzhangxian1008](https://github.com/xzhangxian1008)
    - `CAST(Real/Decimal AS time)`の結果がMySQLと一貫性がない問題を修正する[#3779](https://github.com/pingcap/tiflash/issues/3779) @[mengxin9014](https://github.com/mengxin9014)
    - ストレージ内の一部の廃止されたデータが削除できない問題を修正する[#5570](https://github.com/pingcap/tiflash/issues/5570) @[JaySon-Huang](https://github.com/JaySon-Huang)
    - ページGCがテーブルの作成をブロックする問題を修正する[#5697](https://github.com/pingcap/tiflash/issues/5697) @[JaySon-Huang](https://github.com/JaySon-Huang)
    - `NULL`値を含む列で主キーを作成した後に発生するパニックを修正する [#5859](https://github.com/pingcap/tiflash/issues/5859) @[JaySon-Huang](https://github.com/JaySon-Huang)

+ Tools

    + Backup & Restore (BR)

        - チェックポイントの情報が古くなる可能性がある問題を修正する[#36423](https://github.com/pingcap/tidb/issues/36423) @[YuJuncen](https://github.com/YuJuncen)
        - 復元中の並行処理が大きすぎてリージョンがバランスされない問題を修正する[#37549](https://github.com/pingcap/tidb/issues/37549) @[3pointer](https://github.com/3pointer)
        - 特殊文字が外部ストレージの認証キーに存在する場合にログバックアップチェックポイントがスタックする可能性がある問題を修正する[#37822](https://github.com/pingcap/tidb/issues/37822) @[YuJuncen](https://github.com/YuJuncen)
        - 外部ストレージの認証キーに特殊文字が存在する場合にバックアップと復元の失敗を引き起こす可能性がある問題を修正する[#37469](https://github.com/pingcap/tidb/issues/37469) @[MoCuishle28](https://github.com/MoCuishle28)

    + TiCDC

        - TiCDCがgrpcサービスで誤ったPDアドレスに対して不正確なエラーを返す問題を修正する[#6458](https://github.com/pingcap/tiflow/issues/6458) @[crelax](https://github.com/crelax)
        - `cdc cause cli changefeed list`コマンドが失敗したchangefeedsを返さない問題を修正する[#6334](https://github.com/pingcap/tiflow/issues/6334) @[asddongmen](https://github.com/asddongmen)
        - changefeedの初期化が失敗した場合にTiCDCが利用できない問題を修正する[#6859](https://github.com/pingcap/tiflow/issues/6859) @[asddongmen](https://github.com/asddongmen)

    + TiDB Binlog

        - Compressorがgzipに設定されているときにDrainerが正しくリクエストをPumpに送信できない問題を修正する[#1152](https://github.com/pingcap/tidb-binlog/issues/1152) @[lichunzhu](https://github.com/lichunzhu)

    + TiDB Data Migration (DM)

        - DMが`Specified key was too long`エラーを報告する問題を修正する[#5315](https://github.com/pingcap/tiflow/issues/5315) @[lance6716](https://github.com/lance6716)
        - リレーにエラーが発生した場合のgoroutineリークを修正する[#6193](https://github.com/pingcap/tiflow/issues/6193) @[lance6716](https://github.com/lance6716)
        - `collation_compatible`が`"strict"`に設定されている場合にDMが重複した照合を持つSQLを生成する問題を修正する[#6832](https://github.com/pingcap/tiflow/issues/6832) @[lance6716](https://github.com/lance6716)
        - DM-workerログでの警告メッセージ "found error when get timezone from binlog status_vars" の発生頻度を減らす[#6628](https://github.com/pingcap/tiflow/issues/6628) @[lyzx2001](https://github.com/lyzx2001)
        - latin1データがレプリケーション中に破損する可能性がある問題を修正する[#7028](https://github.com/pingcap/tiflow/issues/7028) @[lance6716](https://github.com/lance6716)

    + TiDB Lightning

        - TiDB LightningがParquetファイル内でスラッシュ、数字、または非ASCII文字で始まる列をサポートしない問題を修正する[#36980](https://github.com/pingcap/tidb/issues/36980) @[D3Hunter](https://github.com/D3Hunter)

## 貢献者

TiDBコミュニティからの以下の貢献者に感謝いたします：

- @[An-DJ](https://github.com/An-DJ)
- @[AnnieoftheStars](https://github.com/AnnieoftheStars)
- @[AntiTopQuark](https://github.com/AntiTopQuark)
- @[blacktear23](https://github.com/blacktear23)
- @[BurtonQin](https://github.com/BurtonQin)（初めての貢献者）
- @[crelax](https://github.com/crelax)
- @[eltociear](https://github.com/eltociear)
- @[fuzhe1989](https://github.com/fuzhe1989)
- @[erwadba](https://github.com/erwadba)
- @[jianzhiyao](https://github.com/jianzhiyao)
- @[joycse06](https://github.com/joycse06)
```
- @[morgo](https://github.com/morgo)
- @[onlyacat](https://github.com/onlyacat)
- @[peakji](https://github.com/peakji)
- @[rzrymiak](https://github.com/rzrymiak)
- @[tisonkun](https://github.com/tisonkun)
- @[whitekeepwork](https://github.com/whitekeepwork)
- @[Ziy1-Tan](https://github.com/Ziy1-Tan)
```