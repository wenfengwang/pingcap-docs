---
title: TiDB 5.1のリリースノート
---

# TiDB 5.1のリリースノート

リリース日: 2021年6月24日

TiDBのバージョン: 5.1.0

v5.1では、主な新機能や改善点は以下の通りです:

- MySQL 8.0の共通テーブル式（CTE）機能をサポートし、SQLステートメントの可読性と実行効率を向上させます。
- コード開発の柔軟性を高めるために、オンラインでのカラム型の変更をサポートします。
- 安定性を向上させるために新しい統計タイプを導入し、デフォルトで実験的な機能として有効になります。
- MySQL 8.0の動的権限機能をサポートし、特定の操作に対するより細かい制御を実装します。
- リードの遅延時間を短縮し、クエリのパフォーマンスを改善するために、Stale Read機能を使ってローカルレプリカから直接データを読み取ることをサポートします（実験的）。
- データベース管理者（DBA）がトランザクションのロックイベントを観察し、デッドロックの問題をトラブleshootするために、Lock View機能を追加します（実験的）。
- バックグラウンドタスクのためにTiKV書き込みレート制限機能を追加して、リードおよびライトリクエストの遅延が安定するようにします。

## 互換性の変更

> **注意:**
>
> 以前のTiDBバージョンからv5.1にアップグレードする際に、中間バージョンの互換性変更ノートをすべて知りたい場合は、対応するバージョンの[リリースノート](/releases/release-notes.md)を確認できます。

### システム変数

| 変数名   | 変更タイプ   | 説明   |
|:----------|:-----------|:-----------|
| [`cte_max_recursion_depth`](/system-variables.md#cte_max_recursion_depth)  | 新たに追加 | 共通テーブル式における最大再帰深さを制御します。 |
| [`init_connect`](/system-variables.md#init_connect)  | 新たに追加 | TiDBサーバーへの初期接続を制御します。 |
| [`tidb_analyze_version`](/system-variables.md#tidb_analyze_version-new-in-v510)  | 新たに追加 | TiDBが統計情報を収集する方法を制御します。この変数のデフォルト値は`2`です。これは実験的な機能です。 |
| [`tidb_enable_enhanced_security`](/system-variables.md#tidb_enable_enhanced_security) | 新たに追加 | 接続しているTiDBサーバーがセキュリティ強化モード（SEM）を有効にしているかを示します。この変数の設定はTiDBサーバーを再起動しないと変更できません。 |
| [`tidb_enforce_mpp`](/system-variables.md#tidb_enforce_mpp-new-in-v51) | 新たに追加 | 最適化ツールのコスト推定を無視し、クエリの実行に強制的にMPPモードを使用するかどうかを制御します。この変数のデータ型は`BOOL`で、デフォルト値は`false`です。 |
| [`tidb_partition_prune_mode`](/system-variables.md#tidb_partition_prune_mode-new-in-v51) | 新たに追加 | パーティションテーブルの動的プルーニングモードを有効にするかどうかを指定します。この機能は実験的です。この変数のデフォルト値は`static`であり、これはパーティションテーブルの動的プルーニングモードがデフォルトで無効であることを意味します。  |

### 設定ファイルパラメータ

| 設定ファイル | 設定項目 | 変更タイプ   | 説明   |
|:----------|:-----------|:-----------|:-----------|
| TiDB構成ファイル | [`security.enable-sem`](/tidb-configuration-file.md#enable-sem)  | 新たに追加  | セキュリティ強化モード（SEM）を有効にするかどうかを制御します。この設定項目のデフォルト値は`false`で、SEMが無効であることを意味します。 |
| TiDB構成ファイル | `performance.committer-concurrency`  | 変更された  | 単一トランザクションのコミットフェーズにおけるコミット操作に関連するリクエストの並列数を制御します。デフォルト値は`16`から`128`に変更されています。 |
| TiDB構成ファイル | [`performance.tcp-no-delay`](/tidb-configuration-file.md#tcp-no-delay)  | 新たに追加  | TCP層でTCP_NODELAYを有効にするかどうかを決定します。デフォルト値は`true`で、TCP_NODELAYが有効であることを意味します。 |
| TiDB構成ファイル | [`performance.enforce-mpp`](/tidb-configuration-file.md#enforce-mpp)  | 新たに追加  | インスタンスレベルでのOptimizerのコスト推定をTiDBが無視し、MPPモードを強制するかどうかを制御します。デフォルト値は`false`です。この設定項目はシステム変数[`tidb_enforce_mpp`](/system-variables.md#tidb_enforce_mpp-new-in-v51)の初期値を制御します。 |
| TiDB構成ファイル | [`pessimistic-txn.deadlock-history-capacity`](/tidb-configuration-file.md#deadlock-history-capacity)  | 新たに追加  | 単一のTiDBサーバーの[`INFORMATION_SCHEMA.DEADLOCKS`](/information-schema/information-schema-deadlocks.md)テーブルに記録できるデッドロックイベントの最大数を設定します。デフォルト値は`10`です。 |
| TiKV構成ファイル | [`abort-on-panic`](/tikv-configuration-file.md#abort-on-panic)  | 新たに追加  | TiKVがパニックした際に`abort`プロセスがコアダンプファイルを生成することを許可するかどうかを設定します。デフォルト値は`false`で、コアダンプファイルを生成できないことを意味します。 |
| TiKV構成ファイル | [`hibernate-regions`](/tikv-configuration-file.md#hibernate-regions)  | 変更された  | デフォルト値が`false`から`true`に変更されています。Regionが長時間アイドル状態になると、自動的に休止状態にするよう設定されます。 |
| TiKV構成ファイル | [`old-value-cache-memory-quota`](/tikv-configuration-file.md#old-value-cache-memory-quota)  | 新たに追加  | TiCDCの古い値によるメモリ使用量の上限を設定します。デフォルト値は`512MB`です。 |
| TiKV構成ファイル | [`sink-memory-quota`](/tikv-configuration-file.md#sink-memory-quota)  | 新たに追加  | TiCDCデータ変更イベントによるメモリ使用量の上限を設定します。デフォルト値は`512MB`です。 |
| TiKV構成ファイル | [`incremental-scan-threads`](/tikv-configuration-file.md#incremental-scan-threads)  | 新たに追加  | 履歴データをインクリメンタルにスキャンするタスクのスレッド数を設定します。デフォルト値は`4`です。 |
| TiKV構成ファイル | [`incremental-scan-concurrency`](/tikv-configuration-file.md#incremental-scan-concurrency)  | 新たに追加  | 履歴データをインクリメンタルにスキャンするタスクの最大並行実行数を設定します。デフォルト値は`6`です。 |
| TiKV構成ファイル | [`soft-pending-compaction-bytes-limit`](/tikv-configuration-file.md#soft-pending-compaction-bytes-limit)  | 変更された  | 保留中の圧縮バイトのソフトリミットを設定します。デフォルト値は`"64GB"`から`"192GB"`に変更されています。 |
| TiKV構成ファイル | [`storage.io-rate-limit`](/tikv-configuration-file.md#storageio-rate-limit)  | 新たに追加  | TiKV書き込みのI/Oレートを制御します。`storage.io-rate-limit.max-bytes-per-sec`のデフォルト値は`"0MB"`です。 |
| TiKV構成ファイル | [`resolved-ts.enable`](/tikv-configuration-file.md#enable)  | 新たに追加  | すべてのRegionリーダーに対して`resolved-ts`を維持するかどうかを決定します。デフォルト値は`true`です。 |
| TiKV構成ファイル | [`resolved-ts.advance-ts-interval`](/tikv-configuration-file.md#advance-ts-interval)  | 新たに追加  | `resolved-ts`の前進間隔を設定します。デフォルト値は`"1s"`です。値を動的に変更できます。 |
| TiKV構成ファイル | [`resolved-ts.scan-lock-pool-size`](/tikv-configuration-file.md#scan-lock-pool-size)  | 新たに追加  | TiKVが`resolved-ts`を初期化する際にMVCC（Multi-Version Concurrency Control）ロックデータをスキャンするために使用するスレッド数を設定します。デフォルト値は`2`です。 |

### その他

- アップグレード前に、TiDB構成の[`feedback-probability`](https://docs.pingcap.com/tidb/v5.1/tidb-configuration-file#feedback-probability)の値を確認してください。値が0でない場合、アップグレード後に"recoverable goroutineでのpanic"エラーが発生しますが、このエラーはアップグレードに影響しません。
- TiDBのGoコンパイラバージョンをgo1.13.7からgo1.16.4にアップグレードし、TiDBのパフォーマンスが向上します。TiDB開発者の場合は、スムーズなコンパイルを保証するためにGoコンパイラバージョンをアップグレードしてください。
- TiDB Binlogを使用するクラスターでクラスターインデックスを持つテーブルを作成しないでください。TiDBのローリングアップグレード中に`alter table ... modify column`や`alter table ... change column`などのステートメントを実行しないでください。
- v5.1以降、システムテーブルのレプリカを設定することはサポートされなくなりました。各テーブルに対してTiFlashレプリカを構築する際に、関連するシステムテーブルのレプリカをクリアする必要があります。それ以外の場合、アップグレードが失敗します。
- TiCDCの`cdc cli changefeed`コマンドでの`--sort-dir`パラメータが廃止されました。代わりに`cdc server`コマンドで`--sort-dir`を設定できます。[#1795](https://github.com/pingcap/tiflow/pull/1795)
- TiDB 5.1 へのアップグレード後、「関数 READ ONLY には noop 実装のみが存在する」というエラーがTiDBで返される場合は、[`tidb_enable_noop_functions`](/system-variables.md#tidb_enable_noop_functions-new-in-v40)の値を `ON` に設定することで、このエラーを無視させることができます。このためには MySQL の `read_only` 変数が TiDB でまだ効果を持たないため（TiDB では 'noop' 動作となる）、TiDB にこの変数が設定されていても、TiDBクラスターにデータを書き込むことができます。

## 新機能

### SQL

- MySQL 8.0 の Common Table Expression (CTE) 機能をサポート

    この機能により、TiDBは階層データを再帰的または非再帰的にクエリし、人事、製造業、金融市場、教育などの複数の分野でのアプリケーションロジックの実装におけるツリークエリのニーズを満たします。

    TiDBでは、`WITH` ステートメントを使用して Common Table Expressions を適用できます。[ユーザードキュメント](/sql-statements/sql-statement-with.md), [#17472](https://github.com/pingcap/tidb/issues/17472)

- MySQL 8.0 のダイナミック権限機能をサポート

    ダイナミック権限は `SUPER` 権限を制限し、より柔軟な権限構成を提供し、より細かい粒度のアクセス制御をTiDBに提供します。 たとえば、ダイナミック権限を使用して、`BACKUP` および `RESTORE` 操作のみを実行できるユーザーアカウントを作成できます。

    サポートされるダイナミック権限は以下の通りです。

    - `BACKUP_ADMIN`
    - `RESTORE_ADMIN`
    - `ROLE_ADMIN`
    - `CONNECTION_ADMIN`
    - `SYSTEM_VARIABLES_ADMIN`

    新規権限を追加するためにプラグインを使用することもできます。すべてのサポートされる権限を確認するには、`SHOW PRIVILEGES` ステートメントを実行してください。[ユーザードキュメント](/privilege-management.md)

- セキュリティ強化モード（SEM）用の新しい構成項目を追加

    セキュリティ強化モードはデフォルトで無効になっています。有効にするには、[ユーザードキュメント](/system-variables.md#tidb_enable_enhanced_security)を参照してください。

- オンラインでのカラム型の変更能力を強化。`ALTER TABLE` ステートメントを使用して、以下を含むカラム型のオンライン変更をサポート:

    - `VARCHAR` から `BIGINT` への変更
    - `DECIMAL` 精度の変更
    - `VARCHAR(10)` の長さを `VARCHAR(5)` に圧縮

    [ユーザードキュメント](/sql-statements/sql-statement-modify-column.md)

- 新しいSQL構文 `AS OF TIMESTAMP` を導入し、指定された時点または指定された時刻範囲から歴史データを読み取るための新しい実験的な機能であるStale Readを実行します。

    [ユーザードキュメント](/stale-read.md), [#21094](https://github.com/pingcap/tidb/issues/21094)

    `AS OF TIMESTAMP` の例は以下のとおりです:

    ```sql
    SELECT * FROM t AS OF TIMESTAMP  '2020-09-06 00:00:00';
    START TRANSACTION READ ONLY AS OF TIMESTAMP '2020-09-06 00:00:00';
    SET TRANSACTION READ ONLY as of timestamp '2020-09-06 00:00:00';
    ```

- 新しい統計の種類 `tidb_analyze_version = 2` を導入（実験的）

    `tidb_analyze_version` はデフォルトで `2` に設定されており、Version 1 におけるハッシュの衝突による大きな誤差を回避し、ほとんどのシナリオで推定の正確性を維持します。

    [ユーザードキュメント](/statistics.md)

### トランザクション

+ ロックビュー機能をサポート（実験的）

    ロックビュー機能は、悲観的ロックのロック競合およびロック待ちに関する情報を提供し、トランザクションのロック状態を観察し、デッドロック問題をトラブルシューティングするのに役立ちます。[#24199](https://github.com/pingcap/tidb/issues/24199)

    ユーザードキュメント:

    - クラスター全体のすべてのTiKVノードで発生する悲観的ロックおよびその他のロックを表示: [`DATA_LOCK_WAITS`](/information-schema/information-schema-data-lock-waits.md)
    - 最近発生したTiDBノードでのいくつかのデッドロックエラーを表示: [`DEADLOCKS`](/information-schema/information-schema-deadlocks.md)
    - TiDBノード上で現在実行中のトランザクション情報を表示: [`TIDB_TRX`](/information-schema/information-schema-tidb-trx.md)

### パフォーマンス

+ データレプリカのStale Readを実験的にサポート

    リードローカルレプリカデータを直接読み取ることで、リード待ち時間を短縮し、クエリパフォーマンスを向上させます
    
    [ユーザードキュメント](/stale-read.md), [#21094](https://github.com/pingcap/tidb/issues/21094)
    
+ ヒバネートリージョン機能をデフォルトで有効にする

    リージョンが長期間非アクティブな状態の場合、自動的にサイレント状態に設定され、リーダーとフォロワー間のハートビート情報のシステムのオーバーヘッドを削減します。
    
    [ユーザードキュメント](/tikv-configuration-file.md#hibernate-regions), [#10266](https://github.com/tikv/tikv/pull/10266)

### 安定性

+ TiCDCのレプリケーション安定性の問題を解決

    - TiCDCのメモリ使用量を改善し、以下のシナリオでOOMを回避します
    - レプリケーションの中断中に大量のデータが蓄積され、1TBを超える場合、再レプリケーションによるOOM問題が発生します。
    - 大量のデータ書き込みがTiCDCでOOM問題を発生させます。
    - 次のシナリオでTiCDCレプリケーションの中断の可能性を低減します:

        [project#11](https://github.com/pingcap/tiflow/projects/11)

        - ネットワークが不安定な場合のレプリケーション中断
        - 一部のTiKV/PD/TiCDCノードがダウンしている場合のレプリケーション中断

+ TiFlashストレージメモリ制御

    リージョンスナップショット生成の速度とメモリ使用量を最適化し、OOMの可能性を低減します

+ TiKVバックグラウンドタスク（TiKVライトレートリミッター）の書き込みレートリミッターを追加

    読み書きリクエストの持続安定性を確保するために、TiKVライトレートリミッターはGCやコンパクションなどのTiKVバックグラウンドタスクの書き込みトラフィックをスムージングします。TiKVバックグラウンドタスクの書き込みレートリミッターのデフォルト値は "0MB" です。この値はクラウドディスクメーカーによって指定されたI/O帯域幅などの最適なI/O帯域幅に設定することをお勧めします。
    
    [ユーザードキュメント](/tikv-configuration-file.md#storageio-rate-limit), [#9156](https://github.com/tikv/tikv/issues/9156)

+ 複数のスケーリングタスクが同時に実行される場合のスケジューリング安定性の問題を解決

### テレメトリ

TiDBは、テレメトリにTiDBクラスターの実行状態、実行ステータス、および失敗ステータスを追加しました。

この動作についての詳細と無効にする方法については、[テレメトリ](/telemetry.md)を参照してください。

## 改善点

+ TiDB

    - 組込み関数 `VITESS_HASH()` をサポート [#23915](https://github.com/pingcap/tidb/pull/23915)
    - 列挙型のデータをTiKVに押し下げて、`WHERE`句で列挙型を使用する際のパフォーマンスを向上させるサポート [#23619](https://github.com/pingcap/tidb/issues/23619)
    - `RENAME USER` 構文をサポート [#23648](https://github.com/pingcap/tidb/issues/23648)
    - ウィンドウ関数の計算を最適化し、ROW_NUMBER() でページングデータを行う際のTiDBのOOM問題を解決 [#23807](https://github.com/pingcap/tidb/issues/23807)
    - 大量の `SELECT` 文を `UNION ALL` で結合する際のTiDBのOOM問題を解決するための `UNION ALL` の計算を最適化 [#21441](https://github.com/pingcap/tidb/issues/21441)
    - パーティション化されたテーブルの動的プルーニングモードを最適化し、パフォーマンスと安定性を向上させる [#24150](https://github.com/pingcap/tidb/issues/24150)
    - 複数のシナリオで発生する `Region is Unavailable` の問題を修正 [project#62](https://github.com/pingcap/tidb/projects/62)
    - 頻繁に `mysql.stats_histograms` テーブルを読み取ることを避け、キャッシュされた統計情報が最新の場合、高いCPU使用率を避ける [#24317](https://github.com/pingcap/tidb/pull/24317)

+ TiKV

    - `zstd` を使用してリージョンスナップショットを圧縮し、重いスケジューリングやスケーリングの際にノード間での大きなスペースの違いを防止 [#10005](https://github.com/tikv/tikv/pull/10005)
    - 複数のケースでのOOM問題を解決 [#10183](https://github.com/tikv/tikv/issues/10183)

        - 各モジュールのメモリ使用量をトラッキングする機能を追加
        - サイズが大きすぎるRaftエントリキャッシュによるOOM問題を解決
        - スタックされたGCタスクによるOOM問題を解決
        - 一度に多数のRaftログからメモリにRaftエントリを取得することによるOOM問題を解決
- リージョンの成長が分割速度を上回る問題を抑制するためにリージョンをより均等に分割する{#9785}（https://github.com/tikv/tikv/issues/9785）

+ TiFlash

    - `Union All`、`TopN`、および`Limit`関数のサポート
    - MPPモードにおいて、左外部結合やセミアンチ結合を含むデカルト積のサポート
    - DDLステートメントの実行とリード操作が互いにブロックされないようにロック操作を最適化
    - TiFlashによる期限切れデータのクリーンアップを最適化
    - TiFlashストレージレベルでの`timestamp`カラムのクエリフィルターのさらなるフィルタリングをサポート
    - クラスタ内に大量のテーブルがある場合の、TiFlashの起動およびスケーラビリティの向上
    - 不明なCPUで実行する際のTiFlash互換性の向上

+ PD

    - `scatter region`スケジューラを追加後に予期しない統計が生じるのを防止{#3602}（https://github.com/pingcap/pd/pull/3602）
    - スケーリングプロセスにおける複数のスケジューリングの問題を解決

        - スケーリング中にスケジューリングが遅くなることを解消するためにレプリカスナップショット生成プロセスを最適化{#3563, #10059, #10001}（https://github.com/tikv/pd/issues/3563） 
        - トラフィックの変化による心拍圧の影響によるスケジューリング遅延を解消{#3693, #3739, #3728, #3751}（https://github.com/tikv/pd/issues/3693）
        - スケジューリングによる大規模クラスタの空間の不一致を減少し、大きな圧縮率の不一致によるバースト問題を防止するためにスケジューリング式を最適化{#3592, #10005}（https://github.com/tikv/pd/issues/3592）

+ Tools

    + Backup & Restore (BR)

        - `mysql`スキーマ内のシステムテーブルのバックアップとリストアをサポート{#1143, #1078}（https://github.com/pingcap/br/pull/1143）
        - バーチャルホストアドレッシングモードに基づくS3互換ストレージのサポートを実施{#10243}（https://github.com/tikv/tikv/pull/10243）
        - バックアップメタのフォーマットを最適化してメモリ使用量を削減{#1171}（https://github.com/pingcap/br/pull/1171）

    + TiCDC

        - 問題の診断に役立つより明確で有用なログメッセージの記述を改善{#1759}（https://github.com/pingcap/tiflow/pull/1759）
        - TiCDCのスキャン速度が下流処理容量を感知できるようにスキャン速度の後戻り機能をサポート{#10151}（https://github.com/tikv/tikv/pull/10151）
        - TiCDCが初期スキャンを実行する際のメモリ使用量を減少{#10133}（https://github.com/tikv/tikv/pull/10133）
        - 悲観的トランザクションにおけるTiCDC古い値のキャッシュヒット率を改善{#10089}（https://github.com/tikv/tikv/pull/10089）

    + Dumpling

        - TiDB v4.0からデータをエクスポートするロジックを改善し、TiDBがメモリ不足（OOM）になる問題を回避{#273}（https://github.com/pingcap/dumpling/pull/273）
        - バックアップ操作が失敗した際にエラーが出力されない問題を修正{#280}（https://github.com/pingcap/dumpling/pull/280）

    + TiDB Lightning

        - データインポート速度を向上させる。最適化結果によると、TPC-Cデータのインポート速度が30%向上し、大規模テーブル（2TB以上）の5つのインデックスを持つテーブルのインポート速度が50%以上向上する{#753}（https://github.com/pingcap/br/pull/753）
        - ターゲットクラスタをインポートする前にデータの前提チェックおよびターゲットクラスタに対する前提チェックを追加し、インポート要件を満たさない場合にエラーを報告してインポートプロセスを拒否する{#999}（https://github.com/pingcap/br/pull/999）
        - Local backendのチェックポイント更新のタイミングを最適化し、ブレークポイントからの再起動の性能を向上させる{#1080}（https://github.com/pingcap/br/pull/1080）

## バグ修正

+ TiDB

    - プロジェクト除外の実行結果がプロジェクション結果が空の場合に誤ってなる問題を修正{#23887}（https://github.com/pingcap/tidb/issues/23887）
    - 特定のケースにおいて、カラムに`NULL`値が含まれる場合に間違ったクエリ結果が得られる問題を修正{#23891}（https://github.com/pingcap/tidb/issues/23891）
    - スキャンに仮想カラムが含まれる場合にMPPプランの生成を実行しないようにする{#23886}（https://github.com/pingcap/tidb/issues/23886）
    - `PointGet`および`TableDual`の誤った再利用を修正{#23187, #23144, #23304, #23290}（https://github.com/pingcap/tidb/issues/23187）
    - 最適化機能によりクエリプランが`IndexMerge`プランをビルドする際のエラーを修正{#23906}（https://github.com/pingcap/tidb/issues/23906）
    - BIT型の型推論エラーを修正{#23832}（https://github.com/pingcap/tidb/issues/23832）
    - `PointGet`オペレータが存在する場合に一部の最適化ヒントが効果を発揮しない問題を修正{#23570}（https://github.com/pingcap/tidb/issues/23570）
    - エラーが発生した際にロールバックに失敗するDDL操作がある問題を修正{#23893}（https://github.com/pingcap/tidb/issues/23893）
    - バイナリリテラル定数のインデックス範囲が誤って構築される問題を修正{#23672}（https://github.com/pingcap/tidb/issues/23672）
    - 特定のケースにおいて`IN`句の間違った結果が得られる問題を修正{#23889}（https://github.com/pingcap/tidb/issues/23889）
    - 一部の文字列関数の間違った結果が得られる問題を修正{#23759}（https://github.com/pingcap/tidb/issues/23759）
    - `REPLACE`操作を実行するためにユーザーがテーブルに対して`INSERT`および`DELETE`権限の両方を必要とする{#23909}（https://github.com/pingcap/tidb/issues/23909）
    - `REPLACE`操作を実行するためにユーザーがテーブルに対して`INSERT`および`DELETE`権限の両方を必要とする{#24070}（https://github.com/pingcap/tidb/pull/24070）
    - バイナリとバイトの比較が誤って実行されることにより、誤った`TableDual`プランが生成される問題を修正{#23846}（https://github.com/pingcap/tidb/issues/23846）
    - プレフィックスインデックスとインデックス結合を使用することにより発生するパニック問題を修正{#24547, #24716, #24717}（https://github.com/pingcap/tidb/issues/24547）
    - トランザクション内で`PointGet`文によって誤って`prepared plan cache`が使用される問題を修正{#24741}（https://github.com/pingcap/tidb/issues/24741）
    - `ascii_bin`または`latin1_bin`の照合が有効な場合に誤ってプレフィックスインデックス値が書き込まれる問題を修正{#24569}（https://github.com/pingcap/tidb/issues/24569）
    - 進行中のトランザクションがGCワーカーによって中断される可能性がある問題を修正{#24591}（https://github.com/pingcap/tidb/issues/24591）
    - `new-collation`が有効になっている際に、クラスター内で`new-row-format`が無効になった場合にクラスタードインデックスでポイントクエリが誤って得られる問題を修正{#24541}（https://github.com/pingcap/tidb/issues/24541）
    - シャッフルハッシュジョインのパーティションキーの変換をリファクタリング{#24490}（https://github.com/pingcap/tidb/pull/24490）
    - `HAVING`句を含むクエリのプランをビルドする際に発生するパニック問題を修正{#24045}（https://github.com/pingcap/tidb/issues/24045）
    - カラムプルーニングの改善が`Apply`および`Join`演算子の結果が誤る問題を修正{#23887}（https://github.com/pingcap/tidb/issues/23887）
    - Asyncコミットからフォールバックされたプライマリロックが解決できない問題を修正{#24384}（https://github.com/pingcap/tidb/issues/24384）
    - 統計情報のGC問題により複製されたfm-sketchレコードを引き起こす可能性がある問題を修正{#24357}（https://github.com/pingcap/tidb/pull/24357）
- 不要在悲観的ロックが`ErrKeyExists`エラーを受け取った場合に不必要な悲観的なロールバックを回避する[#23799](https://github.com/pingcap/tidb/issues/23799)
- `ANSI_QUOTES`が含まれる場合に数値リテラルが認識されない問題を修正する[#24429](https://github.com/pingcap/tidb/issues/24429)
- `INSERT INTO テーブル PARTITION (<partitions>) ... ON DUPLICATE KEY UPDATE`などのステートメントがリストされていないパーティションからデータを読み取ることを禁止する[#24746](https://github.com/pingcap/tidb/issues/24746)
- `GROUP BY`と`UNION`の両方を含むSQLステートメントにおいて`index out of range`エラーが発生する可能性のある問題を修正する[#24281](https://github.com/pingcap/tidb/issues/24281)
- `CONCAT`関数が照合を誤って扱う問題を修正する[#24296](https://github.com/pingcap/tidb/issues/24296)
- `collation_server`グローバル変数が新しいセッションで有効にならない問題を修正する[#24156](https://github.com/pingcap/tidb/pull/24156)

+ TiKV

    - `IN`式で符号付きまたは符号なし整数型をきちんと処理できない問題を修正する[#9821](https://github.com/tikv/tikv/issues/9821)
    - バッチでSSTファイルをインポートした後に多くの空のリージョンが残る問題を修正する[#964](https://github.com/pingcap/br/issues/964)
    - 辞書ファイルが破損した後にTiKVを起動できなくなるバグを修正する[#9886](https://github.com/tikv/tikv/issues/9886)
    - 古い値を読み取ることによって発生するTiCDCのOOM問題を修正する[#9996](https://github.com/tikv/tikv/issues/9996) [#9981](https://github.com/tikv/tikv/issues/9981)
    - 照合が`latin1_bin`の場合、クラスタードプライマリキーカラムの二次インデックスに空の値が含まれる問題を修正する[#24548](https://github.com/pingcap/tidb/issues/24548)
    - パニックが発生した際にTiKVがコアダンプファイルを生成できるようにする`abort-on-panic`構成を追加するが、ユーザーは環境を正しく設定してコアダンプを有効にする必要がある[#10216](https://github.com/tikv/tikv/pull/10216)
    - TiKVがビジーでない時に`point get`クエリのパフォーマンスが低下する問題を修正する[#10046](https://github.com/tikv/tikv/issues/10046)

+ PD

    - 多くのストアが存在する場合にPDリーダー再選が遅い問題を修正する[#3697](https://github.com/tikv/pd/issues/3697)
    - 存在しないストアからエビクトリーダースケジューラを削除する際に発生するパニック問題を修正する[#3660](https://github.com/tikv/pd/issues/3660)
    - オフラインになった同僚がマージされた後、統計情報が更新されない問題を修正する[#3611](https://github.com/tikv/pd/issues/3611)

+ TiFlash

    - 時間型を整数型にキャストする際の結果が不正確な問題を修正する
    - `receiver`が10秒以内に対応するタスクを見つけられないバグを修正する
    - `cancelMPPQuery`に無効なイテレータが存在する可能性がある問題を修正する
    - `bitwise`演算子の動作がTiDBと異なるバグを修正する
    - `prefix key`を使用する際に重なる範囲によって引き起こされる警告の問題を修正する
    - 文字列型を整数型にキャストする際に不正確な結果が返される問題を修正する
    - 連続して高速に書き込むとTiFlashがメモリ不足になる可能性がある問題を修正する
    - テーブルGC中にヌルポインタの例外が発生する可能性がある問題を修正する
    - 削除されたテーブルにデータを書き込む際にTiFlashがパニックする問題を修正する
    - クラスタードプライマリキーに対する共有デルタインデックスを同時にクローンする際に不正確な結果が返される問題を修正する
    - Compaction Filter機能が有効になっている時に発生する可能性があるパニックを修正する
    - Asyncコミットからフォールバックしたロックを解決できない問題を修正する
    - `TIMETIMEZONE`型のキャスト結果が`TIMESTAMP`型を含む場合に不正確な結果が返される問題を修正する
    - Segment Split中に発生するTiFlashパニックの問題を修正する

+ ツール

    + TiDB Lightning

        - KVデータの生成時に発生するTiDB Lightningパニックの問題を修正する[#1127](https://github.com/pingcap/br/pull/1127)
        - データインポート時に合計キーサイズがraftエントリーの制限を超えるためバッチでリージョン分割が失敗するバグを修正する[#969](https://github.com/pingcap/br/issues/969)
        - CSVファイルをインポートする際に、ファイルの最後の行が改行文字(`\r\n`)を含まない場合にエラーが報告される問題を修正する[#1133](https://github.com/pingcap/br/issues/1133)
        - インポート対象のテーブルにdouble型の自動増分列が含まれる場合、auto_incrementの値が異常になるバグを修正する[#1178](https://github.com/pingcap/br/pull/1178)

    + バックアップ＆リストア（BR）
        - 一部のTiKVノードの障害によって引き起こされるバックアップ中断の問題を修正する[#980](https://github.com/pingcap/br/issues/980)

    + TiCDC

        - 統合ソーターで発生する並列処理の問題を修正し、役立たないエラーメッセージをフィルタリングする[#1678](https://github.com/pingcap/tiflow/pull/1678)
        - 冗長なディレクトリの作成がレプリケーションを中断する可能性があるバグを修正する[#1463](https://github.com/pingcap/tiflow/issues/1463)
        - MySQL 5.7 Downstreamが上流のTiDBと同じ動作を維持するために`explicit_defaults_for_timestamp`セッション変数のデフォルト値をONに設定する[#1585](https://github.com/pingcap/tiflow/issues/1585)
        - `io.EOF`の不正な処理がレプリケーションを中断する可能性があるエラーの処理を修正する[#1633](https://github.com/pingcap/tiflow/issues/1633)
        - TiCDCダッシュボードでTiKV CDCエンドポイントのCPUメトリックを修正する[#1645](https://github.com/pingcap/tiflow/pull/1645)
        - いくつかのケースでレプリケーションがブロックされるのを避けるために`defaultBufferChanSize`を増やす[#1259](https://github.com/pingcap/tiflow/issues/1259)
        - Avro出力でタイムゾーン情報が失われる問題を修正する[#1712](https://github.com/pingcap/tiflow/pull/1712)
        - 統合ソーターで一時ファイルを削除し`sort-dir`ディレクトリを共有しないようにするサポートを追加する[#1742](https://github.com/pingcap/tiflow/pull/1742)
        - 多くのステールリージョンが存在する時に発生するKVクライアントでのデッドロックバグを修正する[#1599](https://github.com/pingcap/tiflow/issues/1599)
        - `--cert-allowed-cn`フラグでのヘルプ情報が誤っている問題を修正する[#1697](https://github.com/pingcap/tiflow/pull/1697)
        - MySQLにデータをレプリケートする際にSUPER権限が必要になる`explicit_defaults_for_timestamp`の更新を元に戻す[#1750](https://github.com/pingcap/tiflow/pull/1750)
        - メモリオーバーフローのリスクを減らすためにシンクフローコントロールをサポートする[#1840](https://github.com/pingcap/tiflow/pull/1840)
        - テーブルを移動するとレプリケーションタスクが停止する可能性があるバグを修正する[#1828](https://github.com/pingcap/tiflow/pull/1828)
        - TiKV GCセーフポイントがTiCDCチェンジフィードチェックポイントの停滞によってブロックされる問題を修正する[#1759](https://github.com/pingcap/tiflow/pull/1759)