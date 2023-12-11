---
title: TiDB 6.6.0リリースノート
summary: TiDB 6.6.0の新機能、互換性の変更、改善、およびバグ修正について学びます。

# TiDB 6.6.0リリースノート

リリース日: 2023年2月20日

TiDBバージョン: 6.6.0-[DMR](/releases/versioning.md#development-milestone-releases)

> **Note:**
>
> TiDB 6.6.0-DMRのドキュメントは[アーカイブ](https://docs-archive.pingcap.com/tidb/v6.6/)されました。PingCAPでは、TiDBデータベースの[最新のLTSバージョン](https://docs.pingcap.com/tidb/stable)を使用するよう推奨しています。

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.6/quick-start-with-tidb) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.6.0#version-list)

v6.6.0-DMRでの主な新機能と改善点は以下のとおりです:

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
    <td rowspan="3">スケーラビリティとパフォーマンス<br /></td>
    <td>TiKVが<a href="https://docs.pingcap.com/tidb/v6.6/partitioned-raft-kv" target="_blank">パーティション化されたRaft KVストレージエンジン</a>をサポート (実験的)</td>
    <td>TiKVはパーティション化されたRaft KVストレージエンジンを導入し、各Regionが独立したRocksDBインスタンスを使用するようになりました。これにより、クラスタのストレージ容量をTBからPBまで簡単に拡張できるだけでなく、より安定した書き込みレイテンシと強力なスケーラビリティを提供できます。</td>
  </tr>
  <tr>
    <td>TiKVが<a href="https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_store_batch_size" target="_blank">バッチ集約データリクエスト</a>をサポート</td>
    <td>この改良により、TiKVのバッチ取得操作での合計RPCが大幅に削減されます。データが非常に分散している状況やgRPCスレッドプールのリソースが不足している状況では、コプロセッサリクエストのバッチ化により、パフォーマンスが50%以上向上する場合があります。</td>
  </tr>
  <tr>
    <td>TiFlashが<a href="https://docs.pingcap.com/tidb/v6.6/stale-read" target="_blank">Stale Read</a>および<a href="https://docs.pingcap.com/tidb/v6.6/explain-mpp#mpp-version-and-exchange-data-compression" target="_blank">圧縮データ交換</a>をサポート</td>
    <td>TiFlashは、ステールリード機能をサポートし、リアルタイムの要件に制約がないシナリオでのクエリパフォーマンスを向上させることができます。また、データの圧縮をサポートし、並列データ交換の効率を向上させ、全体的なTPC-Hパフォーマンスが10%向上し、ネットワークの利用率を50%以上節約できます。</td>
  </tr>
  <tr>
    <td rowspan="2">信頼性と可用性<br /></td>
    <td><a href="https://docs.pingcap.com/tidb/v6.6/tidb-resource-control" target="_blank">リソース制御</a> (実験的)</td>
    <td>リソースグループに基づいたリソース管理をサポートし、データベースユーザを対応するリソースグループにマッピングし、実際のニーズに基づいて各リソースグループにクォータを設定できます。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v6.6/sql-plan-management#create-a-binding-according-to-a-historical-execution-plan" target="_blank">過去のSQLバインディング</a>をサポート</td>
    <td>過去の実行計画をバインドし、TiDBダッシュボードで迅速に実行計画をバインドすることができます。</td>
  </tr>
  <tr>
    <td rowspan="2">SQL機能<br /></td>
    <td><a href="https://docs.pingcap.com/tidb/v6.6/foreign-key" target="_blank">外部キー</a> (実験的)</td>
    <td>データ整合性を維持し、データ品質を向上させるために、MySQL互換の外部キー制約をサポートします。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v6.6/sql-statement-create-index#multi-valued-indexes" target="_blank">多値インデックス</a> (実験的)</td>
    <td>MySQL互換の多値インデックスを導入し、JSON型を強化し、MySQL 8.0との互換性を向上させます。</td>
  </tr>
  <tr>
    <td>DBの操作と可観測性<br /></td>
    <td><a href="https://docs.pingcap.com/tidb/v6.6/dm-precheck#check-items-for-physical-import" target="_blank">DMが物理インポートをサポート</a> (実験的)</td>
    <td>TiDBデータ移行（DM）は、TiDB Lightningの物理インポートモードを統合し、フルデータ移行のパフォーマンスを最大10倍向上させます。</td>
  </tr>
</tbody>
</table>

## 機能の詳細

### スケーラビリティ

* パーティション化されたRaft KVストレージエンジン（実験的）のサポート [#11515](https://github.com/tikv/tikv/issues/11515) [#12842](https://github.com/tikv/tikv/issues/12842) @[busyjay](https://github.com/busyjay) @[tonyxuqqi](https://github.com/tonyxuqqi) @[tabokie](https://github.com/tabokie) @[bufferflies](https://github.com/bufferflies) @[5kbpers](https://github.com/5kbpers) @[SpadeA-Tang](https://github.com/SpadeA-Tang) @[nolouch](https://github.com/nolouch)

    TiDB v6.6.0以前では、TiKVのRaftベースのストレージエンジンは単一のRocksDBインスタンスを使用していました。TiDB v6.6.0からは、より大きなクラスタを安定してサポートするために新しいTiKVストレージエンジンが導入され、複数のRocksDBインスタンスを使用してTiKVリージョンデータを格納し、各リージョンのデータは独立したRocksDBインスタンスに独自に格納されます。新しいエンジンでは、RocksDBインスタンスにおけるファイルの数やレベルをより良く制御し、リージョン間のデータ操作を物理的に分離し、より多くのデータを安定して管理できます。この機能は、パーティション化されたRaft KVと見なすことができます。この機能の主な利点は、より優れた書き込みパフォーマンス、より速いスケーリング、および同じハードウェアでのより大きなデータ容量のサポートです。

    現時点では、この機能は実験的であり、本番環境での使用は推奨されていません。

    詳細は、[ドキュメント](/partitioned-raft-kv.md)を参照してください。

* DDL操作の分散並列実行フレームワークのサポート（実験的） [#37125](https://github.com/pingcap/tidb/issues/37125) @[zimulala](https://github.com/zimulala)

    以前のバージョンでは、TiDBクラスタ全体で、スキーマ変更タスクを単一のTiDBインスタンスがDDLオーナーとして処理することが許可されていました。大きなテーブルのDDL操作の並列実行をさらに改善するために、TiDB v6.6.0では、DDLの分散並列実行フレームワークが導入されました。これにより、クラスタ内のすべてのTiDBインスタンスが同じタスクの`StateWriteReorganization`フェーズを同時に実行できるようになり、DDL実行を高速化できます。この機能はシステム変数[`tidb_ddl_distribute_reorg`](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_ddl_distribute_reorg-new-in-v660)で制御され、現時点では`Add Index`操作のみをサポートしています。

### パフォーマンス

* 悲観的ロックキューの安定した起動モデルのサポート [#13298](https://github.com/tikv/tikv/issues/13298) @[MyonKeminta](https://github.com/MyonKeminta)

    アプリケーションが頻繁に単一ポイントの悲観的ロックの競合に遭遇した場合、既存の起動メカニズムではトランザクションがロックを取得するための時間を保証できず、高い長尾タイムラグやロック取得タイムアウトを引き起こします。v6.6.0からは、システム変数[`tidb_pessimistic_txn_aggressive_locking`](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_pessimistic_txn_aggressive_locking-new-in-v660)の値を`ON`に設定することで、悲観的ロックの安定した起動モデルを有効にできます。この起動モデルでは、キューの起動シーケンスを厳密に制御し、無効な起動によるリソースの浪費を避けることができます。深刻なロック競合が発生するシナリオでは、安定した起動モデルにより長いタイムラグが削減され、P99応答時間が改善されます。

    テストでは、これにより長いタイムラグが40-60%削減されることが示されています。

    詳細は、[ドキュメント](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_pessimistic_txn_aggressive_locking-new-in-v660)を参照してください。

* Batch aggregate data requests [#39361](https://github.com/pingcap/tidb/issues/39361) @[cfzjywxk](https://github.com/cfzjywxk) @[you06](https://github.com/you06)

    TiDBはTiKVにデータリクエストを送信する際、データが存在するRegionに応じてリクエストを異なるサブタスクにコンパイルし、各サブタスクは1つのRegionのリクエストのみを処理します。アクセスするデータが広く分散している場合、データのサイズが大きくなくても多くのサブタスクが生成され、それにより多くのRPCリクエストが生成され、余分な時間を消費します。v6.6.0から、TiDBは同じTiKVインスタンスに送信されるデータリクエストを部分的にマージする機能をサポートし、サブタスクの数とRPCリクエストのオーバーヘッドを削減します。データの極めて広い分散状態やgRPCスレッドプールリソースが不足している場合、リクエストのバッチ処理を行うことでパフォーマンスを50%以上向上させることが可能です。

    この機能はデフォルトで有効化されています。システム変数[`tidb_store_batch_size`](/system-variables.md#tidb_store_batch_size)を使用してリクエストのバッチサイズを設定できます。

* Remove the limit on `LIMIT` clauses [#40219](https://github.com/pingcap/tidb/issues/40219) @[fzzf678](https://github.com/fzzf678)

    v6.6.0から、TiDBのプランキャッシュは`LIMIT`パラメータとして変数を持つ実行計画をキャッシュする機能（`LIMIT ?`または`LIMIT 10, ?`など）をサポートします。この機能により、より多くのSQLステートメントがプランキャッシュの恩恵を受けることが可能となり、実行効率が向上します。現在、セキュリティ上の考慮から、TiDBは`?`が10000を超えない実行計画のキャッシュのみをサポートしています。

    詳細については[ドキュメント](/sql-prepared-plan-cache.md)を参照してください。

* TiFlash supports data exchange with compression [#6620](https://github.com/pingcap/tiflash/issues/6620) @[solotzg](https://github.com/solotzg)

    複数のノードで計算を行う際、TiFlashエンジンは異なるノード間でデータの交換が必要です。交換するデータのサイズが非常に大きい場合、データ交換のパフォーマンスは全体の計算効率に影響を及ぼす可能性があります。v6.6.0では、TiFlashエンジンは必要な場合にデータを圧縮して交換するための圧縮メカニズムを導入し、それによりデータ交換の効率を向上させます。

    詳細については[ドキュメント](/explain-mpp.md#mpp-version-and-exchange-data-compression)を参照してください。

* TiFlash supports the Stale Read feature [#4483](https://github.com/pingcap/tiflash/issues/4483) @[hehechen](https://github.com/hehechen)

    Stale Read機能はv5.1.1以降一般提供（GA）されており、特定のタイムスタンプまたは指定された時間範囲内の歴史データを読み取ることができます。Stale ReadはローカルのTiKVレプリカからデータを直接読み取ることで読み取り遅延を減少させ、クエリのパフォーマンスを向上させることができます。v6.6.0以前では、TiFlashはStale Readをサポートしていませんでした。TiFlashレプリカを持つテーブルでもStale ReadはそのTiKVレプリカからのみ読み取ることができました。

    v6.6.0から、TiFlashはStale Read機能をサポートします。[`AS OF TIMESTAMP`](/as-of-timestamp.md)構文や[`tidb_read_staleness`](/tidb-read-staleness.md)システム変数を使用してテーブルの歴史データをクエリする場合、テーブルがTiFlashレプリカを持っている場合、オプティマイザは対応するデータをTiFlashレプリカから読み取ることを選択することができ、クエリのパフォーマンスをさらに向上させることができます。

    詳細については[ドキュメント](/stale-read.md)を参照してください。

* Support pushing down the `regexp_replace` string function to TiFlash [#6115](https://github.com/pingcap/tiflash/issues/6115) @[xzhangxian1008](https://github.com/xzhangxian1008)

### リアビリティ

* Support resource control based on resource groups (experimental) [#38825](https://github.com/pingcap/tidb/issues/38825) @[nolouch](https://github.com/nolouch) @[BornChanger](https://github.com/BornChanger) @[glorv](https://github.com/glorv) @[tiancaiamao](https://github.com/tiancaiamao) @[Connor1996](https://github.com/Connor1996) @[JmPotato](https://github.com/JmPotato) @[hnes](https://github.com/hnes) @[CabinfeverB](https://github.com/CabinfeverB) @[HuSharp](https://github.com/HuSharp)

    今やTiDBクラスターに対してリソースグループを作成し、異なるデータベースユーザーを対応するリソースグループにバインドし、実際の必要性に応じて各リソースグループにクォータを設定できます。クラスターのリソースが限られている場合、同じリソースグループ内のセッションによって使用されるすべてのリソースはクォータに制限されます。これにより、リソースグループが過度に使用されても他のリソースグループのセッションには影響がありません。TiDBはGrafanaダッシュボードで実際のリソース使用状況のビューを提供するため、リソースの合理的な割り当てを支援します。

    リソース制御機能の導入はTiDBにとって画期的なものであり、分散データベースクラスターを複数の論理単位に分割できます。個々のユニットがリソースを過度に使用しても、他のユニットが必要なリソースを余儀なくされることはありません。

    この機能を使用することで、以下のことが可能です。

    - 異なるシステムからの複数の小規模および中規模のアプリケーションを1つのTiDBクラスターに統合できます。アプリケーションのワークロードが大きくなっても他のアプリケーションの正常な運用に影響を与えません。システムワークロードが低い場合、必要なシステムリソースを繁忙なアプリケーションに割り当てることができます。これにより、リソースの最大の利用が実現されます。
    - すべてのテスト環境を1つのTiDBクラスターに統合するか、より多くのリソースを消費するバッチタスクを1つのリソースグループにグループ化することができます。これにより、ハードウェアの利用率が向上し、運用コストが低減される一方で、重要なアプリケーションが常に必要なリソースを確保できるようになります。

  さらに、リソース制御機能の合理的な使用により、クラスターの数が削減され、運用と管理の難しさが軽減され、管理コストが削減されます。

  v6.6では、リソース制御を有効にするにはTiDBのグローバル変数[`tidb_enable_resource_control`](/system-variables.md#tidb_enable_resource_control-new-in-v660)とTiKVの構成項目[`resource-control.enabled`](/tikv-configuration-file.md#resource-control)の両方を有効にする必要があります。現在、サポートされているクォータ方法は"[Request Unit (RU)](/tidb-resource-control.md#what-is-request-unit-ru)"に基づいています。RUはTiDBのCPUやIOなどのシステムリソースの統一された抽象化単位です。

  詳細については[ドキュメント](/tidb-resource-control.md)を参照してください。

* Binding historical execution plans is GA [#39199](https://github.com/pingcap/tidb/issues/39199) @[fzzf678](https://github.com/fzzf678)

    v6.5.0では、TiDBは[`CREATE [GLOBAL | SESSION] BINDING`](/sql-statements/sql-statement-create-binding.md)ステートメント内のバインドターゲットを拡張し、歴史的実行計画に基づいてバインディングを作成する機能をサポートしています。v6.6.0では、この機能はGAです。実行計画の選択は現在のTiDBノードに限られません。任意のTiDBノードで生成された任意の歴史的実行計画を[SQLバインディング](/sql-statements/sql-statement-create-binding.md)のターゲットとして選択できます。これにより、機能の使いやすさがさらに向上します。

    詳細については[ドキュメント](/sql-plan-management.md#create-a-binding-according-to-a-historical-execution-plan)を参照してください。

* Add several optimizer hints [#39964](https://github.com/pingcap/tidb/issues/39964) @[Reminiscent](https://github.com/Reminiscent)

    TiDBはv6.6.0でいくつかの最適化ヒントを追加し、`LIMIT`操作の実行計画選択を制御することができます。

    - [`ORDER_INDEX()`](/optimizer-hints.md#order_indext1_name-idx1_name--idx2_name-)は、指定されたインデックスを使用するようオプティマイザに指示し、データを読み取る際にインデックスの順序を保持し、`Limit + IndexScan(keep order: true)`に類似した計画を生成します。
    - [`NO_ORDER_INDEX()`](/optimizer-hints.md#no_order_indext1_name-idx1_name--idx2_name-)は、指定されたインデックスを使用するようオプティマイザに指示し、データを読み取る際にインデックスの順序を保持しないようにし、`TopN + IndexScan(keep order: false)`に類似した計画を生成します。

    最適化ヒントの継続的な導入は、ユーザーにより多くの介入手段を提供し、SQLパフォーマンスの問題を解決し、全体的なパフォーマンスの安定性を向上させます。

* Support dynamically managing the resource usage of DDL operations (experimental) [#38025](https://github.com/pingcap/tidb/issues/38025) @[hawkingrei](https://github.com/hawkingrei)

    TiDB v6.6.0では、DDL操作のリソース管理を導入し、[DDL分散並列実行フレームワーク](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_ddl_distribute_reorg-new-in-v660)が有効になった場合にのみ、オンラインアプリケーションへのDDL変更の影響を減らすためにこれらの操作のCPU使用率を自動的に制御します。

### 利用可能性
* [SQLでのプレースメントルール](/placement-rules-in-sql.md) において `SURVIVAL_PREFERENCE` の設定をサポートしました [#38605](https://github.com/pingcap/tidb/issues/38605) @[nolouch](https://github.com/nolouch)

    `SURVIVAL_PREFERENCES` はデータの災害復旧可能性を高めるデータの生存の好みの設定を提供します。`SURVIVAL_PREFERENCE` を特定することで以下を制御することができます：

    - クラウドリージョンをまたいで展開された TiDB クラスターでは、クラウドリージョンが障害が発生した際に指定したデータベースやテーブルが別のクラウドリージョンで生存します。
    - 単一のクラウドリージョンに展開された TiDB クラスターでは、可用性ゾーンの障害が発生した際に指定したデータベースやテーブルが別の可用性ゾーンで生存します。

  詳細については [ドキュメント](/placement-rules-in-sql.md#specify-survival-preferences) をご覧ください。

* `FLASHBACK CLUSTER TO TIMESTAMP` 文を使用した DDL 操作のロールバックをサポートします [#14088](https://github.com/tikv/tikv/pull/14088) @[Defined2014](https://github.com/Defined2014) @[JmPotato](https://github.com/JmPotato)

    [`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md) 文は、ガーベージコレクション（GC）のライフタイム内の特定の時点までクラスタ全体を復元することをサポートします。TiDB v6.6.0では、この機能は DDL 操作のロールバックをサポートします。これにより、クラスタ上での DML または DDL の誤操作をすばやく元に戻すことができ、クラスタを数分でロールバックすることができ、特定のデータ変更が発生した時刻を決定するため、クラスタをタイムライン上で複数回ロールバックすることができます。

    詳細については [ドキュメント](/sql-statements/sql-statement-flashback-to-timestamp.md) をご覧ください。

### SQL

* MySQL 互換の外部キー制約をサポートします（実験的） [#18209](https://github.com/pingcap/tidb/issues/18209) @[crazycs520](https://github.com/crazycs520)

    TiDB v6.6.0 では MySQL 互換の外部キー制約機能が導入されました。この機能は、テーブル内またはテーブル間の参照、制約の検証、および連鎖操作をサポートします。この機能は、アプリケーションの TiDB 移行を支援し、データの整合性を維持し、データの品質を向上し、データモデリングを容易にします。

    詳細については [ドキュメント](/foreign-key.md) をご覧ください。

* MySQL 互換の多値インデックスをサポートします（実験的） [#39592](https://github.com/pingcap/tidb/issues/39592) @[xiongjiwei](https://github.com/xiongjiwei) @[qw4990](https://github.com/qw4990)

    TiDB v6.6.0 では MySQL 互換の多値インデックスを導入しました。JSON カラム内の配列の値をフィルタリングする操作は一般的ですが、通常のインデックスではこのような操作の高速化には役立ちません。JSON カラムの配列に多値インデックスを作成すると、フィルタリングのパフォーマンスを大幅に向上させることができます。JSON カラムの配列に多値インデックスがある場合、`MEMBER OF()`、`JSON_CONTAINS()`、`JSON_OVERLAPS()` 関数で検索条件をフィルターすることができ、これにより多くの I/O 消費が削減され、操作速度が向上します。

    多値インデックスの導入により、TiDB の JSON データ型へのサポートがさらに向上し、MySQL 8.0 との互換性も向上します。

    詳細については [ドキュメント](/sql-statements/sql-statement-create-index.md#multi-valued-indexes) をご覧ください。

### DB 操作

* リソース消費の多いタスクのための読み取り専用ストレージノードの構成をサポートします @[v01dstar](https://github.com/v01dstar)

    本番環境では、定期的に多くのリソースを消費する読み取り専用操作がありますが、これが全体のクラスターのパフォーマンスに影響を与えることがあります。例えばバックアップや大規模なデータ読み取りおよび解析などです。TiDB v6.6.0 では、リソース消費の多い読み取り専用タスクのための読み取り専用ストレージノードの構成をサポートしています。現在、TiDB、TiSpark、BR は読み取り専用ストレージノードからデータを読み込むことができます。システム変数 `tidb_replica_read`、TiSpark 設定項目 `spark.tispark.replica_read`、または br コマンドライン引数 `--replica-read-label` を使用して、読み取りデータを指定することができます。これによりクラスターのパフォーマンスの安定性を確保することができます。

    詳細については [ドキュメント](/best-practices/readonly-nodes.md) をご覧ください。

* `store-io-pool-size` の動的な変更をサポートします [#13964](https://github.com/tikv/tikv/issues/13964) @[LykxSassinator](https://github.com/LykxSassinator)

    TiKV の設定項目 [`raftstore.store-io-pool-size`](/tikv-configuration-file.md#store-io-pool-size-new-in-v530) は、Raft I/O タスクを処理するスレッドの許容数を指定します。TiKV のパフォーマンスを調整する際にはこの設定を調整することができます。v6.6.0 以前では、この設定項目は動的に変更することができませんでした。v6.6.0 からは、サーバーを再起動することなくこの設定を変更することができます。つまり、柔軟なパフォーマンスの調整が可能となります。

    詳細については [ドキュメント](/dynamic-config.md) をご覧ください。

* TiDB クラスターの初期化時に実行される SQL スクリプトを指定することをサポートします [#35624](https://github.com/pingcap/tidb/issues/35624) @[morgo](https://github.com/morgo)

    TiDB クラスターを初めて起動する際に、コマンドラインパラメータ `--initialize-sql-file` を構成することで実行する SQL スクリプトを指定することができます。システム変数の値を変更したり、ユーザーを作成したり、権限を付与したりする必要がある場合にこの機能を使用することができます。

    詳細については [ドキュメント](/tidb-configuration-file.md#initialize-sql-file-new-in-v660) をご覧ください。

* TiDB データ移行（DM）は、フル移行のパフォーマンスを最大 10 倍向上させる TiDB Lightning の物理インポートモードと統合されました（実験的） @[lance6716](https://github.com/lance6716)

    v6.6.0 では、DM のフル移行能力が TiDB Lightning の物理インポートモードと統合され、DM によってフルデータ移行のパフォーマンスが最大 10 倍向上し、大容量データの移行時間を大幅に短縮することができます。

    v6.6.0 以前では、大容量データの場合、迅速なフルデータ移行のために TiDB Lightning タスクを別途設定し、それから DM を使用して増分データ移行を行う必要がありました。これは複雑な構成でした。v6.6.0 からは、TiDB Lightning タスクの設定を行わずに大容量データを移行することができます。1 つの DM タスクで移行を行うことができます。

    詳細については [ドキュメント](/dm/dm-precheck.md#check-items-for-physical-import) をご覧ください。

* TiDB Lightning に、ソースファイルとターゲットテーブルの列名が一致しない場合の問題に対処する新しい構成パラメータ `"header-schema-match"` を追加しました @[dsdashun](https://github.com/dsdashun)

    v6.6.0 では、TiDB Lightning に新しいプロファイルパラメータ `"header-schema-match"` が追加されました。デフォルト値は `true` で、これはソース CSV ファイルの最初の行を列名として扱い、その列名をターゲットテーブルと一致させることを意味します。CSV テーブルのヘッダー内のフィールド名がターゲットテーブルの列名と一致しない場合、この構成を `false` に設定することができます。その場合、TiDB Lightning はエラーを無視し、ターゲットテーブルの列の順序でデータのインポートを続行します。

    詳細については [ドキュメント](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) をご覧ください。

* TiDB Lightning は、TiKV にキーと値のペアを送信する際の圧縮転送を有効にすることをサポートします [#41163](https://github.com/pingcap/tidb/issues/41163) @[sleepymole](https://github.com/sleepymole)

    v6.6.0 から、TiDB Lightning はローカルでエンコードされたキーと値のペアを圧縮し、TiKV に送信する際のネットワーク転送を圧縮することをサポートします。これにより、ネットワークを通じて転送されるデータ量が減少し、ネットワーク帯域幅のオーバーヘッドが低減されます。この機能はデフォルトでは無効です。有効にするには、TiDB Lightning の `compress-kv-pairs` 設定項目を `"gzip"` または `"gz"` に設定します。

    詳細については [ドキュメント](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) をご覧ください。

* TiKV-CDC ツールは現在 GA であり、RawKV のデータ変更にサブスクライブすることをサポートします [#48](https://github.com/tikv/migration/issues/48) @[zeminzhou](https://github.com/zeminzhou) @[haojinming](https://github.com/haojinming) @[pingyu](https://github.com/pingyu)

    TiKV-CDC は TiKV クラスター用の CDC（Change Data Capture）ツールです。TiKV と PD を TiDB なしで使用する場合、その組み合わせは RawKV と呼ばれます。TiKV-CDC は RawKV のデータ変更をサブスクライブし、それらをリアルタイムでダウンストリーム TiKV クラスターにレプリケートすることをサポートし、これにより RawKV のクロスクラスターレプリケーションが可能となります。
    詳細については、[ドキュメント](https://tikv.org/docs/latest/concepts/explore-tikv-features/cdc/cdc/)を参照してください。

* TiCDCは、Kafka changefeedでシングルテーブルのスケールアウトをサポートし、changefeedを複数のTiCDCノードに配布します（実験的）[#7720](https://github.com/pingcap/tiflow/issues/7720) @[overvenus](https://github.com/overvenus)

    v6.6.0 より前、上流のテーブルが大量の書き込みを受け入れると、このテーブルのレプリケーション能力をスケールアウトできず、レプリケーションの待ち時間が増加します。TiCDC v6.6.0 から、上流テーブルの changefeed を複数の TiCDC ノードに Kafka sink に配布することができます。つまり、単一のテーブルのレプリケーション能力がスケールアウトされます。

    詳細については、[ドキュメント](/ticdc/ticdc-sink-to-kafka.md#scale-out-the-load-of-a-single-large-table-to-multiple-ticdc-nodes)を参照してください。

* [GORM](https://github.com/go-gorm/gorm) は、TiDB 統合テストを追加しました。TiDB は現在、GORM でデフォルトのサポートされるデータベースです。[#6014](https://github.com/go-gorm/gorm/pull/6014) @[Icemap](https://github.com/Icemap)

    - v1.4.6 では、[GORM MySQL ドライバ](https://github.com/go-gorm/mysql) は、TiDB の `AUTO_RANDOM` 属性に対応しました [#104](https://github.com/go-gorm/mysql/pull/104)
    - v1.4.6 では、[GORM MySQL ドライバ](https://github.com/go-gorm/mysql) は、TiDB への接続時に `Unique` フィールドの `Unique` 属性を `AutoMigrate` 中に変更できない問題を修正しました [#105](https://github.com/go-gorm/mysql/pull/105)
    - [GORM ドキュメント](https://github.com/go-gorm/gorm.io) は、TiDB をデフォルトのデータベースとして言及しています [#638](https://github.com/go-gorm/gorm.io/pull/638)

    詳細については、[GORM ドキュメント](https://gorm.io/docs/index.html)を参照してください。

### 可観測性

* TiDB v6.6.0 では、SQL バインディングをすばやく TiDB ダッシュボード上に作成することがサポートされます [#781](https://github.com/pingcap/tidb-dashboard/issues/781) @[YiniXu9506](https://github.com/YiniXu9506)

    TiDB v6.6.0 では、ステートメント履歴から SQL バインディングを作成できるようになり、TiDB ダッシュボード上で SQL ステートメントを特定の計画にすばやくバインディングできます。

    ユーザーフレンドリーなインターフェースを提供することで、この機能は TiDB での計画バインディングのプロセスを簡素化し、操作の複雑さを減らし、計画のバインディングプロセスの効率とユーザーエクスペリエンスを向上させます。

    詳細については、[ドキュメント](/dashboard/dashboard-statement-details.md#fast-plan-binding)を参照してください。

* キャッシュ実行計画の警告を追加 @[qw4990](https://github.com/qw4990)

    実行計画をキャッシュできない場合、TiDB は警告を表示して診断を容易にします。例えば：

    ```sql
    mysql> PREPARE st FROM 'SELECT * FROM t WHERE a<?';
    クエリが実行されました: 0 行が選択されました (0.00 sec)

    mysql> SET @a='1';
    クエリが実行されました: 0 行が選択されました (0.00 sec)

    mysql> EXECUTE st USING @a;
    空のセットが選択されました: 1件の警告 (0.01 sec)

    mysql> SHOW WARNINGS;
    +---------+------+----------------------------------------------+
    | レベル   | コード | メッセージ                                      |
    +---------+------+----------------------------------------------+
    | 警告 | 1105 | skip plan-cache: '1' may be converted to INT |
    +---------+------+----------------------------------------------+
    ```

    前述の例では、オプティマイザが非 INT 型を INT 型に変換し、パラメータの変更とともに実行計画が変更される可能性があり、TiDB はその計画をキャッシュしないようにしています。

    詳細については、[ドキュメント](/sql-prepared-plan-cache.md#diagnostics-of-prepared-plan-cache)を参照してください。

* 遅いクエリログに `Warnings` フィールドを追加 [#39893](https://github.com/pingcap/tidb/issues/39893) @[time-and-fate](https://github.com/time-and-fate)

    TiDB v6.6.0 では、遅いクエリログに `Warnings` フィールドが追加され、パフォーマンスの問題を診断するのに役立ちます。このフィールドには、遅いクエリの実行中に生成された警告が記録されます。また、TiDB ダッシュボードの遅いクエリページでも警告を表示できます。

    詳細については、[ドキュメント](/identify-slow-queries.md)を参照してください。

* SQL 実行計画の自動的な生成をキャプチャする機能を追加 [#38779](https://github.com/pingcap/tidb/issues/38779) @[Yisaer](https://github.com/Yisaer)

    実行計画のトラブルシューティングの過程で、`PLAN REPLAYER` は現場を保存し、診断の効率を向上させます。ただし、いくつかのシナリオでは、いくつかの実行計画の生成が自由に再現できないため、診断作業が困難になります。

    このような問題に対処するために、TiDB v6.6.0 では、`PLAN REPLAYER` が自動キャプチャの機能を拡張しました。`PLAN REPLAYER CAPTURE` コマンドを使用して、事前にターゲット SQL ステートメントを登録し、同時にターゲット実行計画を指定することができます。TiDB が登録されたターゲットにマッチする SQL ステートメントや実行計画を検出すると、自動的に `PLAN REPLAYER` 情報を生成しパッケージ化します。実行計画が不安定な場合、この機能は診断効率を向上させることができます。

    この機能を使用するには、[`tidb_enable_plan_replayer_capture`](/system-variables.md#tidb_enable_plan_replayer_capture) の値を `ON` に設定してください。

    詳細については、[ドキュメント](/sql-plan-replayer.md#use-plan-replayer-capture)を参照してください。

* ステートメントの要約情報の永続化をサポート (実験的) [#40812](https://github.com/pingcap/tidb/issues/40812) @[mornyx](https://github.com/mornyx)

    v6.6.0 より前、ステートメントの要約データはメモリに保持され、TiDB サーバーの再起動時に失われました。v6.6.0 から、TiDB はステートメントの要約永続化を有効にすることをサポートし、過去のデータを定期的にディスクに書き込むことができます。同時に、システムテーブルのクエリ結果はメモリではなくディスクから導出されます。TiDB が再起動しても、すべての履歴データが残ります。

    詳細については、[ドキュメント](/statement-summary-tables.md#persist-statements-summary)を参照してください。

### セキュリティ

* TiFlash は TLS 証明書の自動ローテーションをサポート [#5503](https://github.com/pingcap/tiflash/issues/5503) @[ywqzzy](https://github.com/ywqzzy)

    v6.6.0 では、TiDB は TiFlash TLS 証明書の自動ローテーションをサポートしています。コンポーネント間のデータ送信暗号化が有効になっている TiDB クラスターでは、TiFlash TLS 証明書の有効期限が切れて新しいものに取り替える必要がある場合、TiDB クラスターを再起動することなく新しい TiFlash TLS 証明書を自動的に読み込むことができます。また、TiDB クラスター内のコンポーネント間で TLS 証明書をローテーションしても、TiDB クラスターの利用に影響を与えません。これにより、クラスターの高可用性が確保されます。

    詳細については、[ドキュメント](/enable-tls-between-components.md)を参照してください。

* TiDB Lightning は AWS IAM ロールのキーとセッショントークンを使用して Amazon S3 データにアクセスすることをサポート [#4075](https://github.com/pingcap/tidb/issues/40750) @[okJiang](https://github.com/okJiang)

    v6.6.0 より前、TiDB Lightning は AWS IAM **ユーザーのアクセスキー**を使用して S3 データにアクセスするだけで、一時セッショントークンを使用して S3 データにアクセスすることはできませんでした。v6.6.0 から、TiDB Lightning は AWS IAM **ロールのアクセスキー＋セッショントークン**を使用して S3 データにアクセスすることをサポートし、データセキュリティを向上させます。

    詳細については、[ドキュメント](/tidb-lightning/tidb-lightning-data-source.md#import-data-from-amazon-s3)を参照してください。

### テレメトリ

- 2023年2月20日以降、[テレメトリ機能](/telemetry.md)は、新しい TiDB および TiDB Dashboard のバージョン（v6.6.0を含む）ではデフォルトで無効になります。デフォルトのテレメトリ設定を使用していた以前のバージョンからアップグレードする場合、アップグレード後もテレメトリ機能は無効になります。具体的なバージョンについては、[TiDB リリースタイムライン](/releases/release-timeline.md)を参照してください。
- v1.11.3 以降、新しくデプロイされた TiUP では、テレメトリ機能はデフォルトで無効になります。以前のバージョンから v1.11.3 またはそれ以降のバージョンにアップグレードする場合、アップグレード前と同じ状態でテレメトリ機能が維持されます。

## 互換性の変更

> **注意:**
>
> このセクションでは、現在のバージョン（v6.6.0）にアップグレードする際に把握する必要のある互換性の変更が提供されます。v6.4.0 またはそれ以前のバージョンから現在のバージョンにアップグレードする場合、中間バージョンで導入された互換性の変更もチェックする必要があります。

### MySQL 互換性
* MySQL互換の外部キー制約をサポート（実験的） [#18209](https://github.com/pingcap/tidb/issues/18209) @[crazycs520](https://github.com/crazycs520)

    詳細については、このドキュメントの[SQL](#sql)セクションおよび[documentation](/foreign-key.md)を参照してください。

* MySQL互換の多値インデックスをサポート（実験的） [#39592](https://github.com/pingcap/tidb/issues/39592) @[xiongjiwei](https://github.com/xiongjiwei) @[qw4990](https://github.com/qw4990)

    詳細については、このドキュメントの[SQL](#sql)セクションおよび[documentation](/sql-statements/sql-statement-create-index.md#multi-valued-indexes)を参照してください。

### システム変数

| 変数名  | 変更タイプ    | 説明 |
|--------|------------------------------|------|
| `tidb_enable_amend_pessimistic_txn` | 削除 | v6.5.0から、この変数は非推奨となりました。v6.6.0から、この変数と`AMEND TRANSACTION`機能は削除されました。TiDBは[`メタロック`](/metadata-lock.md)を使用し、`Information schema is changed`エラーを回避します。 |
| `tidb_enable_concurrent_ddl` | 削除 | この変数は、TiDBが同時実行DDLステートメントを使用するかどうかを制御します。この変数が無効になっているとき、TiDBは古いDDL実行フレームワークを使用し、同時実行DDL実行のサポートが制限されます。v6.6.0から、この変数は削除され、TiDBは古いDDL実行フレームワークをサポートしなくなります。 |
| `tidb_ttl_job_run_interval` | 削除 | この変数は、バックグラウンドでTTLジョブのスケジュール間隔を制御するために使用されます。v6.6.0から、この変数は削除されました。なぜなら、TiDBは各テーブルに`TTLランタイム`を制御するための属性を提供し、`tidb_ttl_job_run_interval`より柔軟な方法で設定できるためです。 |
| [`foreign_key_checks`](/system-variables.md#foreign_key_checks) | 変更 | この変数は、外部キー制約のチェックを有効にするかどうかを制御します。デフォルト値は`OFF`から`ON`に変更され、デフォルトで外部キーのチェックが有効になります。 |
| [`tidb_enable_foreign_key`](/system-variables.md#tidb_enable_foreign_key-new-in-v630) | 変更 | この変数は、外部キー機能を有効にするかどうかを制御します。デフォルト値は`OFF`から`ON`に変更され、デフォルトで外部キーが有効になります。 |
| `tidb_enable_general_plan_cache` | 変更 | この変数は、一般的なプランキャッシュを有効にするかどうかを制御します。v6.6.0から、この変数は[`tidb_enable_non_prepared_plan_cache`](/system-variables.md#tidb_enable_non_prepared_plan_cache)に名前が変更されました。 |
| [`tidb_enable_historical_stats`](/system-variables.md#tidb_enable_historical_stats) | 変更 | この変数は、履歴統計情報を有効にするかどうかを制御します。デフォルト値は`OFF`から`ON`に変更され、デフォルトで履歴統計情報が有効になります。 |
| [`tidb_enable_telemetry`](/system-variables.md#tidb_enable_telemetry-new-in-v402) | 変更 | デフォルト値を`ON`から`OFF`に変更し、TiDBでテレメトリがデフォルトで無効になるようになりました。 |
| `tidb_general_plan_cache_size` | 変更 | この変数は、一般的なプランキャッシュでキャッシュできる実行プランの最大数を制御します。v6.6.0から、この変数は[`tidb_non_prepared_plan_cache_size`](/system-variables.md#tidb_non_prepared_plan_cache_size)に名前が変更されました。 |
| [`tidb_replica_read`](/system-variables.md#tidb_replica_read-new-in-v40) | 変更 | この変数には新しい値のオプション`learner`が追加され、TiDBが読み取り専用ノードからデータを読み取る際にリーダーレプリカとして指定するレプリカを指定します。 |
| [`tidb_replica_read`](/system-variables.md#tidb_replica_read-new-in-v40) | 変更 | この変数には新しい値のオプション`prefer-leader`が追加され、TiDBクラスタの全体的な読み取り可用性を向上させます。このオプションを設定すると、TiDBはリーダーレプリカから読み取ることを優先します。リーダーレプリカのパフォーマンスが著しく低下した場合、TiDBは自動的にフォロワーレプリカから読み取ります。 |
| [`tidb_store_batch_size`](/system-variables.md#tidb_store_batch_size) | 変更 | この変数は、`IndexLookUp`オペレータのCoprocessorタスクのバッチサイズを制御します。`0`はバッチを無効にすることを意味します。v6.6.0から、デフォルト値が`0`から`4`に変更され、1つのバッチのリクエストごとに4つのCoprocessorタスクがバッチ処理されるようになりました。 |
| [`mpp_exchange_compression_mode`](/system-variables.md#mpp_exchange_compression_mode-new-in-v660)  | 追加 | この変数は、MPP Exchangeオペレータのデータ圧縮モードを指定します。これはTiDBがバージョン番号`1`のMPP実行プランを選択した場合に効果を発揮します。デフォルト値`UNSPECIFIED`は、TiDBが自動的に`FAST`圧縮モードを選択することを意味します。 |
| [`mpp_version`](/system-variables.md#mpp_version-new-in-v660)  | 追加 | この変数は、MPP実行プランのバージョンを指定します。バージョンが指定された後、TiDBは指定されたバージョンのMPP実行プランを選択します。デフォルト値`UNSPECIFIED`は、TiDBが自動的に最新のバージョン`1`を選択することを意味します。 |
| [`tidb_ddl_distribute_reorg`](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_ddl_distribute_reorg-new-in-v660) | 追加 | この変数は、DDL再構成フェーズの分散実行を有効にするかどうかを制御します。デフォルト値`OFF`は、デフォルトでDDL再構成フェーズの分散実行が有効になっていません。現在、この変数は`ADD INDEX`にのみ効果があります。 |
| [`tidb_enable_historical_stats_for_capture`](/system-variables.md#tidb_enable_historical_stats_for_capture) | 追加 | この変数は、`PLAN REPLAYER CAPTURE`によってキャプチャされる情報において履歴統計情報をデフォルトで含めるかどうかを制御します。デフォルト値`OFF`は、デフォルトで履歴統計情報が含まれないことを意味します。 |
| [`tidb_enable_plan_cache_for_param_limit`](/system-variables.md#tidb_enable_plan_cache_for_param_limit-new-in-v660) | 追加 | この変数は、Prepared Plan Cacheが`LIMIT`後の`COUNT`を含む実行プランをキャッシュするかどうかを制御します。デフォルト値は`ON`であり、Prepared Plan Cacheはこのような実行プランをキャッシュするサポートをしています。ただし、Prepared Plan Cacheは10,000よりも大きい数をカウントする`COUNT`条件を含む実行プランをキャシュすることはサポートしていません。 |
| [`tidb_enable_plan_replayer_capture`](/system-variables.md#tidb_enable_plan_replayer_capture) | 追加 | この変数は、[`PLAN REPLAYER CAPTURE`機能](/sql-plan-replayer.md#use-plan-replayer-capture-to-capture-target-plans)を有効にするかどうかを制御します。デフォルト値`OFF`は、`PLAN REPLAYER CAPTURE`機能が無効であることを意味します。 |
| [`tidb_enable_resource_control`](/system-variables.md#tidb_enable_resource_control-new-in-v660) | 追加 | この変数は、リソース制御機能を有効にするかどうかを制御します。デフォルト値は`OFF`です。この変数が`ON`に設定されている場合、TiDBクラスタはリソースグループに基づいてアプリケーションのリソース分離をサポートします。 |
| [`tidb_historical_stats_duration`](/system-variables.md#tidb_historical_stats_duration-new-in-v660) | 追加 | この変数は、履歴統計情報をストレージに保持する期間を制御します。デフォルト値は7日です。 |
| [`tidb_index_join_double_read_penalty_cost_rate`](/system-variables.md#tidb_index_join_double_read_penalty_cost_rate-new-in-v660) | 追加 | この変数は、インデックス結合の選択に若干のペナルティコストを追加するかどうかを制御します。デフォルト値`0`は、デフォルトでこの機能が無効であることを意味します。 |
| [`tidb_pessimistic_txn_aggressive_locking`](https://docs.pingcap.com/tidb/v6.6/system-variables#tidb_pessimistic_txn_aggressive_locking-new-in-v660) | 追加 | この変数は、悲観的トランザクションのための強化されたロック解除モデルを使用するかどうかを制御します。デフォルト値`OFF`は、デフォルトでこのようなウェイクアップモデルを使用しないことを意味します。 |
| [`tidb_stmt_summary_enable_persistent`](/system-variables.md#tidb_stmt_summary_enable_persistent-new-in-v660) | 追加 | この変数は読み取り専用です。[ステートメントのサマリー永続化](/statement-summary-tables.md#persist-statements-summary)を有効にするかどうかを制御します。この変数の値は、構成項目[`tidb_stmt_summary_enable_persistent`](/tidb-configuration-file.md#tidb_stmt_summary_enable_persistent-new-in-v660)と同じです。 |
| [`tidb_stmt_summary_filename`](/system-variables.md#tidb_stmt_summary_filename-new-in-v660) | 追加 | この変数は読み取り専用です。[ステートメントのサマリー永続化](/statement-summary-tables.md#persist-statements-summary)が有効になっている場合に永続データが書込まれるファイルを指定します。この変数の値は、構成項目[`tidb_stmt_summary_filename`](/tidb-configuration-file.md#tidb_stmt_summary_filename-new-in-v660)と同じです。 |
| [`tidb_stmt_summary_file_max_backups`](/system-variables.md#tidb_stmt_summary_file_max_backups-new-in-v660) | 新たに追加された | この変数は読み取り専用です。 [ステートメントのサマリー永続化](/statement-summary-tables.md#persist-statements-summary) が有効になっている場合、永続化できるデータファイルの最大数を指定します。この変数の値は、構成項目 [`tidb_stmt_summary_file_max_backups`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_backups-new-in-v660) と同じです。|
| [`tidb_stmt_summary_file_max_days`](/system-variables.md#tidb_stmt_summary_file_max_days-new-in-v660) | 新たに追加された | この変数は読み取り専用です。 [ステートメントのサマリー永続化](/statement-summary-tables.md#persist-statements-summary) が有効になっている場合、永続化されたデータファイルを保持する最大日数を指定します。この変数の値は、構成項目 [`tidb_stmt_summary_file_max_days`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_days-new-in-v660) と同じです。|
| [`tidb_stmt_summary_file_max_size`](/system-variables.md#tidb_stmt_summary_file_max_size-new-in-v660) | 新たに追加された | この変数は読み取り専用です。 [ステートメントのサマリー永続化](/statement-summary-tables.md#persist-statements-summary) が有効になっている場合、永続化されたデータファイルの最大サイズを指定します。この変数の値は、構成項目 [`tidb_stmt_summary_file_max_size`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_size-new-in-v660) と同じです。|

### 構成ファイルパラメータ

| 構成ファイル | 構成パラメータ | 変更タイプ | 説明 |
| -------- | -------- | -------- | -------- |
| TiKV  | `rocksdb.enable-statistics` | 削除された | この構成項目は、RocksDBの統計情報の有効化を指定します。v6.6.0から、この項目は削除されました。RocksDB統計情報は、診断のためにデフォルトですべてのクラスタで有効になっています。詳細については、[#13942](https://github.com/tikv/tikv/pull/13942) を参照してください。 |
| TiKV  | `raftdb.enable-statistics` | 削除された | この構成項目は、Raft RocksDB 統計情報の有効化を指定します。v6.6.0から、この項目は削除されました。Raft RocksDB 統計情報は、診断のためにデフォルトですべてのクラスタで有効になっています。詳細については、[#13942](https://github.com/tikv/tikv/pull/13942) を参照してください。 |
| TiKV | `storage.block-cache.shared` | 削除された | v6.6.0から、この構成項目は削除され、ブロックキャッシュはデフォルトで有効になり、無効にすることはできません。詳細については、[#12936](https://github.com/tikv/tikv/issues/12936) を参照してください。 |
| DM | `on-duplicate` |  削除された | この構成項目は、フルインポートフェーズ中の競合解決メソッドを制御します。v6.6.0では、新しい `on-duplicate-logical` と `on-duplicate-physical` の構成項目が導入され、`on-duplicate` を置き換えます。 |
| TiDB | [`enable-telemetry`](/tidb-configuration-file.md#enable-telemetry-new-in-v402) | 変更された | v6.6.0から、デフォルト値は `true` から `false` に変更され、つまりデフォルトで TiDB でテレメトリが無効になります。 |
| TiKV  | [`rocksdb.defaultcf.block-size`](/tikv-configuration-file.md#block-size) and [`rocksdb.writecf.block-size`](/tikv-configuration-file.md#block-size)  |  変更された  | デフォルト値が `64K` から `32K` に変更されます。  |
| TiKV | [`rocksdb.defaultcf.block-cache-size`](/tikv-configuration-file.md#block-cache-size), [`rocksdb.writecf.block-cache-size`](/tikv-configuration-file.md#block-cache-size), [`rocksdb.lockcf.block-cache-size`](/tikv-configuration-file.md#block-cache-size) | 廃止された | v6.6.0から、これらの構成項目は廃止されました。詳細については、[#12936](https://github.com/tikv/tikv/issues/12936) を参照してください。 |
| PD | [`enable-telemetry`](/pd-configuration-file.md#enable-telemetry) | 変更された | v6.6.0から、デフォルト値は `true` から `false` に変更され、つまりデフォルトで TiDB Dashboard でテレメトリが無効になります。 |
| DM | [`import-mode`](/dm/task-configuration-file-full.md) |  変更された | この構成項目の可能な値が `"sql"` と `"loader"` から `"logical"` と `"physical"` に変更されます。デフォルト値は `"logical"` で、つまりデータをインポートする際に TiDB Lightning の論理インポートモードを使用します。 |
| TiFlash |  [`profile.default.max_memory_usage_for_all_queries`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)  |  変更された  | すべてのクエリで生成される中間データのメモリ使用量の制限を指定します。v6.6.0から、デフォルト値は `0` から `0.8` に変更され、つまり制限が総メモリの80%になります。 |
| TiCDC  | [`consistent.storage`](/ticdc/ticdc-sink-to-mysql.md#prerequisites)  |  変更された  | この構成項目は、リログバックアップが格納されるパスを指定します。`scheme` に GCS と Azure の2つの値オプションが追加されました。 |
| TiDB | [`initialize-sql-file`](/tidb-configuration-file.md#initialize-sql-file-new-in-v660) | 新たに追加された | この構成項目は、TiDB クラスターが初めて起動される際に実行されるSQLスクリプトを指定します。デフォルト値は空です。 |
| TiDB | [`tidb_stmt_summary_enable_persistent`](/tidb-configuration-file.md#tidb_stmt_summary_enable_persistent-new-in-v660) | 新たに追加された | この構成項目は、ステートメントのサマリー永続化を有効にするかどうかを制御します。デフォルト値は `false` で、つまりこの機能はデフォルトでは無効になります。 |
| TiDB | [`tidb_stmt_summary_file_max_backups`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_backups-new-in-v660) | 新たに追加された | ステートメントのサマリー永続化が有効になっている場合、この構成は永続化できるデータファイルの最大数を指定します。`0` はファイル数に制限がないことを意味します。 |
| TiDB | [`tidb_stmt_summary_file_max_days`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_days-new-in-v660) | 新たに追加された | ステートメントのサマリー永続化が有効になっている場合、この構成は永続化されたデータファイルを保持する最大日数を指定します。 |
| TiDB | [`tidb_stmt_summary_file_max_size`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_size-new-in-v660) | 新たに追加された | ステートメントのサマリー永続化が有効になっている場合、この構成は永続化されたデータファイルの最大サイズ（MiB単位）を指定します。 |
| TiDB | [`tidb_stmt_summary_filename`](/tidb-configuration-file.md#tidb_stmt_summary_filename-new-in-v660) | 新たに追加された | ステートメントのサマリー永続化が有効になっている場合、この構成は永続化されたデータが書き込まれるファイルを指定します。 |
| TiKV | [`resource-control.enabled`](/tikv-configuration-file.md#resource-control) | 新たに追加された | 対応するリソースグループのリクエストユニット（RU）に応じて、ユーザーの前景読み書きリクエストのスケジューリングを有効にするかどうかを指定します。デフォルト値は `false` で、つまり対応するリソースグループのRUに応じたスケジューリングを無効にします。 |
| TiKV | [`storage.engine`](/tikv-configuration-file.md#engine-new-in-v660) | 新たに追加された | この構成項目は、ストレージエンジンの種類を指定します。値のオプションは `"raft-kv"` と `"partitioned-raft-kv"` です。この構成項目はクラスターを作成する際にのみ指定でき、一度指定された後に変更することはできません。 |
| TiKV | [`rocksdb.write-buffer-flush-oldest-first`](/tikv-configuration-file.md#write-buffer-flush-oldest-first-new-in-v660) | 新たに追加された | この構成項目は、現在のRocksDBの`memtable`のメモリ使用量が閾値に達したときに使用されるフラッシュ戦略を指定します。  |
| TiKV | [`rocksdb.write-buffer-limit`](/tikv-configuration-file.md#write-buffer-limit-new-in-v660) | 新たに追加された | この構成項目は、単一のTiKV内のすべてのRocksDBインスタンスの`memtable`が使用する総メモリの制限を指定します。デフォルト値は、総メモリの25%です。  |
| PD  | [`pd-server.enable-gogc-tuner`](/pd-configuration-file.md#enable-gogc-tuner-new-in-v660) | 新たに追加された | この構成項目は、GOGCチューナーを有効にするかどうかを制御します。デフォルトでは無効です。 |
| PD  | [`pd-server.gc-tuner-threshold`](/pd-configuration-file.md#gc-tuner-threshold-new-in-v660) | 新たに追加された | この構成項目は、GOGCのチューニングのための最大メモリしきい値の比を指定します。デフォルト値は `0.6` です。 |
| PD  | [`pd-server.server-memory-limit-gc-trigger`](/pd-configuration-file.md#server-memory-limit-gc-trigger-new-in-v660) | 新たに追加された | この構成項目は、PDがGCをトリガーしようとするしきい値比率を指定します。デフォルト値は `0.7` です。 |
| PD | [`pd-server.server-memory-limit`](/pd-configuration-file.md#server-memory-limit-new-in-v660) | 新しく追加された | この構成項目は、PD インスタンスのメモリ制限の割合を指定します。値 `0` はメモリ制限がないことを意味します。 |
| TiCDC | [`scheduler.region-per-span`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 新しく追加された | この構成項目は、リージョン数に基づいてテーブルを複数の複製範囲に分割し、これらの範囲を複数の TiCDC ノードで複製できるように制御します。デフォルト値は `50000` です。 |
| TiDB Lightning | [`compress-kv-pairs`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) | 新しく追加された | この構成項目は、物理的なインポートモードで TiKV に KV ペアを送信する際に圧縮を有効にするかどうかを制御します。デフォルト値は空で、圧縮が有効になっていません。 |
| DM | [`checksum-physical`](/dm/task-configuration-file-full.md) | 新しく追加された | この構成項目は、インポート後にデータの整合性を確認するために、DM が各テーブルに対して `ADMIN CHECKSUM TABLE <table>` を実行するかどうかを制御します。デフォルト値は `"required"` で、インポート後に admin checksum を実行します。Checksum に失敗した場合、DM はタスクを一時停止し、失敗を手動で処理する必要があります。 |
| DM | [`disk-quota-physical`](/dm/task-configuration-file-full.md) | 新しく追加された | この構成項目はディスククォータを設定します。これは TiDB Lightning の [`disk-quota` 構成](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md#configure-disk-quota-new-in-v620) に対応しています。 |
| DM | [`on-duplicate-logical`](/dm/task-configuration-file-full.md) | 新しく追加された | この構成項目は、DM が論理インポートモードで競合するデータをどのように解決するかを制御します。デフォルト値は `"replace"` で、新しいデータを既存のデータで置き換えます。 |
| DM | [`on-duplicate-physical`](/dm/task-configuration-file-full.md) | 新しく追加された | この構成項目は、DM が物理インポートモードで競合するデータをどのように解決するかを制御します。デフォルト値は `"none"` で、競合するデータを解決しないことを意味します。 `"none"` は最良のパフォーマンスを持ちますが、下流のデータベースで不整合なデータを引き起こす可能性があります。 |
| DM | [`sorting-dir-physical`](/dm/task-configuration-file-full.md) | 新しく追加された | この構成項目は、物理インポートモードでローカルKVソートに使用されるディレクトリを指定します。デフォルト値は `dir` 構成と同じです。 |
| sync-diff-inspector | [`skip-non-existing-table`](/sync-diff-inspector/sync-diff-inspector-overview.md#configuration-file-description) | 新しく追加された | この構成項目は、下流のテーブルに上流に存在しない場合に、上流と下流のデータ整合性のチェックをスキップするかどうかを制御します。 |
| TiSpark | [`spark.tispark.replica_read`](/tispark-overview.md#tispark-configurations) | 新しく追加された | この構成項目は、読み取る複製の種類を制御します。値のオプションは `leader`、`follower`、`learner` です。 |
| TiSpark | [`spark.tispark.replica_read.label`](/tispark-overview.md#tispark-configurations) | 新しく追加された | この構成項目は、対象となる TiKV ノードにラベルを設定するために使用されます。 |

### その他

- [`store-io-pool-size`](/tikv-configuration-file.md#store-io-pool-size-new-in-v530) を動的に変更できるようにサポートしました。これにより、より柔軟な TiKV のパフォーマンスチューニングが可能になります。
- `LIMIT` 句の制限を解除し、実行パフォーマンスを向上させました。
- v6.6.0 以降、BR は v6.1.0 より前のクラスタにデータを復元することをサポートしていません。
- v6.6.0 以降、TiDB はパーティションテーブルの列タイプを変更することをサポートしなくなりました。これは潜在的な正確性の問題のためです。

## 改善点

+ TiDB

    - TTL バックグラウンドクリーニングタスクのスケジューリングメカニズムを改善し、単一のテーブルのクリーニングタスクを複数の TiDB ノードで同時に実行できるようにしました [#40361](https://github.com/pingcap/tidb/issues/40361) @[YangKeao](https://github.com/YangKeao)
    - デフォルト以外の区切り文字を設定した後に複数のステートメントを実行して結果を表示する際の列名表示を最適化しました [#39662](https://github.com/pingcap/tidb/issues/39662) @[mjonss](https://github.com/mjonss)
    - 警告メッセージが生成された後のステートメントの実行効率を最適化しました [#39702](https://github.com/pingcap/tidb/issues/39702) @[tiancaiamao](https://github.com/tiancaiamao)
    - `ADD INDEX` の分散データバックフィルをサポートしました（実験的） [#37119](https://github.com/pingcap/tidb/issues/37119) @[zimulala](https://github.com/zimulala)
    - 列のデフォルト値として `CURDATE()` を使用することをサポートしました [#38356](https://github.com/pingcap/tidb/issues/38356) @[CbcWestwolf](https://github.com/CbcWestwolf)
    - `partial order prop push down` で LIST 型のパーティションテーブルをサポートしました [#40273](https://github.com/pingcap/tidb/issues/40273) @[winoros](https://github.com/winoros)
    - オプティマイザヒントと実行計画バインディングの間の競合に対するエラーメッセージを追加しました [#40910](https://github.com/pingcap/tidb/issues/40910) @[Reminiscent](https://github.com/Reminiscent)
    - 一部のシナリオでプランキャッシュを使用する際に、最適でないプランを回避するためのプランキャッシュストラテジーを最適化しました [#40312](https://github.com/pingcap/tidb/pull/40312) [#40218](https://github.com/pingcap/tidb/pull/40218) [#40280](https://github.com/pingcap/tidb/pull/40280) [#41136](https://github.com/pingcap/tidb/pull/41136) [#40686](https://github.com/pingcap/tidb/pull/40686) @[qw4990](https://github.com/qw4990)
    - 定期的に期限切れのリージョンキャッシュをクリアし、メモリリークとパフォーマンス低下を回避します [#40461](https://github.com/pingcap/tidb/issues/40461) @[sticnarf](https://github.com/sticnarf)
    - パーティションテーブルで `MODIFY COLUMN` がサポートされていません [#39915](https://github.com/pingcap/tidb/issues/39915) @[wjhuang2016](https://github.com/wjhuang2016)
    - パーティションテーブルが依存する列のリネームを無効にしました [#40150](https://github.com/pingcap/tidb/issues/40150) @[mjonss](https://github.com/mjonss)
    - パーティションテーブルが依存する列が削除された場合のエラーメッセージを改良しました [#38739](https://github.com/pingcap/tidb/issues/38739) @[jiyfhust](https://github.com/jiyfhust)
    - `FLASHBACK CLUSTER` が `min-resolved-ts` のチェックに失敗した場合にリトライするメカニズムを追加しました[#39836](https://github.com/pingcap/tidb/issues/39836) @[Defined2014](https://github.com/Defined2014)

+ TiKV

    - パーティション化された Raft-KV モードの一部のパラメータのデフォルト値を最適化しました：TiKV 構成項目 `storage.block-cache.capacity` のデフォルト値を 45% から 30% に調整し、`region-split-size` のデフォルト値を `96MiB` から `10GiB` に調整しました。Raft-KV モードと `enable-region-bucket` が `true` の場合、`region-split-size` はデフォルトで 1 GiB に調整されます。[#12842](https://github.com/tikv/tikv/issues/12842) @[tonyxuqqi](https://github.com/tonyxuqqi)
    - Raftstore 非同期書き込みでの優先度スケジューリングをサポートしました [#13730](https://github.com/tikv/tikv/issues/13730) @[Connor1996](https://github.com/Connor1996)
    - 1 コア未満の CPU で TiKV を起動することをサポートしました [#13586](https://github.com/tikv/tikv/issues/13586) [#13752](https://github.com/tikv/tikv/issues/13752) [#14017](https://github.com/tikv/tikv/issues/14017) @[andreid-db](https://github.com/andreid-db)
    - Raftstore 遅延スコアの新しい検出メカニズムを最適化し、`evict-slow-trend-scheduler` を追加しました [#14131](https://github.com/tikv/tikv/issues/14131) @[innerr](https://github.com/innerr)
    - RocksDB のブロックキャッシュを共有化し、CF ごとにブロックキャッシュを別々に設定することをサポートしないようにしました [#12936](https://github.com/tikv/tikv/issues/12936) @[busyjay](https://github.com/busyjay)

+ PD

    - OOM 問題を緩和するためにグローバルメモリ閾値の管理をサポートしました（実験的） [#5827](https://github.com/tikv/pd/issues/5827) @[hnes](https://github.com/hnes)
    - GCプレッシャーを緩和するためにGCチューナーを追加（実験的） [#5827](https://github.com/tikv/pd/issues/5827) @[hnes](https://github.com/hnes)
    - 異常ノードを検出およびスケジュールするための`evict-slow-trend-scheduler`スケジューラを追加 [#5808](https://github.com/tikv/pd/pull/5808) @[innerr](https://github.com/innerr)
    - キースペースマネージャーを追加してキースペースを管理する[#5293](https://github.com/tikv/pd/issues/5293) @[AmoebaProtozoa](https://github.com/AmoebaProtozoa)

+ TiFlash

    - 独立したMVCCビットマップフィルタをサポートして、TiFlashデータスキャンプロセスにおけるMVCCフィルタリング操作を分離し、将来のデータスキャンプロセスの最適化の基盤を提供する[#6296](https://github.com/pingcap/tiflash/issues/6296) @[JinheLin](https://github.com/JinheLin)
    - クエリがない場合、TiFlashのメモリ使用量を最大30%削減する[#6589](https://github.com/pingcap/tiflash/pull/6589) @[hongyunyan](https://github.com/hongyunyan)

+ ツール

    + バックアップ＆リストア（BR）

        - 通常のシナリオでのPITRリカバリの性能を向上させるために、TiKV側のログバックアップファイルの並行ダウンロードを最適化する[#14206](https://github.com/tikv/tikv/issues/14206) @[YuJuncen](https://github.com/YuJuncen)

    + TiCDC

        - バッチ`UPDATE` DMLステートメントをサポートして、TiCDCのレプリケーションパフォーマンスを向上させる[#8084](https://github.com/pingcap/tiflow/issues/8084) @[amyangfei](https://github.com/amyangfei)
        - 非同期モードでMQシンクとMySQLシンクを実装し、シンクのスループットを向上させる[#5928](https://github.com/pingcap/tiflow/issues/5928) @[hicqu](https://github.com/hicqu) @[hi-rustin](https://github.com/hi-rustin)

    + TiDBデータ移行（DM）

        - DMアラートルールとコンテンツを最適化する[#7376](https://github.com/pingcap/tiflow/issues/7376) @[D3Hunter](https://github.com/D3Hunter)

             以前は、"DM_XXX_process_exits_with_error"のようなアラートが関連エラーが発生するたびに発生していました。しかし、一部のアラートはアイドル状態のデータベース接続によって引き起こされ、再接続後に回復することができます。この種のアラートを減らすために、DMはエラーを自動的に回復可能なエラーと回復不可能なエラーの2種類に分類します：

            - 自動的に回復可能なエラーの場合、エラーは2分以内に3回以上発生した場合にのみ、DMがアラートを報告します。
            - 自動的に回復不可能なエラーの場合、DMは元の動作を維持し、即座にアラートを報告します。

        - 非同期/バッチリレーライターを追加してリレーパフォーマンスを最適化する[#4287](https://github.com/pingcap/tiflow/issues/4287) @[GMHDBJD](https://github.com/GMHDBJD)

    + TiDB Lightning

        - 物理インポートモードがキースペースをサポートする[#40531](https://github.com/pingcap/tidb/issues/40531) @[iosmanthus](https://github.com/iosmanthus)
        - `lightning.max-error`による最大競合数の設定をサポートする[#40743](https://github.com/pingcap/tidb/issues/40743) @[dsdashun](https://github.com/dsdashun)
        - BOMヘッダを含むCSVデータファイルをインポートする[#40744](https://github.com/pingcap/tidb/issues/40744) @[dsdashun](https://github.com/dsdashun)
        - TiKVのフロー制限エラーに遭遇した場合の処理ロジックを最適化し、他の利用可能なリージョンを試す[#40205](https://github.com/pingcap/tidb/issues/40205) @[lance6716](https://github.com/lance6716)
        - インポート時のテーブル外部キーのチェックを無効にする[#40027](https://github.com/pingcap/tidb/issues/40027) @[sleepymole](https://github.com/sleepymole)

    + Dumpling

        - 外部キーの設定をエクスポートするサポートを追加する[#39913](https://github.com/pingcap/tidb/issues/39913) @[lichunzhu](https://github.com/lichunzhu)

    + sync-diff-inspector

        - 新しいパラメータ`skip-non-existing-table`を追加して、下流のテーブルが上流に存在しない場合に上流と下流のデータの整合性チェックをスキップするかどうかを制御する[#692](https://github.com/pingcap/tidb-tools/issues/692) @[lichunzhu](https://github.com/lichunzhu) @[liumengya94](https://github.com/liumengya94)

## バグ修正

+ TiDB

    - 統計収集タスクが不正な`datetime`値によって失敗する問題を修正する[#39336](https://github.com/pingcap/tidb/issues/39336) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
    - テーブル作成後に`stats_meta`が作成されない問題を修正する[#38189](https://github.com/pingcap/tidb/issues/38189) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
    - DDLデータバックフィル実行時にトランザクションで頻繁な書き込み競合を修正する[#24427](https://github.com/pingcap/tidb/issues/24427) @[mjonss](https://github.com/mjonss)
    - インゲストモードで空のテーブルに対してインデックスを作成できない問題を修正する[#39641](https://github.com/pingcap/tidb/issues/39641) @[tangenta](https://github.com/tangenta)
    - 同一トランザクション内であるが異なるSQLステートメントに対して`wait_ts`が同じになる問題を修正する[#39713](https://github.com/pingcap/tidb/issues/39713) @[TonsnakeLin](https://github.com/TonsnakeLin)
    - 行の削除プロセス中に列を追加する際に`Assertion Failed`エラーが報告される問題を修正する[#39570](https://github.com/pingcap/tidb/issues/39570) @[wjhuang2016](https://github.com/wjhuang2016)
    - 列の自動増加値が枯渇した後に行を挿入してもエラーが報告されない問題を修正する[#38950](https://github.com/pingcap/tidb/issues/38950) @[Dousir9](https://github.com/Dousir9)
    - `INSERT ignore`ステートメントがカラムが書き込み専用の場合にデフォルト値を入力しない問題を修正する[#40192](https://github.com/pingcap/tidb/issues/40192) @[YangKeao](https://github.com/YangKeao)
    - リソース管理モジュールを無効にしたときにリソースが解放されない問題を修正する[#40546](https://github.com/pingcap/tidb/issues/40546) @[zimulala](https://github.com/zimulala)
    - TTLタスクが適切なタイミングで統計更新をトリガーできない問題を修正する[#40109](https://github.com/pingcap/tidb/issues/40109) @[YangKeao](https://github.com/YangKeao)
    - TiDBが`NULL`値を適切に処理せずに予期しないデータを読み取る問題を修正する[#40158](https://github.com/pingcap/tidb/issues/40158) @[tiancaiamao](https://github.com/tiancaiamao)
    - `MODIFT COLUMN`ステートメントが列のデフォルト値も変更する場合に不正な値がテーブルに書き込まれる問題を修正する[#40164](https://github.com/pingcap/tidb/issues/40164) @[wjhuang2016](https://github.com/wjhuang2016)
    - テーブル内に多くのリージョンがある場合に無効なリージョンキャッシュを持つことによってインデックス追加の操作が非効率化される問題を修正する[#38436](https://github.com/pingcap/tidb/issues/38436) @[tangenta](https://github.com/tangenta)
    - 自動インクリメントIDの割り当て中にデータ競合が発生する問題を修正する[#40584](https://github.com/pingcap/tidb/issues/40584) @[Dousir9](https://github.com/Dousir9)
- JSONのNOT演算子の実装がMySQLと互換性がない問題を修正します[#40683](https://github.com/pingcap/tidb/issues/40683) @[YangKeao](https://github.com/YangKeao)
- 同時ビューによってDDL操作がブロックされる可能性がある問題を修正します[#40352](https://github.com/pingcap/tidb/issues/40352) @[zeminzhou](https://github.com/zeminzhou)
- 分割テーブルの列を変更する際に発生するDDL文の並行実行によるデータ不整合を修正します[#40620](https://github.com/pingcap/tidb/issues/40620) @[mjonss](https://github.com/mjonss) @[mjonss](https://github.com/mjonss)
- `caching_sha2_password`を使用してパスワードを指定せずに認証を行うと "Malformed packet" が報告される問題を修正します[#40831](https://github.com/pingcap/tidb/issues/40831) @[dveeden](https://github.com/dveeden)
- テーブルの主キーにENUMカラムが含まれている場合にTTLタスクが失敗する問題を修正します[#40456](https://github.com/pingcap/tidb/issues/40456) @[lcwangchao](https://github.com/lcwangchao)
- MDLによってブロックされた一部のDDL操作が`mysql.tidb_mdl_view`でクエリできない問題を修正します[#40838](https://github.com/pingcap/tidb/issues/40838) @[YangKeao](https://github.com/YangKeao)
- DDLインジェクション中にデータ競合が発生する可能性がある問題を修正します[#40970](https://github.com/pingcap/tidb/issues/40970) @[tangenta](https://github.com/tangenta)
- タイムゾーンが変更された後にTTLタスクが誤って一部のデータを削除する可能性がある問題を修正します[#41043](https://github.com/pingcap/tidb/issues/41043) @[lcwangchao](https://github.com/lcwangchao)
- `JSON_OBJECT`が一部のケースでエラーを報告する可能性がある問題を修正します[#39806](https://github.com/pingcap/tidb/issues/39806) @[YangKeao](https://github.com/YangKeao)
- TiDBの初期化中にデッドロックが発生する可能性がある問題を修正します[#40408](https://github.com/pingcap/tidb/issues/40408) @[Defined2014](https://github.com/Defined2014)
- メモリ再利用によりシステム変数の値が一部のケースで誤って変更される可能性がある問題を修正します[#40979](https://github.com/pingcap/tidb/issues/40979) @[lcwangchao](https://github.com/lcwangchao)
- インジェストモードで一意のインデックスが作成された場合に、インデックスとデータが不整合になる可能性がある問題を修正します[#40464](https://github.com/pingcap/tidb/issues/40464) @[tangenta](https://github.com/tangenta)
- 同時に同じテーブルを切り捨てる場合、一部の切捨て操作がMDLによってブロックされない問題を修正します[#40484](https://github.com/pingcap/tidb/issues/40484) @[wjhuang2016](https://github.com/wjhuang2016)
- `SHOW PRIVILEGES` ステートメントが不完全な権限リストを返す問題を修正します[#40591](https://github.com/pingcap/tidb/issues/40591) @[CbcWestwolf](https://github.com/CbcWestwolf)
- ユニークインデックスを追加する際にTiDBがデッドロックする可能性がある問題を修正します[#40592](https://github.com/pingcap/tidb/issues/40592) @[tangenta](https://github.com/tangenta)
- `ADMIN RECOVER` ステートメントを実行すると、インデックスデータが破損する可能性がある問題を修正します[#40430](https://github.com/pingcap/tidb/issues/40430) @[xiongjiwei](https://github.com/xiongjiwei)
- クエリされたテーブルに`CAST`式が含まれる場合にクエリが失敗する可能性がある問題を修正します[#40130](https://github.com/pingcap/tidb/issues/40130) @[xiongjiwei](https://github.com/xiongjiwei)
- 特定のケースでユニークインデックスが重複データを生成する可能性がある問題を修正します[#40217](https://github.com/pingcap/tidb/issues/40217) @[tangenta](https://github.com/tangenta)
- 複数の地域がある場合に、テーブルIDが`Prepare`または`Execute`を使用して仮想テーブルをクエリする際にPDのOOM問題を修正します[#39605](https://github.com/pingcap/tidb/issues/39605) @[djshow832](https://github.com/djshow832)
- インデックスを追加する際にデータ競合が発生する可能性がある問題を修正します[#40879](https://github.com/pingcap/tidb/issues/40879) @[tangenta](https://github.com/tangenta)
- 仮想カラムによって`can't find proper physical plan`の問題が発生することを修正します[#41014](https://github.com/pingcap/tidb/issues/41014) @[AilinKid](https://github.com/AilinKid)
- 動的整理モードで分割テーブルのためのグローバルバインディングが作成された後にTiDBが再起動できない問題を修正します[#40368](https://github.com/pingcap/tidb/issues/40368) @[Yisaer](https://github.com/Yisaer)
- `auto analyze`によって優雅なシャットダウンに長い時間がかかる問題を修正します[#40038](https://github.com/pingcap/tidb/issues/40038) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
- IndexMergeオペレーターがメモリ制限動作を引き起こすと、TiDBサーバーがパニックする問題を修正します[#41036](https://github.com/pingcap/tidb/pull/41036) @[guo-shaoge](https://github.com/guo-shaoge)
- 分割テーブルでの`SELECT * FROM table_name LIMIT 1`クエリが遅い問題を修正します[#40741](https://github.com/pingcap/tidb/pull/40741) @[solotzg](https://github.com/solotzg)

+ TiKV

  - `const Enum`型を他の型にキャストする際に発生するエラーを修正します[#14156](https://github.com/tikv/tikv/issues/14156) @[wshwsh12](https://github.com/wshwsh12)
  - 解決済みTSによってネットワークトラフィックが増加する問題を修正します[#14092](https://github.com/tikv/tikv/issues/14092) @[overvenus](https://github.com/overvenus)
  - フェイルした悲観的DMLの後にDML実行中にTiDBとTiKVのネットワーク障害によるデータ不整合を修正します[#14038](https://github.com/tikv/tikv/issues/14038) @[MyonKeminta](https://github.com/MyonKeminta)

+ PD
  
  - リージョンスキャッタータスクが予期しないレプリカを生成する問題を修正します[#5909](https://github.com/tikv/pd/issues/5909) @[HundunDM](https://github.com/HunDunDM)
  - オンライン不安全な復元機能が`auto-detect`モードで停止し、タイムアウトする問題を修正します[#5753](https://github.com/tikv/pd/issues/5753) @[Connor1996](https://github.com/Connor1996)
  - `replace-down-peer`の実行が特定の条件下で遅くなる問題を修正します[#5788](https://github.com/tikv/pd/issues/5788) @[HundunDM](https://github.com/HunDunDM)
  - `ReportMinResolvedTS`の呼び出し頻度が高すぎるとPDのOOM問題が発生する問題を修正します[#5965](https://github.com/tikv/pd/issues/5965) @[HundunDM](https://github.com/HunDunDM)

+ TiFlash

  - TiFlash関連のシステムテーブルをクエリするとタイムアウトする問題を修正します[#6745](https://github.com/pingcap/tiflash/pull/6745) @[lidezhu](https://github.com/lidezhu)
  - セミジョインがカルテシアン積を計算する際に過剰なメモリを使用する問題を修正します[#6730](https://github.com/pingcap/tiflash/issues/6730) @[gengliqi](https://github.com/gengliqi)
  - DECIMALデータ型での除算演算結果が丸められない問題を修正します[#6393](https://github.com/pingcap/tiflash/issues/6393) @[LittleFall](https://github.com/LittleFall)
  - `start_ts`がTiFlashクエリでMPPクエリを誤ってキャンセルする問題を修正します[#43426](https://github.com/pingcap/tidb/issues/43426) @[hehechen](https://github.com/hehechen)

+ Tools

  + バックアップとリストア（BR）
- ログバックアップのリストア時に、ホットリージョンが原因でリストアが失敗する問題を修正 [#37207](https://github.com/pingcap/tidb/issues/37207) @[Leavrth](https://github.com/Leavrth)
- ログバックアップが実行されているクラスタにデータをリストアすると、ログバックアップファイルが復旧できなくなる問題を修正 [#40797](https://github.com/pingcap/tidb/issues/40797) @[Leavrth](https://github.com/Leavrth)
- PITR機能がCAバンドルをサポートしていない問題を修正 [#38775](https://github.com/pingcap/tidb/issues/38775) @[YuJuncen](https://github.com/YuJuncen)
- リカバリ中に一時的なテーブルが重複したことによるパニック問題を修正 [#40797](https://github.com/pingcap/tidb/issues/40797) @[joccau](https://github.com/joccau)
- PITRがPDクラスタの構成変更をサポートしていない問題を修正 [#14165](https://github.com/tikv/tikv/issues/14165) @[YuJuncen](https://github.com/YuJuncen)
- PDとtidb-serverの間の接続障害によりPITRバックアップの進行が進まなくなる問題を修正 [#41082](https://github.com/pingcap/tidb/issues/41082) @[YuJuncen](https://github.com/YuJuncen)
- TiKVがPITRタスクをリッスンできなくなる問題を修正 [#14159](https://github.com/tikv/tikv/issues/14159) @[YuJuncen](https://github.com/YuJuncen)
- TiDBクラスタにPITRバックアップタスクがない場合に「ロックの解決」が頻繁に発生する問題を修正 [#40759](https://github.com/pingcap/tidb/issues/40759) @[joccau](https://github.com/joccau)
- PITRバックアップタスクが削除されると、残留するバックアップデータが新しいタスクのデータの整合性に問題を引き起こす問題を修正 [#40403](https://github.com/pingcap/tidb/issues/40403) @[joccau](https://github.com/joccau)

+ TiCDC

  - `transaction_atomicity`および`protocol`を構成ファイル経由で更新できない問題を修正 [#7935](https://github.com/pingcap/tiflow/issues/7935) @[CharlesCheung96](https://github.com/CharlesCheung96)
  - redoログのストレージパスでの事前チェックが実行されない問題を修正 [#6335](https://github.com/pingcap/tiflow/issues/6335) @[CharlesCheung96](https://github.com/CharlesCheung96)
  - S3ストレージの障害に対するredoログの許容できる期間が不十分な問題を修正 [#8089](https://github.com/pingcap/tiflow/issues/8089) @[CharlesCheung96](https://github.com/CharlesCheung96)
  - TiKVまたはTiCDCノードのスケーリングイン/アウト時など、特定のシナリオでchangefeedがスタックする可能性がある問題を修正 [#8174](https://github.com/pingcap/tiflow/issues/8174) @[hicqu](https://github.com/hicqu)
  - TiKVノード間のトラフィックが高すぎる問題を修正 [#14092](https://github.com/tikv/tikv/issues/14092) @[overvenus](https://github.com/overvenus)
  - 押し出し型シンクが有効な場合のTiCDCのCPU使用率、メモリ制御、およびスループットのパフォーマンス問題を修正 [#8142](https://github.com/pingcap/tiflow/issues/8142) [#8157](https://github.com/pingcap/tiflow/issues/8157) [#8001](https://github.com/pingcap/tiflow/issues/8001) [#5928](https://github.com/pingcap/tiflow/issues/5928) @[hicqu](https://github.com/hicqu) @[hi-rustin](https://github.com/hi-rustin)

+ TiDBデータ移行（DM）

  - `binlog-schema delete`コマンドの実行に失敗する問題を修正 [#7373](https://github.com/pingcap/tiflow/issues/7373) @[liumengya94](https://github.com/liumengya94)
  - 最後のbinlogがスキップされたDDLの場合にチェックポイントが進まない問題を修正 [#8175](https://github.com/pingcap/tiflow/issues/8175) @[D3Hunter](https://github.com/D3Hunter)
  - 1つのテーブルで「update」および「non-update」型の式フィルタが指定されている場合に、すべての`UPDATE`ステートメントがスキップされるバグを修正 [#7831](https://github.com/pingcap/tiflow/issues/7831) @[lance6716](https://github.com/lance6716)
  - テーブルごとに`update-old-value-expr`または`update-new-value-expr`のいずれか一方が設定されている場合に、フィルタルールが適用されずDMがパニックするバグを修正 [#7774](https://github.com/pingcap/tiflow/issues/7774) @[lance6716](https://github.com/lance6716)

+ TiDB Lightning

  - 一部のシナリオでTiDB再起動時にTiDB Lightningのタイムアウトが発生する問題を修正 [#33714](https://github.com/pingcap/tidb/issues/33714) @[lichunzhu](https://github.com/lichunzhu)
  - parallel import中にローカルの重複レコードが発生した場合に競合解決が誤ってスキップされる可能性がある問題を修正 [#40923](https://github.com/pingcap/tidb/issues/40923) @[lichunzhu](https://github.com/lichunzhu)
  - ターゲットクラスタで実行中のTiCDCを正確に検出できない事前チェックの問題を修正 [#41040](https://github.com/pingcap/tidb/issues/41040) @[lance6716](https://github.com/lance6716)
  - 分割リージョンフェーズでTiDB Lightningがパニックする問題を修正 [#40934](https://github.com/pingcap/tidb/issues/40934) @[lance6716](https://github.com/lance6716)
  - 競合解決ロジック（`duplicate-resolution`）が不一致のチェックサムを引き起こす可能性がある問題を修正 [#40657](https://github.com/pingcap/tidb/issues/40657) @[sleepymole](https://github.com/sleepymole)
  - データファイルに閉じられていないデリミタが存在する場合に発生するOOM問題を修正 [#40400](https://github.com/pingcap/tidb/issues/40400) @[buchuitoudegou](https://github.com/buchuitoudegou)
  - エラーレポートでのファイルオフセットがファイルサイズを超える問題を修正 [#40034](https://github.com/pingcap/tidb/issues/40034) @[buchuitoudegou](https://github.com/buchuitoudegou)
  - 新しいバージョンのPDClientによりparallel importが失敗する可能性がある問題を修正 [#40493](https://github.com/pingcap/tidb/issues/40493) @[AmoebaProtozoa](https://github.com/AmoebaProtozoa)
  - TiDB Lightningの事前チェックで以前の失敗したインポートによって残された不正なデータを見つけられない問題を修正 [#39477](https://github.com/pingcap/tidb/issues/39477) @[dsdashun](https://github.com/dsdashun)

## 貢献者

TiDBコミュニティから以下の貢献者に感謝します:

- [morgo](https://github.com/morgo)
- [jiyfhust](https://github.com/jiyfhust)
- [b41sh](https://github.com/b41sh)
- [sourcelliu](https://github.com/sourcelliu)
- [songzhibin97](https://github.com/songzhibin97)
- [mamil](https://github.com/mamil)
- [Dousir9](https://github.com/Dousir9)
- [hihihuhu](https://github.com/hihihuhu)
- [mychoxin](https://github.com/mychoxin)
- [xuning97](https://github.com/xuning97)
- [andreid-db](https://github.com/andreid-db)