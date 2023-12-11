---
title: TiDB 7.4.0 リリースノート
summary: TiDB 7.4.0 の新機能、互換性変更、改善点、バグ修正について学びます。

# TiDB 7.4.0 リリースノート

リリース日: 2023年10月12日

TiDB バージョン: 7.4.0

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v7.4/quick-start-with-tidb) | [インストールパッケージ](https://www.pingcap.com/download/?version=v7.4.0#version-list)

7.4.0 では、以下の主要な機能と改善が導入されています:

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
    <td rowspan="3">信頼性と可用性</td>
    <td><code>IMPORT INTO</code> および <code>ADD INDEX</code> 操作のパフォーマンスと安定性を <a href="https://docs.pingcap.com/tidb/v7.4/tidb-global-sort" target="_blank">グローバルソート</a> を介して向上 (実験的)</td>
    <td>v7.4.0 以前は、<a href="https://docs.pingcap.com/tidb/v7.4/tidb-distributed-execution-framework" target="_blank">分散実行フレームワーク</a> を使用した <code>ADD INDEX</code> や <code>IMPORT INTO</code> などのタスクは局所的および部分的なソートを意味し、最終的に TiKV に部分的なソートの補完を行わせるために多くの余分な作業をさせていました。これらのジョブでは TiDB ノードがソートのためのローカルディスクスペースを割り当てる必要がありましたが、それが TiKV へのロードより前に行われていました。<br/>v7.4.0 でのグローバルソート機能の導入により、データは TiKV にロードされる前にグローバルな並べ替えのために外部共有ストレージ (このバージョンでは S3) に一時的に格納されます。これにより、TiKV が余分なリソースを消費する必要がなくなり、<code>ADD INDEX</code> や <code>IMPORT INTO</code> などの操作のパフォーマンスと安定性が大幅に向上します。</td>
  </tr>
  <tr>
    <td>バックグラウンドタスクのための<a href="https://docs.pingcap.com/tidb/v7.4/tidb-resource-control#manage-background-tasks" target="_blank">リソース制御</a> (実験的)</td>
    <td>v7.1.0 で、リソースおよびストレージアクセスの干渉を緩和するために <a href="https://docs.pingcap.com/tidb/v7.4/tidb-resource-control#use-resource-control-to-achieve-resource-isolation" target="_blank">リソース制御</a> 機能が導入されました。TiDB v7.4.0 では、この制御がバックグラウンドタスクにも適用されます。v7.4.0 ではリソース制御がバックグラウンドタスク（例: auto-analyze、Backup & Restore、TiDB Lightning を使用したバルクロード、オンライン DDL）で生成されるリソースを識別および管理します。これは最終的にすべてのバックグラウンドタスクに適用されます。</td>
  </tr>
  <tr>
    <td>TiFlash が <a href="https://docs.pingcap.com/tidb/v7.4/tiflash-disaggregated-and-s3" target="_blank">ストレージコンピューティング分離と S3</a> をサポート (GA) </td>
    <td>TiFlash の分離されたストレージおよびコンピューティングアーキテクチャおよび S3 共有ストレージが一般提供されます:
      <ul>
        <li>コンピューティングとストレージを分離し、弾力的な HTAP リソース利用のマイルストーンです。</li>
        <li>S3 ベースのストレージエンジンの使用をサポートし、低コストで共有ストレージを提供できます。</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td rowspan="2">SQL</td>
    <td>TiDB が <a href="https://docs.pingcap.com/tidb/v7.4/partitioned-table#convert-a-partitioned-table-to-a-non-partitioned-table" target="_blank">パーティションタイプ管理</a> をサポート </td>
    <td>v7.4.0 以前、Range/List パーティションテーブルは <code>TRUNCATE</code>, <code>EXCHANGE</code>, <code>ADD</code>, <code>DROP</code>, <code>REORGANIZE</code> などのパーティション管理操作をサポートし、Hash/Key パーティションテーブルは <code>ADD</code> および <code>COALESCE</code> などのパーティション管理操作をサポートしていました。
    <p>現在 TiDB は以下のパーティションタイプ管理操作もサポートしています:</p>
      <ul>
        <li>パーティションテーブルを非パーティションテーブルに変換</li>
        <li>既存の非パーティションテーブルをパーティション化</li>
        <li>既存のテーブルのパーティションタイプを変更</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>MySQL 8.0 互換性: <a href="https://docs.pingcap.com/tidb/v7.4/character-set-and-collation#character-sets-and-collations-supported-by-tidb" target="_blank">照合 <code>utf8mb4_0900_ai_ci</code></a> をサポート</td>
    <td>MySQL 8.0 の注目すべき変更の1つは、デフォルトの文字セットが utf8mb4 で、utf8mb4 の既定の照合順序が <code>utf8mb4_0900_ai_ci</code> であることです。TiDB v7.4.0 はこれをサポートすることで、MySQL 8.0 からの移行やレプリケーションがよりスムーズに行えるようになります。</td>
  </tr>
  <tr>
    <td>DB操作と可観測性</td>
    <td><code>IMPORT INTO</code> および <code>ADD INDEX</code> SQL ステートメントを実行するための各 TiDB ノードを指定 (実験的)</td>
    <td>既存の TiDB ノードの一部または新しく追加された TiDB ノードのいずれかで <code>IMPORT INTO</code> または <code>ADD INDEX</code> SQL ステートメントを実行するかを指定できます。このアプローチにより、TiDB ノードの残りの部分からのリソースの分離が可能となり、前述のSQLステートメントの実行に対するビジネス操作への影響を防ぎながら、最適なパフォーマンスを確保します。</td>
  </tr>
</tbody>
</table>

## 機能の詳細

### 拡張性

* 分散実行フレームワークのバックエンドの `ADD INDEX` または `IMPORT INTO` タスクを並列に実行するための TiDB ノードの選択をサポート (実験的) [#46453](https://github.com/pingcap/tidb/pull/46453) @[ywqzzy](https://github.com/ywqzzy)

    リソースが多く使用されるクラスタでの `ADD INDEX` または `IMPORT INTO` タスクの並列実行は、TiDB ノードの大量のリソースを消費し、クラスタのパフォーマンスの低下につながる可能性があります。v7.4.0 からは、システム変数 [`tidb_service_scope`](/system-variables.md#tidb_service_scope-new-in-v740) を使用して、[TiDB バックエンドタスク分散実行フレームワーク](/tidb-distributed-execution-framework.md) の各 TiDB ノードのサービス範囲を制御できます。いくつかの既存の TiDB ノードを選択するか、新しい TiDB ノードの TiDB サービス範囲を設定し、並列の `ADD INDEX` および `IMPORT INTO` タスクをこれらのノードのみで実行します。このメカニズムにより、既存のサービスに影響を与えることなく、パフォーマンスの影響を回避できます。

    詳細については、[ドキュメント](/system-variables.md#tidb_service_scope-new-in-v740) を参照してください。

* パーティションRaft KVストレージエンジンを強化 (実験的) [#11515](https://github.com/tikv/tikv/issues/11515) [#12842](https://github.com/tikv/tikv/issues/12842) @[busyjay](https://github.com/busyjay) @[tonyxuqqi](https://github.com/tonyxuqqi) @[tabokie](https://github.com/tabokie) @[bufferflies](https://github.com/bufferflies) @[5kbpers](https://github.com/5kbpers) @[SpadeA-Tang](https://github.com/SpadeA-Tang) @[nolouch](https://github.com/nolouch)

    TiDB v6.6.0 では、パーティションRaft KVストレージエンジンが実験的な機能として導入され、複数のRocksDBインスタンスを使用してTiKVリージョンデータを格納し、各リージョンのデータは独立したRocksDBインスタンスに独立して格納されます。

    v7.4.0 では、TiDB はパーティションRaft KVストレージエンジンの互換性と安定性をさらに向上させます。大規模なデータテストを通じて、DM、Dumpling、TiDB Lightning、TiCDC、BR、PITRなど、TiDBエコシステムのツールと機能との互換性が確保されています。また、パーティション化Raft KVストレージエンジンは、読み書きの混在したワークロードの下でより安定したパフォーマンスを提供し、特に書き込み重視のシナリオに適しています。さらに、各TiKVノードは8コアCPUをサポートし、8TBのデータストレージを構成でき、64GBのメモリを搭載することができます。

    詳細については、[ドキュメント](/partitioned-raft-kv.md) を参照してください。

* TiFlashは分離されたストレージとコンピュート・アーキテクチャ（GA）をサポートしています [#6882](https://github.com/pingcap/tiflash/issues/6882) @[JaySon-Huang](https://github.com/JaySon-Huang) @[JinheLin](https://github.com/JinheLin) @[breezewish](https://github.com/breezewish) @[lidezhu](https://github.com/lidezhu) @[CalvinNeo](https://github.com/CalvinNeo) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)

    v7.0.0では、TiFlashは実験的な機能として分離されたストレージとコンピュート・アーキテクチャを導入しました。改良が加えられ、TiFlash用の分離されたストレージとコンピュート・アーキテクチャはv7.4.0からGAになりました。

    このアーキテクチャでは、TiFlashノードは2つのタイプ（コンピュート・ノードとライト・ノード）に分かれ、S3 APIと互換性のあるオブジェクト・ストレージをサポートしています。両方のノードのタイプは、計算量やストレージ容量を独立してスケーリングできます。分離されたストレージとコンピュート・アーキテクチャでは、TiFlashのレプリカの作成、データのクエリ、オプティマイザ・ヒントの指定など、カップルド・ストレージとコンピュート・アーキテクチャと同じようにTiFlashを使用することができます。

    なお、TiFlashの**分離されたストレージとコンピュート・アーキテクチャ**と**カップルド・ストレージとコンピュート・アーキテクチャ**は同じクラスターで使用することはできず、相互に変換することもできません。TiFlashを展開する際にどちらのアーキテクチャを使用するかを構成できます。

    詳細は[ドキュメント](/tiflash/tiflash-disaggregated-and-s3.md)を参照してください。

### パフォーマンス

* JSONオペレータ `MEMBER OF` をTiKVにプッシュダウンするサポート [#46307](https://github.com/pingcap/tidb/issues/46307) @[wshwsh12](https://github.com/wshwsh12)

    * `value MEMBER OF(json_array)`

    詳細は[ドキュメント](/functions-and-operators/expressions-pushed-down.md)を参照してください。

* 任意のフレーム定義タイプを持つウィンドウ関数をTiFlashにプッシュダウンするサポート [#7376](https://github.com/pingcap/tiflash/issues/7376) @[xzhangxian1008](https://github.com/xzhangxian1008)

    v7.4.0より前では、TiFlashは`PRECEDING`や`FOLLOWING`を含むウィンドウ関数をサポートしておらず、そのようなフレーム定義を含むすべてのウィンドウ関数をTiFlashにプッシュダウンすることができませんでした。v7.4.0からは、TiFlashはすべてのウィンドウ関数のフレーム定義をサポートしています。この機能は自動的に有効になり、関連する要件が満たされた場合、フレーム定義を含むウィンドウ関数は自動的にTiFlashにプッシュダウンされ、実行されます。

* クラウドストレージベースのグローバルソート機能を導入し、`ADD INDEX`および`IMPORT INTO`タスクのパフォーマンスと安定性を向上させました（実験的） [#45719](https://github.com/pingcap/tidb/issues/45719) @[wjhuang2016](https://github.com/wjhuang2016)

* プリペアドステートメントでの実行計画のキャッシュをサポート（GA） [#36598](https://github.com/pingcap/tidb/issues/36598) @[qw4990](https://github.com/qw4990)

    v7.0.0では、TiDBは並行OLTPの負荷容量を改善するために実験的な非プリペアドプランキャッシュを導入しました。v7.4.0からは、この機能がGAになります。実行計画のキャッシュはさらに多くのシナリオに適用され、TiDBの並行処理能力が向上します。

    非プリペアドプランキャッシュの有効化には追加のメモリやCPUオーバーヘッドが発生する場合があり、すべての状況に適しているわけではありません。v7.4.0から、この機能はデフォルトで無効になります。[`tidb_enable_non_prepared_plan_cache`](/system-variables.md#tidb_enable_non_prepared_plan_cache)を使用して有効にし、[`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710)を使用してキャッシュサイズを制御できます。

    また、この機能はデフォルトでDMLステートメントをサポートせず、SQLステートメントには一定の制限があります。詳細については[制限事項](/sql-non-prepared-plan-cache.md#restrictions)を参照してください。

* TiFlashはクエリレベルのデータスピリングをサポートしています [#7738](https://github.com/pingcap/tiflash/issues/7738) @[windtalker](https://github.com/windtalker)

* ユーザー定義TiKVリードタイムアウトをサポートしています [#45380](https://github.com/pingcap/tidb/issues/45380) @[crazycs520](https://github.com/crazycs520)

* オプティマイザ・ヒントを使用して一時的にシステム変数値を変更するサポートを追加しました [#45892](https://github.com/pingcap/tidb/issues/45892) @[winoros](https://github.com/winoros)
TiDB v7.1.0では、リソース制御機能が一般に利用可能になり、TiDBおよびTiKVのリソース管理機能を提供します。v7.4.0では、TiFlashはリソース制御機能をサポートし、TiDBの全体的なリソース管理機能を向上させます。TiFlashのリソース制御は、既存のTiDBリソース制御機能と完全に互換性があり、既存のリソースグループはTiDB、TiKV、およびTiFlashのリソースを同時に管理します。

TiFlashリソース制御機能を有効にするかどうかを制御するには、TiFlashパラメータの`enable_resource_control`を構成できます。この機能を有効にすると、TiFlashはTiDBのリソースグループ構成に基づいてリソースのスケジューリングと管理を行い、全体のリソースの合理的な割り当てと使用を確保します。

詳細については[ドキュメント](/tidb-resource-control.md)を参照してください。

* TiFlashはパイプライン実行モデル(GA)をサポートしています[#6518](https://github.com/pingcap/tiflash/issues/6518) @[SeaRise](https://github.com/SeaRise)

v7.2.0から、TiFlashはパイプライン実行モデルを導入しています。このモデルはすべてのスレッドリソースを一元管理し、タスクの実行を均一にスケジューリングしてスレッドリソースの利用を最大化し、リソースの過度な使用を避けます。v7.4.0では、TiFlashはスレッドリソースの使用統計を改善し、パイプライン実行モデルがGA機能としてデフォルトで有効になります。この機能はTiFlashリソース制御機能と相互依存しているため、TiDB v7.4.0では、以前のバージョンでパイプライン実行モデルを有効にするために使用されていた`tidb_enable_tiflash_pipeline_model`変数が削除されます。代わりに、TiFlashパラメータの`tidb_enable_resource_control`を構成することで、パイプライン実行モデルとTiFlashリソース制御機能を有効または無効にできます。

詳細については[ドキュメント](/tiflash/tiflash-pipeline-model.md)を参照してください。

* 最適化モードのオプションを追加[#46080](https://github.com/pingcap/tidb/issues/46080) @[time-and-fate](https://github.com/time-and-fate)

v7.4.0では、TiDBは新しいシステム変数[`tidb_opt_objective`](/system-variables.md#tidb_opt_objective-new-in-v740)を導入し、オプティマイザによって使用される推定方法を制御します。デフォルト値`moderate`は、オプティマイザの以前の動作を維持し、ランタイム統計を使用してデータの変更に基づいて推定を調整します。この変数を`determinate`に設定すると、オプティマイザはランタイムの補正を考慮せずに統計のみに基づいて実行計画を生成します。

長期安定なOLTPアプリケーションや既存の実行計画に自信がある場合は、テストの後に`determinate`モードに切り替えることをお勧めします。これにより、計画の変更を減らすことができます。

詳細については[ドキュメント](/system-variables.md#tidb_opt_objective-new-in-v740)を参照してください。

* TiDBリソース制御はバックグラウンドタスクの管理をサポート(実験的) [#44517](https://github.com/pingcap/tidb/issues/44517) @[glorv](https://github.com/glorv)

データバックアップや自動統計情報の収集などのバックグラウンドタスクは低優先度ですが、多くのリソースを消費します。これらのタスクは通常、定期的または不定期にトリガされます。実行中は多くのリソースを消費し、オンラインの高優先度タスクのパフォーマンスに影響を与えます。v7.4.0から、TiDBリソース制御機能はバックグラウンドタスクの管理をサポートします。この機能により、低優先度タスクがオンラインアプリケーションのパフォーマンスに与える影響が軽減され、合理的なリソース割り当てが可能になり、クラスタの安定性が大幅に向上します。

TiDBは次の種類のバックグラウンドタスクをサポートしています:

- `lightning`: [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)または[`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)を使用してインポートタスクを実行します。
- `br`: [BR](/br/backup-and-restore-overview.md)を使用してバックアップおよびリストアタスクを実行します。PITRはサポートされていません。
- `ddl`: Reorg DDLのバッチデータ書き戻しフェーズ中のリソース使用の制御。
- `stats`: TiDBによって手動で実行されるまたは自動でトリガされる[統計情報の収集](/statistics.md#collect-statistics)タスク。

デフォルトでは、バックグラウンドタスクとしてマークされたタスクの種類は空であり、バックグラウンドタスクの管理は無効になっています。このデフォルトの動作は、TiDB v7.4.0以前のバージョンと同じです。バックグラウンドタスクを管理するには、`default`リソースグループのバックグラウンドタスクの種類を手動で変更する必要があります。

詳細については[ドキュメント](/tidb-resource-control.md#manage-background-tasks)を参照してください。

* 統計情報のロック機能を強化する[#46351](https://github.com/pingcap/tidb/issues/46351) @[hi-rustin](https://github.com/hi-rustin)

v7.4.0では、TiDBが統計情報の[ロックとロック解除](/statistics.md#lock-statistics)の機能を強化しました。運用のセキュリティを確保するために、統計情報のロックとロック解除にはあらかじめ統計情報の収集と同じ権限が必要です。さらに、TiDBは特定のパーティションに対する統計情報のロックとロック解除をサポートし、より柔軟性を提供します。データベースのクエリと実行計画に自信があり、変更を防止したい場合は、統計情報をロックして安定性を向上させることができます。

詳細については[ドキュメント](/statistics.md#lock-statistics)を参照してください。

* テーブルのハッシュ結合を選択するかどうかを制御するシステム変数を導入する[#46695](https://github.com/pingcap/tidb/issues/46695) @[coderplay](https://github.com/coderplay)

MySQL 8.0では、テーブルのハッシュ結合が新機能として導入されました。この機能は主に比較的大きなテーブルと結果セットを結合するために使用されます。ただし、トランザクションのワークロードやMySQL 5.7で実行されるいくつかのアプリケーションでは、テーブルのハッシュ結合はパフォーマンスのリスクをもたらす場合があります。MySQLは[`optimizer_switch`](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html#optflag_block-nested-loop)を提供しており、グローバルレベルまたはセッションレベルでテーブルのハッシュ結合を選択するかどうかを制御します。

v7.4.0から、TiDBはテーブルのハッシュ結合を制御するためのシステム変数[`tidb_opt_enable_hash_join`](/system-variables.md#tidb_opt_enable_hash_join-new-in-v740)を導入します。デフォルトでは(`ON`)で有効になっています。実行計画のロールバックの可能性を減らし、システムの安定性を向上させるために、実行計画でテーブル間のハッシュ結合を選択する必要がないと確信している場合は、変数を`OFF`に変更できます。

詳細については[ドキュメント](/system-variables.md#tidb_opt_enable_hash_join-new-in-v740)を参照してください。

### SQL

* TiDBはパーティションタイプの管理をサポートしています[#42728](https://github.com/pingcap/tidb/issues/42728) @[mjonss](https://github.com/mjonss)

v7.4.0以前では、TiDBのパーティションテーブルのパーティションタイプは変更できませんでした。v7.4.0から、TiDBはパーティションテーブルを非パーティションテーブルに変更したり、非パーティションテーブルをパーティションテーブルに変更したり、パーティションタイプを変更することをサポートします。したがって、パーティションテーブルのパーティションタイプと個数を柔軟に調整できます。たとえば、`ALTER TABLE t PARTITION BY ...` 文を使用してパーティションタイプを変更できます。

詳細については[ドキュメント](/partitioned-table.md#convert-a-partitioned-table-to-a-non-partitioned-table)を参照してください。

* TiDBは`ROLLUP`修飾子と`GROUPING`関数の使用をサポートしています[#44487](https://github.com/pingcap/tidb/issues/44487) @[AilinKid](https://github.com/AilinKid)

`WITH ROLLUP`修飾子と`GROUPING`関数は、多次元データの集計において一般的に使用されます。v7.4.0から、`WITH ROLLUP`修飾子と`GROUPING`関数を`GROUP BY`句で使用できます。たとえば、`SELECT ... FROM ... GROUP BY ... WITH ROLLUP`のように`WITH ROLLUP`修飾子を使用できます。

詳細については[ドキュメント](/functions-and-operators/group-by-modifier.md)を参照してください。

### DB操作

* 照合`utf8mb4_0900_ai_ci`と`utf8mb4_0900_bin`をサポート[#37566](https://github.com/pingcap/tidb/issues/37566) @[YangKeao](https://github.com/YangKeao) @[zimulala](https://github.com/zimulala) @[bb7133](https://github.com/bb7133)

TiDB v7.4.0では、MySQL 8.0からのデータ移行を強化し、`utf8mb4_0900_ai_ci`および`utf8mb4_0900_bin`の2つの照合を追加しました。`utf8mb4_0900_ai_ci`はMySQL 8.0のデフォルトの照合です。

TiDB v7.4.0では、MySQL 8.0と互換性のある`default_collation_for_utf8mb4`というシステム変数が導入されました。これにより、utf8mb4文字セットのデフォルトの照合を指定し、MySQL 5.7以前のバージョンからの移行やデータレプリケーションとの互換性が提供されます。

詳細については[ドキュメント](/character-set-and-collation.md#character-sets-and-collations-supported-by-tidb)を参照してください。

### オブザーバビリティ
* ログにセッション接続 ID とセッション別名を追加するサポート [#46071](https://github.com/pingcap/tidb/issues/46071) @[lcwangchao](https://github.com/lcwangchao)

     SQL 実行の問題をトラブルシューティングする際に、TiDB コンポーネントのログの内容を関連付けて原因を特定することがしばしば必要です。v7.4.0 から TiDB は、セッション接続 ID (`CONNECTION_ID`) を TiDB ログ、スロークエリログ、および TiKV 上のコプロセッサのスローログなど、セッションに関連するログに書き込むことができます。セッション接続 ID を基に、複数の種類のログの内容を関連付けてトラブルシューティングや診断の効率を向上させることができます。

     また、セッションレベルのシステム変数 [`tidb_session_alias`](/system-variables.md#tidb_session_alias-new-in-v740) を設定することで、上記のログにカスタム識別子を追加することができます。この機能を使用して、アプリケーションの識別情報をログに注入することで、ログの内容をアプリケーションと関連付け、アプリケーションからログへのリンクを構築し、診断の難易度を低減させることができます。

* TiDB ダッシュボードはテーブルビューで実行計画を表示するサポート [#1589](https://github.com/pingcap/tidb-dashboard/issues/1589) @[baurine](https://github.com/baurine)

    v7.4.0 から TiDB ダッシュボードは、**遅いクエリ** と **SQL ステートメント** ページで実行計画をテーブルビューで表示して、診断体験を向上させます。

    詳細は [ドキュメント](/dashboard/dashboard-statement-details.md) を参照してください。

### データ移行

* `IMPORT INTO` 機能を強化する [#46704](https://github.com/pingcap/tidb/issues/46704) @[D3Hunter](https://github.com/D3Hunter)

    v7.4.0 から、`IMPORT INTO` ステートメントに `CLOUD_STORAGE_URI` オプションを追加して [Global Sort](/tidb-global-sort.md) 機能 (実験的) を有効にすることができます。これによりインポートのパフォーマンスと安定性が向上します。`CLOUD_STORAGE_URI` オプションでは、エンコードされたデータのためのクラウドストレージアドレスを指定できます。

    さらに、v7.4.0 では、`IMPORT INTO` 機能が以下の機能を導入しました：

    - `Split_File` オプションを構成するサポート。これにより、大きな CSV ファイルを複数の 256 MiB の小さな CSV ファイルに分割して並列処理を行い、インポートのパフォーマンスを向上させることができます。
    - 圧縮された CSV ファイルや SQL ファイルのインポートをサポートする機能。対応する圧縮形式には `.gzip`, `.gz`, `.zstd`, `.zst`, `.snappy` が含まれます。

    詳細は [ドキュメント](/sql-statements/sql-statement-import-into.md) を参照してください。

* Dumpling は、CSV ファイルへのデータエクスポート時にユーザー定義の終端文字列をサポートする [#46982](https://github.com/pingcap/tidb/issues/46982) @[GMHDBJD](https://github.com/GMHDBJD)

    v7.4.0 より前は、Dumpling は CSV ファイルへのデータエクスポート時に行終端文字を `"\r\n"` として使用していました。その結果、`"\n"` のみを終端文字として認識する特定の下流システムは、エクスポートされた CSV ファイルを解析できず、またはファイルを解析する前にサードパーティツールを使用しなければならない状況が生じました。

    v7.4.0 からは、Dumpling が新しいパラメータ `--csv-line-terminator` を導入しました。このパラメータを使用して、CSV ファイルへのデータエクスポート時に希望の終端文字列を指定できます。このパラメータは `"\r\n"` および `"\n"` をサポートします。既定の終端文字列は v7.4.0 より前のバージョンと一貫性を保つために `"\r\n"` です。

    詳細は [ドキュメント](/dumpling-overview.md#option-list-of-dumpling) を参照してください。

* TiCDC は、Pulsar へのデータレプリケーションをサポート [#9413](https://github.com/pingcap/tiflow/issues/9413) @[yumchina](https://github.com/yumchina) @[asddongmen](https://github.com/asddongmen)

    Pulsar はクラウドネイティブで分散メッセージストリーミングプラットフォームであり、リアルタイムのデータストリーミング体験を大幅に向上させます。v7.4.0 から TiCDC は、Pulsar への `canal-json` フォーマットの変更データレプリケーションをサポートし、Pulsar とのシームレスな統合を実現します。この機能により、TiCDC は TiDB の変更データを簡単にキャプチャし、Pulsar にレプリケートする能力を提供し、データ処理および分析機能の新たな可能性を提供します。Pulsar から新たに生成された変更データを読み取り、処理するための独自のコンシューマアプリケーションを開発することができます。

    詳細は [ドキュメント](/ticdc/ticdc-sink-to-pulsar.md) を参照してください。

- TiCDC は Claim-Check パターンで大容量メッセージを処理する機能を改良しました [#9153](https://github.com/pingcap/tiflow/issues/9153) @[3AceShowHand](https://github.com/3AceShowHand)

    v7.4.0 より前は、TiCDC は Kafka の最大メッセージサイズ (`max.message.bytes`) を超える大容量メッセージを下流に送信できませんでした。v7.4.0 からは、Kafka を下流に指定して changefeed を構成する際に、大容量メッセージを保存する外部ストレージ場所を指定し、外部ストレージアドレスを含む参照メッセージを Kafka に送信することができます。コンシューマがこの参照メッセージを受信するとき、外部ストレージアドレスからメッセージ内容を取得することができます。

    詳細は [ドキュメント](/ticdc/ticdc-sink-to-kafka.md#send-large-messages-to-external-storage) を参照してください。

## 互換性の変更

> **注記：**
>
> このセクションは、現在のバージョン (v7.4.0) からのアップグレード時に注意が必要な互換性の変更を提供します。v7.2.0 やそれ以前のバージョンからのアップグレードを行う場合、中間バージョンで導入された互換性の変更もチェックする必要があります。

### 動作の変更

- v7.4.0 以降、TiDB は MySQL 8.0 の基本的な機能と互換性があり、`version()` は `8.0.11` で開始されます。

- TiFlash を以前のバージョンから v7.4.0 にアップグレードした後、元のバージョンに inplace ダウングレードすることはサポートされなくなります。これは、v7.4 以降、TiFlash が PageStorage V3 のデータ圧縮ロジックを最適化し、データ圧縮中に生成される読み取りおよび書き込みの増幅を減らす変更が行われたためです。

- [`TIDB_PARSE_TSO_LOGICAL()`](/functions-and-operators/tidb-functions.md#tidb-specific-functions) 関数が追加され、TSO タイムスタンプの論理部分を抽出することができます。

- `information_schema.CHECK_CONSTRAINTS` テーブルが追加され、MySQL 8.0 との互換性が向上しました。

### システム変数

| 変数名 | 変更のタイプ | 説明 |
|--------|------------------------------|------|
| `tidb_enable_tiflash_pipeline_model` | 削除済み | この変数は TiFlash リソース制御機能が有効になっている場合に TiFlash パイプライン実行モデルを有効にするかどうかを制御するために使用されていました。v7.4.0 からは、TiFlash パイプライン実行モデルは自動的に有効になります。 |
| [`tidb_enable_non_prepared_plan_cache`](/system-variables.md#tidb_enable_non_prepared_plan_cache) | 変更済み | 追加のテストを経て、デフォルトの値を `ON` から `OFF` に変更しました。つまり、非プリペアド実行プランキャッシュが無効になります。 |
| [`default_collation_for_utf8mb4`](/system-variables.md#default_collation_for_utf8mb4-new-in-v740) | 新規追加 | `utf8mb4` 文字セットのデフォルト照合順序を制御します。デフォルト値は `utf8mb4_bin` です。 |
| [`tidb_cloud_storage_uri`](/system-variables.md#tidb_cloud_storage_uri-new-in-v740) | 新規追加 | [Global Sort](/tidb-global-sort.md) を有効にするためのクラウドストレージ URI を指定します。 |
| [`tidb_opt_enable_hash_join`](/system-variables.md#tidb_opt_enable_hash_join-new-in-v740) | 新規追加 | オプティマイザがテーブルのハッシュ結合を選択するかどうかを制御します。デフォルト値は `ON` です。`OFF` に設定すると、オプティマイザは他に実行プランが利用できない場合以外はテーブルのハッシュ結合を避けます。 |
| [`tidb_opt_objective`](/system-variables.md#tidb_opt_objective-new-in-v740) | 新規追加 | この変数はオプティマイザの目的を制御します。`moderate` は、TiDB v7.4.0 より前のバージョンと同様のデフォルト動作を維持し、オプティマイザはより良い実行プランを生成するためにより多くの情報を使用しようとします。`determinate` モードはより控えめになり、実行プランをより安定させます。 |
| [`tidb_schema_version_cache_limit`](/system-variables.md#tidb_schema_version_cache_limit-new-in-v740) | 新規追加 | この変数は TiDB インスタンスにキャッシュできる過去のスキーマバージョン数を制限します。デフォルト値は `16` であり、TiDB は通常 16 個の過去のスキーマバージョンをキャッシュします。 |
| [`tidb_service_scope`](/system-variables.md#tidb_service_scope-new-in-v740) | 追加 | この変数は、インスタンスレベルのシステム変数です。これを使用して、[TiDB分散実行フレームワーク](/tidb-distributed-execution-framework.md)の下でTiDBノードのサービススコープを制御できます。 TiDBノードの`tidb_service_scope`を`background`に設定すると、TiDB分散実行フレームワークはそのTiDBノードをバックグラウンドタスク（[`ADD INDEX`](/sql-statements/sql-statement-add-index.md)および[`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)など）を実行するようスケジュールします。 |
| [`tidb_session_alias`](/system-variables.md#tidb_session_alias-new-in-v740) | 追加 | 現在のセッションに関連するログの`session_alias`列の値を制御します。 |
| [`tiflash_mem_quota_query_per_node`](/system-variables.md#tiflash_mem_quota_query_per_node-new-in-v740) | 追加 | TiFlashノード上のクエリの最大メモリ使用量を制限します。クエリのメモリ使用量がこの制限を超えると、TiFlashはエラーを返し、クエリを終了します。デフォルト値は`0`で、制限はありません。|
| [`tiflash_query_spill_ratio`](/system-variables.md#tiflash_query_spill_ratio-new-in-v740) | 追加 | TiFlashの[クエリレベルのスピリング](/tiflash/tiflash-spill-disk.md#query-level-spilling)の閾値を制御します。デフォルト値は`0.7`です。 |
| [`tikv_client_read_timeout`](/system-variables.md#tikv_client_read_timeout-new-in-v740) | 追加 | TiDBがクエリでTiKV RPC読み取りリクエストを送信するためのタイムアウトを制御します。デフォルト値`0`は、通常40秒のデフォルトタイムアウトが使用されることを示します。 |

### 設定ファイルパラメータ

| 設定ファイル | 設定パラメータ | 変更タイプ | 説明 |
| -------- | -------- | -------- | -------- |
| TiDB | [`enable-stats-cache-mem-quota`](/tidb-configuration-file.md#enable-stats-cache-mem-quota-new-in-v610) | 変更 | デフォルト値が`false`から`true`に変更され、TiDB統計値のキャッシュのメモリ制限がデフォルトで有効になります。 |
| TiKV | [<code>rocksdb.\[defaultcf\|writecf\|lockcf\].periodic-compaction-seconds</code>](/tikv-configuration-file.md#periodic-compaction-seconds-new-in-v720) | 変更 | デフォルト値が`"30d"`から`"0s"`に変更され、RocksDBの定期コンパクションがデフォルトで無効化されます。 この変更により、TiDBのアップグレード後に大量のコンパクションがトリガされ、フロントエンドの読み取りおよび書き込みパフォーマンスに影響を与えることが防がれます。 |
| TiKV | [<code>rocksdb.\[defaultcf\|writecf\|lockcf\].ttl</code>](/tikv-configuration-file.md#ttl-new-in-v720) | 変更 | デフォルト値が`"30d"`から`"0s"`に変更され、SSTファイルがデフォルトでTTLによってコンパクションをトリガしないようになり、フロントエンドの読み取りおよび書き込みパフォーマンスに影響を与えることが防がれます。 |
| TiFlash | [`flash.compact_log_min_gap`](/tiflash/tiflash-configuration.md) | 追加 | 現在のRaftステートマシンによって前進された`applied_index`と前回のディスクスパイリング時の`applied_index`の間のギャップが`compact_log_min_gap`を超えると、TiFlashはTiKVから`CompactLog`コマンドを実行し、データをディスクにスピルします。 |
| TiFlash | [`profiles.default.enable_resource_control`](/tiflash/tiflash-configuration.md) | 追加 | TiFlashリソース制御機能の有効化を制御します。 |
| TiFlash | [`storage.format_version`](/tiflash/tiflash-configuration.md) | 変更 | デフォルト値が`4`から`5`に変更されます。 新しいフォーマットは、より小さなファイルをマージすることで物理ファイルの数を減らすことができます。 |
| Dumpling  | [`--csv-line-terminator`](/dumpling-overview.md#option-list-of-dumpling) | 追加 | CSVファイルの所望の終端子を指定します。 このオプションは、`"\r\n"`および`"\n"`をサポートしています。 デフォルト値は`"\r\n"`で、これは以前のバージョンと一致しています。 |
| TiCDC | [`claim-check-storage-uri`](/ticdc/ticdc-sink-to-kafka.md#send-large-messages-to-external-storage) | 追加 | `large-message-handle-option`を`claim-check`に設定した場合、 `claim-check-storage-uri`を有効な外部ストレージアドレスに設定する必要があります。 そうしないと、チェンジフィードの作成がエラーになります。 |
| TiCDC | [`large-message-handle-compression`](/ticdc/ticdc-sink-to-kafka.md#ticdc-data-compression) | 追加 | エンコード中に圧縮を有効にするかどうかを制御します。 デフォルト値は空で、無効を意味します。 |
| TiCDC | [`large-message-handle-option`](/ticdc/ticdc-sink-to-kafka.md#send-large-messages-to-external-storage) | 変更 | この設定項目に`claim-check`という新しい値が追加されました。 これを`claim-check`に設定すると、TiCDC Kafka sinkはメッセージサイズが上限を超えると、この大きなメッセージのアドレスを含むメッセージを外部ストレージに送信し、Kafkaにメッセージを送信します。 |

## 廃止された機能

+ [Mydumper](https://docs.pingcap.com/tidb/v4.0/mydumper-overview) はv7.5.0で廃止され、そのほとんどの機能は[Dumpling](/dumpling-overview.md)に置き換えられました。 Mydumperの代わりにDumplingを使用することを強くお勧めします。
+ TiKV-importer はv7.5.0で廃止されます。 代替として[TiDB Lightningの物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode.md)を使用することを強くお勧めします。

## 改善点

+ TiDB

    - パーティションテーブルにおける`ANALYZE`操作のメモリ使用量とパフォーマンスを最適化しました [#47071](https://github.com/pingcap/tidb/issues/47071) [#47104](https://github.com/pingcap/tidb/issues/47104) @[hawkingrei](https://github.com/hawkingrei)
    - 統計情報のガベージコレクションのメモリ使用量とパフォーマンスを最適化しました [#31778](https://github.com/pingcap/tidb/issues/31778) @[winoros](https://github.com/winoros)
    - インデックスマージインターセクションの`limit`のパッシュダウンを最適化し、クエリパフォーマンスを向上させました [#46863](https://github.com/pingcap/tidb/issues/46863) @[AilinKid](https://github.com/AilinKid)
    - `IndexLookup`が多くのテーブル取得タスクを関与させる場合に誤ってフルテーブルスキャンを選択する可能性を最小限に抑えるために、コストモデルを改善しました [#45132](https://github.com/pingcap/tidb/issues/45132) @[qw4990](https://github.com/qw4990)
    - `join on unique keys`のクエリパフォーマンスを向上させるために、ジョイン排除ルールを最適化しました [#46248](https://github.com/pingcap/tidb/issues/46248) @[fixdb](https://github.com/fixdb)
    - `バイナリ`に多値インデックス列の照合を変更し、実行の失敗を避けます [#46717](https://github.com/pingcap/tidb/issues/46717) @[YangKeao](https://github.com/YangKeao)

+ TiKV

    - OOMを防ぐためにResolverのメモリ使用量を最適化しました [#15458](https://github.com/tikv/tikv/issues/15458) @[overvenus](https://github.com/overvenus)
    - ルーターオブジェクトのLRUCacheを廃止し、メモリ使用量を減らし、OOMを防ぎました [#15430](https://github.com/tikv/tikv/issues/15430) @[Connor1996](https://github.com/Connor1996)
    - TiCDC Resolverのメモリ使用量を減らしました [#15412](https://github.com/tikv/tikv/issues/15412) @[overvenus](https://github.com/overvenus)
    - RocksDBコンパクションによって引き起こされるメモリの揺らぎを減らしました [#15324](https://github.com/tikv/tikv/issues/15324) @[overvenus](https://github.com/overvenus)
    - Partitioned Raft KVの流量制御モジュールで引き起こされるメモリ消費を減らしました [#15269](https://github.com/tikv/tikv/issues/15269) @[overvenus](https://github.com/overvenus)
    - PDクライアントの接続リトライのプロセスで、エラーリトライ中にリトライ間隔を徐々に増やし、PDへの負荷を減らすためにバックオフメカニズムを追加しました [#15428](https://github.com/tikv/tikv/issues/15428) @[nolouch](https://github.com/nolouch)
    - RocksDBの`background_compaction`の動的調整をサポートしました [#15424](https://github.com/tikv/tikv/issues/15424) @[glorv](https://github.com/glorv)

+ PD
- TSOトレーシング情報の最適化により、TSO関連の問題の調査が容易になりました [#6856](https://github.com/tikv/pd/pull/6856) @[tiancaiamao](https://github.com/tiancaiamao)
    - メモリ使用量を減らすためにHTTPクライアント接続を再利用するサポートを追加しました [#6913](https://github.com/tikv/pd/issues/6913) @[nolouch](https://github.com/nolouch)
    - バックアップクラスタが切断されたときに、PDがクラスタ状態を自動更新するスピードを改善しました [#6883](https://github.com/tikv/pd/issues/6883) @[disksing](https://github.com/disksing)
    - リソースコントロールクライアントの構成取得方法を強化し、最新の構成を動的に取得するようにしました [#7043](https://github.com/tikv/pd/issues/7043) @[nolouch](https://github.com/nolouch)

+ TiFlash

    - TiFlashの書き込みパフォーマンスを向上させ、TiFlash書き込みプロセスのスピリングポリシーを最適化することでランダム書き込みワークロードを改善しました [#7564](https://github.com/pingcap/tiflash/issues/7564) @[CalvinNeo](https://github.com/CalvinNeo)
    - TiFlashのRaftレプリケーションプロセスに関する追加のメトリクスを追加しました [#8068](https://github.com/pingcap/tiflash/issues/8068) @[CalvinNeo](https://github.com/CalvinNeo)
    - ファイルシステムinodeの枯渇を避けるため、小さなファイルの数を減らしました [#7595](https://github.com/pingcap/tiflash/issues/7595) @[hongyunyan](https://github.com/hongyunyan)

+ ツール

    + バックアップとリストア（BR）

        - Regionリーダーシップの移行時にPITRログバックアップの進行の遅延が増加する問題を軽減しました [#13638](https://github.com/tikv/tikv/issues/13638) @[YuJuncen](https://github.com/YuJuncen)
        - `MaxIdleConns`および`MaxIdleConnsPerHost`パラメータを設定することで、ログバックアップおよびPITRリストアタスクの接続再利用を強化しました [#46011](https://github.com/pingcap/tidb/issues/46011) @[Leavrth](https://github.com/Leavrth)
        - PDまたは外部S3ストレージに接続できない場合のBRの耐障害性を向上しました [#42909](https://github.com/pingcap/tidb/issues/42909) @[Leavrth](https://github.com/Leavrth)
        - 新しいリストアパラメータ`WaitTiflashReady`を追加しました。このパラメータが有効になると、リストア操作はTiFlashレプリカが正常にレプリケートされた後に完了します [#43828](https://github.com/pingcap/tidb/issues/43828) [#46302](https://github.com/pingcap/tidb/issues/46302) @[3pointer](https://github.com/3pointer)
        - ログバックアップのCPUオーバーヘッドを減らしました `resolve lock` [#40759](https://github.com/pingcap/tidb/issues/40759) @[3pointer](https://github.com/3pointer)

    + TiCDC

        - `ADD INDEX`のDDL操作をレプリケートする実行ロジックを最適化し、後続のDMLステートメントをブロックしないようにしました [#9644](https://github.com/pingcap/tiflow/issues/9644) @[sdojjy](https://github.com/sdojjy)

    + TiDB Lightning

        - リージョン散布フェーズ中のTiDB Lightningのリトライロジックを最適化しました [#46203](https://github.com/pingcap/tidb/issues/46203) @[mittalrishabh](https://github.com/mittalrishabh)
        - データインポートフェーズ中の`no leader`エラーのためのTiDB Lightningのリトライロジックを最適化しました [#46253](https://github.com/pingcap/tidb/issues/46253) @[lance6716](https://github.com/lance6716)

## バグ修正

+ TiDB

    - `BatchPointGet`オペレーターがハッシュパーティションされていないテーブルに対して不正な結果を返す問題を修正しました [#45889](https://github.com/pingcap/tidb/issues/45889) @[Defined2014](https://github.com/Defined2014)
    - ハッシュパーティションされたテーブルに対して`BatchPointGet`オペレーターが不正な結果を返す問題を修正しました [#46779](https://github.com/pingcap/tidb/issues/46779) @[jiyfhust](https://github.com/jiyfhust)
    - TiDBパーサーが一貫した状態で残り、解析エラーを引き起こす問題を修正しました [#45898](https://github.com/pingcap/tidb/issues/45898) @[qw4990](https://github.com/qw4990)
    - `EXCHANGE PARTITION`が制約をチェックしない問題を修正しました [#45922](https://github.com/pingcap/tidb/issues/45922) @[mjonss](https://github.com/mjonss)
    - `tidb_enforce_mpp`システム変数が正しく復元されない問題を修正しました [#46214](https://github.com/pingcap/tidb/issues/46214) @[djshow832](https://github.com/djshow832)
    - `LIKE`句の`_`が誤って処理される問題を修正しました [#46287](https://github.com/pingcap/tidb/issues/46287) [#46618](https://github.com/pingcap/tidb/issues/46618) @[Defined2014](https://github.com/Defined2014)
    - TiDBがスキーマを取得できなかったときに`schemaTs`が0に設定される問題を修正しました [#46325](https://github.com/pingcap/tidb/issues/46325) @[hihihuhu](https://github.com/hihihuhu)
    - `AUTO_ID_CACHE=1`が設定されている場合に`Duplicate entry`が発生する問題を修正しました [#46444](https://github.com/pingcap/tidb/issues/46444) @[tiancaiamao](https://github.com/tiancaiamao)
    - `AUTO_ID_CACHE=1`が設定されている場合にTiDBがパニック後に遅く回復する問題を修正しました [#46454](https://github.com/pingcap/tidb/issues/46454) @[tiancaiamao](https://github.com/tiancaiamao)
    - `AUTO_ID_CACHE=1`が設定されている場合に`SHOW CREATE TABLE`での`next_row_id`が不正である問題を修正しました [#46545](https://github.com/pingcap/tidb/issues/46545) @[tiancaiamao](https://github.com/tiancaiamao)
    - サブクエリでCTEを使用する際に解析中にパニックが発生する問題を修正しました [#45838](https://github.com/pingcap/tidb/issues/45838) @[djshow832](https://github.com/djshow832)
    - `EXCHANGE PARTITION`が失敗またはキャンセルされた場合にリストパーティションの制限が元のテーブルに残る問題を修正しました [#45920](https://github.com/pingcap/tidb/issues/45920) [#45791](https://github.com/pingcap/tidb/issues/45791) @[mjonss](https://github.com/mjonss)
    - リストパーティションの定義が`NULL`と空の文字列の両方を使用することをサポートしない問題を修正しました [#45694](https://github.com/pingcap/tidb/issues/45694) @[mjonss](https://github.com/mjonss)
    - パーティション交換中にパーティション定義に準拠しないデータを検出できない問題を修正しました [#46492](https://github.com/pingcap/tidb/issues/46492) @[mjonss](https://github.com/mjonss)
    - `tmp-storage-quota`の構成が反映されない問題を修正しました [#45161](https://github.com/pingcap/tidb/issues/45161) [#26806](https://github.com/pingcap/tidb/issues/26806) @[wshwsh12](https://github.com/wshwsh12)
    - `WEIGHT_STRING()`関数が照合と一致しない問題を修正しました [#45725](https://github.com/pingcap/tidb/issues/45725) @[dveeden](https://github.com/dveeden)
    - インデックス結合でのエラーがクエリがスタックする原因となる問題を修正しました [#45716](https://github.com/pingcap/tidb/issues/45716) @[wshwsh12](https://github.com/wshwsh12)
    - `DATETIME`または`TIMESTAMP`列と数値定数を比較する際にMySQLとの挙動が一貫していない問題を修正しました [#38361](https://github.com/pingcap/tidb/issues/38361) @[yibin87](https://github.com/yibin87)
    - 符号なしの型と`Duration`型の定数を比較した際に発生する不正な結果を修正しました [#45410](https://github.com/pingcap/tidb/issues/45410) @[wshwsh12](https://github.com/wshwsh12)
- アクセスパスの剪定ロジックが`READ_FROM_STORAGE(TIFLASH[...])`ヒントを無視する問題を修正し、`Can't find a proper physical plan`エラーが発生する問題を修正 [#40146](https://github.com/pingcap/tidb/issues/40146) @[AilinKid](https://github.com/AilinKid)
    - `GROUP_CONCAT`が`ORDER BY`カラムを解析できない問題を修正 [#41986](https://github.com/pingcap/tidb/issues/41986) @[AilinKid](https://github.com/AilinKid)
    - ネストが深い式でHashCodeが繰り返し計算され、高いメモリ使用量とOOMを引き起こす問題を修正 [#42788](https://github.com/pingcap/tidb/issues/42788) @[AilinKid](https://github.com/AilinKid)
    - CASTが精度の損失がない場合に、`cast(col)=range`条件がCASTによるフルスキャンを引き起こす問題を修正 [#45199](https://github.com/pingcap/tidb/issues/45199) @[AilinKid](https://github.com/AilinKid)
    - MPP実行プランでUnion経由でAggregationが押し下げられた場合に、結果が正しくない問題を修正 [#45850](https://github.com/pingcap/tidb/issues/45850) @[AilinKid](https://github.com/AilinKid)
    - `in (?)`に一致しないバインディングが`in (?, ... ?)`に一致しない問題を修正 [#44298](https://github.com/pingcap/tidb/issues/44298) @[qw4990](https://github.com/qw4990)
    - `non-prep plan cache`が実行計画を再利用する際に接続照合を考慮していないために発生するエラーを修正 [#47008](https://github.com/pingcap/tidb/issues/47008) @[qw4990](https://github.com/qw4990)
    - 実行された計画が計画キャッシュにヒットしない場合に警告が報告されない問題を修正 [#46159](https://github.com/pingcap/tidb/issues/46159) @[qw4990](https://github.com/qw4990)
    - `plan replayer dump explain`がエラーを報告する問題を修正 [#46197](https://github.com/pingcap/tidb/issues/46197) @[time-and-fate](https://github.com/time-and-fate)
    - CTEを使用したDMLステートメントの実行がパニックを引き起こす問題を修正 [#46083](https://github.com/pingcap/tidb/issues/46083) @[winoros](https://github.com/winoros)
    - `TIDB_INLJ`ヒントが2つのサブクエリを結合する際に効果を持たない問題を修正 [#46160](https://github.com/pingcap/tidb/issues/46160) @[qw4990](https://github.com/qw4990)
    - `MERGE_JOIN`の結果が正しくない問題を修正 [#46580](https://github.com/pingcap/tidb/issues/46580) @[qw4990](https://github.com/qw4990)

+ TiKV

    - Titanが有効になっているとTiKVの起動に失敗し、`Blob file deleted twice`エラーが発生する問題を修正 [#15454](https://github.com/tikv/tikv/issues/15454) @[Connor1996](https://github.com/Connor1996)
    - Thread VoluntaryとThread Nonvoluntaryの監視パネルにデータがない問題を修正 [#15413](https://github.com/tikv/tikv/issues/15413) @[SpadeA-Tang](https://github.com/SpadeA-Tang)
    - 連続して増加するraftstore-applysのデータエラーを修正 [#15371](https://github.com/tikv/tikv/issues/15371) @[Connor1996](https://github.com/Connor1996)
    - 不正確なRegionのメタデータによるTiKVのパニックの問題を修正 [#13311](https://github.com/tikv/tikv/issues/13311) @[zyguan](https://github.com/zyguan)
    - `sync_recovery`から`sync`に切り替えた後、QPSが0に下がる問題を修正 [#15366](https://github.com/tikv/tikv/issues/15366) @[nolouch](https://github.com/nolouch)
    - オンライン不安定リカバリがタイムアウトしても中止しない問題を修正 [#15346](https://github.com/tikv/tikv/issues/15346) @[Connor1996](https://github.com/Connor1996)
    - CpuRecordによる潜在的なメモリリークの問題を修正 [#15304](https://github.com/tikv/tikv/issues/15304) @[overvenus](https://github.com/overvenus)
    - バックアップクラスタがダウンしてプライマリクラスタがクエリされた際に、`"Error 9002: TiKV server timeout"`が発生する問題を修正 [#12914](https://github.com/tikv/tikv/issues/12914) @[Connor1996](https://github.com/Connor1996)
    - プライマリクラスタが回復した後にTiKVが再起動すると、バックアップTiKVがスタックする問題を修正 [#12320](https://github.com/tikv/tikv/issues/12320) @[disksing](https://github.com/disksing)

+ PD

    - Flashback中にRegion情報が更新および保存されない問題を修正 [#6912](https://github.com/tikv/pd/issues/6912) @[overvenus](https://github.com/overvenus)
    - ストア設定の同期が遅れるためPD Leadersの切り替えが遅い問題を修正 [#6918](https://github.com/tikv/pd/issues/6918) @[bufferflies](https://github.com/bufferflies)
    - グループがScatter Peersで考慮されていない問題を修正 [#6962](https://github.com/tikv/pd/issues/6962) @[bufferflies](https://github.com/bufferflies)
    - RU消費が0未満の場合にPDがクラッシュする問題を修正 [#6973](https://github.com/tikv/pd/issues/6973) @[CabinfeverB](https://github.com/CabinfeverB)
    - 変更された分離レベルがデフォルトの配置ルールに同期されていない問題を修正 [#7121](https://github.com/tikv/pd/issues/7121) @[rleungx](https://github.com/rleungx)
    - クライアントが定期的に`min-resolved-ts`を更新することによってPDが大規模なクラスタではOOMを引き起こす問題を修正 [#46664](https://github.com/pingcap/tidb/issues/46664) @[HuSharp](https://github.com/HuSharp)

+ TiFlash

    - Grafanaで`max_snapshot_lifetime`メトリックが正しく表示されない問題を修正 [#7713](https://github.com/pingcap/tiflash/issues/7713) @[JaySon-Huang](https://github.com/JaySon-Huang)
    - 最大期間に関する一部のメトリックが正しくない問題を修正 [#8076](https://github.com/pingcap/tiflash/issues/8076) @[CalvinNeo](https://github.com/CalvinNeo)
    - TiDBが誤ってMPPタスクの失敗を報告する問題を修正 [#7177](https://github.com/pingcap/tiflash/issues/7177) @[yibin87](https://github.com/yibin87)

+ Tools

    + バックアップ＆リストア（BR）

        - バックアップが失敗した際に実際のエラーを隠す誤ったエラーメッセージ`resolve lock timeout`を修正 [#43236](https://github.com/pingcap/tidb/issues/43236) @[YuJuncen](https://github.com/YuJuncen)
        - PITRを使用した暗黙のプライマリキーの復旧が競合を引き起こす可能性がある問題を修正 [#46520](https://github.com/pingcap/tidb/issues/46520) @[3pointer](https://github.com/3pointer)
        - PITRを使用したメタkvの復旧がエラーを引き起こす可能性がある問題を修正 [#46578](https://github.com/pingcap/tidb/issues/46578) @[Leavrth](https://github.com/Leavrth)
        - BR統合テストケースのエラーを修正 [#45561](https://github.com/pingcap/tidb/issues/46561) @[purelind](https://github.com/purelind)

    + TiCDC

        - TiCDCがPDのスケーリングアップとダウン中に無効な古いアドレスにアクセスする問題を修正 [#9584](https://github.com/pingcap/tiflow/issues/9584) @[fubinzh](https://github.com/fubinzh) @[asddongmen](https://github.com/asddongmen)
        - 一部のシナリオでchangefeedが失敗する問題を修正 [#9309](https://github.com/pingcap/tiflow/issues/9309) [#9450](https://github.com/pingcap/tiflow/issues/9450) [#9542](https://github.com/pingcap/tiflow/issues/9542) [#9685](https://github.com/pingcap/tiflow/issues/9685) @[hicqu](https://github.com/hicqu) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 上流で1つのトランザクションで複数の行のユニークキーが変更されると、レプリケーションの書き込み競合が発生する可能性がある問題を修正 [#9430](https://github.com/pingcap/tiflow/issues/9430) @[sdojjy](https://github.com/sdojjy)
        - 同一のDDLステートメントで複数のテーブルが名前変更されると、レプリケーションエラーが発生する問題を修正 [#9476](https://github.com/pingcap/tiflow/issues/9476) [#9488](https://github.com/pingcap/tiflow/issues/9488) @[CharlesCheung96](https://github.com/CharlesCheung96) @[asddongmen](https://github.com/asddongmen)
        - CSVファイルにおいて中国語文字が検証されない問題を修正 [#9609](https://github.com/pingcap/tiflow/issues/9609) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 全てのchangefeedが削除された後に上流のTiDB GCがブロックされる問題を修正 [#9633](https://github.com/pingcap/tiflow/issues/9633) @[sdojjy](https://github.com/sdojjy)
        - `scale-out`が有効になっている場合に、書き込みキーがノード間で均一に分散されない問題を修正 [#9665](https://github.com/pingcap/tiflow/issues/9665) @[sdojjy](https://github.com/sdojjy)
        - 機密なユーザー情報がログに記録される問題を修正 [#9690](https://github.com/pingcap/tiflow/issues/9690) @[sdojjy](https://github.com/sdojjy)

    + TiDBデータ移行（DM）

        - 大文字小文字を区別しない照合と競合を正しく処理できない問題を修正 [#9489](https://github.com/pingcap/tiflow/issues/9489) @[hihihuhu](https://github.com/hihihuhu)
        - DM検証プロセスのデッドロック問題を修正し、リトライを強化 [#9257](https://github.com/pingcap/tiflow/issues/9257) @[D3Hunter](https://github.com/D3Hunter)
        - 失敗したDDLがスキップされ、その後のDDLが実行されない時に、DMによって返されるレプリケーション遅延が増加し続ける問題を修正 [#9605](https://github.com/pingcap/tiflow/issues/9605) @[D3Hunter](https://github.com/D3Hunter)
        - オンラインDDLがスキップされる際に、DMが上流のテーブルスキーマを適切に追跡できない問題を修正 [#9587](https://github.com/pingcap/tiflow/issues/9587) @[GMHDBJD](https://github.com/GMHDBJD)
        - 楽観的モードでタスクを再開する際にすべてのDMLがスキップされる問題を修正 [#9588](https://github.com/pingcap/tiflow/issues/9588) @[GMHDBJD](https://github.com/GMHDBJD)
        - 楽観的モードでパーティションDDLがスキップされる問題を修正 [#9788](https://github.com/pingcap/tiflow/issues/9788) @[GMHDBJD](https://github.com/GMHDBJD)

    + TiDB Lightning

        - `NONCLUSTERED auto_increment`および`AUTO_ID_CACHE=1`テーブルをTiDB Lightningがインポートした後にデータの挿入がエラーを返す問題を修正 [#46100](https://github.com/pingcap/tidb/issues/46100) @[tiancaiamao](https://github.com/tiancaiamao)
        - `checksum = "optional"`の際にもチェックサムがエラーを報告する問題を修正 [#45382](https://github.com/pingcap/tidb/issues/45382) @[lyzx2001](https://github.com/lyzx2001)
        - PDクラスターアドレスが変更された際にデータのインポートが失敗する問題を修正 [#43436](https://github.com/pingcap/tidb/issues/43436) @[lichunzhu](https://github.com/lichunzhu)

## 貢献者

TiDBコミュニティの以下の貢献者に感謝します：

- [aidendou](https://github.com/aidendou)
- [coderplay](https://github.com/coderplay)
- [fatelei](https://github.com/fatelei)
- [highpon](https://github.com/highpon)
- [hihihuhu](https://github.com/hihihuhu)（初めての貢献者）
- [isabella0428](https://github.com/isabella0428)
- [jiyfhust](https://github.com/jiyfhust)
- [JK1Zhang](https://github.com/JK1Zhang)
- [joker53-1](https://github.com/joker53-1)（初めての貢献者）
- [L-maple](https://github.com/L-maple)
- [mittalrishabh](https://github.com/mittalrishabh)
- [paveyry](https://github.com/paveyry)
- [shawn0915](https://github.com/shawn0915)
- [tedyu](https://github.com/tedyu)
- [yumchina](https://github.com/yumchina)
- [ZhuohaoHe](https://github.com/ZhuohaoHe)