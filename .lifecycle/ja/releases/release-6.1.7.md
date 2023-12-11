---
title: TiDB 6.1.7 リリースノート
summary: TiDB 6.1.7 の改善とバグ修正について学びます。

# TiDB 6.1.7 リリースノート

リリース日: 2023年7月12日

TiDB バージョン: 6.1.7

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.1/quick-start-with-tidb) | [本番環境展開](https://docs.pingcap.com/tidb/v6.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.1.7#version-list)

## 改善点

+ TiDB

    - 内部トランザクションのリトライで悲観的トランザクションを使用して、リトライ失敗を回避し、時間を短縮するための改善 [#38136](https://github.com/pingcap/tidb/issues/38136) @[jackysp](https://github.com/jackysp)

+ ツール

    + TiCDC

        - バッチ `UPDATE` DML ステートメントをサポートし、TiCDC レプリケーションのパフォーマンスを向上させる [#8084](https://github.com/pingcap/tiflow/issues/8084) @[amyangfei](https://github.com/amyangfei)

    + TiDB Lightning

        - インポート後に SQL でチェックサムを検証し、安定性を向上させるための改善 [#41941](https://github.com/pingcap/tidb/issues/41941) @[GMHDBJD](https://github.com/GMHDBJD)

## バグ修正

+ TiDB

    - 空の `processInfo` によって引き起こされるパニックの問題を修正 [#43829](https://github.com/pingcap/tidb/issues/43829) @[zimulala](https://github.com/zimulala)
    - PD の時間が急激に変化すると `resolve lock` がハングアップする可能性のある問題を修正 [#44822](https://github.com/pingcap/tidb/issues/44822) @[zyguan](https://github.com/zyguan)
    - 共通テーブル式（CTE）を含むクエリがディスクの空き容量不足を引き起こす可能性のある問題を修正 [#44477](https://github.com/pingcap/tidb/issues/44477) @[guo-shaoge](https://github.com/guo-shaoge)
    - CTE と相関サブクエリを同時に使用すると、クエリ結果が正しくないかパニックが発生する可能性のある問題を修正 [#44649](https://github.com/pingcap/tidb/issues/44649) [#38170](https://github.com/pingcap/tidb/issues/38170) [#44774](https://github.com/pingcap/tidb/issues/44774) @[winoros](https://github.com/winoros) @[guo-shaoge](https://github.com/guo-shaoge)
    - `SELECT CAST(n AS CHAR)` ステートメントのクエリ結果が、ステートメント内の `n` が負の数の場合に正しくない問題を修正 [#44786](https://github.com/pingcap/tidb/issues/44786) @[xhebox](https://github.com/xhebox)
    - 特定のケースでの TiDB のクエリパニックの問題を修正 [#40857](https://github.com/pingcap/tidb/issues/40857) @[Dousir9](https://github.com/Dousir9)
    - SQL コンパイルエラーログが非リダクトされる問題を修正 [#41831](https://github.com/pingcap/tidb/issues/41831) @[lance6716](https://github.com/lance6716)
    - テーブルのパーティション定義で `FLOOR()` 関数を使用してパーティション化された列を丸めると、`SELECT` ステートメントがエラーを返す問題を修正 [#42323](https://github.com/pingcap/tidb/issues/42323) @[jiyfhust](https://github.com/jiyfhust)
    - パーティション化されたテーブルのクエリがリージョンの分割中にエラーを引き起こす可能性のある問題を修正 [#43144](https://github.com/pingcap/tidb/issues/43144) @[lcwangchao](https://github.com/lcwangchao)
    - 統計情報の読み取り時の不要なメモリ使用量の問題を修正 [#42052](https://github.com/pingcap/tidb/issues/42052) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
    - 大量の空のパーティション化されたテーブルを作成した後の過剰なメモリ使用量の問題を修正 [#44308](https://github.com/pingcap/tidb/issues/44308) @[hawkingrei](https://github.com/hawkingrei)
    - `tidb_opt_agg_push_down` が有効の場合にクエリが正しくない結果を返す問題を修正 [#44795](https://github.com/pingcap/tidb/issues/44795) @[AilinKid](https://github.com/AilinKid)
    - 共通テーブル式の結合結果が間違っている問題を修正 [#38170](https://github.com/pingcap/tidb/issues/38170) @[wjhuang2016](https://github.com/wjhuang2016)
    - 無駄な悲観的トランザクションの余剰ロックが GC によるロック解決時にデータの正確性に影響を与える可能性のある問題を修正 [#43243](https://github.com/pingcap/tidb/issues/43243) @[MyonKeminta](https://github.com/MyonKeminta)
    - キャッシュテーブルに新しい列が追加された後に、その値が列のデフォルト値ではなく `NULL` になる問題を修正 [#42928](https://github.com/pingcap/tidb/issues/42928) @[lqs](https://github.com/lqs)
    - インデックス結合のプローブフェーズで、パーティション化されたテーブルの対応する行が見つからない場合に TiDB がエラーを返す問題を修正 [#43686](https://github.com/pingcap/tidb/issues/43686) @[AilinKid](https://github.com/AilinKid) @[mjonss](https://github.com/mjonss)
    - データベースを削除すると GC 進行が遅くなる問題を修正 [#33069](https://github.com/pingcap/tidb/issues/33069) @[tiancaiamao](https://github.com/tiancaiamao)
    - `ON UPDATE` ステートメントが主キーを正しく更新しない場合にデータとインデックスが一貫していない問題を修正 [#44565](https://github.com/pingcap/tidb/issues/44565) @[zyguan](https://github.com/zyguan)
    - テーブルの名前変更中に TiCDC が一部の行の変更を失う可能性のある問題を修正 [#43338](https://github.com/pingcap/tidb/issues/43338) @[tangenta](https://github.com/tangenta)
    - パーティション化されたテーブルの削除されたパーティション内の Placement Rules が正しく設定されず、リサイクルされない問題を修正 [#44116](https://github.com/pingcap/tidb/issues/44116) @[lcwangchao](https://github.com/lcwangchao)
    - `tidb_scatter_region` が有効の場合、パーティションが切り詰められた後に Region が自動的に分割されない問題を修正 [#43174](https://github.com/pingcap/tidb/issues/43174) [#43028](https://github.com/pingcap/tidb/issues/43028)
    - 多くのパーティションや TiFlash レプリカを持つパーティション化されたテーブルの `TRUNCATE TABLE` 実行時の書き込み競合による DDL リトライの問題を修正 [#42940](https://github.com/pingcap/tidb/issues/42940) @[mjonss](https://github.com/mjonss)
    - ウィンドウ関数を TiFlash にプッシュダウンする際の誤った実行計画の問題を修正 [#43922](https://github.com/pingcap/tidb/issues/43922) @[gengliqi](https://github.com/gengliqi)
    - 非相関のサブクエリを含むステートメントで共通テーブル式（CTE）を使用すると、正しくない結果が返される可能性のある問題を修正 [#44051](https://github.com/pingcap/tidb/issues/44051) @[winoros](https://github.com/winoros)
    - `memTracker` をカーソルフェッチと共に使用するとメモリリークが発生する問題を修正 [#44254](https://github.com/pingcap/tidb/issues/44254) @[YangKeao](https://github.com/YangKeao)
    - `INFORMATION_SCHEMA.DDL_JOBS` テーブルの `QUERY` 列のデータ長が列定義を超える問題を修正 [#42440](https://github.com/pingcap/tidb/issues/42440) @[tiancaiamao](https://github.com/tiancaiamao)
    - `min, max` クエリ結果が正しくない問題を修正 [#43805](https://github.com/pingcap/tidb/issues/43805) @[wshwsh12](https://github.com/wshwsh12)
    - テーブルを分析する際に TiDB が構文エラーを報告する問題を修正 [#43392](https://github.com/pingcap/tidb/issues/43392) @[guo-shaoge](https://github.com/guo-shaoge)
- `SHOW PROCESSLIST`ステートメントが、長いサブクエリ実行時間のトランザクションのTxnStartを表示できない問題を修正[#40851](https://github.com/pingcap/tidb/issues/40851)@[crazycs520](https://github.com/crazycs520)の問題を修正
- `ADMIN SHOW DDL JOBS`の結果に表名が欠落する問題を修正`DROP TABLE`操作が実行中の場合[#42268](https://github.com/pingcap/tidb/issues/42268)@[tiancaiamao](https://github.com/tiancaiamao)の問題を修正
- IPv6環境でTiDBアドレスが正しく表示されない問題を修正[#43260](https://github.com/pingcap/tidb/issues/43260)@[nexustar](https://github.com/nexustar)の問題を修正
- `AES_DECRYPT`式を使用するとSQLステートメントが`runtime error: index out of range`エラーを報告する問題を修正[#43063](https://github.com/pingcap/tidb/issues/43063)@[lcwangchao](https://github.com/lcwangchao)の問題を修正
- `SUBPARTITION`を使用してパーティション化されたテーブルを作成する際に警告が表示されない問題を修正[#41198](https://github.com/pingcap/tidb/issues/41198) [#41200](https://github.com/pingcap/tidb/issues/41200)@[mjonss](https://github.com/mjonss)の問題を修正
- CTEを使用するクエリによってTiDBがハングする問題を修正[#43749](https://github.com/pingcap/tidb/issues/43749) [#36896](https://github.com/pingcap/tidb/issues/36896)@[guo-shaoge](https://github.com/guo-shaoge)の問題を修正
- パーティション化されたテーブルのパーティションの切り捨てが、パーティションの配置ルールを無効にする原因となる可能性がある問題を修正[#44031](https://github.com/pingcap/tidb/issues/44031)@[lcwangchao](https://github.com/lcwangchao)の問題を修正
- CTEの結果が述部をプッシュダウンする際に正しくない問題を修正[#43645](https://github.com/pingcap/tidb/issues/43645)@[winoros](https://github.com/winoros)の問題を修正
- `auto-commit`の変更がトランザクションのコミット動作に影響を与える問題を修正[#36581](https://github.com/pingcap/tidb/issues/36581)@[cfzjywxk](https://github.com/cfzjywxk)
+ TiKV
    - TiDB LightningがSSTファイルの漏洩を引き起こす可能性がある問題を修正[#14745](https://github.com/tikv/tikv/issues/14745)@[YuJuncen](https://github.com/YuJuncen)の問題を修正
    - 暗号化キーIDの競合が古いキーの削除を引き起こす可能性がある問題を修正[#14585](https://github.com/tikv/tikv/issues/14585)@[tabokie](https://github.com/tabokie)の問題を修正
    - Continuous Profilingでファイルハンドルが漏洩する問題を修正[#14224](https://github.com/tikv/tikv/issues/14224)@[tabokie](https://github.com/tabokie)
+ PD
    - gRPCが予期しないフォーマットのエラーを返す問題を修正[#5161](https://github.com/tikv/pd/issues/5161)@[HuSharp](https://github.com/HuSharp)
+ Tools
    + Backup & Restore (BR)
        - `resolved lock timeout`が一部のケースで誤って報告される問題を修正[#43236](https://github.com/pingcap/tidb/issues/43236)@[YuJuncen](https://github.com/YuJuncen)の問題を修正
        - TiKVノードがクラッシュした際のバックアップの遅延問題を修正[#42973](https://github.com/pingcap/tidb/issues/42973)@[YuJuncen](https://github.com/YuJuncen)
    + TiCDC
        - TiCDCがダウンストリームのKafka-on-Pulsarでchangefeedを作成できない問題を修正[#8892](https://github.com/pingcap/tiflow/issues/8892)@[hi-rustin](https://github.com/hi-rustin)
        - PDアドレスまたはリーダーの障害が発生した際にTiCDCが自動的に回復できない問題を修正[#8812](https://github.com/pingcap/tiflow/issues/8812) [#8877](https://github.com/pingcap/tiflow/issues/8877)@[asddongmen](https://github.com/asddongmen)
        - ダウンストリームがKafkaの場合、TiCDCが過度なワークロードを引き起こす問題を修正[#8957](https://github.com/pingcap/tiflow/issues/8957) [#8959](https://github.com/pingcap/tiflow/issues/8959)@[hi-rustin](https://github.com/hi-rustin)
        - PDの障害が発生した際にTiCDCがスタックする問題を修正[#8808](https://github.com/pingcap/tiflow/issues/8808) [#8812](https://github.com/pingcap/tiflow/issues/8812) [#8877](https://github.com/pingcap/tiflow/issues/8877)@[asddongmen](https://github.com/asddongmen)
    + TiDB Lightning
        - ロジカルインポートモードで、インポート中にダウンストリームのテーブルを削除すると、TiDB Lightningメタデータが時間通りに更新されない問題を修正[#44614](https://github.com/pingcap/tidb/issues/44614)@[dsdashun](https://github.com/dsdashun)
        - 競合条件によりディスククォータが正確でない可能性がある問題を修正[#44867](https://github.com/pingcap/tidb/issues/44867)@[D3Hunter](https://github.com/D3Hunter)
        - 大量のデータをインポートする際に、`write to tikv with no leader returned`の問題を修正[#43055](https://github.com/pingcap/tidb/issues/43055)@[lance6716](https://github.com/lance6716)
        - データファイルに閉じていないデリミタがある場合に、可能性のあるOOMの問題を修正[#40400](https://github.com/pingcap/tidb/issues/40400)@[buchuitoudegou](https://github.com/buchuitoudegou)
        - ワイドテーブルをインポートする際にOOMが発生する可能性がある問題を修正[#43728](https://github.com/pingcap/tidb/issues/43728)@[D3Hunter](https://github.com/D3Hunter)
    + TiDB Binlog
        - etcdクライアントが初期化中に最新のノード情報を自動的に同期しない問題を修正[#1236](https://github.com/pingcap/tidb-binlog/issues/1236)@[lichunzhu](https://github.com/lichunzhu)
        - 古いTiKVクライアントバージョンによるDrainerのパニック問題を、TiKVクライアントのアップグレードによって修正[#1170](https://github.com/pingcap/tidb-binlog/issues/1170)@[lichunzhu](https://github.com/lichunzhu)
        - フィルタされていない失敗したDDLステートメントがタスクエラーを引き起こす問題を修正[#1228](https://github.com/pingcap/tidb-binlog/issues/1228)@[lichunzhu](https://github.com/lichunzhu)