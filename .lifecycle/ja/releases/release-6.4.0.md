---
title: TiDB 6.4.0のリリースノート
---

# TiDB 6.4.0のリリースノート

リリース日: 2022年11月17日

TiDBバージョン: 6.4.0-DMR

> **注意:**
>
> TiDB 6.4.0-DMRのドキュメントは[アーカイブ](https://docs-archive.pingcap.com/tidb/v6.4/)されました。PingCAPでは、TiDBデータベースの[最新のLTSバージョン](https://docs.pingcap.com/tidb/stable)をご利用いただくことをお勧めしています。

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.4/quick-start-with-tidb) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.4.0#version-list)

v6.4.0-DMRでは、主な新機能と改善点は以下の通りです:

- [`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md)を使用して、クラスタを特定の時点に復元するサポート(実験的)。
- TiDBインスタンスの[グローバルメモリ使用量の追跡](/configure-memory-usage.md)をサポート(実験的)。
- [Linear Hashパーティショニング構文](/partitioned-table.md#how-tidb-handles-linear-hash-partitions)との互換性を持たせる。
- 高性能でグローバルで単調増加な[`AUTO_INCREMENT`](/auto-increment.md#mysql-compatibility-mode)をサポート(実験的)。
- [JSONタイプ](/data-type-json.md)の配列データの範囲選択をサポート。
- ディスク障害やI/Oが停滞するなどの極端な状況での障害回復を加速。
- テーブル結合順序を決定するための[動的計画アルゴリズム](/join-reorder.md#example-the-dynamic-programming-algorithm-of-join-reorder)を追加。
- 相関サブクエリのdecorrelationの実行可否を制御するための[新しい最適化ヒント `NO_DECORRELATE`](/optimizer-hints.md#no_decorrelate)を導入。
- [クラスタ診断](/dashboard/dashboard-diagnostics-access.md)機能がGAになる。
- TiFlashは[暗号化at rest](/encryption-at-rest.md#tiflash)のためにSM4アルゴリズムをサポート。
- 指定されたテーブルのTiFlashレプリカを即座に[コンパクト化するためのSQLステートメントのサポート](/sql-statements/sql-statement-alter-table-compact.md#compact-tiflash-replicas-of-specified-partitions-in-a-table)をサポート。
- EBSボリュームスナップショットを使用してTiDBクラスタを[バックアップ](https://docs.pingcap.com/tidb-in-kubernetes/v1.4/backup-to-aws-s3-by-snapshot)するサポート。
- DMが[ダウンストリームマージテーブルの拡張列に上流データソース情報を書き込む](/dm/dm-table-routing.md#extract-table-schema-and-source-information-and-write-into-the-merged-table)のサポート。

## 新機能

### SQL

* テーブルの指定されたパーティションのTiFlashレプリカを即座にコンパクト化するためのSQLステートメントのサポート [#5315](https://github.com/pingcap/tiflash/issues/5315) @[hehechen](https://github.com/hehechen)

    v6.2.0以降、TiDBはTiFlashのフルテーブルレプリカの物理データを即座に[コンパクト化する機能](/sql-statements/sql-statement-alter-table-compact.md#alter-table--compact)をサポートしています。TiFlashの物理データを即座にコンパクト化するためにSQLステートメントを手動で実行することで、ストレージスペースを削減し、クエリパフォーマンスを向上させることができます。v6.4.0では、TiFlashレプリカデータのコンパクト化の粒度を改良し、指定されたテーブルのTiFlashレプリカを即座にコンパクト化することをサポートします。

    SQLステートメント`ALTER TABLE table_name COMPACT [PARTITION PartitionNameList] [engine_type REPLICA]`を実行することで、指定されたテーブルのTiFlashレプリカを即座にコンパクト化することができます。

    詳細については、[ユーザドキュメント](/sql-statements/sql-statement-alter-table-compact.md#compact-tiflash-replicas-of-specified-partitions-in-a-table)を参照してください。

* `FLASHBACK CLUSTER TO TIMESTAMP`を使用して、クラスタを特定の時点に復元するサポート(実験的) [#37197](https://github.com/pingcap/tidb/issues/37197) [#13303](https://github.com/tikv/tikv/issues/13303) @[Defined2014](https://github.com/Defined2014) @[bb7133](https://github.com/bb7133) @[JmPotato](https://github.com/JmPotato) @[Connor1996](https://github.com/Connor1996) @[HuSharp](https://github.com/HuSharp) @[CalvinNeo](https://github.com/CalvinNeo)

    `FLASHBACK CLUSTER TO TIMESTAMP`構文を使用して、ガベージコレクション（GC）寿命内でクラスタを特定の時点に迅速に復元できます。この機能により、DMLの誤操作を簡単かつ迅速に元に戻すことができます。例えば、`WHERE`句のない`DELETE`を誤って実行した後、数分で元のクラスタを復元するためにこの構文を使用できます。この機能はデータベースバックアップに依存せず、異なる時点でデータをロールバックすることをサポートします。なお、`FLASHBACK CLUSTER TO TIMESTAMP`はデータベースバックアップを置き換えることはできません。

    `FLASHBACK CLUSTER TO TIMESTAMP`を実行する前に、TiCDCなどのツールで実行されているPITRとレプリケーションタスクを一時停止し、`FLASHBACK`が完了した後に再開する必要があります。そうしないと、レプリケーションタスクが失敗する可能性があります。

    詳細については、[ユーザドキュメント](/sql-statements/sql-statement-flashback-to-timestamp.md)を参照してください。

* `FLASHBACK DATABASE`を使用して、削除されたデータベースを復元するサポート [#20463](https://github.com/pingcap/tidb/issues/20463) @[erwadba](https://github.com/erwadba)

    `FLASHBACK DATABASE`を使用することで、ガベージコレクション（GC）ライフタイム内で`DROP`によって削除されたデータベースとそのデータを復元することができます。この機能は外部ツールに依存しません。SQLステートメントを使用して、迅速にデータとメタデータを復元することができます。

    詳細については、[ユーザドキュメント](/sql-statements/sql-statement-flashback-database.md)を参照してください。

### セキュリティ

* TiFlashは暗号化at restのためにSM4アルゴリズムをサポート [#5953](https://github.com/pingcap/tiflash/issues/5953) @[lidezhu](https://github.com/lidezhu)

    TiFlashの暗号化at restにSM4アルゴリズムを追加しました。暗号化at restを構成する際、`tiflash-learner.toml`構成ファイルで`data-encryption-method`の値を`sm4-ctr`に設定することで、SM4暗号化機能を有効にすることができます。

    詳細については、[ユーザドキュメント](/encryption-at-rest.md#tiflash)を参照してください。

### 可観測性

* クラスタ診断機能がGAになる [#1438](https://github.com/pingcap/tidb-dashboard/issues/1438) @[Hawkson-jee](https://github.com/Hawkson-jee)

    TiDB Dashboardの[クラスタ診断](/dashboard/dashboard-diagnostics-access.md)機能は、指定された時間範囲内でクラスタに存在する可能性がある問題を診断し、診断結果とクラスタ関連の負荷監視情報を[診断レポート](/dashboard/dashboard-diagnostics-report.md)の形でまとめます。この診断レポートはWebページ形式であり、ブラウザからページを保存して保存したページリンクを回覧することができます。

    診断レポートを使用すると、クラスタの基本的な健康情報（負荷、コンポーネントの状態、時間消費、構成など）を素早く把握することができます。クラスタに一般的な問題がある場合は、[診断情報](/dashboard/dashboard-diagnostics-report.md#diagnostic-information)の結果で原因を特定することができます。

### パフォーマンス

* コプロセッサタスクの並行性適応メカニズムを導入 [#37724](https://github.com/pingcap/tidb/issues/37724) @[you06](https://github.com/you06)

    コプロセッサタスクの数が増加するにつれて、TiKVの処理速度に応じて、TiDBは自動的に並行性を増やします（[`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency)の値を調整）し、コプロセッサタスクのキューを減らし、それにより待ち時間を短縮します。

* テーブル結合順序を決定するための動的計画アルゴリズムを導入 [#37825](https://github.com/pingcap/tidb/issues/37825) @[winoros](https://github.com/winoros)

    以前のバージョンでは、TiDBはテーブルの結合順序を決定するために貪欲アルゴリズムを使用していました。v6.4.0では、TiDBオプティマイザは[動的計画アルゴリズム](/join-reorder.md#example-the-dynamic-programming-algorithm-of-join-reorder)を導入しました。動的計画アルゴリズムは貪欲アルゴリズムよりも多くの可能な結合順序を列挙できるため、より良い実行計画を見つける可能性を高め、一部のシナリオでSQLの実行効率を向上させます。

    動的プログラミングアルゴリズムは時間を消費するため、TiDBのJoin Reorderアルゴリズムの選択は [`tidb_opt_join_reorder_threshold`](/system-variables.md#tidb_opt_join_reorder_threshold)変数によって制御されます。Join Reorderに参加するノードの数がこの閾値を超える場合、TiDBは貪欲アルゴリズムを使用します。そうでなければ、TiDBは動的プログラミングアルゴリズムを使用します。

    詳細については、[ユーザドキュメント](/join-reorder.md)を参照してください。
* インデックス接頭語はヌル値のフィルタリングをサポートしています。 [#21145](https://github.com/pingcap/tidb/issues/21145) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)

この機能は、インデックス接頭語の最適化です。テーブルの列にインデックス接頭語がある場合、SQLステートメントの列の`IS NULL`または`IS NOT NULL`条件はプレフィックスで直接フィルタリングできます。これにより、このような場合のテーブル検索を回避し、SQL実行のパフォーマンスを向上させることができます。

詳細については、[ユーザードキュメント](/system-variables.md#tidb_opt_prefix_index_single_scan-new-in-v640)を参照してください。

* TiDBチャンク再利用メカニズムを強化 [#38606](https://github.com/pingcap/tidb/issues/38606) @[keeplearning20221](https://github.com/keeplearning20221)

以前のバージョンでは、TiDBは`writechunk`関数でのみチャンクを再利用していました。TiDB v6.4.0では、チャンク再利用のメカニズムをExecutor内の演算子に拡張しました。チャンクを再利用することで、TiDBは頻繁にメモリの開放を要求する必要がなく、一部のシナリオでSQLクエリがより効率的に実行されます。システム変数[`tidb_enable_reuse_chunk`](/system-variables.md#tidb_enable_reuse_chunk-new-in-v640)を使用して、チャンクオブジェクトを再利用するかどうかを制御できます。デフォルトで有効になっています。

詳細については、[ユーザードキュメント](/system-variables.md#tidb_enable_reuse_chunk-new-in-v640)を参照してください。

* 相関サブクエリのデコレーションを実行するかどうかを制御する新しいオプティマイザヒント`NO_DECORRELATE`を導入 [#37789](https://github.com/pingcap/tidb/issues/37789) @[time-and-fate](https://github.com/time-and-fate)

デフォルトで、TiDBは常に相関サブクエリをデコレーション実行するように再構築し、通常は実行効率が向上します。しかし、一部のシナリオでは、デコレーションは実行効率を低下させます。v6.4.0では、TiDBは、指定されたクエリブロックに対してデコレーションを実行しないようにするためのオプティマイザヒント`NO_DECORRELATE`を導入し、一部のシナリオでのクエリパフォーマンスを向上させます。

詳細については、[ユーザードキュメント](/optimizer-hints.md#no_decorrelate)を参照してください。

* パーティションテーブルの統計収集のパフォーマンスを改善 [#37977](https://github.com/pingcap/tidb/issues/37977) @[Yisaer](https://github.com/Yisaer)

v6.4.0では、TiDBはパーティションテーブルの統計収集戦略を最適化しました。システム変数[`tidb_auto_analyze_partition_batch_size`](/system-variables.md#tidb_auto_analyze_partition_batch_size-new-in-v640)を使用して、並列でパーティションテーブルの統計を収集するための同時性を設定し、収集を高速化し、解析時間を短縮できます。

### 安定性

* ディスク障害やI/Oがスタックしたような極端な状況での障害復旧を加速化 [#13648](https://github.com/tikv/tikv/issues/13648) @[LykxSassinator](https://github.com/LykxSassinator)

企業ユーザーにとって、データベースの可用性は最も重要なメトリクスの1つです。しかし、複雑なハードウェア環境では、障害の迅速な検知と復旧は常にデータベース可用性の課題の1つでした。v6.4.0では、TiDBはTiKVノードの状態検出メカニズムを完全に最適化しています。ディスク障害やI/Oがスタックしたような極端な状況でも、TiDBはノードの状態を迅速に報告し、アクティブなウェイクアップメカニズムを使用してリーダー選出を事前に開始し、クラスターの自己修復を加速します。この最適化により、ディスク障害の場合にクラスターの回復時間を50%程度短縮できます。

* TiDBメモリ使用量のグローバル制御 [#37816](https://github.com/pingcap/tidb/issues/37816) @[wshwsh12](https://github.com/wshwsh12)

v6.4.0では、TiDBは実験的な機能としてTiDBインスタンスのグローバルメモリ使用量を追跡するグローバルメモリ制御を導入しました。システム変数[`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640)を使用して、グローバルメモリ使用量の上限を設定できます。メモリ使用量が閾値に達すると、TiDBはより多くの空きメモリを回収して解放しようとします。メモリ使用量が閾値を超えると、TiDBは最も高いメモリ使用量を持つSQL操作を特定し、システムの問題を回避するためにキャンセルします。

TiDBインスタンスのメモリ消費に潜在的なリスクがある場合、TiDBは事前に診断情報を収集し、指定されたディレクトリに書き込むことで問題の診断を容易にします。同時に、TiDBはメモリ使用量を表示するためのシステムテーブルビュー[`INFORMATION_SCHEMA.MEMORY_USAGE`](/information-schema/information-schema-memory-usage.md)と[`INFORMATION_SCHEMA.MEMORY_USAGE_OPS_HISTORY`](/information-schema/information-schema-memory-usage-ops-history.md)を提供し、メモリ使用量をより良く理解するのに役立ちます。

グローバルメモリ制御はTiDBメモリ管理の画期的な進歩です。インスタンスのグローバルビューを導入し、メモリの体系的な管理を採用することで、より多くの重要なシナリオでデータベースの安定性とサービスの可用性を大幅に向上させることができます。

詳細については、[ユーザードキュメント](/configure-memory-usage.md)を参照してください。

* レンジ構築オプティマイザのメモリ使用量を制御する [#37176](https://github.com/pingcap/tidb/issues/37176) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)

v6.4.0では、システム変数[`tidb_opt_range_max_size`](/system-variables.md#tidb_opt_range_max_size-new-in-v640)を導入して、レンジを構築するオプティマイザの最大メモリ使用量を制限します。メモリ使用量が限界を超えると、オプティマイザはより粗い範囲を構築するようになり、より正確な範囲よりもメモリ消費を減らすことができます。SQLステートメントに多くの`IN`条件がある場合、この最適化により、コンパイルのメモリ使用量を大幅に減らし、システムの安定性を確保できます。

詳細については、[ユーザードキュメント](/system-variables.md#tidb_opt_range_max_size-new-in-v640)を参照してください。

* 統計情報を同期的にロードする機能をサポート（GA） [#37434](https://github.com/pingcap/tidb/issues/37434) @[chrysan](https://github.com/chrysan)

TiDB v6.4.0では、統計情報の同期的なロード機能をデフォルトで有効にしました。この機能により、SQLの最適化のためにSQLステートメントを実行する際に、大規模な統計情報（ヒストグラム、TopN、Count-Min Sketchなど）をメモリに同期的にロードできます。

詳細については、[ユーザードキュメント](/system-variables.md#tidb_stats_load_sync_wait-new-in-v540)を参照してください。

* ライトウェイトトランザクション書き込みの応答時間へのバッチ書き込みリクエストの影響を軽減する [#13313](https://github.com/tikv/tikv/issues/13313) @[glorv](https://github.com/glorv)

一部のシステムのビジネスロジックでは、定期的なバッチDMLタスクが必要ですが、これらのバッチ書き込みタスクの処理はオンライントランザクションの遅延を増加させます。v6.3.0では、TiKVはハイブリッドワークロードシナリオで読み取りリクエストのスケジューリングを最適化するために、[`readpool.unified.auto-adjust-pool-size`](/tikv-configuration-file.md#auto-adjust-pool-size-new-in-v630)設定項目を有効にすることで、すべての読み取りリクエストのためにUnifyReadPoolスレッドプールのサイズを自動的に調整できました。v6.4.0では、TiKVはダイナミックに書き込みリクエストを識別し、1回のポーリングでFSM（有限状態マシン）に対してApplyスレッドが書き込む最大バイト数を制御し、バッチ書き込みリクエストがトランザクションの書き込みの応答時間にどのような影響を与えるかを軽減します。

### 使用の容易さ

* TiKV API V2が一般に利用可能になりました（GA） [#11745](https://github.com/tikv/tikv/issues/11745) @[pingyu](https://github.com/pingyu)

v6.1.0以前、TiKVはクライアントから渡される生データのみを保存しているため、基本的なキー値の読み書き機能のみを提供していました。また、異なるコーディング方法とスコープのデータ範囲により、TiDB、Transactional KV、RawKVは同じTiKVクラスター内で同時に使用できず、この場合は複数のクラスターが必要となり、機械と展開コストが増加します。

TiKV API V2は新しいRawKVストレージ形式とアクセスインターフェースを提供し、次の利点を提供します。

- データは、データの変更タイムスタンプに基づいてMVCCで保存され、Change Data Capture（CDC）が実装されています。この機能は実験的であり、[TiKV-CDC](https://github.com/tikv/migration/blob/main/cdc/README.md)で詳細に説明されています。
- データは異なる使用法に合わせて範囲指定され、API V2は単一のクラスター内でTiDB、Transactional KV、RawKVアプリケーションが共存することをサポートします。
- マルチテナンシ向けの機能をサポートするためにKey Spaceフィールドを予約します。

TiKV API V2を有効にするには、TiKVの構成ファイルの`[storage]`セクションで`api-version = 2`を設定します。

詳細については、[ユーザードキュメント](/tikv-configuration-file.md#api-version-new-in-v610)を参照してください。

* [TiFlashのデータレプリケーション進捗の精度を向上](https://github.com/pingcap/tiflash/issues/4902) @[hehechen](https://github.com/hehechen)

    TiDBでは、`INFORMATION_SCHEMA.TIFLASH_REPLICA`テーブルの`PROGRESS`フィールドが使用され、TiKVからTiFlashレプリカへのデータ複製の進捗を表します。以前のTiDBバージョンでは、`PROCESS`フィールドはTiFlashレプリカの作成中のデータ複製の進捗のみを提供していました。TiFlashレプリカが作成された後、TiKVの対応するテーブルに新しいデータがインポートされた場合、このフィールドは更新されず、新しいデータのTiKVからTiFlashへの複製進捗を表示しません。

    v6.4.0では、TiDBはTiFlashレプリカのデータレプリケーション進捗の更新メカニズムを改善しました。TiFlashレプリカが作成された後、TiKVの対応するテーブルに新しいデータがインポートされた場合、[`INFORMATION_SCHEMA.TIFLASH_REPLICA`](/information-schema/information-schema-tiflash-replica.md)テーブルの`PROGRESS`値は、新しいデータに対するTiKVからTiFlashへの実際の複製進捗を表示するように更新されます。この改善により、TiFlashのデータレプリケーションの実際の進捗を簡単に表示できます。

    詳細については、[ユーザードキュメント](/information-schema/information-schema-tiflash-replica.md)を参照してください。

### MySQL互換性

* [Linear Hashパーティショニング構文との互換性をサポート](https://github.com/pingcap/tidb/issues/38450) @[mjonss](https://github.com/mjonss)

    以前のバージョンでは、TiDBはHash、Range、およびListパーティショニングをサポートしていました。v6.4.0からは、TiDBは[MySQL Linear Hashパーティショニング](https://dev.mysql.com/doc/refman/5.7/en/partitioning-linear-hash.html)の構文とも互換性があります。

    TiDBでは、既存のMySQL Linear HashパーティションのDDLステートメントを直接実行でき、TiDBは対応するHashパーティションテーブルを作成します（TiDBにはLinear Hashパーティションはありません）。また、既存のMySQL Linear HashパーティションのDMLステートメントを直接実行でき、TiDBは対応するTiDB Hashパーティションのクエリ結果を通常返します。この機能により、TiDBの構文がMySQL Linear Hashパーティションと互換性があり、MySQLベースのアプリケーションをTiDBにシームレスに移行することができます。

    パーティションの数が2の累乗の場合、TiDBハッシュパーティションテーブル内の行は、MySQL Linear Hashパーティションテーブル内の行と同じように分散されます。それ以外の場合、TiDB内のこれらの行の分布はMySQLと異なります。

    詳細については、[ユーザードキュメント](/partitioned-table.md#how-tidb-handles-linear-hash-partitions)を参照してください。

* 高性能でグローバルに単調増加する`AUTO_INCREMENT`のサポート（実験的） [#38442](https://github.com/pingcap/tidb/issues/38442) @[tiancaiamao](https://github.com/tiancaiamao)

    TiDB v6.4.0では、`AUTO_INCREMENT` MySQL互換モードを導入しました。このモードでは、すべてのTiDBインスタンスでIDが単調増加するように中央集権型の自動インクリメントID割り当てサービスが導入されます。この機能により、自動増分IDでクエリ結果を簡単にソートできるようになります。MySQL互換モードを使用するには、テーブルを作成する際に`AUTO_ID_CACHE`を`1`に設定する必要があります。以下は例です：

    ```sql
    CREATE TABLE t (a INT AUTO_INCREMENT PRIMARY KEY) AUTO_ID_CACHE = 1;
    ```

    詳細については、[ユーザードキュメント](/auto-increment.md#mysql-compatibility-mode)を参照してください。

* JSONタイプでの配列データの範囲選択をサポート [#13644](https://github.com/tikv/tikv/issues/13644) @[YangKeao](https://github.com/YangKeao)

    v6.4.0から、TiDBではMySQL互換の[範囲選択構文](https://dev.mysql.com/doc/refman/8.0/en/json.html#json-paths)が使用できます。

    - `to`キーワードを使用することで、配列要素の開始位置と終了位置を指定し、配列内の連続する範囲の要素を選択できます。`0`を使用することで、配列内の最初の要素の位置を指定できます。例えば、`$[0 to 2]`を使用することで、配列の最初の3つの要素を選択できます。

    - `last`キーワードを使用することで、配列内の最後の要素の位置を指定でき、これにより右から左に位置を設定できます。例えば、`$[last-2 to last]`を使用することで、配列の最後の3つの要素を選択できます。

    この機能により、SQLステートメントの記述プロセスが簡素化され、JSONタイプの互換性が向上し、MySQLアプリケーションをTiDBに移行する難しさが軽減されます。

* データベースユーザの追加説明のサポートを強化 [#38172](https://github.com/pingcap/tidb/issues/38172) @[CbcWestwolf](https://github.com/CbcWestwolf)

    TiDB v6.4では、[`CREATE USER`](/sql-statements/sql-statement-create-user.md)または[`ALTER USER`](/sql-statements/sql-statement-alter-user.md)を使用して、データベースユーザに追加説明を追加できます。TiDBでは、テキストコメントを`COMMENT`を使用して追加したり、JSONフォーマットで構造化された属性のセットを`ATTRIBUTE`を使用して追加したりできます。

    さらに、TiDB v6.4.0では、[`USER_ATTRIBUTES`](/information-schema/information-schema-user-attributes.md)テーブルが追加され、ユーザコメントとユーザ属性の情報を表示できます。

    ```sql
    CREATE USER 'newuser1'@'%' COMMENT 'This user is created only for test';
    CREATE USER 'newuser2'@'%' ATTRIBUTE '{"email": "user@pingcap.com"}';
    SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES;
    ```

    ```sql
    +-----------+------+---------------------------------------------------+
    | USER      | HOST | ATTRIBUTE                                         |
    +-----------+------+---------------------------------------------------+
    | newuser1  | %    | {"comment": "This user is created only for test"} |
    | newuser1  | %    | {"email": "user@pingcap.com"}                     |
    +-----------+------+---------------------------------------------------+
    2 rows in set (0.00 sec)
    ```

   この機能により、TiDBはMySQL構文との互換性が向上し、MySQLエコシステムのツールやプラットフォームにTiDBを統合しやすくなります。

### バックアップとリストア

* EBSボリュームスナップショットを使用したTiDBクラスタのバックアップをサポート [#33849](https://github.com/pingcap/tidb/issues/33849) @[fengou1](https://github.com/fengou1)

    TiDBクラスタがEKS上に展開され、AWS EBSボリュームを使用している場合、TiDB Operatorを使用してボリュームスナップショットとメタデータをAWS S3にバックアップすることができます。

    - バックアップの影響を最小限に抑える。たとえば、QPSおよびトランザクションの遅延に影響を与えず、クラスタのCPUおよびメモリを占有しない。
    - 短時間でのデータのバックアップとリストア。たとえば、1時間以内にバックアップを完了し、2時間でデータをリストアする。

    詳細については、[ユーザードキュメント](https://docs.pingcap.com/tidb-in-kubernetes/v1.4/backup-to-aws-s3-by-snapshot)を参照してください。

### データ移行

* DMは、上流データソース情報を下流マージ済みテーブルの拡張列に書き込むことをサポート [#37797](https://github.com/pingcap/tidb/issues/37797) @[lichunzhu](https://github.com/lichunzhu)

    上流からTiDBへのシャードスキーマおよびテーブルのマージ時に、ターゲットテーブルにいくつかのフィールド（拡張列）を手動で追加し、DMタスクの構成時にそれらの値を指定できます。たとえば、上流のシャードスキーマとテーブルの名前を拡張列に指定すると、DMによって下流に書き込まれるデータにはスキーマ名とテーブル名が含まれます。下流のデータが異常に見える場合、この機能を使用して、ターゲットテーブル内のデータソース情報（スキーマ名やテーブル名など）を迅速に特定できます。

    詳細については、[マージ済みテーブルに表、スキーマ、およびソース情報を抽出し書き込む](/dm/dm-table-routing.md#extract-table-schema-and-source-information-and-write-into-the-merged-table)を参照してください。

* DMは、一部の強制チェック項目をオプション項目に変更することで事前チェックメカニズムを最適化 [#7333](https://github.com/pingcap/tiflow/issues/7333) @[lichunzhu](https://github.com/lichunzhu)

    データ移行タスクをスムーズに実行するために、DMはタスク開始時に自動的に[事前チェック](/dm/dm-precheck.md)をトリガーし、チェック結果を返します。DMは事前チェックが合格した後にのみデータ移行を開始します。

    v6.4.0では、DMは次の3つのチェック項目を強制からオプションへ変更し、事前チェックの合格率を向上させました。

    - 上流テーブルがTiDBと非互換な文字セットを使用しているかどうかのチェック。
    - 上流テーブルに主キー制約または一意キー制約があるかどうかのチェック。
    - 上流データベースのデータベースID `server_id`がプライマリセカンダリ構成で指定されているかどうかのチェック。
* DMサポートインクリメンタルマイグレーションタスク [#7393](https://github.com/pingcap/tiflow/issues/7393) @[GMHDBJD](https://github.com/GMHDBJD)

    v6.4.0から、binlogの位置やGTIDを指定せずに直接インクリメンタルマイグレーションを実行できます。DMは、タスクが開始された後に上流から生成されたbinlogファイルを自動的に取得し、これらのインクリメンタルデータを下流にマイグレーションします。これにより、ユーザーは煩雑な理解と複雑な構成から解放されます。

    詳細については、[DM高度なタスク設定ファイル](/dm/task-configuration-file-full.md)を参照してください。

* DMは、マイグレーションタスクの状態表示を追加しました [#7343](https://github.com/pingcap/tiflow/issues/7343) @[okJiang](https://github.com/okJiang)

    v6.4.0では、DMはマイグレーションタスクの性能と進捗状況をより直感的に理解し、トラブルシューティングの参考情報を提供するために、より多くのパフォーマンスおよび進行状況表示を追加しました。

    * データの取り込みおよびエクスポートのパフォーマンスを示す状態表示（バイト/秒）を追加します。
    * 下流データベースにデータを書き込むためのパフォーマンス指標をTPSからRPS（行/秒）に変更します。
    * DMフルマイグレーションタスクのデータエクスポートの進捗状況を示す進捗表示を追加します。

    これらの表示についての詳細は、[TiDBデータマイグレーションでタスクの状態をクエリする](/dm/dm-query-status.md)を参照してください。

### TiDBデータ共有サブスクリプション

- TiCDCは、`3.2.0`バージョンのKafkaにデータをレプリケートすることをサポートします [#7191](https://github.com/pingcap/tiflow/issues/7191) @[3AceShowHand](https://github.com/3AceShowHand)

    v6.4.0から、TiCDCは`3.2.0`のKafkaに[データをレプリケートする](/replicate-data-to-kafka.md)をサポートします。

## 互換性の変更

### システム変数

| 変数名 | 変更タイプ | 説明 |
|--------|------------------------------|------|
| [`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630) | 変更済み | GLOBALスコープを削除し、[`pessimistic-txn.constraint-check-in-place-pessimistic`](/tidb-configuration-file.md#constraint-check-in-place-pessimistic-new-in-v640)構成項目を使用してデフォルト値を変更できるようにします。この変数はTiDBが悲観的トランザクションで一意制約を確認するタイミングを制御します。 |
| [`tidb_ddl_flashback_concurrency`](/system-variables.md#tidb_ddl_flashback_concurrency-new-in-v640) | 変更済み | v6.4.0から有効であり、[`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md)の並行性を制御します。デフォルト値は`64`です。 |
| [`tidb_enable_clustered_index`](/system-variables.md#tidb_enable_clustered_index-new-in-v50) | 変更済み | デフォルト値を`INT_ONLY`から`ON`へ変更し、プライマリキーがデフォルトでクラスタインデックスとして作成されるようにします。|
| [`tidb_enable_paging`](/system-variables.md#tidb_enable_paging-new-in-v540) | 変更済み | デフォルト値を`OFF`から`ON`に変更し、ページングの方法をコプロセッサリクエスト送信用にデフォルトで使用するようにします。 |
| [`tidb_enable_prepared_plan_cache`](/system-variables.md#tidb_enable_prepared_plan_cache-new-in-v610) | 変更済み | SESSIONスコープを追加します。この変数は[Prepared Plan Cache](/sql-prepared-plan-cache.md)を有効にするかどうかを制御します。 |
| [`tidb_memory_usage_alarm_ratio`](/system-variables.md#tidb_memory_usage_alarm_ratio) | 変更済み | デフォルト値を`0.8`から`0.7`に変更します。この変数はtidb-serverメモリアラームをトリガするメモリ使用率の割合を制御します。 |
| [`tidb_opt_agg_push_down`](/system-variables.md#tidb_opt_agg_push_down) | 変更済み | GLOBALスコープを追加します。この変数は、最適化操作で集約関数をJoin、Projection、およびUnionAllの位置前に押し下げるかどうかを制御します。 |
| [`tidb_prepared_plan_cache_size`](/system-variables.md#tidb_prepared_plan_cache_size-new-in-v610) | 変更済み | SESSIONスコープを追加します。この変数はセッションでキャッシュできる計画の最大数を制御します。 |
| [`tidb_stats_load_sync_wait`](/system-variables.md#tidb_stats_load_sync_wait-new-in-v540) | 変更済み | デフォルト値を`0`から`100`に変更し、SQL実行が同期的に完全な列統計を待機する時間を最大で100ミリ秒にします。 |
| [`tidb_stats_load_pseudo_timeout`](/system-variables.md#tidb_stats_load_pseudo_timeout-new-in-v540) | 変更済み | デフォルト値を`OFF`から`ON`に変更し、同期的に完全な列統計のタイムアウト後にSQL最適化が擬似統計を使用するようにします。 |
| [`last_sql_use_alloc`](/system-variables.md#last_sql_use_alloc-new-in-v640) | 新規追加 | 直前のステートメントがキャッシュされたチャンクオブジェクト（チャンク割り当て）を使用するかどうかを示します。この変数は読み取り専用であり、デフォルト値は`OFF`です。 |
| [`tidb_auto_analyze_partition_batch_size`](/system-variables.md#tidb_auto_analyze_partition_batch_size-new-in-v640) | 新規追加 | 分析するパーティションテーブルの統計を自動的に収集する際にTiDBが一度に分析できるパーティションの数を指定します。デフォルト値は`1`です。|
| [`tidb_enable_external_ts_read`](/system-variables.md#tidb_enable_external_ts_read-new-in-v640) | 新規追加 | TiDBが[`tidb_external_ts`](/system-variables.md#tidb_external_ts-new-in-v640)で指定されたタイムスタンプでデータを読み取るかどうかを制御します。デフォルト値は`OFF`です。 |
| [`tidb_enable_gogc_tuner`](/system-variables.md#tidb_enable_gogc_tuner-new-in-v640) | 新規追加 | GOGC Tunerを有効にするかどうかを制御します。デフォルト値は`ON`です。 |
| [`tidb_enable_reuse_chunk`](/system-variables.md#tidb_enable_reuse_chunk-new-in-v640) | 新規追加 | TiDBがチャンクオブジェクトキャッシュを有効にするかどうかを制御します。デフォルト値は`ON`であり、TiDBはキャッシュされたチャンクオブジェクトを優先し、キャッシュにない場合にのみシステムから要求します。`OFF`の場合、TiDBは直接システムからチャンクオブジェクトを要求します。 |
| [`tidb_enable_prepared_plan_cache_memory_monitor`](/system-variables.md#tidb_enable_prepared_plan_cache_memory_monitor-new-in-v640) | 新規追加 | 実行計画のPrepared Plan Cacheにキャッシュされるメモリをカウントするかどうかを制御します。デフォルト値は`ON`です。|
| [`tidb_external_ts`](/system-variables.md#tidb_external_ts-new-in-v640) | 新規追加 | デフォルト値は`0`です。[`tidb_enable_external_ts_read`](/system-variables.md#tidb_enable_external_ts_read-new-in-v640)が`ON`に設定されている場合、TiDBはこの変数で指定されたタイムスタンプでデータを読み取ります。|
| [`tidb_gogc_tuner_threshold`](/system-variables.md#tidb_gogc_tuner_threshold-new-in-v640) | 新規追加 | GOGCのチューニングの最大メモリ閾値を指定します。メモリがこの閾値を超えると、GOGC Tunerは動作を停止します。デフォルト値は`0.6`です。 |
| [`tidb_memory_usage_alarm_keep_record_num`](/system-variables.md#tidb_memory_usage_alarm_keep_record_num-new-in-v640) | 新規追加 | tidb-serverメモリ使用量がメモリアラーム閾値を超えてアラームをトリガーした場合に、TiDBはデフォルトで最近の5回のアラームで生成されたステータスファイルを保持します。この数値をこの変数で調整できます。 |
| [`tidb_opt_prefix_index_single_scan`](/system-variables.md#tidb_opt_prefix_index_single_scan-new-in-v640) | 新規追加 | TiDBオプティマイザーが一部のフィルタ条件をプレフィックスインデックスに押し下げ、不要なテーブル検索を回避し、クエリのパフォーマンスを向上させるかどうかを制御します。デフォルト値は`ON`です。 |
| [`tidb_opt_range_max_size`](/system-variables.md#tidb_opt_range_max_size-new-in-v640) | 新規追加 | オプティマイザーがスキャン範囲を構築するためのメモリ使用量の上限を指定します。デフォルト値は`67108864`（64 MiB）です。 |
| [`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640) | 新規追加 | オプティマイザーがスキャン範囲を構築するためのメモリ使用量の上限を制御します（実験的）。デフォルト値は`0`であり、メモリ制限はありません。 |
| [`tidb_server_memory_limit_gc_trigger`](/system-variables.md#tidb_server_memory_limit_gc_trigger-new-in-v640) | 新規追加 | GCをトリガーしようとするTiDBのしきい値を制御します（実験的）。デフォルト値は`70%`です。 |
| [`tidb_server_memory_limit_sess_min_size`](/system-variables.md#tidb_server_memory_limit_sess_min_size-new-in-v640) | 新たに追加されました | メモリ制限を有効にした後、TiDBは現在のインスタンスで最も多くのメモリを使用するSQLステートメントを終了します。この変数は、終了するSQLステートメントの最小メモリ使用量を指定します。デフォルト値は `134217728` (128 MiB) です。|

### 設定ファイルパラメータ

| 設定ファイル | 設定パラメータ | 変更タイプ | 説明 |
| -------- | -------- | -------- | -------- |
| TiDB | `tidb_memory_usage_alarm_ratio` | 削除済み | この設定項目はもはや有効ではありません。 |
| TiDB | `memory-usage-alarm-ratio` | 削除済み | システム変数[`tidb_memory_usage_alarm_ratio`](/system-variables.md#tidb_memory_usage_alarm_ratio)に置き換えられました。この設定項目が TiDB v6.4.0 よりも古いバージョンで設定されている場合、アップグレード後には効果がありません。 |
| TiDB | [`pessimistic-txn.constraint-check-in-place-pessimistic`](/tidb-configuration-file.md#constraint-check-in-place-pessimistic-new-in-v640) | 新たに追加されました | システム変数[`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630) のデフォルト値を制御します。デフォルト値は `true` です。 |
| TiDB | [`tidb-max-reuse-chunk`](/tidb-configuration-file.md#tidb-max-reuse-chunk-new-in-v640) | 新たに追加されました | チャンク割り当ての最大キャッシュされたチャンクオブジェクト数を制御します。デフォルト値は `64` です。|
| TiDB | [`tidb-max-reuse-column`](/tidb-configuration-file.md#tidb-max-reuse-column-new-in-v640) | 新たに追加されました | チャンク割り当ての最大キャッシュされたカラムオブジェクト数を制御します。デフォルト値は `256` です。 |
| TiKV | [`cdc.raw-min-ts-outlier-threshold`](https://docs.pingcap.com/tidb/v6.2/tikv-configuration-file#raw-min-ts-outlier-threshold-new-in-v620) | 廃止済み | この設定項目はもはや有効ではありません。 |
| TiKV | [`causal-ts.alloc-ahead-buffer`](/tikv-configuration-file.md#alloc-ahead-buffer-new-in-v640) | 新たに追加されました | 予め割り当てられた TSO キャッシュサイズ (期間で指定)。デフォルト値は `3s` です。 |
| TiKV | [`causal-ts.renew-batch-max-size`](/tikv-configuration-file.md#renew-batch-max-size-new-in-v640)| 新たに追加されました | タイムスタンプリクエスト内の TSO の最大数を制御します。デフォルト値は `8192` です。 |
| TiKV | [`raftstore.apply-yield-write-size`](/tikv-configuration-file.md#apply-yield-write-size-new-in-v640) | 新たに追加されました | Applyスレッドが1回のポーリングでFSM（有限状態機械）に書き込むことのできる最大バイト数を制御します。デフォルト値は `32KiB` です。これはソフト制限です。 |
| PD | [`tso-update-physical-interval`](/pd-configuration-file.md#tso-update-physical-interval) | 新たに追加されました | v6.4.0から有効となり、PDがTSOの物理時間を更新する間隔を制御します。デフォルト値は `50ms` です。 |
| TiFlash | [`data-encryption-method`](/tiflash/tiflash-configuration.md#configure-the-tiflash-learnertoml-file) | 修正済み | 新しい値オプション `sm4-ctr` を導入します。この設定項目が `sm4-ctr` に設定されている場合、データは格納前にSM4を使用して暗号化されます。 |
| DM | [`routes.route-rule-1.extract-table`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced) | 新たに追加されました | シャーディングシナリオで使用され、シャードされたテーブルのソース情報を抽出するために任意的に使用されます。抽出された情報は、データソースを識別するために合併テーブルに書き込まれます。このパラメータが設定されている場合、事前にダウンストリームで合併テーブルを手動で作成する必要があります。 |
| DM | [`routes.route-rule-1.extract-schema`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced) | 新たに追加されました | シャーディングシナリオで使用され、シャードされたスキーマのソース情報を抽出するために任意的に使用されます。抽出された情報は、データソースを識別するために合併テーブルに書き込まれます。このパラメータが設定されている場合、事前にダウンストリームで合併テーブルを手動で作成する必要があります。 |
| DM | [`routes.route-rule-1.extract-source`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced) | 新たに追加されました | シャーディングシナリオで使用され、ソースインスタンス情報を抽出するために任意的に使用されます。抽出された情報は、データソースを識別するために合併テーブルに書き込まれます。このパラメータが設定されている場合、事前にダウンストリームで合併テーブルを手動で作成する必要があります。 |
| TiCDC | [`transaction-atomicity`](/ticdc/ticdc-sink-to-mysql.md#configure-sink-uri-for-mysql-or-tidb) | 修正済み | デフォルト値を `table` から `none` に変更します。この変更により、レプリケーションの遅延やOOMリスクが低減されます。さらに、TiCDCは今後、全トランザクションを分割するのではなく、一部のトランザクションのみを分割します（単一トランザクションのサイズが1024行を超える場合）。 |

### その他

- v6.4.0から、`mysql.user` テーブルには2つの新しい列 `User_attributes` と `Token_issuer` が追加されます。以前のTiDBバージョンのバックアップデータから `mysql` スキーマのシステムテーブルを [リストア](/br/br-snapshot-guide.md#restore-tables-in-the-mysql-schema) する場合、BRは `mysql.user` テーブルの `column count mismatch` エラーを報告します。`mysql` スキーマのシステムテーブルをリストアしない場合、このエラーは報告されません。
- Dumplingによって書き出されたファイルの名前が [エクスポートファイルの形式](/dumpling-overview.md#format-of-exported-files) に一致し、非圧縮形式で終わる場合（例: `test-schema-create.sql.origin` や `test.table-schema.sql.origin`）、TiDB Lightningがそれらを処理する方法が変更されます。v6.4.0より前では、インポート対象のファイルに含まれている場合、TiDB Lightningはそのようなファイルのインポートをスキップします。v6.4.0から、TiDB Lightningはそのようなファイルがサポートされていない圧縮形式を使用していると見なすため、インポートタスクは失敗します。
- v6.4.0から、`SYSTEM_VARIABLES_ADMIN` または `SUPER` 権限を持つチェンジフィードのみ、TiCDC Syncpoint機能を使用できます。

## 改善

+ TiDB

    - `lc_messages` のnoop変数を変更可能にする [#38231](https://github.com/pingcap/tidb/issues/38231) @[djshow832](https://github.com/djshow832)
    - クラスタ化された複合索引の最初の列として `AUTO_RANDOM` 列をサポートする [#38572](https://github.com/pingcap/tidb/issues/38572) @[tangenta](https://github.com/tangenta)
    - 内部トランザクションのリトライで悲観的トランザクションを使用して、リトライ失敗を避けて時間を節約する [#38136](https://github.com/pingcap/tidb/issues/38136) @[jackysp](https://github.com/jackysp)

+ TiKV

    - `apply-yield-write-size` を追加し、Applyスレッドが1回のポーリングでFSMに大容量のデータを書き込む際のRaftstoreの混雑を緩和する [#13313](https://github.com/tikv/tikv/issues/13313) @[glorv](https://github.com/glorv)
    - リージョンのリーダーを移行する前にエントリーキャッシュをウォームアップして、リーダー転送プロセス中のQPSの揺らぎを避ける [#13060](https://github.com/tikv/tikv/issues/13060) @[cosven](https://github.com/cosven)
    - `json_constains` オペレータをCoprocessorにプッシュダウンするサポートを追加する [#13592](https://github.com/tikv/tikv/issues/13592) @[lizhenhuan](https://github.com/lizhenhuan)
    - 一部のシナリオでフラッシュパフォーマンスを改善するために、`CausalTsProvider` の非同期関数を追加する [#13428](https://github.com/tikv/tikv/issues/13428) @[zeminzhou](https://github.com/zeminzhou)

+ PD

    - ホットリージョンスケジューラーのv2アルゴリズムがGAとなりました。一部のシナリオでは、v2アルゴリズムが構成された次元でのバランシングを改善し、無効なスケジューリングを低減します [#5021](https://github.com/tikv/pd/issues/5021) @[HundunDM](https://github.com/hundundm)
    - 事前タイムアウトを回避するためにオペレータステップのタイムアウトメカニズムを最適化する [#5596](https://github.com/tikv/pd/issues/5596) @[bufferflies](https://github.com/bufferflies)
    - 大規模クラスターでスケジューラのパフォーマンスを改善する [#5473](https://github.com/tikv/pd/issues/5473) @[bufferflies](https://github.com/bufferflies)
    - PDが提供しない外部タイムスタンプの使用をサポートする [#5637](https://github.com/tikv/pd/issues/5637) @[lhy1024](https://github.com/lhy1024)
+ TiFlash

    - TiFlashのMPPエラーハンドリングロジックをリファクタリングして、MPPの安定性をさらに向上させる [#5095](https://github.com/pingcap/tiflash/issues/5095) @[windtalker](https://github.com/windtalker)
    - TiFlash計算プロセスのソートを最適化し、JoinおよびAggregationのキーハンドリングを最適化する [#5294](https://github.com/pingcap/tiflash/issues/5294) @[solotzg](https://github.com/solotzg)
    - デコードのメモリ使用量を最適化し、不要な転送カラムを削除してJoinのパフォーマンスを向上させる [#6157](https://github.com/pingcap/tiflash/issues/6157) @[yibin87](https://github.com/yibin87)

+ ツール

    + TiDB ダッシュボード

        - モニタリングページでTiFlashのメトリクスを表示するサポートを追加し、そのページでのメトリクスの表示を最適化する [#1440](https://github.com/pingcap/tidb-dashboard/issues/1440) @[YiniXu9506](https://github.com/YiniXu9506)
        - 遅いクエリリストとSQLステートメントリストの結果の行数を表示する機能を追加する [#1443](https://github.com/pingcap/tidb-dashboard/issues/1443) @[baurine](https://github.com/baurine)
        - Alertmanagerが存在しない場合に、DashboardがAlertmanagerエラーを報告しないように最適化する [#1444](https://github.com/pingcap/tidb-dashboard/issues/1444) @[baurine](https://github.com/baurine)

    + バックアップ & リストア (BR)

        - メタデータのロードメカニズムを改善し、PITR中のメモリ使用量を大幅に減少させるために、必要なときだけメタデータをメモリに読み込むように改善する [#38404](https://github.com/pingcap/tidb/issues/38404) @[YuJuncen](https://github.com/YuJuncen)

    + TiCDC

        - ExchangeパーティションDDLステートメントのレプリケーションをサポートする [#639](https://github.com/pingcap/tiflow/issues/639) @[asddongmen](https://github.com/asddongmen)
        - MQシンクモジュールの非バッチ送信パフォーマンスを改善する [#7353](https://github.com/pingcap/tiflow/issues/7353) @[hi-rustin](https://github.com/hi-rustin)
        - テーブルに大量のリージョンを持つ場合のTiCDCプラーラのパフォーマンスを改善する [#7078](https://github.com/pingcap/tiflow/issues/7078) [#7281](https://github.com/pingcap/tiflow/issues/7281) @[sdojjy](https://github.com/sdojjy)
        - Syncpointが有効な場合、`tidb_enable_external_ts_read`変数を使用して下流のTiDBで過去のデータを読み取るサポートを追加する [#7419](https://github.com/pingcap/tiflow/issues/7419) @[asddongmen](https://github.com/asddongmen)
        - レプリケーションの安定性を向上させるために、デフォルトでトランザクションの分割を有効にし、safeModeを無効にする [#7505](https://github.com/pingcap/tiflow/issues/7505) @[asddongmen](https://github.com/asddongmen)

    + TiDBデータ移行 (DM)

        - `operate-source update`コマンドの不要な削除を行う [#7246](https://github.com/pingcap/tiflow/issues/7246) @[buchuitoudegou](https://github.com/buchuitoudegou)
        - 上流データベースがTiDBと互換性のないDDLステートメントを使用する場合に、DMフルインポートが失敗する問題を修正する。DDLステートメントを使用して事前にターゲットテーブルのスキーマをTiDBで作成し、インポートが成功するようにする[#37984](https://github.com/pingcap/tidb/issues/37984) @[lance6716](https://github.com/lance6716)

    + TiDB Lightning

        - スキーマファイルのスキャンロジックを最適化してスキーマファイルのスキャンを高速化する [#38598](https://github.com/pingcap/tidb/issues/38598) @[dsdashun](https://github.com/dsdashun)

## バグの修正

+ TiDB

    - 新しいインデックスを作成した後に発生する可能性のあるインデックスの不整合の問題を修正する [#38165](https://github.com/pingcap/tidb/issues/38165) @[tangenta](https://github.com/tangenta)
    - `INFORMATION_SCHEMA.TIKV_REGION_STATUS`テーブルの権限の問題を修正する [#38407](https://github.com/pingcap/tidb/issues/38407) @[CbcWestwolf](https://github.com/CbcWestwolf)
    - `mysql.tables_priv`テーブルの中の`grantor`フィールドが欠落している問題を修正する [#38293](https://github.com/pingcap/tidb/issues/38293) @[CbcWestwolf](https://github.com/CbcWestwolf)
    - 共通テーブル式のJOIN結果が間違っている可能性のある問題を修正する[#38170](https://github.com/pingcap/tidb/issues/38170) @[wjhuang2016](https://github.com/wjhuang2016)
    - 共通テーブル式のUNION結果が間違っている可能性のある問題を修正する [#37928](https://github.com/pingcap/tidb/issues/37928) @[YangKeao](https://github.com/YangKeao)
    - **transaction region num**モニタリングパネルの情報が正しくない問題を修正する [#38139](https://github.com/pingcap/tidb/issues/38139) @[jackysp](https://github.com/jackysp)
    - クエリ内の条件が誤ってプロジェクションにプッシュダウンされる問題を修正する [#35623](https://github.com/pingcap/tidb/issues/35623) @[Reminiscent](https://github.com/Reminiscent)
    - `isNullRejected`のチェック結果が`AND`および`OR`の場合に間違ったクエリ結果になる問題を修正する [#38304](https://github.com/pingcap/tidb/issues/38304) @[Yisaer](https://github.com/Yisaer)
    - `GROUP_CONCAT`の`ORDER BY`がアウタージョインを除外する際に考慮されない問題を修正する [#18216](https://github.com/pingcap/tidb/issues/18216) @[winoros](https://github.com/winoros)
    - 誤ってプッシュダウンされた条件がJoin Reorderによって破棄されると誤ったクエリ結果が発生する問題を修正する [#38736](https://github.com/pingcap/tidb/issues/38736) @[winoros](https://github.com/winoros)

+ TiKV

    - 複数の`cgroup`および`mountinfo`レコードがあるとTiDBの起動に失敗する問題を修正する [#13660](https://github.com/tikv/tikv/issues/13660) @[tabokie](https://github.com/tabokie)
    - TiKVメトリクス `tikv_gc_compaction_filtered`の誤った表現を修正する [#13537](https://github.com/tikv/tikv/issues/13537) @[Defined2014](https://github.com/Defined2014)
    - 異常な`delete_files_in_range`によるパフォーマンス問題を修正する [#13534](https://github.com/tikv/tikv/issues/13534) @[tabokie](https://github.com/tabokie)
    - スナップショット取得時の有効期限切れのリースによる異常な領域競合を修正する[#13553](https://github.com/tikv/tikv/issues/13553) @[SpadeA-Tang](https://github.com/SpadeA-Tang)
    - 最初のバッチで`FLASHBACK`が失敗するとエラーが発生する問題を修正する [#13672](https://github.com/tikv/tikv/issues/13672) [#13704](https://github.com/tikv/tikv/issues/13704) [#13723](https://github.com/tikv/tikv/issues/13723) @[HuSharp](https://github.com/HuSharp)

+ PD

    - ストリームのタイムアウトが正確でない問題を修正し、リーダースイッチを高速化することで修正する [#5207](https://github.com/tikv/pd/issues/5207) @[CabinfeverB](https://github.com/CabinfeverB)

+ TiFlash

    - PageStorageのGCがページ削除マーカーを適切にクリアしないと、大きすぎるWALファイルによるOOMの問題を修正する [#6163](https://github.com/pingcap/tiflash/issues/6163) @[JaySon-Huang](https://github.com/JaySon-Huang)

+ ツール

    + TiDB ダッシュボード

        - 複雑なSQLステートメントの実行計画をクエリ中に問い合わせた際にTiDBのOOMの問題を修正する [#1386](https://github.com/pingcap/tidb-dashboard/issues/1386) @[baurine](https://github.com/baurine)
        - Top SQL スイッチが NgMonitoring が PD ノードへの接続を失うときに効果を発揮しない可能性がある問題を修正しました [#164](https://github.com/pingcap/ng-monitoring/issues/164) @[zhongzc](https://github.com/zhongzc)

    + バックアップ & リストア (BR)

        - リストア処理中に PD リーダースイッチによって引き起こされる復元の失敗問題を修正しました [#36910](https://github.com/pingcap/tidb/issues/36910) @[MoCuishle28](https://github.com/MoCuishle28)
        - ログバックアップタスクを一時停止できない問題を修正しました [#38250](https://github.com/pingcap/tidb/issues/38250) @[joccau](https://github.com/joccau)
        - BR がログバックアップデータを誤って削除する問題を修正しました [#38939](https://github.com/pingcap/tidb/issues/38939) @[Leavrth](https://github.com/leavrth)
        - Azure Blob Storage や Google Cloud Storage に保存されたログバックアップデータを初めて削除する際に BR がデータを正しく削除できない問題を修正しました [#38229](https://github.com/pingcap/tidb/issues/38229) @[Leavrth](https://github.com/leavrth)

    + TiCDC

        - `changefeed query` の結果における `sasl-password` がマスクされない問題を修正しました [#7182](https://github.com/pingcap/tiflow/issues/7182) @[dveeden](https://github.com/dveeden)
        - etcd トランザクションで過剰な操作がコミットされると TiCDC が利用できなくなる可能性がある問題を修正しました [#7131](https://github.com/pingcap/tiflow/issues/7131) @[asddongmen](https://github.com/asddongmen)
        - リドゥログが誤って削除される可能性がある問題を修正しました [#6413](https://github.com/pingcap/tiflow/issues/6413) @[asddongmen](https://github.com/asddongmen)
        - Kafka Sink V2 でワイドテーブルのレプリケーションにおけるパフォーマンスの低下を修正しました [#7344](https://github.com/pingcap/tiflow/issues/7344) @[hi-rustin](https://github.com/hi-rustin)
        - チェックポイントの ts が誤って進められる可能性がある問題を修正しました [#7274](https://github.com/pingcap/tiflow/issues/7274) @[hi-rustin](https://github.com/hi-rustin)
        - マウンターモジュールの不適切なログレベルによって過剰なログが出力される問題を修正しました [#7235](https://github.com/pingcap/tiflow/issues/7235) @[hi-rustin](https://github.com/hi-rustin)
        - TiCDC クラスタに二つのオーナーが存在する可能性がある問題を修正しました [#4051](https://github.com/pingcap/tiflow/issues/4051) @[asddongmen](https://github.com/asddongmen)

    + TiDB データ移行 (DM)

        - DM WebUI が誤った `allow-list` パラメータを生成する問題を修正しました [#7096](https://github.com/pingcap/tiflow/issues/7096) @[zoubingwu](https://github.com/zoubingwu)
        - DM-worker が起動または終了する際にデータ競合が発生する可能性がある問題を修正しました [#6401](https://github.com/pingcap/tiflow/issues/6401) @[liumengya94](https://github.com/liumengya94)
        - `UPDATE` や `DELETE` ステートメントのデータが存在しない場合にイベントを無視する問題を修正しました [#6383](https://github.com/pingcap/tiflow/issues/6383) @[GMHDBJD](https://github.com/GMHDBJD)
        - `query-status` コマンドを実行した後に `secondsBehindMaster` フィールドが表示されない問題を修正しました [#7189](https://github.com/pingcap/tiflow/issues/7189) @[GMHDBJD](https://github.com/GMHDBJD)
        - チェックポイントの更新が大きなトランザクションを引き起こす可能性がある問題を修正しました [#5010](https://github.com/pingcap/tiflow/issues/5010) @[lance6716](https://github.com/lance6716)
        - フルタスクモードでタスクが同期段階に入ってすぐに失敗すると、DM が上流テーブルのスキーマ情報を失う可能性がある問題を修正しました [#7159](https://github.com/pingcap/tiflow/issues/7159) @[lance6716](https://github.com/lance6716)
        - 一貫性チェックが有効になっているとデッドロックが発生する可能性がある問題を修正しました [#7241](https://github.com/pingcap/tiflow/issues/7241) @[buchuitoudegou](https://github.com/buchuitoudegou)
        - タスクの事前チェックに `INFORMATION_SCHEMA` テーブルの `SELECT` 権限が必要な問題を修正しました [#7317](https://github.com/pingcap/tiflow/issues/7317) @[lance6716](https://github.com/lance6716)
        - 空の TLS 構成によってエラーが発生する問題を修正しました [#7384](https://github.com/pingcap/tiflow/issues/7384) @[liumengya94](https://github.com/liumengya94)

    + TiDB Lightning

        - Apache Parquet ファイルをバイナリエンコード形式の列を含む対象テーブルにインポートする際の性能低下を修正しました [#38351](https://github.com/pingcap/tidb/issues/38351) @[dsdashun](https://github.com/dsdashun)

    + TiDB Dumpling

        - 多くのテーブルをエクスポートする際に Dumpling がタイムアウトする可能性がある問題を修正しました [#36549](https://github.com/pingcap/tidb/issues/36549) @[lance6716](https://github.com/lance6716)
        - 一貫性ロックが有効になっているが上流にターゲットテーブルが存在しない場合にロックエラーが報告される問題を修正しました [#38683](https://github.com/pingcap/tidb/issues/38683) @[lance6716](https://github.com/lance6716)

## 貢献者

TiDB コミュニティからの以下の貢献者に感謝いたします:

- [645775992](https://github.com/645775992)
- [An-DJ](https://github.com/An-DJ)
- [AndrewDi](https://github.com/AndrewDi)
- [erwadba](https://github.com/erwadba)
- [fuzhe1989](https://github.com/fuzhe1989)
- [goldwind-ting](https://github.com/goldwind-ting) (初めての貢献者)
- [h3n4l](https://github.com/h3n4l)
- [igxlin](https://github.com/igxlin) (初めての貢献者)
- [ihcsim](https://github.com/ihcsim)
- [JigaoLuo](https://github.com/JigaoLuo)
- [morgo](https://github.com/morgo)
- [Ranxy](https://github.com/Ranxy)
- [shenqidebaozi](https://github.com/shenqidebaozi) (初めての貢献者)
- [taofengliu](https://github.com/taofengliu) (初めての貢献者)
- [TszKitLo40](https://github.com/TszKitLo40)
- [wxbty](https://github.com/wxbty) (初めての貢献者)
- [zgcbj](https://github.com/zgcbj)