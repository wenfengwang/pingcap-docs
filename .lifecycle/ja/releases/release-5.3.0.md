---
title: TiDB 5.3 リリースノート
---

# TiDB 5.3 リリースノート

リリース日: 2021年11月30日

TiDB バージョン: 5.3.0

v5.3 では、主な新機能または改善点は以下の通りです:

+ アプリケーションロジックの簡略化とパフォーマンスの向上のために一時テーブルを導入
+ テーブルとパーティションの属性の設定をサポート
+ TiDB ダッシュボードでの最小権限を持つユーザーの作成をサポートし、システムのセキュリティを向上
+ TiDB でのタイムスタンプ処理フローを最適化し、全体的なパフォーマンスを向上
+ TiDB データ移行（DM）のパフォーマンスを向上し、MySQL から TiDB へのデータ移行においてより低いレイテンシでデータを移行
+ 複数の TiDB Lightning インスタンスを使用して並列インポートをサポートし、フルデータ移行の効率を向上
+ クラスタの現地情報の保存と復元を1つの SQL ステートメントでサポートし、実行計画に関連する問題のトラブルシューティングの効率を向上
+ データベースパフォーマンスの可観察性を向上するために、連続プロファイリングの実験機能をサポート
+ ストレージとコンピューティングエンジンを引き続き最適化し、システムのパフォーマンスと安定性を向上
+ Raftstore スレッドプールから I/O 操作を分離することで TiKV の書き込みレイテンシを低減（デフォルトでは無効）

## 互換性の変更

> **注意:**
>
> v5.3.0 へのアップグレード時に、すべての中間バージョンの互換性変更ノートを確認したい場合は、対応するバージョンの[リリースノート](/releases/release-notes.md)を確認できます。

### システム変数

|  変数名    |  変更タイプ    |  説明    |
| :---------- | :----------- | :----------- |
| [`tidb_enable_noop_functions`](/system-variables.md#tidb_enable_noop_functions-new-in-v40) | 変更 | TiDB で一時テーブルがサポートされたため、`CREATE TEMPORARY TABLE` と `DROP TEMPORARY TABLE` を有効にする必要はもはやありません。
| [`tidb_enable_pseudo_for_outdated_stats`](/system-variables.md#tidb_enable_pseudo_for_outdated_stats-new-in-v530) | 新規追加 | テーブルの統計情報が有効期限切れになった時の最適化プログラムの動作を制御します。デフォルト値は `ON` です。テーブルの変更行数が総行数の80%以上の場合（この割合は構成 [`pseudo-estimate-ratio`](/tidb-configuration-file.md#pseudo-estimate-ratio) で調整可能）、最適化プログラムは、総行数以外の統計情報が信頼性を失ったと判断し、疑似統計情報を使用します。`OFF` に設定しても統計情報が期限切れになっても、最適化プログラムはそれらを使用します。
| [`tidb_enable_tso_follower_proxy`](/system-variables.md#tidb_enable_tso_follower_proxy-new-in-v530) | 新規追加  | TSO Follower Proxy 機能を有効または無効にするかを決定します。デフォルト値は `OFF` で、つまり TSO Follower Proxy 機能は無効です。この場合、TiDB は PD リーダーからの TSO のみを取得します。この機能が有効な場合、TiDB は TSO 取得時にリクエストを均等にすべての PD ノードに送信します。その後、PD フォロワーは TSO リクエストを転送して PD リーダーの CPU 負荷を軽減します。
| [`tidb_tso_client_batch_max_wait_time`](/system-variables.md#tidb_tso_client_batch_max_wait_time-new-in-v530) | 新規追加 | TiDB が PD から TSO をリクエストする際のバッチ保存操作に対する最大待機時間を設定します。デフォルト値は `0` で、追加の待機時間はありません。
| [`tidb_tmp_table_max_size`](/system-variables.md#tidb_tmp_table_max_size-new-in-v530) | 新規追加  | 単一の[一時テーブル](/temporary-tables.md)の最大サイズを制限します。一時テーブルがこのサイズを超えると、エラーが発生します。

### 設定ファイルパラメータ

|  設定ファイル    |  設定項目  | 変更タイプ |  説明  |
| :---------- | :----------- | :----------- | :----------- |
| TiDB | [`prepared-plan-cache.capacity`](/tidb-configuration-file.md#capacity)  | 変更 | キャッシュされるステートメントの数を制御します。デフォルト値は `100` から `1000` に変更されます。
| TiKV | [`storage.reserve-space`](/tikv-configuration-file.md#reserve-space) | 変更 | TiKV の開始時にディスク保護用に予約されたスペースを制御します。v5.3.0 以降では、予約スペースの80%が、ディスクの操作および保守に必要な追加ディスクスペースに使用され、残りの20%が一時ファイルの保存に使用されます。
| TiKV | `memory-usage-limit` | 変更  | この設定項目は TiDB v5.3.0 で新規追加され、その値は storage.block-cache.capacity に基づいて計算されます。
| TiKV | [`raftstore.store-io-pool-size`](/tikv-configuration-file.md#store-io-pool-size-new-in-v530) | 新規追加 | Raft I/O タスクを処理するスレッドの許容数を制御します。このスレッドプールのサイズです。このスレッドプールのサイズを変更する場合は、[TiKV スレッドプールのパフォーマンスチューニング](/tune-tikv-thread-performance.md#performance-tuning-for-tikv-thread-pools)を参照してください。
| TiKV | [`raftstore.raft-write-size-limit`](/tikv-configuration-file.md#raft-write-size-limit-new-in-v530) | 新規追加 | ラフトデータがディスクに書き込まれる閾値を決定します。この設定項目の値よりデータのサイズが大きい場合、データがディスクに書き込まれます。`raftstore.store-io-pool-size` の値が `0` の場合、この設定項目は効果がありません。
| TiKV | `raftstore.raft-msg-flush-interval` | 新規追加 | ラフトメッセージがバッチで送信される間隔を決定します。この設定項目で指定された間隔ごとに、ラフトメッセージが送信されます。`raftstore.store-io-pool-size` の値が `0` の場合、この設定項目は効果がありません。
| TiKV | `raftstore.raft-reject-transfer-leader-duration`  | 削除 | リーダーが新しく追加されたノードに移動する最小期間を決定します。
| PD | [`log.file.max-days`](/pd-configuration-file.md#max-days) | 変更 | ログが保持される最大日数を制御します。デフォルト値は `1` から `0` に変更されます。
| PD | [`log.file.max-backups`](/pd-configuration-file.md#max-backups) | 変更 | 保持されるログの最大数を制御します。デフォルト値は `7` から `0` に変更されます。
| PD | [`patrol-region-interval`](/pd-configuration-file.md#patrol-region-interval) | 変更 | replicaChecker がリージョンの健康状態をチェックする実行頻度を制御します。この値が小さいほど、replicaChecker が高速に実行されます。通常、このパラメータを調整する必要はありません。デフォルト値は `100ms` から `10ms` に変更されます。
| PD | [`max-snapshot-count`](/pd-configuration-file.md#max-snapshot-count) | 変更 | 単一のストアが同時に受信または送信するスナップショットの最大数を制御します。PD スケジューラはこの設定に依存して、通常のトラフィックに使用されるリソースが先に取られないようにします。デフォルト値は `3` から `64` に変更されます。
| PD | [`max-pending-peer-count`](/pd-configuration-file.md#max-pending-peer-count) | 変更 | 単一のストアで保留中のピアの最大数を制御します。PD スケジューラは、一部のノードで古いログのリージョンが生成されるのを防ぐためにこの設定に依存します。デフォルト値は `16` から `64` に変更されます。
| TiDB Lightning | `meta-schema-name` | 新規追加 | ターゲットクラスタ内の各 TiDB Lightning インスタンスのメタ情報が格納されるスキーマ名。デフォルト値は "lightning_metadata" です。

### その他

- 一時テーブル:

    - v5.3.0 より前の TiDB クラスタでローカル一時テーブルを作成した場合、これらのテーブルは実際には通常のテーブルであり、クラスタが v5.3.0 または以降のバージョンにアップグレードされると通常のテーブルとして扱われます。v5.3.0 または以降のバージョンの TiDB クラスタでグローバル一時テーブルを作成した場合、クラスタを v5.3.0 より前のバージョンにダウングレードすると、これらのテーブルが通常のテーブルとして扱われ、データエラーが発生します。
    - v5.3.0 以降、TiCDC および BR は[グローバル一時テーブル](/temporary-tables.md#global-temporary-tables)をサポートします。v5.3.0 より前の TiCDC および BR を使用して、グローバル一時テーブルを下流にレプリケートすると、テーブル定義エラーが発生します。
    - 次のクラスタが v5.3.0 または以降であることが期待されます。そうでない場合、グローバル一時テーブルを作成するとデータエラーが報告されます:

        - TiDB マイグレーションツールを使用してインポートするクラスタ
        - TiDB マイグレーションツールを使用して復元したクラスタ
        - TiDB マイグレーションツールを使用してレプリケーションタスクで下流クラスタ
    - 一時テーブルの互換性情報については、[MySQL 一時テーブルとの互換性](/temporary-tables.md#compatibility-with-mysql-temporary-tables)と[その他の TiDB 機能との互換性制限](/temporary-tables.md#compatibility-restrictions-with-other-tidb-features)を参照してください。

- v5.3.0以前にリリースされた場合、システム変数が不正な値に設定されていると、TiDBはエラーを報告します。v5.3.0以降のリリースでは、システム変数が不正な値に設定されている場合、「|Warning | 1292 | Truncated incorrect xxx: 'xx'」のような警告が付いて成功を返します。
- `SHOW CREATE VIEW`を実行するために`SHOW VIEW`権限が必要である問題を修正しました。現在は`SHOW CREATE VIEW`ステートメントを実行するには`SHOW VIEW`権限が必要です。
- `sql_auto_is_null`というシステム変数がnoop関数に追加されました。`tidb_enable_noop_functions = 0/OFF`の場合、この変数の値を変更するとエラーが発生します。
- `GRANT ALL ON performance_schema.*`構文はもはや許可されていません。TiDBでこのステートメントを実行するとエラーが発生します。
- v5.3.0より前に新しいインデックスが追加された場合、自動解析が指定された時間枠外で予期しないタイミングでトリガーされる問題を修正しました。v5.3.0では、`tidb_auto_analyze_start_time`および`tidb_auto_analyze_end_time`変数を通じて時間枠を設定した後、自動解析はその時間枠内のみトリガーされます。
- プラグインのデフォルトの保存ディレクトリを`""`から`/data/deploy/plugin`に変更しました。
- DMコードは、[TiCDCコードリポジトリの"dm"フォルダ](https://github.com/pingcap/tiflow/tree/master/dm)に移行されました。現在、DMはTiDBのバージョン番号に従います。v2.0.xの隣に、新しいDMバージョンがv5.3.0となり、v2.0.xからv5.3.0にリスクなしでアップグレードできます。
- Prometheusのデフォルトのデプロイバージョンがv2.8.1から[v2.27.1](https://github.com/prometheus/prometheus/releases/tag/v2.27.1)にアップグレードされました。このバージョンは2021年5月にリリースされました。このバージョンでは、より多くの機能が提供され、セキュリティの問題が修正されています。v2.27.1では、v2.8.1と比較してアラートの時刻表現がUnixタイムスタンプからUTCに変更されています。詳細は、詳細については [Prometheus commit](https://github.com/prometheus/prometheus/commit/7646cbca328278585be15fa615e22f2a50b47d06)を参照してください。

## 新機能

### SQL

- **データ配置ルールをSQLインターフェースを使用して設定する（実験的な機能）**

    SQLインターフェースを使用してデータ配置ルールを設定する`[CREATE | ALTER] PLACEMENT POLICY`構文をサポートします。この機能を使用すると、特定のリージョン、データセンター、ラック、ホスト、またはレプリカ数のルールに従ってテーブルやパーティションをスケジュールできます。この機能を使用することで、アプリケーションのコストを下げ、柔軟性を高める要求に応えることができます。典型的なユーザーシナリオは以下のとおりです。

    - 異なるアプリケーションの複数のデータベースをマージして、データベースの保守コストを下げ、ルール設定を通じてアプリケーションリソースの分離を実現します。
    - 重要なデータのレプリカ数を増やして、アプリケーションの可用性とデータの信頼性を高めます。
    - 新しいデータをSSDに保存し、古いデータをHHDに保存して、データのアーカイブと保管のコストを下げます。
    - ホットスポットデータのリーダーを高性能なTiKVインスタンスにスケジュールします。
    - 低コストのストレージメディアにコールドデータを分離し、コスト効率を高めます。

    [ユーザードキュメント](/placement-rules-in-sql.md), [#18030](https://github.com/pingcap/tidb/issues/18030)

- **一時テーブル**

    `CREATE [GLOBAL] TEMPORARY TABLE`ステートメントをサポートし、一時テーブルを作成できます。この機能を使用すると、アプリケーションの計算プロセスで生成された一時データを簡単に管理できます。一時データはメモリに保存され、`tidb_tmp_table_max_size`変数を使用して一時テーブルのサイズを制限できます。TiDBは以下の種類の一時テーブルをサポートしています。

    - グローバル一時テーブル
        - クラスタ内のすべてのセッションから見え、テーブルのスキーマが永続的です。
        - トランザクションレベルのデータ分離を提供します。一時データはトランザクション内でのみ有効です。トランザクションが終了すると、データは自動的に削除されます。
    - ローカル一時テーブル
        - 現在のセッションのみに見え、テーブルスキーマは永続的ではありません。
        - テーブル名を複製できます。アプリケーションのための複雑な命名ルールを設計する必要はありません。
        - セッションレベルのデータ分離を提供し、よりシンプルなアプリケーションロジックを設計できます。トランザクションが終了すると、一時テーブルが削除されます。

    [ユーザードキュメント](/temporary-tables.md), [#24169](https://github.com/pingcap/tidb/issues/24169)

- **`FOR UPDATE OF TABLES`構文をサポート**

    複数のテーブルを結合するSQLステートメントに対して、TiDBは`OF TABLES`に含まれるテーブルに関連した行に対して厳格なロックを取得することをサポートします。

    [ユーザードキュメント](/sql-statements/sql-statement-select.md), [#28689](https://github.com/pingcap/tidb/issues/28689)

- **テーブル属性**

    表またはパーティションの属性を設定する`ALTER TABLE [PARTITION] ATTRIBUTES`ステートメントをサポートします。現在、TiDBは`merge_option`属性の設定のみをサポートしています。この属性を追加することにより、リージョンのマージ動作を明示的に制御できます。

    ユーザーシナリオ: `SPLIT TABLE`操作を実行する際、PDパラメータ[`split-merge-interval`](/pd-configuration-file.md#split-merge-interval)によって、一定期間データが挿入されない場合には、空のリージョンがデフォルトで自動的にマージされます。この場合、テーブル属性を`merge_option=deny`に設定して、自動的なリージョンのマージを回避することができます。

    [ユーザードキュメント](/table-attributes.md), [#3839](https://github.com/tikv/pd/issues/3839)

### セキュリティ

- **TiDB Dashboardで最小特権のユーザーを作成するサポート**

    TiDB Dashboardのアカウントシステムは、TiDB SQLのアカウントシステムと一致しています。TiDB Dashboardにアクセスするユーザーは、TiDB SQLユーザーの権限に基づいて認証および承認されます。そのため、TiDB Dashboardでは制限された権限、または読み取り専用権限のみが必要です。TiDB Dashboardにアクセスするための最小特権原則に基づいてユーザーを構成できるため、特権の高いユーザーのアクセスを避け、セキュリティを向上できます。

    TiDB Dashboardへのアクセス用に最小特権のSQLユーザーを作成し、サインインすることをお勧めします。これにより、特権の高いユーザーのアクセスを避け、セキュリティを向上できます。

    [ユーザードキュメント](/dashboard/dashboard-user.md)

### パフォーマンス

- **PDのタイムスタンプ処理フローを最適化**

    TiDBはPDのタイムスタンプ処理フローを最適化し、PD Follower Proxyを有効にして、PDクライアントがバッチでTSOをリクエストする際に必要なバッチ待機時間を変更することで、PDのタイムスタンプ処理負荷を低減します。これにより、システム全体のスケーラビリティが向上します。

    - システム変数[`tidb_enable_tso_follower_proxy`](/system-variables.md#tidb_enable_tso_follower_proxy-new-in-v530)を使用して、PD Follower Proxyを有効または無効にできます。PDのTSOリクエスト負荷が高い場合、PD Follower Proxyを有効にすることで、リクエストサイクル中に収集されたTSOリクエストをフォロワーからリーダーノードにバッチで転送できます。これにより、クライアントとリーダー間の直接相互作用の回数が効果的に減少し、リーダーへの負荷が軽減され、TiDB全体のパフォーマンスが向上します。

    > **注意:**
    >
    > クライアント数が少なく、PDリーダーCPU負荷がいっぱいでない場合、PD Follower Proxyを有効にすることは推奨されません。

    - システム変数[`tidb_tso_client_batch_max_wait_time`](/system-variables.md#tidb_tso_client_batch_max_wait_time-new-in-v530)を使用して、PDクライアントがTSOをバッチリクエストするための最大待機時間を設定できます。この時間の単位はミリ秒です。PDにTSOリクエスト負荷が高い場合、この待機時間を増やすことで、より大きなバッチサイズを得ることができ、負荷を軽減し、スループットを向上させることができます。

    > **注意:**
    >
    > TSOリクエスト負荷が高くない場合、この変数値を変更することは推奨されません。

    [ユーザードキュメント](/system-variables.md#tidb_tso_client_batch_max_wait_time-new-in-v530), [#3149](https://github.com/tikv/pd/issues/3149)

### 安定性

- **一部のストアが永続的に破損した後のオンライン不安全なリカバリのサポート（実験的な機能）**

    `pd-ctl unsafe remove-failed-stores`コマンドをサポートし、オンラインデータの不安全なリカバリが実行できます。ほとんどのデータレプリカが永続的な損傷（ディスクの損傷など）といった問題に直面し、これらの問題がアプリケーション内のデータ範囲が読み取り不可能または書き込み不可能にする場合、PDで実装されたオンライン不安全なリカバリ機能を使用してデータを回復し、データが再び読み取り可能または書き込み可能になるようにできます。

    機能関連の操作を実行する際には、TiDBチームのサポートを得ることをお勧めします。

    [ユーザードキュメント](/online-unsafe-recovery.md), [#10483](https://github.com/tikv/tikv/issues/10483)

### データ移行

- **DMレプリケーションのパフォーマンス向上**

    MySQLからTiDBへの低レイテンシーデータレプリケーションを確保するための以下の機能をサポートします。

    - 単一の行上の複数の更新を1つのステートメントにまとめる
    - 複数の行のバッチ更新を1つのステートメントにマージする

- **より良いDMクラスタの管理のためのDM OpenAPIを追加する（実験的な機能）**
    DMは、DMクラスタをクエリおよび操作するためのOpenAPI機能を提供します。これは[dmctlツール](/dm/dmctl-introduction.md)の機能に類似しています。

現在、DM OpenAPIは実験的な機能であり、デフォルトでは無効になっています。本番環境での使用は推奨されていません。

[ユーザードキュメント](/dm/dm-open-api.md)

- **TiDB Lightning Parallel Import**

    TiDB Lightningは、オリジナルの機能を拡張する並列インポート機能を提供します。複数のLightningインスタンスを同時に展開し、単一または複数のテーブルを下流のTiDBに並列にインポートすることができます。お客様が使用する方法を変更することなく、データ移行能力を大幅に向上させ、データをよりリアルタイムに移行してさらなる処理、統合、分析が可能となります。これにより企業のデータ管理の効率が向上します。

    テストでは、10のTiDB Lightningインスタンスを使用して、合計20テラバイトのMySQLデータを8時間以内にTiDBにインポートできます。複数テーブルのインポートのパフォーマンスも向上しています。単一のTiDB Lightningインスタンスは250 GiB/hでのインポートをサポートし、全体の移行は元のパフォーマンスよりも8倍速くなります。

    [ユーザードキュメント](/tidb-lightning/tidb-lightning-distributed-import.md)

- **TiDB Lightning Prechecks**

    TiDB Lightningは、マイグレーションタスクを実行する前に構成のチェックを行う機能を提供します。これはデフォルトで有効です。この機能はディスクスペースや実行構成に関するいくつかのルーチンチェックを自動的に実行します。主な目的は、その後のインポートプロセス全体がスムーズに進むことを確認することです。

    [ユーザードキュメント](/tidb-lightning/tidb-lightning-prechecks.md)

- **TiDB LightningはGBK文字セットのファイルのインポートをサポート**

    ソースデータファイルの文字セットを指定することができます。TiDB Lightningは、インポートプロセス中にソースファイルを指定された文字セットからUTF-8エンコーディングに変換します。

    [ユーザードキュメント](/tidb-lightning/tidb-lightning-configuration.md)

- **Sync-diff-inspectorの改善**

    - 比較速度を375MB/sから700MB/sに向上
    - 比較中にTiDBノードのメモリ消費をほぼ半分に削減
    - 比較中に進捗バーを最適化し表示する

    [ユーザードキュメント](/sync-diff-inspector/sync-diff-inspector-overview.md)

### 診断の効率

- **クラスターの現場情報を保存および復元**

    TiDBクラスターの問題を特定しトラブルシューティングする際には、システムおよびクエリプランの情報を提供する必要があります。これらの情報を簡単かつ効率的に取得し、トラブルシューティングを行い、問題を簡単にアーカイブするために、TiDB v5.3.0で`PLAN REPLAYER`コマンドが導入されました。このコマンドにより、クラスターの現場情報を簡単に保存および復元することができ、トラブルシューティングの効率が向上し、問題を管理するための手助けとなります。

    `PLAN REPLAYER`の特徴は以下の通りです：

    - オンサイトトラブルシューティング時のTiDBクラスターの情報をZIP形式のファイルにエクスポートし、保存用に提供します。
    - 他のTiDBクラスターからエクスポートされたZIP形式のファイルをインポートします。このファイルには、前者のTiDBクラスターにおけるオンサイトトラブルシューティング時の情報が含まれます。

    [ユーザードキュメント](/sql-plan-replayer.md)、[#26325](https://github.com/pingcap/tidb/issues/26325)

### TiDBデータ共有サブスクリプション

- **TiCDCの最終整合性レプリケーション**

    TiCDCは、災害シナリオでの最終整合性レプリケーション機能を提供します。主要なTiDBクラスターで災害が発生し、短期間でサービスを再開できない場合、TiCDCはセカンダリクラスター内のデータの整合性を保証する機能を提供する必要があります。同時に、TiCDCはビジネスが迅速にトラフィックをセカンダリクラスターに切り替えることを可能にし、データベースが長時間利用できなくなり、ビジネスに影響を与えることを避けます。

    この機能はTiCDCがTiDBクラスターから増分データをTiDB/Aurora/MySQL/MariaDBのセカンダリ関係データベースにレプリケートすることをサポートします。主要なクラスターがクラッシュした場合、TiCDCは、災害前にTiCDCのレプリケーション状態が正常でレプリケーションの遅延が小さい条件下で、セカンダリクラスターを主要クラスター内の特定スナップショットに回復できます。これによりデータ損失が30分以下となり、RTO <= 5分、RPO <= 30分となります。

    [ユーザードキュメント](/ticdc/ticdc-sink-to-mysql.md#eventually-consistent-replication-in-disaster-scenarios)

- **TiCDCはTiCDCタスクを管理するためのHTTPプロトコルOpenAPIをサポート**

    TiDB v5.3.0以降では、TiCDC OpenAPIが一般提供（GA）されています。本番環境でOpenAPIを使用してTiCDCクラスターをクエリおよび操作できます。

### デプロイとメンテナンス

- **連続プロファイリング（実験的な機能）**

    TiDBダッシュボードは、連続プロファイリング機能をサポートし、TiDBクラスターが実行されているときに実時間でインスタンスパフォーマンス分析結果を自動的に保存します。フレームグラフでパフォーマンス分析結果を確認でき、トラブルシューティング時間が短縮されます。

    この機能はデフォルトで無効になっており、TiDBダッシュボードの**連続プロファイル**ページで有効にする必要があります。

    この機能は、TiUP v1.7.0以上を使用してアップグレードまたはインストールされたクラスターでのみ利用可能です。

    [ユーザードキュメント](/dashboard/continuous-profiling.md)

## テレメトリ

TiDBは、TEMPORARY TABLE機能の使用の有無に関する情報をテレメトリレポートに追加しました。これにはテーブル名やテーブルデータは含まれません。

テレメトリに関する詳細およびこの動作を無効にする方法については、[Telemetry](/telemetry.md)を参照してください。

## 削除された機能

TiCDC v5.3.0から、TiDBクラスター間の循環レプリケーション機能（v5.0.0の実験的な機能）が削除されました。TiCDCをアップグレードする前にこの機能を使用してデータをレプリケートしていた場合、アップグレード後でも関連するデータには影響がありません。

## 改良点

+ TiDB

    - クエリに影響を受けるSQLステートメントをデバッグログに表示し、問題の診断に役立つ [#27718](https://github.com/pingcap/tidb/issues/27718)
    - SQL論理レイヤーでデータのバックアップおよびデータのリストア時のサイズを表示するサポートを追加 [#27247](https://github.com/pingcap/tidb/issues/27247)
    - `tidb_analyze_version`が`2`の場合のANALYZEのデフォルトの収集ロジックを改善し、収集を加速しリソースオーバーヘッドを削減
    - `ANALYZE TABLE table_name COLUMNS col_1, col_2, ... , col_n`構文を導入。この構文により、広範なテーブルの統計情報を一部の列のみに収集できるようになり、統計情報の収集速度が向上します。

+ TiKV

    - ストレージの安定性向上のためのディスクスペース保護を強化
    - TiKVがディスクフルエラー時にパニックする可能性がある問題を解決するため、TiKVはディスク残り容量が過剰なトラフィックによって枯渇されることを防ぐ2段階の閾値防御メカニズムを導入します。また、閾値がトリガされた際にスペースを回収する能力を提供します。残り容量の閾値がトリガされると、一部の書き込み操作が失敗し、TiKVはディスクフルエラーとディスクフルノードのリストを返します。この場合、スペースを回復してサービスを復元するには、`Drop/Truncate Table`を実行するか、ノードをスケーリングアウトする必要があります。
    - L0フローコントロールのアルゴリズムを単純化 [#10879](https://github.com/tikv/tikv/issues/10879)
    - Raftクライアントモジュールのエラーログレポートを改善 [#10944](https://github.com/tikv/tikv/pull/10944)
    - 性能ボトルネックとなることを回避するため、ログスレッドの改善
    - 書き込みクエリのより多くの統計タイプを追加
    - I/O操作をRaftstoreスレッドプールから分離することで書き込み遅延を低減（デフォルトでは無効）。調整についての詳細については[Tune TiKV Thread Pool Performance](/tune-tikv-thread-performance.md)を参照してください [#10540](https://github.com/tikv/tikv/issues/10540)

+ PD

    - ホットスポットスケジューラーでQPS次元に書き込みクエリのより多くのタイプを追加 [#3869](https://github.com/tikv/pd/issues/3869)
    - スケジューラーのパフォーマンスを向上させるためにバランスリージョンスケジューラーのリトライ制限を動的に調整するサポートを追加 [#3744](https://github.com/tikv/pd/issues/3744)
    - TiDBダッシュボードをv2021.10.08.1にアップデート [#4070](https://github.com/tikv/pd/pull/4070)
    - 不健康なピアを持つリージョンのスケジュールを許可するように、リーダースケジューラーが速い終了プロセスをサポート [#4093](https://github.com/tikv/pd/issues/4093)
    - スケジューラーの終了プロセスを高速化するための改良 [#4146](https://github.com/tikv/pd/issues/4146)

+ TiFlash

    - TableScanオペレーターの実行効率を大幅に向上
    - Exchangeオペレーターの実行効率を向上
    - ストレージエンジンのGC中の書き込み増幅とメモリ使用量を低減（実験的な機能）
- TiFlashの再起動時にTiFlashの安定性と利用可能性を向上させ、再起動後のクエリ障害の可能性を低減します
- MPPエンジンに複数の新しいStringおよびTime関数のプッシュダウンをサポート

  - String関数: LIKE pattern, FORMAT(), LOWER(), LTRIM(), RTRIM(), SUBSTRING_INDEX(), TRIM(), UCASE(), UPPER()
  - 数学関数: ROUND (decimal, int)
  - 日付および時刻関数: HOUR(), MICROSECOND(), MINUTE(), SECOND(), SYSDATE()
  - 型変換関数: CAST(time, real)
  - 集約関数: GROUP_CONCAT(), SUM(enum)

- 512ビットSIMDをサポート
- 古いデータのクリーンアップアルゴリズムを強化し、ディスク使用量を減らし、ファイルの読み取りを効率的に行います
- 一部の非LinuxシステムでダッシュボードがメモリやCPU情報を表示しない問題を修正
- TiFlashログファイルの命名スタイルをTiKVに一貫させ、logger.countおよびlogger.sizeの動的変更をサポートします
- 列ベースファイルのデータ検証能力を向上（チェックサム、実験的機能）

+ ツール

  + TiCDC

    - Kafkaシンク構成項目`MaxMessageBytes`のデフォルト値を64MBから1MBに減らし、大きなメッセージがKafka Brokerによって拒否される問題を修正する[#3104](https://github.com/pingcap/tiflow/pull/3104)
    - レプリケーションパイプラインでメモリ使用量を削減する[#2553](https://github.com/pingcap/tiflow/issues/2553) [#3037](https://github.com/pingcap/tiflow/pull/3037) [#2726](https://github.com/pingcap/tiflow/pull/2726)
    - 同期リンク、メモリGC、およびストックデータスキャンプロセスの可観測性を向上させるための監視項目と警告ルールを最適化する[#2735](https://github.com/pingcap/tiflow/pull/2735) [#1606](https://github.com/pingcap/tiflow/issues/1606) [#3000](https://github.com/pingcap/tiflow/pull/3000) [#2985](https://github.com/pingcap/tiflow/issues/2985) [#2156](https://github.com/pingcap/tiflow/issues/2156)
    - 同期タスクのステータスが正常な場合、ユーザーを誤誘導しないように歴史的なエラーメッセージを表示しません[#2242](https://github.com/pingcap/tiflow/issues/2242)

## バグ修正

+ TiDB

  - パーティション化されたテーブルの集約演算子をプッシュダウンする際にスキーマ列の浅いコピーによる誤った実行計画が原因で発生するエラーを修正[#27797](https://github.com/pingcap/tidb/issues/27797) [#26554](https://github.com/pingcap/tidb/issues/26554)
  - `plan cache`が符号なしフラグの変更を検出できない問題を修正[#28254](https://github.com/pingcap/tidb/issues/28254)
  - パーティション関数が範囲外の場合の誤ったパーティション剪定を修正[#28233](https://github.com/pingcap/tidb/issues/28233)
  - 一部のケースで`join`のための無効な計画をプランナーがキャッシュする問題を修正[#28087](https://github.com/pingcap/tidb/issues/28087)
  - ハッシュ列型が`enum`の場合の誤った`IndexLookUpJoin`を修正[#27893](https://github.com/pingcap/tidb/issues/27893)
  - アイドル接続のリサイクルが一部のレアケースでリクエストの送信をブロックする可能性があるバッチクライアントバグを修正 [#27688](https://github.com/pingcap/tidb/pull/27688)
  - ターゲットクラスターでチェックサムを実行できない場合にTiDB Lightningのパニック問題を修正[#27686](https://github.com/pingcap/tidb/pull/27686)
  - 一部のケースで`date_add`および`date_sub`関数の誤った結果を修正[#27232](https://github.com/pingcap/tidb/issues/27232)
  - ベクトル化された式で`hour`関数の誤った結果を修正[#28643](https://github.com/pingcap/tidb/issues/28643)
  - MySQL 5.1またはそれ以前のクライアントバージョンに接続する際の認証エラーを修正  [#27855](https://github.com/pingcap/tidb/issues/27855)
  - 新しいインデックスが追加されると、自動分析が指定された時間外にトリガーされる可能性がある問題を修正[#28698](https://github.com/pingcap/tidb/issues/28698)
  - 任意のセッション変数を設定すると`tidb_snapshot`が無効になるバグを修正[#28683](https://github.com/pingcap/tidb/pull/28683)
  - 多くの欠落したピアリージョンを持つクラスターに対してBRが機能しない問題を修正[#27534](https://github.com/pingcap/tidb/issues/27534)
  - サポートされていない`cast`がTiFlashにプッシュダウンされた場合の`DECIMAL overflow`のエラーメッセージが欠落している問題を修正[#23907](https://github.com/pingcap/tidb/issues/23907)
  - `%s value is out of range in '%s'`エラーメッセージに`DECIMAL overflow`が含まれていない問題を修正 [#27964](https://github.com/pingcap/tidb/issues/27964)
  - 一部のコーナーケースでMPPノードの可用性検出が機能しない問題を修正[#3118](https://github.com/pingcap/tics/issues/3118)
  - `MPP task ID`の割り当て時の`DATA RACE`問題を修正[#27952](https://github.com/pingcap/tidb/issues/27952)
  - 空の`dual table`を削除した後のMPPクエリの`INDEX OUT OF RANGE`エラーを修正[#28250](https://github.com/pingcap/tidb/issues/28250)
  - MPPクエリの不正なエラーログ`cannot found column in Schema column`の問題を修正[#28149](https://github.com/pingcap/tidb/pull/28149)
  - TiFlashがシャットダウンされるとTiDBがパニックする可能性がある問題を修正[#28096](https://github.com/pingcap/tidb/issues/28096)
  - 安全でない3DES（Triple Data Encryption Algorithm）ベースのTLS暗号スイートのサポートを削除する[#27859](https://github.com/pingcap/tidb/pull/27859)
  - プレチェックでLightningがオフラインTiKVノードに接続し、インポートの失敗を引き起こす問題を修正[#27826](https://github.com/pingcap/tidb/pull/27826)
  - 多くのファイルをテーブルにインポートする際にプレチェックにかかる時間が過剰である問題を修正[#27605](https://github.com/pingcap/tidb/issues/27605)
  - 表示を間違える`between`の式で間違った照合が推論される問題を修正[#27146](https://github.com/pingcap/tidb/issues/27146)
  - `group_concat`関数が照合を考慮していない問題を修正[#27429](https://github.com/pingcap/tidb/issues/27429)
  - `extract`関数の引数が負の期間の場合に発生する結果の誤りを修正[#27236](https://github.com/pingcap/tidb/issues/27236)
  - `NO_UNSIGNED_SUBTRACTION`が設定されている場合にパーティションの作成に失敗する問題を修正[#26765](https://github.com/pingcap/tidb/issues/26765)
  - 列の剪定および集約プッシュダウンに副作用を持つ式を回避する[#27106](https://github.com/pingcap/tidb/issues/27106)
  - 不要なgRPCログを削除する[#24190](https://github.com/pingcap/tidb/issues/24190)
  - 精度関連の問題を修正するために有効な10進桁の長さを制限する[#3091](https://github.com/pingcap/tics/issues/3091)
  - `plus`式でオーバーフローをチェックする間違った方法を修正する[#26977](https://github.com/pingcap/tidb/issues/26977)
  - `new collation`データを持つテーブルから統計情報をダンプする際の`data too long`エラーの問題を修正[#27024](https://github.com/pingcap/tidb/issues/27024)
  - リトライされたトランザクションのステートメントが`TIDB_TRX`に含まれない問題を修正[#28670](https://github.com/pingcap/tidb/pull/28670)
  - `CREATE SCHEMA`が新しいスキーマの場合に`character_set_server`および`collation_server`で指定された文字セットを使用しない問題を修正[#27214](https://github.com/pingcap/tidb/issues/27214)

+ TiKV
- リージョンの移行時にRaftstoreのデッドロックが発生してTiKVが利用できなくなる問題を修正しました。対処方法はスケジューリングを無効にして利用できないTiKVを再起動することです [#10909](https://github.com/tikv/tikv/issues/10909)
- CDCが頻繁にCongestエラーによるスキャンのリトライを追加する問題を修正しました [#11082](https://github.com/tikv/tikv/issues/11082)
- チャネルがフルの場合にRaft接続が切断される問題を修正しました [#11047](https://github.com/tikv/tikv/issues/11047)
- Raftクライアントの実装においてバッチメッセージが大きすぎる問題を修正しました [#9714](https://github.com/tikv/tikv/issues/9714)
- `resolved_ts`において一部のコルーチンがリークする問題を修正しました [#10965](https://github.com/tikv/tikv/issues/10965)
- 応答のサイズが4 GiBを超えた場合にコプロセッサーでパニックが発生する問題を修正しました [#9012](https://github.com/tikv/tikv/issues/9012)
- スナップショットガベージコレクション（GC）がスナップショットファイルをガベージコレクトできない場合にGCスナップショットファイルを逃す問題を修正しました [#10813](https://github.com/tikv/tikv/issues/10813)
- コプロセッサーリクエストの処理中にタイムアウトによるパニックが発生する問題を修正しました [#10852](https://github.com/tikv/tikv/issues/10852)
- 統計スレッドの監視データによるメモリリークを修正しました [#11195](https://github.com/tikv/tikv/issues/11195)
- 一部のプラットフォームからcgroup情報を取得する際にパニックが発生する問題を修正しました [#10980](https://github.com/tikv/tikv/pull/10980)
- MVCC削除バージョンがコンパクションフィルタGCによって削除されないことによるスキャンパフォーマンスの低下を修正しました [#11248](https://github.com/tikv/tikv/pull/11248)

+ PD

    - 構成されたピア数を超えるピアのデータと保留中のステータスを持つピアを不正確に削除するPDの問題を修正しました [#4045](https://github.com/tikv/pd/issues/4045)
    - ダウンしたピアを適切な時間で修正しないPDの問題を修正しました [#4077](https://github.com/tikv/pd/issues/4077)
    - 空のリージョンをスケジュールできないスキャッターレンジスケジューラーの問題を修正しました [#4118](https://github.com/tikv/pd/pull/4118)
    - キーマネージャーがあまりにも多くのCPUを消費する問題を修正しました [#4071](https://github.com/tikv/pd/issues/4071)
    - ホットリージョンスケジューラーの構成を設定する際に発生するデータ競合の問題を修正しました [#4159](https://github.com/tikv/pd/issues/4159)
    - スタックしたリージョンシンカによって遅延したリーダー選出を修正しました [#3936](https://github.com/tikv/pd/issues/3936)

+ TiFlash

    - TiFlashストアサイズの統計が正確でない問題を修正しました
    - ライブラリ`nsl`の不在によりTiFlashが特定のプラットフォームで起動に失敗する問題を修正しました
    - 書き込みプレッシャーがかかる場合に無限待ちをブロックする `wait index` をブロックしました（デフォルトタイムアウトは5分）、これによりTiFlashがデータレプリケーションのために長すぎる時間待たされることを防止します
    - ログ量が多い場合にログ検索の遅延と結果が得られない問題を修正しました
    - 古い履歴的なログを検索する際に最新のログのみが検索される問題を修正しました
    - 新しい照合が有効になっている場合に、間違った結果が得られる可能性のある問題を修正しました
    - SQLステートメントに極端に長いネストされた式が含まれる場合にパースエラーが発生する可能性がある問題を修正しました
    - Exchange演算子の `Block schema mismatch` エラーが発生する可能性がある問題を修正しました
    - Decimal型を比較する際に `Can't compare` エラーが発生する可能性がある問題を修正しました
    - `substringUTF8` 関数の3番目の引数が定数でない場合に `3rd arguments of function substringUTF8 must be constants` エラーが発生する可能性がある問題を修正しました

+ Tools

    + TiCDC

        - アップストリームのTiDBインスタンスが予期せず終了した際にTiCDCレプリケーションタスクが終了する可能性がある問題を修正しました [#3061](https://github.com/pingcap/tiflow/issues/3061)
        - TiKVが同じリージョンに重複したリクエストを送信する際にTiCDCプロセスがパニックする可能性がある問題を修正しました [#2386](https://github.com/pingcap/tiflow/issues/2386)
        - 下流のTiDB/MySQLの可用性を検証する際に不必要なCPUを消費する問題を修正しました [#3073](https://github.com/pingcap/tiflow/issues/3073)
        - TiCDCによって生成されるKafkaメッセージのボリュームが`max-message-size`で制約されない問題を修正しました [#2962](https://github.com/pingcap/tiflow/issues/2962)
        - Kafkaメッセージの書き込み中にエラーが発生するとTiCDC同期タスクが一時停止する可能性がある問題を修正しました [#2978](https://github.com/pingcap/tiflow/issues/2978)
        - `force-replicate` が有効になっている場合に有効なインデックスを持たない一部の分割されたテーブルが無視される可能性がある問題を修正しました [#2834](https://github.com/pingcap/tiflow/issues/2834)
        - ストックデータのスキャンに失敗する可能性があり、スキャンが長時間かかるとTiKVがGCを実行するためにスキャンが失敗する可能性がある問題を修正しました [#2470](https://github.com/pingcap/tiflow/issues/2470)
        - 一部の型の列をOpen Protocol形式にエンコードする際にパニックが発生する可能性がある問題を修正しました [#2758](https://github.com/pingcap/tiflow/issues/2758)
        - 一部の型の列をAvro形式にエンコードする際にパニックが発生する可能性がある問題を修正しました [#2648](https://github.com/pingcap/tiflow/issues/2648)

    + TiDB Binlog

        - 大部分のテーブルがフィルタされた場合に特殊な負荷下でチェックポイントを更新できない問題を修正しました [#1075](https://github.com/pingcap/tidb-binlog/pull/1075)