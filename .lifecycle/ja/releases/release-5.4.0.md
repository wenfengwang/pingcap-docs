---
title: TiDB 5.4 リリースノート
---

# TiDB 5.4 リリースノート

リリース日: 2022年2月15日

TiDB バージョン: 5.4.0

v5.4 では、主な新機能または改善点は以下の通りです:

- GBK文字セットのサポート
- 複数の列のインデックスでのフィルタリング結果をマージする Index Merge の使用をサポート
- セッション変数を使用して古いデータの読み取りをサポート
- 統計情報を収集する構成を永続化する機能のサポート
- TiKV のログストレージエンジンとして Raft Engine を使用する機能のサポート (実験的)
- クラスタへのバックアップの影響を最適化
- バックアップストレージとして Azure Blob ストレージの使用をサポート
- TiFlash および MPP エンジンの安定性と性能を継続的に向上
- TiDB Lightning に、既存のデータをインポートすることを許可するスイッチを追加
- Continuous Profiling 機能の最適化 (実験的)
- TiSpark はユーザー識別と認証をサポート

## 互換性の変更

> **注意:**
>
> 以前の TiDB バージョンから v5.4.0 にアップグレードする際、すべての中間バージョンの互換性変更のノートを確認する場合は、対応するバージョンの [リリースノート](/releases/release-notes.md) を参照してください。

### システム変数

<table>
<thead>
  <tr>
    <th>変数名</th>
    <th>変更タイプ</th>
    <th>説明</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_enable_column_tracking-new-in-v540"><code>tidb_enable_column_tracking</code></a></td>
    <td>新規追加</td>
    <td>TiDB が <code>PREDICATE COLUMNS</code> を収集することを許可するかを制御します。デフォルト値は <code>OFF</code> です。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_enable_paging-new-in-v540"><code>tidb_enable_paging</code></a></td>
    <td>新規追加</td>
    <td><code>IndexLookUp</code> オペレーターでコプロセッサーリクエストを送信するためのページング方式を使用するかどうかを制御します。デフォルト値は <code>OFF</code> です。<br/><code>IndexLookup</code> と <code>Limit</code> を使用し、<code>Limit</code> を <code>IndexScan</code> にプッシュダウンできない読み取りクエリでは、読み取りクエリのレイテンシが高く、TiKV の <code>unified read pool</code> の CPU 使用率が高くなる場合があります。このような場合、<code>tidb_enable_paging</code> を <code>ON</code> に設定すると、TiDB はより少ないデータを処理するため、クエリのレイテンシとリソース消費が削減されます。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_enable_top_sql-new-in-v540"><code>tidb_enable_top_sql</code></a></td>
    <td>新規追加</td>
    <td>Top SQL 機能を有効にするかどうかを制御します。デフォルト値は <code>OFF</code> です。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_persist_analyze_options-new-in-v540"><code>tidb_persist_analyze_options</code></a></td>
    <td>新規追加</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/statistics#persist-analyze-configurations">ANALYZE 構成の永続化</a>機能を有効にするかどうかを制御します。デフォルト値は <code>ON</code> です。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_read_staleness-new-in-v540"><code>tidb_read_staleness</code></a></td>
    <td>新規追加</td>
    <td>現在のセッションで読み取れる歴史データの範囲を制御します。デフォルト値は <code>0</code> です。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_regard_null_as_point-new-in-v540"><code>tidb_regard_null_as_point</code></a></td>
    <td>新規追加</td>
    <td>オプティマイザーが、null の等価条件をインデックスアクセスの接頭辞条件として使用できるかどうかを制御します。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_stats_load_sync_wait-new-in-v540"><code>tidb_stats_load_sync_wait</code></a></td>
    <td>新規追加</td>
    <td>統計情報の同期読み込みを有効にするかどうかを制御します。デフォルト値の <code>0</code> は、機能が無効で統計情報が非同期に読み込まれることを意味します。この機能が有効になると、この変数は SQL 最適化が同期読み込み統計情報をタイムアウトする前に待機できる最大時間を制御します。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_stats_load_pseudo_timeout-new-in-v540"><code>tidb_stats_load_pseudo_timeout</code></a></td>
    <td>新規追加</td>
    <td>同期読み込み統計情報がタイムアウトした場合、SQL が失敗するか (<code>OFF</code>) または擬似統計情報を使用するか (<code>ON</code>) を制御します。デフォルト値は <code>OFF</code> です。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_backoff_lock_fast"><code>tidb_backoff_lock_fast</code></a></td>
    <td>変更済み</td>
    <td>デフォルト値が <code>100</code> から <code>10</code> に変更されました。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_enable_index_merge-new-in-v40"><code>tidb_enable_index_merge</code></a></td>
    <td>変更済み</td>
    <td>デフォルト値が <code>OFF</code> から <code>ON</code> に変更されました。<ul><li>以前の TiDB クラスタを v4.0.0 より前のバージョンから v5.4.0 以降にアップグレードする場合、この変数はデフォルトで <code>OFF</code> です。</li><li>v4.0.0 以降の TiDB クラスタを v5.4.0 以降にアップグレードする場合は、この変数はアップグレード前と同じままです。</li><li>v5.4.0 以降の新規作成の TiDB クラスタでは、この変数はデフォルトで <code>ON</code> になります。</li></ul></td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_store_limit-new-in-v304-and-v40"><code>tidb_store_limit</code></a></td>
    <td>変更済み</td>
    <td>v5.4.0 より前では、この変数はインスタンスレベルおよびグローバルレベルで構成できました。v5.4.0 以降では、この変数はグローバル構成のみサポートしています。</td>
  </tr>
</tbody>
</table>

### 設定ファイルパラメータ

|  設定ファイル    |  設定 |  変更タイプ  | 説明    |
| :---------- | :----------- | :----------- | :----------- |
| TiDB | [`stats-load-concurrency`](/tidb-configuration-file.md#stats-load-concurrency-new-in-v540) | 新規追加 |  TiDB が同期的に統計情報を処理できる最大カラム数を制御します。デフォルト値は `5` です。  |
| TiDB | [`stats-load-queue-size`](/tidb-configuration-file.md#stats-load-queue-size-new-in-v540)   | 新規追加 |  TiDB が同期的に統計情報をキャッシュできる最大カラムリクエスト数を制御します。デフォルト値は `1000` です。  |
| TiKV | [`snap-generator-pool-size`](/tikv-configuration-file.md#snap-generator-pool-size-new-in-v540) | 新規追加 | `snap-generator` スレッドプールのサイズです。デフォルト値は `2` です。 |
| TiKV | `log.file.max-size`, `log.file.max-days`, `log.file.max-backups` | 新規追加  | 詳細については、[TiKV Configuration File - `log.file`](/tikv-configuration-file.md#logfile-new-in-v540) を参照してください。 |
| TiKV | `raft-engine` | Newly added | Includes `enable`, `dir`, `batch-compression-threshold`, `bytes-per-sync`, `target-file-size`, `purge-threshold`, `recovery-mode`, `recovery-read-block-size`, `recovery-read-block-size`, and `recovery-threads`. For details, see [TiKV Configuration File - `raft-engine`](/tikv-configuration-file.md#raft-engine).|
| TiKV | [`backup.enable-auto-tune`](/tikv-configuration-file.md#enable-auto-tune-new-in-v540) | Newly added | In v5.3.0, the default value is `false`. Since v5.4.0, the default value is changed to `true`. This parameter controls whether to limit the resources used by backup tasks to reduce the impact on the cluster when the cluster resource utilization is high. In the default configuration, the speed of backup tasks might slow down. |
| TiKV | `log-level`, `log-format`, `log-file`, `log-rotation-size` | Modified | The names of TiKV log parameters are replaced with the names that are consistent with TiDB log parameters, which are `log.level`, `log.format`, `log.file.filename`, and `log.enable-timestamp`. If you only set the old parameters, and their values are set to non-default values, the old parameters remain compatible with the new parameters. If both old and new parameters are set, the new parameters take effect. For details, see [TiKV Configuration File - log](/tikv-configuration-file.md#log-new-in-v540). |
| TiKV | `log-rotation-timespan` | Deleted | The timespan between log rotations. After this timespan passes, a log file is rotated, which means a timestamp is appended to the file name of the current log file, and a new log file is created. |
| TiKV | `allow-remove-leader` | Deleted | Determines whether to allow deleting the main switch. |
| TiKV | `raft-msg-flush-interval` | Deleted | Determines the interval at which Raft messages are sent in batches. The Raft messages are sent in batches at every interval specified by this configuration item. |
| PD | [`log.level`](/pd-configuration-file.md#level) | Modified | The default value is changed from "INFO" to "info", guaranteed to be case-insensitive. |
| TiFlash | [`profile.default.enable_elastic_threadpool`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) | Newly added | Determines whether to enable or disable the elastic thread pool function. Enabling this configuration item can significantly improve TiFlash CPU utilization in high concurrency scenarios. The default value is `false`. |
| TiFlash | [`storage.format_version`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) | Newly added | Specifies the version of DTFile. The default value is `2`, under which hashes are embedded in the data file. You can also set the value to `3`. When it is `3`, the data file contains metadata and token data checksum, and supports multiple hash algorithms. |
| TiFlash | [`logger.count`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) | Modified | The default value is changed to `10`. |
| TiFlash | [`status.metrics_port`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) | Modified | The default value is changed to `8234`. |
| TiFlash | [`raftstore.apply-pool-size`](/tiflash/tiflash-configuration.md#configure-the-tiflash-learnertoml-file) | Newly added | The allowable number of threads in the pool that flushes Raft data to storage. The default value is `4`. |
| TiFlash | [`raftstore.store-pool-size`](/tiflash/tiflash-configuration.md#configure-the-tiflash-learnertoml-file) | Newly added | The allowable number of threads that process Raft, which is the size of the Raftstore thread pool. The default value is `4`. |
| TiDBデータ移行(DM) | [`collation_compatible`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced) | Newly added | The mode to sync the default collation in `CREATE` SQL statements. The value options are "loose" (by default) and "strict".  |
| TiCDC | `max-message-bytes` | Modified| Change the default value of `max-message-bytes` in Kafka sink to `104857601` (10MB)  |
| TiCDC | `partition-num`     | Modified | Change the default value of `partition-num` in Kafka Sink from `4` to `3`. It makes TiCDC send messages to Kafaka partitions more evenly. |
| TiDB Lightning | `meta-schema-name` | Modified | Specifies the schema name for the metadata in the target TiDB. From v5.4.0, this schema is created only if you have enabled [parallel import](/tidb-lightning/tidb-lightning-distributed-import.md) (the corresponding parameter is `tikv-importer.incremental-import = true`). |
| TiDB Lightning | `task-info-schema-name` |  Newly added  | Specifies the name of the database where duplicated data is stored when TiDB Lightning detects conflicts. By default, the value is "lightning_task_info". Specify this parameter only if you have enabled the "duplicate-resolution" feature. |
| TiDB Lightning | `incremental-import` | Newly added | Determines whether to allow importing data to tables where data already exists. The default value is `false`. |

### その他

- TiDBとPDの間にインターフェースが追加されました。`information_schema.TIDB_HOT_REGIONS_HISTORY` システムテーブルを使用する場合、対応するバージョンのTiDBにPDを使用する必要があります。
- TiDBサーバー、PDサーバー、TiKVサーバーは、ログ関連のパラメータを管理するために統一された命名方法を開始しました。詳細については[TiKV構成ファイル - log](/tikv-configuration-file.md#log-new-in-v540)を参照してください。
- v5.4.0以降、Plan Cacheを経由してキャッシュされた実行計画に対してSQLバインディングを作成すると、対応するクエリのキャッシュされた計画が無効になります。新しいバインディングは、v5.4.0以前にキャッシュされた実行計画に影響しません。
- v5.4およびそれ以前のバージョンでは、[TiDBデータ移行(DM)](https://docs.pingcap.com/tidb-data-migration/v5.3/)のドキュメントはTiDBのドキュメントとは独立していました。v5.4以降、DMのドキュメントはTiDBのドキュメントに統合されており、同じバージョンが使用されます。DMのドキュメントを参照するには、[DMドキュメント](/dm/dm-overview.md)に直接アクセスしてください。
- 実験的な機能である時点リカバリ (PITR) とcdclogが削除されました。v5.4.0以降、cdclogベースのPITRおよびcdclogはサポートされなくなりました。
- システム変数を「DEFAULT」に設定する動作をよりMySQL互換性のあるものに変更 [#29680](https://github.com/pingcap/tidb/pull/29680)
- システム変数`lc_time_names`を読み取り専用に設定 [#30084](https://github.com/pingcap/tidb/pull/30084)
- `tidb_store_limit`のスコープをINSTANCEまたはGLOBALからGLOBALに変更 [#30756](https://github.com/pingcap/tidb/pull/30756)
- カラムにゼロが含まれる場合、整数型のカラムを時間型のカラムに変換することを禁止する[#25728](https://github.com/pingcap/tidb/pull/25728)
- 浮動小数点値を挿入する際に`Inf`または`NAN`値に対してエラーが報告されない問題を修正する [#30148](https://github.com/pingcap/tidb/pull/30148)
- オートIDが範囲外の場合、 `REPLACE` ステートメントが他の行を誤って変更する問題を修正 [#30301](https://github.com/pingcap/tidb/pull/30301)

## 新機能

### SQL

- **TiDBはv5.4.0以降でGBK文字セットをサポート**

    v5.4.0より前は、TiDBは`ascii`, `binary`, `latin1`, `utf8`, `utf8mb4`文字セットをサポートしていました。

    中国のユーザーをよりよくサポートするために、TiDBはv5.4.0以降でGBK文字セットをサポートします。TiDBクラスターを初めて初期化する際にTiDB構成ファイルで[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)オプションを有効にすると、TiDBのGBK文字セットは`gbk_bin`と`gbk_chinese_ci`の両方の照合順序をサポートします。

    GBK文字セットを使用する際は、互換性の制限に注意する必要があります。詳細については、[文字セットと照合順序 - GBK](/character-set-gbk.md)を参照してください。

### セキュリティ

- **TiSparkはユーザー認証と認可をサポート**

    TiSpark 2.5.0以降、TiSparkはデータベースユーザー認証およびデータベースまたはテーブルレベルでの読み取り/書き込み認可をサポートします。この機能を有効にすると、不正なバッチタスクの実行を防止し、オンラインクラスターの安定性とデータセキュリティを向上させることができます。
    この機能はデフォルトで無効になっています。有効にすると、TiSparkを通じて操作するユーザーが必要な権限を持っていない場合は、TiSparkから例外が発生します。

    [ユーザー ドキュメント](/tispark-overview.md#security)

- **TiUPはルートユーザーの初期パスワードを生成することをサポート**

    クラスターを起動する際のコマンドに `--init` パラメータが導入されました。このパラメータを使用すると、TiUPを使用して展開されたTiDBクラスターでは、データベースのルートユーザーの初期強力なパスワードをTiUPが生成します。これにより、空のパスワードを持つルートユーザーを使用するセキュリティリスクを回避し、データベースのセキュリティを確保できます。

    [ユーザー ドキュメント](/production-deployment-using-tiup.md#step-7-start-a-tidb-cluster)

### パフォーマンス

- **カラムストアエンジン TiFlashおよびコンピューティングエンジン MPPの安定性とパフォーマンスの改善を継続**

    - MPPエンジンへのより多くの関数を押し込む機能をサポート:

        - 文字列関数: `LPAD()`, `RPAD()`, `STRCMP()`
        - 日付関数: `ADDDATE(string, real)`, `DATE_ADD(string, real)`, `DATE_SUB(string, real)`, `SUBDATE(string, real)`, `QUARTER()`

    - 資源利用率を向上させるために、弾力的なスレッドプール機能を導入します（実験的）
    - TiKVからデータを複製する際のデータ変換の効率を向上させ、データ複製の全体的なパフォーマンスを50%向上させます
    - 一部の設定項目のデフォルト値をチューニングすることで、TiFlashのパフォーマンスと安定性を向上させます。HTAPハイブリッド負荷の場合、単一テーブルの簡単なクエリのパフォーマンスが最大20%向上します。

    ユーザードキュメント: [サポートされるプッシュダウン演算](/tiflash/tiflash-supported-pushdown-calculations.md), [tiflash.tomlファイルの構成](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)

- **セッション変数を介して指定された時間範囲内の履歴データを読み込む**

    TiDBはRaft合意アルゴリズムに基づくマルチレプリカ分散データベースです。高同時性および高スループットのアプリケーションシナリオに対応するため、TiDBはフォロワーレプリカを通じて読み取り性能をスケーリングアウトできます。

    異なるアプリケーションシナリオでは、TiDBは2つのフォロワー読み取りモードを提供します: 強力な一貫性読み取りと弱い一貫性履歴読み取り。強力な一貫性読み取りモードはリアルタイムデータが必要なアプリケーションシナリオに適しています。しかし、このモードでは、リーダーとフォロワー間のデータ複製遅延やスループットの低下により、読み取りリクエストの待機時間が長くなる可能性があります、特に地理的に分散した展開の場合。

    リアルタイムデータに対する厳格な要件を持たないアプリケーションシナリオの場合、履歴読み取りモードが推奨されます。このモードにより、待機時間が短くなり、スループットが向上します。TiDBは現在、次の方法で履歴データの読み取りをサポートしています: 過去の特定の時間点からデータを読み取るためのSQLステートメントの使用、または過去の特定の時間点を基にした読み取り専用トランザクションの開始。どちらの方法も、特定の時点の履歴データまたは指定された時間範囲内の履歴データの読み取りをサポートしています。詳細については、[AS OF TIMESTAMP句を使用した履歴データの読み取り](/as-of-timestamp.md)を参照してください。

    v5.4.0以降、TiDBはセッション変数を介して指定された時間範囲内の履歴データを読み取る機能を改善し、運用状況において低遅延かつ高スループットな読み取りリクエストをサポートします。次のように変数を設定できます:

    ```sql
    set @@tidb_replica_read=leader_and_follower
    set @@tidb_read_staleness="-5"
    ```

    この設定により、TiDBは最も近いリーダーまたはフォロワーノードを選択し、5秒以内の最新の履歴データを読み込むことができます。

    [ユーザー ドキュメント](/tidb-read-staleness.md)

- **Index MergeのGAリリース**

    Index MergeはTiDB v4.0でSQLの最適化のための実験的なフィーチャとして導入されました。このメソッドは、クエリが複数のデータ列のスキャンを必要とする場合、条件フィルタリングを大幅に高速化します。次のクエリを例に挙げます。 `WHERE` ステートメントでは、`OR`で接続されたフィルタリング条件がそれぞれ列_key1_と列_key2_に対してインデックスを持っている場合、Index Mergeフィーチャはそれぞれのインデックスを同時にフィルタリングし、クエリ結果をマージして統合結果を返します。

    ```sql
    SELECT * FROM table WHERE key1 <= 100 OR key2 = 200;
    ```

    v4.0以前のTiDBでは、テーブルのクエリは一度に1つのインデックスの使用をサポートしていました。複数のデータ列をクエリする場合、個々の列のインデックスを使用して正確なクエリ結果を短時間で取得するには、Index Mergeを有効にできます。Index Mergeは不要なフルテーブルスキャンを回避し、大量の複合インデックスの構築を必要としません。

    v5.4.0では、Index MergeがGAとなりました。ただし、以下の制限に注意する必要があります。

    - Index Mergeは析出標準形 (X<sub>1</sub> ⋁ X<sub>2</sub> ⋁ …X<sub>n</sub>) のみをサポートします。つまり、`WHERE`節でのフィルタリング条件が`OR`で接続されているときのみ機能します。
  
    - v5.4.0以降の新しく展開されたTiDBクラスターでは、この機能がデフォルトで有効化されています。v5.4.0以降の既存のTiDBクラスターの場合は、この機能はアップグレード前の設定を引き継ぎ、必要に応じて設定を変更できます（v4.0以前のTiDBクラスターの場合、この機能は存在せず、デフォルトで無効です）。

    [ユーザー ドキュメント](/explain-index-merge.md)

- **Raft Engineのサポート（実験的）**

    TiKVでログストレージエンジンとして[Raft Engine](https://github.com/tikv/raft-engine)を使用する機能をサポートします。RocksDBと比較して、Raft EngineはTiKVのI/O書き込みトラフィックを最大40%、CPU使用率を10%減少させ、特定の負荷下で前景のスループットを約5%向上させ、テールレイテンシーを20%削減します。さらに、Raft Engineはログの再利用効率を向上させ、極端な状況下でのログの蓄積問題を解決します。
  
    Raft Engineはまだ実験的な機能であり、デフォルトで無効になっています。なお、v5.4.0でのRaft Engineのデータ形式は以前のバージョンと互換性がありません。クラスターをアップグレードする前に、すべてのTiKVノードでRaft Engineが無効になっていることを確認する必要があります。Raft Engineはv5.4.0以降のバージョンでのみ使用することをお勧めします。

    [ユーザー ドキュメント](/tikv-configuration-file.md#raft-engine)

- **`PREDICATE COLUMNS`の統計情報の収集のサポート（実験的）**

    ほとんどの場合、SQL文を実行する際、オプティマイザは特定の列の統計情報のみを使用します（`WHERE`、`JOIN`、`ORDER BY`、`GROUP BY`ステートメントでの列など）。これらの使用される列は`PREDICATE COLUMNS`と呼ばれます。

    v5.4.0以降、[`tidb_enable_column_tracking`](/system-variables.md#tidb_enable_column_tracking-new-in-v540)システム変数の値を`ON`に設定すると、TiDBは`PREDICATE COLUMNS`を収集できるようになります。

    この設定後、TiDBは100 * [`stats-lease`](/tidb-configuration-file.md#stats-lease)ごとに`PREDICATE COLUMNS`情報を`mysql.column_stats_usage`システムテーブルに書き込みます。ビジネスのクエリパターンが安定している場合、`ANALYZE TABLE TableName PREDICATE COLUMNS`構文を使用して`PREDICATE COLUMNS`の統計情報のみを収集することで、統計情報の収集のオーバーヘッドを大幅に削減できます。
  
    [ユーザー ドキュメント](/statistics.md#collect-statistics-on-some-columns)

- **統計情報を同期的に読み込む機能のサポート（実験的）**

    v5.4.0以降、TiDBは同期的に統計情報を読み込む機能を導入しました。この機能はデフォルトで無効になっています。機能を有効にした後、TiDBはSQL文を実行する際に大規模な統計情報（ヒストグラム、TopN、Count-Min Sketchの統計情報など）をメモリに同期的に読み込むことができます。これにより、SQLの最適化のための統計情報の完全性が向上します。

    [ユーザー ドキュメント](/statistics.md#load-statistics)

### 安定性

- **ANALYZE設定の永続化をサポート**

    統計情報は、オプティマイザが実行計画を生成する際に参照する基本情報の1つです。統計情報の正確さは、生成された実行計画が合理的であるかどうかに直接影響します。統計情報の正確性を確保するためには、時には異なる表、パーティション、インデックスに対して異なる収集設定を行う必要があります。

    v5.4.0以降、TiDBは一部の`ANALYZE`設定を永続化できるようになりました。この機能により、既存の設定を将来の統計情報の収集で簡単に再利用することができます。

    `ANALYZE`設定永続化機能はデフォルトで有効になっています（システム変数`tidb_analyze_version`は`2`であり、[`tidb_persist_analyze_options`](/system-variables.md#tidb_persist_analyze_options-new-in-v540)はデフォルトで`ON`になっています）。この機能を使用すると、`ANALYZE`ステートメントを手動で実行した際に、ステートメントで指定された永続化設定を記録できます。記録された後、TiDBは次回統計情報を自動的に更新するか、またはこれらの設定を指定せずに手動で統計情報を収集する際に、記録された設定に従って統計情報を収集します。

    [ユーザー ドキュメント](/statistics.md#persist-analyze-configurations)
### 高可用性と災害対策

- **クラスターにおけるバックアップタスクの影響を軽減**

    バックアップ & リストア（BR）には、自動調整機能（デフォルトで有効）が導入されています。この機能は、クラスターのリソース使用状況を監視し、バックアップタスクによるクラスターへの影響を調整することができます。場合によっては、クラスターのハードウェアリソースを増やしてバックアップ向けに自動調整機能を有効にすると、クラスターへのバックアップタスクの影響を10%以下に抑えることができます。

   [ユーザードキュメント](/br/br-auto-tune.md)

- **バックアップのターゲットストレージとしてのAzure Blob Storageのサポート**

   バックアップ & リストア（BR）は、Azure Blob Storageをリモートバックアップストレージとしてサポートしています。TiDBをAzure Cloudで展開する場合、クラスターデータをAzure Blob Storageサービスにバックアップできます。

    [ユーザードキュメント](/br/backup-and-restore-storages.md)

### データ移行

- **TiDB Lightningには、データをインポートするかどうかを判断する新機能が導入されています**

    TiDB Lightningには、`incremental-import`という構成項目が導入されています。これは、データを持つテーブルにインポートを許可するかどうかを決定します。デフォルト値は `false` です。並列インポートモードを使用する場合は、この構成を `true` に設定する必要があります。

    [ユーザードキュメント](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)

- **TiDB Lightningには、並列インポートのためのメタ情報を保存するスキーマ名が導入されています**

    TiDB Lightningには `meta-schema-name` 構成項目が導入されています。並列インポートモードでは、このパラメータは、ターゲットクラスターの各TiDB Lightningインスタンスのためのメタ情報を保存するスキーマ名を指定します。デフォルト値は "lightning_metadata" です。このパラメータに設定する値は、同じ並列インポートに参加する各TiDB Lightningインスタンスで同じである必要があります。そうでない場合、インポートされたデータの正確性を保証できません。

    [ユーザードキュメント](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)

- **TiDB Lightningには重複解決が導入されています**

    Local-backendモードでは、TiDB Lightningは、データインポートが完了する前に重複データを出力し、その重複データをデータベースから削除します。インポートが完了した後に重複データを解決し、アプリケーションのルールに従って挿入する適切なデータを選択できます。後続の増分データ移行フェーズで発生する重複データによるデータ不整合を避けるために、重複データに基づいて上流データソースをきれいにすることをお勧めします。

    [ユーザードキュメント](/tidb-lightning/tidb-lightning-error-resolution.md)

- **TiDBデータ移行（DM）におけるリレーログの使用を最適化**

    - `source` 構成で `enable-relay` スイッチを復元します。
    - `start-relay` および `stop-relay` コマンドを使用してリレーログの動的な有効化と無効化をサポートします。
    - リレーログの状態を `source` にバインドします。`source` は、どのDM-workerに移行されても有効または無効の元の状態を保持します。
    - リレーログのストレージパスをDM-worker構成ファイルに移動します。

    [ユーザードキュメント](/dm/relay-log.md)

- **DMの中で[照合](/character-set-and-collation.md)の処理を最適化**

    `collation_compatible` 構成項目を追加します。値のオプションは、`loose`（デフォルト）と `strict` です:

    - アプリケーションが照合に厳密な要件を持たず、クエリ結果の照合が上流と下流で異なる場合は、デフォルトの `loose` モードを使用してエラーを回避できます。
    - アプリケーションが照合に厳密な要件を持ち、照合が上流と下流で一貫している必要がある場合は、 `strict` モードを使用できます。ただし、下流が上流のデフォルト照合をサポートしていない場合、データレプリケーションでエラーが発生する可能性があります。

   [ユーザードキュメント](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)

- **DMの `transfer source` を最適化してレプリケーションタスクをスムーズに実行するサポートを追加**

    DM-workerノードの負荷が不均衡の場合、`transfer source` コマンドを使用して `source` の構成を手動で別の負荷に移行できます。最適化後、`transfer source` コマンドは手動操作が簡素化されます。DMは他の操作を内部で完了するため、関連するすべてのタスクを一時停止する代わりに、ソースをスムーズに転送できます。

- **DM OpenAPIが一般提供（GA）となりました**

    DMはAPIを介して日常管理をサポートし、データソースの追加およびタスクの管理を含みます。v5.4.0ではDM OpenAPIがGAになりました。

    [ユーザードキュメント](/dm/dm-open-api.md)

### 診断効率

- **Top SQL（実験的な機能）**

    デフォルトでは無効化されている新しい実験的な機能、Top SQLが導入され、パフォーマンスを消費するクエリを簡単に見つけるのに役立ちます。

    [ユーザードキュメント](/dashboard/top-sql.md)

### TiDBデータ共有サブスクリプション

- **TiCDCがクラスターへの影響を最適化**

    TiCDCを使用する場合、TiDBクラスターのパフォーマンスに与える影響を大幅に減らします。テスト環境では、TiCDCのTiDBへのパフォーマンス影響を5%未満に低減することができます。

### デプロイとメンテナンス

- **継続的プロファイリングを強化（実験的）**

    - より多くのコンポーネントをサポート: TiDB、PD、およびTiKVに加えて、TiDB v5.4.0はTiFlashのCPUプロファイリングをサポートします。
    - より多様なプロファイリング表示の形式: CPUプロファイリングとゴルーチン結果をフレームチャートで表示できるようになりました。
    - より多くのデプロイ環境をサポート: 継続的プロファイリングは、TiDB Operatorを使用してデプロイされたクラスターでも利用できます。

   継続的プロファイリングはデフォルトで無効になっており、TiDBダッシュボードで有効にできます。

    継続的プロファイリングは、TiUP v1.9.0以降またはTiDB Operator v1.3.0以降でデプロイまたはアップグレードされたクラスターに適用できます。

    [ユーザードキュメント](/dashboard/continuous-profiling.md)

## 改善点

+ TiDB

    - キャッシュされたクエリプランをクリアするための `ADMIN {SESSION | INSTANCE | GLOBAL} PLAN_CACHE` 構文をサポートしました [#30370](https://github.com/pingcap/tidb/pull/30370)

+ TiKV

    - Coprocessorは、ストリームのようにリクエストを処理するためのページングAPIをサポートしました [#11448](https://github.com/tikv/tikv/issues/11448)
    - 読み取り操作が二次ロックの解決を待たなくてもよいように `read-through-lock` をサポートしました [#11402](https://github.com/tikv/tikv/issues/11402)
    - ディスク容量不足によるパニックを避けるためのディスク保護メカニズムを追加しました [#10537](https://github.com/tikv/tikv/issues/10537)
    - アーカイブおよびログのローテーションをサポートしました [#11651](https://github.com/tikv/tikv/issues/11651)
    - Raftクライアントによるシステムコールを減らし、CPU効率を向上させました [#11309](https://github.com/tikv/tikv/issues/11309)
    - CoprocessorがTiKVに対してサブストリングをプッシュダウンすることをサポートしました [#11495](https://github.com/tikv/tikv/issues/11495)
    - Read Committed分離レベルでのスキャンパフォーマンスを向上させるために、ロックの読み取りをスキップすることができるようになりました [#11485](https://github.com/tikv/tikv/issues/11485)
    - バックアップ操作で使用されるスレッドプールのデフォルトサイズを削減し、ストレスが高い場合のスレッドプールの使用を制限するようにしました [#11000](https://github.com/tikv/tikv/issues/11000)
    - `Apply` スレッドプールと `Store` スレッドプールのサイズを動的に調整できるようになりました [#11159](https://github.com/tikv/tikv/issues/11159)
    - `snap-generator` スレッドプールのサイズを構成することができるようになりました [#11247](https://github.com/tikv/tikv/issues/11247)
    - 多くのファイルで読み取りと書き込みが頻繁に行われる場合に発生するグローバルロック競合の問題を最適化しました [#250](https://github.com/tikv/rocksdb/pull/250)

+ PD

    - デフォルトで歴史的なホットスポット情報を記録するようにしました [#25281](https://github.com/pingcap/tidb/issues/25281)
    - リクエストソースを識別するためのHTTPコンポーネントに署名を追加しました [#4490](https://github.com/tikv/pd/issues/4490)
    - TiDBダッシュボードをv2021.12.31に更新しました [#4257](https://github.com/tikv/pd/issues/4257)

+ TiFlash

    - ローカルオペレータの通信を最適化しました
    - 非一時的スレッドの数を増やして、スレッドの頻繁な作成や破棄を防ぐようにしました

+ Tools

    + バックアップ & リストア（BR）

        - 暗号化バックアップの際にキーの有効性を確認する機能を追加しました [#29794](https://github.com/pingcap/tidb/issues/29794)

    + TiCDC

        - "EventFeed retry rate limited" ログの回数を削減しました [#4006](https://github.com/pingcap/tiflow/issues/4006)
        - 多くのテーブルをレプリケートする際のレプリケーション遅延を削減しました [#3900](https://github.com/pingcap/tiflow/issues/3900)
        - TiKVストアがダウンしている場合に、KVクライアントが回復するまでの時間を短縮しました [#3191](https://github.com/pingcap/tiflow/issues/3191)

    + TiDBデータ移行（DM）
      - リレーが有効になっている場合のCPU使用率を低減する[#2214](https://github.com/pingcap/dm/issues/2214)

    + TiDB ライトニング

        - TiDBバックエンドモードでデータを書き込むために、デフォルトで楽観的なトランザクションを使用してパフォーマンスを向上させる[#30953](https://github.com/pingcap/tidb/pull/30953)

    + ダンプリング

        - ダンプリングがデータベースバージョンをチェックする際の互換性を向上させる[#29500](https://github.com/pingcap/tidb/pull/29500)
        - `CREATE DATABASE`と`CREATE TABLE`をダンプする際にデフォルトの照合順序を追加する[#3420](https://github.com/pingcap/tiflow/issues/3420)

## バグ修正

+ TiDB

    - クラスタをv4.xからv5.xにアップグレードする際に`tidb_analyze_version`の値が変更される問題を修正する[#25422](https://github.com/pingcap/tidb/issues/25422)
    - サブクエリで異なる照合順序を使用した場合に発生する間違った結果の問題を修正する[#30748](https://github.com/pingcap/tidb/issues/30748)
    - TiDBにおいて`concat(ifnull(time(3))`の結果がMySQLと異なる問題を修正する[#29498](https://github.com/pingcap/tidb/issues/29498)
    - 楽観的トランザクションモードでの潜在的なデータインデックスの不一致の問題を修正する[#30410](https://github.com/pingcap/tidb/issues/30410)
    - TiKVに式をプッシュダウンできない場合のIndexMergeのクエリ実行プランが間違っている問題を修正する[#30200](https://github.com/pingcap/tidb/issues/30200)
    - 並列カラム型変更がスキーマとデータの不一致を引き起こす問題を修正する[#31048](https://github.com/pingcap/tidb/issues/31048)
    - サブクエリが含まれる場合のIndexMergeクエリ結果が間違っている問題を修正する[#30913](https://github.com/pingcap/tidb/issues/30913)
    - クライアントでFetchSizeが非常に大きく設定されていると発生するパニックの問題を修正する[#30896](https://github.com/pingcap/tidb/issues/30896)
    - LEFT JOINが誤ってINNER JOINに変換される問題を修正する[#20510](https://github.com/pingcap/tidb/issues/20510)
    - `CASE-WHEN`式と照合が同時に使用された場合に発生するパニックの問題を修正する[#30245](https://github.com/pingcap/tidb/issues/30245)
    - `IN`の値にバイナリ定数が含まれる場合に発生する間違ったクエリ結果の問題を修正する[#31261](https://github.com/pingcap/tidb/issues/31261)
    - CTEにサブクエリが含まれる場合に発生する誤ったクエリ結果の問題を修正する[#31255](https://github.com/pingcap/tidb/issues/31255)
    - `INSERT ... SELECT ... ON DUPLICATE KEY UPDATE`ステートメントの実行時に発生するパニックの問題を修正する[#28078](https://github.com/pingcap/tidb/issues/28078)
    - INDEX HASH JOINが`send on closed channel`エラーを返す問題を修正する[#31129](https://github.com/pingcap/tidb/issues/31129)

+ TiKV

    - MVCC削除レコードがGCによってクリアされない問題を修正する[#11217](https://github.com/tikv/tikv/issues/11217)
    - 悲観的トランザクションモードでのPrewriteリクエストの再試行が稀にデータの不一致のリスクを引き起こす問題を修正する[#11187](https://github.com/tikv/tikv/issues/11187)
    - GCスキャンがメモリオーバーフローを引き起こす問題を修正する[#11410](https://github.com/tikv/tikv/issues/11410)
    - ディスク容量がいっぱいの状態でRocksDBのフラッシュまたはコンパクションがパニックを引き起こす問題を修正する[#11224](https://github.com/tikv/tikv/issues/11224)

+ PD

    - `flow-round-by-digit`によるリージョン統計の影響を受けない問題を修正する[#4295](https://github.com/tikv/pd/issues/4295)
    - ターゲットストアがダウンしているためにスケジューリングオペレータが高速に失敗しない問題を修正する[#3353](https://github.com/tikv/pd/issues/3353)
    - オフラインストア上のリージョンがマージされない問題を修正する[#4119](https://github.com/tikv/pd/issues/4119)
    - ホットスポット統計からコールドスポットデータを削除できない問題を修正する[#4390](https://github.com/tikv/pd/issues/4390)

+ TiFlash

    - MPPクエリが停止した場合にTiFlashがパニックする問題を修正する
    - `where <string>`句を含むクエリが誤った結果を返す問題を修正する
    - 整数型のプライマリキーのカラムタイプをより大きな範囲に設定した場合に発生するデータ不一致の潜在的な問題を修正する
    - 入力時刻が1970-01-01 00:00:01 UTCよりも前の場合に、`unix_timestamp`の動作がTiDBまたはMySQLと一貫性がない問題を修正する
    - TiFlashが再起動した後に`EstablishMPPConnection`エラーを返す問題を修正する
    - `CastStringAsDecimal`の動作がTiFlashとTiDB/TiKVで一貫していない問題を修正する
    - クエリ結果で`DB::Exception: Encode type of coprocessor response is not CHBlock`エラーが返る問題を修正する
    - `castStringAsReal`の動作がTiFlashとTiDB/TiKVで一貫していない問題を修正する
    - TiFlashでの`date_add_string_xxx`関数の返される結果がMySQLと一貫していない問題を修正する

+ ツール

    + Backup & Restore（BR）

        - リストア操作終了後にリージョン分布が均等でない可能性がある問題を修正する[#30425](https://github.com/pingcap/tidb/issues/30425)
        - バックアップストレージとして`minio`を使用する際にエンドポイントで`'/'`を指定できない問題を修正する[#30104](https://github.com/pingcap/tidb/issues/30104)
        - システムテーブルが同時にバックアップされるとテーブル名の更新が失敗するためシステムテーブルをリストアできない問題を修正する[#29710](https://github.com/pingcap/tidb/issues/29710)

    + TiCDC

        - `min.insync.replicas`が`replication-factor`よりも小さい場合にレプリケーションが実行されない問題を修正する[#3994](https://github.com/pingcap/tiflow/issues/3994)
        - `cached region`監視メトリクスが負の値を持つ問題を修正する[#4300](https://github.com/pingcap/tiflow/issues/4300)
        - `mq sink write row`に監視データがない問題を修正する[#3431](https://github.com/pingcap/tiflow/issues/3431)
        - `sql mode`の互換性の問題を修正する[#3810](https://github.com/pingcap/tiflow/issues/3810)
        - レプリケーションタスクが削除された際に発生する潜在的なパニックの問題を修正する[#3128](https://github.com/pingcap/tiflow/issues/3128)
        - デフォルトカラム値を出力する際に発生するパニックおよびデータの不一致の問題を修正する[#3929](https://github.com/pingcap/tiflow/issues/3929)
        - デフォルトの値がレプリケーションされない問題を修正する[#3793](https://github.com/pingcap/tiflow/issues/3793)
        - デッドロックがレプリケーションタスクのスタックを停止させる潜在的な問題を修正する[#4055](https://github.com/pingcap/tiflow/issues/4055)
        - ディスクが完全に書き込まれたときにログが出力されない問題を修正する[#3362](https://github.com/pingcap/tiflow/issues/3362)
        - DDLステートメントの特殊コメントがレプリケーションタスクを停止させる問題を修正する[#3755](https://github.com/pingcap/tiflow/issues/3755)
        - RHELリリースでタイムゾーンの問題が原因でサービスを開始できない問題を修正する[#3584](https://github.com/pingcap/tiflow/issues/3584)
        - 不正確なチェックポイントによって発生する潜在的なデータロスの問題を修正する[#3545](https://github.com/pingcap/tiflow/issues/3545)
        - コンテナ環境でのOOMの問題を修正する[#1798](https://github.com/pingcap/tiflow/issues/1798)
        - `config.Metadata.Timeout`の不正確な構成によってレプリケーションタスクが停止する問題を修正する[#3352](https://github.com/pingcap/tiflow/issues/3352)

    + TiDBデータ移行（DM）

        - `CREATE VIEW`ステートメントがデータレプリケーションを中断する問題を修正する[#4173](https://github.com/pingcap/tiflow/issues/4173)
        - DDLステートメントがスキップされた後にスキーマをリセットする必要がある問題を修正する[#4177](https://github.com/pingcap/tiflow/issues/4177)
        - DDLステートメントがスキップされた後にテーブルチェックポイントがタイムリーに更新されない問題を修正する[#4184](https://github.com/pingcap/tiflow/issues/4184)
- TiDBバージョンの互換性の問題をパーサーバージョンと解消する [#4298](https://github.com/pingcap/tiflow/issues/4298)
        - ステータスをクエリする際にのみ、syncerメトリクスが更新される問題を修正する [#4281](https://github.com/pingcap/tiflow/issues/4281)

    + TiDB Lightning

        - TiDB Lightningが`mysql.tidb`テーブルにアクセスする権限がない場合に発生する間違ったimport結果の問題を修正する [#31088](https://github.com/pingcap/tidb/issues/31088)
        - TiDB Lightningが再起動されると一部のチェックがスキップされる問題を修正する [#30772](https://github.com/pingcap/tidb/issues/30772)
        - S3パスが存在しない場合にTiDB Lightningがエラーを報告しない問題を修正する [#30674](https://github.com/pingcap/tidb/pull/30674)

    + TiDB Binlog

        - `CREATE PLACEMENT POLICY`ステートメントと非互換であるため、Drainerが失敗する問題を修正する [#1118](https://github.com/pingcap/tidb-binlog/issues/1118)