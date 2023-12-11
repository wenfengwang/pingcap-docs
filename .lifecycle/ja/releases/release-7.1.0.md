---
title: TiDB 7.1.0 のリリースノート
summary: TiDB 7.1.0 の新機能、互換性の変更、改善、およびバグ修正についての情報をご覧ください。

# TiDB 7.1.0 のリリースノート

リリース日: 2023年5月31日

TiDB バージョン: 7.1.0

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v7.1/quick-start-with-tidb) | [本番環境での展開](https://docs.pingcap.com/tidb/v7.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v7.1.0#version-list)

TiDB 7.1.0 は長期サポートリリース（LTS）です。

前の LTS バージョン 6.5.0 と比較して、7.1.0 には[6.6.0-DMR](/releases/release-6.6.0.md)、[7.0.0-DMR](/releases/release-7.0.0.md)でリリースされた新機能、改善、およびバグ修正に加えて、以下の主な機能と改善が含まれています:

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
    <td rowspan="4">拡張性およびパフォーマンス</td>
    <td>TiFlash が<a href="https://docs.pingcap.com/tidb/v7.1/tiflash-disaggregated-and-s3" target="_blank">分離されたストレージとコンピュートアーキテクチャ、および S3 共有ストレージ</a>をサポート（実験的、v7.0.0 で導入）</td>
    <td>TiFlash はクラウドネイティブアーキテクチャをオプションとして導入します:
      <ul>
        <li>TiFlash のコンピュートとストレージを分離し、弾力的な HTAP リソース利用の到達点となります。</li>
        <li>低コストで共有ストレージを提供できる S3 ベースのストレージエンジンを導入します。</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>TiKV が<a href="https://docs.pingcap.com/tidb/v7.1/system-variables#tidb_store_batch_size" target="_blank">バッチ集約データリクエスト</a>をサポート（v6.6.0 で導入）</td>
    <td>この改善により、TiKV バッチ取得操作における総 RPC 数が大幅に減少します。データが非常に分散しており gRPC スレッドプールのリソースが不足している状況では、バッチ処理リクエストによりパフォーマンスが50%以上向上することがあります。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/troubleshoot-hot-spot-issues#scatter-read-hotspots" target="_blank">負荷ベースのレプリカ読み取り</a></td>
    <td>読み取りホットスポットのシナリオでは、TiDB はホットスポットとなっている TiKV ノードの読み取りリクエストをそのレプリカにリダイレクトできます。この機能により読み取りホットスポットを効率的に分散させ、クラスタリソースの利用を最適化できます。負荷ベースのレプリカ読み取りをトリガーするためのしきい値を制御するには、<a href="https://docs.pingcap.com/tidb/v7.1/system-variables#tidb_load_based_replica_read_threshold-new-in-v700" target="_blank"><code>tidb_load_based_replica_read_threshold</code></a>システム変数を調整できます。</td>
  </tr>
  <tr>
      <td>TiKV が<a href="https://docs.pingcap.com/tidb/v7.1/partitioned-raft-kv" target="_blank">パーティション化された Raft KV ストレージエンジン</a>をサポート（実験的）</td>
    <td>TiKV は新しい世代のストレージエンジン、パーティション化された Raft KV を導入します。各データリージョンに専用の RocksDB インスタンスを許可することで、クラスタのストレージ容量を TB レベルから PB レベルに拡張し、より安定した書き込み待ち時間とより強力な拡張性を提供できます。</td>
    </tr>
  <tr>
    <td rowspan="2">信頼性および可用性</td>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/tidb-resource-control" target="_blank">リソースグループによるリソース管理</a>（GA）</td>
   <td>リソースグループに基づいたリソース管理をサポートし、同じクラスタ内の異なるワークロード用にリソースを割り当て、分離します。この機能により、マルチアプリケーションクラスタの安定性が大幅に向上し、マルチテナンシーの基盤が整います。v7.1.0 では、実際のワークロードまたはハードウェア展開に基づいてシステム容量を見積もる機能が導入されています。</td>
  </tr>
  <tr>
    <td>TiFlash が<a href="https://docs.pingcap.com/tidb/v7.1/tiflash-spill-disk" target="_blank">ディスクへのスピル</a>をサポート（v7.0.0 で導入）</td>
    <td>TiFlash は中間結果をディスクにスピルして、集計やソート、ハッシュ結合などのデータ集約処理におけるメモリ不足を緩和できます。</td>
  </tr>
  <tr>
    <td rowspan="3">SQL</td>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/sql-statement-create-index#multi-valued-indexes" target="_blank">マルチバリューインデックス</a>（GA）</td>
    <td>MySQL 互換のマルチバリューインデックスをサポートし、JSON タイプを強化して MySQL 8.0 との互換性を向上させます。この機能により、マルチバリューカラムのメンバーシップチェックの効率が向上します。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/time-to-live" target="_blank">行レベルの TTL</a>（v7.0.0 でGA）</td>
    <td>データベースサイズの管理をサポートし、特定の年齢のデータを自動的に期限切れにすることでパフォーマンスを改善します。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/generated-columns" target="_blank">生成列</a>（GA）</td>
    <td>生成列の値は、列定義内の SQL 式によってリアルタイムに計算されます。この機能により、一部のアプリケーションロジックをデータベースレベルにプッシュして、クエリの効率を向上させます。</td>
  </tr>
  <tr>
    <td rowspan="2">セキュリティ</td>
    <td><a href="https://docs.pingcap.com/tidb/v7.1/security-compatibility-with-mysql" target="_blank">LDAP 認証</a></td>
    <td>TiDB は MySQL 8.0 と互換性のある LDAP 認証をサポートしています。</td>
  </tr>
  <tr>
    <td><a href="https://static.pingcap.com/files/2023/09/18204824/TiDB-Database-Auditing-User-Guide1.pdf" target="_blank">監査ログの拡張</a>（<a href="https://www.pingcap.com/tidb-enterprise" target="_blank">エンタープライズエディション</a>のみ）</td>
    <td>TiDBエンタープライズエディションはデータベース監査機能を強化しました。より細かいイベントフィルタリングコントロール、ユーザーフレンドリーなフィルタ設定、JSON 形式の新しいファイル出力フォーマット、および監査ログのライフサイクル管理を提供することで、システムの監査能力を大幅に向上させます。</td>
  </tr>
</tbody>
</table>

## 機能の詳細

### パフォーマンス

* パーティション化された Raft KV ストレージエンジンを強化（実験的） [#11515](https://github.com/tikv/tikv/issues/11515) [#12842](https://github.com/tikv/tikv/issues/12842) @[busyjay](https://github.com/busyjay) @[tonyxuqqi](https://github.com/tonyxuqqi) @[tabokie](https://github.com/tabokie) @[bufferflies](https://github.com/bufferflies) @[5kbpers](https://github.com/5kbpers) @[SpadeA-Tang](https://github.com/SpadeA-Tang) @[nolouch](https://github.com/nolouch)

    TiDB v6.6.0 では、実験的な機能としてパーティション化された Raft KV ストレージエンジンが導入され、複数の RocksDB インスタンスを使用して TiKV リージョンデータを格納し、各リージョンのデータは独自の RocksDB インスタンスに独立して格納されます。新しいストレージエンジンを使用することで、RocksDB インスタンス内のファイル数とレベルをより効果的に制御し、リージョン間のデータ操作の物理的分離を実現し、より多くのデータを安定して管理できます。元の TiKV ストレージエンジンと比較して、同じハードウェア条件と混合読み書きシナリオの下で、パーティション化された Raft KV ストレージエンジンを使用することで、書き込みスループットを約2倍にし、弾力的なスケーリング時間を約5分の1に短縮できます。

    TiDB v7.1.0 では、パーティション化された Raft KV ストレージエンジンが TiDB Lightning、BR、TiCDC などのツールをサポートします。

    現時点では、この機能は実験的なものであり、本番環境での使用は推奨されません。このエンジンは新たに作成されたクラスタでのみ使用でき、元の TiKV ストレージエンジンから直接アップグレードすることはできません。

    詳細については、[ドキュメント](/partitioned-raft-kv.md)を参照してください。

* [TiFlashは遅延マテリアライゼーション（GA）をサポート](https://github.com/pingcap/tiflash/issues/5829) @ [Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)

    v7.0.0では、TiFlashに遅延マテリアライゼーションが実験機能として導入された。この機能はデフォルトで無効になっており（[`tidb_opt_enable_late_materialization`](/system-variables.md#tidb_opt_enable_late_materialization-new-in-v700) システム変数のデフォルトは`OFF`）、`SELECT`ステートメントのフィルタ条件（`WHERE`句）を処理する際、TiFlashはクエリに必要な列のデータをすべて読み込み、その後クエリ条件に基づいてデータをフィルタリングおよび集計する。遅延マテリアライゼーションが有効な場合、TiDBは一部のフィルタ条件をTableScan演算子にプッシュダウンする。すなわち、TiFlashはまずTableScan演算子にプッシュダウンされたフィルタ条件に関連する列データをスキャンし、条件に一致する行をフィルタリングした後、これらの行の他の列データをスキャンしてさらなる計算を行い、これによりデータ処理のI/Oスキャンおよび計算が低減される。

    v7.1.0から、TiFlashの遅延マテリアライゼーション機能は一般に利用可能となり、デフォルトで有効になる（[`tidb_opt_enable_late_materialization`](/system-variables.md#tidb_opt_enable_late_materialization-new-in-v700) システム変数のデフォルトは`ON`）。TiDBオプティマイザは、統計およびクエリのフィルタ条件に基づいて、TableScan演算子にプッシュダウンするフィルタを決定する。

    詳細については、[ドキュメント](/tiflash/tiflash-late-materialization.md)を参照してください。

* [TiFlashはMPP Joinアルゴリズムの自動選択をサポート](https://github.com/pingcap/tiflash/issues/7084) @ [solotzg](https://github.com/solotzg)

    TiFlashのMPPモードは複数のJoinアルゴリズムをサポートしている。v7.1.0以前では、TiDBはMPPモードがブロードキャストハッシュジョインアルゴリズムを使用するかどうかを、[`tidb_broadcast_join_threshold_count`](/system-variables.md#tidb_broadcast_join_threshold_count-new-in-v50)および[`tidb_broadcast_join_threshold_size`](/system-variables.md#tidb_broadcast_join_threshold_size-new-in-v50)変数および実際のデータ量に基づいて決定していた。

    v7.1.0では、TiDBは、[`tidb_prefer_broadcast_join_by_exchange_data_size`](/system-variables.md#tidb_prefer_broadcast_join_by_exchange_data_size-new-in-v710)変数を導入し、ネットワーク転送のオーバーヘッドを最小限に抑えるためにMPP Joinアルゴリズムを選択するかどうかを制御するようになった。この変数はデフォルトで無効になっており、デフォルトのアルゴリズム選択手法はv7.1.0以前と同じままである。この変数を有効にするには、`ON`に設定する。有効になると、[`tidb_broadcast_join_threshold_count`](/system-variables.md#tidb_broadcast_join_threshold_count-new-in-v50)および[`tidb_broadcast_join_threshold_size`](/system-variables.md#tidb_broadcast_join_threshold_size-new-in-v50)変数を手動で調整する必要がなくなり（これらの変数はこの時点では効果を持たない）、TiDBは自動的に異なるJoinアルゴリズムによるネットワーク転送の閾値を推定し、全体的にオーバーヘッドが最小のアルゴリズムを選択するため、ネットワークトラフィックを削減し、MPPクエリのパフォーマンスを向上させる。

    詳細については、[ドキュメント](/tiflash/use-tiflash-mpp-mode.md#algorithm-support-for-the-mpp-mode)を参照してください。

* リードホットスポットを緩和するための負荷ベースのレプリカリードをサポート [#14151](https://github.com/tikv/tikv/issues/14151) @[sticnarf](https://github.com/sticnarf) @[you06](https://github.com/you06)

    リードホットスポットのシナリオでは、ホットスポットになっているTiKVノードは読み込みリクエストを時間内に処理することができず、リクエストが待ち行列に入る。しかし、この時点でTiKVリソースがすべて枯渇しているわけではない。応答時間を短縮するため、TiDB v7.1.0では負荷ベースのレプリカリード機能を導入し、ホットスポットになっているTiKVノードで待ち行列にならずに他のTiKVノードからデータを読み込むことができるようになる。[`tidb_load_based_replica_read_threshold`](/system-variables.md#tidb_load_based_replica_read_threshold-new-in-v700) システム変数を使用して読み込みリクエストの待ち行列長を制御することができる。リーダーノードの推定待ち時間がこの閾値を超えた場合、TiDBはフォロワーノードからデータを読み込むことを優先する。この機能により、読み込みホットスポットを散乱させない場合に、リードスループットは通常の状況と比較して70％から200％向上する。

    詳細については、[ドキュメント](/troubleshoot-hot-spot-issues.md#scatter-read-hotspots)を参照してください。

* プリペアされていないステートメント用の実行計画のキャッシュ機能を強化（実験機能） [#36598](https://github.com/pingcap/tidb/issues/36598) @[qw4990](https://github.com/qw4990)

    TiDB v7.0.0では、コンカレントOLTPの負荷容量を改善するために、プリペアされていない計画キャッシュを実験機能として導入した。v7.1.0では、この機能を強化し、より多くのSQLステートメントをキャッシュできるようになった。

    メモリ使用量を改善するため、TiDB v7.1.0は非プリペアおよびプリペアされた計画キャッシュのキャッシュプールをマージした。システム変数[`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710) を使用してキャッシュサイズを制御することができる。[`tidb_prepared_plan_cache_size`](/system-variables.md#tidb_prepared_plan_cache_size-new-in-v610)および[`tidb_non_prepared_plan_cache_size`](/system-variables.md#tidb_non_prepared_plan_cache_size) システム変数は非推奨となった。

    将来の互換性を維持するため、以前のバージョンからv7.1.0またはそれ以降にアップグレードする際には、キャッシュサイズ`tidb_session_plan_cache_size` は `tidb_prepared_plan_cache_size` の値と同じになり、[`tidb_enable_non_prepared_plan_cache`](/system-variables.md#tidb_enable_non_prepared_plan_cache) はアップグレード前の設定のままとなる。十分なパフォーマンステストを行った後に、`tidb_enable_non_prepared_plan_cache` を使用して非プリペアされた計画キャッシュを有効にすることができる。新規作成のクラスターでは、非プリペアされた計画キャッシュはデフォルトで有効になっている。

    DMLステートメントをデフォルトでサポートしていない非プリペアされた計画キャッシュ。この制限を解除するためには、[`tidb_enable_non_prepared_plan_cache_for_dml`](/system-variables.md#tidb_enable_non_prepared_plan_cache_for_dml-new-in-v710) システム変数を `ON` に設定する。

    詳細については、[ドキュメント](/sql-non-prepared-plan-cache.md)を参照してください。

* [DDLの分散並列実行フレームワーク（実験機能）をサポート](https://github.com/pingcap/tidb/issues/41495) @ [benjamin2037](https://github.com/benjamin2037)

    TiDB v7.1.0以前では、1つのTiDBノードしかDDLオーナーとして同時にDDLタスクを実行することができなかった。TiDB v7.1.0からは、新しい分散並列実行フレームワークにおいて、複数のTiDBノードが並列で同じDDLタスクを実行することができるようになり、これによりTiDBクラスターのリソースを効果的に活用し、DDLのパフォーマンスを大幅に向上させることができる。また、TiDBノードを追加することでDDLのパフォーマンスを線形に向上させることができる。ただし、この機能は現在実験的なものであり、`ADD INDEX`操作のみをサポートしている。

    分散フレームワークを使用するには、[`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710) の値を `ON` に設定してください：

    ```sql
    SET GLOBAL tidb_enable_dist_task = ON;
    ```

    詳細については、[ドキュメント](/tidb-distributed-execution-framework.md)を参照してください。

###信頼性

* リソースコントロールが一般利用可能（GA）になりました [#38825](https://github.com/pingcap/tidb/issues/38825) @[nolouch](https://github.com/nolouch) @[BornChanger](https://github.com/BornChanger) @[glorv](https://github.com/glorv) @[tiancaiamao](https://github.com/tiancaiamao) @[Connor1996](https://github.com/Connor1996) @[JmPotato](https://github.com/JmPotato) @[hnes](https://github.com/hnes) @[CabinfeverB](https://github.com/CabinfeverB) @[HuSharp](https://github.com/HuSharp)

    TiDBはリソースグループに基づくリソースコントロール機能を強化し、v7.1.0で一般利用可能となった。この機能により、TiDBクラスターのリソース利用効率とパフォーマンスが大幅に向上した。リソースコントロール機能の導入はTiDBにとって重要なマイルストーンである。分散データベースクラスターを複数の論理単位に分割し、異なるデータベースユーザをそれぞれのリソースグループにマッピングし、必要に応じて各リソースグループのクォータを設定することができる。クラスターリソースが限られている場合、同じリソースグループ内のセッションによって使用されるリソースはすべてクォータに制限される。このようにして、リソースグループが過剰に消費されても、他のリソースグループ内のセッションに影響を与えない。
この機能を使用すると、さまざまなシステムからの複数の小規模および中規模のアプリケーションを1つのTiDBクラスタに組み合わせることができます。アプリケーションのワークロードが増加しても、他のアプリケーションの正常な動作に影響を与えません。システムのワークロードが低い場合でも、設定されたクォータを超えている場合でも、ビジーアプリケーションに必要なシステムリソースを割り当てることができ、リソースの最大利用を実現することができます。さらに、リソース制御機能を合理的に使用することで、クラスタの数を減らし、運用および保守の難しさを軽減し、管理コストを削減することができます。

TiDB v7.1.0では、この機能により、実際のワークロードまたはハードウェアデプロイメントに基づいてシステム容量を推定する機能が導入されています。推定能力により、容量の計画により正確な参照が提供され、エンタープライズレベルのシナリオの安定したニーズを満たすためにTiDBリソースの割り当てをよりよく管理することができます。

ユーザーエクスペリエンスを向上させるために、TiDBダッシュボードは[リソースマネージャページ](/dashboard/dashboard-resource-manager.md)を提供しています。このページでリソースグループ構成を表示し、クラスタ容量を視覚的な方法で推定し、合理的なリソース割り当てを容易にすることができます。

詳細については、[ドキュメント](/tidb-resource-control.md)を参照してください。

* ファストオンラインDDLのチェックポイントメカニズムをサポートして、耐障害性と自動回復能力を向上させる[#42164](https://github.com/pingcap/tidb/issues/42164) @[tangenta](https://github.com/tangenta)

    TiDB v7.1.0では、[ファストオンラインDDL](/ddl-introduction.md)にチェックポイントメカニズムを導入し、ファストオンラインDDLの耐障害性と自動回復能力を大幅に向上させました。TiDBオーナーノードが障害によって再起動されたり変更されたりしても、TiDBは定期的に自動更新されるチェックポイントから進行状況を復旧できるため、DDLの実行がより安定し、効率的になります。

    詳細については、[ドキュメント](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)を参照してください。

* チェックポイントリストアをサポートするバックアップ＆リストア[#42339](https://github.com/pingcap/tidb/issues/42339) @[Leavrth](https://github.com/Leavrth)

    スナップショットリストアまたはログリストアは、ディスクの枯渇やノードのクラッシュなどの回復可能なエラーによって中断される可能性があります。TiDB v7.1.0より前では、中断前の復旧進行が無効になり、エラーが解消された後でもリストアを最初からやり直す必要がありました。大規模なクラスタでは、追加のコストがかかります。

    TiDB v7.1.0から、バックアップ＆リストア（BR）はチェックポイントリストア機能を導入し、中断されたリストアを継続できるようになりました。この機能により、中断されたリストアのほとんどの復旧進行が保持されることができます。

    詳細については、[ドキュメント](/br/br-checkpoint-restore.md)を参照してください。

* 統計情報のロード戦略を最適化[#42160](https://github.com/pingcap/tidb/issues/42160) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)

    TiDB v7.1.0では、実験的な機能として軽量統計情報初期化を導入しました。軽量統計情報初期化は、起動時に読み込む必要がある統計情報の数を大幅に減らすことができ、そのため統計情報のロード速度が向上します。この機能により、TiDBの安定性が複雑なランタイム環境で向上し、TiDBノードが再起動した場合の全体のサービスへの影響が軽減されます。この機能を有効にするには、パラメータ[`lite-init-stats`](/tidb-configuration-file.md#lite-init-stats-new-in-v710)を`true`に設定できます。

    TiDBの起動中、初期統計情報が完全に読み込まれる前に実行されたSQL文は、最適でない実行計画を持つ場合があり、パフォーマンスの問題を引き起こす可能性があります。そのような問題を回避するために、TiDB v7.1.0では、設定パラメータ[`force-init-stats`](/tidb-configuration-file.md#force-init-stats-new-in-v710)を導入しました。このオプションを使用すると、起動中に統計情報の初期化が完了した後にのみTiDBがサービスを提供するかどうかを制御できます。このパラメータはデフォルトで無効になっています。

    詳細については、[ドキュメント](/statistics.md#load-statistics)を参照してください。

* TiCDCは、単一行データのデータ完全性検証機能をサポート[#8718](https://github.com/pingcap/tiflow/issues/8718) [#42747](https://github.com/pingcap/tidb/issues/42747)@[3AceShowHand](https://github.com/3AceShowHand) @[zyguan](https://github.com/zyguan)

    v7.1.0から、TiCDCはデータ完全性検証機能を導入し、チェックサムアルゴリズムを使用して単一行データの整合性を検証します。この機能により、TiDBからのデータの書き込み、TiCDCを介したレプリケーション、そしてKafkaクラスタへの書き込みの過程でエラーが発生していないかどうかを確認できます。データ完全性検証機能は、Kafkaをダウンストリームとして使用するチェンジフィードのみをサポートし、現在はAvroプロトコルをサポートしています。

    詳細については、[ドキュメント](/ticdc/ticdc-integrity-check.md)を参照してください。

* TiCDCはDDLレプリケーション操作を最適化[#8686](https://github.com/pingcap/tiflow/issues/8686) @[hi-rustin](https://github.com/hi-rustin)

    v7.1.0以前は、大規模なテーブルのすべての行に影響を与えるDDL操作（列の追加または削除など）を行うと、TiCDCのレプリケーション遅延が著しく増加しました。v7.1.0から、TiCDCはこのレプリケーション操作を最適化し、DDL操作がダウンストリームの遅延に与える影響を軽減します。

    詳細については、[ドキュメント](/ticdc/ticdc-faq.md#does-ticdc-replicate-data-changes-caused-by-lossy-ddl-operations-to-the-downstream)を参照してください。

* TiDB LightningのTiBレベルデータのインポート時の安定性を向上[#43510](https://github.com/pingcap/tidb/issues/43510) [#43657](https://github.com/pingcap/tidb/issues/43657)@[D3Hunter](https://github.com/D3Hunter) @[lance6716](https://github.com/lance6716)

    v7.1.0から、TiDB LightningはTiBレベルのデータをインポートする際の安定性を向上するために、4つの構成項目を追加しました。

    - `tikv-importer.region-split-batch-size`は、バッチでRegionを分割するときのRegionの数を制御します。デフォルト値は`4096`です。
    - `tikv-importer.region-split-concurrency`は、Regionを分割するときの並行性を制御します。デフォルト値はCPUコア数です。
    - `tikv-importer.region-check-backoff-limit`は、Regionの分割とスキャッタ操作の後にRegionがオンラインになるまでのリトライ回数を制御します。デフォルト値は`1800`で、最大リトライ間隔は2秒です。リトライ回数は、リトライ間にRegionがオンラインになった場合、増加しません。
    - `tikv-importer.pause-pd-scheduler-scope`は、TiDB LightningがPDスケジューリングを一時停止するスコープを制御します。値のオプションは`"table"`と`"global"`です。デフォルト値は`"table"`です。v6.1.0より前のTiDBバージョンでは、データインポート中にグローバルスケジューリングを一時停止するために`"global"`オプションのみを設定できます。v6.1.0以降では、`"table"`オプションがサポートされるようになったため、スケジューリングはターゲットテーブルデータを保存するRegionのみに一時停止されます。大容量のデータのシナリオでは、この構成項目を`"global"`に設定することをお勧めします。

    詳細については、[ドキュメント](/tidb-lightning/tidb-lightning-configuration.md)を参照してください。

### SQL

* `INSERT INTO SELECT`ステートメントを使用してTiFlashクエリの結果を保存する機能をサポート(GA)[#37515](https://github.com/pingcap/tidb/issues/37515)@[gengliqi](https://github.com/gengliqi)

    v6.5.0から、TiDBは`INSERT INTO SELECT`ステートメントの`SELECT`句（解析クエリ）をTiFlashにプッシュダウンする機能をサポートしています。これにより、TiFlashクエリの結果を`INSERT INTO`で指定されたTiDBテーブルに簡単に保存して、さらなる解析のために利用することができます。これは結果のキャッシュ効果（つまり、結果の具現化）として機能します。

    v7.1.0では、この機能が一般に利用可能になりました。`INSERT INTO SELECT`ステートメントの実行中に、オプティマイザはTiFlashレプリカのコスト推定と[SQLモード](/sql-mode.md)に基づいてクエリをTiFlashにプッシュダウンするかどうかを適切に決定できます。したがって、実験的な段階で導入された`tidb_enable_tiflash_read_for_write_stmt`システム変数は廃止されました。`INSERT INTO SELECT`ステートメントの計算規則は`STRICT SQL Mode`の要件を満たさないため、TiDBは、現在のセッションの[SQLモード](/sql-mode.md)が厳格でない（つまり、`sql_mode`値が`STRICT_TRANS_TABLES`と`STRICT_ALL_TABLES`を含まない）場合にのみ`INSERT INTO SELECT`ステートメントの`SELECT`句をTiFlashにプッシュダウンすることを許可しています。

    詳細については、[ドキュメント](/tiflash/tiflash-results-materialization.md)を参照してください。

* MySQL互換の多値インデックスが一般に利用可能になりました(GA)[#39592](https://github.com/pingcap/tidb/issues/39592) @[xiongjiwei](https://github.com/xiongjiwei) @[qw4990](https://github.com/qw4990) @[YangKeao](https://github.com/YangKeao)
    JSONカラム内の配列値のフィルタリングは一般的な操作ですが、通常のインデックスではこのような操作の速度向上に役立ちません。配列に多値インデックスを作成すると、フィルタリングのパフォーマンスが大幅に向上します。JSONカラム内の配列に多値インデックスがある場合、`MEMBER OF()`、`JSON_CONTAINS()`、`JSON_OVERLAPS()`関数でフィルタリング条件を取得することができます。これによりI/Oの消費が減少し、操作速度が向上します。

v7.1.0では、多値インデックス機能が一般的に利用可能（GA）になります。この機能は、より完全なデータ型をサポートし、TiDBツールと互換性があります。本番環境でのJSON配列の検索操作を高速化するために多値インデックスを使用できます。

詳細については、[ドキュメント](/sql-statements/sql-statement-create-index.md#multi-valued-indexes)を参照してください。

* ハッシュとキー分割テーブルのパーティション管理を改善 [#42728](https://github.com/pingcap/tidb/issues/42728) @[mjonss](https://github.com/mjonss)

    v7.1.0以前では、TiDBのハッシュおよびキー分割テーブルは`TRUNCATE PARTITION`パーティション管理ステートメントのみをサポートしていました。v7.1.0以降、ハッシュおよびキー分割テーブルは`ADD PARTITION`および`COALESCE PARTITION`パーティション管理ステートメントもサポートします。そのため、必要に応じてハッシュおよびキー分割テーブルのパーティション数を柔軟に調整できます。たとえば、`ADD PARTITION`ステートメントでパーティション数を増やしたり、`COALESCE PARTITION`ステートメントでパーティション数を減らしたりできます。

詳細については、[ドキュメント](/partitioned-table.md#manage-hash-and-key-partitions)を参照してください。

* Range INTERVALパーティショニングの構文が一般的に利用可能になりました (GA) [#35683](https://github.com/pingcap/tidb/issues/35683) @[mjonss](https://github.com/mjonss)

    Range INTERVALパーティショニングの構文（v6.3.0で導入）がGAになりました。この構文を使用すると、すべてのパーティションを列挙することなく、所望の間隔でRangeパーティショニングを定義できます。これにより、RangeパーティショニングDDLステートメントの長さが劇的に短縮されます。この構文は元のRangeパーティショニングと同等です。

詳細については、[ドキュメント](/partitioned-table.md#range-interval-partitioning)を参照してください。

* 生成列が一般的に利用可能になりました (GA) @[bb7133](https://github.com/bb7133)

    生成列はデータベースの貴重な機能です。テーブルを作成する際、他のカラムの値に基づいてカラムの値を計算することができます。この方法で、ユーザーが明示的に挿入または更新するのではなく、生成列の値を定義できます。生成列は仮想カラムまたは格納されたカラムのいずれかにすることができます。TiDBは以前のバージョンからMySQL互換の生成列をサポートしており、この機能はv7.1.0でGAになります。

生成列を使用することで、TiDBのMySQL互換性が向上し、MySQLからの移行プロセスが簡素化されます。さらに、データの保守の複雑さが低減し、データの整合性とクエリの効率が向上します。

詳細については、[ドキュメント](/generated-columns.md)を参照してください。

### DB操作

* 手動でDDL操作をキャンセルせずに滑らかなクラスタのアップグレードをサポート (実験的) [#39751](https://github.com/pingcap/tidb/issues/39751) @[zimulala](https://github.com/zimulala)

    TiDB v7.1.0以前では、クラスタをアップグレードするためには、アップグレード前に実行中またはキューに入れたDDLタスクを手動でキャンセルし、アップグレード後に再度追加する必要がありました。

    アップグレード体験を向上するために、TiDB v7.1.0では自動的にDDLタスクの一時停止と再開をサポートしています。v7.1.0以降、手動でDDLタスクを事前にキャンセルせずにクラスタをアップグレードできます。アップグレード前にTiDBは自動的に実行中またはキューに入れたユーザーDDLタスクを一時停止し、ローリングアップグレード後これらのタスクを再開します。これにより、TiDBクラスタのアップグレードが簡単になります。

詳細については、[ドキュメント](/smooth-upgrade-tidb.md)を参照してください。

### 観測可能性

* オプティマイザ診断情報の強化 [#43122](https://github.com/pingcap/tidb/issues/43122) @[time-and-fate](https://github.com/time-and-fate)

    十分な情報を取得することはSQLパフォーマンス診断の鍵です。v7.1.0では、TiDBはさまざまな診断ツールにオプティマイザの実行時情報を追加し、実行計画の選択方法に関するより良い洞察を提供し、SQLパフォーマンスの問題のトラブルシューティングを支援します。新しい情報は以下のとおりです。

    * [`PLAN REPLAYER`](/sql-plan-replayer.md)の出力に`debug_trace.json`が含まれます。
    * [`EXPLAIN`](/explain-walkthrough.md)の出力に`operator info`の部分統計の詳細が含まれます。
    * 遅いクエリの`Stats`フィールドに部分統計の詳細が含まれます。

  詳細については、[`PLAN REPLAYER`を使用してクラスタの現場情報を保存および復元](/sql-plan-replayer.md)、[`EXPLAIN`の解説](/explain-walkthrough.md)、および[遅いクエリの識別](/identify-slow-queries.md)を参照してください。

### セキュリティ

* TiFlashシステムテーブル情報のクエリに使用されるインターフェースを置換 [#6941](https://github.com/pingcap/tiflash/issues/6941) @[flowbehappy](https://github.com/flowbehappy)

    v7.1.0以降、TiDBの[`INFORMATION_SCHEMA.TIFLASH_TABLES`](/information-schema/information-schema-tiflash-tables.md)および[`INFORMATION_SCHEMA.TIFLASH_SEGMENTS`](/information-schema/information-schema-tiflash-segments.md)システムテーブルのクエリサービスは、TiFlashがHTTPポートの代わりにgRPCポートを使用し、HTTPサービスのセキュリティリスクを回避します。

* LDAP認証のサポート [#43580](https://github.com/pingcap/tidb/issues/43580) @[YangKeao](https://github.com/YangKeao)

    v7.1.0以降、TiDBはLDAP認証をサポートし、`authentication_ldap_sasl`および`authentication_ldap_simple`の2つの認証プラグインを提供します。

詳細については、[ドキュメント](/security-compatibility-with-mysql.md)を参照してください。

* データベース監査機能の強化（エンタープライズエディション）

    v7.1.0では、TiDBエンタープライズエディションはデータベース監査機能を強化し、その容量を大幅に拡張し、ユーザーエクスペリエンスを向上させ、企業のデータベースセキュリティコンプライアンスのニーズを満たします。

    - より細かい監査イベントの定義とより細かい監査設定のための「フィルター」と「ルール」という概念を導入します。
    - より使いやすい構成方法を提供するため、JSON形式でルールを定義できます。
    - 自動ログローテーションとスペース管理機能を追加し、保存期間とログサイズの両方のログローテーションをサポートします。
    - 監査ログをTEXTおよびJSON形式で出力します。これにより、サードパーティツールとの統合が容易になります。
    - 監査ログの隠蔽をサポートします。すべてのリテラルを置換してセキュリティを強化できます。

  データベース監査はTiDBエンタープライズエディションの重要な機能です。この機能は、企業のデータセキュリティとコンプライアンスの要件を満たすために、強力な監視および監査ツールを提供します。エンタープライズマネージャは、データベース操作のソースと影響を追跡して違法なデータ盗難や改ざんを防ぐのに役立ちます。さらに、データベース監査は様々な規制およびコンプライアンス要件を満たすために企業を支援し、合法的かつ倫理的なコンプライアンスを確保します。この機能は企業情報セキュリティに重要な価値を持ちます。

  詳細については、[ユーザーガイド](https://static.pingcap.com/files/2023/09/18204824/TiDB-Database-Auditing-User-Guide1.pdf)を参照してください。この機能はTiDBエンタープライズエディションに含まれています。この機能を使用するには、[TiDBエンタープライズ](https://www.pingcap.com/tidb-enterprise)ページにアクセスしてTiDBエンタープライズエディションを入手してください。

## 互換性の変更

> **注意:**
>
> このセクションは、v7.0.0から現在のバージョン（v7.1.0）にアップグレードする際に把握する必要がある互換性の変更を提供します。v6.6.0またはそれ以前のバージョンから現在のバージョンにアップグレードする場合、中間バージョンで導入された互換性の変更も確認する必要があります。

### 動作変更

* セキュリティの向上のため、TiFlashはHTTPサービスポート（デフォルトは`8123`）を非推奨にし、代わりにgRPCポートを使用します。

    TiFlashをv7.1.0にアップグレードした場合、TiDBがv7.1.0にアップグレードされる際、TiDBはTiFlashシステムテーブル（[`INFORMATION_SCHEMA.TIFLASH_TABLES`](/information-schema/information-schema-tiflash-tables.md)および[`INFORMATION_SCHEMA.TIFLASH_SEGMENTS`](/information-schema/information-schema-tiflash-segments.md)）を読み取ることができません。
* TiDB Lightning は、TiDB クラスターのバージョンに基づいて、v6.2.0 から v7.0.0 の TiDB バージョンでグローバル スケジューリングの一時停止を決定します。TiDB クラスター バージョンが v6.1.0 以上の場合、スケジューリングは対象のテーブル データを格納している Region に対してのみ一時停止され、対象のテーブルのインポートが完了した後に再開されます。その他のバージョンの場合、TiDB Lightning はグローバル スケジューリングを一時停止します。TiDB v7.1.0 からは、[`pause-pd-scheduler-scope`](/tidb-lightning/tidb-lightning-configuration.md) を構成して、グローバル スケジューリングの一時停止を制御できます。デフォルトでは、TiDB Lightning は対象のテーブル データを格納する Region に対してスケジューリングを一時停止します。対象のクラスターのバージョンが v6.1.0 よりも古い場合は、エラーが発生します。この場合、パラメーターの値を `"global"` に変更して再試行できます。

* TiDB v7.1.0 で [`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md) を使用する場合、一部のリージョンが FLASHBACK プロセスの完了後も残る場合があります。v7.1.0 でこの機能の使用を避けることをお勧めします。詳細については、issue [#44292](https://github.com/pingcap/tidb/issues/44292) を参照してください。この問題が発生した場合は、[TiDB スナップショット バックアップとリストア](/br/br-snapshot-guide.md) 機能を使用してデータをリストアすることができます。

### システム変数

| 変数名 | 変更タイプ | 説明 |
|--------|------------------------------|------|
| [`tidb_enable_tiflash_read_for_write_stmt`](/system-variables.md#tidb_enable_tiflash_read_for_write_stmt-new-in-v630) | 廃止 | デフォルト値を `OFF` から `ON` に変更します。[`tidb_allow_mpp = ON`](/system-variables.md#tidb_allow_mpp-new-in-v50) の場合、最適化プログラムは [SQL モード](/sql-mode.md) と TiFlash レプリカのコスト見積もりに基づいてクエリを TiFlash にプッシュするかどうかを賢明に決定します。 |
| [`tidb_non_prepared_plan_cache_size`](/system-variables.md#tidb_non_prepared_plan_cache_size) | 廃止 | v7.1.0 から、このシステム変数は廃止されます。 [`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710) を使用して、キャッシュできるプランの最大数を制御できます。 |
| [`tidb_prepared_plan_cache_size`](/system-variables.md#tidb_prepared_plan_cache_size-new-in-v610) | 廃止 | v7.1.0 から、このシステム変数は廃止されます。 [`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710) を使用して、キャッシュできるプランの最大数を制御できます。 |
| `tidb_ddl_distribute_reorg` | 削除 | この変数は [`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710) に名前が変更されました。 |
| [`default_authentication_plugin`](/system-variables.md#default_authentication_plugin) | 変更 | `authentication_ldap_sasl` と `authentication_ldap_simple` の 2 つの新しい値オプションが導入されました。 |
| [`tidb_load_based_replica_read_threshold`](/system-variables.md#tidb_load_based_replica_read_threshold-new-in-v700) | 変更 | v7.1.0 から有効となり、負荷ベースのレプリカ リードをトリガーする閾値を制御します。さらなるテストの後、デフォルト値を `"0s"` から `"1s"` に変更します。 |
| [`tidb_opt_enable_late_materialization`](/system-variables.md#tidb_opt_enable_late_materialization-new-in-v700) | 変更 | デフォルト値を `OFF` から `ON` に変更し、TiFlash の遅延マテリアリゼーション機能をデフォルトで有効にします。 |
| [`authentication_ldap_sasl_auth_method_name`](/system-variables.md#authentication_ldap_sasl_auth_method_name-new-in-v710) | 新規追加 | LDAP SASL 認証の認証メソッド名を指定します。 |
| [`authentication_ldap_sasl_bind_base_dn`](/system-variables.md#authentication_ldap_sasl_bind_base_dn-new-in-v710) | 新規追加 | LDAP SASL 認証で、検索の範囲を制限します。`AS ...` 句なしでユーザーが作成された場合、TiDB はユーザー名に応じて LDAP サーバー内で `dn` を自動的に検索します。 |
| [`authentication_ldap_sasl_bind_root_dn`](/system-variables.md#authentication_ldap_sasl_bind_root_dn-new-in-v710) | 新規追加 | LDAP SASL 認証で、LDAP サーバーにログインするために使用する `dn` を指定します。 |
| [`authentication_ldap_sasl_bind_root_pwd`](/system-variables.md#authentication_ldap_sasl_bind_root_pwd-new-in-v710) | 新規追加 | LDAP SASL 認証で、LDAP サーバーにユーザーを検索するために使用するパスワードを指定します。 |
| [`authentication_ldap_sasl_ca_path`](/system-variables.md#authentication_ldap_sasl_ca_path-new-in-v710) | 新規追加 | LDAP SASL 認証で、StartTLS 接続用の証明機関ファイルの絶対パスを指定します。 |
| [`authentication_ldap_sasl_init_pool_size`](/system-variables.md#authentication_ldap_sasl_init_pool_size-new-in-v710) | 新規追加 | LDAP SASL 認証で、LDAP サーバーへの接続プールの初期接続数を指定します。 |
| [`authentication_ldap_sasl_max_pool_size`](/system-variables.md#authentication_ldap_sasl_max_pool_size-new-in-v710) | 新規追加 | LDAP SASL 認証で、LDAP サーバーへの接続プールの最大接続数を指定します。 |
| [`authentication_ldap_sasl_server_host`](/system-variables.md#authentication_ldap_sasl_server_host-new-in-v710) | 新規追加 | LDAP SASL 認証で、LDAP サーバーのホストを指定します。 |
| [`authentication_ldap_sasl_server_port`](/system-variables.md#authentication_ldap_sasl_server_port-new-in-v710) | 新規追加 | LDAP SASL 認証で、LDAP サーバーの TCP/IP ポート番号を指定します。 |
| [`authentication_ldap_sasl_tls`](/system-variables.md#authentication_ldap_sasl_tls-new-in-v710) | 新規追加 | LDAP SASL 認証で、プラグインによるLDAP サーバーへの接続が StartTLS で保護されるかどうかを指定します。 |
| [`authentication_ldap_simple_auth_method_name`](/system-variables.md#authentication_ldap_simple_auth_method_name-new-in-v710) | 新規追加 | LDAP シンプル認証での認証方法名を指定します。`SIMPLE` のみをサポートします。 |
| [`authentication_ldap_simple_bind_base_dn`](/system-variables.md#authentication_ldap_simple_bind_base_dn-new-in-v710) | 新規追加 | LDAP シンプル認証で、検索の範囲を制限します。`AS ...` 句なしでユーザーが作成された場合、TiDB はユーザー名に応じて LDAP サーバー内で `dn` を自動的に検索します。 |
| [`authentication_ldap_simple_bind_root_dn`](/system-variables.md#authentication_ldap_simple_bind_root_dn-new-in-v710) | 新規追加 | LDAP シンプル認証で、LDAP サーバーにログインするために使用する `dn` を指定します。 |
| [`authentication_ldap_simple_bind_root_pwd`](/system-variables.md#authentication_ldap_simple_bind_root_pwd-new-in-v710) | 新規追加 | LDAP シンプル認証で、LDAP サーバーにユーザーを検索するために使用するパスワードを指定します。 |
| [`authentication_ldap_simple_ca_path`](/system-variables.md#authentication_ldap_simple_ca_path-new-in-v710) | 新規追加 | LDAP シンプル認証で、StartTLS 接続用の証明機関ファイルの絶対パスを指定します。 |
| [`authentication_ldap_simple_init_pool_size`](/system-variables.md#authentication_ldap_simple_init_pool_size-new-in-v710) | 新規追加 | LDAP シンプル認証で、LDAP サーバーへの接続プールの初期接続数を指定します。 |
| [`authentication_ldap_simple_max_pool_size`](/system-variables.md#authentication_ldap_simple_max_pool_size-new-in-v710) | 新規追加 | LDAP シンプル認証で、LDAP サーバーへの接続プールの最大接続数を指定します。 |
| [`authentication_ldap_simple_server_host`](/system-variables.md#authentication_ldap_simple_server_host-new-in-v710) | 新規追加 | LDAP シンプル認証で、LDAP サーバーのホストを指定します。 |
| [`authentication_ldap_simple_server_port`](/system-variables.md#authentication_ldap_simple_server_port-new-in-v710) | 新規追加 | LDAP シンプル認証で、LDAP サーバーの TCP/IP ポート番号を指定します。 |
| [`authentication_ldap_simple_tls`](/system-variables.md#authentication_ldap_simple_tls-new-in-v710) | 新規追加 | LDAP シンプル認証で、プラグインによる LDAP サーバーへの接続が StartTLS で保護されるかどうかを指定します。 |
| [`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710) | 新規追加 | 分散実行フレームワークを有効にするかどうかを制御します。分散実行を有効にした後、DDL、インポート、およびその他のサポートされるバックエンド タスクは、クラスター内の複数の TiDB ノードによって共同で完了されます。この変数は `tidb_ddl_distribute_reorg` から名前が変更されました。 |
| [`tidb_enable_non_prepared_plan_cache_for_dml`](/system-variables.md#tidb_enable_non_prepared_plan_cache_for_dml-new-in-v710) | 新規追加 | DML ステートメントのために [非準備プラン キャッシュ](/sql-non-prepared-plan-cache.md) 機能を有効にするかどうかを制御します。 |
| [`tidb_enable_row_level_checksum`](/system-variables.md#tidb_enable_row_level_checksum-new-in-v710) | 新たに追加 | 単一行データのTiCDCデータ整合性検証の有効化を制御します。|
| [`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v710) | 新たに追加 | より細かい最適化の制御を提供し、最適化の動作変更によるパフォーマンスの低下を防ぐのに役立ちます。|
| [`tidb_plan_cache_invalidation_on_fresh_stats`](/system-variables.md#tidb_plan_cache_invalidation_on_fresh_stats-new-in-v710) | 新たに追加 | 関連するテーブルの統計情報が更新されたときに、プランキャッシュを自動的に無効にするかどうかを制御します。|
| [`tidb_plan_cache_max_plan_size`](/system-variables.md#tidb_plan_cache_max_plan_size-new-in-v710) | 新たに追加 | プリペアドプランキャッシュまたはノンプリペアドプランキャッシュにキャッシュできるプランの最大サイズを制御します。|
| [`tidb_prefer_broadcast_join_by_exchange_data_size`](/system-variables.md#tidb_prefer_broadcast_join_by_exchange_data_size-new-in-v710) | 新たに追加 | ネットワーク転送のオーバーヘッドを最小限に抑えるアルゴリズムを使用するかどうかを制御します。この変数が有効になっている場合、TiDBはそれぞれ`Broadcast Hash Join`と`Shuffled Hash Join`を使用してネットワークで交換されるデータのサイズを見積り、その後小さいサイズの方を選択します。この変数が有効になった後、[`tidb_broadcast_join_threshold_count`](/system-variables.md#tidb_broadcast_join_threshold_count-new-in-v50)と[`tidb_broadcast_join_threshold_size`](/system-variables.md#tidb_broadcast_join_threshold_size-new-in-v50)は効果を持ちません。|
| [`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710) | 新たに追加 | キャッシュできるプランの最大数を制御します。プリペアドプランキャッシュとノンプリペアドプランキャッシュは同じキャッシュを共有します。|

### 設定ファイルのパラメータ

| 設定ファイル | 設定パラメータ | 変更タイプ | 説明 |
| -------- | -------- | -------- | -------- |
| TiDB | [`performance.force-init-stats`](/tidb-configuration-file.md#force-init-stats-new-in-v710) | 新たに追加 | TiDB起動中にサービスを提供する前に統計情報の初期化が完了するまで待機するかどうかを制御します。|
| TiDB | [`performance.lite-init-stats`](/tidb-configuration-file.md#lite-init-stats-new-in-v710) | 新たに追加 | TiDB起動中に軽量統計情報初期化を使用するかどうかを制御します。|
| TiDB | [`log.timeout`](/tidb-configuration-file.md#timeout-new-in-v710) | 新たに追加 | TiDBのログ書き込み操作のタイムアウトを設定します。ログが書き込まれないディスク障害の場合、この構成項目はタイムアウトを設定せずにTiDBプロセスをハングからパニックにトリガーします。デフォルト値は`0`で、タイムアウトは設定されていません。|
| TiKV | [`region-compact-min-redundant-rows`](/tikv-configuration-file.md#region-compact-min-redundant-rows-new-in-v710) | 新たに追加 | RocksDBのコンパクションをトリガーするために必要な冗長MVCC行の数を設定します。デフォルト値は`50000`です。|
| TiKV | [`region-compact-redundant-rows-percent`](/tikv-configuration-file.md#region-compact-redundant-rows-percent-new-in-v710) | 新たに追加 | RocksDBのコンパクションをトリガーするために必要な冗長MVCC行のパーセンテージを設定します。デフォルト値は`20`です。|
| TiKV | [`split.byte-threshold`](/tikv-configuration-file.md#byte-threshold-new-in-v50) | 変更 | [`region-split-size`](/tikv-configuration-file.md#region-split-size)が4 GB以上の場合、デフォルト値を`30MiB`から`100MiB`に変更します。|
| TiKV | [`split.qps-threshold`](/tikv-configuration-file.md#qps-threshold) | 変更 | [`region-split-size`](/tikv-configuration-file.md#region-split-size)が4 GB以上の場合、デフォルト値を`3000`から`7000`に変更します。|
| TiKV | [`split.region-cpu-overload-threshold-ratio`](/tikv-configuration-file.md#region-cpu-overload-threshold-ratio-new-in-v620) | 変更 | [`region-split-size`](/tikv-configuration-file.md#region-split-size)が4 GB以上の場合、デフォルト値を`0.25`から`0.75`に変更します。|
| TiKV | [`region-compact-check-step`](/tikv-configuration-file.md#region-compact-check-step) | 変更 | パーティション化されたRaft KVが有効になっている場合 (`storage.engine="partitioned-raft-kv"`)、デフォルト値を`100`から`5`に変更します。|
| PD | [`store-limit-version`](/pd-configuration-file.md#store-limit-version-new-in-v710) | 新たに追加 | ストアリミットのモードを制御します。値のオプションは`"v1"`と`"v2"`です。|
| PD | [`schedule.enable-diagnostic`](/pd-configuration-file.md#enable-diagnostic-new-in-v630) | 変更 | スケジューラの診断機能がデフォルトで有効になるよう、デフォルト値を`false`から`true`に変更します。|
| TiFlash | `http_port` | 削除 | HTTPサービスポート（デフォルトは`8123`）を非推奨とします。|
| TiDB Lightning | [`tikv-importer.pause-pd-scheduler-scope`](/tidb-lightning/tidb-lightning-configuration.md) | 新たに追加 | TiDB LightningがPDスケジューリングを一時停止する範囲を制御します。デフォルト値は`"table"`で、値のオプションは`"global"`と`"table"`です。|
| TiDB Lightning | [`tikv-importer.region-check-backoff-limit`](/tidb-lightning/tidb-lightning-configuration.md) | 新たに追加 | 分割とスキャッタ操作後にRegionがオンラインになるまで待機するリトライ回数を制御します。デフォルト値は`1800`で、リトライ間隔の最大値は2秒です。リトライ間にオンラインになったRegionがある場合、リトライ回数は増加しません。|
| TiDB Lightning | [`tikv-importer.region-split-batch-size`](/tidb-lightning/tidb-lightning-configuration.md) | 新たに追加 | バッチでRegionを分割する際のRegion数を制御します。デフォルト値は`4096`です。|
| TiDB Lightning | [`tikv-importer.region-split-concurrency`](/tidb-lightning/tidb-lightning-configuration.md) | 新たに追加 | Regionを分割する際の並行性を制御します。デフォルト値はCPUコア数です。|
| TiCDC | [`insecure-skip-verify`](/ticdc/ticdc-sink-to-kafka.md) | 新たに追加 | Kafkaへのデータ複製のシナリオにおいてTLSが有効になっている際に、認証アルゴリズムが設定されるかどうかを制御します。|
| TiCDC | [`integrity.corruption-handle-level`](/ticdc/ticdc-changefeed-config.md#cli-and-configuration-parameters-of-ticdc-changefeeds) | 新たに追加 | 単一行データのチェックサム検証が失敗した場合のChangefeedのログレベルを指定します。デフォルト値は`"warn"`で、値のオプションは`"warn"`と`"error"`です。|
| TiCDC | [`integrity.integrity-check-level`](/ticdc/ticdc-changefeed-config.md#cli-and-configuration-parameters-of-ticdc-changefeeds) | 新たに追加 | 単一行データのチェックサム検証を有効にするかどうかを制御します。デフォルト値は`"none"`で、機能を無効にします。|
| TiCDC | [`sink.only-output-updated-columns`](/ticdc/ticdc-changefeed-config.md#cli-and-configuration-parameters-of-ticdc-changefeeds) | 新たに追加 | 更新された列のみを出力するかどうかを制御します。デフォルト値は`false`です。|
| TiCDC | [`sink.enable-partition-separator`](/ticdc/ticdc-changefeed-config.md#cli-and-configuration-parameters-of-ticdc-changefeeds) | 変更 | 追加のテスト後、テーブルのパーティションがデフォルトで別のディレクトリに格納されるよう、デフォルト値を`false`から`true`に変更します。パーティション化されたテーブルのレプリケーション中のデータ損失の潜在的な問題を避けるため、デフォルトで`true`を保持することを推奨します。|

## 改善点

+ TiDB
    - `SHOW INDEX`のCardinality列で対応する列の異なる値の数を表示する[#42227](https://github.com/pingcap/tidb/issues/42227) @[winoros](https://github.com/winoros)
    - TTLスキャンクエリがTiKVブロックキャッシュに影響を与えないよう、`SQL_NO_CACHE`を使用する[#43206](https://github.com/pingcap/tidb/issues/43206) @[lcwangchao](https://github.com/lcwangchao)
    - `MAX_EXECUTION_TIME`に関連するエラーメッセージをMySQLと互換性があるよう改善する[#43031](https://github.com/pingcap/tidb/issues/43031) @[dveeden](https://github.com/dveeden)
    - IndexLookUpでパーティション化されたテーブルでMergeSort演算子を使用する機能をサポートする[#26166](https://github.com/pingcap/tidb/issues/26166) @[Defined2014](https://github.com/Defined2014)
    - `caching_sha2_password`を改良して、MySQLと互換性を持たせる[#43576](https://github.com/pingcap/tidb/issues/43576) @[asjdf](https://github.com/asjdf)

+ TiKV

    - パーティション化されたRaft KVを使用する際のスプリット操作の書き込みQPSへの影響を削減する[#14447](https://github.com/tikv/tikv/issues/14447) @[SpadeA-Tang](https://github.com/SpadeA-Tang)
    - パーティション化されたRaft KVを使用する際のスナップショットが占有するスペースを最適化する[#14581](https://github.com/tikv/tikv/issues/14581) @[bufferflies](https://github.com/bufferflies)
    - TiKVでリクエスト処理の各段階についてより詳細な時間情報を提供する[#12362](https://github.com/tikv/tikv/issues/12362) @[cfzjywxk](https://github.com/cfzjywxk)
    - ログバックアップでPDをメタストアとして使用する[#13867](https://github.com/tikv/tikv/issues/13867) @[YuJuncen](https://github.com/YuJuncen)

+ PD

    - スナップショットの実行詳細に基づいて自動的にストア容量制限のサイズを調整するコントローラを追加する。このコントローラを有効にするには、`store-limit-version`を`v2`に設定する。有効になると、スケーリングインまたはスケーリングアウトの速度を制御するために手動で`store limit`構成を調整する必要がなくなります[#6147](https://github.com/tikv/pd/issues/6147) @[bufferflies](https://github.com/bufferflies)
    - ホットスポットスケジューラによる不安定な負荷を持つ領域の頻繁なスケジューリングを避けるために、履歴的な負荷情報を追加する。ストレージエンジンがraft-kv2の場合[#6297](https://github.com/tikv/pd/issues/6297) @[bufferflies](https://github.com/bufferflies)
    - リーダーのヘルスチェックメカニズムを追加する。ETCDリーダーが存在するPDサーバがリーダーに選出されない場合、PDはETCDリーダーをアクティブに切り替えて、PDリーダーが利用可能であることを保証する[#6403](https://github.com/tikv/pd/issues/6403) @[nolouch](https://github.com/nolouch)

+ TiFlash

    - 分散ストレージおよびコンピュートアーキテクチャにおけるTiFlashのパフォーマンスと安定性を向上する[#6882](https://github.com/pingcap/tiflash/issues/6882) @[JaySon-Huang](https://github.com/JaySon-Huang) @[breezewish](https://github.com/breezewish) @[JinheLin](https://github.com/JinheLin)
    - Semi JoinまたはAnti Semi Joinでのクエリパフォーマンスを最適化する。より小さいテーブルをビルドサイドとして選択する[#7280](https://github.com/pingcap/tiflash/issues/7280) @[yibin87](https://github.com/yibin87)
    - デフォルトの構成でBRおよびTiDB LightningからTiFlashへのデータインポートのパフォーマンスを向上する[#7272](https://github.com/pingcap/tiflash/issues/7272) @[breezewish](https://github.com/breezewish)

+ ツール

    + バックアップおよびリストア (BR)

        - ログバックアップ中にTiKV構成項目`log-backup.max-flush-interval`を変更するサポートを追加する[#14433](https://github.com/tikv/tikv/issues/14433) @[joccau](https://github.com/joccau)

    + TiCDC

        - 複製されたデータをオブジェクトストレージにレプリケートするシナリオにおいて、DDLイベントが発生した際のディレクトリ構造を最適化する[#8890](https://github.com/pingcap/tiflow/issues/8890) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - TiCDC複製タスクが失敗した際に、上流のGC TLSの設定方法を最適化する[#8403](https://github.com/pingcap/tiflow/issues/8403) @[charleszheng44](https://github.com/charleszheng44)
        - Kafka-on-Pulsarへのデータレプリケートをサポートする[#8892](https://github.com/pingcap/tiflow/issues/8892) @[hi-rustin](https://github.com/hi-rustin)
        - Kafkaにデータをレプリケートする際に、更新後に変更された列のみをレプリケートするために、オープンプロトコルプロトコルを使用することをサポートする[#8706](https://github.com/pingcap/tiflow/issues/8706) @[sdojjy](https://github.com/sdojjy)
        - 下流の故障またはその他のシナリオでのTiCDCのエラーハンドリングを最適化する[#8657](https://github.com/pingcap/tiflow/issues/8657) @[hicqu](https://github.com/hicqu)
        - TLSを有効にするシナリオにおいて、認証アルゴリズムを設定するかどうかを制御する構成項目`insecure-skip-verify`を追加する[#8867](https://github.com/pingcap/tiflow/issues/8867) @[hi-rustin](https://github.com/hi-rustin)

    + TiDB Lightning

        - リージョンの不均一な分布に関連するPrecheck項目の重大度レベルを`Critical`から`Warn`に変更して、データのインポート時にユーザーをブロックするのを避ける[#42836](https://github.com/pingcap/tidb/issues/42836) @[okJiang](https://github.com/okJiang)
        - データインポート中に`unknown RPC`エラーが発生した際のリトライメカニズムを追加する[#43291](https://github.com/pingcap/tidb/issues/43291) @[D3Hunter](https://github.com/D3Hunter)
        - リージョンジョブのリトライメカニズムを強化する[#43682](https://github.com/pingcap/tidb/issues/43682) @[lance6716](https://github.com/lance6716)

## バグ修正

+ TiDB

    - パーティションの再構成後に`ANALYZE TABLE`を手動で実行するように促すプロンプトがない問題を修正する[#42183](https://github.com/pingcap/tidb/issues/42183) @[CbcWestwolf](https://github.com/CbcWestwolf)
    - `DROP TABLE`操作が実行されている際の`ADMIN SHOW DDL JOBS`の結果にテーブル名が欠落している問題を修正する[#42268](https://github.com/pingcap/tidb/issues/42268) @[tiancaiamao](https://github.com/tiancaiamao)
    - Grafanaモニタリングパネルで`Ignore Event Per Minute`および`Stats Cache LRU Cost`チャートが正常に表示されない可能性がある問題を修正する[#42562](https://github.com/pingcap/tidb/issues/42562) @[pingandb](https://github.com/pingandb)
    - `INFORMATION_SCHEMA.COLUMNS`テーブルをクエリする際に`ORDINAL_POSITION`列が不正な結果を返す問題を修正する[#43379](https://github.com/pingcap/tidb/issues/43379) @[bb7133](https://github.com/bb7133)
    - パーミッションテーブルの一部の列における大文字小文字の問題を修正する[#41048](https://github.com/pingcap/tidb/issues/41048) @[bb7133](https://github.com/bb7133)
    - キャッシュテーブルに新しい列が追加された後、その値がデフォルト値ではなく`NULL`になる問題を修正する[#42928](https://github.com/pingcap/tidb/issues/42928) @[lqs](https://github.com/lqs)
    - CTE結果がプレディケートを押し下げる際に不正確である問題を修正する[#43645](https://github.com/pingcap/tidb/issues/43645) @[winoros](https://github.com/winoros)
    - 多くのパーティションを持つパーティションテーブルおよびTiFlashレプリカで`TRUNCATE TABLE`を実行する際のDDLリトライが書き込み競合によって引き起こされる問題を修正する[#42940](https://github.com/pingcap/tidb/issues/42940) @[mjonss](https://github.com/mjonss)
    - パーティションテーブルで`SUBPARTITION`を使用する際の警告がない問題を修正する[#41198](https://github.com/pingcap/tidb/issues/41198) [#41200](https://github.com/pingcap/tidb/issues/41200) @[mjonss](https://github.com/mjonss)
    - 生成された列における値のオーバーフロー問題の処理でMySQLとの非互換性の問題を修正する[#40066](https://github.com/pingcap/tidb/issues/40066) @[jiyfhust](https://github.com/jiyfhust)
    - `REORGANIZE PARTITION`を他のDDL操作と同時に実行できない問題を修正する[#42442](https://github.com/pingcap/tidb/issues/42442) @[bb7133](https://github.com/bb7133)
    - DDLのパーティション再構成タスクをキャンセルすると、後続のDDL操作が失敗する可能性のある問題を修正する[#42448](https://github.com/pingcap/tidb/issues/42448) @[lcwangchao](https://github.com/lcwangchao)
    - 特定の条件下での削除操作における不正確なアサーションの問題を修正する[#42426](https://github.com/pingcap/tidb/issues/42426) @[tiancaiamao](https://github.com/tiancaiamao)
- TiDBサーバーがcgroup情報を読み取る際のエラーにより開始できない問題を修正します。エラーメッセージは「cgroup v1からファイルmemory.statを読み取れません: open /sys/memory.stat: no such file or directory」と表示されます。[#42659](https://github.com/pingcap/tidb/issues/42659) @[hawkingrei](https://github.com/hawkingrei)
- グローバルインデックスを持つパーティションテーブルのパーティションキーを更新する際に発生する「Duplicate Key」の問題を修正します。[#42312](https://github.com/pingcap/tidb/issues/42312) @[L-maple](https://github.com/L-maple)
- TTLモニタリングパネルの「Scan Worker Time By Phase」チャートがデータを表示しない問題を修正します。[#42515](https://github.com/pingcap/tidb/issues/42515) @[lcwangchao](https://github.com/lcwangchao)
- グローバルインデックスを持つパーティションテーブルの一部のクエリが正しくない結果を返す問題を修正します。[#41991](https://github.com/pingcap/tidb/issues/41991) [#42065](https://github.com/pingcap/tidb/issues/42065) @[L-maple](https://github.com/L-maple)
- パーティションテーブルを再編成する過程で一部のエラーログが表示される問題を修正します。[#42180](https://github.com/pingcap/tidb/issues/42180) @[mjonss](https://github.com/mjonss)
- 「INFORMATION_SCHEMA.DDL_JOBS」テーブルの「QUERY」列のデータ長が列定義を超える可能性がある問題を修正します。[#42440](https://github.com/pingcap/tidb/issues/42440) @[tiancaiamao](https://github.com/tiancaiamao)
- `INFORMATION_SCHEMA.CLUSTER_HARDWARE`テーブルがコンテナ内で正しくない値を表示する問題を修正します。[#42851](https://github.com/pingcap/tidb/issues/42851) @[hawkingrei](https://github.com/hawkingrei)
- `ORDER BY` + `LIMIT`を使用してパーティションテーブルをクエリした際に不正確な結果が返される問題を修正します。[#43158](https://github.com/pingcap/tidb/issues/43158) @[Defined2014](https://github.com/Defined2014)
- Ingestメソッドを使用して複数のDDLタスクが同時に実行される問題を修正します。[#42903](https://github.com/pingcap/tidb/issues/42903) @[tangenta](https://github.com/tangenta)
- `LIMIT`を使用してパーティションテーブルをクエリした際に誤った値が返される問題を修正します。[#24636](https://github.com/pingcap/tidb/issues/24636)
- IPv6環境でTiDBアドレスが正しく表示されない問題を修正します。[#43260](https://github.com/pingcap/tidb/issues/43260) @[nexustar](https://github.com/nexustar)
- `tidb_enable_tiflash_read_for_write_stmt`および`tidb_enable_exchange_partition`のシステム変数の値がコンテナ内で正しく表示されない問題を修正します。[#43281](https://github.com/pingcap/tidb/issues/43281) @[gengliqi](https://github.com/gengliqi)
- `tidb_scatter_region`が有効な場合、パーティションが切り捨てられた後にRegionが自動的に分割されない問題を修正します。[#43174](https://github.com/pingcap/tidb/issues/43174) [#43028](https://github.com/pingcap/tidb/issues/43028) @[jiyfhust](https://github.com/jiyfhust)
- 生成された列を持つテーブルに対してチェックを追加し、これらの列に対するサポートされていないDDL操作に対してエラーを報告します。[#38988](https://github.com/pingcap/tidb/issues/38988) [#24321](https://github.com/pingcap/tidb/issues/24321) @[tiancaiamao](https://github.com/tiancaiamao)
- 特定の型変換エラーにおいてエラーメッセージが正しくない問題を修正します。[#41730](https://github.com/pingcap/tidb/issues/41730) @[hawkingrei](https://github.com/hawkingrei)
- TiDBノードが正常にシャットダウンされた後に、このノードでトリガーされたDDLタスクがキャンセルされる問題を修正します。 [#43854](https://github.com/pingcap/tidb/issues/43854) @[zimulala](https://github.com/zimulala)
- PDメンバーアドレスが変更された際に、`AUTO_INCREMENT`列のIDの割り当てが長時間ブロックされる問題を修正します。[#42643](https://github.com/pingcap/tidb/issues/42643) @[tiancaiamao](https://github.com/tiancaiamao)
- DDL実行中に`GC lifetime is shorter than transaction duration`エラーが報告される問題を修正します。[#40074](https://github.com/pingcap/tidb/issues/40074) @[tangenta](https://github.com/tangenta)
- メタデータロックが予期せずDDL実行をブロックする問題を修正します。[#43755](https://github.com/pingcap/tidb/issues/43755) @[wjhuang2016](https://github.com/wjhuang2016)
- IPv6環境でクラスターが一部のシステムビューをクエリできない問題を修正します。[#43286](https://github.com/pingcap/tidb/issues/43286) @[Defined2014](https://github.com/Defined2014)
- 動的プルーニングモードでのINNER JOINにおいてパーティションを見つけられない問題を修正します。[#43686](https://github.com/pingcap/tidb/issues/43686) @[mjonss](https://github.com/mjonss)
- テーブルを解析する際にTiDBが構文エラーを報告する問題を修正します。[#43392](https://github.com/pingcap/tidb/issues/43392) @[guo-shaoge](https://github.com/guo-shaoge)
- TiCDCがテーブルの名前変更中に一部の行の変更を失う可能性がある問題を修正します。[#43338](https://github.com/pingcap/tidb/issues/43338) @[tangenta](https://github.com/tangenta)
- クライアントがカーソル読み取りを使用する際にTiDBサーバーがクラッシュする問題を修正します。[#38116](https://github.com/pingcap/tidb/issues/38116) @[YangKeao](https://github.com/YangKeao)
- `ADMIN SHOW DDL JOBS LIMIT`が正しくない結果を返す問題を修正します。[#42298](https://github.com/pingcap/tidb/issues/42298) @[CbcWestwolf](https://github.com/CbcWestwolf)
- `UNION`でユニオンビューと一時テーブルをクエリする際にTiDBパニック問題を修正します。[#42563](https://github.com/pingcap/tidb/issues/42563) @[lcwangchao](https://github.com/lcwangchao)
- トランザクション内で複数のステートメントをコミットした際に、表の名前変更が反映されない問題を修正します。[#39664](https://github.com/pingcap/tidb/issues/39664) @[tiancaiamao](https://github.com/tiancaiamao)
- 時間変換時に準備されたプランキャッシュと非準備プランキャッシュの動作の非互換性の問題を修正します。[#42439](https://github.com/pingcap/tidb/issues/42439) @[qw4990](https://github.com/qw4990)
- Decimal型のプランキャッシュによる誤った結果の問題を修正します。[#43311](https://github.com/pingcap/tidb/issues/43311) @[qw4990](https://github.com/qw4990)
- ヌル許容アンチジョイン（NAAJ）でTiDBパニックが発生する問題を修正します。ヌル許容アンチジョイン（NAAJ）でTiDBパニックが発生する問題を修正します。[#42459](https://github.com/pingcap/tidb/issues/42459) @[AilinKid](https://github.com/AilinKid)
- RC分離レベルの悲観的なトランザクションでDML実行の失敗がデータとインデックスの整合性に影響を与える可能性がある問題を修正します。[#43294](https://github.com/pingcap/tidb/issues/43294) @[ekexium](https://github.com/ekexium)
- いくつかの極端なケースで、悲観的なトランザクションの最初のステートメントがリトライされた際、このトランザクションのロック解除がトランザクションの正確性に影響を与える可能性がある問題を修正します。[#42937](https://github.com/pingcap/tidb/issues/42937) @[MyonKeminta](https://github.com/MyonKeminta)
- いくつかの稀なケースで、GCがロックを解除する際に残存する悲観的なトランザクションのロックがデータの正確性に影響を与える可能性がある問題を修正します。[#43243](https://github.com/pingcap/tidb/issues/43243) @[MyonKeminta](https://github.com/MyonKeminta)
- `LOCK` to `PUT`最適化によって特定のクエリで重複したデータが返される問題を修正します。[#28011](https://github.com/pingcap/tidb/issues/28011) @[zyguan](https://github.com/zyguan)
- ユニークインデックスのロック挙動がデータが変更された場合と変更されていない場合で一貫性がない問題を修正します。[#36438](https://github.com/pingcap/tidb/issues/36438) @[zyguan](https://github.com/zyguan)

+ TiKV
- `tidb_pessimistic_txn_fair_locking`を有効にした場合、極端なケースで失敗したRPCリトライによって引き起こされる期限切れのリクエストが、ロック解決操作中にデータの整合性に影響を与える問題を修正しました[#14551](https://github.com/tikv/tikv/issues/14551) @[MyonKeminta](https://github.com/MyonKeminta)
    - `tidb_pessimistic_txn_fair_locking`を有効にした場合、極端なケースで失敗したRPCリトライによって引き起こされる期限切れのリクエストがトランザクションの整合性に影響を与える問題を修正しました[#14311](https://github.com/tikv/tikv/issues/14311) @[MyonKeminta](https://github.com/MyonKeminta)
    - 暗号化キーIDの競合が古いキーの削除を引き起こす可能性がある問題を修正しました[#14585](https://github.com/tikv/tikv/issues/14585) @[tabokie](https://github.com/tabokie)
    - クラスタが以前のバージョンからv6.5またはそれ以降のバージョンにアップグレードされた際に蓄積されたロックレコードによるパフォーマンス低下の問題を修正しました[#14780](https://github.com/tikv/tikv/issues/14780) @[MyonKeminta](https://github.com/MyonKeminta)
    - PITR（Point-In-Time Recovery）回復プロセス中に`raft entry is too large`エラーが発生する問題を修正しました[#14313](https://github.com/tikv/tikv/issues/14313) @[YuJuncen](https://github.com/YuJuncen)
    - PITR回復プロセス中に`log_batch`が2 GBを超えることによってTiKVがパニックを起こす問題を修正しました[#13848](https://github.com/tikv/tikv/issues/13848) @[YuJuncen](https://github.com/YuJuncen)

+ PD

    - TiKVがパニックした後、PDモニタリングパネルの`low space store`の数が異常な問題を修正しました[#6252](https://github.com/tikv/pd/issues/6252) @[HuSharp](https://github.com/HuSharp)
    - PDリーダー切り替え後にリージョンヘルスモニタリングデータが削除される問題を修正しました[#6366](https://github.com/tikv/pd/issues/6366) @[iosmanthus](https://github.com/iosmanthus)
    - ルールチェッカーが`schedule=deny`ラベルで異常なリージョンを修復できない問題を修正しました[#6426](https://github.com/tikv/pd/issues/6426) @[nolouch](https://github.com/nolouch)
    - TiKVまたはTiFlashの再起動後に一部の既存ラベルが失われる問題を修正しました[#6467](https://github.com/tikv/pd/issues/6467) @[JmPotato](https://github.com/JmPotato)
    - レプリケーションモードで学習者ノードが存在する場合にレプリケーションステータスを切り替えられない問題を修正しました[#14704](https://github.com/tikv/tikv/issues/14704) @[nolouch](https://github.com/nolouch)

+ TiFlash

    - 遅延マテリアライゼーションを有効にした後に`TIMESTAMP`または`TIME`タイプのデータをクエリするとエラーが発生する問題を修正しました[#7455](https://github.com/pingcap/tiflash/issues/7455) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)
    - 大規模な更新トランザクションがTiFlashが繰り返しエラーを報告し再起動する問題を修正しました[#7316](https://github.com/pingcap/tiflash/issues/7316) @[JaySon-Huang](https://github.com/JaySon-Huang)

+ Tools

    + Backup & Restore (BR)

        - TiKVノードがクラッシュした際のバックアップの遅延の問題を修正しました[#42973](https://github.com/pingcap/tidb/issues/42973) @[YuJuncen](https://github.com/YuJuncen)
        - いくつかのケースでバックアップの失敗による正確でないエラーメッセージの問題を修正しました[#43236](https://github.com/pingcap/tidb/issues/43236) @[YuJuncen](https://github.com/YuJuncen)

    + TiCDC

        - TiCDCタイムゾーン設定の問題を修正しました[#8798](https://github.com/pingcap/tiflow/issues/8798) @[hi-rustin](https://github.com/hi-rustin)
        - PDアドレスまたはリーダーが失敗した際にTiCDCが自動的に回復できない問題を修正しました[#8812](https://github.com/pingcap/tiflow/issues/8812) [#8877](https://github.com/pingcap/tiflow/issues/8877) @[asddongmen](https://github.com/asddongmen)
        - 上流のTiKVノードの1つがクラッシュした際にチェックポイントの遅延が増加する問題を修正しました[#8858](https://github.com/pingcap/tiflow/issues/8858) @[hicqu](https://github.com/hicqu)
        - 上流で`EXCHANGE PARTITION`操作が正しくダウンストリームにレプリケートされない問題を修正しました[#8914](https://github.com/pingcap/tiflow/issues/8914) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 特定のシナリオでソーターコンポーネントの過剰なメモリ使用によって発生するOOMの問題を修正しました[#8974](https://github.com/pingcap/tiflow/issues/8974) @[hicqu](https://github.com/hicqu)
        - ダウンストリームのKafkaシンクがローリング再起動された際にTiCDCノードがパニックする問題を修正しました[#9023](https://github.com/pingcap/tiflow/issues/9023) @[asddongmen](https://github.com/asddongmen)

    + TiDB Data Migration (DM)

        - ラテン1データがレプリケーション中に破損する可能性がある問題を修正しました[#7028](https://github.com/pingcap/tiflow/issues/7028) @[lance6716](https://github.com/lance6716)

    + TiDB Dumpling

        - `UNSIGNED INTEGER`タイプのプライマリキーがチャンク分割に使用できない問題を修正しました[#42620](https://github.com/pingcap/tidb/issues/42620) @[lichunzhu](https://github.com/lichunzhu)
        - `--output-file-template`が正しく設定されていない場合にTiDB Dumplingがパニックする問題を修正しました[#42391](https://github.com/pingcap/tidb/issues/42391) @[lichunzhu](https://github.com/lichunzhu)

    + TiDB Binlog

        - 失敗したDDLステートメントに遭遇した際にエラーが発生する可能性がある問題を修正しました[#1228](https://github.com/pingcap/tidb-binlog/issues/1228) @[okJiang](https://github.com/okJiang)

    + TiDB Lightning

        - データインポート中のパフォーマンス低下の問題を修正しました[#42456](https://github.com/pingcap/tidb/issues/42456) @[lance6716](https://github.com/lance6716)
        - 大量のデータをインポートする際に`write to tikv with no leader returned`エラーが発生する問題を修正しました[#43055](https://github.com/pingcap/tidb/issues/43055) @[lance6716](https://github.com/lance6716)
        - データインポート中に過剰な`keys within region is empty, skip doIngest`ログが発生する問題を修正しました[#43197](https://github.com/pingcap/tidb/issues/43197) @[D3Hunter](https://github.com/D3Hunter)
        - 部分書き込み中にパニックが発生する可能性がある問題を修正しました[#43363](https://github.com/pingcap/tidb/issues/43363) @[lance6716](https://github.com/lance6716)
        - 広範なテーブルをインポートする際にOOMが発生する可能性がある問題を修正しました[#43728](https://github.com/pingcap/tidb/issues/43728) @[D3Hunter](https://github.com/D3Hunter)
        - TiDB Lightning Grafanaダッシュボードでデータが欠落する問題を修正しました[#43357](https://github.com/pingcap/tidb/issues/43357) @[lichunzhu](https://github.com/lichunzhu)
        - `keyspace-name`の設定が間違っている場合にデータインポートがスキップされる問題を修正しました[#43684](https://github.com/pingcap/tidb/issues/43684) @[zeminzhou](https://github.com/zeminzhou)
        - あるケースで範囲部分書き込み中にデータインポートがスキップされる問題を修正しました[#43768](https://github.com/pingcap/tidb/issues/43768) @[lance6716](https://github.com/lance6716)

## パフォーマンステスト

TiDB v7.1.0のパフォーマンスについては、TiDB専用クラスタの[TPC-Cパフォーマンステストレポート](https://docs.pingcap.com/tidbcloud/v7.1.0-performance-benchmarking-with-tpcc)および[Sysbenchパフォーマンステストレポート](https://docs.pingcap.com/tidbcloud/v7.1.0-performance-benchmarking-with-sysbench)を参照できます。

## 貢献者

TiDBコミュニティの以下の貢献者に感謝いたします:

- [blacktear23](https://github.com/blacktear23)
- [ethercflow](https://github.com/ethercflow)
- [hihihuhu](https://github.com/hihihuhu)
- [jiyfhust](https://github.com/jiyfhust)
- [L-maple](https://github.com/L-maple)
- [lqs](https://github.com/lqs)
- [pingandb](https://github.com/pingandb)
- [yorkhellen](https://github.com/yorkhellen)
- [yujiarista](https://github.com/yujiarista) (初めての貢献者)