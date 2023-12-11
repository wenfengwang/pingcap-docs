---
title: TiDB 5.2 リリースノート
---

# TiDB 5.2 リリースノート

リリース日: 2021年8月27日

TiDB バージョン: 5.2.0

> **警告:**
>
> このバージョンにはいくつかの既知の問題があり、これらの問題は新しいバージョンで修正されています。最新の 5.2.x バージョンをご使用いただくことをお勧めします。

v5.2 での主な新機能と改善点は以下の通りです:

- 式インデックスで複数の関数を使用することをサポートし、クエリパフォーマンスを大幅に向上させる
- オプティマイザの基数推定の精度を向上させ、最適な実行計画の選択を支援
- ロックビュー機能の一般提供（GA）を発表し、トランザクションのロックイベントを観察しデッドロック問題のトラブルシューティングを支援
- TiFlash I/O トラフィック制限機能を追加し、TiFlash の読み書きの安定性を向上
- TiKV に新しいフローコントロールメカニズムを導入し、以前の RocksDB 書き込み停止メカニズムを置き換えて TiKV のフローコントロールの安定性を向上
- データ移行 (DM) の運用およびメンテナンスを簡素化し、管理コストを削減
- TiCDC が HTTP プロトコル OpenAPI をサポートし、TiCDC タスクを管理する。Kubernetes および独自ホスト環境の両方に対して、よりユーザーフレンドリーな操作方法を提供（実験的な機能）

## 互換性の変更

> **注:**
>
> 以前の TiDB バージョンから v5.2 にアップグレードする際、すべての中間バージョンの互換性変更ノートを知りたい場合は、対応するバージョンの[リリースノート](/releases/release-notes.md)を確認できます。

### システム変数

| 変数名    |  変更タイプ    |  説明  |
| :---------- | :----------- | :----------- |
| [`default_authentication_plugin`](/system-variables.md#default_authentication_plugin) | 新規追加 | サーバーが広告する認証方法を設定します。デフォルト値は `mysql_native_password` です。  |
| [`tidb_enable_auto_increment_in_generated`](/system-variables.md#tidb_enable_auto_increment_in_generated) | 新規追加 | 生成列または式インデックスを作成する際に `AUTO_INCREMENT` 列を含めるかどうかを決定します。デフォルト値は `OFF` です。  |
| [`tidb_opt_enable_correlation_adjustment`](/system-variables.md#tidb_opt_enable_correlation_adjustment) | 新規追加 | オプティマイザが列順序相関に基づいて行数を推定するかどうかを制御します。デフォルト値は `ON` です。  |
| [`tidb_opt_limit_push_down_threshold`](/system-variables.md#tidb_opt_limit_push_down_threshold) | 新規追加 | Limit または TopN 演算子を TiKV にプッシュダウンするかどうかを判定する閾値を設定します。デフォルト値は `100` です。  |
| [`tidb_stmt_summary_max_stmt_count`](/system-variables.md#tidb_stmt_summary_max_stmt_count-new-in-v40) | 変更 | ステートメントサマリーテーブルがメモリに保存するステートメントの最大数を設定します。デフォルト値は `200` から `3000` に変更されました。  |
| `tidb_enable_streaming` | 廃止 | `enable-streaming` システム変数は廃止され、これ以上使用しないことをお勧めします。  |

### 設定ファイルパラメータ

| 設定ファイル    |  設定項目    |  変更タイプ    |  説明    |
| :---------- | :----------- | :----------- | :----------- |
| TiDB 設定ファイル  | [`pessimistic-txn.deadlock-history-collect-retryable`](/tidb-configuration-file.md#deadlock-history-collect-retryable) |  新規追加  | [`INFORMATION\_SCHEMA.DEADLOCKS`](/information-schema/information-schema-deadlocks.md) テーブルが再試行可能なデッドロックエラーメッセージを収集するかどうかを制御します。  |
| TiDB 設定ファイル  | [`security.auto-tls`](/tidb-configuration-file.md#auto-tls) |  新規追加  | 起動時に TLS 証明書を自動的に生成するかどうかを決定します。デフォルト値は `false` です。  |
| TiDB 設定ファイル  | `stmt-summary.max-stmt-count` | 変更 | ステートメントサマリーテーブルに保存できる SQL カテゴリの最大数を示します。デフォルト値は `200` から `3000` に変更されました。  |
| TiDB 設定ファイル  | `experimental.allow-expression-index`  | 廃止 | TiDB 設定ファイルの `allow-expression-index` 構成は廃止されました。  |
| TiKV 設定ファイル  | [`raftstore.cmd-batch`](/tikv-configuration-file.md#cmd-batch)  |  新規追加  | リクエストのバッチ処理を有効にするかどうかを制御します。有効にすると書き込みパフォーマンスが大幅に向上します。デフォルト値は `true` です。  |
| TiKV 設定ファイル  | [`raftstore.inspect-interval`](/tikv-configuration-file.md#inspect-interval)  |  新規追加  | ある間隔で TiKV は Raftstore コンポーネントのレイテンシを検査します。この設定項目は検査の間隔を指定します。レイテンシがこの値を超えると、この検査はタイムアウトとしてマークされます。デフォルト値は `500ms` です。  |
| TiKV 設定ファイル  | [`raftstore.max-peer-down-duration`](/tikv-configuration-file.md#max-peer-down-duration)  | 変更 | ピアに許可される最長の無効期間を示します。タイムアウトしたピアは `down` とマークされ、後で PD がそれを削除しようとします。デフォルト値は `5m` から `10m` に変更されました。  |
| TiKV 設定ファイル  | [`server.raft-client-queue-size`](/tikv-configuration-file.md#raft-client-queue-size)  |  新規追加  | TiKV の Raft メッセージのキューサイズを指定します。デフォルト値は `8192` です。  |
| TiKV 設定ファイル  | [`storage.flow-control.enable`](/tikv-configuration-file.md#enable)  |  新規追加  | フローコントロールメカニズムを有効にするかどうかを決定します。デフォルト値は `true` です。  |
| TiKV 設定ファイル  | [`storage.flow-control.memtables-threshold`](/tikv-configuration-file.md#memtables-threshold)  |  新規追加  | kvDB のメンテーブル数がこの閾値に達すると、フローコントロールメカニズムが動作し始めます。デフォルト値は `5` です。  |
| TiKV 設定ファイル  | [`storage.flow-control.l0-files-threshold`](/tikv-configuration-file.md#l0-files-threshold)  |  新規追加  | kvDB の L0 ファイル数がこの閾値に達すると、フローコントロールメカニズムが動作し始めます。デフォルト値は `9` です。  |
| TiKV 設定ファイル  | [`storage.flow-control.soft-pending-compaction-bytes-limit`](/tikv-configuration-file.md#soft-pending-compaction-bytes-limit)  |  新規追加  | KvDB でのペンディングコンパクションバイト数がこの閾値に到達すると、フローコントロールメカニズムが一部の書き込みリクエストを拒否し `ServerIsBusy` エラーを報告します。デフォルト値は「192GB」です。  |
| TiKV 設定ファイル  | [`storage.flow-control.hard-pending-compaction-bytes-limit`](/tikv-configuration-file.md#hard-pending-compaction-bytes-limit)  |  新規追加  | KvDB でのペンディングコンパクションバイト数がこの閾値に到達すると、フローコントロールメカニズムがすべての書き込みリクエストを拒否し `ServerIsBusy` エラーを報告します。デフォルト値は「1024GB」です。  |

### その他

- アップグレード前に、[`tidb_evolve_plan_baselines`](/system-variables.md#tidb_evolve_plan_baselines-new-in-v40) システム変数の値が `ON` かどうかを確認してください。値が `ON` の場合、`OFF` に設定してください。そうしないとアップグレードが失敗します。
- TiDB クラスターを v4.0 から v5.2 にアップグレードした場合、[`tidb_multi_statement_mode`](/system-variables.md#tidb_multi_statement_mode-new-in-v4011) のデフォルト値が `WARN` から `OFF` に変更されます。
- アップグレード前に、TiDB 設定の [`feedback-probability`](https://docs.pingcap.com/tidb/v5.2/tidb-configuration-file#feedback-probability) の値を確認してください。値が `0` でない場合、アップグレード後に "recoverable goroutine でのパニック" エラーが発生しますが、このエラーはアップグレードに影響しません。
- TiDB は現在、MySQL 5.7 の noop 変数 `innodb_default_row_format` と互換性があります。この変数を設定しても効果はありません。[#23541](https://github.com/pingcap/tidb/issues/23541)
- TiDB 5.2 から、システムのセキュリティを向上させるため、クライアントからの接続の転送層を暗号化することが推奨されます（ただし必須ではありません）。TiDB は、自動 TLS 機能を提供し、自動的に暗号化を構成して有効にします。自動 TLS 機能を使用するには、TiDB の設定ファイルで [`security.auto-tls`](/tidb-configuration-file.md#auto-tls) を `true` に設定してから TiDB をアップグレードしてください。
- MySQL 8.0 からの移行を容易にし、セキュリティを向上させるために、`caching_sha2_password` 認証メソッドをサポートします。

## 新機能

### SQL

- **式インデックスで複数の関数を使用することをサポート**

    式インデックスは式に作成できる特殊なインデックスの一種です。式インデックスが作成されると、TiDB は式ベースのクエリをサポートし、クエリパフォーマンスが大幅に向上します。

    [ユーザドキュメント](/sql-statements/sql-statement-create-index.md), [#25150](https://github.com/pingcap/tidb/issues/25150)

- **Oracle の `translate` 関数をサポート**
`translate`関数は、文字列内の文字のすべての出現を他の文字に置き換えます。TiDBでは、この関数はOracleとは異なり、空の文字列を`NULL`として扱いません。

[ユーザードキュメント](/functions-and-operators/string-functions.md)

- **HashAggのスピルサポート**

    ディスクにHashAggをスピルサポートします。HashAgg演算子を含むSQL文でメモリ不足（OOM）が発生した場合、この演算子の同時実行数を`1`に設定してディスクスピルをトリガーすることで、メモリ負荷を軽減できます。

    [ユーザードキュメント](/configure-memory-usage.md#other-memory-control-behaviors-of-tidb-server), [#25882](https://github.com/pingcap/tidb/issues/25882)

- **オプティマイザのカーディナリティ推定の精度向上**

    - TopN/LimitのTiDBの推定精度を向上させます。たとえば、`order by col limit x`条件を含む大きなテーブルに対するページネーションクエリに対して、TiDBはより簡単に適切なインデックスを選択し、クエリの応答時間を短縮できます。
    - 範囲外の推定の精度を向上させます。たとえば、ある日の統計情報が更新されていなくても、TiDBは`where date=Now()`を含むクエリに対して正確に対応するインデックスを選択できます。
    - `tidb_opt_limit_push_down_threshold`変数を導入し、Limit/TopNのオプティマイザの動作を制御します。これにより、誤った推定によりLimit/TopNを押し下げることができない場合の問題を解消します。

    [ユーザードキュメント](/system-variables.md#tidb_opt_limit_push_down_threshold), [#26085](https://github.com/pingcap/tidb/issues/26085)

- **オプティマイザのインデックス選択の改善**

    インデックス選択のためのプルーニングルールを追加します。TiDBは統計情報を使用する前にこれらのルールを使用して、選択の可能性のあるインデックスの範囲を絞り込むため、非最適なインデックスを選択する可能性を減らします。

    [ユーザードキュメント](/choose-index.md)

### トランザクション

- **Lock Viewの一般提供（GA）**

    Lock View機能は、悲観ロックのロック競合およびロック待ちに関する情報を提供し、DBAがトランザクションロックイベントを観察し、デッドロック問題をトラブルシューティングするのに役立ちます。

    v5.2では、次の改善がLock Viewに加えられています:

    - Lock View関連テーブルにSQLダイジェスト列に加えて、これらのテーブルに対応する正規化されたSQLテキストを表示する列を追加します。SQLダイジェストに対応するステートメントを手動でクエリする必要はありません。
    - クラスタ内の一連のSQLダイジェストに対応する正規化されたSQLステートメント（形式や引数を含まない形式）をクエリする`TIDB_DECODE_SQL_DIGESTS`関数を追加します。これにより、トランザクションによって過去に実行されたステートメントを簡単にクエリできます。
    - `DATA_LOCK_WAITS`および`DEADLOCKS`システムテーブルに、キーから解釈されるテーブル名、行ID、インデックス値などのキー情報を示す列を追加します。これにより、キーが属するテーブルを特定し、キー情報を解釈するなどの操作が簡素化されます。
    - `DEADLOCKS`テーブルで再試行可能なデッドロックエラーの情報収集をサポートし、このようなエラーによって引き起こされた問題をトラブルシューティングするのがより簡単になります。エラーの収集はデフォルトでは無効になっており、`pessimistic-txn.deadlock-history-collect-retryable`構成を使用して有効にすることができます。
    - `TIDB_TRX`システムテーブルで、クエリを実行中のトランザクションとアイドルのトランザクションを区別して表示できるようにします。`Normal`ステートは`Running`および`Idle`の2つに分割されます。

    ユーザードキュメント:

    - クラスタ内のすべてのTiKVノードで発生している悲観ロック待ちイベントを表示: [`DATA_LOCK_WAITS`](/information-schema/information-schema-data-lock-waits.md)
    - TiDBノードで最近発生したデッドロックエラーを表示: [`DEADLOCKS`](/information-schema/information-schema-deadlocks.md)
    - TiDBノードで実行中のトランザクションを表示: [`TIDB_TRX`](/information-schema/information-schema-tidb-trx.md)

- **`AUTO_RANDOM`または`SHARD_ROW_ID_BITS`属性を持つテーブルにインデックスを追加するユーザーシナリオの最適化**

### 安定性

- **TiFlash I/Oトラフィック制限の追加**

    この新機能は、小さな特定サイズのディスク帯域幅を持つクラウドストレージに適しています。デフォルトでは無効になっています。

    TiFlash I/Oレート制限機能は、読み書きタスクのI/Oリソースの過剰な競合を回避する新しいメカニズムを提供します。読み書きタスクに対する応答をバランスし、読み書きワークロードに応じて自動的にレートを制限します。

    [ユーザードキュメント](/tiflash/tiflash-configuration.md)

- **TiKVフロー制御の安定性の向上**

    TiKVは以前のRocksDB書き込み停滞メカニズムの代わりに新しいフロー制御メカニズムを導入します。この新しいメカニズムは、RocksDB層ではなくTiKVスケジューラ層でコンパクションのストレスが蓄積した場合にフロー制御を実行し、次の問題を回避します:

    - RocksDB書き込み停滞によってRaftstoreが停滞する。
    - Raft選挙がタイムアウトし、ノードのリーダーが移行される。

    この新しいメカニズムは、書き込みトラフィックが高いときにQPSが低下する問題を緩和するためのフロー制御アルゴリズムを改善します。

    [ユーザードキュメント](/tikv-configuration-file.md#storageflow-control), [#10137](https://github.com/tikv/tikv/issues/10137)

- **クラスタ内の単一の遅いTiKVノードによって引き起こされた影響からの自動検出と回復**

    TiKVは遅いノード検出メカニズムを導入します。このメカニズムは、TiKV Raftstoreのレートを検査してスコアを計算し、そのスコアをPDにストアハートビートを介して報告します。同時に、PDに`evict-slow-store-scheduler`スケジューラを追加し、単一の遅いTiKVノード上のリーダーを自動的に排除します。これにより、クラスタ全体への影響が軽減されます。また、遅いノードに関するより多くの警告項目も導入され、問題を迅速に特定して解決するのに役立ちます。

    [ユーザードキュメント]( /tikv-configuration-file.md#inspect-interval), [#10539](https://github.com/tikv/tikv/issues/10539)

### データ移行

- **データ移行（DM）の操作を簡素化**

    DM v2.0.6は、VIPを使用してデータソースの変更イベント（フェイルオーバーまたはプラン変更）を自動的に特定し、新しいデータソースインスタンスに自動接続できるようになり、データ複製の遅延時間を短縮し、操作手順を簡素化します。

- TiDB Lightningは、CSVデータでカスタマイズされた行終端記号をサポートし、MySQL LOAD DATA CSVデータ形式と互換性があります。そのため、TiDB Lightningをデータフローアーキテクチャに直接使用できます。

    [#1297](https://github.com/pingcap/br/pull/1297)

### TiDBデータ共有サブスクリプション

TiCDCは、TiCDCタスクを管理するためにHTTPプロトコル（OpenAPI）を使用することをサポートし、Kubernetesおよびセルフホスト環境の両方にとってより使いやすい操作方法です。 (実験的な機能)

[#2411](https://github.com/pingcap/tiflow/issues/2411)

### デプロイと運用

Apple M1チップを搭載したMacコンピューターで`tiup playground`コマンドを実行することをサポートします。

## 機能強化

+ ツール

    + TiCDC

        - TiDB向けに設計されたバイナリMQフォーマットを追加します。これはJSONベースのオープンプロトコルよりもコンパクトです [#1621](https://github.com/pingcap/tiflow/pull/1621)
        - ファイルソータのサポートを削除します [#2114](https://github.com/pingcap/tiflow/pull/2114)
        - ログローテーション構成をサポートします [#2182](https://github.com/pingcap/tiflow/pull/2182)

    + TiDB Lightning

        - カスタマイズされた行終端記号（`\r`および`\n`以外）のサポートを追加します [#1297](https://github.com/pingcap/br/pull/1297)
        - 式インデックスおよび仮想生成列に依存するインデックスをサポートします [#1407](https://github.com/pingcap/br/pull/1407)

    + Dumpling

        - MySQL互換のデータベースのバックアップをサポートしますが、`START TRANSACTION ... WITH CONSISTENT SNAPSHOT`や`SHOW CREATE TABLE`はサポートしません [#311](https://github.com/pingcap/dumpling/pull/311)

## 改善

+ TiDB

    - 組み込み関数`json_unquote()`をTiKVにプッシュダウンすることをサポートします [#24415](https://github.com/pingcap/tidb/issues/24415)
    - デュアルテーブルから`union`ブランチを削除することをサポートします [#25614](https://github.com/pingcap/tidb/pull/25614)
    - 集約演算子のコストファクターを最適化します [#25241](https://github.com/pingcap/tidb/pull/25241)
    - MPP外部結合がテーブル行数に基づいてビルドテーブルを選択できるようにします [#25142](https://github.com/pingcap/tidb/pull/25142)
    - リージョンに基づいて異なるTiFlashノード間でMPPクエリワークロードをバランスすることをサポートします [#24724](https://github.com/pingcap/tidb/pull/24724)
    - MPPクエリの実行後に、キャッシュ内の古いリージョンを無効にするサポートを追加 [#24432](https://github.com/pingcap/tidb/pull/24432)
    - ビルトイン関数`str_to_date`のMySQL互換性を向上させ、フォーマット指定子`%b/%M/%r/%T`に対応 [#25767](https://github.com/pingcap/tidb/pull/25767)
    - 同じクエリに対して異なるバインディングを再作成した後に複数のTiDBで一貫性のないバインディングキャッシュが作成される問題を修正 [#26015](https://github.com/pingcap/tidb/pull/26015)
    - アップグレード後に既存のバインディングをキャッシュに読み込むことができない問題を修正 [#23295](https://github.com/pingcap/tidb/pull/23295)
    - `SHOW BINDINGS`の結果を (`original_sql`, `update_time`) で並べ替えるサポートを追加 [#26139](https://github.com/pingcap/tidb/pull/26139)
    - バインディングが存在する場合のクエリ最適化のロジックを改善し、クエリの最適化回数を減らす [#26141](https://github.com/pingcap/tidb/pull/26141)
    - "deleted"の状態でのバインディングのガベージコレクションの自動完了をサポート [#26206](https://github.com/pingcap/tidb/pull/26206)
    - `EXPLAIN VERBOSE`の結果にクエリ最適化に使用されるかどうかの情報を表示するサポートを追加 [#26930](https://github.com/pingcap/tidb/pull/26930)
    - 現在のTiDBインスタンスにおけるバインディングキャッシュに対応するタイムスタンプを表示するための新しいステータス変動 `last_plan_binding_update_time` を追加 [#26340](https://github.com/pingcap/tidb/pull/26340)
    - バインディング進化の開始または `admin evolve bindings` の実行時にエラーを報告し、他の機能に影響を与えるベースライン進化を禁止する機能をサポート（現在は実験的な機能のため、TiDB Self-Hostedバージョンでは無効） [#26333](https://github.com/pingcap/tidb/pull/26333)

+ PD

    - ホットリージョンのスケジューリングに対するより多くのQPS次元を追加し、スケジューリングの優先度の調整をサポート [#3869](https://github.com/tikv/pd/issues/3869)
    - TiFlashのライトホットスポットのためのホットリージョンバランススケジューリングをサポート [#3900](https://github.com/tikv/pd/pull/3900)

+ TiFlash

    - 演算子を追加: `MOD / %`, `LIKE`
    - 文字列関数を追加: `ASCII()`, `COALESCE()`, `LENGTH()`, `POSITION()`, `TRIM()`
    - 数学関数を追加: `CONV()`, `CRC32()`, `DEGREES()`, `EXP()`, `LN()`, `LOG()`, `LOG10()`, `LOG2()`, `POW()`, `RADIANS()`, `ROUND(decimal)`, `SIN()`, `MOD()`
    - 日付関数を追加: `ADDDATE(string, real)`, `DATE_ADD(string, real)`, `DATE()`
    - その他の関数を追加: `INET_NTOA()`, `INET_ATON()`, `INET6_ATON`, `INET6_NTOA()`
    - 新しい照合が有効になっている場合、MPPモードでのシャッフルされたハッシュ結合計算とシャッフルされたハッシュ集約計算をサポート
    - MPPのパフォーマンスを向上するための基本コードを最適化
    - `STRING`型を`DOUBLE`型にキャストするサポート
    - 右外部結合の非結合データを複数のスレッドを使用して最適化
    - MPPクエリにおける古いリージョンの自動的な無効化をサポート

+ ツール

    + TiCDC

        - kvクライアントの増分スキャンに並行性制限を追加 [#1899](https://github.com/pingcap/tiflow/pull/1899)
        - TiCDCは常に古い値を内部で取得できるようにする [#2271](https://github.com/pingcap/tiflow/pull/2271)
        - 回復不能なDMLエラーが発生した際にTiCDCが失敗して素早く終了できるようにする [#1928](https://github.com/pingcap/tiflow/pull/1928)
        - リージョンの初期化直後に `resolve lock` を直ちに実行できない問題を修正 [#2235](https://github.com/pingcap/tiflow/pull/2235)
        - 高並行性のもとでゴルーチンの数を減らすためにworkerpoolを最適化する [#2201](https://github.com/pingcap/tiflow/pull/2201)

    + Dumpling

        - `tidb_rowid`を介してTiDB v3.xテーブルを常に分割して、TiDBメモリを節約するサポートを追加 [#301](https://github.com/pingcap/dumpling/pull/301)
        - 安定性を向上させるためにDumplingが`information_schema`へのアクセスを減らす [#305](https://github.com/pingcap/dumpling/pull/305)

## バグ修正

+ TiDB

    - `SET`型のカラムでマージ結合を使用すると正しくない結果が返される問題を修正 [#25669](https://github.com/pingcap/tidb/issues/25669)
    - `IN`式の引数でデータの破損が発生する問題を修正 [#25591](https://github.com/pingcap/tidb/issues/25591)
    - GCのセッションがグローバル変数に影響を受けることを回避するための対策 [#24976](https://github.com/pingcap/tidb/issues/24976)
    - ウィンドウ関数クエリで `limit` を使用した際に発生するパニックの問題を解決 [#25344](https://github.com/pingcap/tidb/issues/25344)
... (The complete translation is too long for a single response.)
- 複数のスケジューラー間のスケジューリングの競合により、予定されているスケジューリングが生成されない問題を修正します。[#3807](https://github.com/tikv/pd/issues/3807) [#3778](https://github.com/tikv/pd/issues/3778)

+ TiFlash

    - スプリットの失敗により、TiFlashが再起動し続ける問題を修正します
    - TiFlashがデルタデータを削除できない可能性のある問題を修正します
    - `CAST`関数で非バイナリ文字のための誤ったパディングを追加するバグを修正します
    - 複雑な`GROUP BY`列を扱う集計クエリの結果が正しくない問題を修正します
    - 重い書き込み圧力下で発生するTiFlashのパニック問題を修正します
    - 右側の結合キーがnullableでなく、左側の結合キーがnullableである場合に発生するパニックを修正します
    - `read-index`リクエストに長い時間がかかる可能性のある問題を修正します
    - 読み込み負荷が重い場合に発生するパニック問題を修正します
    - `Date_Format`関数が`STRING`型引数と`NULL`値で呼び出された場合に発生するパニック問題を修正します

+ ツール

    + TiCDC

        - チェックポイントの更新時にTiCDCオーナーが異常終了するバグを修正します [#1902](https://github.com/pingcap/tiflow/issues/1902)
        - 成功したチェンジフィードの作成直後にチェンジフィードが失敗するバグを修正します [#2113](https://github.com/pingcap/tiflow/issues/2113)
        - ルールフィルタの無効なフォーマットによりチェンジフィードが失敗するバグを修正します [#1625](https://github.com/pingcap/tiflow/issues/1625)
        - TiCDCオーナーがパニックを起こした際のDDLの損失問題を修正します [#1260](https://github.com/pingcap/tiflow/issues/1260)
        - 4.0.xクラスターにおけるデフォルトのソートエンジンオプションでのCLIの互換性の問題を修正します [#2373](https://github.com/pingcap/tiflow/issues/2373)
        - TiCDCが`ErrSchemaStorageTableMiss`エラーを受け取った際にチェンジフィードが予期せずリセットされる可能性があるバグを修正します [#2422](https://github.com/pingcap/tiflow/issues/2422)
        - TiCDCが`ErrGCTTLExceeded`エラーを受け取った際にチェンジフィードを削除できないバグを修正します [#2391](https://github.com/pingcap/tiflow/issues/2391)
        - TiCDCが大きなテーブルをcdclogに同期できない問題を修正します [#1259](https://github.com/pingcap/tiflow/issues/1259) [#2424](https://github.com/pingcap/tiflow/issues/2424)
        - TiCDCがテーブルを再スケジュールしている間に複数のプロセッサが同じテーブルにデータを書き込む可能性があるバグを修正します [#2230](https://github.com/pingcap/tiflow/issues/2230)

    + バックアップ＆リストア（BR）

        - リストア時にBRがすべてのシステムテーブルをスキップするバグを修正します [#1197](https://github.com/pingcap/br/issues/1197) [#1201](https://github.com/pingcap/br/issues/1201)
        - cdclogをリストアする際にDDL操作が欠落する問題を修正します [#870](https://github.com/pingcap/br/issues/870)

    + TiDB Lightning

        - Parquetファイルで`DECIMAL`データ型を解析できない問題を修正します [#1272](https://github.com/pingcap/br/pull/1272)
        - テーブルスキーマをリストアする際にエラー"Error 9007: Write conflict"が発生する問題を修正します [#1290](https://github.com/pingcap/br/issues/1290)
        - ローカルバックエンドモードでデータ損失によりチェックサム不一致エラーが発生する可能性がある問題を修正します [#1403](https://github.com/pingcap/br/issues/1403)
        - クラスタードインデックスとの互換性の問題を修正します [#1362](https://github.com/pingcap/br/issues/1362)

    + Dumpling

        - Dumpling GCセーフポイントが設定されるのが遅すぎてデータのエクスポートに失敗するバグを修正します [#290](https://github.com/pingcap/dumpling/pull/290)
        - 特定のMySQLバージョンでアップストリームデータベースからテーブル名をエクスポートする際にDumplingがスタックする問題を修正します [#322](https://github.com/pingcap/dumpling/issues/322)