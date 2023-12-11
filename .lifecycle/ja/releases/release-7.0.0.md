---
title: TiDB 7.0.0 リリースノート
summary: TiDB 7.0.0 における新機能、互換性の変更、改善、バグ修正などの情報について学びましょう。
---

# TiDB 7.0.0 リリースノート

リリース日: 2023年3月30日

TiDB バージョン: 7.0.0-[DMR](/releases/versioning.md#development-milestone-releases)

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v7.0/quick-start-with-tidb) | [インストールパッケージ](https://www.pingcap.com/download/?version=v7.0.0#version-list)

v7.0.0-DMR における主な新機能と改善点は以下の通りです：

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
    <td rowspan="2">スケーラビリティおよびパフォーマンス<br/></td>
    <td>セッションレベルの<a href="https://docs.pingcap.com/tidb/v7.0/sql-non-prepared-plan-cache" target="_blank">非プリペアドSQLプランキャッシュ</a> (実験的)</td>
    <td>自動的にセッションレベルでプランキャッシュを再利用し、同じ SQL パターンに対するクエリ時間を短縮するため、事前にプリペアステートメントを手動で設定することなく、コンパイルを削減する機能をサポートします。</td>
  </tr>
  <tr>
    <td>TiFlash は<a href="https://docs.pingcap.com/tidb/v7.0/tiflash-disaggregated-and-s3" target="_blank">分離されたストレージおよび計算アーキテクチャ、および S3共有ストレージ</a>をサポート (実験的)</td>
    <td>TiFlash は、クラウドネイティブアーキテクチャをオプションとして導入します：
      <ul>
        <li>データの弾力的な HTAP リソース利用のために、TiFlash の計算とストレージを分離</li>
        <li>共有ストレージを低コストで提供する S3ベースのストレージエンジンを導入</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td rowspan="2">信頼性および可用性<br/></td>
    <td><a href="https://docs.pingcap.com/tidb/v7.0/tidb-resource-control" target="_blank">リソースコントロールの強化</a> (実験的) </td>
    <td>1 つのクラスタ内の様々なアプリケーションやワークロード向けにリソースグループを割り当て、分離する機能をサポートします。このリリースでは、TiDB は異なるリソースバインディングモード（ユーザー、セッション、ステートメントレベル）およびユーザー定義の優先度をサポートします。また、リソースの合計量の推定を行うためのコマンドも使用できます。</td>
  </tr>
  <tr>
    <td>TiFlash は<a href="https://docs.pingcap.com/tidb/v7.0/tiflash-spill-disk" target="_blank">ディスクへのスピル</a>をサポート</td>
    <td>集約、ソート、ハッシュ結合などのデータ集約型操作における OOM を緩和するため、TiFlash は中間結果をディスクへスピルする機能をサポートします。</td>
  </tr>
  <tr>
    <td rowspan="2">SQL</td>
    <td><a href="https://docs.pingcap.com/tidb/v7.0/time-to-live" target="_blank">行レベル TTL</a> (GA)</td>
    <td>特定の年齢のデータを自動的に失効させることで、データベースのサイズの管理やパフォーマンスの向上をサポートします。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v7.0/partitioned-table#reorganize-partitions" target="_blank"><code>LIST</code>/<code>RANGE</code>パーティションの再編成</a></td>
    <td><code>REORGANIZE PARTITION</code> ステートメントを使用して、隣接するパーティションのマージや 1 つのパーティションを複数に分割することで、パーティショニングされたテーブルの使いやすさを提供します。</td>
  </tr>
  <tr>
    <td rowspan="2">DB操作および監視<br/></td>
    <td>TiDB は<a href="https://docs.pingcap.com/tidb/v7.0/sql-statement-load-data" target="_blank"><code>LOAD DATA</code> ステートメント</a>の機能を強化 (実験的)</td>
    <td>TiDB は、S3/GCS からのデータインポートなどをサポートするように、<code>LOAD DATA</code> SQL ステートメントの機能を強化します。<br/></td>
  </tr>
  <tr>
    <td>TiCDC は<a href="https://docs.pingcap.com/tidb/v7.0/ticdc-sink-to-cloud-storage" target="_blank">オブジェクトストレージシンク</a>をサポート (GA)</td>
    <td>TiCDC は、Amazon S3、GCS、Azure Blob Storage、NFS などのオブジェクトストレージサービスへの行の変更イベントのレプリケーションをサポートします。<br/></td>
  </tr>
</tbody>
</table>

## 機能の詳細

### スケーラビリティ

* TiFlash は分離されたストレージおよび計算アーキテクチャをサポートし、このアーキテクチャでオブジェクトストレージをサポートします (実験的) [#6882](https://github.com/pingcap/tiflash/issues/6882) @[flowbehappy](https://github.com/flowbehappy)

    v7.0.0 以前では、TiFlash は結合されたストレージおよび計算アーキテクチャのみをサポートしています。このアーキテクチャでは、各 TiFlash ノードがストレージと計算の両方のノードとして機能し、その計算およびストレージの機能は独立して拡張できません。また、TiFlash ノードはローカルストレージのみを使用できます。

    v7.0.0 から、TiFlash は分離されたストレージおよび計算アーキテクチャもサポートします。このアーキテクチャでは、TiFlash ノードは 2 種類に分かれており（計算ノードおよび書き込みノード）、S3 API に互換性のあるオブジェクトストレージをサポートしています。両種類のノードは計算またはストレージの能力を個別にスケーリングできます。**分離されたストレージおよび計算アーキテクチャ** と **結合されたストレージおよび計算アーキテクチャ** は、同一のクラスタで使用することや相互に変換することはできません。TiFlash を展開する際に、どのアーキテクチャを使用するかを構成できます。

    詳細については、[ドキュメント](/tiflash/tiflash-disaggregated-and-s3.md)を参照してください。

### パフォーマンス

* Fast Online DDL と PITR の互換性を実現 [#38045](https://github.com/pingcap/tidb/issues/38045) @[Leavrth](https://github.com/Leavrth)

    TiDB v6.5.0 では、[Fast Online DDL](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630) は [PITR](/br/backup-and-restore-overview.md) と完全に互換性がありません。完全なデータバックアップを確保するためには、まず PITR バックグラウンドバックアップタスクを停止し、Fast Online DDL を使用してインデックスを迅速に追加し、その後で PITR バックアップタスクを再開することが推奨されています。

    TiDB v7.0.0 からは、Fast Online DDL と PITR が完全に互換性を持ちます。PITR を介してクラスタデータを復元する際、ログバックアップ中に追加されたインデックス操作は自動的にリプレイされ、互換性が実現されます。

    詳細については、[ドキュメント](/ddl-introduction.md)を参照してください。

* TiFlash は null-aware semi join および null-aware anti semi join 演算子をサポート [#6674](https://github.com/pingcap/tiflash/issues/6674) @[gengliqi](https://github.com/gengliqi)

    相関サブクエリで `IN`、`NOT IN`、`= ANY`、または `!= ALL` 演算子を使用する場合、TiDB はそれらをセミジョインまたはアンチセミジョインに変換することで計算パフォーマンスを最適化します。結合キーの列に `NULL` が含まれる可能性がある場合は、null-aware ジョインアルゴリズムが必要となります。具体的には [Null-aware semi join](/explain-subqueries.md#null-aware-semi-join-in-and--any-subqueries) および [Null-aware anti semi join](/explain-subqueries.md#null-aware-anti-semi-join-not-in-and--all-subqueries) が該当します。

    v7.0.0 以前、TiFlash は null-aware semi join および null-aware anti semi join 演算子をサポートしておらず、これらのサブクエリを TiFlash に直接プッシュダウンできませんでした。v7.0.0 から、TiFlash は null-aware semi join および null-aware anti semi join 演算子をサポートします。SQL ステートメントにこれらの相関サブクエリが含まれ、クエリ内のテーブルに TiFlash のレプリカがある場合、かつ [MPP モード](/tiflash/use-tiflash-mpp-mode.md) が有効にされている場合、オプティマイザーは自動的に TiFlash に null-aware semi join および null-aware anti semi join 演算子をプッシュダウンするかどうかを判断し、全体的なパフォーマンスを向上させます。

    詳細については、[ドキュメント](/tiflash/tiflash-supported-pushdown-calculations.md)を参照してください。

* TiFlash は FastScan を使用できるようになりました (GA) [#5252](https://github.com/pingcap/tiflash/issues/5252) @[hongyunyan](https://github.com/hongyunyan)

    v6.3.0 から、TiFlash は実験的な機能として FastScan を導入しました。v7.0.0 では、この機能が一般に使用可能になりました。システム変数 [`tiflash_fastscan`](/system-variables.md#tiflash_fastscan-new-in-v630) を使用して FastScan を有効にできます。この機能により、強力な整合性を犠牲にすることで、テーブルのスキャンパフォーマンスを大幅に向上させます。対応するテーブルが、`UPDATE`/`DELETE` 操作を行わずに `INSERT` 操作のみを含む場合、FastScan は強力な整合性を保持し、スキャンパフォーマンスを向上させることができます。
```
    [ドキュメント](/tiflash/use-fastscan.md)で詳細を確認してください。

* TiFlashは遅延マテリアライゼーションをサポートしています（実験的）[#5829](https://github.com/pingcap/tiflash/issues/5829) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)

    `SELECT`ステートメントでフィルタ条件（`WHERE`句）を処理する際、TiFlashはデフォルトでクエリによって必要とされる列のすべてのデータを読み取り、その後、クエリ条件に基づいてデータをフィルタおよび集約します。遅延マテリアライゼーションは、TableScan演算子への一部のフィルタ条件のプッシュダウンをサポートする最適化メソッドです。つまり、TiFlashはまずプッシュダウンされたフィルタ条件に関連する列データをスキャンし、条件を満たす行をフィルタし、その後、これらの行の他の列データをスキャンしてさらなる計算を行い、これによりデータ処理のIOスキャンおよび計算が削減されます。

    TiFlashの遅延マテリアライゼーション機能はデフォルトで無効です。`tidb_opt_enable_late_materialization`システム変数を`OFF`に設定することで有効にできます。この機能が有効になると、TiDBオプティマイザーは統計およびフィルタ条件に基づいて、どのフィルタ条件をプッシュダウンするかを決定します。

    [ドキュメント](/tiflash/tiflash-late-materialization.md)で詳細を確認してください。

* 非プリペアドステートメントの実行計画をキャッシュするサポートを追加（実験的）[#36598](https://github.com/pingcap/tidb/issues/36598) @[qw4990](https://github.com/qw4990)

    実行計画キャッシュは、同時OLTPの負荷容量を向上させるために重要です。TiDBはすでに[プリペアド実行計画キャッシュ](/sql-prepared-plan-cache.md)をサポートしています。v7.0.0では、非プリペアドステートメントの実行計画もキャッシュすることができます。これにより実行計画キャッシュの範囲が拡大し、TiDBの同時処理能力が向上します。

    この機能はデフォルトで無効です。`tidb_enable_non_prepared_plan_cache`システム変数を`ON`に設定することで有効にできます。安定性の観点から、TiDB v7.0.0では非プリペアド実行計画のキャッシュ用に新しい領域を割り当てており、システム変数`tidb_non_prepared_plan_cache_size`を使用してキャッシュサイズを設定できます。また、この機能には特定のSQLステートメントに制限があります。詳細は[制限事項](/sql-non-prepared-plan-cache.md#restrictions)を参照してください。

    [ドキュメント](/sql-non-prepared-plan-cache.md)で詳細を確認してください。

* TiDBはサブクエリに対する実行計画キャッシュ制約を削除しました[#40219](https://github.com/pingcap/tidb/issues/40219) @[fzzf678](https://github.com/fzzf678)

    TiDB v7.0.0では、サブクエリの実行計画キャッシュ制約が削除されました。これにより、`SELECT * FROM t WHERE a > (SELECT ...)`のようなサブクエリを含むSQLステートメントの実行計画をキャッシュすることが可能となり、実行計画キャッシュの適用範囲がさらに拡大し、SQLクエリの実行効率が向上します。

    [ドキュメント](/sql-prepared-plan-cache.md)で詳細を確認してください。

* TiKVはログの自動生成をサポートしました [#14371](https://github.com/tikv/tikv/issues/14371) @[LykxSassinator](https://github.com/LykxSassinator)

    v6.3.0では、TiKVは[Raftログの再利用](/tikv-configuration-file.md#enable-log-recycle-new-in-v630)機能を導入し、書き込み負荷による長時間の待ち時間を削減しました。ただし、ログの再利用は一定の閾値に達したときにのみ有効になるため、この機能によるスループットの向上を直接体験することが難しいです。

    v7.0.0では、ユーザーエクスペリエンスを向上させるために、初期化時にログの再利用用に空のログファイルを自動的に生成する`raft-engine.prefill-for-recycle`という新しい構成項目が導入されました。この構成が有効になると、TiKVは初期化中に一括で空のログファイルを自動的に生成し、初期化後すぐにログの再利用が行われることを保証します。

    [ドキュメント](/tikv-configuration-file.md#prefill-for-recycle-new-in-v700)で詳細を確認してください。

* [ウィンドウ関数](/functions-and-operators/expressions-pushed-down.md)からTopNまたはLimit演算子を派生させ、ウィンドウ関数のパフォーマンスを向上させるサポートを追加 [#13936](https://github.com/tikv/tikv/issues/13936) @[windtalker](https://github.com/windtalker)

    この機能はデフォルトで無効です。有効にするには、セッション変数[tidb_opt_derive_topn](/system-variables.md#tidb_opt_derive_topn-new-in-v700)を`ON`に設定します。

    [ドキュメント](/derive-topn-from-window.md)で詳細を確認してください。

* ファストオンラインDDLを使用して一意のインデックスを作成するサポートを追加 [#40730](https://github.com/pingcap/tidb/issues/40730) @[tangenta](https://github.com/tangenta)

    TiDB v6.5.0では、ファストオンラインDDLを使用して通常の二次インデックスの作成をサポートしています。TiDB v7.0.0では、ファストオンラインDDLを使用して一意のインデックスの作成をサポートしています。大規模なテーブルへの一意のインデックスの追加は、v6.1.0と比較して数倍高速になる見込みです。

    [ドキュメント](/ddl-introduction.md)で詳細を確認してください。

###信頼性

* リソース制御機能を強化（実験的）[#38825](https://github.com/pingcap/tidb/issues/38825) @[nolouch](https://github.com/nolouch) @[BornChanger](https://github.com/BornChanger) @[glorv](https://github.com/glorv) @[tiancaiamao](https://github.com/tiancaiamao) @[Connor1996](https://github.com/Connor1996) @[JmPotato](https://github.com/JmPotato) @[hnes](https://github.com/hnes) @[CabinfeverB](https://github.com/CabinfeverB) @[HuSharp](https://github.com/HuSharp)

    TiDBはリソースグループに基づいてリソース制御機能を強化しました。この機能により、TiDBクラスターのリソース利用効率とパフォーマンスが大幅に向上します。リソース制御機能の導入はTiDBの画期的なマイルストーンです。分散データベースクラスターを複数の論理単位に分割し、異なるデータベースユーザーを対応するリソースグループにマッピングし、必要に応じて各リソースグループにクォータを設定できます。クラスターのリソースが制限されている場合、同じリソースグループに属するセッションが使用するすべてのリソースはクォータに制限されます。これにより、リソースグループが過度に消費されても、他のリソースグループのセッションに影響を与えません。

    この機能を使用すると、異なるシステムから複数の中小規模のアプリケーションを単一のTiDBクラスターに組み合わせることができます。特定のアプリケーションのワークロードが大きくなった場合、他のアプリケーションの正常な動作に影響を与えません。システムのワークロードが低い場合でも、必要なシステムリソースが超過していても、ビジーなアプリケーションにシステムリソースが割り当てられるため、リソースの最大利用ができます。また、リソース制御機能の合理的な使用により、クラスターの数を減らし、運用および保守の難しさを軽減し、管理コストを節約できます。

    この機能は、Grafanaの実際のリソース使用状況に対する組み込みのリソース制御ダッシュボードを提供しており、リソースをより合理的に割り当てるのを支援します。また、セッションレベルおよびステートメントレベル（ヒント）に基づいた動的なリソース管理機能をサポートしています。この機能の導入により、TiDBクラスターのリソース使用に対するより正確な制御が可能となり、実際の必要に基づいてクォータを動的に調整できます。

    TiDB v7.0.0では、リソースグループに対して絶対のスケジュール優先度（`PRIORITY`）を設定して、重要なサービスにリソースを確保することができます。また、リソースグループの設定方法も拡張されています。

    リソースグループは以下の方法で使用できます：

    - ユーザーレベル。[`CREATE USER`](/sql-statements/sql-statement-create-user.md)または[`ALTER USER`](/sql-statements/sql-statement-alter-user.md)ステートメントを使用して、ユーザーを特定のリソースグループにバインドします。リソースグループをユーザーにバインドした後、そのユーザーが新しく作成したセッションは自動的に対応するリソースグループにバインドされます。
    - セッションレベル。現在のセッションで使用するリソースグループを[`SET RESOURCE GROUP`](/sql-statements/sql-statement-set-resource-group.md)を使用して設定します。
    - ステートメントレベル。現在のステートメントで使用するリソースグループを[`RESOURCE_GROUP()`](/optimizer-hints.md#resource_groupresource_group_name)を使用して設定します。

  [ドキュメント](/tidb-resource-control.md)で詳細を確認してください。

* ファストオンラインDDLのためのチェックポイントメカニズムをサポートし、信頼性と自動回復能力を向上させました [#42164](https://github.com/pingcap/tidb/issues/42164) @[tangenta](https://github.com/tangenta)

    TiDB v7.0.0では、ファストオンラインDDLに対してチェックポイントメカニズムを導入しました。これにより、DDLの進行状況を定期的に記録および同期することで、TiDB DDLオーナーの障害や切り替えが発生しても、ファストオンラインDDLモードでのDDL操作を継続して実行できるようになります。これにより、DDLの実行が安定し、効率的になります。

    [ドキュメント](/ddl-introduction.md)で詳細を確認してください。
```
* TiFlashはディスクへのスパイリングをサポートしています[#6528](https://github.com/pingcap/tiflash/issues/6528) @[windtalker](https://github.com/windtalker)

   実行パフォーマンスを向上させるために、TiFlashは可能な限りデータをメモリ内で完全に実行します。データの量がメモリの合計サイズを超えると、TiFlashはメモリ不足によるシステムクラッシュを避けるためにクエリを終了します。そのため、TiFlashが処理できるデータの量は利用可能なメモリによって制限されます。

   v7.0.0から、TiFlashはディスクへのスピリングをサポートしています。オペレータのメモリ使用量の閾値（[`tidb_max_bytes_before_tiflash_external_group_by`](/system-variables.md#tidb_max_bytes_before_tiflash_external_group_by-new-in-v700)、[`tidb_max_bytes_before_tiflash_external_sort`](/system-variables.md#tidb_max_bytes_before_tiflash_external_sort-new-in-v700)、および[`tidb_max_bytes_before_tiflash_external_join`](/system-variables.md#tidb_max_bytes_before_tiflash_external_join-new-in-v700)）を調整することで、オペレータが使用できるメモリの最大量を制御できます。オペレータが使用するメモリが閾値を超えると、自動的にデータをディスクに書き込みます。これにより一部のパフォーマンスが犠牲になりますが、より多くのデータを処理できるようになります。

   詳細については、[ドキュメント](/tiflash/tiflash-spill-disk.md)を参照してください。

* 統計データの収集効率を向上させる[#41930](https://github.com/pingcap/tidb/issues/41930) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)

   v7.0.0で、TiDBは統計データの収集ロジックをさらに最適化し、収集時間を約25%削減しました。この最適化により、大規模なデータベースクラスターの操作効率と安定性が向上し、統計データの収集がクラスターパフォーマンスに与える影響が軽減されます。

* MPP最適化のための新しいオプティマイザヒントを追加する[#39710](https://github.com/pingcap/tidb/issues/39710) @[Reminiscent](https://github.com/Reminiscent)

   v7.0.0で、TiDBはMPP実行プランの生成に影響を与える一連のオプティマイザヒントを追加しました。

   - [`SHUFFLE_JOIN()`](/optimizer-hints.md#shuffle_joint1_name--tl_name-): MPPに影響を与えます。指定されたテーブルでShuffle Joinアルゴリズムを使用するようオプティマイザにヒントを与えます。
   - [`BROADCAST_JOIN()`](/optimizer-hints.md#broadcast_joint1_name--tl_name-): MPPに影響を与えます。指定されたテーブルでBroadcast Joinアルゴリズムを使用するようオプティマイザにヒントを与えます。
   - [`MPP_1PHASE_AGG()`](/optimizer-hints.md#mpp_1phase_agg): MPPに影響を与えます。指定されたクエリブロック内のすべての集計関数に対して1段階の集計アルゴリズムを使用するようオプティマイザにヒントを与えます。
   - [`MPP_2PHASE_AGG()`](/optimizer-hints.md#mpp_2phase_agg): MPPに影響を与えます。指定されたクエリブロック内のすべての集計関数に対して2段階の集計アルゴリズムを使用するようオプティマイザにヒントを与えます。

  MPPオプティマイザヒントはHTAPクエリに介入するのに役立ち、HTAPワークロードのパフォーマンスと安定性を向上させます。

  詳細については[ドキュメント](/optimizer-hints.md)を参照してください。

* オプティマイザヒントは、結合方法と結合順序の指定をサポートする[#36600](https://github.com/pingcap/tidb/issues/36600) @[Reminiscent](https://github.com/Reminiscent)

  v7.0.0で、オプティマイザヒント[`LEADING()`](/optimizer-hints.md#leadingt1_name--tl_name-)は結合方法に影響を与え、その動作は互換性があります。複数のテーブル結合の場合、効果的に最適な結合方法と結合順序を指定でき、それにより実行計画に対するオプティマイザヒントのコントロールを向上させることができます。

  新しいヒントの動作にはわずかな変更があります。将来の互換性を確保するために、TiDBはシステム変数[`tidb_opt_advanced_join_hint`](/system-variables.md#tidb_opt_advanced_join_hint-new-in-v700)を導入しています。この変数が`OFF`に設定されていると、オプティマイザヒントの動作は以前のバージョンと互換性があります。以前のバージョンからv7.0.0またはそれ以降のバージョンにクラスターをアップグレードすると、この変数が`OFF`に設定されます。より柔軟なヒントの動作を得るためには、その動作がパフォーマンスの低下を引き起こさないことを確認した後、この変数を`ON`に設定することを強くお勧めします。

  詳細については[ドキュメント](/optimizer-hints.md)を参照してください。

### 利用可能性

* `prefer-leader`オプションをサポートして、読み取り操作の可用性を高め、不安定なネットワーク状況における応答遅延を削減する[#40905](https://github.com/pingcap/tidb/issues/40905) @[LykxSassinator](https://github.com/LykxSassinator)

  システム変数[`tidb_replica_read`](/system-variables.md#tidb_replica_read-new-in-v40)を使用して、TiDBのデータ読み取り動作を制御できます。v7.0.0では、この変数に`prefer-leader`オプションが追加されました。変数が`prefer-leader`に設定されていると、TiDBは読み取り操作を行うためにリーダー副本の選択を優先します。リーダー副本の処理速度が著しく遅くなった場合、たとえばディスクやネットワークパフォーマンスの変動に起因する場合、TiDBは他の利用可能なフォロワー副本を選択して読み取り操作を行い、高い可用性を提供し、応答遅延を削減します。

  詳細については[ドキュメント](/develop/dev-guide-use-follower-read.md)を参照してください。

### SQL

* タイム・ツー・リヴ（TTL）が一般提供可能になりました[#39262](https://github.com/pingcap/tidb/issues/39262) @[lcwangchao](https://github.com/lcwangchao) @[YangKeao](https://github.com/YangKeao)

   TTLは行レベルのライフサイクル管理ポリシーを提供します。TiDBでは、TTL属性が設定されたテーブルは、構成に基づいて期限切れの行データを自動的に確認および削除します。TTLの目的は、クラスターワークロードに対する影響を最小限に抑えながら、ユーザーが不要なデータを定期的に整理できるようにすることです。

   詳細については[ドキュメント](/time-to-live.md)を参照してください。

* `ALTER TABLE…REORGANIZE PARTITION`をサポートする[#15000](https://github.com/pingcap/tidb/issues/15000) @[mjonss](https://github.com/mjonss)

   TiDBは`ALTER TABLE...REORGANIZE PARTITION`構文をサポートしています。この構文を使用すると、データの損失なしでテーブルの一部またはすべてのパーティションを再構成でき、マージ、分割、またはその他の変更を行うことができます。

   詳細については[ドキュメント](/partitioned-table.md#reorganize-partitions)を参照してください。

* Keyパーティショニングをサポートする[#41364](https://github.com/pingcap/tidb/issues/41364) @[TonsnakeLin](https://github.com/TonsnakeLin)

   TiDBは現在、Keyパーティショニングをサポートしています。Keyパーティショニングとハッシュパーティショニングの両方は、データを特定数のパーティションに均等に分散できます。違いは、ハッシュパーティショニングは指定された整数式または整数カラムに基づいてデータを分散するのに対し、Keyパーティショニングはカラムリストに基づいてデータを分散し、Keyパーティショニングのパーティション列は整数型に制限されません。

   詳細については[ドキュメント](/partitioned-table.md#key-partitioning)を参照してください。

### DB操作

* TiCDCはストレージサービスへの変更データのレプリケーションをサポートします（GA）[#6797](https://github.com/pingcap/tiflow/issues/6797) @[zhaoxinyu](https://github.com/zhaoxinyu)

   TiCDCはAmazon S3、GCS、Azure Blob Storage、NFS、およびその他のS3互換のストレージサービスに変更データをレプリケートすることをサポートします。ストレージサービスは手頃な価格で使いやすいです。Kafkaを使用していない場合、ストレージサービスを使用できます。TiCDCは変更ログをファイルに保存し、それをストレージサービスに送信します。ストレージサービスからは、独自のコンシューマプログラムが定期的に新しく生成された変更ログファイルを読むことができます。現在、TiCDCはカナルJSON形式とCSV形式の変更ログをストレージサービスにレプリケートすることができます。

   詳細については[ドキュメント](/ticdc/ticdc-sink-to-cloud-storage.md)を参照してください。

* TiCDC OpenAPI v2 [#8019](https://github.com/pingcap/tiflow/issues/8019) @[sdojjy](https://github.com/sdojjy)

   TiCDCはOpenAPI v2を提供しています。OpenAPI v1と比較して、OpenAPI v2はレプリケーションタスクに対してより包括的なサポートを提供します。TiCDC OpenAPIが提供する機能は[`cdc cli`ツール](/ticdc/ticdc-manage-changefeed.md)のサブセットです。OpenAPI v2を使用してTiCDCクラスターの状態を問い合わせ、操作することができます。例えば、TiCDCノードの状態を取得したり、クラスターのヘルス状態を確認したり、レプリケーションタスクを管理したりすることができます。

   詳細については[ドキュメント](/ticdc/ticdc-open-api-v2.md)を参照してください。

* [DBeaver](https://dbeaver.io/) v23.0.1はデフォルトでTiDBをサポートしています[#17396](https://github.com/dbeaver/dbeaver/issues/17396) @[Icemap](https://github.com/Icemap)

   - 独立したTiDBモジュール、アイコン、ロゴが提供されています。
   - デフォルトの構成は[ TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)をサポートしており、これによりTiDB Serverlessへの接続が容易になります。
    - TiDBのバージョンを認識して、外部キータブを表示または非表示にする機能をサポートしています。
    - `EXPLAIN`の結果でSQL実行計画を視覚化する機能をサポートしています。
    - `PESSIMISTIC`、`OPTIMISTIC`、`AUTO_RANDOM`、`PLACEMENT`、`POLICY`、`REORGANIZE`、`EXCHANGE`、`CACHE`、`NONCLUSTERED`、`CLUSTERED`などのTiDBのキーワードをハイライトする機能をサポートしています。
    - `TIDB_BOUNDED_STALENESS`、`TIDB_DECODE_KEY`、`TIDB_DECODE_PLAN`、`TIDB_IS_DDL_OWNER`、`TIDB_PARSE_TSO`、`TIDB_VERSION`、`TIDB_DECODE_SQL_DIGESTS`、`TIDB_SHARD`などのTiDBの関数をハイライトする機能をサポートしています。

詳細については、[DBeaver documentation](https://github.com/dbeaver/dbeaver/wiki)を参照してください。

### データ移行

* `LOAD DATA` ステートメントの機能を強化し、クラウドストレージからのデータのインポートをサポートしています（実験的）[#40499](https://github.com/pingcap/tidb/issues/40499) @[lance6716](https://github.com/lance6716)

    TiDB v7.0.0より前、`LOAD DATA` ステートメントはクライアントサイドからのデータファイルのみをインポートできました。クラウドストレージからデータをインポートするには、TiDB Lightningに依存する必要がありました。ただし、TiDB Lightningを別途展開すると、追加の展開および管理コストが発生します。v7.0.0では、`LOAD DATA` ステートメントを使用して直接クラウドストレージからデータをインポートできます。この機能の一部の例を以下に示します。

    - Amazon S3およびGoogle Cloud StorageからTiDBへのデータのインポートをサポートしています。ワイルドカードを使用して複数のソースファイルを一度にTiDBにインポートすることをサポートしています。
    - `DEFINED NULL BY`を使用してヌルを定義することをサポートしています。
    - CSVおよびTSV形式のソースファイルをサポートしています。

    詳細については、[documentation](/sql-statements/sql-statement-load-data.md)を参照してください。

* TiDB Lightningは、TiKVにキー・バリューペアを送信する際の圧縮転送をサポートしています（GA）[#41163](https://github.com/pingcap/tidb/issues/41163) @[gozssky](https://github.com/gozssky)

    v6.6.0から、TiDB Lightningは、ローカルでエンコードおよび並び替えられたキー・バリューペアをTiKVに送信する際に圧縮する機能をサポートし、ネットワーク経由で転送されるデータ量を減らし、ネットワーク帯域のオーバーヘッドを低減します。この機能がサポートされる以前のTiDBバージョンでは、TiDB Lightningは比較的高いネットワーク帯域幅を必要とし、データ量が大きい場合には高いトラフィック料金が発生します。

    v7.0.0では、この機能がGAになり、デフォルトでは無効になっています。有効にするには、TiDB Lightningの`compress-kv-pairs`構成項目を`"gzip"`または`"gz"`に設定できます。

    詳細については、[documentation](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)を参照してください。

## 互換性の変更

> **注意:**
>
> このセクションでは、現在のバージョン（v7.0.0）にアップグレードする際に把握する必要のある互換性の変更を提供します。v6.5.0またはそれより前のバージョンから現在のバージョンにアップグレードする場合は、中間バージョンで導入された互換性の変更も確認する必要があります。

### MySQLの互換性

* TiDBは、自動増分列がインデックスである必要がなくなりました[#40580](https://github.com/pingcap/tidb/issues/40580) @[tiancaiamao](https://github.com/tiancaiamao)

    v7.0.0より前は、TiDBの動作はMySQLと一貫しており、自動増分列がインデックスまたはインデックス接頭辞である必要がありました。v7.0.0から、自動増分列がインデックスまたはインデックスの接頭辞である必要がなくなりました。これにより、テーブルの主キーを柔軟に定義し、自動増分列を使用してソートやページネーションをより便利に実装できます。また、自動増分列によって発生する書き込みホットスポット問題を回避し、クラスターインデックスを使用したクエリパフォーマンスを向上させます。新しいリリースでは、次の構文を使用してテーブルを作成できます。

    ```sql
    CREATE TABLE test1 (
        `id` int(11) NOT NULL AUTO_INCREMENT,
        `k` int(11) NOT NULL DEFAULT '0',
        `c` char(120) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
        PRIMARY KEY(`k`, `id`)
    );
    ```

    この機能は、TiCDCデータレプリケーションには影響しません。

    詳細については、[documentation](/mysql-compatibility.md#auto-increment-id)を参照してください。

* TiDBは、以下の例に示すようにKeyパーティションをサポートしています。

    ```sql
    CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE DEFAULT '9999-12-31',
    job_code INT,
    store_id INT) PARTITION BY KEY(store_id) PARTITIONS 4;
    ```

    v7.0.0から、TiDBはKeyパーティションをサポートし、MySQLの`PARTITION BY LINEAR KEY`構文を解析できます。ただし、TiDBは`LINEAR`キーワードを無視し、代わりに非線形ハッシュアルゴリズムを使用します。現在、`KEY`パーティションタイプは、空のパーティション列リストを持つパーティション文をサポートしていません。

    詳細については、[documentation](/partitioned-table.md#key-partitioning)を参照してください。

### 動作の変更

* TiCDCは、Avro形式でレプリケートされたテーブルに含まれる`FLOAT`データのエンコードが誤っている問題を修正しました[#8490](https://github.com/pingcap/tiflow/issues/8490) @[3AceShowHand](https://github.com/3AceShowHand)

    TiCDCクラスタをv7.0.0にアップグレードする際に、Avroを使用してレプリケートされたテーブルに`FLOAT`データ型が含まれる場合は、アップグレード前にConfluent Schema Registryの互換性ポリシーを`None`に手動で調整する必要があります。これにより、チェンジフィードがスキーマを正常に更新できるようになります。そうしないと、アップグレード後にチェンジフィードはスキーマを更新できず、エラー状態に入ります。

* v7.0.0から、[`tidb_dml_batch_size`](/system-variables.md#tidb_dml_batch_size)システム変数は、[`LOAD DATA`ステートメント](/sql-statements/sql-statement-load-data.md)には影響しません。

### システム変数

| 変数名  | 変更タイプ    | 説明 |
|--------|------------------------------|------|
| `tidb_pessimistic_txn_aggressive_locking` | 削除 | この変数は[`tidb_pessimistic_txn_fair_locking`](/system-variables.md#tidb_pessimistic_txn_fair_locking-new-in-v700)に名前が変更されました。 |
| [`tidb_enable_non_prepared_plan_cache`](/system-variables.md#tidb_enable_non_prepared_plan_cache) | 変更 | v7.0.0以降に有効となり、[非準備プランキャッシュ](/sql-non-prepared-plan-cache.md)機能を有効にするかどうかを制御します。 |
| [`tidb_enable_null_aware_anti_join`](/system-variables.md#tidb_enable_null_aware_anti_join-new-in-v630) | 変更 | 追加のテストの結果、デフォルト値を`OFF`から`ON`に変更し、TiDBでは特殊なセット演算子`NOT IN`および`!= ALL`によって生成された副問い合わせにおいてNull-Aware Hash Joinを適用します。 |
| [`tidb_enable_resource_control`](/system-variables.md#tidb_enable_resource_control-new-in-v660) | 変更 | リソースグループによるクラスタのリソースの分離をデフォルトで有効にするため、デフォルト値を`OFF`から`ON`に変更します。リソース制御はv7.0.0でデフォルトで有効になり、必要な場合はこの機能を使用できます。|
| [`tidb_non_prepared_plan_cache_size`](/system-variables.md#tidb_non_prepared_plan_cache_size) | 変更 | v7.0.0以降に有効となり、[非準備プランキャッシュ](/sql-non-prepared-plan-cache.md)によってキャッシュされる実行計画の最大数を制御します。 |
| [`tidb_rc_read_check_ts`](/system-variables.md#tidb_rc_read_check_ts-new-in-v600) | 変更 | v7.0.0から、この変数は準備ステートメントプロトコルでのカーソルフェッチリードにはもはや影響しません。 |
| [`tidb_enable_inl_join_inner_multi_pattern`](/system-variables.md#tidb_enable_inl_join_inner_multi_pattern-new-in-v700) | 新規追加 | この変数は、内部テーブルに`Selection`または`Projection`演算子がある場合にIndex Joinがサポートされるかどうかを制御します。|
| [`tidb_enable_plan_cache_for_subquery`](/system-variables.md#tidb_enable_plan_cache_for_subquery-new-in-v700) | 新規追加 | この変数は、サブクエリを含むクエリをキャッシュするかどうかを制御します。 |
| [`tidb_enable_plan_replayer_continuous_capture`](/system-variables.md#tidb_enable_plan_replayer_continuous_capture-new-in-v700) | 新規追加 | この変数は、[`PLAN REPLAYER CONTINUOUS CAPTURE`](/sql-plan-replayer.md#use-plan-replayer-continuous-capture)機能を有効にするかどうかを制御します。デフォルト値`OFF`は、この機能を無効にします。 |
| [`tidb_load_based_replica_read_threshold`](/system-variables.md#tidb_load_based_replica_read_threshold-new-in-v700) | 新たに追加された | この変数は、負荷ベースレプリカ読み取りをトリガーする閾値を設定します。この変数で制御される機能は、TiDB v7.0.0では完全に機能していません。デフォルト値を変更しないでください。 |
| [`tidb_opt_advanced_join_hint`](/system-variables.md#tidb_opt_advanced_join_hint-new-in-v700) | 新たに追加された | この変数は、結合方法のヒントが結合再配置の最適化に影響を与えるかどうかを制御します。デフォルト値は`ON`であり、新しい互換性のある制御モードが使用されます。値が`OFF`の場合、v7.0.0以前の挙動が使用されます。将来の互換性のために、この変数の値は、クラスターが以前のバージョンからv7.0.0以降にアップグレードされる際に`OFF`に設定されます。 |
| [`tidb_opt_derive_topn`](/system-variables.md#tidb_opt_derive_topn-new-in-v700) | 新たに追加された | この変数は、[Window FunctionsからTopNまたはLimitを導出](/derive-topn-from-window.md)する最適化ルールを有効にするかどうかを制御します。デフォルト値は`OFF`であり、最適化ルールは有効になっていません。 |
| [`tidb_opt_enable_late_materialization`](/system-variables.md#tidb_opt_enable_late_materialization-new-in-v700) | 新たに追加された | この変数は、[TiFlash Late Materialization](/tiflash/tiflash-late-materialization.md)機能を有効にするかどうかを制御します。デフォルト値は`OFF`であり、この機能は有効になっていません。 |
| [`tidb_opt_ordering_index_selectivity_threshold`](/system-variables.md#tidb_opt_ordering_index_selectivity_threshold-new-in-v700) | 新たに追加された | この変数は、SQLステートメントに`ORDER BY`および`LIMIT`句が含まれ、フィルタリング条件がある場合に、オプティマイザーがインデックスを選択する方法を制御します。 |
| [`tidb_pessimistic_txn_fair_locking`](/system-variables.md#tidb_pessimistic_txn_fair_locking-new-in-v700) | 新たに追加された | シングル行競合シナリオのトランザクションのテイルレイテンシを減少させるために、強化された悲観的ロック待機モデルを有効にするかどうかを制御します。デフォルト値は`ON`です。クラスターが以前のバージョンからv7.0.0以降にアップグレードされる際に、この変数の値は`OFF`に設定されます。 |
| [`tidb_ttl_running_tasks`](/system-variables.md#tidb_ttl_running_tasks-new-in-v700) | 新たに追加された | この変数は、クラスタ全体でのTTLタスクの同時実行を制限するために使用されます。デフォルト値の`-1`は、TTLタスクがTiKVノードの数と同じであることを意味します。 |

### 設定ファイルパラメータ

| 設定ファイル | 設定パラメータ | 変更タイプ | 説明 |
| -------- | -------- | -------- | -------- |
| TiKV | `server.snap-max-write-bytes-per-sec` | 削除されました | このパラメータは[`server.snap-io-max-bytes-per-sec`](/tikv-configuration-file.md#snap-io-max-bytes-per-sec)に名前が変更されました。 |
| TiKV | [`raft-engine.enable-log-recycle`](/tikv-configuration-file.md#enable-log-recycle-new-in-v630) | 修正済み | デフォルト値が`false`から`true`に変更されました。 |
| TiKV | [`resolved-ts.advance-ts-interval`](/tikv-configuration-file.md#advance-ts-interval) | 修正済み | デフォルト値が`"1s"`から`"20s"`に変更されました。この変更により、Resolved TSの定期的なアドバンスの間隔が増加し、TiKVノード間のトラフィック消費が減少します。 |
| TiKV | [`resource-control.enabled`](/tikv-configuration-file.md#resource-control) | 修正済み | デフォルト値が`false`から`true`に変更されました。 |
| TiKV | [`raft-engine.prefill-for-recycle`](/tikv-configuration-file.md#prefill-for-recycle-new-in-v700) | 新たに追加された | Raftエンジンのログ再利用に空のログファイルを生成するかどうかを制御します。デフォルト値は`false`です。 |
| PD | [`degraded-mode-wait-duration`](/pd-configuration-file.md#degraded-mode-wait-duration) | 新たに追加された | [Resource Control](/tidb-resource-control.md)に関連する構成項目です。劣化モードをトリガーする待機時間を制御します。デフォルト値は`0s`です。 |
| PD | [`read-base-cost`](/pd-configuration-file.md#read-base-cost) | 新たに追加された | [Resource Control](/tidb-resource-control.md)に関連する構成項目です。読み取りリクエストからRUへの変換の基準要素を制御します。デフォルト値は`0.25`です。 |
| PD | [`read-cost-per-byte`](/pd-configuration-file.md#read-cost-per-byte) | 新たに追加された | [Resource Control](/tidb-resource-control.md)に関連する構成項目です。読み取りフローからRUへの変換の基準要素を制御します。デフォルト値は`1/ (64 * 1024)`です。 |
| PD | [`read-cpu-ms-cost`](/pd-configuration-file.md#read-cpu-ms-cost) | 新たに追加された | [Resource Control](/tidb-resource-control.md)に関連する構成項目です。CPUからRUへの変換の基準要素を制御します。デフォルト値は`1/3`です。 |
| PD | [`write-base-cost`](/pd-configuration-file.md#write-base-cost) | 新たに追加された | [Resource Control](/tidb-resource-control.md)に関連する構成項目です。書き込みリクエストからRUへの変換の基準要素を制御します。デフォルト値は`1`です。 |
| PD | [`write-cost-per-byte`](/pd-configuration-file.md#write-cost-per-byte) | 新たに追加された | [Resource Control](/tidb-resource-control.md)に関連する構成項目です。書き込みフローからRUへの変換の基準要素を制御します。デフォルト値は`1/1024`です。 |
| TiFlash | [`mark_cache_size`](/tiflash/tiflash-configuration.md) | 修正済み | TiFlashのデータブロックのメタデータのデフォルトキャッシュ限界を、不必要なメモリ使用量を減らすために`5368709120`から`1073741824`に変更されました。 |
| TiFlash | [`minmax_index_cache_size`](/tiflash/tiflash-configuration.md) | 修正済み | TiFlashのデータブロックの最大最小インデックスのデフォルトキャッシュ限界を、不必要なメモリ使用量を減らすために`5368709120`から`1073741824`に変更されました。 |
| TiFlash | [`flash.disaggregated_mode`](/tiflash/tiflash-disaggregated-and-s3.md) | 新たに追加された | TiFlashの分離アーキテクチャにおいて、このTiFlashノードがライトノードまたはコンピュートノードであるかを示します。値は`tiflash_write`または`tiflash_compute`になります。 |
| TiFlash | [`storage.s3.endpoint`](/tiflash/tiflash-disaggregated-and-s3.md) | 新たに追加された | S3に接続するためのエンドポイントです。 |
| TiFlash | [`storage.s3.bucket`](/tiflash/tiflash-disaggregated-and-s3.md) | 新たに追加された | TiFlashがすべてのデータを格納するバケットです。 |
| TiFlash | [`storage.s3.root`](/tiflash/tiflash-disaggregated-and-s3.md) | 新たに追加された | S3バケット内のデータストレージのルートディレクトリです。 |
| TiFlash | [`storage.s3.access_key_id`](/tiflash/tiflash-disaggregated-and-s3.md) | 新たに追加された | S3へアクセスするための`ACCESS_KEY_ID`です。 |
| TiFlash | [`storage.s3.secret_access_key`](/tiflash/tiflash-disaggregated-and-s3.md) | 新たに追加された | S3へアクセスするための`SECRET_ACCESS_KEY`です。 |
| TiFlash | [`storage.remote.cache.dir`](/tiflash/tiflash-disaggregated-and-s3.md) | 新たに追加された | TiFlashコンピュートノードのローカルデータキャッシュディレクトリです。 |
| TiFlash | [`storage.remote.cache.capacity`](/tiflash/tiflash-disaggregated-and-s3.md) | 新たに追加された | TiFlashコンピュートノードのローカルデータキャッシュディレクトリのサイズです。 |
| TiDB Lightning | [`add-index-by-sql`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) | 新たに追加された | 物理的なインポートモードでインデックスを追加する際にSQLを使用するかどうかを制御します。デフォルト値は`false`であり、TiDB Lightningは行データとインデックスデータの両方をKVペアにエンコードしてTiKVに一緒にインポートします。インデックスの作成に失敗した場合でも、データの整合性に影響を与えません。 |
| TiCDC | [`enable-table-across-nodes`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 新たに追加された | テーブルを複数のTiCDCノードでレプリケートできる複数の同期範囲に分割するかどうかを決定します。 |
| TiCDC      | [`region-threshold`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 新たに追加された | `enable-table-across-nodes`が有効になっている場合、この機能は`region-threshold`のリージョンが`region-threshold`以上のテーブルにのみ影響します。      |
| DM | [`analyze`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)  | 新たに追加された | CHECKSUM 完了後に各テーブルに対して `ANALYZE TABLE <table>` 操作を実行するかどうかを制御します。 `"required"`/`"optional"`/`"off"` として構成できます。既定値は `"optional"` です。 |
| DM | [`range-concurrency`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)  | 新たに追加された | dm-worker が KV データを TiKV に書き込む並行性を制御します。 |
| DM | [`compress-kv-pairs`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)  | 新たに追加された | dm-worker が KV データを TiKV に送信する際に圧縮を有効にするかどうかを制御します。現在は gzip のみをサポートしています。既定値は圧縮なしを表す空です。 |
| DM | [`pd-addr`](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)  | 新たに追加された | 物理的インポートモードにおける下流 PD サーバのアドレスを制御します。1 つまたは複数の PD サーバを記入できます。この構成項目が空の場合、既定で TiDB クエリから PD アドレス情報を使用します。 |

## 改善

+ TiDB

    - 複数の `DISTINCT` を含む単一の `SELECT` 文の SQL クエリのパフォーマンスを最適化するために `EXPAND` 演算子を導入する [#16581](https://github.com/pingcap/tidb/issues/16581) @[AilinKid](https://github.com/AilinKid)
    - Index Join に対してより多くの SQL フォーマットをサポートする [#40505](https://github.com/pingcap/tidb/issues/40505) @[Yisaer](https://github.com/Yisaer)
    - 特定のケースで TiDB でパーティション化されたテーブルデータをグローバルに整列しないようにする [#26166](https://github.com/pingcap/tidb/issues/26166) @[Defined2014](https://github.com/Defined2014)
    - `fair lock mode` と `lock only if exists` を同時に使用するサポートを追加する [#42068](https://github.com/pingcap/tidb/issues/42068) @[MyonKeminta](https://github.com/MyonKeminta)
    - トランザクション遅延ログとトランザクション内部イベントの出力をサポートする [#41863](https://github.com/pingcap/tidb/issues/41863) @[ekexium](https://github.com/ekexium)
    - `ILIKE` 演算子をサポートする [#40943](https://github.com/pingcap/tidb/issues/40943) @[xzhangxian1008](https://github.com/xzhangxian1008)

+ PD

    - ストア制限によるスケジューリング失敗の監視メトリクスを追加する [#6043](https://github.com/tikv/pd/issues/6043) @[nolouch](https://github.com/nolouch)

+ TiFlash

    - 書き込みパスにおける TiFlash のメモリ使用量を削減する [#7144](https://github.com/pingcap/tiflash/issues/7144) @[hongyunyan](https://github.com/hongyunyan)
    - 多くのテーブルを含むシナリオにおける TiFlash の再起動時間を短縮する [#7146](https://github.com/pingcap/tiflash/issues/7146) @[hongyunyan](https://github.com/hongyunyan)
    - `ILIKE` 演算子のプッシュダウンをサポートする [#6740](https://github.com/pingcap/tiflash/issues/6740) @[xzhangxian1008](https://github.com/xzhangxian1008)

+ Tools

    + TiCDC

        - TiDB クラスタの大規模データ統合シナリオにおける単一の大規模テーブルのデータ変更を複数の TiCDC ノードに分散させるサポートを追加し、Kafka が下流の場合のスケーラビリティの問題を解決する [#8247](https://github.com/pingcap/tiflow/issues/8247) @[overvenus](https://github.com/overvenus)

            TiCDC 構成項目 `enable_table_across_nodes` を `true` に設定することでこの機能を有効にできます。 `region_threshold` を使用して、テーブルの Region 数がこの閾値を超えた場合、TiCDC は対応するテーブルのデータ変更を複数の TiCDC ノードに分散させるようになります。

        - レドーアプライヤーにおけるトランザクションの分割をサポートし、そのスループットを向上させ、災害復旧シナリオにおける RTO を削減する [#8318](https://github.com/pingcap/tiflow/issues/8318) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 単一のテーブルをより均等に複数の TiCDC ノードに分割するためのテーブルスケジューリングを改善する [#8247](https://github.com/pingcap/tiflow/issues/8247) @[overvenus](https://github.com/overvenus)
        - MQ シンクにおける Large Row 監視メトリクスを追加する [#8286](https://github.com/pingcap/tiflow/issues/8286) @[hi-rustin](https://github.com/hi-rustin)
        - 1 つの Region が複数のテーブルのデータを含むシナリオにおける TiKV と TiCDC ノード間のネットワークトラフィックを削減する [#6346](https://github.com/pingcap/tiflow/issues/6346) @[overvenus](https://github.com/overvenus)
        - Checkpoint TS と Resolved TS の P99 メトリクスパネルを Lag 分析パネルに移動する [#8524](https://github.com/pingcap/tiflow/issues/8524) @[hi-rustin](https://github.com/hi-rustin)
        - レドーログにおける DDL イベントの適用をサポートする [#8361](https://github.com/pingcap/tiflow/issues/8361) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 上流の書き込みスループットに基づいてテーブルを TiCDC ノードに分割しスケジューリングするサポートを追加する [#7720](https://github.com/pingcap/tiflow/issues/7720) @[overvenus](https://github.com/overvenus)

    + TiDB Lightning

        - TiDB Lightning 物理インポートモードは、データのインポートとインデックスのインポートを分離して実行することで、インポート速度と安定性を向上させます [#42132](https://github.com/pingcap/tidb/issues/42132) @[gozssky](https://github.com/gozssky)

            `add-index-by-sql` パラメータを追加しました。既定値は `false` で、TiDB Lightning はこの値の場合、行データとインデックスデータを KV ペアにエンコードして一緒に TiKV にインポートします。 `true` に設定すると、TiDB Lightning は行データをインポートした後に `ADD INDEX` SQL 文を使用してインデックスを追加し、インポート速度と安定性を向上させます。

        - `tikv-importer.keyspace-name` パラメータを追加しました。既定値は空文字列で、TiDB Lightning は自動的に対応するテナントのキースペース名を取得してデータをインポートします。値を指定すると、指定されたキースペース名が使用されます。このパラメータは、マルチテナントの TiDB クラスタにデータをインポートする際の TiDB Lightning の設定の柔軟性を提供します [#41915](https://github.com/pingcap/tidb/issues/41915) @[lichunzhu](https://github.com/lichunzhu)

## バグ修正

+ TiDB

    - v6.5.1 から後のバージョンに TiDB をアップグレードする際の更新漏れ問題を修正する [#41502](https://github.com/pingcap/tidb/issues/41502) @[chrysan](https://github.com/chrysan)
    - アップグレード後に一部のシステム変数の既定値が変更されない問題を修正する [#41423](https://github.com/pingcap/tidb/issues/41423) @[crazycs520](https://github.com/crazycs520)
    - インデックス追加に関連する Coprocessor リクエストタイプが不明と表示される問題を修正する [#41400](https://github.com/pingcap/tidb/issues/41400) @[tangenta](https://github.com/tangenta)
    - インデックスを追加する際に "PessimisticLockNotFound" を返す問題を修正する [#41515](https://github.com/pingcap/tidb/issues/41515) @[tangenta](https://github.com/tangenta)
    - ユニークインデックスを追加する際に誤って `found duplicate key` を返す問題を修正する [#41630](https://github.com/pingcap/tidb/issues/41630) @[tangenta](https://github.com/tangenta)
    - インデックスを追加する際のパニック問題を修正する [#41880](https://github.com/pingcap/tidb/issues/41880) @[tangenta](https://github.com/tangenta)
    - 実行中に TiFlash が生成列に関するエラーを報告する問題を修正する [#40663](https://github.com/pingcap/tidb/issues/40663) @[guo-shaoge](https://github.com/guo-shaoge)
    - 時間型が存在する場合に統計情報を正しく取得できない可能性がある問題を修正する [#41938](https://github.com/pingcap/tidb/issues/41938) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
    - 準備されたプランキャッシュが有効な場合にフルインデックススキャンがエラーを引き起こす可能性がある問題を修正する [#42150](https://github.com/pingcap/tidb/issues/42150) @[fzzf678](https://github.com/fzzf678)
    - `IFNULL(NOT NULL COLUMN, ...)`が誤った結果を返す可能性がある問題を修正します [#41734](https://github.com/pingcap/tidb/issues/41734) @[LittleFall](https://github.com/LittleFall)
    - パーティション化されたテーブルのすべてのデータが単一のリージョンにある場合に、TiDBが誤った結果を生成する可能性がある問題を修正します [#41801](https://github.com/pingcap/tidb/issues/41801) @[Defined2014](https://github.com/Defined2014)
    - 1つのSQLステートメント内で異なるパーティション化されたテーブルが現れた場合に、TiDBが誤った結果を生成する可能性がある問題を修正します [#42135](https://github.com/pingcap/tidb/issues/42135) @[mjonss](https://github.com/mjonss)
    - パーティション化されたテーブルに新しいインデックスを追加した後に、統計情報の自動収集が正しくトリガーされない可能性がある問題を修正します
[#41638](https://github.com/pingcap/tidb/issues/41638) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
    - 統計情報を連続して2回収集した後に、TiDBが誤った列の統計情報を読み取る可能性がある問題を修正します [#42073](https://github.com/pingcap/tidb/issues/42073) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
    - プランキャッシュの有効化時に、IndexMergeが誤った結果を生成する可能性がある問題を修正します [#41828](https://github.com/pingcap/tidb/issues/41828) @[qw4990](https://github.com/qw4990)
    - IndexMergeがゴルーチンのリークを引き起こす可能性がある問題を修正します [#41605](https://github.com/pingcap/tidb/issues/41605) @[guo-shaoge](https://github.com/guo-shaoge)
    - BIGINTでない符号なし整数が文字列/10進数と比較された際に誤った結果を生成する可能性がある問題を修正します [#41736](https://github.com/pingcap/tidb/issues/41736) @[LittleFall](https://github.com/LittleFall)
    - メモリの制限超過により前の`ANALYZE`ステートメントをキャンセルすると、同じセッションで現在の`ANALYZE`ステートメントもキャンセルされる可能性がある問題を修正します
[#41825](https://github.com/pingcap/tidb/issues/41825) @[XuHuaiyu](https://github.com/XuHuaiyu)
    - バッチコプロセッサの情報収集中にデータ競合が発生する可能性がある問題を修正します [#41412](https://github.com/pingcap/tidb/issues/41412) @[you06](https://github.com/you06)
    - パーティション化されたテーブルのMVCC情報を印刷する際にアサーションエラーが発生し、印刷が阻止される可能性がある問題を修正します
[#40629](https://github.com/pingcap/tidb/issues/40629) @[ekexium](https://github.com/ekexium)
    - フェアなロックモードが存在しないキーにロックを加える可能性がある問題を修正します [#41527](https://github.com/pingcap/tidb/issues/41527) @[ekexium](https://github.com/ekexium)
    - `INSERT IGNORE`および`REPLACE`ステートメントが値を変更しないキーにロックを加えない可能性がある問題を修正します [#42121](https://github.com/pingcap/tidb/issues/42121) @[zyguan](https://github.com/zyguan)

+ PD

    - リージョンスキャッタ操作がリーダーの均等な分布を引き起こす可能性がある問題を修正します [#6017](https://github.com/tikv/pd/issues/6017) @[HunDunDM](https://github.com/HunDunDM)
    - 起動中にPDメンバーを取得する際にデータ競合が発生する可能性がある問題を修正します [#6069](https://github.com/tikv/pd/issues/6069) @[rleungx](https://github.com/rleungx)
    - ホットスポット統計を収集する際にデータ競合が発生する可能性がある問題を修正します [#6069](https://github.com/tikv/pd/issues/6069) @[lhy1024](https://github.com/lhy1024)
    - プレイスメントルールを切り替えると、リーダーの均等な分布が引き起こされる可能性がある問題を修正します [#6195](https://github.com/tikv/pd/issues/6195) @[bufferflies](https://github.com/bufferflies)

+ TiFlash

    - Decimalの除算が特定のケースで最後の桁を正しく切り上げない問題を修正します [#7022](https://github.com/pingcap/tiflash/issues/7022) @[LittleFall](https://github.com/LittleFall)
    - Decimalのキャストが特定のケースで誤って切り上げる問題を修正します [#6994](https://github.com/pingcap/tiflash/issues/6994) @[windtalker](https://github.com/windtalker)
    - 新しい照合を有効にした後にTopN/Sort演算子が誤った結果を生成する問題を修正します [#6807](https://github.com/pingcap/tiflash/issues/6807) @[xzhangxian1008](https://github.com/xzhangxian1008)
    - 単一のTiFlashノードで12百万行を超える結果セットを集約するとTiFlashがエラーを報告する問題を修正します [#6993](https://github.com/pingcap/tiflash/issues/6993) @[windtalker](https://github.com/windtalker)

+ ツール

    + バックアップ＆リストア（BR）

        - PITRリカバリプロセス中のリージョンリトライに不足した待機時間の問題を修正します [#42001](https://github.com/pingcap/tidb/issues/42001) @[joccau](https://github.com/joccau)
        - PITRリカバリプロセス中に`memory is limited`エラーが発生し、リカバリが失敗する問題を修正します [#41983](https://github.com/pingcap/tidb/issues/41983) @[joccau](https://github.com/joccau)
        - PDノードがダウンした場合、PITRログバックアップの進行が進まない問題を軽減します [#14184](https://github.com/tikv/tikv/issues/14184) @[YuJuncen](https://github.com/YuJuncen)
        - リーダーマイグレーションが発生すると、PITRログバックアップの待ち時間が増加する問題を緩和します [#13638](https://github.com/tikv/tikv/issues/13638) @[YuJuncen](https://github.com/YuJuncen)

    + TiCDC

        - changefeedを再起動するとデータが失われたり、チェックポイントが進まなくなる可能性がある問題を修正します [#8242](https://github.com/pingcap/tiflow/issues/8242) @[overvenus](https://github.com/overvenus)
        - DDL sinkでデータ競合が発生する可能性がある問題を修正します [#8238](https://github.com/pingcap/tiflow/issues/8238) @[3AceShowHand](https://github.com/3AceShowHand)
        - `stopped`状態のchangefeedが自動的に再起動する可能性がある問題を修正します [#8330](https://github.com/pingcap/tiflow/issues/8330) @[sdojjy](https://github.com/sdojjy)
        - 下流のKafkaサーバーがすべて利用できない場合にTiCDCサーバーがパニックする問題を修正します [#8523](https://github.com/pingcap/tiflow/issues/8523) @[3AceShowHand](https://github.com/3AceShowHand)
        - 下流がMySQLであり、実行されたステートメントがTiDBと互換性がない場合にデータが失われる可能性がある問題を修正します [#8453](https://github.com/pingcap/tiflow/issues/8453) @[asddongmen](https://github.com/asddongmen)
        - ローリングアップグレードがTiCDC OOMを引き起こす可能性がある問題を修正します [#8329](https://github.com/pingcap/tiflow/issues/8329) @[overvenus](https://github.com/overvenus)
        - Kubernetes上でTiCDCクラスターの優雅なアップグレードが失敗する可能性がある問題を修正します [#8484](https://github.com/pingcap/tiflow/issues/8484) @[overvenus](https://github.com/overvenus)

    + TiDBデータ移行（DM）

        - DMワーカーノードでGoogle Cloud Storageを使用した際に、リクエスト頻度制限に達してデータを書き込めなくなる問題を修正します [#8482](https://github.com/pingcap/tiflow/issues/8482) @[maxshuang](https://github.com/maxshuang)
        - 複数のDMタスクが同時に下流データを複製し、すべて下流メタデータテーブルを使用してブレークポイント情報を記録すると、すべてのタスクのブレークポイント情報が同じメタデータテーブルに書き込まれ、同じタスクIDが使用される問題を修正します
[#8500](https://github.com/pingcap/tiflow/issues/8500) @[maxshuang](https://github.com/maxshuang)
    
    + TiDB Lightning

        - 物理インポートモードを使用してデータをインポートする際に、ターゲットテーブルの複合主キーに`auto_random`列があるが、ソースデータで列の値が指定されていない場合にTiDB Lightningが自動的に`auto_random`列のデータを生成しない問題を修正します
[#41454](https://github.com/pingcap/tidb/issues/41454) @[D3Hunter](https://github.com/D3Hunter)
- ロジカルインポートモードを使用してデータをインポートする際に、ターゲットクラスターに `CONFIG` 権限が不足しているため、インポートが失敗する問題を修正しました [#41915](https://github.com/pingcap/tidb/issues/41915) @[lichunzhu](https://github.com/lichunzhu)

## 貢献者

TiDBコミュニティの以下の貢献者に感謝いたします:

- [AntiTopQuark](https://github.com/AntiTopQuark)
- [blacktear23](https://github.com/blacktear23)
- [BornChanger](https://github.com/BornChanger)
- [Dousir9](https://github.com/Dousir9)
- [erwadba](https://github.com/erwadba)
- [HappyUncle](https://github.com/HappyUncle)
- [jiyfhust](https://github.com/jiyfhust)
- [L-maple](https://github.com/L-maple)
- [liumengya94](https://github.com/liumengya94)
- [woofyzhao](https://github.com/woofyzhao)
- [xiaguan](https://github.com/xiaguan)