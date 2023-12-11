---
title: TiDB 6.2.0 リリースノート
---

# TiDB 6.2.0 リリースノート

リリース日: 2022年8月23日

TiDB バージョン: 6.2.0-DMR

> **注意:**
>
> TiDB 6.2.0-DMR のドキュメントは[アーカイブ](https://docs-archive.pingcap.com/tidb/v6.2/)されました。PingCAPはTiDBデータベースの[最新LTSバージョン](https://docs.pingcap.com/tidb/stable)の使用をお勧めします。

v6.2.0-DMR では、主な新機能と改善点は以下のとおりです:

* TiDB ダッシュボードは[ビジュアル実行計画](https://docs.pingcap.com/tidb/v6.2/dashboard-slow-query#visual-execution-plans)をサポートし、実行計画を直感的に表示します。
* TiDB ダッシュボードに[モニタリングページ](/dashboard/dashboard-monitoring.md)を追加し、パフォーマンス解析とチューニングを効率的に行えるようにしました。
* TiDB の[Lock View](/information-schema/information-schema-data-lock-waits.md)は楽観的トランザクションの待機情報を表示し、ロックの衝突を迅速に特定できるようにサポートしました。
* TiFlashは[新しいストレージフォーマット](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)をサポートし、安定性とパフォーマンスを向上させました。
* [Fine Grained Shuffle 機能](/system-variables.md#tiflash_fine_grained_shuffle_batch_size-new-in-v620)により、複数スレッドでのウィンドウ関数の並列実行が可能になりました。
* 新しい並列DDLフレームワーク: DDLステートメントがブロックされにくく、実行効率が向上します。
* TiKVは[CPU使用率を自動調整](/tikv-configuration-file.md#background-quota-limiter)し、安定した効率的なデータベース操作を保証します。
* [時間点リカバリ (PITR)](/br/backup-and-restore-overview.md)が導入され、過去の任意の時点のTiDBクラスターのスナップショットを新しいクラスターに復元できるようになりました。
* TiDB Lightningは物理インポートモードでのテーブルレベルでの[スケジューリングの一時停止](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md#scope-of-pausing-scheduling-during-import)をサポートし、クラスターレベルではなくなりました。
* BRは[ユーザーおよび権限データのリストア](/br/br-snapshot-guide.md#restore-tables-in-the-mysql-schema)をサポートし、バックアップとリストアの手順がよりスムーズになりました。
* TiCDCは特定のDDLイベントのフィルタリングをサポートすることで、より多くのデータレプリケーションシナリオを解除しました(/ticdc/ticdc-filter.md)。
* [`SAVEPOINT`メカニズム](/sql-statements/sql-statement-savepoint.md)がサポートされ、トランザクション内で柔軟にロールバックポイントを制御できるようになりました。
* TiDBは1つの`ALTER TABLE`ステートメントで複数のカラムまたはインデックスを追加、削除、変更できるようになりました[/sql-statements/sql-statement-alter-table.md)。
* [クロスクラスターRawKVレプリケーション](/tikv-configuration-file.md#api-version-new-in-v610)がサポートされるようになりました。

## 新機能

### SQL

* 物理データの圧縮機能がGAになりました

    TiFlashバックエンドは、特定の条件に基づいて物理データを自動的に圧縮し、無用なデータのバックログを減らし、データストレージ構造を最適化します。

    データの圧縮が自動的にトリガーされる前に、TiFlashテーブルにはしばしば一定量の無用なデータがあります。この機能を使えば、適切なタイミングを選んでSQLステートメントを手動で実行し、TiFlashの物理データを直ちに圧縮でき、ストレージスペースの使用を減らし、クエリのパフォーマンスを向上させることができます。この機能はTiDB v6.1で実験的なものでしたが、TiDB v6.2.0で一般提供版(GA)となりました。

    [ユーザードキュメント](/sql-statements/sql-statement-alter-table-compact.md#alter-table--compact) [#4145](https://github.com/pingcap/tiflash/issues/4145) @[breezewish](https://github.com/breezewish)

### 可観測性

* TiDBダッシュボードをPDから分割

    TiDBダッシュボードがPDからモニタリングノードに移動しました。これによりTiDBダッシュボードがPDに与える影響が軽減され、PDがより安定するようになりました。

    @[Hawkson-jee](https://github.com/Hawkson-jee)

* TiDBダッシュボードにモニタリングページを追加

    新しいモニタリングページには、パフォーマンスチューニングに必要な主要な指標が表示され、これを基に、[パフォーマンスチューニング方法](/performance-tuning-methods.md)に基づいてパフォーマンスを分析し、チューニングできるようになりました。

    具体的には、グローバルおよびトップダウンの視点からユーザーの応答時間とデータベース時間を分析し、ユーザー応答時間のボトルネックがデータベースの問題によるものかどうかを確認できます。ボトルネックがデータベースにある場合は、データベース時間の概要とSQLレイテンシの詳細を使用してボトルネックを特定し、パフォーマンスをチューニングできます。

    [ユーザードキュメント](/dashboard/dashboard-monitoring.md) [#1381](https://github.com/pingcap/tidb-dashboard/issues/1381) @[YiniXu9506](https://github.com/YiniXu9506)

* TiDBダッシュボードがビジュアル実行計画をサポート
    TiDBダッシュボードは、SQLステートメントおよびモニタリングページを通じて視覚的な実行計画と基本的な診断サービスを提供します。この機能により、クエリプランの各ステップを直感的に識別できます。そのため、複雑で大きなクエリの実行を理解する際に特に役立ちます。また、各クエリ実行計画では、TiDBダッシュボードが実行の詳細を自動的に分析し、潜在的な問題を特定し、特定のクエリプランの実行に必要な時間を短縮するための最適化提案を提供します。

    [ユーザードキュメント](https://docs.pingcap.com/tidb/v6.2/dashboard-slow-query#visual-execution-plans) [#1224](https://github.com/pingcap/tidb-dashboard/issues/1224) @[time-and-fate](https://github.com/time-and-fate)

* Lock Viewが楽観的トランザクションの待機情報を表示
    ロックの衝突が多すぎると、重大なパフォーマンス問題を引き起こす可能性があり、そのような問題をトラブルシューティングするためにロックの衝突を検出することが必要です。v6.2.0以前、TiDBは`INFORMATION_SCHEMA.DATA_LOCK_WAITS`システムビューを使用してロックの衝突関係を表示していましたが、これにより楽観的トランザクションの待機情報は表示できませんでした。TiDB v6.2.0では`DATA_LOCK_WAITS`ビューが拡張され、ビューには楽観的トランザクションを悲観的ロックによってブロックされたものがリストされます。この機能により、ユーザーはロックの衝突を迅速に検出し、アプリケーションの改善のための基盤を提供されるため、ロックの衝突の頻度を減らし、全体のパフォーマンスを向上させることができます。
    
    [ユーザードキュメント](/information-schema/information-schema-data-lock-waits.md) [#34609](https://github.com/pingcap/tidb/issues/34609) @[longfangsong](https://github.com/longfangsong)

### パフォーマンス
* `LEADING`オプティマイザヒントを改善して外部結合の順序をサポート
    v6.1.0で導入されたオプティマイザヒント`LEADING`は、テーブルの結合順序を変更するためのものでした。ただし、このヒントは外部結合を含むクエリには適用できませんでした。詳細については[`LEADING`ドキュメント](/optimizer-hints.md#leadingt1_name--tl_name-)を参照してください。v6.2.0では、TiDBはこの制限を解除しました。外部結合を含むクエリでも、このヒントを使用してテーブルの結合順序を指定し、SQL実行のパフォーマンスを向上させ、実行計画の突然の変更を回避できるようになります。
    
    [ユーザードキュメント](/optimizer-hints.md#leadingt1_name--tl_name-) [#29932](https://github.com/pingcap/tidb/issues/29932) @[Reminiscent](https://github.com/Reminiscent)

* 新しいオプティマイザ`SEMI_JOIN_REWRITE`を追加して`EXISTS`クエリのパフォーマンスを改善
    特定のシナリオでは、`EXISTS`を含むクエリには最適な実行計画がない可能性があり、実行に時間がかかりすぎる可能性があります。 v6.2.0では、このようなシナリオ向けにオプティマイザがリライトルールを追加し、クエリで`SEMI_JOIN_REWRITE`を使用して強制的にオプティマイザにクエリをリライトさせ、より良いクエリパフォーマンスを得ることができます。
    
    [ユーザードキュメント](/optimizer-hints.md#semi_join_rewrite) [#35323](https://github.com/pingcap/tidb/issues/35323) @[winoros](https://github.com/winoros)

* 新しいオプティマイザヒント`MERGE`を追加して分析クエリのパフォーマンスを改善
    共通表式(CTE)はクエリロジックを簡素化する効果的な手段です。複雑なクエリを記述するのに広く使用されています。v6.2.0の前は、CTEはTiFlash環境で自動的に展開できませんでした。これは、MPPの実行効率をある程度制限していました。v6.2.0では、MySQL互換のオプティマイザヒント`MERGE`が導入されました。このヒントを使用すると、オプティマイザはCTEを展開できるようになり、CTEクエリ結果の消費者はクエリをTiFlashで並行して実行できるようになり、一部の分析クエリのパフォーマンスが向上します。
    
    [ユーザードキュメント](/optimizer-hints.md#merge) [#36122](https://github.com/pingcap/tidb/issues/36122) @[dayicklp](https://github.com/dayicklp)

* 一部の分析シナリオでの集計操作のパフォーマンスを最適化
「名前」

「場所」

「日時」

+ {R}
+ {R}
  + {R}
    + {R}
      + {R}
        + {R}
          + {R}
            + {R}
Separately, after version 6.2.0, new rewriting rules are introduced to improve the performance of `COUNT(DISTINCT)` queries on a single column, in the case where a serious data skew exists due to uneven distribution of the aggregated column and the aggregated column has many different values in an OLAP scenario using TiFlash for aggregation operations.

[TiDB User Document](/system-variables.md#tidb_opt_skew_distinct_agg-new-in-v620)

[#36169](https://github.com/pingcap/tidb/issues/36169)

@[fixdb](https://github.com/fixdb)

+ TiDB supports concurrent DDL operations

In version 6.2.0, TiDB introduces a new concurrent DDL framework, which enables DDL statements to be concurrently executed on different table objects. This fix the issue in which DDL operations are blocked by DDL operations on other tables. Additionally, TiDB supports concurrent DDL execution when adding an index on multiple tables or changing a column type, which improves the efficiency of DDL execution.

[#32031](https://github.com/pingcap/tidb/issues/32031)

@[wjhuang2016](https://github.com/wjhuang2016)

+ Optimizer enhances the estimation of string matching

In TiDB version 6.2.0, the estimation method in the scenarios such as string matching or the condition is `like '%xyz'` or using a regular expression `regex ()` is enhanced. The new method combines the TopN information of statistics and system variables to improve the estimation accuracy, allowing for the modification of the match selectivity manually, thus improving the SQL performance.

[TiDB User Document](/system-variables.md#tidb_default_string_match_selectivity-new-in-v620)

[#36209](https://github.com/pingcap/tidb/issues/36209)

@[time-and-fate](https://github.com/time-and-fate)

+ Window functions pushed down to TiFlash can be executed in multiple threads

After the Fine Grained Shuffle feature is enabled, window functions can be executed in multiple threads, instead of in a single thread. This feature significantly reduces the query response time without changing user behavior. The granularity of the shuffle can be controlled by adjusting the value of the variables.

[TiDB User Document](/system-variables.md#tiflash_fine_grained_shuffle_batch_size-new-in-v620)

[#4631](https://github.com/pingcap/tiflash/issues/4631)

@[guo-shaoge](https://github.com/guo-shaoge)

+ TiFlash supports a newer version of storage format

The new storage format in TiFlash significantly reduces the CPU usage caused by GC in high-concurrency and heavy workload scenarios and reduces the IO traffic of background tasks, thereby boosting stability under high concurrency and heavy workloads, while significantly reducing space amplification and disk waste. In version 6.2.0, data is stored in the new storage format by default. It should be noted that if TiFlash is upgraded from earlier versions to v6.2.0, in-place downgrade on TiFlash cannot be performed, as earlier TiFlash versions cannot recognize the new storage format.

For more information about upgrading TiFlash, see [TiFlash Upgrade Guide](/tiflash-upgrade-guide.md).

[TiDB User Document](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)

[#3594](https://github.com/pingcap/tiflash/issues/3594)

@[JaySon-Huang](https://github.com/JaySon-Huang) @[lidezhu](https://github.com/lidezhu) @[jiaqizho](https://github.com/jiaqizho)

+ TiFlash optimizes data scanning performance in multiple concurrency scenarios (experimental)

TiFlash reduces duplicate reads of the same data by merging read operations of the same data, and optimizes the resource overhead in the case of multiple concurrent tasks to improve data scanning performance. It avoids the situation where the same data has to be read separately in each task or even the same data may be read multiple times at the same time, if the same data is involved in multiple concurrent tasks.

[TiDB User Document](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)

[#5376](https://github.com/pingcap/tiflash/issues/5376)

@[JinheLin](https://github.com/JinheLin)

+ TiFlash adds FastScan for data scanning to increase read and write speed by sacrificing data consistency (experimental)

In version 6.2.0, FastScan is introduced in TiDB. It supports skipping consistency checks to significantly increase the speed. FastScan is suitable for scenarios that do not require high accuracy and consistency of data such as offline analysis tasks. Previously, to ensure data consistency, TiFlash needed to perform data consistency checks during the data scanning process to find the required data from multiple different versions.

When upgrading from an earlier version to TiDB v6.2.0, FastScan is not enabled by default for all tables to ensure data consistency. FastScan can be independently enabled for each table and if the table is set to FastScan in TiDB v6.2.0, it will be disabled when downgraded to a lower version. However, this does not affect the normal data read. In this case, it is equivalent to strong consistency read.

[TiDB User Document](/tiflash/use-fastscan.md)

[#5252](https://github.com/pingcap/tiflash/issues/5252)

@[hongyunyan](https://github.com/hongyunyan)

### Stability

+ TiKV supports automatically tuning the CPU usage (experimental)

Starting from version 6.2.0, TiDB supports setting the CPU usage rate of background requests using the TiKV configuration file, thereby limiting the CPU usage ratio of background operations such as automatically collecting statistics in TiKV, and avoiding the resource preemption of user operations by background operations in extreme cases. This ensures that the operations of the database are stable and efficient.

At the same time, TiDB also supports automatically adjusting CPU usage. Then, TiKV will adaptively adjust the CPU resources occupied by background requests according to the CPU usage of the instance. This feature is disabled by default.

[TiDB User Document](/tikv-configuration-file.md#background-quota-limiter)

[#12503](https://github.com/tikv/tikv/issues/12503)

@[BornChanger](https://github.com/BornChanger)

### Ease of use

+ TiKV supports listing detailed configuration information using command-line flags

Starting from version 6.2.0, tikv-server supports a new command-line flag [`—-config-info`](/command-line-flags-for-tikv-configuration.md#--config-info-format) that lists default and current values of all TiKV configuration items, helps users to quickly verify the startup parameters of the TiKV process, and improves usability.

[TiDB User Document](/command-line-flags-for-tikv-configuration.md#--config-info-format)

[#12492](https://github.com/tikv/tikv/issues/12492)

@[glorv](https://github.com/glorv)

### MySQL compatibility

+ TiDB supports modifying multiple columns or indexes in a single `ALTER TABLE` statement

Before version 6.2.0, TiDB only supports single DDL changes, which leads to incompatible DDL operations when migrating heterogeneous databases. It also takes extra effort to modify a complex DDL statement into multiple TiDB-supported simple DDL statements. Additionally, some users rely on the ORM framework to create assembly in SQL, thus causing SQL incompatibility. Since version 6.2.0, TiDB supports modifying multiple schema objects in a single SQL statement, which is convenient for users to implement SQL and improves usability.

[TiDB User Document](/sql-statements/sql-statement-alter-table.md)

[#14766](https://github.com/pingcap/tidb/issues/14766)

@[tangenta](https://github.com/tangenta)

+ Support setting savepoints in transactions

In some complex application scenarios, you might need to manage many operations in a transaction, and sometimes you might need to roll back some operations in the transaction. "Savepoint" is a nameable mechanism for the internal implementation of transactions. With this mechanism, you can flexibly control the rollback points within a transaction, thereby managing the more complex transactions and having more freedom in designing diverse applications.
### ユーザードキュメント(/sql-statements/sql-statement-savepoint.md) [#6840](https://github.com/pingcap/tidb/issues/6840) @[crazycs520](https://github.com/crazycs520)

### データ移行

* BRは、ユーザーデータと権限データの復元をサポートしています

    BRは通常の復元を実行する際にユーザーデータと権限データの復元をサポートしています。ユーザーデータと権限データを復元するためには、追加の復元プランは必要ありません。この機能を有効にするには、BRを使用してデータを復元する際に `--with-sys-table` パラメータを指定してください。

    ユーザードキュメント(/br/br-snapshot-guide.md#restore-tables-in-the-mysql-schema) [#35395](https://github.com/pingcap/tidb/issues/35395) @[D3Hunter](https://github.com/D3Hunter)

* バックアップおよびスナップショットに基づく時点復旧（PITR）のサポート

    PITRは、バックアップとスナップショットに基づいて実装されています。これにより、クラスタのスナップショットを過去のあらゆる時点に新しいクラスタに復元することが可能となります。この機能は以下の要件を満たします：

    - 災害復旧におけるRPOを20分未満に削減します。
    - アプリケーションからの誤った書き込みの処理を行います。たとえば、エラーイベントの前のデータにロールバックします。
    - 法律および規制要件を満たすために履歴データの監査を実行します。

    この機能には使用上の制約があります。詳細は、ユーザードキュメントを参照してください。

    ユーザードキュメント(/br/backup-and-restore-overview.md) [#29501](https://github.com/pingcap/tidb/issues/29501) @[joccau](https://github.com/joccau)

* DMは連続データ検証（実験的）をサポートしています

    連続データ検証は、データ移行の際に上流のバイナリログと下流に書き込まれたデータを連続して比較するために使用されます。検証ツールは、データの整合性に問題がある場合や欠落したレコードなどのデータ異常を特定します。

    この機能により、通常のフルデータ検証スキームにおける検証の遅延および過剰なリソース消費の問題が解決されます。

    ユーザードキュメント(/dm/dm-continuous-data-validation.md) [#4426](https://github.com/pingcap/tiflow/issues/4426) @[D3Hunter](https://github.com/D3Hunter) @[buchuitoudegou](https://github.com/buchuitoudegou)

* Amazon S3バケットのリージョンを自動的に特定します

    データ移行タスクは、Amazon S3バケットのリージョンを自動的に特定できます。リージョンパラメータを明示的に渡す必要はありません。

    [#34275](https://github.com/pingcap/tidb/issues/34275) @[WangLe1321](https://github.com/WangLe1321)

* TiDB Lightningにおけるディスククォータの設定をサポートします（実験的）

    TiDB Lightningが物理的なインポートモードでデータをインポート（backend='local'）する際、sorted-kv-dirにはソースデータを格納するための十分なスペースが必要です。ディスクスペースが不足していると、インポートタスクが失敗する可能性があります。新しい `disk_quota` 構成を使用して、TiDB Lightningが使用するディスクスペースの総量を制限することで、sorted-kv-dirに十分なストレージスペースがない状態でも、インポートタスクを正常に完了できます。

    ユーザードキュメント(/tidb-lightning/tidb-lightning-physical-import-mode-usage.md#configure-disk-quota-new-in-v620) [#446](https://github.com/pingcap/tidb-lightning/issues/446) @[buchuitoudegou](https://github.com/buchuitoudegou)

* TiDB Lightningは、物理的なインポートモードでデータを本番クラスタにインポートすることをサポートします

    以前は、TiDB Lightningの物理的なインポートモード（backend='local'）は、ターゲットクラスタに大きな影響を与えました。たとえば、移行中にPD全体のスケジューリングが一時停止しました。したがって、以前の物理的なインポートモードは、初期データインポートにのみ適していました。

    TiDB Lightningは、既存の物理的なインポートモードを改善しました。テーブルのスケジューリングを一時停止することで、インポートの影響をクラスタレベルからテーブルレベルに軽減します。つまり、インポート中でないテーブルの読み込みと書き込みができます。

    この機能には手動の構成は不要です。TiDBクラスタがv6.1.0以上のバージョンであり、TiDB Lightningがv6.2.0以上のバージョンである場合、新しい物理的なインポートモードが自動的に有効になります。

    ユーザードキュメント(/tidb-lightning/tidb-lightning-physical-import-mode-usage.md#scope-of-pausing-scheduling-during-import) [#35148](https://github.com/pingcap/tidb/issues/35148) @[gozssky](https://github.com/gozssky)

* [TiDB Lightningのユーザードキュメント](/tidb-lightning/tidb-lightning-overview.md)を再構成し、その構造をより合理的かつ明確にしました。"backend" の用語も新しいユーザーの理解の障壁を下げるように変更されています：

    - "local backend" を "physical import mode" に置き換えます。
    - "tidb backend" を "logical import mode" に置き換えます。

### TiDBデータ共有サブスクリプション

* クロスクラスタRawKVレプリケーションをサポート（実験的）

    RawKVのデータ変更の購読をサポートし、新しいコンポーネントTiKV-CDCを使用して、データ変更をリアルタイムにダウンストリームのTiKVクラスタにレプリケートすることが可能となります。

    ユーザードキュメント(/tikv-configuration-file.md#api-version-new-in-v610) [#11965](https://github.com/tikv/tikv/issues/11965) @[pingyu](https://github.com/pingyu)

* DDLおよびDMLイベントのフィルタリングをサポート

    特定の機会に、増分データ変更ログのフィルタ規則を設定したい場合があります。たとえば、DROP TABLEなどの高リスクDDLイベントをフィルタリングする。v6.2.0以降、TiCDCは特定のタイプのDDLイベントをフィルタリングし、SQL式に基づいてDMLイベントをフィルタリングすることをサポートしています。これにより、TiCDCがより多くのデータレプリケーションシーンに適用可能となります。

    ユーザードキュメント(/ticdc/ticdc-filter.md) [#6160](https://github.com/pingcap/tiflow/issues/6160) @[asddongmen](https://github.com/asddongmen)

## 互換性変更

### システム変数

| 変数名 | 変更タイプ | 説明 |
| --- | --- | --- |
| [tidb_enable_new_cost_interface](/system-variables.md#tidb_enable_new_cost_interface-new-in-v620) | 新規追加 | この変数は、[リファクタされたコストモデルの実装](/cost-model.md#cost-model-version-2)を有効にするかどうかを制御します。 |
| [tidb_cost_model_version](/system-variables.md#tidb_cost_model_version-new-in-v620) | 新規追加 | TiDBは物理最適化中にインデックスとオペレータを選択するためにコストモデルを使用します。この変数はコストモデルバージョンを選択するために使用されます。TiDB v6.2.0 では、内部テストで以前のバージョンよりも正確なコストモデルバージョン2を導入しています。 |
| tidb_enable_concurrent_ddl | 新規追加 | この変数は、TiDBに並列DDLステートメントを使用するかどうかを制御します。この変数を変更しないでください。この変数を無効にするリスクには調査が必要であり、クラスタのメタデータが破損する可能性があります。 |
| [tiflash_fine_grained_shuffle_stream_count](/system-variables.md#tiflash_fine_grained_shuffle_stream_count-new-in-v620) | 新規追加 | この変数は、ウィンドウ関数の並列実行の並行度を制御します。ウィンドウ関数がTiFlashで実行される際に使用されます。 |
| [tiflash_fine_grained_shuffle_batch_size](/system-variables.md#tiflash_fine_grained_shuffle_batch_size-new-in-v620) | 新規追加 | Fine Grained Shuffleが有効になっている場合、TiFlashにプッシュダウンされたウィンドウ関数を並列で実行できます。この変数は、送信者によって送信されるデータのバッチサイズを制御します。累積行数がこの値を超えると、送信者はデータを送信します。 |
| [tidb_default_string_match_selectivity](/system-variables.md#tidb_default_string_match_selectivity-new-in-v620) | 新規追加 | この変数は、フィルタ条件での`like`、`rlike`、および`regexp` 関数のデフォルトのセレクティビティを設定するために使用されます。また、これはこれらの関数の推定に役立つかどうかも制御します。 |
| [tidb_enable_analyze_snapshot](/system-variables.md#tidb_enable_analyze_snapshot-new-in-v620) | 新規追加 | この変数は、`ANALYZE`を実行する際に履歴データまたは最新データを読み取るかどうかを制御します。 |
| [tidb_generate_binary_plan](/system-variables.md#tidb_generate_binary_plan-new-in-v620) | 新規追加 | この変数は、遅いログやステートメントの要約でバイナリエンコードされた実行計画を生成するかどうかを制御します。 |
| [tidb_opt_skew_distinct_agg](/system-variables.md#tidb_opt_skew_distinct_agg-new-in-v620) | 新規追加 | この変数は、オプティマイザが `DISTINCT` 付きの集約関数を、たとえば `SELECT b、COUNT(DISTINCT a) FROM t GROUP BY b` を `SELECT b、COUNT(a) FROM (SELECT b、a FROM t GROUP BY b、a) t GROUP BY b` にリライトするかどうかを設定します。 |
| [tidb_enable_noop_variables](/system-variables.md#tidb_enable_noop_variables-new-in-v620) | 新規追加 | この変数は、`SHOW [GLOBAL] VARIABLES` の結果に `noop` 変数を表示するかどうかを制御します。 |
| [tidb_min_paging_size](/system-variables.md#tidb_min_paging_size-new-in-v620) | 追加された | この変数は、コプロセッサのページングリクエスト処理中に許可される最大の行数を設定するために使用されます。 |
| [tidb_txn_commit_batch_size](/system-variables.md#tidb_txn_commit_batch_size-new-in-v620) | 追加された | この変数は、TiDBがTiKVに送信するトランザクションコミットリクエストのバッチサイズを制御するために使用されます。 |
| tidb_enable_change_multi_schema | 削除された | この変数は、1つの `ALTER TABLE` ステートメントで複数のカラムやインデックスを変更できるかを制御するために使用されます。 |
| [tidb_enable_outer_join_reorder](/system-variables.md#tidb_enable_outer_join_reorder-new-in-v610) | 変更された | この変数は、TiDBの Join Reorder アルゴリズムが Outer Join をサポートするかどうかを制御します。v6.1.0ではデフォルト値は `ON` で、Outer Join のサポートがデフォルトで有効になっています。v6.2.0からはデフォルト値が `OFF` となり、サポートがデフォルトで無効になります。 |

### 設定ファイルパラメータ

| 設定ファイル | 設定 | 変更タイプ | 説明 |
| --- | --- | --- | --- |
| TiDB | feedback-probability | 削除された | この設定はもはや効果がなく、推奨されません。 |
| TiDB | query-feedback-limit | 削除された | この設定はもはや効果がなく、推奨されません。 |
| TiKV | [server.simplify-metrics](/tikv-configuration-file.md#simplify-metrics-new-in-v620) | 追加された | この設定は、返される監視メトリクスを簡略化するかどうかを指定します。 |
| TiKV | [quota.background-cpu-time](/tikv-configuration-file.md#background-cpu-time-new-in-v620) | 追加された | この設定は、TiKVのバックグラウンドが読み書きリクエストを処理するために使用するCPUリソースのソフト制限を指定します。 |
| TiKV | [quota.background-write-bandwidth](/tikv-configuration-file.md#background-write-bandwidth-new-in-v620) | 追加された | この設定は、バックグラウンドトランザクションがデータを書き込む際の帯域幅のソフト制限を指定します（現在は効果がありません）。 |
| TiKV | [quota.background-read-bandwidth](/tikv-configuration-file.md#background-read-bandwidth-new-in-v620) | 追加された | この設定は、バックグラウンドトランザクションや Coprocessor がデータを読む際の帯域幅のソフト制限を指定します（現在は効果がありません）。 |
| TiKV | [quota.enable-auto-tune](/tikv-configuration-file.md#enable-auto-tune-new-in-v620) | 追加された | この設定は、クォータの自動チューニングを有効にするかどうかを指定します。この設定が有効な場合、TiKVはインスタンスの負荷に基づいてバックグラウンドリクエストのクォータを動的に調整します。 |
| TiKV | rocksdb.enable-pipelined-commit | 削除された | この設定はもはや効果がありません。 |
| TiKV | gc-merge-rewrite | 削除された | この設定はもはや効果がありません。 |
| TiKV | [log-backup.enable](/tikv-configuration-file.md#enable-new-in-v620) | 追加された | この設定は、TiKVでログバックアップを有効にするかどうかを制御します。 |
| TiKV | [log-backup.file-size-limit](/tikv-configuration-file.md#file-size-limit-new-in-v620) | 追加された | この設定は、ログバックアップデータのサイズ制限を指定します。この制限に達すると、データは自動的に外部ストレージにフラッシュされます。 |
| TiKV | [log-backup.initial-scan-pending-memory-quota](/tikv-configuration-file.md#initial-scan-pending-memory-quota-new-in-v620) | 追加された | この設定は、増分スキャンデータを保持するために使用されるキャッシュのクォータを指定します。 |
| TiKV | [log-backup.max-flush-interval](/tikv-configuration-file.md#max-flush-interval-new-in-v620) | 追加された | この設定は、ログバックアップで外部ストレージにバックアップデータを書き込む最大インターバルを指定します。 |
| TiKV | [log-backup.initial-scan-rate-limit](/tikv-configuration-file.md#initial-scan-rate-limit-new-in-v620) | 追加された | この設定は、ログバックアップでの増分データスキャンのスループット制限を指定します。 |
| TiKV | [log-backup.num-threads](/tikv-configuration-file.md#num-threads-new-in-v620) | 追加された | この設定は、ログバックアップで使用されるスレッド数を指定します。 |
| TiKV | [log-backup.temp-path](/tikv-configuration-file.md#temp-path-new-in-v620) | 追加された | この設定は、ログファイルが外部ストレージにフラッシュされる前に書き込まれる一時的なパスを指定します。 |
| TiKV | [rocksdb.defaultcf.format-version](/tikv-configuration-file.md#format-version-new-in-v620) | 追加された | SSTファイルのフォーマットバージョンを指定します。 |
| TiKV | [rocksdb.writecf.format-version](/tikv-configuration-file.md#format-version-new-in-v620) | 追加された | SSTファイルのフォーマットバージョンを指定します。 |
| TiKV | [rocksdb.lockcf.format-version](/tikv-configuration-file.md#format-version-new-in-v620) | 追加された | SSTファイルのフォーマットバージョンを指定します。 |
| PD | replication-mode.dr-auto-sync.wait-async-timeout | 削除された | この設定は効果がありませんので削除されました。 |
| PD | replication-mode.dr-auto-sync.wait-sync-timeout | 削除された | この設定は効果がありませんので削除されました。 |
| TiFlash | [`storage.format_version`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) | 変更された | `format_version` のデフォルト値は`4`に変更され、v6.2.0以降のデフォルトフォーマットとなり、書き込みの増加とバックグラウンドタスクのリソース消費を削減します。 |
| TiFlash | [profiles.default.dt_enable_read_thread](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) | 追加された | この設定は、ストレージエンジンからの読み取りリクエストを処理するためにスレッドプールを使用するかどうかを制御します。デフォルト値は `false` です。 |
| TiFlash | [profiles.default.dt_page_gc_threshold](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) | 追加された | この設定は、PageStorageデータファイル内の有効データの最小比率を指定します。 |
| TiCDC | [--overwrite-checkpoint-ts](/ticdc/ticdc-manage-changefeed.md#resume-a-replication-task) | 追加された | この設定は、`cdc cli changefeed resume` サブコマンドに追加されました。 |
| TiCDC | [--no-confirm](/ticdc/ticdc-manage-changefeed.md#resume-a-replication-task) | 追加された | この設定は、`cdc cli changefeed resume` サブコマンドに追加されました。 |
| DM | [mode](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced) | 追加された | この設定は検証パラメータです。オプションの値は `full`、 `fast`、 `none` です。デフォルト値は `none` で、データの検証は行いません。 |
| DM | [worker-count](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced) | 追加された | この設定は検証パラメータであり、バックグラウンドでの検証ワーカーの数を指定します。デフォルト値は `4` です。 |
| DM | [row-error-delay](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced) | 追加された | この設定は検証パラメータです。指定された時間内に行が検証されない場合、エラー行としてマークされます。デフォルト値は `30m` で、30分を表します。 |
| TiDB Lightning | [tikv-importer.store-write-bwlimit](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) | 追加された | この設定は、TiDB Lightningが各TiKVストアにデータを書き込む際の書き込み帯域幅を決定します。デフォルト値は `0` で、帯域幅が制限されていないことを示します。 |
| TiDB Lightning | [tikv-importer.disk-quota](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md#configure-disk-quota-new-in-v620) | 追加された | この設定は、TiDB Lightningが使用するストレージスペースの制限を指定します。 |

### その他

- TiFlash の `format_version` は `4` から `3` にはダウングレードできません。詳細は、[TiFlash アップグレードガイド](/tiflash-upgrade-guide.md)を参照してください。
- v6.2.0以降のバージョンでは、`dt_enable_logical_split` のデフォルト値を `false` に維持することが強く推奨されます。詳細については、既知の問題 [#5576](https://github.com/pingcap/tiflash/issues/5576) を参照してください。
- バックアップクラスタに TiFlash レプリカがある場合、PITR を実行すると、復元クラスタには TiFlash レプリカのデータが含まれません。TiFlash レプリカからデータを復元するには、TiFlash レプリカを手動で構成する必要があります。`exchange partition` DDL ステートメントを実行すると、PITR が失敗する場合があります。上流データベースが TiDB Lightning の物理インポートモードを使用してデータをインポートする場合、データはログバックアップでバックアップできません。データインポート後に完全バックアップを実行することをお勧めします。PITRのその他の互換性の問題については、[PITR limitations](/br/backup-and-restore-overview.md#before-you-use) を参照してください。
- Since TiDB v6.2.0, you can restore table in `mysql` schema by specifying the parameter `--with-sys-table=true` when restoring data.
- When you execute the `ALTER TABLE` statement to add, drop, or modify multiple columns or indexes, TiDB checks table consistency by comparing the table before and after statement execution, regardless of the change in the same DDL statement. The execution order of the DDLs is not fully compatible with MySQL in some scenarios.
- If the TiDB component is v6.2.0 or later, the TiKV component should not be earlier than v6.2.0.
- TiKV adds a configuration item `split.region-cpu-overload-threshold-ratio` that supports [dynamic configuration](/dynamic-config.md#modify-tikv-configuration-dynamically).
- Slow query logs, `information_schema.statements_summary`, and `information_schema.slow_query`can export `binary_plan`, or execution plans encoded in the binary format.
- Two columns are added to the `SHOW TABLE ... REGIONS` statement: `SCHEDULING_CONSTRAINTS` and `SCHEDULING_STATE`, which respectively indicate Region scheduling constraints in Placement in SQL and the current scheduling state.
- Since TiDB v6.2.0, you can capture data changes of RawKV via [TiKV-CDC](https://github.com/tikv/migration/tree/main/cdc).
- When `ROLLBACK TO SAVEPOINT` is used to roll back a transaction to a specified savepoint, MySQL releases the locks held only after the specified savepoint, while in TiDB pessimistic transaction, TiDB does not immediately release the locks held after the specified savepoint. Instead, TiDB releases all locks when the transaction is committed or rolled back.
- Since TiDB v6.2.0, the `SELECT tidb_version()` statement also returns Store type (tikv or unistore).
- TiDB no longer has hidden system variables.
- TiDB v6.2.0 introduces two new system tables:
    - `INFORMATION_SCHEMA.VARIABLES_INFO`: used for viewing information about TiDB system variables.
    - `PERFORMANCE_SCHEMA.SESSION_VARIABLES`: used for viewing information about TiDB session-level system variables.

## Removed feature

Since TiDB v6.2.0, backing up and restoring RawKV using BR is deprecated.

## Improvements

+ TiDB

    - Support the `SHOW COUNT(*) WARNINGS` and `SHOW COUNT(*) ERRORS` statements [#25068](https://github.com/pingcap/tidb/issues/25068) @[likzn](https://github.com/likzn)
    - Add validation check for some system variables [#35048](https://github.com/pingcap/tidb/issues/35048) @[morgo](https://github.com/morgo)
    - Optimize the error messages for some type conversions [#32447](https://github.com/pingcap/tidb/issues/32744) @[fanrenhoo](https://github.com/fanrenhoo)
    - The `KILL` command now supports DDL operations [#24144](https://github.com/pingcap/tidb/issues/24144) @[morgo](https://github.com/morgo)
    - Make the output of `SHOW TABLES/DATABASES LIKE …` more MySQL-compatible. The column names in the output contain the `LIKE` value [#35116](https://github.com/pingcap/tidb/issues/35116) @[likzn](https://github.com/likzn)
    - Improve the performance of JSON-related functions [#35859](https://github.com/pingcap/tidb/issues/35859) @[wjhuang2016](https://github.com/wjhuang2016)
    - Improve the verification speed of password login using SHA-2 [#35998](https://github.com/pingcap/tidb/issues/35998) @[virusdefender](https://github.com/virusdefender)
    - Simplify some log outputs [#36011](https://github.com/pingcap/tidb/issues/36011) @[dveeden](https://github.com/dveeden)
    - Optimize the Coprocessor communication protocol. This can greatly reduce the memory consumption of the TiDB processes when reading data, and further alleviate the OOM issue in the scenario of scanning tables and exporting data by Dumpling. The system variable `tidb_enable_paging` is introduced to control whether to enable this communication protocol (with the scope of SESSION or GLOBAL). This protocol is disabled by default. To enable it, set the variable value to `true` [#35633](https://github.com/pingcap/tidb/issues/35633) @[tiancaiama](https://github.com/tiancaiamao) @[wshwsh12](https://github.com/wshwsh12)
    - Optimize the accuracy of memory tracking for some operators (HashJoin, HashAgg, Update, Delete) ([#35634](https://github.com/pingcap/tidb/issues/35634), [#35631](https://github.com/pingcap/tidb/issues/35631), [#35635](https://github.com/pingcap/tidb/issues/35635) @[wshwsh12](https://github.com/wshwsh12)) ([#34096](https://github.com/pingcap/tidb/issues/34096) @[ekexium](https://github.com/ekexium))

    - The system table `INFORMATION_SCHEMA.DATA_LOCK_WAIT` supports recording the locking information of optimistic transactions [#34609](https://github.com/pingcap/tidb/issues/34609) @[longfangson](https://github.com/longfangsong)
    - Add some monitoring metrics for transactions [#34456](https://github.com/pingcap/tidb/issues/34456) @[longfangsong](https://github.com/longfangsong)

+ TiKV

    - Support compressing the metrics response using gzip to reduce the HTTP body size [#12355](https://github.com/tikv/tikv/issues/12355) @[glorv](https://github.com/glorv)
    - Improve the readability of the TiKV panel in Grafana Dashboard [#12007](https://github.com/tikv/tikv/issues/12007) @[kevin-xianliu](https://github.com/kevin-xianliu)
    - Optimize the commit pipeline performance of the Apply operator [#12898](https://github.com/tikv/tikv/issues/12898) @[ethercflow](https://github.com/ethercflow)
    - Support dynamically modifying the number of sub-compaction operations performed concurrently in RocksDB (`rocksdb.max-sub-compactions`) [#13145](https://github.com/tikv/tikv/issues/13145) @[ethercflow](https://github.com/ethercflow)

+ PD

    - Support the statistical dimension of CPU usage of a Region and enhance the usage scenarios of Load Base Split [#12063](https://github.com/tikv/tikv/issues/12063) @[Jmpotato](https://github.com/JmPotato)

+ TiFlash

    - Refine error handling of the TiFlash MPP engine, thereby enhancing stability [#5095](https://github.com/pingcap/tiflash/issues/5095) @[windtalker](https://github.com/windtalker) @[yibin87](https://github.com/yibin87)

    - Optimize the comparison and sorting of UTF8_BIN and UTF8MB4_BIN collations [#5294](https://github.com/pingcap/tiflash/issues/5294) @[solotzg](https://github.com/solotzg)

+ Tools

    - Backup & Restore (BR)

        - Adjust the backup data directory structure to fix backup failure caused by S3 rate limiting in large cluster backup [#30087](https://github.com/pingcap/tidb/issues/30087) @[MoCuishle28](https://github.com/MoCuishle28)

    - TiCDC

        - Reduce performance overhead caused by runtime context switching in multi-Region scenarios [#5610](https://github.com/pingcap/tiflow/issues/5610) @[hicqu](https://github.com/hicqu)

        - Optimize redo log performance, and fix meta and data inconsistency problems ([#6011](https://github.com/pingcap/tiflow/issues/6011) @[CharlesCheung96](https://github.com/CharlesCheung96)) ([#5924](https://github.com/pingcap/tiflow/issues/5924) @[zhaoxinyu](https://github.com/zhaoxinyu)) ([#6277](https://github.com/pingcap/tiflow/issues/6277) @[hicqu](https://github.com/hicqu))

    - TiDB Lightning

        - Add more retryable errors, including EOF, Read index not ready, and Coprocessor timeout [#36674](https://github.com/pingcap/tidb/issues/36674), [#36566](https://github.com/pingcap/tidb/issues/36566) @[D3Hunter](https://github.com/D3Hunter)

    - TiUP
- TiUPを使用して新しいクラスターを展開すると、node-exporterは[1.3.1](https://github.com/prometheus/node_exporter/releases/tag/v1.3.1)バージョンを使用し、blackbox-exporterは[0.21.1](https://github.com/prometheus/blackbox_exporter/releases/tag/v0.21.1)バージョンを使用して、異なるシステムや環境での展開が成功することを確認します

## バグ修正

+ TiDB

    - クエリ条件でパーティションキーが使用され、コレートがクエリパーティションテーブルと異なる場合に誤ってパーティションが刈り込まれる問題を修正しました [#32749](https://github.com/pingcap/tidb/issues/32749) @[mjonss](https://github.com/mjonss)
    - `SET ROLE`がホストに大文字が含まれている場合、付与されたロールと一致しない問題を修正しました [#33061](https://github.com/pingcap/tidb/issues/33061) @[morgo](https://github.com/morgo)
    - `auto_increment`を持つ列を削除できない問題を修正しました [#34891](https://github.com/pingcap/tidb/issues/34891) @[Defined2014](https://github.com/Defined2014)
    - `SHOW CONFIG`が削除された構成項目を表示する問題を修正しました [#34867](https://github.com/pingcap/tidb/issues/34867) @[morgo](https://github.com/morgo)
    - `SHOW DATABASES LIKE …`が大文字と小文字を区別する問題を修正しました [#34766](https://github.com/pingcap/tidb/issues/34766) @[e1ijah1](https://github.com/e1ijah1)
    - `SHOW TABLE STATUS LIKE ...`が大文字と小文字を区別する問題を修正しました [#7518](https://github.com/pingcap/tidb/issues/7518) @[likzn](https://github.com/likzn)
    - `max-index-length`が非厳密モードでエラーを報告し続ける問題を修正しました [#34931](https://github.com/pingcap/tidb/issues/34931) @[e1ijah1](https://github.com/e1ijah1)
    - `ALTER COLUMN ... DROP DEFAULT`が機能しない問題を修正しました [#35018](https://github.com/pingcap/tidb/issues/35018) @[Defined2014](https://github.com/Defined2014)
    - テーブルを作成する際、列のデフォルト値とタイプが一貫しておらず、自動的に修正されない問題を修正しました [#34881](https://github.com/pingcap/tidb/issues/34881) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)
    - `DROP USER`を実行した後、`mysql.columns_priv`テーブルのデータが同期的に削除されない問題を修正しました [#35059](https://github.com/pingcap/tidb/issues/35059) @[lcwangchao](https://github.com/lcwangchao)
    - 一部のシステムのスキーマ内でテーブルを作成することを禁止することでDDLの混み合いの問題を修正しました [#35205](https://github.com/pingcap/tidb/issues/35205) @[tangenta](https://github.com/tangenta)
    - パーティションされたテーブルのクエリが一部のケースで "index-out-of-range" および "non used index" エラーを報告する問題を修正しました [#35181](https://github.com/pingcap/tidb/issues/35181) @[mjonss](https://github.com/mjonss)
    - `INTERVAL expr unit + expr`がエラーを報告する可能性がある問題を修正しました [#30253](https://github.com/pingcap/tidb/issues/30253) @[mjonss](https://github.com/mjonss)
    - トランザクション内で作成された一時テーブルが見つからないバグを修正しました [#35644](https://github.com/pingcap/tidb/issues/35644) @[djshow832](https://github.com/djshow832)
    - `ENUM`列の照合を設定した場合にパニックが発生する問題を修正しました [#31637](https://github.com/pingcap/tidb/issues/31637) @[wjhuang2016](https://github.com/wjhuang2016)
    - 1つのPDノードがダウンした場合、`information_schema.TIKV_REGION_STATUS`のクエリが他のPDノードをリトライしないことによる失敗の問題を修正しました [#35708](https://github.com/pingcap/tidb/issues/35708) @[tangenta](https://github.com/tangenta)
- `SHOW CREATE TABLE …`が `SET character_set_results = GBK`の後にsetまたは`ENUM`列を正しく表示できない問題を修正しました [#31338](https://github.com/pingcap/tidb/issues/31338) @[tangenta](https://github.com/tangenta)
    - システム変数 `tidb_log_file_max_days`と `tidb_config`の誤ったスコープを修正しました [#35190](https://github.com/pingcap/tidb/issues/35190) @[morgo](https://github.com/morgo)
    - `SHOW CREATE TABLE`の出力がMySQLと互換性がない、または `ENUM`または`SET`列に対して不適切な問題を修正しました [#36317](https://github.com/pingcap/tidb/issues/36317) @[Defined2014](https://github.com/Defined2014)
    - テーブルを作成する際、`LONG BYTE`列の動作がMySQLと互換性がない問題を修正しました [#36239](https://github.com/pingcap/tidb/issues/36239) @[Defined2014](https://github.com/Defined2014)
    - `auto_increment = x`が一時テーブルに効果を与えない問題を修正しました [#36224](https://github.com/pingcap/tidb/issues/36224) @[djshow832](https://github.com/djshow832)
    - 同時に列を変更する際の誤ったデフォルト値を修正しました [#35846](https://github.com/pingcap/tidb/issues/35846) @[wjhuang2016](https://github.com/wjhuang2016)
    - 可用性を向上させるために、不健康なTiKVノードにリクエストを送信しないようにしました [#34906](https://github.com/pingcap/tidb/issues/34906) @[sticnarf](https://github.com/sticnarf)
    - `LOAD DATA`ステートメントで列リストが機能しない問題を修正しました [#35198](https://github.com/pingcap/tidb/issues/35198) @[SpadeA-Tang](https://github.com/SpadeA-Tang)
    - 一部のシナリオで疑似ロックが非一意的なセカンダリインデックスに誤って追加される問題を修正しました [#36235](https://github.com/pingcap/tidb/issues/36235) @[ekexium](https://github.com/ekexium)

+ TiKV

    - 悲観的トランザクションで `WriteConflict` エラーを報告しないようにしました [#11612](https://github.com/tikv/tikv/issues/11612) @[sticnarf](https://github.com/sticnarf)
    - 非同期コミットが有効になっている場合、悲観的トランザクションで重複したコミットレコードが発生する可能性のある問題を修正しました [#12615](https://github.com/tikv/tikv/issues/12615) @[sticnarf](https://github.com/sticnarf)
    - `storage.api-version`を `1` から `2` に変更する際にTiKVがパニックする問題を修正しました [#12600](https://github.com/tikv/tikv/issues/12600) @[pingyu](https://github.com/pingyu)
    - TiKVとPD間のRegionサイズ構成が一貫していない問題を修正しました [#12518](https://github.com/tikv/tikv/issues/12518) @[5kbpers](https://github.com/5kbpers)
    - TiKVがPDクライアントとの接続を維持し続ける問題を修正しました [#12506](https://github.com/tikv/tikv/issues/12506), [#12827](https://github.com/tikv/tikv/issues/12827) @[Connor1996](https://github.com/Connor1996)
    - 空の文字列に対する型変換を行う際にTiKVがパニックする問題を修正しました [#12673](https://github.com/tikv/tikv/issues/12673) @[wshwsh12](https://github.com/wshwsh12)
    - `DATETIME`値に分数と `Z`が含まれる場合に発生する時刻解析エラーの問題を修正しました [#12739](https://github.com/tikv/tikv/issues/12739) @[gengliqi](https://github.com/gengliqi)
    - Apply演算子によってTiKV RocksDBに書き込まれるperfコンテキストが粗い粒度である問題を修正しました [#11044](https://github.com/tikv/tikv/issues/11044) @[LykxSassinator](https://github.com/LykxSassinator)
    - バックアップ/インポート/CDCの設定が無効な場合にTiKVが起動に失敗する問題を修正しました [#12771](https://github.com/tikv/tikv/issues/12771) @[3pointer](https://github.com/3pointer)
  - ピアが分割されて同時に破棄される場合に発生する可能性があるパニックの問題を修正します [#12825](https://github.com/tikv/tikv/issues/12825) @[BusyJay](https://github.com/BusyJay)
    - 地域のマージプロセスでスナップショットによってログが追いつく場合に発生する可能性があるパニックの問題を修正します [#12663](https://github.com/tikv/tikv/issues/12663) @[BusyJay](https://github.com/BusyJay)
    - `max_sample_size` が `0` に設定されている場合に発生する統計解析時のパニックの問題を修正します [#11192](https://github.com/tikv/tikv/issues/11192) @[LykxSassinator](https://github.com/LykxSassinator)
    - Raft Engine が有効になっている場合に暗号化キーがクリアされない問題を修正します [#12890](https://github.com/tikv/tikv/issues/12890) @[tabokie](https://github.com/tabokie)
    - `get_valid_int_prefix` 関数が TiDB と互換性がない問題を修正します。 例えば、`FLOAT` タイプが誤って `INT` に変換される問題がありました [#13045](https://github.com/tikv/tikv/issues/13045) @[guo-shaoge](https://github.com/guo-shaoge)
    - 新しい地域のコミットログ期間が長すぎるため、QPS が低下する問題を修正します [#13077](https://github.com/tikv/tikv/issues/13077) @[Connor1996](https://github.com/Connor1996)
    - 地域のハートビートが中断された後に PD が TiKV に再接続しない問題を修正します [#12934](https://github.com/tikv/tikv/issues/12934) @[bufferflies](https://github.com/bufferflies)

+ ツール

    + バックアップとリストア（BR）

        - BR が速度制限されたバックアップタスクを完了した後に速度制限をリセットしない問題を修正します [#31722](https://github.com/pingcap/tidb/issues/31722) @[MoCuishle28](https://github.com/MoCuishle28)

## 貢献者

TiDB コミュニティの以下の貢献者に感謝します:

- [e1ijah1](https://github.com/e1ijah1)
- [PrajwalBorkar](https://github.com/PrajwalBorkar)
- [likzn](https://github.com/likzn)
- [rahulk789](https://github.com/rahulk789)
- [virusdefender](https://github.com/virusdefender)
- [joycse06](https://github.com/joycse06)
- [morgo](https://github.com/morgo)
- [ixuh12](https://github.com/ixuh12)
- [blacktear23](https://github.com/blacktear23)
- [johnhaxx7](https://github.com/johnhaxx7)
- [GoGim1](https://github.com/GoGim1)
- [renbaoshuo](https://github.com/renbaoshuo)
- [Zheaoli](https://github.com/Zheaoli)
- [fanrenhoo](https://github.com/fanrenhoo)
- [njuwelkin](https://github.com/njuwelkin)
- [wirybeaver](https://github.com/wirybeaver)
- [hey-kong](https://github.com/hey-kong)
- [fatelei](https://github.com/fatelei)
- [eastfisher](https://github.com/eastfisher): 初めての貢献者
- [Juneezee](https://github.com/Juneezee): 初めての貢献者