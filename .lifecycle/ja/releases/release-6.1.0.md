---
title: TiDB 6.1.0 リリースノート
summary: TiDB 6.1.0 の新機能、互換性の変更、改善、およびバグ修正について学びます。
---

# TiDB 6.1.0 リリースノート

リリース日: 2022年6月13日

TiDB バージョン: 6.1.0

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.1/quick-start-with-tidb) | [プロダクションデプロイ](https://docs.pingcap.com/tidb/v6.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.1.0#version-list)

6.1.0 では、主な新機能や改善点は次のとおりです:

* リストパーティショニングおよびリストCOLUMNSパーティショニングがGAになり、MySQL 5.7 と互換性があります
* TiFlash パーティションテーブル (ダイナミックプルーニング) がGAになりました
* ユーザーレベルのロック管理をサポートし、MySQL と互換性があります
* トランザクションでない DML ステートメントをサポートします (`DELETE` のみをサポート)
* TiFlash がオンデマンドデータ圧縮をサポート
* MPP がウィンドウ関数フレームワークを導入
* TiCDC が Avro 経由で Kafka に変更ログのレプリケーションをサポート
* TiCDC がレプリケーション中に大規模なトランザクションを分割することをサポートし、大規模なトランザクションによるレプリケーションの遅延を大幅に削減
* シャーディングテーブルのマージと移行の楽観モードがGAになりました

## 新機能

### SQL

* リストパーティショニングとリストCOLUMNSパーティショニングがGAになりました。どちらも MySQL 5.7 と互換性があります。

    ユーザードキュメント: [リストパーティショニング](/partitioned-table.md#list-partitioning), [リストCOLUMNSパーティショニング](/partitioned-table.md#list-columns-partitioning)

* TiFlash がコンパクトコマンドを始動することをサポートします (実験的)

    TiFlash v6.1.0 では、既存のバックグラウンドコンパクションメカニズムに基づいて物理データを手動で圧縮する `ALTER TABLE ... COMPACT` ステートメントが導入されました。このステートメントを使用すると、必要に応じてデータを早期の形式に更新し、いつでも読み書きパフォーマンスを改善できます。v6.1.0 にクラスタをアップグレードする際には、このステートメントを実行してデータを圧縮することが推奨されます。このステートメントは標準のSQL構文の拡張であり、したがってMySQLクライアントと互換性があります。通常、TiFlashのアップグレード以外のシナリオでは、このステートメントを使用する必要はありません。

    [ユーザードキュメント](/sql-statements/sql-statement-alter-table-compact.md), [#4145](https://github.com/pingcap/tiflash/issues/4145)

* TiFlash がウィンドウ関数フレームワークを実装し、以下のウィンドウ関数をサポートします:

    * `RANK()`
    * `DENSE_RANK()`
    * `ROW_NUMBER()`

    [ユーザードキュメント](/tiflash/tiflash-supported-pushdown-calculations.md), [#33072](https://github.com/pingcap/tidb/issues/33072)

### 可観測性

* Continuous Profiling がARMアーキテクチャとTiFlashをサポートします。

    [ユーザードキュメント](/dashboard/continuous-profiling.md)

* Grafana はパフォーマンス概要ダッシュボードを追加し、全体のパフォーマンス診断のためのシステムレベルのエントリを提供します。

    TiDBの可視化監視コンポーネントGrafanaの新しいダッシュボードであるパフォーマンス概要は、データベース時間の分解と異なる色でこれらの指標を表示することで、全体のパフォーマンスボトルネックを一目で特定できます。これにより、パフォーマンス診断時間が大幅に短縮され、パフォーマンス分析と診断が容易になります。

    [ユーザードキュメント](/performance-tuning-overview.md)

### パフォーマンス

* カスタマイズされたリージョンサイズをサポート (実験的)

    リージョンをより大きなサイズに設定すると、リージョンの数が効果的に減少し、リージョンを管理しやすくなり、クラスタのパフォーマンスと安定性が向上します。この機能は、リージョン内のより小さい範囲であるバケットの概念を導入します。リージョンがより大きなサイズに設定されている場合、バケットをクエリ単位として使用することで、並行クエリのパフォーマンスが最適化されます。バケットをクエリ単位として使用することで、スケジュールの効率性と負荷バランスが確保され、ホットなリージョンのサイズも動的に調整されます。この機能は現在実験的です。プロダクション環境で使用することはお勧めしません。

    [ユーザードキュメント](/tune-region-performance.md), [#11515](https://github.com/tikv/tikv/issues/11515)

* デフォルトのログストレージエンジンとしてRaft Engineをサポート

    v6.1.0 以降、TiDB はログのデフォルトストレージエンジンとして Raft Engine を使用します。RocksDBと比較して、Raft Engine はTiKVの I/O 書き込みトラフィックを最大40%、CPU使用率を10%削減し、一部の負荷下で前面スループットを約5%向上させ、テールレイテンシを20%削減します。

    [ユーザードキュメント](/tikv-configuration-file.md#raft-engine), [#95](https://github.com/tikv/raft-engine/issues/95)

* ジョイン順序ヒント構文をサポート

    * `LEADING` ヒントは、オプティマイザに指定された順序をジョイン操作の接頭辞として使用するようにリマインドします。適切なジョインの接頭辞を使用することで、クエリのパフォーマンスが向上します。
    * `STRAIGHT_JOIN` ヒントは、`FROM` 句のテーブルの順序と一貫性のある順序でテーブルを結合するようにオプティマイザにリマインドします。

    これにより、テーブル結合の順序を修正する方法が提供されます。ヒントの適切な使用により、SQLのパフォーマンスとクラスタの安定性が効果的に向上します。

    ユーザードキュメント: [`LEADING`](/optimizer-hints.md#leadingt1_name--tl_name-), [`STRAIGHT_JOIN`](/optimizer-hints.md#straight_join), [#29932](https://github.com/pingcap/tidb/issues/29932)

* TiFlash がさらに4つの関数をサポート

    * `FROM_DAYS`
    * `TO_DAYS`
    * `TO_SECONDS`
    * `WEEKOFYEAR`

    [ユーザードキュメント](/tiflash/tiflash-supported-pushdown-calculations.md), [#4679](https://github.com/pingcap/tiflash/issues/4679), [#4678](https://github.com/pingcap/tiflash/issues/4678), [#4677](https://github.com/pingcap/tiflash/issues/4677)

* TiFlash がダイナミックプルーニングモードでのパーティションテーブルをサポート

    OLAPシナリオでのパフォーマンスを向上させるために、ダイナミックプルーニングモードがパーティションテーブルにサポートされています。TiDB が v6.0.0 より前のバージョンからアップグレードされた場合、既存のパーティションテーブルの統計情報を手動で更新して、パフォーマンスを最大限に引き出すことを推奨します (v6.1.0 へのアップグレード後に新規インストールや新しいパーティションには必要ありません)。

    ユーザードキュメント: [MPPモードでのパーティションテーブルへのアクセス](/tiflash/use-tiflash-mpp-mode.md#access-partitioned-tables-in-the-mpp-mode), [ダイナミックプルーニングモード](/partitioned-table.md#dynamic-pruning-mode), [#3873](https://github.com/pingcap/tiflash/issues/3873)

### 安定性

* SST破損からの自動回復をサポート

    RocksDB がバックグラウンドで損傷したSSTファイルを検出した場合、TiKV は影響を受けるピアのスケジュールを試み、他のレプリカを使用してデータを回復します。 `background-error-recovery-window` パラメータを使用して回復の最大許容時間を設定できます。回復操作が時間ウィンドウ内に完了しない場合、TiKV はパニックを起こします。この機能により、修復可能な損傷したストレージを自動的に検出および回復するため、クラスタの安定性が向上します。

    [ユーザードキュメント](/tikv-configuration-file.md#background-error-recovery-window-new-in-v610), [#10578](https://github.com/tikv/tikv/issues/10578)

* トランザクショナルでないDMLステートメントをサポート

    大規模なデータ処理のシナリオでは、大規模なトランザクションを持つ単一のSQLステートメントがクラスタの安定性とパフォーマンスに悪影響を及ぼすことがあります。v6.1.0 以降、TiDB は、`DELETE` ステートメントを複数のステートメントに分割してバッチ処理を行う構文を提供することをサポートします。分割されたステートメントはトランザクションのアトミック性と分離性を犠牲にしますが、クラスタの安定性を大幅に向上させます。詳細な構文については、[`BATCH`](/sql-statements/sql-statement-batch.md) を参照してください。

    [ユーザードキュメント](/non-transactional-dml.md)

* TiDB が最大GC待機時間の設定をサポート

    TiDB のトランザクションは、マルチバージョン同時実行制御（MVCC）メカニズムを採用しています。新しく書き込まれたデータが古いデータを上書きする際、古いデータは置換されず、両方のデータのバージョンが保持されます。古いデータはGarbage Collection（GC）タスクによって定期的にクリーンアップされ、ストレージスペースを回収し、クラスタのパフォーマンスと安定性を向上させます。デフォルトでは、GC は10分ごとにトリガーされます。長時間実行されるトランザクションが対応する過去のデータにアクセスできるようにするため、実行中のトランザクションがある場合、GC タスクは遅延されます。GC タスクの最大遅延時間を制御するために、TiDB はシステム変数 [`tidb_gc_max_wait_time`](/system-variables.md#tidb_gc_max_wait_time-new-in-v610) を導入します。最大遅延時間を超過すると、GC は強制的に実行されます。変数のデフォルト値は24時間です。この機能により、GC待機時間と長時間実行されるトランザクションとの関係を制御し、クラスタの安定性が向上します。

    [ユーザードキュメント](/system-variables.md#tidb_gc_max_wait_time-new-in-v610)
* TiDBは自動統計収集タスクの最大実行時間を設定することができます

   データベースは統計を収集することで、データの分布を効果的に把握することができ、これにより合理的な実行計画を生成し、SQLの実行効率を向上させることができます。TiDBは定期的に背景でデータの変更が頻繁に行われるデータオブジェクトの統計を収集します。しかし、統計の収集はクラスターリソースを消費し、業務ピーク時にビジネスの安定した運用に影響を与える可能性があります。

   v6.1.0から、TiDBは背景統計収集の最大実行時間を制御する[`tidb_max_auto_analyze_time`](/system-variables.md#tidb_max_auto_analyze_time-new-in-v610)を導入し、デフォルトで12時間に設定されています。アプリケーションがリソースボトルネックに遭遇しない場合、TiDBが適切に統計を収集できるように、この変数を変更しないことをお勧めします。

   [ユーザードキュメント](/system-variables.md)

### 使いやすさ

* 複数のレプリカが失われたときのワンストップオンラインデータ復旧をサポート

   TiDB v6.1.0以前、複数のリージョンレプリカがマシンの障害によって失われた場合、ユーザーはすべてのTiKVサーバーを停止し、TiKV Controlを使用してTiKVを1つずつ復旧する必要がありました。TiDB v6.1.0以降、復旧プロセスは完全に自動化されており、TiKVを停止する必要はなく、他のアプリケーションに影響を与えることはありません。復旧プロセスは、PD Controlを使用してトリガーでき、よりユーザーフレンドリーな概要情報を提供します。

   [ユーザードキュメント](/online-unsafe-recovery.md), [#10483](https://github.com/tikv/tikv/issues/10483)

* 履歴統計収集タスクの表示をサポート

   `SHOW ANALYZE STATUS`ステートメントを使用してクラスターレベルの統計収集タスクを表示できます。TiDB v6.1.0以前では、`SHOW ANALYZE STATUS`ステートメントはインスタンスレベルのタスクのみを表示し、TiDBの再起動後に履歴タスクレコードがクリアされます。そのため、過去の統計収集の時間と詳細を表示することはできませんでした。TiDB v6.1.0以降、統計収集タスクの履歴レコードは永続化され、クラスターの再起動後にクエリすることができるようになり、統計の異常によるクエリパフォーマンスの問題のトラブルシューティングの参考情報を提供します。

   [ユーザードキュメント](/sql-statements/sql-statement-show-analyze-status.md)

* TiDB、TiKV、TiFlashの設定を動的に変更する機能をサポート

   以前のTiDBバージョンでは、構成項目を変更した後、変更を有効にするためにクラスターを再起動する必要がありました。これはオンラインサービスを中断する可能性があります。この問題に対処するため、TiDB v6.1.0は動的構成機能を導入し、クラスターを再起動せずにパラメータの変更を検証できるようにする具体的な最適化が行われています。

    * 一部のTiDB構成項目をシステム変数に変換し、動的に変更および永続化できるようにします。なお、変換後は元の構成項目は非推奨となります。変換された構成項目の詳細なリストについては、[Configuration file parameters](#configuration-file-parameters)を参照してください。
    * 一部のTiKVパラメータのオンライン構成をサポートします。パラメータの詳細なリストについては、[Others](#others)を参照してください。
    * TiFlashの構成項目`max_threads`をシステム変数`tidb_max_tiflash_threads`に変換し、構成を動的に変更および永続化できるようにします。なお、変換後も元の構成項目は残ります。

   以前のバージョンからアップグレードされた（オンラインおよびオフラインアップグレードを含む）v6.1.0クラスターの場合は、次の点に注意してください。

    * アップグレード前に構成ファイルで指定された構成項目がすでに存在する場合、TiDBはアップグレードプロセス中に自動的に構成項目の値を対応するシステム変数の値に更新します。このようにして、アップグレード後、パラメータの最適化によってシステムの動作に影響を与えなくなります。
    * 上記の自動更新は、アップグレード中の1回のみ行われます。アップグレード後、非推奨の構成項目はもはや有効ではありません。

   この機能により、システムを再起動しサービスを中断する代わりに、パラメータを動的に変更し、それを検証および永続化できます。これにより、日常のメンテナンスが容易になります。

   [ユーザードキュメント](/dynamic-config.md)

* クエリまたは接続をグローバルに殺す機能をサポート

    `enable-global-kill`構成を使用してグローバルKill機能を制御できます（デフォルトで有効）。

    TiDB v6.1.0以前の場合、ある操作が多くのリソースを消費し、クラスターの安定性に問題を引き起こす場合、ターゲットのTiDBインスタンスに接続して `KILL TIDB ${id};`コマンドを実行してターゲットの接続および操作を終了する必要がありました。多くのTiDBインスタンスの場合、この方法は使用しにくく、誤った操作を行いやすくなります。v6.1.0からは、`enable-global-kill`構成が導入され、デフォルトで有効化されています。任意のTiDBインスタンスでkillコマンドを実行して、プロキシを介してクライアントとTiDBの間に誤って他のクエリまたはセッションを誤って終了させる心配をせずに、指定された接続と操作を終了することができます。現時点では、TiDBはCtrl+Cを使用してクエリまたはセッションを終了することをサポートしていません。

    [ユーザードキュメント](/tidb-configuration-file.md#enable-global-kill-new-in-v610), [#8854](https://github.com/pingcap/tidb/issues/8854)

* TiKV API V2 (実験的)

    v6.1.0以前、TiKVを使用するとRaw Key Valueストレージとして、TiKVはクライアントから渡される生データの基本的なキー値の読み書き機能のみを提供していました。

    TiKV API V2は、新しいRaw Key Valueストレージフォーマットとアクセスインタフェースを提供し、次のような機能を含みます。

    * データはMVCCで保管され、データの変更タイムスタンプが記録されます。この機能により、Change Data Captureや増分バックアップとリストアを実装するための基盤が整います。
    * データは異なる使用方法に応じてスコープが付けられ、単一のTiDBクラスター、トランザクショナルKV、RawKVアプリケーションの共存をサポートします。

  <Warning>

  根本的なストレージ形式の大きな変更のため、API V2を有効にした後は、TiKVクラスターをv6.1.0より前のバージョンにダウングレードすることはできません。TiKVのダウングレードはデータの破損を引き起こす可能性があります。

  </Warning>

    [ユーザードキュメント](/tikv-configuration-file.md#api-version-new-in-v610), [#11745](https://github.com/tikv/tikv/issues/11745)

### MySQL互換性

* MySQLとのユーザーレベルロック管理の互換性をサポート

    ユーザーレベルロックは、MySQLが組み込み関数を通じて提供するユーザー名付きのロック管理システムです。ロック関数は、ロックのブロック、待ち、およびその他のロック管理機能を提供できます。ユーザーレベルロックは、Rails、Elixir、およびEctoなどのORMフレームワークでも広く使用されています。v6.1.0以降、TiDBはMySQL互換のユーザーレベルロック管理をサポートし、`GET_LOCK`、`RELEASE_LOCK`、`RELEASE_ALL_LOCKS`関数をサポートします。

    [ユーザードキュメント](/functions-and-operators/locking-functions.md), [#14994](https://github.com/pingcap/tidb/issues/14994)

### データ移行

* 分割されたテーブルのマージおよび移行のための楽観的モードがGAとなります

    DMは、楽観的モードで分割されたテーブルのデータをマージおよび移行するタスクのシナリオテストを大量に追加し、日常的な使用の90%をカバーしています。悲観的モードと比較して、楽観的モードはよりシンプルで効率的に使用できます。使用方法に慣れてから、楽観的モードをできるだけ使用することをお勧めします。

    [ユーザードキュメント](/dm/feature-shard-merge-optimistic.md#restrictions)

* DM WebUIは指定されたパラメータに従ってタスクを開始する機能をサポートします

    マイグレーションタスクを開始する際に、開始時間と安全モードの期間を指定できます。これは、多くのソースを持つ増分マイグレーションタスクを作成する場合に特に便利であり、各ソースに対して明示的にバイナリログの開始位置を指定する必要がなくなります。

    [ユーザードキュメント](/dm/dm-webui-guide.md), [#5442](https://github.com/pingcap/tiflow/issues/5442)

### TiDBデータ共有サブスクリプション

* TiDBはさまざまなサードパーティーデータエコシステムとのデータ共有をサポートします

    * TiCDCは、TiDBのインクリメンタルデータをAvro形式でKafkaに送信することをサポートし、Confluentを介してKSQLやSnowflakeなどのサードパーティーとデータ共有が可能です。

        [ユーザードキュメント](/ticdc/ticdc-avro-protocol.md), [#5338](https://github.com/pingcap/tiflow/issues/5338)

    * TiCDCは、Canal-json形式と組み合わせて、TiDBから異なるKafkaトピックにインクリメンタルデータをテーブルごとに送信することをサポートし、これによりFlinkと直接データを共有できます。

        [ユーザードキュメント](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink), [#4423](https://github.com/pingcap/tiflow/issues/4423)

    * TiCDCは、SASL GSSAPI認証タイプをサポートし、Kafkaを使用したSASL認証の例を追加します。

        [ユーザードキュメント](/ticdc/ticdc-sink-to-kafka.md#ticdc-uses-the-authentication-and-authorization-of-kafka), [#4423](https://github.com/pingcap/tiflow/issues/4423)

* TiCDCは`charset=GBK`テーブルのレプリケーションをサポートします。

    [ユーザードキュメント](/character-set-gbk.md#component-compatibility), [#4806](https://github.com/pingcap/tiflow/issues/4806)

## 互換性の変更

### システム変数

| 変数名 | 変更タイプ | 説明 |
|---|---|---|
| [`tidb_enable_list_partition`](/system-variables.md#tidb_enable_list_partition-new-in-v50) | 変更 | デフォルト値が `OFF` から `ON` に変更されました。 |
| [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) | 変更 | この変数が GLOBAL スコープを追加し、変数の値はクラスタに保持されるようになりました。 |
| [`tidb_query_log_max_len`](/system-variables.md#tidb_query_log_max_len) | 変更 | 変数スコープが INSTANCE から GLOBAL に変更されました。変数の値はクラスタに保持され、値の範囲も `[0, 1073741824]` に変更されました。 |
| [`require_secure_transport`](/system-variables.md#require_secure_transport-new-in-v610) | 追加 | この設定は以前は `tidb.toml` のオプションでした（`security.require-secure-transport`）。ただし TiDB v6.1.0 からはシステム変数に変更されました。 |
| [`tidb_committer_concurrency`](/system-variables.md#tidb_committer_concurrency-new-in-v610) | 追加 | この設定は以前は `tidb.toml` のオプションでした（`performance.committer-concurrency`）。ただし TiDB v6.1.0 からはシステム変数に変更されました。 |
| [`tidb_enable_auto_analyze`](/system-variables.md#tidb_enable_auto_analyze-new-in-v610) | 追加 | この設定は以前は `tidb.toml` のオプションでした（`run-auto-analyze`）。ただし TiDB v6.1.0 からはシステム変数に変更されました。 |
| [`tidb_enable_new_only_full_group_by_check`](/system-variables.md#tidb_enable_new_only_full_group_by_check-new-in-v610) | 追加 | この変数は TiDB が `ONLY_FULL_GROUP_BY` チェックを実行する際の挙動を制御します。 |
| [`tidb_enable_outer_join_reorder`](/system-variables.md#tidb_enable_outer_join_reorder-new-in-v610) | 追加 | v6.1.0 から TiDB の Join Reorder アルゴリズムが Outer Join をサポートしました。この変数はそのサポート挙動を制御し、デフォルト値は `ON` です。 |
| [`tidb_enable_prepared_plan_cache`](/system-variables.md#tidb_enable_prepared_plan_cache-new-in-v610) | 追加 | この設定は以前は `tidb.toml` のオプションでした（`prepared-plan-cache.enabled`）。ただし TiDB v6.1.0 からはシステム変数に変更されました。 |
| [`tidb_gc_max_wait_time`](/system-variables.md#tidb_gc_max_wait_time-new-in-v610) | 追加 | この変数は、GC セーフポイントが未コミットトランザクションによってブロックされる最大時間を設定するために使用されます。 |
| [tidb_max_auto_analyze_time](/system-variables.md#tidb_max_auto_analyze_time-new-in-v610) | 追加 | この変数は、auto analyze の最大実行時間を指定するために使用されます。 |
| [`tidb_max_tiflash_threads`](/system-variables.md#tidb_max_tiflash_threads-new-in-v610) | 追加 | この変数は、TiFlash がリクエストを実行する際の最大並列性を設定するために使用されます。 |
| [`tidb_mem_oom_action`](/system-variables.md#tidb_mem_oom_action-new-in-v610) | 追加 | この設定は以前は `tidb.toml` のオプションでした（`oom-action`）。ただし TiDB v6.1.0 からはシステム変数に変更されました。 |
| [`tidb_mem_quota_analyze`](/system-variables.md#tidb_mem_quota_analyze-new-in-v610) | 追加 | この変数は、TiDB が統計情報を更新する際の最大メモリ使用量を制御します。これにはユーザーによって手動で実行される [`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md) と TiDB バックグラウンドでの自動解析タスクが含まれます。 |
| [`tidb_nontransactional_ignore_error`](/system-variables.md#tidb_nontransactional_ignore_error-new-in-v610) | 追加 | この変数は、非トランザクショナル DML ステートメントでエラーが発生した場合にエラーを直ちに返すかどうかを指定します。 |
| [`tidb_prepared_plan_cache_memory_guard_ratio`](/system-variables.md#tidb_prepared_plan_cache_memory_guard_ratio-new-in-v610) | 追加 | この設定は以前は `tidb.toml` のオプションでした（`prepared-plan-cache.memory-guard-ratio`）。ただし TiDB v6.1.0 からはシステム変数に変更されました。 |
| [`tidb_prepared_plan_cache_size`](/system-variables.md#tidb_prepared_plan_cache_size-new-in-v610) | 追加 | この設定は以前は `tidb.toml` のオプションでした（`prepared-plan-cache.capacity`）。ただし TiDB v6.1.0 からはシステム変数に変更されました。 |
| [`tidb_stats_cache_mem_quota`](/system-variables.md#tidb_stats_cache_mem_quota-new-in-v610) | 追加 | この変数は、TiDB 統計情報キャッシュのメモリクォータを設定します。 |

### 設定ファイルパラメータ

| 設定ファイル | 設定 | 変更タイプ | 説明 |
|---|---|---|---|
| TiDB | `committer-concurrency` | 削除 | システム変数 `tidb_committer_concurrency` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は対応するシステム変数を変更する必要があります。 |
| TiDB | `lower-case-table-names` | 削除 | 現在、TiDB は `lower_case_table_name=2` のみをサポートしています。別の値が設定された場合、クラスタが v6.1.0 にアップグレードされた後、その値は失われます。 |
| TiDB | `mem-quota-query` | 削除 | システム変数 `tidb_mem_quota_query` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は対応するシステム変数を変更する必要があります。 |
| TiDB | `oom-action` | 削除 | システム変数 `tidb_mem_oom_action` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は対志するシステム変数を変更する必要があります。 |
| TiDB | `prepared-plan-cache.capacity` | 削除 | システム変数 `tidb_prepared_plan_cache_size` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は対応するシステム変数を変更する必要があります。 |
| TiDB | `prepared-plan-cache.enabled` | 削除 | システム変数 `tidb_enable_prepared_plan_cache` に置き換えられました。 この構成項目はもはや有効ではなく、値を変更する場合は対応するシステム変数を変更する必要があります。 |
| TiDB | `query-log-max-len` | 削除 | システム変数 `tidb_query_log_max_len` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は対応するシステム変数を変更する必要があります。 |
| TiDB | `require-secure-transport` | 削除 | システム変数 `require_secure_transport` に置き換えられました。この構成項目はもはや有効ではなく、値を変更する場合は対応するシステム変数を変更する必要があります。 |
| TiDB | `run-auto-analyze` | 削除 | システム変数 `tidb_enable_auto_analyze` に置き換えられました。 この構成項目はもはや有効ではなく、値を変更する場合は対応するシステム変数を変更する必要があります。 |
| TiDB | [`enable-global-kill`](/tidb-configuration-file.md#enable-global-kill-new-in-v610) | 追加 | グローバル・キル（インスタンス間でクエリまたは接続を終了させる）機能を有効にするかどうかを制御します。値が `true` の場合、`KILL` および `KILL TIDB` ステートメントのどちらもがインスタンス間でクエリまたは接続を終了させることができるため、クエリや接続の誤って終了することを心配する必要はありません。 |
| TiDB | [`enable-stats-cache-mem-quota`](/tidb-configuration-file.md#enable-stats-cache-mem-quota-new-in-v610) | 追加 | 統計情報キャッシュのメモリクォータを有効にするかどうかを制御します。 |
| TiKV | [`raft-engine.enable`](/tikv-configuration-file.md#enable-1) | 変更 | デフォルト値が `FALSE` から `TRUE` に変更されました。 |
| TiKV | [`region-max-keys`](/tikv-configuration-file.md#region-max-keys) | 変更 | デフォルト値が 1440000 から `region-split-keys / 2 * 3` に変更されました。 |
| TiKV | [`region-max-size`](/tikv-configuration-file.md#region-max-size) | 変更 | デフォルト値が 144 MB から `region-split-size / 2 * 3` に変更されました。 |
| TiKV | [`coprocessor.enable-region-bucket`](/tikv-configuration-file.md#enable-region-bucket-new-in-v610) | 追加 | 1つのリージョンをバケットと呼ばれるより小さな範囲に分割するかどうかを決定します。 |
| TiKV | [`coprocessor.region-bucket-size`](/tikv-configuration-file.md#region-bucket-size-new-in-v610) | 追加 | `enable-region-bucket` が true の場合のバケットのサイズです。 |
| TiKV | [`causal-ts.renew-batch-min-size`](/tikv-configuration-file.md#renew-batch-min-size) | 追加 | ローカルにキャッシュされたタイムスタンプの最小数。 |
| TiKV | [`causal-ts.renew-interval`](/tikv-configuration-file.md#renew-interval) | 追加 | ローカルにキャッシュされたタイムスタンプが更新される間隔。 |
| TiKV | [`max-snapshot-file-raw-size`](/tikv-configuration-file.md#max-snapshot-file-raw-size-new-in-v610) |新たに追加| スナップショットファイルのサイズがこの値を超えると、スナップショットファイルが複数のファイルに分割されます。|
| TiKV | [`raft-engine.memory-limit`](/tikv-configuration-file.md#memory-limit) |新たに追加| Raft Engineのメモリ使用量を制限します。|
| TiKV | [`storage.background-error-recovery-window`](/tikv-configuration-file.md#background-error-recovery-window-new-in-v610) |新たに追加| RocksDBが回復可能なバックグラウンドエラーを検出した後、許容される最大の回復時間です。|
| TiKV | [`storage.api-version`](/tikv-configuration-file.md#api-version-new-in-v610) |新たに追加| TiKVが生のキー・バリューストアとして機能する場合にTiKVが使用するストレージ形式およびインターフェースバージョン。|
| PD | [`schedule.max-store-preparing-time`](/pd-configuration-file.md#max-store-preparing-time-new-in-v610) |新たに追加| ストアがオンライン化するための最大待機時間を制御します。|
| TiCDC | [`enable-tls`](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka) |新たに追加| 下流のKafkaインスタンスに接続する際にTLSを使用するかどうか。|
| TiCDC | `sasl-gssapi-user`<br/>`sasl-gssapi-password`<br/>`sasl-gssapi-auth-type`<br/>`sasl-gssapi-service-name`<br/>`sasl-gssapi-realm`<br/>`sasl-gssapi-key-tab-path`<br/>`sasl-gssapi-kerberos-config-path` |新たに追加| KafkaのSASL/GSSAPI認証をサポートするために使用されます。詳細は[こちら](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka)を参照してください。|
| TiCDC | [`avro-decimal-handling-mode`](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka)<br/>[`avro-bigint-unsigned-handling-mode`](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka) |新たに追加| Avro形式の出力詳細を決定します。|
| TiCDC | [`dispatchers.topic`](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink) |新たに追加| TiCDCが増分データを異なるKafkaトピックにどのようにディスパッチするかを制御します。|
| TiCDC | [`dispatchers.partition`](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink) |新たに追加| `dispatchers.partition`は`dispatchers.dispatcher`の別名です。TiCDCが増分データをKafkaパーティションにどのようにディスパッチするかを制御します。|
| TiCDC | [`schema-registry`](/ticdc/ticdc-sink-to-kafka.md#integrate-ticdc-with-kafka-connect-confluent-platform) |新たに追加| Avroスキーマを保存するスキーマレジストリエンドポイントを指定します。|
| DM | `dmctl start-relay`コマンドの`worker` |削除済み| このパラメータの使用はお勧めしません。シンプルな実装を提供します。|
| DM | ソース構成ファイルの`relay-dir` |削除済み| ワーカー構成ファイル内の同じ構成項目に置き換えられました。|
| DM | タスク構成ファイルの`is-sharding` |削除済み| `shard-mode`構成項目に置き換えられました。|
| DM | タスク構成ファイルの`auto-fix-gtid` |削除済み| v5.xで非推奨となり、v6.1.0で正式に削除されました。|
| DM | ソース構成ファイルの`meta-dir`および`charset` |削除済み| v5.xで非推奨となり、v6.1.0で正式に削除されました。|

### その他

* デフォルトでプリペアドプランキャッシュを有効にする

    新しいクラスタでは、`Prepare` / `Execute`リクエストの実行計画をキャッシュするため、プリペアドプランキャッシュはデフォルトで有効になります。後続の実行では、クエリプランの最適化をスキップし、パフォーマンスが向上します。アップグレードされたクラスタは、構成ファイルから設定を引き継ぎます。新しいクラスタでは、新しいデフォルト値が使用されます。これにより、プリペアドプランキャッシュがデフォルトで有効になり、各セッションで最大100のプランをキャッシュできます（`capacity=100`）。この機能のメモリ消費については、[プリペアドプランキャッシュのメモリ管理](/sql-prepared-plan-cache.md#memory-management-of-prepared-plan-cache)を参照してください。

* TiDB v6.1.0以前では、`SHOW ANALYZE STATUS`はインスタンスレベルのタスクを表示し、TiDBを再起動するとタスクレコードがクリアされました。TiDB v6.1.0以降では、`SHOW ANALYZE STATUS`はクラスタレベルのタスクを表示し、再起動後もタスクレコードが保持されます。`tidb_analyze_version = 2`の場合、`Job_info`列に`解析オプション`情報が追加されます。

* TiKVの損傷したSSTファイルはTiKVプロセスがパニックする原因となる場合があります。TiDB v6.1.0以前では、損傷したSSTファイルがあるとすぐにTiKVがパニックしました。TiDB v6.1.0以降では、SSTファイルが損傷してから1時間後にTiKVプロセスがパニックします。

* 以下のTiKV設定項目は[動的に値を変更](/dynamic-config.md#modify-tikv-configuration-dynamically)することができます：

    * `raftstore.raft-entry-max-size`
    * `quota.foreground-cpu-time`
    * `quota.foreground-write-bandwidth`
    * `quota.foreground-read-bandwidth`
    * `quota.max-delay-duration`
    * `server.grpc-memory-pool-quota`
    * `server.max-grpc-send-msg-len`
    * `server.raft-msg-max-batch-size`

* v6.1.0では、一部の構成ファイルパラメータがシステム変数に変換されます。以前のバージョンからアップグレードされたv6.1.0クラスタについては、以下の点に留意してください：

    * アップグレード前に構成ファイルで指定されている構成項目がすでに存在する場合、TiDBは自動的にアップグレードプロセス中に構成項目の値を対応するシステム変数の値に更新します。これにより、アップグレード後、パラメータの最適化によりシステムの動作が変わらないようになります。
    * 上記の自動更新はアップグレード中に1度だけ行われます。アップグレード後、非推奨の構成項目はもはや有効ではありません。

* DM WebUIからダッシュボードページが削除されました。

* `dispatchers.topic`および`dispatchers.partition`が有効になっている場合、TiCDCをv6.1.0より古いバージョンにダウングレードできません。

* Avroプロトコルを使用したTiCDC Changefeedをv6.1.0より古いバージョンにダウングレードすることはできません。

## 改善

+ TiDB

    - `UnionScanRead`オペレータのパフォーマンスを向上させる[#32433](https://github.com/pingcap/tidb/issues/32433)
    - `EXPLAIN`の出力でタスクタイプの表示を改善する（MPPタスクタイプを追加）[#33332](https://github.com/pingcap/tidb/issues/33332)
    - 列のデフォルト値として`rand()`を使用する機能をサポートする[#10377](https://github.com/pingcap/tidb/issues/10377)
    - 列のデフォルト値として`uuid()`を使用する機能をサポートする[#33870](https://github.com/pingcap/tidb/issues/33870)
    - `latin1`から`utf8`/`utf8mb4`への列の文字セット変更をサポートする[#34008](https://github.com/pingcap/tidb/issues/34008)

+ TiKV

    - インメモリの悲観的ロックを使用する場合にCDCの旧値ヒット率を向上させる[#12279](https://github.com/tikv/tikv/issues/12279)
    - 利用できないRaftstoreを検出するヘルスチェックを改善し、TiKVクライアントがリージョンキャッシュを適時に更新できるようにする[#12398](https://github.com/tikv/tikv/issues/12398)
    - Raft Engineのメモリ制限を設定する機能をサポートする[#12255](https://github.com/tikv/tikv/issues/12255)
    - 損傷したSSTファイルを自動的に検出および削除し、製品の可用性を向上させる[#10578](https://github.com/tikv/tikv/issues/10578)
    - CDCがRawKVをサポートする[#11965](https://github.com/tikv/tikv/issues/11965)
    - 大きなスナップショットファイルを複数のファイルに分割する機能をサポートする[#11595](https://github.com/tikv/tikv/issues/11595)
    - Raftstoreからスナップショットのガベージコレクションをバックグラウンドスレッドに移動し、スナップショットGCがRaftstoreのメッセージループをブロックするのを防止する[#11966](https://github.com/tikv/tikv/issues/11966)
    - 最大メッセージ長（`max-grpc-send-msg-len`）およびgRPCメッセージの最大バッチサイズ（`raft-msg-max-batch-size`）の動的設定をサポートする[#12334](https://github.com/tikv/tikv/issues/12334)
    - Raftを介したオンラインのunsafe回復計画の実行をサポートする[#10483](https://github.com/tikv/tikv/issues/10483)

+ PD
    - リージョンラベルのTTL（有効期限）をサポートする[#4694](https://github.com/tikv/pd/issues/4694)
- サポートリージョンのバケツ [#4668](https://github.com/tikv/pd/issues/4668)
    - デフォルトでスワッガーサーバーのコンパイルを無効にする [#4932](https://github.com/tikv/pd/issues/4932)

+ TiFlash

    - 集約演算子のメモリ計算を最適化し、マージフェーズでより効率的なアルゴリズムを使用するようにします [#4451](https://github.com/pingcap/tiflash/issues/4451)

+ ツール

    + バックアップ＆リストア（BR）

        - 空のデータベースのバックアップとリストアをサポートします [#33866](https://github.com/pingcap/tidb/issues/33866)

    + TiDB Lightning

        - スキャッタリージョンをバッチモードに最適化し、スキャッタリージョンプロセスの安定性を向上させます [#33618](https://github.com/pingcap/tidb/issues/33618)

    + TiCDC

        - TiCDCはレプリケーション中に大規模トランザクションの分割をサポートし、大規模トランザクションに起因するレプリケーションの遅延を大幅に減少させます [#5280](https://github.com/pingcap/tiflow/issues/5280)

## バグ修正

+ TiDB

    - `in` 関数が `bit` タイプのデータを処理する際に発生する可能性があるパニックの問題を修正します [#33070](https://github.com/pingcap/tidb/issues/33070)
    - `UnionScan` 演算子が順序を維持できないためクエリ結果が間違ってしまう問題を修正します [#33175](https://github.com/pingcap/tidb/issues/33175)
    - 特定の場合に Merge Join 演算子が誤った結果を取得する問題を修正します [#33042](https://github.com/pingcap/tidb/issues/33042)
    - 動的プルーニングモードで `index join` の結果が誤る可能性がある問題を修正します [#33231](https://github.com/pingcap/tidb/issues/33231)
    - パーティション化されたテーブルの一部のパーティションが削除された時にデータがガベージコレクションされない可能性がある問題を修正します [#33620](https://github.com/pingcap/tidb/issues/33620)
    - クラスターの PD ノードが置き換えられた後に一部の DDL ステートメントが一時的にスタックする可能性がある問題を修正します [#33908](https://github.com/pingcap/tidb/issues/33908)
    - グラフャナダッシュボードで遅いクエリを確認すると `INFORMATION_SCHEMA.CLUSTER_SLOW_QUERY` テーブルをクエリした時に TiDB サーバーがメモリ不足になる可能性がある問題を修正します。この問題は `max_allowed_packet` システム変数が効果を発揮しない可能性があります [#33893](https://github.com/pingcap/tidb/issues/33893)
    - TopSQL モジュールでのメモリリーク問題を修正します [#34525](https://github.com/pingcap/tidb/issues/34525) [#34502](https://github.com/pingcap/tidb/issues/34502)
    - PointGet プランで Plan Cache が間違っている可能性がある問題を修正します [#32371](https://github.com/pingcap/tidb/issues/32371)
    - Plan Cache が RC 分離レベルで開始された時にクエリ結果が誤る可能性がある問題を修正します [#34447](https://github.com/pingcap/tidb/issues/34447)

+ TiKV

    - TiKV インスタンスがオフラインになった時に Raft ログの遅延が増加する問題を修正します [#12161](https://github.com/tikv/tikv/issues/12161)
    - マージ対象のリージョンが無効であるため TiKV が予期せず peers を破壊し、パニックを起こす問題を修正します [#12232](https://github.com/tikv/tikv/issues/12232)
    - v5.3.1 またはそれ以降のバージョンから v6.0.0 またはそれ以降のバージョンにアップグレードする時に TiKV が `failed to load_latest_options` エラーを報告する問題を修正します  [#12269](https://github.com/tikv/tikv/issues/12269)
    - メモリリソースが不足している時に Raft ログを追加することによって引き起こされる OOM の問題を修正します [#11379](https://github.com/tikv/tikv/issues/11379)
    - peers の破壊とバッチリージョンの分割が競合して TiKV パニックが発生する問題を修正します [#12368](https://github.com/tikv/tikv/issues/12368)
    - `stats_monitor` がデッドループに陥ると TiKV のメモリ使用量が急増する問題を修正します [#12416](https://github.com/tikv/tikv/issues/12416)
    - Follower Read を使用する時に TiKV が `invalid store ID 0` エラーを報告する問題を修正します [#12478](https://github.com/tikv/tikv/issues/12478)

+ PD 

    -  `not leader` の間違ったステータスコードを修正します [#4797](https://github.com/tikv/pd/issues/4797)
    -  一部の端末ケースでの TSO フォールバックのバグを修正します [#4884](https://github.com/tikv/pd/issues/4884)
    -  PD リーダーの変更後に削除されたトンストアが再表示される問題を修正します ​​[#4941](https://github.com/tikv/pd/issues/4941)
    -  PD リーダーの変更後にスケジューリングがすぐに開始されない問題を修正します [#4769](https://github.com/tikv/pd/issues/4769)

+ TiDB ダッシュボード

    - Top SQL が有効になる前に実行された SQL 文の CPU オーバーヘッドを収集できないバグを修正します  [#33859](https://github.com/pingcap/tidb/issues/33859)

+ TiFlash

    - 大量の INSERT および DELETE 操作後の潜在的なデータの不整合性を修正します  [#4956](https://github.com/pingcap/tiflash/issues/4956)

+ ツール

    + TiCDC

        - DDL スキーマのバッファリング方法を最適化することで余分なメモリを使用する問題を修正します [#1386](https://github.com/pingcap/tiflow/issues/1386)
        - 特定の増分スキャンシナリオで発生するデータの損失を修正します  [#5468](https://github.com/pingcap/tiflow/issues/5468)

    + TiDB データ移行（DM）

        - `start-time` のタイムゾーンの問題を修正し、DM の動作を下流のタイムゾーンから上流のタイムゾーンに変更します [#5271](https://github.com/pingcap/tiflow/issues/5471)
        - タスクが自動的に再開された後にディスク容量をさらに取り込む問題を修正します  [#3734](https://github.com/pingcap/tiflow/issues/3734) [#5344](https://github.com/pingcap/tiflow/issues/5344)
        - チェックポイントのフラッシュによって失敗した行のデータがスキップされる問題を修正します  [#5279](https://github.com/pingcap/tiflow/issues/5279)
        - 下流でフィルタリングされた DDL を手動で実行するとタスクの再開に失敗する可能性がある問題を修正します [#5272](https://github.com/pingcap/tiflow/issues/5272)
        - `case-sensitive: true` が設定されていない時に大文字のテーブルをレプリケートできない問題を修正します [#5255](https://github.com/pingcap/tiflow/issues/5255)
        - `SHOW CREATE TABLE` ステートメントのインデックスでプライマリキーが最初でない場合に DM ワーカーのパニック問題を修正します  [#5159](https://github.com/pingcap/tiflow/issues/5159)
        - GTID が有効になっているか、タスクが自動的に再開された時に CPU 使用率が上昇し、大量のログが印刷される問題を修正します [#5063](https://github.com/pingcap/tiflow/issues/5063)
        - DM WebUI でオフラインオプションおよびその他の使用問題を修正します  [#4993](https://github.com/pingcap/tiflow/issues/4993)
        - 上流のGTIDが空の場合に増分タスクの開始に失敗する問題を修正します [#3731](https://github.com/pingcap/tiflow/issues/3731)
        - 空の構成が dm-master でパニックを引き起こす問題を修正します  [#3732](https://github.com/pingcap/tiflow/issues/3732)

    + TiDB Lightning

        - プリチェックがローカルディスクリソースとクラスターの可用性をチェックしない問題を修正します  [#34213](https://github.com/pingcap/tidb/issues/34213)
        - スキーマの不正確なルーティングの問題を修正します  [#33381](https://github.com/pingcap/tidb/issues/33381)
        - TiDB Lightning がパニックした時に PD 構成が正しく復元されない問題を修正します  [#31733](https://github.com/pingcap/tidb/issues/31733)
        - `auto_increment` 列のデータが範囲外であるためにローカルバックエンドのインポートが失敗する問題を修正します  [#29737](https://github.com/pingcap/tidb/issues/27937)
        - `auto_random` または `auto_increment` 列が null の場合にローカルバックエンドのインポートが失敗する問題を修正します  [#34208](https://github.com/pingcap/tidb/issues/34208)