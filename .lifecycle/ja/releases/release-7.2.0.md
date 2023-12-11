---
title: TiDB 7.2.0 リリースノート
summary: TiDB 7.2.0 における新機能、互換性の変更、改善、バグ修正について学びます。

# TiDB 7.2.0 リリースノート

リリース日: 2023年6月29日

TiDB バージョン: 7.2.0

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v7.2/quick-start-with-tidb) | [インストールパッケージ](https://www.pingcap.com/download/?version=v7.2.0#version-list)

7.2.0 では以下の主な機能と改善が導入されています:

<table>
<thead>
  <tr>
    <th>カテゴリ</th>
    <th>機能</th>
    <th>説明</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="2">スケーラビリティとパフォーマンス</td>
    <td>リソースグループのサポート <a href="https://docs.pingcap.com/tidb/v7.2/tidb-resource-control#manage-queries-that-consume-more-resources-than-expected-runaway-queries"> 実験的にランアウェイクエリの管理 </a></td>
    <td>より詳細なクエリタイムアウトの管理が可能になり、クエリの分類に基づいた異なる動作が可能となりました。指定した閾値を満たすクエリは優先度を下げられるか、中断されることがあります。</td>
  </tr>
  <tr>
    <td>TiFlash は <a href="https://docs.pingcap.com/tidb/v7.2/tiflash-pipeline-model">パイプライン実行モデル</a> をサポート (実験的)</td>
    <td>TiFlash はパイプライン実行モデルをサポートし、スレッドリソースの制御を最適化します。</td>
  </tr>
  <tr>
    <td rowspan="1">SQL</td>
    <td>データのインポートのための新しい SQL ステートメント <a href="https://docs.pingcap.com/tidb/v7.2/sql-statement-import-into">IMPORT INTO</a> をサポート (実験的)</td>
   <td>TiDB Lightning の展開とメンテナンスを簡略化するために、TiDB はリモートから TiDB に直接インポートする新しい SQL ステートメント <code>IMPORT INTO</code> を導入しました。</td>
  </tr>
  <tr>
    <td rowspan="2">DB操作および可観測性</td>
    <td>DDL が一時停止および再開操作をサポート (実験的)</td>
    <td>この新機能により、インデックス作成などのリソース消費の激しい DDL 操作を一時停止し、リソースを保存しオンライントラフィックへの影響を最小限に抑えることができます。必要に応じてこれらの操作をシームレスに再開でき、キャンセルして再起動する必要はありません。この機能により、リソース利用率が向上し、ユーザーエクスペリエンスが向上し、スキーマ変更が合理化されます。</td>
  </tr>
</tbody>
</table>

## 機能の詳細

### パフォーマンス

* 次の2つの[ウィンドウ関数](/tiflash/tiflash-supported-pushdown-calculations.md)を TiFlash にプッシュダウンサポート [#7427](https://github.com/pingcap/tiflash/issues/7427) @[xzhangxian1008](https://github.com/xzhangxian1008)

    * `FIRST_VALUE`
    * `LAST_VALUE`

* TiFlash はパイプライン実行モデルをサポート (実験的) [#6518](https://github.com/pingcap/tiflash/issues/6518) @[SeaRise](https://github.com/SeaRise)

    v7.2.0 より前では、TiFlash エンジンの各タスクは実行中にスレッドリソースを個別に要求していました。TiFlash はスレッドリソースの使用を制限し、過度な使用を防いでいましたが、この問題は完全には解決されなかった。この問題に対処するため、v7.2.0 から TiFlash はパイプライン実行モデルを導入しました。このモデルではすべてのスレッドリソースを中央で管理し、タスクの実行を一元的にスケジュールします。これによりスレッドリソースの使用を最大限に活用しながら過度な使用を避けることができます。パイプライン実行モデルを有効または無効にするには [`tidb_enable_tiflash_pipeline_model`](https://docs.pingcap.com/tidb/v7.2/system-variables#tidb_enable_tiflash_pipeline_model-new-in-v720) システム変数を変更してください。

    詳細については [ドキュメント](/tiflash/tiflash-pipeline-model.md) を参照してください。

* TiFlash はスキーマのレプリケーションの遅延を削減 [#7630](https://github.com/pingcap/tiflash/issues/7630) @[hongyunyan](https://github.com/hongyunyan)

    テーブルのスキーマが変更されると、TiFlash は TiKV から最新のスキーマをタイムリーにレプリケートする必要があります。v7.2.0 より前では、TiFlash はデータベース内のテーブルのスキーマ変更を検出すると、TiFlash レプリカの無いテーブルも含め、このデータベース内のすべてのテーブルのスキーマを再度レプリケートする必要がありました。その結果、多数のテーブルを含むデータベースでは、TiFlash を使用して単一のテーブルからデータを読み取る場合でも、すべてのテーブルのスキーマレプリケーションを TiFlash が完了するまで待つことがありました。

    v7.2.0 では、TiFlash はスキーマのレプリケーションメカニズムを最適化し、TiFlash レプリカを持つテーブルのスキーマのみをレプリケートするようにサポートします。TiFlash レプリカを持つテーブルのスキーマ変更が検出された場合、TiFlash はそのテーブルのスキーマのみをレプリケートし、TiFlash データのレプリケーションへのDDL操作の影響を最小限に抑えます。この最適化は自動的に適用され、手動での構成は必要ありません。

* 統計情報収集のパフォーマンスを向上 [#44725](https://github.com/pingcap/tidb/issues/44725) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)

    TiDB v7.2.0 では、統計情報収集戦略を最適化し、重複する情報やオプティマイザにとって価値の低い情報をスキップしました。統計情報収集の全体的な速度が30%改善されました。この改善により、TiDB はデータベースの統計情報をより適切なタイミングで更新し、生成される実行計画をより正確にすることで、全体的なデータベースのパフォーマンスを向上させることができます。

    デフォルトでは、統計情報収集は `JSON`、 `BLOB`、 `MEDIUMBLOB` および `LONGBLOB` 型の列をスキップします。デフォルトの動作を変更するには、[`tidb_analyze_skip_column_types`](/system-variables.md#tidb_analyze_skip_column_types-new-in-v720) システム変数を設定できます。TiDB は `JSON`、 `BLOB`、 `TEXT` 型およびサブタイプをスキップすることができます。

* データおよびインデックスの整合性チェックのパフォーマンスを向上 [#43693](https://github.com/pingcap/tidb/issues/43693) @[wjhuang2016](https://github.com/wjhuang2016)

    [`ADMIN CHECK [TABLE|INDEX]`](/sql-statements/sql-statement-admin-check-table-index.md) ステートメントは、テーブルのデータとそれに対応するインデックスとの間の整合性をチェックするために使用されます。v7.2.0 では、TiDB はデータ整合性のチェック方法を最適化し、[`ADMIN CHECK [TABLE|INDEX]`](/sql-statements/sql-statement-admin-check-table-index.md) の実行効率を大幅に改善しました。大量のデータがある場合、この最適化により数百倍のパフォーマンス向上が得られます。

    この最適化はデフォルトで有効化されています（デフォルトでは [`tidb_enable_fast_table_check`](/system-variables.md#tidb_enable_fast_table_check-new-in-v720) が `ON` になっています）、大規模なテーブルのデータ整合性チェックに必要な時間が大幅に削減され、運用効率が向上します。

### 信頼性

* 予期せぬリソース消費クエリを自動的に管理 (実験的) [#43691](https://github.com/pingcap/tidb/issues/43691) @[Connor1996](https://github.com/Connor1996) @[CabinfeverB](https://github.com/CabinfeverB) @[glorv](https://github.com/glorv) @[HuSharp](https://github.com/HuSharp) @[nolouch](https://github.com/nolouch)

    データベースの安定性への最も一般的な脅威は、急激な SQL パフォーマンスの問題による総合的なデータベースパフォーマンスの低下です。SQL パフォーマンスの問題の原因は、完全にテストされていない新しい SQL ステートメント、データボリュームの急激な変化、実行計画の急激な変更など、多くの原因があります。これらの問題を根本的に完全に避けることは困難です。TiDB v7.2.0 は予期せぬリソース消費クエリを管理する機能を提供します。この機能により、パフォーマンスの問題が発生した際に影響範囲を迅速に縮小できます。

    これらのクエリを管理するために、リソースグループのクエリの最大実行時間を設定できます。クエリの実行時間がこの制限を超えると、クエリは自動的に優先度を下げるかキャンセルされます。また、テキストまたは実行計画で識別されたクエリにすぐに一致する時間を設定できます。これにより、問題のあるクエリの高同時実行を防ぎ、予想以上のリソースを消費することができます。

    予期せぬリソース消費クエリを自動管理することにより、予期せぬクエリのパフォーマンス問題に迅速に対処する手段を提供します。この機能により、データベース全体のパフォーマンスに対する問題の影響を軽減し、データベースの安定性が向上します。

```
  詳細については、[documentation](/tidb-resource-control.md#manage-queries-that-consume-more-resources-than-expected-runaway-queries) を参照してください。

* ヒストリカル実行プランに従ってバインディングを作成する能力を強化する [#39199](https://github.com/pingcap/tidb/issues/39199) @[qw4990](https://github.com/qw4990)

    TiDB v7.2.0 により、[ヒストリカル実行プランに従ってバインディングを作成する能力](/sql-plan-management.md#create-a-binding-according-to-a-historical-execution-plan) が強化されました。この機能により、複雑なステートメントの解析とバインディングプロセスが改善され、バインディングがより安定し、以下の新しいヒントがサポートされます：

    - [`AGG_TO_COP()`](/optimizer-hints.md#agg_to_cop)
    - [`LIMIT_TO_COP()`](/optimizer-hints.md#limit_to_cop)
    - [`ORDER_INDEX`](/optimizer-hints.md#order_indext1_name-idx1_name--idx2_name-)
    - [`NO_ORDER_INDEX()`](/optimizer-hints.md#no_order_indext1_name-idx1_name--idx2_name-)

より詳細については、[documentation](/sql-plan-management.md) を参照してください。

* 最適化漏れコントロールメカニズムを導入し、最適化の振る舞いを細かく制御することを提供する [#43169](https://github.com/pingcap/tidb/issues/43169) @[time-and-fate](https://github.com/time-and-fate)

    より合理的な実行プランを生成するために、TiDBの最適化プロセスは製品のイテレーションを経て進化しています。しかし、特定のシナリオでは、変更がパフォーマンスの低下につながる可能性があります。TiDB v7.2.0 では、最適化漏れコントロールを導入し、最適化の細かい振る舞いを制御することができます。これにより、新しい変更をロールバックしたり制御したりできます。

    それぞれの制御可能な振る舞いは、修正番号に対応するGitHubの問題で説明されています。すべての制御可能な振る舞いは、[Optimizer Fix Controls](/optimizer-fix-controls.md) にリストされています。[`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v710) システム変数を設定して、1つまたは複数の振る舞いに対するターゲット値を設定することで、振る舞い制御を実現できます。

    最適化漏れコントロールメカニズムは、TiDBの最適化を細かく制御できるようにします。アップグレードプロセスによって引き起こされたパフォーマンスの問題を修正する新たな手段を提供し、TiDBの安定性を向上させます。

    詳細については、[documentation](/optimizer-fix-controls.md) を参照してください。

* 軽量統計情報の初期化が一般的に利用可能になりました(GA) [#42160](https://github.com/pingcap/tidb/issues/42160) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)

    v7.2.0 から、軽量統計情報の初期化機能が一般的に利用可能になりました。軽量統計情報の初期化により、起動時に読み込む必要のある統計情報の数が大幅に減少し、統計情報の読み込み速度が向上します。この機能により、TiDBの安定性が複雑なランタイム環境で向上し、TiDBノードの再起動時に全体サービスに与える影響が軽減されます。

    v7.2.0以降の新しく作成されたクラスタでは、TiDBはデフォルトで起動時に軽量統計情報を読み込み、サービスを提供する前に読み込みが完了するまで待機します。以前のバージョンからアップグレードされたクラスタでは、TiDB構成項目 [`lite-init-stats`](/tidb-configuration-file.md#lite-init-stats-new-in-v710) および [`force-init-stats`](/tidb-configuration-file.md#force-init-stats-new-in-v710) を `true` に設定することで、この機能を有効にできます。

    詳細については、[documentation](/statistics.md#load-statistics) を参照してください。

### SQL

* `CHECK`制約をサポートする [#41711](https://github.com/pingcap/tidb/issues/41711) @[fzzf678](https://github.com/fzzf678)

    v7.2.0から、`CHECK`制約を使用して、テーブルの1つ以上の列の値を指定した条件に合わせることができます。`CHECK`制約がテーブルに追加されると、TiDBは、その制約が満たされているかどうかを確認し、テーブルへのデータの挿入または更新の前に制約が満たされているかをチェックします。制約を満たすデータのみが書き込み可能です。

    この機能はデフォルトで無効になっています。[`tidb_enable_check_constraint`](/system-variables.md#tidb_enable_check_constraint-new-in-v720) システム変数を `ON` に設定することで有効にできます。

    詳細については、[documentation](/constraints.md#check) を参照してください。

### DB操作

* DDLジョブが一時停止および再開操作をサポートする（実験的） [#18015](https://github.com/pingcap/tidb/issues/18015) @[godouxm](https://github.com/godouxm)

    TiDB v7.2.0以前では、DDLジョブが実行中にビジネスピークに遭遇すると、ビジネスに与える影響を減らすために、DDLジョブを手動でキャンセルするしかありませんでした。v7.2.0では、DDLジョブを一時停止および再開する機能が導入されました。これらの操作を使用すると、ピーク時にDDLジョブを一時停止し、ピークが終了した後に再開することができ、アプリケーションのワークロードに影響を与えることを避けることができます。

    たとえば、`ADMIN PAUSE DDL JOBS 1,2;` や `ADMIN RESUME DDL JOBS 1,2;` を使用して複数のDDLジョブを一時停止および再開できます。

    詳細については、[documentation](/ddl-introduction.md#ddl-related-commands) を参照してください。

### データ移行

* データインポートの効率を大幅に改善する新しいSQLステートメント `IMPORT INTO` を導入する（実験的） [#42930](https://github.com/pingcap/tidb/issues/42930) @[D3Hunter](https://github.com/D3Hunter)

    `IMPORT INTO` ステートメントは、TiDBライトニングの[物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode.md)機能を統合しています。このステートメントを使用すると、CSV、SQL、PARQUETなどの形式のデータを、TiDBの空のテーブルに迅速にインポートできます。このインポート方式により、独立したTiDBライトニングの展開と管理の必要がなくなり、データインポートの複雑さが減少し、インポート効率が大幅に向上します。

    Amazon S3やGCSに保存されているデータファイルについて、[バックエンドタスク分散実行フレームワーク](/tidb-distributed-execution-framework.md)が有効になっている場合、`IMPORT INTO`ではデータインポートジョブを複数のサブジョブに分割し、それらを複数のTiDBノードに並列インポートすることができ、さらにインポートのパフォーマンスが向上します。

    詳細については、[documentation](/sql-statements/sql-statement-import-into.md) を参照してください。

* TiDBライトニングは、Latin-1文字セットでのソースファイルのインポートをサポートします [#44434](https://github.com/pingcap/tidb/issues/44434) @[lance6716](https://github.com/lance6716)

    この機能により、TiDBライトニングを使用してLatin-1文字セットでのソースファイルを直接TiDBにインポートすることができます。v7.2.0以前は、そのようなファイルをインポートするには、追加の前処理や変換が必要でした。v7.2.0以降では、TiDBライトニングのインポートタスクを設定する際に `character-set = "latin1"` を指定するだけで済みます。その後、TiDBライトニングはインポートプロセス中に文字セットの変換を自動的に処理してデータの整合性と精度を確保します。

    詳細については、[documentation](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) を参照してください。

## 互換性の変更

> **注記:**
>
> このセクションでは、v7.1.0から現在のバージョン（v7.2.0）にアップグレードする際に把握する必要のある互換性の変更が提供されます。v7.0.0またはそれ以前のバージョンから現在のバージョンにアップグレードする場合は、中間バージョンで導入された互換性の変更も確認する必要があります。

### システム変数

| 変数名 | 変更タイプ | 説明 |
|--------|------------------------------|------|
| [`last_insert_id`](/system-variables.md#last_insert_id) | 修正 | 最大値を `9223372036854775807` から `18446744073709551615` に変更し、MySQLと一致させました。 |
| [`tidb_enable_non_prepared_plan_cache`](/system-variables.md#tidb_enable_non_prepared_plan_cache) | 修正 | 追加のテスト後にデフォルト値を `OFF` から `ON` に変更し、非プリペアド実行プランキャッシュを有効にしました。 |
| [`tidb_remove_orderby_in_subquery`](/system-variables.md#tidb_remove_orderby_in_subquery-new-in-v610) | 修正 | 追加のテスト後にデフォルト値を `OFF` から `ON` に変更し、最適化プロセスでサブクエリ内の `ORDER BY` 句を削除するようにしました。 |
| [`tidb_analyze_skip_column_types`](/system-variables.md#tidb_analyze_skip_column_types-new-in-v720) | 追加 | `ANALYZE`コマンドを実行する際に統計情報収集をスキップする列の型を制御します。この変数は、[`tidb_analyze_version = 2`](/system-variables.md#tidb_analyze_version-new-in-v510) に適用されます。`ANALYZE TABLE t COLUMNS c1, ..., cn` の構文を使用する場合、指定された列の型が`tidb_analyze_skip_column_types`に含まれている場合、その列の統計情報は収集されません。|
```
| [`tidb_enable_check_constraint`](/system-variables.md#tidb_enable_check_constraint-new-in-v720) | 新たに追加された | `CHECK` 制約を有効にするかどうかを制御します。デフォルト値は `OFF` で、つまり、この機能は無効になっています。 |
| [`tidb_enable_fast_table_check`](/system-variables.md#tidb_enable_fast_table_check-new-in-v720) | 新たに追加された | チェックサムベースのアプローチを使用して、テーブルのデータとインデックスの整合性を迅速にチェックするかどうかを制御します。デフォルト値は `ON` で、つまり、この機能は有効になっています。 |
| [`tidb_enable_tiflash_pipeline_model`](https://docs.pingcap.com/tidb/v7.2/system-variables#tidb_enable_tiflash_pipeline_model-new-in-v720) | 新たに追加された | TiFlash の新しい実行モデル、[パイプラインモデル](/tiflash/tiflash-pipeline-model.md) を有効にするかどうかを制御します。デフォルト値は `OFF` で、つまり、パイプラインモデルは無効になっています。 |
| [`tidb_expensive_txn_time_threshold`](/system-variables.md#tidb_expensive_txn_time_threshold-new-in-v720) | 新たに追加された | 高コストなトランザクションのログを記録するしきい値を制御します。デフォルトでは 600 秒です。トランザクションの期間がしきい値を超え、かつトランザクションがコミットされずロールバックされない場合は、高コストなトランザクションと見なされ、ログが記録されます。 |

### 設定ファイルのパラメータ

| 設定ファイル | 設定パラメータ | 変更タイプ | 説明 |
| -------- | -------- | -------- | -------- |
| TiDB | [`lite-init-stats`](/tidb-configuration-file.md#lite-init-stats-new-in-v710) | 変更されました | 追加のテスト後、デフォルト値を `false` から `true` に変更し、これにより、TiDB が起動時にデフォルトで軽量な統計の初期化を使用して初期化効率を改善します。 |
| TiDB | [`force-init-stats`](/tidb-configuration-file.md#force-init-stats-new-in-v710) | 変更されました | [`lite-init-stats`](/tidb-configuration-file.md#lite-init-stats-new-in-v710) に合わせて、デフォルト値を `false` から `true` に変更し、TiDB が起動時にサービスを提供する前に統計の初期化が完了するのを待ちます。 |
| TiKV | [<code>rocksdb.\[defaultcf\|writecf\|lockcf\].compaction-guard-min-output-file-size</code>](/tikv-configuration-file.md#compaction-guard-min-output-file-size) | 変更されました | デフォルト値を `"8MB"` から `"1MB"` に変更して、RocksDB のコンパクションタスクのデータ量を減らします。 |
| TiKV | [<code>rocksdb.\[defaultcf\|writecf\|lockcf\].optimize-filters-for-memory</code>](/tikv-configuration-file.md#optimize-filters-for-memory-new-in-v720) | 新たに追加された | メモリ内部のフラグメント化を最小限に抑えるブルーム/Ribbon フィルタを生成するかどうかを制御します。 |
| TiKV | [<code>rocksdb.\[defaultcf\|writecf\|lockcf\].periodic-compaction-seconds</code>](/tikv-configuration-file.md#periodic-compaction-seconds-new-in-v720) | 新たに追加された | 定期的なコンパクションの時間間隔を制御します。この値より古い更新を持つ SST ファイルは、コンパクションの対象となり、元々その SST ファイルが存在したレベルに書き直されます。 |
| TiKV | [<code>rocksdb.\[defaultcf\|writecf\|lockcf\].ribbon-filter-above-level</code>](/tikv-configuration-file.md#ribbon-filter-above-level-new-in-v720) | 新たに追加された | この値以上のレベルにおいて Ribbon フィルタを使用し、この値未満のレベルにおいてブロックベースのブルームフィルタを使用するかどうかを制御します。 |
| TiKV | [<code>rocksdb.\[defaultcf\|writecf\|lockcf\].ttl</code>](/tikv-configuration-file.md#ttl-new-in-v720) | 新たに追加された | TTL より古い更新を持つ SST ファイルは、自動的にコンパクションの対象になります。 |
| TiDB Lightning | `send-kv-pairs` | 廃止されました | v7.2.0 以降、`send-kv-pairs` パラメータは廃止されました。物理インポートモードで TiKV にデータを送信する際の 1 つのリクエストの最大サイズを制御するには、[`send-kv-size`](/tidb-lightning/tidb-lightning-configuration.md) を使用できます。 |
| TiDB Lightning | [`character-set`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) | 変更されました | データインポートのサポートされる文字セットに対して新しい値オプション `latin1` を導入しました。このオプションを使用して、Latin-1 文字セットを持つソースファイルをインポートできます。 |
| TiDB Lightning | [`send-kv-size`](/tidb-lightning/tidb-lightning-configuration.md) | 新たに追加された | 物理インポートモードで TiKV にデータを送信する際の 1 つのリクエストの最大サイズを指定します。キー値のペアのサイズが指定された閾値に達すると、TiDB Lightning はそれらをすぐに TiKV に送信します。これにより、大きなワイドテーブルをインポートする際に TiDB Lightning ノードがメモリに余りすぎたキー値のペアを蓄積してしまうことによる OOM 問題を回避できます。このパラメータを調整することで、メモリ使用量とインポート速度のバランスを見つけることができ、インポートプロセスの安定性と効率を向上させることができます。 |
| データ移行 | [`strict-optimistic-shard-mode`](/dm/feature-shard-merge-optimistic.md) | 新たに追加された | この設定項目は、TiDB Data Migration v2.0 での DDL シャードマージの動作と互換性を保つために使用されます。この設定項目を楽観的モードで有効にできます。これを有効にすると、レプリケーションタスクは Type 2 DDL ステートメントに遭遇したときに中断されます。複数のテーブルでの DDL 変更の間に依存関係がある場合、適時な中断が行われます。レプリケーションタスクを再開する前に、各テーブルの DDL ステートメントを手動で処理して、上流と下流のデータの整合性を確保する必要があります。 |
| TiCDC | [`sink.protocol`](/ticdc/ticdc-changefeed-config.md) | 変更されました | 下流が Kafka の場合、`"open-protocol"` を指定してエンコードメッセージに使用するプロトコル形式を指定します。 |
| TiCDC | [`sink.delete-only-output-handle-key-columns`](/ticdc/ticdc-changefeed-config.md) | 新たに追加された | DELETE イベントの出力を指定します。このパラメータは `"canal-json"` および `"open-protocol"` プロトコルの場合にのみ有効です。デフォルト値は `false` で、つまり、すべての列を出力します。これを `true` に設定すると、プライマリキー列または一意のインデックス列のみが出力されます。 |

## 改善点

+ TiDB

    - 複雑な条件をインデックススキャン範囲に変換するロジックを最適化して、索引スキャン範囲の構築をサポートするようにしました [#41572](https://github.com/pingcap/tidb/issues/41572) [#44389](https://github.com/pingcap/tidb/issues/44389) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
    - 新しい監視メトリクス `Stale Read OPS` および `Stale Read Traffic` を追加しました [#43325](https://github.com/pingcap/tidb/issues/43325) @[you06](https://github.com/you06)
    - ステール読み取りのリトライリーダーがロックに遭遇した場合、TiDB はロックを解決し、その後でリーダーで強制的にリトライすることで不要なオーバーヘッドを回避します [#43659](https://github.com/pingcap/tidb/issues/43659) @[you06](https://github.com/you06)
    - 推定時間を使用してステール読み取りのタイムスタンプを計算し、ステール読み取りのオーバーヘッドを削減します [#44215](https://github.com/pingcap/tidb/issues/44215) @[you06](https://github.com/you06)
    - 長時間実行されるトランザクションのためのログとシステム変数を追加しました [#41471](https://github.com/pingcap/tidb/issues/41471) @[crazycs520](https://github.com/crazycs520)
    - 低帯域幅ネットワーク下でのデータ集中型クエリのパフォーマンスを向上させ、帯域幅コストを削減するため、TiDB を圧縮した MySQL プロトコルを介して接続するサポートを追加しました。これは `zlib` および `zstd` ベースの圧縮をサポートしています。[#22605](https://github.com/pingcap/tidb/issues/22605) @[dveeden](https://github.com/dveeden)
    - `utf8` および `utf8mb3` の両方を、遺産の三バイト UTF-8 文字セットエンコーディングとして認識しました。これにより、MySQL 8.0 から TiDB への遺産 UTF-8 エンコーディングを持つテーブルの移行が容易になります [#26226](https://github.com/pingcap/tidb/issues/26226) @[dveeden](https://github.com/dveeden)
    - `UPDATE` ステートメントでの代入に `:=` を使用するサポートを追加しました [#44751](https://github.com/pingcap/tidb/issues/44751) @[CbcWestwolf](https://github.com/CbcWestwolf)

+ TiKV

    - パラメーターダウン失敗などのシナリオで PD 接続のリトライ間隔を構成するサポートを追加しました `pd.retry-interval` [#14964](https://github.com/tikv/tikv/issues/14964) @[rleungx](https://github.com/rleungx)
    - グローバルリソース使用量を組み込んだリソース制御スケジューリングアルゴリズムを最適化しました [#14604](https://github.com/tikv/tikv/issues/14604) @[Connor1996](https://github.com/Connor1996)
- `check_leader`リクエストのトラフィックを削減するためにgzip圧縮を使用します [#14553](https://github.com/tikv/tikv/issues/14553) @[you06](https://github.com/you06) に関するメトリクスを追加します
- TiKVが書き込みコマンドを処理する際に詳細な時間情報を提供します [#12362](https://github.com/tikv/tikv/issues/12362) @[cfzjywxk](https://github.com/cfzjywxk)

+ PD

    - 他のリクエストへの影響を防ぐために、PDリーダー選出用の独立したgRPC接続を使用します [#6403](https://github.com/tikv/pd/issues/6403) @[rleungx](https://github.com/rleungx)
    - マルチリージョンのシナリオでのホットスポットの問題を緩和するために、バケットの分割をデフォルトで有効にします [#6433](https://github.com/tikv/pd/issues/6433) @[bufferflies](https://github.com/bufferflies)

+ ツール

    + バックアップとリストア (BR)

        - 共有アクセス署名（SAS）によるAzure Blob Storageへのアクセスをサポートします [#44199](https://github.com/pingcap/tidb/issues/44199) @[Leavrth](https://github.com/Leavrth)

    + TiCDC

        - リプリケーション先のオブジェクトストレージサービスにおいてDDL操作が発生した際にデータファイルのディレクトリ構造を最適化します [#8891](https://github.com/pingcap/tiflow/issues/8891) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - KafkaへのリプリケーションシナリオにおいてOAUTHBEARER認証をサポートします [#8865](https://github.com/pingcap/tiflow/issues/8865) @[hi-rustin](https://github.com/hi-rustin)
        - Kafkaへのリプリケーションシナリオにおいて`DELETE`操作のハンドルキーのみを出力するオプションを追加します [#9143](https://github.com/pingcap/tiflow/issues/9143) @[3AceShowHand](https://github.com/3AceShowHand)

    + TiDBデータマイグレーション（DM）

        - MySQL 8.0での圧縮されたbinlogの読み取りをインクリメンタルレプリケーションのデータソースとしてサポートします [#6381](https://github.com/pingcap/tiflow/issues/6381) @[dveeden](https://github.com/dveeden)

    + TiDB Lightning

        - エラースイッチングによるエラーの回避のためにインポート時のリトライメカニズムを最適化します [#44478](https://github.com/pingcap/tidb/pull/44478) @[lance6716](https://github.com/lance6716)
        - 安定性を向上させるためにインポート後にSQLでチェックサムを検証します [#41941](https://github.com/pingcap/tidb/issues/41941) @[GMHDBJD](https://github.com/GMHDBJD)
        - ワイドテーブルのインポート時のOOM問題を最適化します [#43853](https://github.com/pingcap/tidb/issues/43853) @[D3Hunter](https://github.com/D3Hunter)

## バグ修正

+ TiDB

    - CTEを含むクエリがTiDBをハングさせる問題を修正します [#43749](https://github.com/pingcap/tidb/issues/43749) [#36896](https://github.com/pingcap/tidb/issues/36896) @[guo-shaoge](https://github.com/guo-shaoge)
    - `min, max`クエリの結果が正しくない問題を修正します [#43805](https://github.com/pingcap/tidb/issues/43805) @[wshwsh12](https://github.com/wshwsh12)
    - `SHOW PROCESSLIST`ステートメントが長いサブクエリ時間のトランザクションの`TxnStart`を表示できない問題を修正します [#40851](https://github.com/pingcap/tidb/issues/40851) @[crazycs520](https://github.com/crazycs520)
    - `TxnScope`の欠如によりステイルリードグローバル最適化が効果を発揮しない問題を修正します [#43365](https://github.com/pingcap/tidb/issues/43365) @[you06](https://github.com/you06)
    - フラッシュバックエラーを処理する前にフォロワーリードが適切に処理されないことによりクエリエラーが発生する問題を修正します [#43673](https://github.com/pingcap/tidb/issues/43673) @[you06](https://github.com/you06)
    - `ON UPDATE`ステートメントがプライマリキーを正しく更新しない場合、データとインデックスが不整合になる問題を修正します [#44565](https://github.com/pingcap/tidb/issues/44565) @[zyguan](https://github.com/zyguan)
    - `UNIX_TIMESTAMP()`関数の上限をMySQL 8.0.28以降のバージョンと整合した`3001-01-19 03:14:07.999999 UTC`に変更します [#43987](https://github.com/pingcap/tidb/issues/43987) @[YangKeao](https://github.com/YangKeao)
    - インゲストモードでインデックスの追加が失敗する問題を修正します [#44137](https://github.com/pingcap/tidb/issues/44137) @[tangenta](https://github.com/tangenta)
    - ロールバック状態でDDLタスクをキャンセルすると関連するメタデータにエラーが発生する問題を修正します [#44143](https://github.com/pingcap/tidb/issues/44143) @[wjhuang2016](https://github.com/wjhuang2016)
    - `memTracker`をカーソルフェッチと共に使用するとメモリリークが発生する問題を修正します [#44254](https://github.com/pingcap/tidb/issues/44254) @[YangKeao](https://github.com/YangKeao)
    - データベースの削除による遅いGC進捗を修正します [#33069](https://github.com/pingcap/tidb/issues/33069) @[tiancaiamao](https://github.com/tiancaiamao)
    - インデックスジョインのプローブフェーズでパーティションテーブルの対応行が見つからない場合にTiDBがエラーを返さない問題を修正します [#43686](https://github.com/pingcap/tidb/issues/43686) @[AilinKid](https://github.com/AilinKid) @[mjonss](https://github.com/mjonss)
    - `SUBPARTITION`を使用してパーティションテーブルを作成する際に警告が表示されない問題を修正します [#41198](https://github.com/pingcap/tidb/issues/41198) [#41200](https://github.com/pingcap/tidb/issues/41200) @[mjonss](https://github.com/mjonss)
    - `MAX_EXECUTION_TIME`を超えてクエリをキルした際にTiDBがエラーを返さない問題を修正します [#43031](https://github.com/pingcap/tidb/issues/43031) @[dveeden](https://github.com/dveeden)
    - `LEADING`ヒントがブロックエイリアスのクエリをサポートしていない問題を修正します [#44645](https://github.com/pingcap/tidb/issues/44645) @[qw4990](https://github.com/qw4990)
    - `LAST_INSERT_ID()`関数の戻り値の型をVARCHARからLONGLONGに変更します。これはMySQLの動作に整合させるためのものです [#44574](https://github.com/pingcap/tidb/issues/44574) @[Defined2014](https://github.com/Defined2014)
    - 非相関サブクエリを含むステートメントで不正な結果が返される問題を修正します [#44051](https://github.com/pingcap/tidb/issues/44051) @[winoros](https://github.com/winoros)
    - Join Reorderが不正なアウタージョイン結果を引き起こす問題を修正します [#44314](https://github.com/pingcap/tidb/issues/44314) @[AilinKid](https://github.com/AilinKid)
    - `PREPARE stmt FROM "ANALYZE TABLE xxx"`が`tidb_mem_quota_query`によってキルされる可能性がある問題を修正します [#44320](https://github.com/pingcap/tidb/issues/44320) @[chrysan](https://github.com/chrysan)

+ TiKV

    - TiKVがステイルペシミスティックロックのコンフリクトを処理する際にトランザクションが不正な値を返す問題を修正します [#13298](https://github.com/tikv/tikv/issues/13298) @[cfzjywxk](https://github.com/cfzjywxk)
    - インメモリペシミスティックロックがフォールバックの失敗やデータの不整合を引き起こす可能性がある問題を修正します [#13303](https://github.com/tikv/tikv/issues/13303) @[JmPotato](https://github.com/JmPotato)
    - TiKVがステイルリクエストを処理する際にフェアロックが不正確である可能性がある問題を修正します [#13298](https://github.com/tikv/tikv/issues/13298) @[cfzjywxk](https://github.com/cfzjywxk)
- `autocommit` および `point get replica read` の問題を修正して、一貫性を保てなくなる可能性がある問題を修正しました [#14715](https://github.com/tikv/tikv/issues/14715) @[cfzjywxk](https://github.com/cfzjywxk)

+ PD

    - ある一部のケースで冗長なレプリカを自動修復できない問題を修正しました [#6573](https://github.com/tikv/pd/issues/6573) @[nolouch](https://github.com/nolouch)

+ TiFlash

    - クエリが非常に大きなJoinビルド側のデータに多数の小さな文字列型列が含まれている場合に、クエリが必要なより多くのメモリを消費する問題を修正しました [#7416](https://github.com/pingcap/tiflash/issues/7416) @[yibin87](https://github.com/yibin87)

+ ツール

    + バックアップおよびリストア（BR）

        - あるケースで `checksum mismatch` が誤って報告される問題を修正しました [#44472](https://github.com/pingcap/tidb/issues/44472) @[Leavrth](https://github.com/Leavrth)
        - あるケースで `resolved lock timeout` が誤って報告される問題を修正しました [#43236](https://github.com/pingcap/tidb/issues/43236) @[YuJuncen](https://github.com/YuJuncen)
        - TiDBが統計情報を復元する際にパニックを起こす可能性がある問題を修正しました [#44490](https://github.com/pingcap/tidb/issues/44490) @[tangenta](https://github.com/tangenta)

    + TiCDC

        - あるケースで Resolved TS が適切に進まない問題を修正しました [#8963](https://github.com/pingcap/tiflow/issues/8963) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - AvroまたはCSVプロトコルを使用する場合、`UPDATE` 操作が古い値を出力できない問題を修正しました [#9086](https://github.com/pingcap/tiflow/issues/9086) @[3AceShowHand](https://github.com/3AceShowHand)
        - データをKafkaにレプリケートする際に、下流メタデータを頻繁に読み取ることで過剰な下流圧力が生じる問題を修正しました [#8959](https://github.com/pingcap/tiflow/issues/8959) @[hi-rustin](https://github.com/hi-rustin)
        - TiDBまたはMySQLにデータをレプリケートする際に、下流双方向レプリケーション関連の変数を頻繁に設定することで過剰な下流ログが生じる問題を修正しました [#9180](https://github.com/pingcap/tiflow/issues/9180) @[asddongmen](https://github.com/asddongmen)
        - PDノードのクラッシュがTiCDCノードの再起動を引き起こす問題を修正しました [#8868](https://github.com/pingcap/tiflow/issues/8868) @[asddongmen](https://github.com/asddongmen)
        - TiCDCがダウンストリームKafka-on-Pulsarでchangefeedを作成できない問題を修正しました [#8892](https://github.com/pingcap/tiflow/issues/8892) @[hi-rustin](https://github.com/hi-rustin)

    + TiDB Lightning

        - `experimental.allow-expression-index` が有効でデフォルト値がUUIDの場合にTiDB Lightningがパニックを起こす問題を修正しました [#44497](https://github.com/pingcap/tidb/issues/44497) @[lichunzhu](https://github.com/lichunzhu)
        - タスクがデータファイルを分割する際に、タスクが終了するとTiDB Lightningがパニックを起こす問題を修正しました [#43195](https://github.com/pingcap/tidb/issues/43195) @[lance6716](https://github.com/lance6716)

## 貢献者

TiDBコミュニティの以下の貢献者に感謝します:

- [asjdf](https://github.com/asjdf)
- [blacktear23](https://github.com/blacktear23)
- [Cavan-xu](https://github.com/Cavan-xu)
- [darraes](https://github.com/darraes)
- [demoManito](https://github.com/demoManito)
- [dhysum](https://github.com/dhysum)
- [HappyUncle](https://github.com/HappyUncle)
- [jiyfhust](https://github.com/jiyfhust)
- [L-maple](https://github.com/L-maple)
- [nyurik](https://github.com/nyurik)
- [SeigeC](https://github.com/SeigeC)
- [tangjingyu97](https://github.com/tangjingyu97)