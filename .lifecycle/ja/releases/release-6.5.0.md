---
title: TiDB 6.5.0のリリースノート
summary: TiDB 6.5.0の新機能、互換性の変更、改善、バグ修正について学びます。

# TiDB 6.5.0のリリースノート

リリース日: 2022年12月29日

TiDBバージョン: 6.5.0

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.5/quick-start-with-tidb) | [プロダクション展開](https://docs.pingcap.com/tidb/v6.5/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.5.0#version-list)

TiDB 6.5.0は、Long-Term Support Release（LTS）です。

TiDB [6.4.0-DMR](/releases/release-6.4.0.md)と比較して、TiDB 6.5.0では以下の主な機能と改善が導入されています。

> **ヒント:**
>
> 6.5.0では、[6.2.0-DMR](/releases/release-6.2.0.md)、[6.3.0-DMR](/releases/release-6.3.0.md)、および[6.4.0-DMR](/releases/release-6.4.0.md)でリリースされた新機能、改善、バグ修正も含まれています。

- 6.1.0 LTSと6.5.0 LTSバージョン間の変更の完全なリストについては、リリースノートに加えて、[6.2.0-DMRのリリースノート](/releases/release-6.2.0.md)、[6.3.0-DMRのリリースノート](/releases/release-6.3.0.md)、[6.4.0-DMRのリリースノート](/releases/release-6.4.0.md)も参照してください。
- 6.1.0 LTSと6.5.0 LTSバージョン間の主な機能の迅速な比較については、[TiDB機能](/basic-features.md)の`v6.1`と`v6.5`コラムを確認できます。

- [インデックスアクセラレーション](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)機能が一般利用可能（GA）となり、v6.1.0と比較してインデックスの追加のパフォーマンスが約10倍向上します。
- TiDBのグローバルメモリ制御がGAとなり、[`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640)を介してメモリ消費の閾値を制御できます。
- 高性能かつグローバルに単調増加する[`AUTO_INCREMENT`](/auto-increment.md#mysql-compatibility-mode)カラム属性がGAとなり、MySQLと互換性があります。
- [`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md)はTiCDCとPITRと互換性があり、GAとなりました。
- より精度の高い[コストモデルバージョン2](/cost-model.md#cost-model-version-2)がTiDBオプティマイザを強化し、`AND`によって接続された式を[Index Merge](/explain-index-merge.md)のサポートで一般利用可能としました。
- `JSON_EXTRACT()`関数をTiFlashにプッシュダウンするサポートを追加しました。
- パスワードコンプライアンス監査要件を満たす[パスワード管理](/password-management.md)ポリシーのサポートを追加しました。
- TiDB LightningとDumplingは圧縮されたSQLおよびCSVファイルの[インポート](/tidb-lightning/tidb-lightning-data-source.md)と[エクスポート](/dumpling-overview.md#improve-export-efficiency-through-concurrency)をサポートします。
- TiDB Data Migration（DM）[継続的なデータ検証](/dm/dm-continuous-data-validation.md)がGAとなりました。
- TiDB Backup & Restoreはスナップショットチェックポイントバックアップをサポートし、[PITR](/br/br-pitr-guide.md#run-pitr)の回復パフォーマンスを50%向上させ、一般的なシナリオでのRPOを5分まで短縮しました。
- [Kafkaへのデータ複製](/replicate-data-to-kafka.md)のTiCDCスループットを4000行/秒から35000行/秒に向上させ、レプリケーションの遅延を2秒に短縮しました。
- 行レベルの[Time to live（TTL）](/time-to-live.md)を提供してデータライフサイクルを管理する（実験的）。
- TiCDCは、Amazon S3、Azure Blob Storage、NFSなどのオブジェクトストレージに変更されたログを[レプリケート](/ticdc/ticdc-sink-to-cloud-storage.md)するサポートを追加しました（実験的）。

## 新機能

### SQL

* TiDBのインデックスの追加パフォーマンスが約10倍向上しました（GA）[ #35983 ](https://github.com/pingcap/tidb/issues/35983) @[benjamin2037](https://github.com/benjamin2037) @[tangenta](https://github.com/tangenta)

    TiDB v6.3.0では、[インデックスアクセラレーション](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)が実験的な機能として導入され、インデックスを追加する際のバックフィリングの速度を向上させました。v6.5.0では、この機能がGAとなり、デフォルトで有効になり、大きなテーブルでのパフォーマンスはv6.1.0と比較して約10倍速くなることが期待されます。このアクセラレーション機能は、単一のSQLステートメントが直列にインデックスを追加するシナリオに適しています。複数のSQLステートメントが並列でインデックスを追加する場合、そのうちの1つしかアクセラレーションされません。

* DDL変更中にDMLの成功率を向上させるための軽量なメタデータロックを導入しました（GA）[ #37275 ](https://github.com/pingcap/tidb/issues/37275) @[wjhuang2016](https://github.com/wjhuang2016)

    TiDB v6.3.0では、[メタデータロック](/metadata-lock.md)が実験的な機能として導入されました。DMLステートメントによって引き起こされる`情報スキーマが変更された`エラーを回避するために、TiDBはテーブルメタデータの変更中にDMLとDDLの優先順位を調整し、進行中のDDLが古いメタデータを持つDMLがコミットするのを待機させます。v6.5.0では、この機能がGAとなり、デフォルトで有効になります。さまざまなDDL変更シナリオに適しています。既存のクラスタをv6.5.0以降にアップグレードすると、TiDBは自動的にメタデータロックを有効にします。この機能を無効にするには、システム変数[`tidb_enable_metadata_lock`](/system-variables.md#tidb_enable_metadata_lock-new-in-v630)を`OFF`に設定できます。

    詳細については、[ドキュメント](/metadata-lock.md)を参照してください。

* `FLASHBACK CLUSTER TO TIMESTAMP`を使用して特定の時点にクラスタを復元するサポートを追加しました（GA）[ #37197 ](https://github.com/pingcap/tidb/issues/37197) [ #13303 ](https://github.com/tikv/tikv/issues/13303) @[Defined2014](https://github.com/Defined2014) @[bb7133](https://github.com/bb7133) @[JmPotato](https://github.com/JmPotato) @[Connor1996](https://github.com/Connor1996) @[HuSharp](https://github.com/HuSharp) @[CalvinNeo](https://github.com/CalvinNeo)

    v6.4.0以降、TiDBは[`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md)ステートメントを実験的な機能として導入しました。このステートメントを使用して、クラスタをガベージコレクション（GC）の寿命内の特定の時点に復元できます。v6.5.0では、この機能はTiCDCとPITRと互換性があり、GAとなりました。この機能により、DMLの誤操作を簡単に元に戻し、数分で元のクラスタを復元し、データの変更時点を確定するための異なる時点にデータをロールバックできます。

    詳細については、[ドキュメント](/sql-statements/sql-statement-flashback-to-timestamp.md)を参照してください。

* `INSERT`、`REPLACE`、`UPDATE`、`DELETE`を含む非トランザクショナルDMLステートメントを完全にサポートしました[ #33485 ](https://github.com/pingcap/tidb/issues/33485) @[ekexium](https://github.com/ekexium)

    大規模データ処理のシナリオでは、大きなトランザクションを持つ単一のSQLステートメントにより、クラスタの安定性とパフォーマンスに悪影響を及ぼすことがあります。非トランザクショナルDMLステートメントは、複数のSQLステートメントに分割されたDMLステートメントで内部的に実行されます。これらの分割ステートメントはトランザクションの原子性と分離性を犠牲にしますが、クラスタの安定性を大幅に向上させます。TiDBは、v6.1.0以降、非トランザクショナル`DELETE`ステートメントをサポートしており、v6.5.0以降、非トランザクショナル`INSERT`、`REPLACE`、`UPDATE`ステートメントをサポートしています。

    詳細については、[非トランザクショナルDMLステートメント](/non-transactional-dml.md)と[`BATCH`構文](/sql-statements/sql-statement-batch.md)を参照してください。

* Time to live（TTL）をサポートしました（実験的）[ #39262 ](https://github.com/pingcap/tidb/issues/39262) @[lcwangchao](https://github.com/lcwangchao)
    TTLは行レベルのデータライフタイム管理を提供します。TiDBでは、TTL属性があるテーブルは自動でデータの寿命をチェックし、行レベルで期限切れのデータを削除します。TTLは、オンラインの読み書きワークロードに影響を与えることなく、定期的かつ迅速に不要なデータを整理するのに役立つよう設計されています。

詳細は[ドキュメント](/time-to-live.md)を参照してください。

* TiFlashクエリの結果を`INSERT INTO SELECT`ステートメントで保存する機能をサポート（実験的）[#37515](https://github.com/pingcap/tidb/issues/37515) @[gengliqi](https://github.com/gengliqi)

v6.5.0から、TiDBは`INSERT INTO SELECT`ステートメントの`SELECT`句（解析クエリ）をTiFlashにプッシュダウンする機能をサポートしています。これにより、TiFlashクエリの結果を`INSERT INTO`で指定されたTiDBテーブルに簡単に保存し、さらなる解析のための結果のキャッシング（つまり、結果のマテリアリゼーション）が可能となります。例：

```sql
INSERT INTO t2 SELECT Mod(x,y) FROM t1;
```

実験段階では、この機能はデフォルトで無効になっています。有効にするには、[`tidb_enable_tiflash_read_for_write_stmt`](/system-variables.md#tidb_enable_tiflash_read_for_write_stmt-new-in-v630)システム変数を`ON`に設定できます。この機能に対して`INSERT INTO`で指定された結果テーブルに特別な制限はありませんので、その結果テーブルにTiFlashレプリカを追加するかどうかは自由です。この機能の典型的な使用シナリオには以下があります：

- TiFlashを使用した複雑な解析クエリの実行
- TiFlashクエリの結果を再利用したり、非常に同時に発生するオンラインリクエストを処理したりする必要がある場合
- 入力データのサイズよりも比較的小さい結果セットが必要な場合（できる限り100 MiBより小さい）

詳細は[ドキュメント](/tiflash/tiflash-results-materialization.md)を参照してください。

* 履歴実行計画のバインドをサポート（実験的）[#39199](https://github.com/pingcap/tidb/issues/39199) @[fzzf678](https://github.com/fzzf678)

SQLステートメントについて、実行中に様々な要因により、オプティマイザーは前回の最適な実行計画ではなく新しい実行計画を選択することがあり、SQLパフォーマンスに影響を与えることがあります。このような場合、もしも最適な実行計画がまだクリアされていなければ、それはSQL実行履歴にまだ存在します。

v6.5.0では、TiDBは[`CREATE [GLOBAL | SESSION] BINDING`](/sql-statements/sql-statement-create-binding.md)ステートメントでバインドオブジェクトを拡張し、履歴実行計画のバインドをサポートしています。SQLステートメントの実行計画が変更になった場合、`CREATE [GLOBAL | SESSION] BINDING`ステートメントで`plan_digest`を指定することで、元の実行計画をクイックにリカバリーすることができます。ただし、元の実行計画は、SQL実行履歴メモリテーブル（例：`statements_summary`）にまだ存在している場合に限ります。この機能により、実行計画の変更に関する問題の処理を簡略化し、保守作業の効率を向上させることができます。

詳細は[ドキュメント](/sql-plan-management.md#create-a-binding-according-to-a-historical-execution-plan)を参照してください。

### セキュリティ

* パスワード複雑性ポリシーをサポート [#38928](https://github.com/pingcap/tidb/issues/38928) @[CbcWestwolf](https://github.com/CbcWestwolf)

このポリシーが有効になると、パスワードを設定する際にTiDBはパスワードの長さ、大文字と小文字の使用、数字、特殊文字の十分な使用、パスワードが辞書と一致し、パスワードがユーザー名と一致するかどうかをチェックします。これにより、セキュアなパスワードを設定できます。

TiDBはパスワード強度を検証するためのSQL機能[`VALIDATE_PASSWORD_STRENGTH()`](https://dev.mysql.com/doc/refman/5.7/en/encryption-functions.html#function_validate-password-strength)を提供します。

詳細は[ドキュメント](/password-management.md#password-complexity-policy)を参照してください。

* パスワード有効期限ポリシーをサポート [#38936](https://github.com/pingcap/tidb/issues/38936) @[CbcWestwolf](https://github.com/CbcWestwolf)

TiDBは手動有効期限、グローバルレベルの自動有効期限、およびアカウントレベルの自動有効期限を含むパスワード有効期限ポリシーの構成をサポートしています。このポリシーが有効になると、定期的にパスワードを変更する必要があります。これにより、パスワードの長期使用によるパスワード漏洩のリスクが低減され、パスワードのセキュリティが向上します。

詳細は[ドキュメント](/password-management.md#password-expiration-policy)を参照してください。

* パスワード再利用ポリシーをサポート [#38937](https://github.com/pingcap/tidb/issues/38937) @[keeplearning20221](https://github.com/keeplearning20221)

TiDBはグローバルレベルのパスワード再利用ポリシーとアカウントレベルのパスワード再利用ポリシーを構成することができます。このポリシーが有効になると、指定された期間内に使用したパスワードや直近で使用したいくつかのパスワードを再利用できなくなります。これにより、同じパスワードを繰り返し使用することによるパスワード漏洩のリスクが低減され、パスワードのセキュリティが向上します。

詳細は[ドキュメント](/password-management.md#password-reuse-policy)を参照してください。

* ログイン失敗の追跡と一時的なアカウントロックポリシーをサポート [#38938](https://github.com/pingcap/tidb/issues/38938) @[lastincisor](https://github.com/lastincisor)

このポリシーが有効になると、TiDBに対して連続して間違ったパスワードでログインした場合、アカウントが一時的にロックされます。ロック時間が終了すると、アカウントは自動的にロック解除されます。

詳細は[ドキュメント](/password-management.md#failed-login-tracking-and-temporary-account-locking-policy)を参照してください。

### 観測性

* TiDBダッシュボードを独立したPodとしてKubernetes上に展開できるようになりました [#1447](https://github.com/pingcap/tidb-dashboard/issues/1447) @[SabaPing](https://github.com/SabaPing)

TiDB v6.5.0（および以降）およびTiDB Operator v1.4.0（および以降）は、TiDBダッシュボードをKubernetes上で独立したPodとして展開することをサポートしています。TiDB Operatorを使用すると、このPodのIPアドレスにアクセスしてTiDBダッシュボードを起動することができます。

TiDBダッシュボードを独立して展開すると以下の利点があります：

- TiDBダッシュボードの計算作業がPDノードに圧力を与えません。これにより、より安定したクラスタの操作が保証されます。
- PDノードが利用できない場合でも、ユーザーは診断のためにTiDBダッシュボードにアクセスできます。
- インターネット経由でTiDBダッシュボードにアクセスすると、PDの特権インタフェースは関与しないため、クラスタのセキュリティリスクが低減されます。

詳細は[ドキュメント](https://docs.pingcap.com/tidb-in-kubernetes/dev/get-started#deploy-tidb-dashboard-independently)を参照してください。

* パフォーマンス概要ダッシュボードにTiFlashとCDC（Change Data Capture）パネルを追加 [#39230](https://github.com/pingcap/tidb/issues/39230) @[dbsid](https://github.com/dbsid)

v6.1.0以降、TiDBはGrafanaでパフォーマンス概要ダッシュボードを導入し、TiDB、TiKV、PDの全体的なパフォーマンス診断のためのシステムレベルのエントリを提供しています。v6.5.0では、パフォーマンス概要ダッシュボードにTiFlashとCDCパネルが追加されました。これにより、v6.5.0以降、パフォーマンス概要ダッシュボードを使用して、TiDBクラスタの全コンポーネントのパフォーマンスを分析することができます。

TiFlashとCDCパネルは、TiFlashとTiCDCモニタリング情報を再編成し、TiFlashとTiCDCのパフォーマンスの問題を分析およびトラブルシューティングする効率を大幅に向上させることができます。

- [TiFlashパネル](/grafana-performance-overview-dashboard.md#tiflash)では、TiFlashクラスタのリクエストタイプ、レイテンシ解析、リソース使用概要を簡単に表示できます。
- [CDCパネル](/grafana-performance-overview-dashboard.md#cdc)では、TiCDCクラスタのヘルス状況、レプリケーションレイテンシ、データフロー、下流書き込みレイテンシを簡単に表示できます。

詳細は[ドキュメント](/performance-tuning-methods.md)を参照してください。

### パフォーマンス

* [INDEX MERGE](/glossary.md#index-merge)が`AND`で接続された式をサポート [#39333](https://github.com/pingcap/tidb/issues/39333) @[guo-shaoge](https://github.com/guo-shaoge) @[time-and-fate](https://github.com/time-and-fate) @[hailanwhu](https://github.com/hailanwhu)

v6.5.0以前では、TiDBは`OR`で接続されたフィルタ条件に対するインデックスマージのみをサポートしていました。v6.5.0からは、`WHERE`句で`AND`に接続されたフィルタ条件に対するインデックスマージをサポートしています。これにより、TiDBのインデックスマージはより一般的なクエリフィルタ条件の組み合わせをカバーできるようになり、ユニオン（`OR`）の関係に限定されなくなります。現在のv6.5.0では、最適化されたインデックスマージは`OR`条件のみを自動的にサポートしています。`AND`条件のインデックスマージを有効にするには、[`USE_INDEX_MERGE`](/optimizer-hints.md#use_index_merget1_name-idx1_name--idx2_name-)ヒントを使用する必要があります。

インデックスマージの詳細については、[v5.4.0リリースノート](/releases/release-5.4.0.md#performance)と[Explain Index Merge](/explain-index-merge.md)を参照してください。
* TiFlashへの次のJSON関数のプッシュダウンをサポートします[#39458](https://github.com/pingcap/tidb/issues/39458) @[yibin87](https://github.com/yibin87)

    * `->`
    * `->>`
    * `JSON_EXTRACT()`

  JSON形式は、アプリケーションデータモデリングの柔軟な方法を提供します。そのため、ますます多くのアプリケーションがデータ交換やデータ保存のためにJSON形式を使用しています。JSON関数をTiFlashにプッシュダウンすることで、JSONタイプのデータを分析する効率を向上させ、より多くのリアルタイムアナリティクスシナリオでTiDBを使用することができます。

* 次の文字列関数のTiFlashへのプッシュダウンをサポートします[#6115](https://github.com/pingcap/tiflash/issues/6115) @[xzhangxian1008](https://github.com/xzhangxian1008)

    * `regexp_like`
    * `regexp_instr`
    * `regexp_substr`

* [ビュー](/views.md)で実行計画生成を干渉させるためのグローバルオプティマイザーヒントをサポートします[#37887](https://github.com/pingcap/tidb/issues/37887) @[Reminiscent](https://github.com/Reminiscent)

    一部のビューアクセスシナリオでは、ビュー内のクエリの実行計画を最適なパフォーマンスを達成するために干渉させるためにオプティマイザーヒントを使用する必要があります。v6.5.0から、TiDBはビュー内のクエリブロックにグローバルヒントを追加することをサポートしており、これによりクエリで定義されたヒントがビューで有効になります。この機能により、ネストされたビューを含む複雑なSQLステートメントにヒントを注入し、実行計画の制御を強化し、複雑なステートメントのパフォーマンスを安定化させる方法が提供されます。グローバルヒントを使用するには、[クエリブロックに名前を付ける](/optimizer-hints.md#step-1-define-the-query-block-name-of-the-view-using-the-qb_name-hint)必要があり、[ヒントリファレンスを指定](/optimizer-hints.md#step-2-add-the-target-hints)する必要があります。

    詳細については、[ドキュメント](/optimizer-hints.md#hints-that-take-effect-globally)を参照してください。

* [パーティション化されたテーブル](/partitioned-table.md)のソーティング操作をTiKVにプッシュダウンするサポートを提供します[#26166](https://github.com/pingcap/tidb/issues/26166) @[winoros](https://github.com/winoros)

    [パーティション化されたテーブル](/partitioned-table.md)機能はv6.1.0以降GAとなっていますが、TiDBは引き続きそのパフォーマンスを改善しています。v6.5.0では、`ORDER BY`や`LIMIT`などのソーティング操作をTiKVにプッシュダウンすることをサポートしており、これによりネットワークI/Oのオーバーヘッドが削減され、パーティション化されたテーブルを使用する際のSQLパフォーマンスが向上します。

* オプティマイザーにより、より正確なコストモデルバージョン2 (GA) を導入します[#35240](https://github.com/pingcap/tidb/issues/35240) @[qw4990](https://github.com/qw4990)

    TiDB v6.2.0では、[Cost Model Version 2](/cost-model.md#cost-model-version-2)を実験的な機能として導入しました。このモデルはより正確なコスト見積り手法を使用してオプティマイザーが最適な実行計画を選択するのを支援します。特にTiFlashが展開されている場合、Cost Model Version 2は適切なストレージエンジンを自動的に選択し、多くの手動干渉を回避します。一定期間の実際のシーンでのテストの後、このモデルはv6.5.0でGAとなりました。v6.5.0からは、新しく作成されたクラスタはデフォルトでCost Model Version 2を使用します。v6.5.0にアップグレードしたクラスタの場合、Cost Model Version 2がクエリプランに変更を引き起こす可能性があるため、十分なパフォーマンステストの後に新しいコストモデルを使用するために、[`tidb_cost_model_version = 2`](/system-variables.md#tidb_cost_model_version-new-in-v620)変数を設定できます。

    Cost Model Version 2は、TiDBオプティマイザーの全体的な機能を大幅に改善し、より強力なHTAPデータベースに向けてTiDBを進化させるのに役立ちます。

    詳細については[ドキュメント](/cost-model.md#cost-model-version-2)を参照してください。

* TiFlashは、テーブル行数取得操作を最適化します[#37165](https://github.com/pingcap/tidb/issues/37165) @[elsa0520](https://github.com/elsa0520)

    データ分析シナリオでは、フィルタ条件のない`COUNT(*)`を使用してテーブルの実際の行数を取得する操作が一般的です。v6.5.0では、TiFlashは`COUNT(*)`のリライトを最適化し、最も短い列定義でNULLでない列を自動的に選択して行数をカウントするようにしており、これによりTiFlashでのI/O操作の数を効果的に減らし、行数を取得する実行効率を向上させます。

### 安定性

* グローバルメモリ制御機能がGAになりました[#37816](https://github.com/pingcap/tidb/issues/37816) @[wshwsh12](https://github.com/wshwsh12)

    v6.4.0以降、TiDBはグローバルメモリ制御を実験的な機能として導入しました。v6.5.0でGAとなり、メインメモリ消費を追跡できるようになりました。グローバルメモリ消費が[`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640)で定義された閾値に達すると、TiDBはGCを行ったりSQL操作をキャンセルしたりしてメモリ使用量を制限します。セッション内のトランザクションによって消費されるメモリ(最大値は以前に[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)で設定されていました)は、現在メモリ管理モジュールによって追跡されます。ひとつのセッションのメモリ消費がシステム変数[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)で定義された閾値に達すると、システム変数[`tidb_mem_oom_action`](/system-variables.md#tidb_mem_oom_action-new-in-v610)で定義された動作(CANCEL(つまり、操作のキャンセル)がデフォルトです)がトリガーされます。将来の互換性を確保するために、[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)がデフォルト値ではない値として構成されている場合、TiDBは引き続き`txn-total-size-limit`で設定されたメモリをトランザクションが使用できるようにします。

    TiDB v6.5.0以降を使用している場合、[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)を削除し、トランザクションのメモリ使用量に個別の制限を設定しないことをお勧めします。代わりにグローバルメモリの管理にシステム変数[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)と[`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640)を使用することで、メモリ使用の効率が向上します。

    詳細については[ドキュメント](/configure-memory-usage.md)を参照してください。

### 利便性

* TiFlashの`TableFullScan`オペレーターの実行情報を`EXPLAIN ANALYZE`出力で洗練します[#5926](https://github.com/pingcap/tiflash/issues/5926) @[hongyunyan](https://github.com/hongyunyan)

    `EXPLAIN ANALYZE`文は実行計画とランタイム統計情報を出力するために使用されます。v6.5.0では、TiFlashは`TableFullScan`オペレーターの実行情報にDMFile関連の実行情報を追加することで、TiFlashデータスキャンのステータス情報をより直感的に表示するようにしました。これによりTiFlashのパフォーマンスをより簡単に分析できるようになります。

    詳細については[ドキュメント](/sql-statements/sql-statement-explain-analyze.md)を参照してください。

* 実行計画をJSON形式で出力するサポートを提供します[#39261](https://github.com/pingcap/tidb/issues/39261) @[fzzf678](https://github.com/fzzf678)

    v6.5.0では、TiDBは実行計画の出力形式を拡張しました。`EXPLAIN`文で`FORMAT = "tidb_json"`を指定することで、SQL実行計画をJSON形式で出力することができます。この機能により、SQLデバッグツールや診断ツールが実行計画をより便利かつ正確に読み取ることができ、その結果、SQLの診断とチューニングの利便性が向上します。

    詳細については[ドキュメント](/sql-statements/sql-statement-explain.md)を参照してください。

### MySQL互換性

* 高性能でグローバルに単調増加する`AUTO_INCREMENT`カラム属性をサポートします(GA) [#38442](https://github.com/pingcap/tidb/issues/38442) @[tiancaiamao](https://github.com/tiancaiamao)

    v6.4.0以降、TiDBは`AUTO_INCREMENT`のMySQL互換モードを実験的な機能として導入しました。このモードでは、すべてのTiDBインスタンスでIDが単調増加することを保証する中央集権的な自動増分ID割り当てサービスが導入されています。この機能により、自動増分IDでクエリ結果を簡単にソートできるようになります。v6.5.0では、この機能がGAとなりました。この機能を使用するには、テーブル作成時に`AUTO_ID_CACHE`を`1`に設定する必要があります。以下は例です。

    ```sql
    CREATE TABLE t(a int AUTO_INCREMENT key) AUTO_ID_CACHE 1;
    ```

    詳細については[ドキュメント](/auto-increment.md#mysql-compatibility-mode)を参照してください。
### データ移行

* サポートされています。データのエクスポートとインポートSQLおよびCSVファイルのgzip、snappy、およびzstd圧縮形式[#38514](https://github.com/pingcap/tidb/issues/38514) @[lichunzhu](https://github.com/lichunzhu)

   Dumplingは、これらの圧縮形式でデータを圧縮したSQLおよびCSVファイルにデータをエクスポートできます：gzip、snappy、およびzstd。TiDB Lightningはまた、これらの形式で圧縮されたファイルのインポートをサポートしています。

   以前は、CSVおよびSQLファイルを保存するためにデータエクスポートまたはインポートに対して大きなストレージスペースを提供する必要があり、高いストレージコストが発生していました。この機能のリリースにより、データファイルを圧縮することでストレージコストを大幅に削減することができます。

   詳細については、[ドキュメンテーション](/dumpling-overview.md#improve-export-efficiency-through-concurrency)を参照してください。

* Binlogの解析能力を最適化[#924](https://github.com/pingcap/dm/issues/924) @[gmhdbjd](https://github.com/GMHDBJD)

   TiDBは、移行タスクに含まれていないスキーマとテーブルのbinlogイベントをフィルタリングし、解析効率と安定性を向上させます。このポリシーはv6.5.0でデフォルトで有効になります。追加の構成は必要ありません。

   以前は、移行されたのがわずかなテーブルであっても、アップストリームの全てのbinlogファイルが解析される必要がありました。移行される必要のないbinlogファイル内のテーブルのbinlogイベントも解析する必要があり、これは非効率でした。また、これらのbinlogイベントが解析をサポートしていない場合、タスクは失敗します。移行タスクのテーブルのbinlogイベントのみ解析することで、binlogの解析効率が大幅に向上し、タスクの安定性が向上します。

* TiDB LightningでのディスククォータがGAに[#446](https://github.com/pingcap/tidb-lightning/issues/446) @[buchuitoudegou](https://github.com/buchuitoudegou)

    TiDB Lightningでディスクのクオータを構成できます。ディスククォータが不十分な場合、TiDB Lightningはソースデータの読み取りと一時ファイルの書き込みを停止し、代わりにソートされたキー値をまずTiKVに書き込み、その後にTiDB Lightningがローカルの一時ファイルを削除した後にインポートプロセスを続行します。

    以前は、TiDB Lightningは物理モードを使用してデータをインポートする際、生データのエンコーディング、ソート、および分割のために大量の一時ファイルをローカルディスクに作成していました。ローカルディスクの容量が不足すると、ファイルへの書き込みに失敗し、TiDB Lightningはエラーで終了しました。この機能により、TiDB Lightningのタスクはローカルディスクへの上書きを回避できます。

    詳細については、[ドキュメンテーション](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md#configure-disk-quota-new-in-v620)を参照してください。

* DMでの連続データ検証がGAに[#4426](https://github.com/pingcap/tiflow/issues/4426) @[D3Hunter](https://github.com/D3Hunter)

    アップストリームからダウンストリームのデータへの増分データの移行の過程で、データフローによってエラーやデータ損失が発生する可能性があります。クレジットや証券取引などのデータ整合性が強く要求されるシナリオでは、移行後にデータの整合性を確認するためにフルボリュームのチェックサムを実行できます。ただし、一部の増分レプリケーションシナリオでは、アップストリームとダウンストリームの書き込みが連続的で中断されずに続いているため、すべてのデータに対する整合性チェックを行うことは難しいです。

    以前は、データを検証するためにビジネスを中断する必要がありましたが、この機能によりビジネスを中断せずに増分データの検証を行うことができます。

    詳細については、[ドキュメンテーション](/dm/dm-continuous-data-validation.md)を参照してください。

### TiDBデータ共有サブスクリプション

* TiCDCは、変更ログをストレージシンクスへレプリケートする機能をサポート（実験的）[#6797](https://github.com/pingcap/tiflow/issues/6797) @[zhaoxinyu](https://github.com/zhaoxinyu)

   TiCDCは、Amazon S3、Azure Blob Storage、NFS、および他のS3互換ストレージサービスに変更ログをレプリケートすることをサポートしています。クラウドストレージは手頃な価格で使いやすいです。Kafkaを使用していない場合はストレージシンクを使用できます。TiCDCは変更ログをファイルに保存し、それをストレージシステムに送信します。ストレージシステムからは、コンシューマプログラムが新しく生成された変更ログファイルを定期的に読み取ります。

   ストレージシンクは、canal-jsonおよびCSV形式の変更ログをサポートしています。詳細については、[ドキュメンテーション](/ticdc/ticdc-sink-to-cloud-storage.md)を参照してください。

* TiCDCは2つのクラスタ間で双方向レプリケーションをサポート[#38587](https://github.com/pingcap/tidb/issues/38587) @[xiongjiwei](https://github.com/xiongjiwei) @[asddongmen](https://github.com/asddongmen)

   TiCDCは、2つのTiDBクラスタ間で双方向レプリケーションをサポートしています。アプリケーションのために地理的に分散された複数のアクティブデータセンターを構築する必要がある場合は、この機能をソリューションとして使用できます。あるTiDBクラスタから別のTiDBクラスタへのTiCDC changefeedに`bdr-mode = true`パラメータを構成することで、2つのTiDBクラスタ間で双方向データレプリケーションを実現できます。

   詳細については、[ドキュメンテーション](/ticdc/ticdc-bidirectional-replication.md)を参照してください。

* TiCDCはTLSのオンライン更新をサポート[#7908](https://github.com/pingcap/tiflow/issues/7908) @[CharlesCheung96](https://github.com/CharlesCheung96)

   データベースシステムのセキュリティを維持するために、システムで使用される証明書の有効期限ポリシーを設定する必要があります。有効期限が切れた後は、新しい証明書が必要です。TiCDC v6.5.0はTLS証明書のオンライン更新をサポートしています。レプリケーションタスクを中断することなく、TiCDCは証明書を自動的に検出および更新できます。手動での介入は必要ありません。

* TiCDCのパフォーマンスが大幅に向上[#7540](https://github.com/pingcap/tiflow/issues/7540) [#7478](https://github.com/pingcap/tiflow/issues/7478) [#7532](https://github.com/pingcap/tiflow/issues/7532) @[sdojjy](https://github.com/sdojjy) @[3AceShowHand](https://github.com/3AceShowHand)

   TiDBクラスタのテストシナリオでは、TiCDCのパフォーマンスが大幅に向上しています。特に、[Kafkaへのデータレプリケーション](/replicate-data-to-kafka.md)シナリオでは、単一のTiCDCが処理できる最大行数変更が30K行/秒に達し、レプリケーションの遅延が10秒に短縮されています。さらに、TiKVおよびTiCDCのローリングアップグレード中でも、レプリケーションの遅延は30秒未満です。

   災害対策（DR）シナリオでは、TiCDCのリトライログとSyncpointが有効になっている場合、TiCDCのスループットは4000行/秒から35000行/秒に向上し、レプリケーションの遅延を2秒まで制限できます。

### バックアップとリストア

* TiDB Backup & Restoreはスナップショットチェックポイントバックアップをサポート[#38647](https://github.com/pingcap/tidb/issues/38647) @[Leavrth](https://github.com/Leavrth)

   TiDBスナップショットバックアップは、チェックポイントからバックアップを再開することをサポートしています。Backup & Restore（BR）が回復可能なエラーに遭遇すると、バックアップを再試行します。ただし、複数回のリトライが失敗した場合にはBRが終了します。チェックポイントバックアップ機能により、長期間の回復可能な失敗、例えば数十分間のネットワークの障害などが再試行されることが可能になります。

   BRが終了した後1時間以内にシステムを回復しない場合、バックアップを失敗させる可能性があるため、バックアップデータがGCメカニズムによって再利用される可能性があります。詳細については、[ドキュメンテーション](/br/br-checkpoint-backup.md#backup-retry-must-be-prior-to-gc)を参照してください。

* PITRのパフォーマンスが著しく向上しました@[joccau](https://github.com/joccau)

   ログのリストア段階では、TiKVのリストア速度は9 MiB/sに達しました。これは以前より50%高速です。リストア速度は拡張可能であり、災害復旧（DR）シナリオでのRTOが大幅に削減されています。DRシナリオのRPOは5分間まで短縮できます。通常のクラスタの運用およびメンテナンス（OM）では、例えばローリングアップグレードが行われたりTiKVが1台ダウンしている場合、RPOは5分になります。

* TiKV-BR GA：RawKVのバックアップとリストアをサポート[#67](https://github.com/tikv/migration/issues/67) @[pingyu](https://github.com/pingyu) @[haojinming](https://github.com/haojinming)

    TiKV-BRはTiKVクラスタで使用されるバックアップおよびリストアツールです。TiKVとPDは、TiDBを使用せずにKVデータベースを構成することができます。これをRawKVと呼びます。TiKV-BRは、RawKVを使用する製品のデータバックアップとリストアをサポートしています。TiKV-BRはまた、TiKVクラスタ向けに[`api-version`](/tikv-configuration-file.md#api-version-new-in-v610)を`API V1`から`API V2`にアップグレードできます。

    詳細については、[ドキュメンテーション](https://tikv.org/docs/latest/concepts/explore-tikv-features/backup-restore/)を参照してください。

## 互換性の変更

### システム変数

| 変数名 | 変更タイプ | 説明 |

|--------|------------------------------|------|
|`tidb_enable_amend_pessimistic_txn`|非推奨|v6.5.0から、この変数は非推奨となり、TiDBはデフォルトで[メタデータロック](/metadata-lock.md)機能を使用して`Information schema is changed`エラーを避けます。|
| [`tidb_enable_outer_join_reorder`](/system-variables.md#tidb_enable_outer_join_reorder-new-in-v610) |修正済み|さらなるテストの後、デフォルト値を`OFF`から`ON`に変更し、[Join Reorder](/join-reorder.md)アルゴリズムでのOuter Joinサポートをデフォルトで有効にします。|
| [`tidb_cost_model_version`](/system-variables.md#tidb_cost_model_version-new-in-v620) |修正済み|さらなるテストの後、デフォルト値を`1`から`2`に変更し、Cost Model Version 2がデフォルトでインデックス選択とオペレータ選択に使用されるようになります。|
| [`tidb_enable_gc_aware_memory_track`](/system-variables.md#tidb_enable_gc_aware_memory_track) |修正済み|デフォルト値を`ON`から`OFF`に変更します。テストでGC-aware memory trackが不正確であることがわかり、追跡される分析メモリサイズがあまりにも大きいため、メモリ追跡が無効になります。また、Golang 1.19では、GC-aware memory trackによって追跡されるメモリが全体のメモリにほとんど影響を与えません。|
| [`tidb_enable_metadata_lock`](/system-variables.md#tidb_enable_metadata_lock-new-in-v630) |修正済み|さらなるテストの後、デフォルト値を`OFF`から`ON`に変更し、メタデータロック機能がデフォルトで有効になります。|
| [`tidb_enable_tiflash_read_for_write_stmt`](/system-variables.md#tidb_enable_tiflash_read_for_write_stmt-new-in-v630) |修正済み|v6.5.0から有効になります。作業中のSQLステートメントに含まれる`INSERT`、`DELETE`、および`UPDATE`操作の読み取り操作をTiFlashにプッシュダウンできるかどうかを制御します。デフォルト値は`OFF`です。|
| [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630) |修正済み|さらなるテストの後、デフォルト値を`OFF`から`ON`に変更し、`ADD INDEX`および`CREATE INDEX`の高速化がデフォルトで有効になります。|
| [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) |修正済み|TiDB v6.5.0以前のバージョンでは、この変数はクエリのメモリクォータの閾値値を設定するために使用されます。TiDB v6.5.0以降のバージョンでは、DMLステートメントのメモリをより正確に制御するために、この変数はセッションのメモリクォータの閾値値を設定するために使用されます。|
| [`tidb_replica_read`](/system-variables.md#tidb_replica_read-new-in-v40) |修正済み|v6.5.0から、TiDBノード間の負荷分散を最適化するために、この変数が`closest-adaptive`に設定され、読み取りリクエストの推定結果が[`tidb_adaptive_closest_read_threshold`](/system-variables.md#tidb_adaptive_closest_read_threshold-new-in-v630)以上の場合、`closest-adaptive`構成が各可用性ゾーンで有効になるTiDBノードの数が制限され、常に最も少ないTiDBノード数の可用性ゾーンの数と同じになります。他のTiDBノードは自動的にリーダーレプリカから読み取ります。|
| [`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640) |修正済み|デフォルト値を`0`から`80%`に変更します。TiDBのグローバルメモリ制御がGAになることに伴い、このデフォルト値の変更により、デフォルトでメモリ制御が有効になり、TiDBインスタンスのメモリ制限がデフォルトで合計メモリの80%に設定されます。|
| [`default_password_lifetime`](/system-variables.md#default_password_lifetime-new-in-v650) |新しく追加|自動的なパスワードの有効期限ポリシーを設定し、ユーザーにパスワードの定期的な変更を要求します。デフォルト値`0`は、パスワードの有効期限が設定されていないことを示します。|
| [`disconnect_on_expired_password`](/system-variables.md#disconnect_on_expired_password-new-in-v650) |新しく追加|パスワードが期限切れの場合、TiDBがクライアント接続を切断するかどうかを示します。この変数は読み取り専用です。|
| [`password_history`](/system-variables.md#password_history-new-in-v650) |新しく追加|パスワード変更の回数に基づいてパスワード再利用を制限するパスワード再利用ポリシーを確立するために使用されます。デフォルト値`0`は、パスワードの再利用ポリシーが無効になっていることを意味します。|
| [`password_reuse_interval`](/system-variables.md#password_reuse_interval-new-in-v650) |新しく追加|時間経過に基づいてパスワード再利用を制限するパスワード再利用ポリシーを確立するために使用されます。デフォルト値`0`は、時間経過に基づくパスワード再利用ポリシーが無効になっていることを意味します。|
| [`tidb_auto_build_stats_concurrency`](/system-variables.md#tidb_auto_build_stats_concurrency-new-in-v650) |新しく追加|統計情報の自動更新の実行並列性を設定するために使用されます。デフォルト値は`1`です。|
| [`tidb_cdc_write_source`](/system-variables.md#tidb_cdc_write_source-new-in-v650) |新しく追加|この変数が0以外の値に設定されると、このセッションで書き込まれたデータはTiCDCによって書き込まれたものと見なされます。この変数はTiCDCによってのみ変更できます。いかなる場合もこの変数を手動で変更しないでください。|
| [`tidb_index_merge_intersection_concurrency`](/system-variables.md#tidb_index_merge_intersection_concurrency-new-in-v650) |新しく追加|インデックスマージが実行する交差操作の最大並列性を設定します。TiDBが動的プルーニングモードでパーティションテーブルにアクセスする場合のみ有効です。|
| [`tidb_source_id`](/system-variables.md#tidb_source_id-new-in-v650) |新しく追加|この変数は、[双方向レプリケーション](/ticdc/ticdc-bidirectional-replication.md)クラスタ内の異なるクラスタIDを構成するために使用されます。|
| [`tidb_sysproc_scan_concurrency`](/system-variables.md#tidb_sysproc_scan_concurrency-new-in-v650) |新しく追加|TiDBが内部SQLステートメント（統計情報の自動更新など）を実行する際に実行されるスキャン操作の並列性を設定するために使用されます。デフォルト値は`1`です。|
| [`tidb_ttl_delete_batch_size`](/system-variables.md#tidb_ttl_delete_batch_size-new-in-v650) |新しく追加|この変数はTTLジョブの単一の`DELETE`トランザクションで削除できる最大行数を設定するために使用されます。|
| [`tidb_ttl_delete_rate_limit`](/system-variables.md#tidb_ttl_delete_rate_limit-new-in-v650) |新しく追加|この変数はTTLジョブの単一ノードあたりに許可される最大`DELETE`ステートメント数を制限するために使用されます。この変数を`0`に設定すると、制限は適用されません。|
| [`tidb_ttl_delete_worker_count`](/system-variables.md#tidb_ttl_delete_worker_count-new-in-v650) |新しく追加|この変数は各TiDBノードでのTTLジョブの最大並列性を設定するために使用されます。|
| [`tidb_ttl_job_enable`](/system-variables.md#tidb_ttl_job_enable-new-in-v650) |新しく追加|この変数はTTLジョブを有効にするかどうかを制御するために使用されます。`OFF`に設定すると、TTL属性を持つすべてのテーブルは自動的に期限切れのデータのクリーンアップを停止します。|
| `tidb_ttl_job_run_interval` |新しく追加|この変数はバックグラウンドでのTTLジョブのスケジューリング間隔を制御するために使用されます。たとえば、現在の値が`1h0m0s`に設定されている場合、TTL属性を持つ各テーブルは1時間ごとに期限切れのデータをクリーンアップします。|
| [`tidb_ttl_job_schedule_window_start_time`](/system-variables.md#tidb_ttl_job_schedule_window_start_time-new-in-v650) |新しく追加|この変数はバックグラウンドでのTTLジョブのスケジューリングウィンドウの開始時刻を制御するために使用されます。この変数の値を変更する際には注意してください。短いウィンドウでは期限切れのデータのクリーンアップが失敗する可能性があります。|
| [`tidb_ttl_job_schedule_window_end_time`](/system-variables.md#tidb_ttl_job_schedule_window_end_time-new-in-v650) |新しく追加|この変数はバックグラウンドでのTTLジョブのスケジューリングウィンドウの終了時刻を制御するために使用されます。この変数の値を変更する際には注意してください。短いウィンドウでは期限切れのデータのクリーンアップが失敗する可能性があります。|
| [`tidb_ttl_scan_batch_size`](/system-variables.md#tidb_ttl_scan_batch_size-new-in-v650) |新しく追加|この変数はTTLジョブで期限切れデータをスキャンするために使用される各`SELECT`ステートメントの`LIMIT`値を設定するために使用されます。|
| [`tidb_ttl_scan_worker_count`](/system-variables.md#tidb_ttl_scan_worker_count-new-in-v650) |新しく追加|この変数は各TiDBノードでのTTLスキャンジョブの最大並列性を設定するために使用されます。|
| [`validate_password.check_user_name`](/system-variables.md#validate_passwordcheck_user_name-new-in-v650) | 追加 | パスワードの複雑さチェック項目。パスワードがユーザー名と一致するかどうかをチェックします。この変数は、[`validate_password.enable`](/system-variables.md#validate_passwordenable-new-in-v650) が有効になっている場合のみ有効です。デフォルト値は `ON` です。 |
| [`validate_password.dictionary`](/system-variables.md#validate_passworddictionary-new-in-v650) | 追加 | パスワードの複雑さチェック項目。パスワードが辞書内の単語に一致するかどうかをチェックします。この変数は、[`validate_password.enable`](/system-variables.md#validate_passwordenable-new-in-v650) が有効になっていて、[`validate_password.policy`](/system-variables.md#validate_passwordpolicy-new-in-v650) が `2` (STRONG) に設定されている場合のみ有効です。デフォルト値は `""` です。 |
| [`validate_password.enable`](/system-variables.md#validate_passwordenable-new-in-v650) | 追加 | この変数は、パスワードの複雑さチェックを実行するかどうかを制御します。この変数が `ON` に設定されている場合、TiDB はパスワードの複雑さチェックを行います。デフォルト値は `OFF` です。 |
| [`validate_password.length`](/system-variables.md#validate_passwordlength-new-in-v650) | 追加 | パスワードの複雑さチェック項目。パスワードの長さが十分かどうかをチェックします。デフォルトでは、最小パスワード長は `8` です。この変数は、[`validate_password.enable`](/system-variables.md#validate_passwordenable-new-in-v650) が有効になっている場合のみ有効です。 |
| [`validate_password.mixed_case_count`](/system-variables.md#validate_passwordmixed_case_count-new-in-v650) | 追加 | パスワードの複雑さチェック項目。パスワードに十分な大文字と小文字が含まれているかどうかをチェックします。この変数は、[`validate_password.enable`](/system-variables.md#validate_passwordenable-new-in-v650) が有効になっていて、[`validate_password.policy`](/system-variables.md#validate_passwordpolicy-new-in-v650) が `1` (MEDIUM) 以上に設定されている場合のみ有効です。デフォルト値は `1` です。 |
| [`validate_password.number_count`](/system-variables.md#validate_passwordnumber_count-new-in-v650) | 追加 | パスワードの複雑さチェック項目。パスワードに十分な数値が含まれているかどうかをチェックします。この変数は、[`validate_password.enable`](/system-variables.md#password_reuse_interval-new-in-v650) が有効になっていて、[`validate_password.policy`](/system-variables.md#validate_passwordpolicy-new-in-v650) が `1` (MEDIUM) 以上に設定されている場合のみ有効です。デフォルト値は `1` です。 |
| [`validate_password.policy`](/system-variables.md#validate_passwordpolicy-new-in-v650) | 追加 | この変数は、パスワードの複雑さチェックポリシーを制御します。値は `0`、`1`、または `2`（LOW、MEDIUM、またはSTRONGに対応）であることができます。この変数は、[`validate_password.enable`](/system-variables.md#password_reuse_interval-new-in-v650) が有効になっている場合のみ有効です。デフォルト値は `1` です。 |
| [`validate_password.special_char_count`](/system-variables.md#validate_passwordspecial_char_count-new-in-v650) | 追加 | パスワードの複雑さチェック項目。パスワードに十分な特殊文字が含まれているかどうかをチェックします。この変数は、[`validate_password.enable`](/system-variables.md#password_reuse_interval-new-in-v650) が有効になっていて、[`validate_password.policy`](/system-variables.md#validate_passwordpolicy-new-in-v650) が `1` (MEDIUM) 以上に設定されている場合のみ有効です。デフォルト値は `1` です。 |

### 設定ファイルパラメータ

| 設定ファイル | 設定パラメータ | 変更タイプ | 説明 |
| -------- | -------- | -------- | -------- |
| TiDB | [`server-memory-quota`](/tidb-configuration-file.md#server-memory-quota-new-in-v409) | 廃止 | v6.5.0からこの設定項目は廃止されました。代わりに、メモリをグローバルに管理するためにシステム変数[`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640)を使用してください。 |
| TiDB | [`disconnect-on-expired-password`](/tidb-configuration-file.md#disconnect-on-expired-password-new-in-v650) | 追加 | パスワードの有効期限が切れた場合にTiDBがクライアント接続を切断するかどうかを決定します。デフォルト値は `true` で、これはパスワードの有効期限が切れた場合にクライアント接続が切断されることを意味します。 |
| TiKV | `raw-min-ts-outlier-threshold` | 削除 | この設定項目はv6.4.0で廃止され、v6.5.0で削除されました。 |
| TiKV | [`cdc.min-ts-interval`](/tikv-configuration-file.md#min-ts-interval) | 変更 | CDCの遅延を緩和するため、デフォルト値を `1s` から `200ms` に変更しました。 |
| TiKV | [`memory-use-ratio`](/tikv-configuration-file.md#memory-use-ratio-new-in-v650) | 追加 | PITRログのリカバリで利用可能なメモリの総システムメモリに対する比率を示します。 |
| TiCDC | [`sink.terminator`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 追加 | 2つのデータ変更イベントを区切るために使用される行終端記号を示します。デフォルト値は空で、これは `\r\n` が使用されることを意味します。 |
| TiCDC | [`sink.date-separator`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 追加 | ファイルディレクトリの日付区切り文字種類を示します。値のオプションは `none`、`year`、`month`、`day` です。 `none` がデフォルト値であり、日付が区切られていないことを意味します。 |
| TiCDC | [`sink.enable-partition-separator`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 追加 | パーティションを区切り文字列として使用するかどうかを指定します。デフォルト値は `false` で、これはテーブルのパーティションが別のディレクトリに保存されないことを意味します。 |
| TiCDC | [`sink.csv.delimiter`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 追加 | フィールド間の区切り文字を示します。値はASCII文字である必要があり、デフォルト値は`,`です。 |
| TiCDC | [`sink.csv.quote`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 追加 | フィールドを囲む引用符を示します。デフォルト値は `"` です。値が空の場合、引用符は使用されません。 |
| TiCDC | [`sink.csv.null`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 追加 | CSV列がnullのときに表示される文字を指定します。デフォルト値は `\N` です。|
| TiCDC | [`sink.csv.include-commit-ts`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 追加 | CSV行にcommit-tsを含めるかどうかを指定します。デフォルト値は `false` です。 |

### その他

- v6.5.0から、`mysql.user` テーブルに新たに `Password_reuse_history` と `Password_reuse_time` の2つの新しい列が追加されました。
- v6.5.0から、[index acceleration](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630) 機能がデフォルトで有効になりました。この機能は、単一の `ALTER TABLE` ステートメントで複数の列やインデックスの変更を行うときに完全に互換性がありません。インデックスのアクセラレーションを使用して一意のインデックスを追加する場合、同じステートメントで他の列やインデックスを変更しないようにする必要があります。この機能は[PITR（ポイントインタイムリカバリ）](/br/br-pitr-guide.md)とも互換性がありません。インデックスアクセラレーション機能を使用する場合は、バックグラウンドで実行中のPITRバックアップタスクがないことを確認する必要があります。そうしないと予期しない結果が発生する可能性があります。詳細は[ドキュメント](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)を参照してください。

## 廃止された機能

v6.5.0から、v4.0.7で導入された `AMEND TRANSACTION` メカニズムは廃止され、[メタデータロック](/metadata-lock.md)に置き換えられました。

## 改善点

+ TiDB

    - `BIT` および `CHAR` 列について、`INFORMATION_SCHEMA.COLUMNS` の結果をMySQLと一貫性があるように修正しました [#25472](https://github.com/pingcap/tidb/issues/25472) @[hawkingrei](https://github.com/hawkingrei)
    - TiFlash MPPモードでTiFlashノードのTiDBプロービングメカニズムを最適化し、ノードが異常な場合のパフォーマンス影響を緩和しました[#39686](https://github.com/pingcap/tidb/issues/39686) @[hackersean](https://github.com/hackersean)

+ TiKV

    - ディスク容量が不足している場合、Raft Engineへの書き込みを停止してディスク容量を尽きることを避けるようにしました[#13642](https://github.com/tikv/tikv/issues/13642) @[jiayang-zheng](https://github.com/jiayang-zheng)
    - `json_valid` 関数をTiKVにプッシュダウンするサポートを追加しました [#13571](https://github.com/tikv/tikv/issues/13571) @[lizhenhuan](https://github.com/lizhenhuan)
- シングルバックアップリクエストで複数のデータ範囲をバックアップする機能をサポートする[#13701](https://github.com/tikv/tikv/issues/13701) @[Leavrth](https://github.com/Leavrth)
- ラスオトライブラリの更新により、AWSのアジア太平洋地域（ap-southeast-3）にデータをバックアップする機能をサポートする[#13751](https://github.com/tikv/tikv/issues/13751) @[3pointer](https://github.com/3pointer)
- 悲観的トランザクションの競合を減らす[#13298](https://github.com/tikv/tikv/issues/13298) @[MyonKeminta](https://github.com/MyonKeminta)
- 外部ストレージオブジェクトをキャッシュすることで復旧性能を向上させる[#13798](https://github.com/tikv/tikv/issues/13798) @[YuJuncen](https://github.com/YuJuncen)
- TiCDC複製遅延を減らすために、CheckLeaderを専用スレッドで実行する[#13774](https://github.com/tikv/tikv/issues/13774) @[overvenus](https://github.com/overvenus)
- Checkpointsのプルモデルをサポートする[#13824](https://github.com/tikv/tikv/issues/13824) @[YuJuncen](https://github.com/YuJuncen)
- crossbeam-channelの更新により、sender側でのスピン問題を避ける[#13815](https://github.com/tikv/tikv/issues/13815) @[sticnarf](https://github.com/sticnarf)
- TiKVでのバッチCoprocessorタスク処理をサポートする[#13849](https://github.com/tikv/tikv/issues/13849) @[cfzjywxk](https://github.com/cfzjywxk)
- 失敗復旧時の待機時間を減らすために、TiKVにリージョンを起こすよう通知する[#13648](https://github.com/tikv/tikv/issues/13648) @[LykxSassinator](https://github.com/LykxSassinator)
- コード最適化により、メモリ使用量のリクエストサイズを減らす[#13827](https://github.com/tikv/tikv/issues/13827) @[BusyJay](https://github.com/BusyJay)
- コードの拡張可能性を向上させるために、Raft拡張を導入する[#13827](https://github.com/tikv/tikv/issues/13827) @[BusyJay](https://github.com/BusyJay)
- tikv-ctlを使用して特定のキー範囲に含まれるリージョンを問い合わせる機能をサポートする[#13760](https://github.com/tikv/tikv/issues/13760) @[HuSharp](https://github.com/HuSharp)
- 更新されないが連続してロックされた行の読み込みと書き込みのパフォーマンスを向上させる[#13694](https://github.com/tikv/tikv/issues/13694) @[sticnarf](https://github.com/sticnarf)

+ PD

    - 高並行性下でのロック競合を減らし、ハートビートの処理能力を向上させるために、ロックの粒度を最適化する[#5586](https://github.com/tikv/pd/issues/5586) @[rleungx](https://github.com/rleungx)
    - 大規模クラスタ向けのスケジューラパフォーマンスを最適化し、スケジューリングポリシーの生成を加速する[#5473](https://github.com/tikv/pd/issues/5473) @[bufferflies](https://github.com/bufferflies)
    - リージョンの読み込み速度を改善する[#5606](https://github.com/tikv/pd/issues/5606) @[rleungx](https://github.com/rleungx)
    - リージョンのハートビートの処理を最適化することで不要なオーバーヘッドを減らす[#5648](https://github.com/tikv/pd/issues/5648) @[rleungx](https://github.com/rleungx)
    - 自動的にトンブストアをガベージコレクションする機能を追加する[#5348](https://github.com/tikv/pd/issues/5348) @[nolouch](https://github.com/nolouch)

+ TiFlash

    - SQL側でバッチ処理がないシナリオにおける書き込みパフォーマンスを改善する[#6404](https://github.com/pingcap/tiflash/issues/6404) @[lidezhu](https://github.com/lidezhu)
    - `explain analyze`出力にTableFullScanの詳細を追加する[#5926](https://github.com/pingcap/tiflash/issues/5926) @[hongyunyan](https://github.com/hongyunyan)

+ Tools

    + TiDB ダッシュボード

        - 遅いクエリページに3つの新しいフィールド「Is Prepared?」「Is Plan from Cache?」「Is Plan from Binding?」を追加する[#1451](https://github.com/pingcap/tidb-dashboard/issues/1451) @[shhdgit](https://github.com/shhdgit)

    + バックアップ&リストア（BR）

        - バックアップログデータのクリーニングプロセス中のBRメモリ使用量を最適化する[#38869](https://github.com/pingcap/tidb/issues/38869) @[Leavrth](https://github.com/Leavrth)
        - リストア処理中のPDリーダースイッチによる復元失敗を修正する[#36910](https://github.com/pingcap/tidb/issues/36910) @[MoCuishle28](https://github.com/MoCuishle28)
        - ログバックアップにおいてOpenSSLプロトコルを使用することでTLS互換性を向上させる[#13867](https://github.com/tikv/tikv/issues/13867) @[YuJuncen](https://github.com/YuJuncen)

    + TiCDC

        - Kafkaプロトコルエンコーダのパフォーマンスを改善する[#7540](https://github.com/pingcap/tiflow/issues/7540) [#7532](https://github.com/pingcap/tiflow/issues/7532) [#7543](https://github.com/pingcap/tiflow/issues/7543) @[3AceShowHand](https://github.com/3AceShowHand) @[sdojjy](https://github.com/sdojjy)

    + TiDB データ移行（DM）

        - ブロックリストのテーブルデータを解析しないことで、DMのデータレプリケーションパフォーマンスを向上させる[#7622](https://github.com/pingcap/tiflow/pull/7622) @[GMHDBJD](https://github.com/GMHDBJD)
        - 非同期書き込みとバッチ書き込みを使用することでDMリレーの書き込み効率を向上させる[#7580](https://github.com/pingcap/tiflow/pull/7580) @[GMHDBJD](https://github.com/GMHDBJD)
        - DM precheckのエラーメッセージを最適化する[#7621](https://github.com/pingcap/tiflow/issues/7621) @[buchuitoudegou](https://github.com/buchuitoudegou)
        - 古いMySQLバージョンの`SHOW SLAVE HOSTS`の互換性を向上させる[#5017](https://github.com/pingcap/tiflow/issues/5017) @[lyzx2001](https://github.com/lyzx2001)

## バグ修正

+ TiDB

    - 一部の場合で発生するチャンク再利用機能のメモリチャンクの誤用の問題を修正する[#38917](https://github.com/pingcap/tidb/issues/38917) @[keeplearning20221](https://github.com/keeplearning20221)
    - `tidb_constraint_check_in_place_pessimistic`の内部セッションがグローバル設定に影響を受ける問題を修正する[#38766](https://github.com/pingcap/tidb/issues/38766) @[ekexium](https://github.com/ekexium)
    - `AUTO_INCREMENT`列が`CHECK`制約と連携できない問題を修正する[#38894](https://github.com/pingcap/tidb/issues/38894) @[YangKeao](https://github.com/YangKeao)
    - `INSERT IGNORE INTO`を使用して`STRING`タイプのデータを`SMALLINT`タイプの自動増分列に挿入するとエラーが発生する問題を修正する[#38483](https://github.com/pingcap/tidb/issues/38483) @[hawkingrei](https://github.com/hawkingrei)
    - パーティションされたテーブルのパーティション列を変更する操作でヌルポインタエラーが発生する問題を修正する[#38932](https://github.com/pingcap/tidb/issues/38932) @[mjonss](https://github.com/mjonss)
    - パーティションテーブルのパーティション列を変更するとDDLがハングする問題を修正する[#38530](https://github.com/pingcap/tidb/issues/38530) @[mjonss](https://github.com/mjonss)
    - v4.0.16からv6.4.0にアップグレード後に`ADMIN SHOW JOB`操作がパニックする問題を修正する[#38980](https://github.com/pingcap/tidb/issues/38980) @[tangenta](https://github.com/tangenta)
    - `tidb_decode_key`関数がパーティションテーブルのエンコーディングを正しく解析できない問題を修正する[#39304](https://github.com/pingcap/tidb/issues/39304) @[Defined2014](https://github.com/Defined2014)
- gRPCエラーログがログローテーション中に正しいログファイルにリダイレクトされない問題を修正 [#38941](https://github.com/pingcap/tidb/issues/38941) @[xhebox](https://github.com/xhebox)
    - TiDBがreadエンジンとして構成されていない場合に`BEGIN; SELECT... FOR UPDATE;`ポイントクエリの予期しない実行計画を生成する問題を修正 [#39344](https://github.com/pingcap/tidb/issues/39344) @[Yisaer](https://github.com/Yisaer)
    - 誤って`StreamAgg`をTiFlashにプッシュダウンすることで誤った結果が発生する問題を修正 [#39266](https://github.com/pingcap/tidb/issues/39266) @[fixdb](https://github.com/fixdb)

+ TiKV

    - Raft Engine ctlでのエラーを修正 [#11119](https://github.com/tikv/tikv/issues/11119) @[tabokie](https://github.com/tabokie)
    - `compact raft`コマンドを実行する際に`Get raft db is not allowed`エラーが発生する問題を修正 [#13515](https://github.com/tikv/tikv/issues/13515) @[guoxiangCN](https://github.com/guoxiangCN)
    - TLSが有効になっているとログバックアップが機能しない問題を修正 [#13867](https://github.com/tikv/tikv/issues/13867) @[YuJuncen](https://github.com/YuJuncen)
    - Geometryフィールドタイプのサポート問題を修正 [#13651](https://github.com/tikv/tikv/issues/13651) @[dveeden](https://github.com/dveeden)
    - 新しい照合順序が有効にされていない場合に`LIKE`演算子の`_`が非ASCII文字に一致しない問題を修正 [#13769](https://github.com/tikv/tikv/issues/13769) @[YangKeao](https://github.com/YangKeao)
    - `reset-to-version`コマンドを実行する際にtikv-ctlが予期せず終了する問題を修正 [#13829](https://github.com/tikv/tikv/issues/13829) @[tabokie](https://github.com/tabokie)

+ PD

    - `balance-hot-region-scheduler`の設定が変更されていない場合に永続化されない問題を修正 [#5701](https://github.com/tikv/pd/issues/5701) @[HunDunDM](https://github.com/HunDunDM)
    - アップグレードプロセス中に`rank-formula-version`がプリアップグレード構成を保持しない問題を修正 [#5698](https://github.com/tikv/pd/issues/5698) @[HunDunDM](https://github.com/HunDunDM)

+ TiFlash

    - TiFlashを再起動した後にデルタレイヤーのカラムファイルをコンパクトできない問題を修正 [#6159](https://github.com/pingcap/tiflash/issues/6159) @[lidezhu](https://github.com/lidezhu)
    - TiFlashファイルオープンOPSが高すぎる問題を修正 [#6345](https://github.com/pingcap/tiflash/issues/6345) @[JaySon-Huang](https://github.com/JaySon-Huang)

+ Tools

    + バックアップ＆リストア (BR)

        - BRが削除すべきでないデータを誤って削除する問題を修正 [#38939](https://github.com/pingcap/tidb/issues/38939) @[Leavrth](https://github.com/Leavrth)
        - 古いフレームワークを使用するとリストアタスクが失敗する問題を修正 [#39150](https://github.com/pingcap/tidb/issues/39150) @[MoCuishle28](https://github.com/MoCuishle28)
        - Alibaba CloudとHuawei CloudがAmazon S3ストレージと完全に互換性がないため、バックアップが失敗する問題を修正 [#39545](https://github.com/pingcap/tidb/issues/39545) @[3pointer](https://github.com/3pointer)

    + TiCDC

        - TiCDCがPDリーダーがクラッシュした場合にスタックする問題を修正 [#7470](https://github.com/pingcap/tiflow/issues/7470) @[zeminzhou](https://github.com/zeminzhou)
        - DDLステートメントを最初に実行し、その後Changefeedを一時停止して再開するシナリオでデータ損失が発生する問題を修正 [#7682](https://github.com/pingcap/tiflow/issues/7682) @[asddongmen](https://github.com/asddongmen)
        - TiCDCが誤ってエラーを報告する問題を修正 [#7744](https://github.com/pingcap/tiflow/issues/7744) @[overvenus](https://github.com/overvenus)
        - 下流ネットワークが利用できない場合にシンクコンポーネントがスタックする問題を修正 [#7706](https://github.com/pingcap/tiflow/issues/7706) @[hicqu](https://github.com/hicqu)
        - ユーザーがレプリケーションタスクを速やかに削除し、同じタスク名で別のタスクを作成した場合にデータが失われる問題を修正 [#7657](https://github.com/pingcap/tiflow/issues/7657) @[overvenus](https://github.com/overvenus)

    + TiDBデータマイグレーション (DM)

        - アップストリームデータベースがGTIDモードを有効にしていてもデータがない場合に`task-mode:all`タスクを開始できない問題を修正 [#7037](https://github.com/pingcap/tiflow/issues/7037) @[liumengya94](https://github.com/liumengya94)
        - 既存のワーカーが終了する前に新しいDMワーカーがスケジュールされるとデータが複数回複製される問題を修正 [#7658](https://github.com/pingcap/tiflow/issues/7658) @[GMHDBJD](https://github.com/GMHDBJD)
        - アップストリームデータベースが正規表現を使用して権限を付与する場合にDM事前チェックが合格しない問題を修正 [#7645](https://github.com/pingcap/tiflow/issues/7645) @[lance6716](https://github.com/lance6716)

    + TiDB Lightning

        - TiDB Lightningが巨大なソースデータファイルをインポートする際にメモリリークが発生する問題を修正 [#39331](https://github.com/pingcap/tidb/issues/39331) @[dsdashun](https://github.com/dsdashun)
        - データを並行してインポートする際にTiDB Lightningが衝突を正しく検出できない問題を修正 [#39476](https://github.com/pingcap/tidb/issues/39476) @[dsdashun](https://github.com/dsdashun)

## コントリビューター

TiDBコミュニティから以下のコントリビューターに感謝します:

- [e1ijah1](https://github.com/e1ijah1)
- [guoxiangCN](https://github.com/guoxiangCN) (初めての貢献者)
- [jiayang-zheng](https://github.com/jiayang-zheng)
- [jiyfhust](https://github.com/jiyfhust)
- [mikechengwei](https://github.com/mikechengwei)
- [pingandb](https://github.com/pingandb)
- [sashashura](https://github.com/sashashura)
- [sourcelliu](https://github.com/sourcelliu)
- [wxbty](https://github.com/wxbty)