---
title: TiDB 7.1.2 リリースノート
summary: TiDB 7.1.2 での互換性の変更、改善、およびバグ修正について学びます。

# TiDB 7.1.2 リリースノート

リリース日: 2023年10月25日

TiDB バージョン: 7.1.2

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v7.1/quick-start-with-tidb) | [本番環境での展開](https://docs.pingcap.com/tidb/v7.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v7.1.2#version-list)

## 互換性の変更

- [`tidb_opt_enable_hash_join`](https://docs.pingcap.com/tidb/v7.1/system-variables#tidb_opt_enable_hash_join-new-in-v712) システム変数を導入し、最適化プログラムがテーブルのためにハッシュ結合を選択するかどうかを制御するようにします。[#46695](https://github.com/pingcap/tidb/issues/46695) @[coderplay](https://github.com/coderplay)
- デフォルトで RocksDB の定期的なコンパクションを無効にし、TiKV RocksDB のデフォルト動作を v6.5.0 以前のバージョンと一貫性のあるものとします。この変更は、アップグレード後に大量のコンパクションによる性能への影響を防ぐものです。さらに、TiKV は2つの新しい構成項目 [`rocksdb.[defaultcf|writecf|lockcf].periodic-compaction-seconds`](https://docs.pingcap.com/tidb/v7.1/tikv-configuration-file#periodic-compaction-seconds-new-in-v712) と [`rocksdb.[defaultcf|writecf|lockcf].ttl`](https://docs.pingcap.com/tidb/v7.1/tikv-configuration-file#ttl-new-in-v712) を導入し、RocksDB の定期的なコンパクションを手動で構成できるようにします。[#15355](https://github.com/tikv/tikv/issues/15355) @[LykxSassinator](https://github.com/LykxSassinator)
- TiCDC は [`sink.csv.binary-encoding-method`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) 構成項目を導入し、CSV プロトコル内のバイナリデータのエンコーディング方法を制御するようにします。デフォルト値は `'base64'` です。[#9373](https://github.com/pingcap/tiflow/issues/9373) @[CharlesCheung96](https://github.com/CharlesCheung96)
- TiCDC は [`large-message-handle-option`](/ticdc/ticdc-sink-to-kafka.md#handle-messages-that-exceed-the-kafka-topic-limit) 構成項目を導入します。これはデフォルトで空であり、つまりメッセージサイズが Kafka トピックの制限を超えた場合に changefeed が失敗することを意味します。この構成が `"handle-key-only"` に設定されている場合、メッセージがサイズ制限を超える場合、ハンドルキーのみが送信されてメッセージサイズを減らします。減らされたメッセージが依然として制限を超える場合、その後 changefeed が失敗します。[#9680](https://github.com/pingcap/tiflow/issues/9680) @[3AceShowHand](https://github.com/3AceShowHand)

## 改善

+ TiDB

    - [`NO_MERGE_JOIN()`](/optimizer-hints.md#no_merge_joint1_name--tl_name-)、[`NO_INDEX_JOIN()`](/optimizer-hints.md#no_index_joint1_name--tl_name-)、[`NO_INDEX_MERGE_JOIN()`](/optimizer-hints.md#no_index_merge_joint1_name--tl_name-)、[`NO_HASH_JOIN()`](/optimizer-hints.md#no_hash_joint1_name--tl_name-)、および [`NO_INDEX_HASH_JOIN()`](/optimizer-hints.md#no_index_hash_joint1_name--tl_name-) を含む新しい最適化ヒントを追加しました。[#45520](https://github.com/pingcap/tidb/issues/45520) @[qw4990](https://github.com/qw4990)
    - コプロセッサに関連するリクエストソース情報を追加しました。[#46514](https://github.com/pingcap/tidb/issues/46514) @[you06](https://github.com/you06)
    - TiDB ノードのアップグレード状態の開始と終了を示すために `/upgrade/start` および `upgrade/finish` API を追加しました。[#47172](https://github.com/pingcap/tidb/issues/47172) @[zimulala](https://github.com/zimulala)

+ TiKV

    - コンパクション機構を最適化しました。リージョンが分割されると、分割するキーがない場合、過剰な MVCC バージョンを排除するコンパクションがトリガーされます。[#15282](https://github.com/tikv/tikv/issues/15282) @[SpadeA-Tang](https://github.com/SpadeA-Tang)
    - ルーターオブジェクト内の LRUCache を削除してメモリ使用量を減らし、OOM を防止します。[#15430](https://github.com/tikv/tikv/issues/15430) @[Connor1996](https://github.com/Connor1996)
    - `Max gap of safe-ts` および `Min safe ts region` メトリクスを追加し、`tikv-ctl get-region-read-progress` コマンドを導入して、resolved-ts および safe-ts の状態をよりよく観察し、診断できるようにしました。[#15082](https://github.com/tikv/tikv/issues/15082) @[ekexium](https://github.com/ekexium)
    - TiKV でいくつかの RocksDB 構成を公開し、TTL や定期的なコンパクションなどの機能を無効にできるようにしました。[#14873](https://github.com/tikv/tikv/issues/14873) @[LykxSassinator](https://github.com/LykxSassinator)
    - PD クライアントの接続リトライプロセスでのバックオフメカニズムを追加し、エラーリトライ中にリトライ間隔を徐々に増加させて PD の負荷を減らすようにしました。[#15428](https://github.com/tikv/tikv/issues/15428) @[nolouch](https://github.com/nolouch)
    - 他のスレッドに影響を与えないように、Titan マニフェストファイルの書き込み時に mutex を保持しないようにしました。[#15351](https://github.com/tikv/tikv/issues/15351) @[Connor1996](https://github.com/Connor1996)
    - OOM を防ぐために Resolver のメモリ使用量を最適化しました。[#15458](https://github.com/tikv/tikv/issues/15458) @[overvenus](https://github.com/overvenus)

+ PD

    - PD 起動時のバックオフメカニズムを最適化し、呼び出しの失敗時に RPC リクエストの頻度を減らします。[#6556](https://github.com/tikv/pd/issues/6556) @[nolouch](https://github.com/nolouch) @[rleungx](https://github.com/rleungx) @[HuSharp](https://github.com/HuSharp)
    - コール元が切断されたときに CPU およびメモリを解放するために `GetRegions` インターフェースのキャンセルメカニズムを導入しました。[#6835](https://github.com/tikv/pd/issues/6835) @[lhy1024](https://github.com/lhy1024)

+ TiFlash

    - Grafana でインデックスデータのメモリ使用量の監視メトリクスを追加しました。[#8050](https://github.com/pingcap/tiflash/issues/8050) @[hongyunyan](https://github.com/hongyunyan)

+ ツール

    + バックアップ・リストア(BR)

        - log バックアップおよび PITR リストアタスクの接続再利用を強化するために HTTP クライアントで `MaxIdleConns` および `MaxIdleConnsPerHost` パラメータを設定することで CPU オーバーヘッドを削減しました。[#46011](https://github.com/pingcap/tidb/issues/46011) @[Leavrth](https://github.com/Leavrth)
        - log バックアップ `resolve lock` の CPU オーバーヘッドを削減しました。[#40759](https://github.com/pingcap/tidb/issues/40759) @[3pointer](https://github.com/3pointer)
        - 新しいリストアパラメータ `WaitTiflashReady` を追加しました。このパラメータを有効にすると、TiFlash レプリカが正常に複製された後にリストア操作が完了します。[#43828](https://github.com/pingcap/tidb/issues/43828) [#46302](https://github.com/pingcap/tidb/issues/46302) @[3pointer](https://github.com/3pointer)

    + TiCDC

        - いくつかの TiCDC モニタリングメトリクスおよびアラームルールを最適化しました。[#9047](https://github.com/pingcap/tiflow/issues/9047) @[asddongmen](https://github.com/asddongmen)
        - 過大なメッセージサイズによる changefeed の失敗を避けるために、Kafka Sink は[ハンドルキーデータのみを送信](/ticdc/ticdc-sink-to-kafka.md#handle-messages-that-exceed-the-kafka-topic-limit)するようサポートします。[#9680](https://github.com/pingcap/tiflow/issues/9680) @[3AceShowHand](https://github.com/3AceShowHand)
        - `ADD INDEX` DDL 操作のレプリケーションロジックを最適化し、後続の DML ステートメントをブロックしないようにしました。[#9644](https://github.com/pingcap/tiflow/issues/9644) @[sdojjy](https://github.com/sdojjy)
- タスクの失敗後に TiCDC がリトライする際のステータスメッセージを改善する [#9483](https://github.com/pingcap/tiflow/issues/9483) @[asddongmen](https://github.com/asddongmen)

+ TiDB データ移行（DM）

    - 互換性のないDDLステートメントに対して厳密な楽観的モードをサポートする [#9112](https://github.com/pingcap/tiflow/issues/9112) @[GMHDBJD](https://github.com/GMHDBJD)

+ TiDB Lightning

    - インポートタスクのパフォーマンス向上のために`checksum-via-sql`のデフォルト値を`false`に変更する [#45368](https://github.com/pingcap/tidb/issues/45368) [#45094](https://github.com/pingcap/tidb/issues/45094) @[GMHDBJD](https://github.com/GMHDBJD)
    - データインポートフェーズでの`no leader`エラーに対する TiDB Lightning のリトライロジックを最適化する [#46253](https://github.com/pingcap/tidb/issues/46253) @[lance6716](https://github.com/lance6716)

## バグの修正

+ TiDB

    - `GROUP_CONCAT`が`ORDER BY`カラムを解析できない問題を修正する[#41986](https://github.com/pingcap/tidb/issues/41986)@[AilinKid](https://github.com/AilinKid)
    - システムテーブル`INFORMATION_SCHEMA.TIKV_REGION_STATUS`を照会すると、一部のケースで誤った結果が返される問題を修正する[#45531](https://github.com/pingcap/tidb/issues/45531)@[Defined2014](https://github.com/Defined2014)
    - TiDBのアップグレードがDDLリースよりも読み取りメタデータに時間がかかると処理が停滞する問題を修正する[#45176](https://github.com/pingcap/tidb/issues/45176)@[zimulala](https://github.com/zimulala)
    - CTEを使用したDMLステートメントの実行がパニックを引き起こす問題を修正する[#46083](https://github.com/pingcap/tidb/issues/46083)@[winoros](https://github.com/winoros)
    - パーティション定義に準拠しないデータを検出できない問題を修正する[#46492](https://github.com/pingcap/tidb/issues/46492)@[mjonss](https://github.com/mjonss)
    - `MERGE_JOIN`の結果が正しくない問題を修正する[#46580](https://github.com/pingcap/tidb/issues/46580)@[qw4990](https://github.com/qw4990)
    - 符号なし型と`Duration`型定数を比較したときに発生する不正な結果を修正する[#45410](https://github.com/pingcap/tidb/issues/45410)@[wshwsh12](https://github.com/wshwsh12)
    - `AUTO_ID_CACHE=1`が設定されていると`Duplicate entry`が発生する問題を修正する[#46444](https://github.com/pingcap/tidb/issues/46444)@[tiancaiamao](https://github.com/tiancaiamao)
    - TTL実行時のメモリリーク問題を修正する[#45510](https://github.com/pingcap/tidb/issues/45510)@[lcwangchao](https://github.com/lcwangchao)
    - 接続の切断がgo coroutineリークを引き起こす問題を修正する[#46034](https://github.com/pingcap/tidb/issues/46034)@[pingyu](https://github.com/pingyu)
    - Index Joinのエラーがクエリのスタックを引き起こす問題を修正する[#45716](https://github.com/pingcap/tidb/issues/45716)@[wshwsh12](https://github.com/wshwsh12)
    - ハッシュ分割テーブルの`BatchPointGet`演算子が誤った結果を返す問題を修正する[#46779](https://github.com/pingcap/tidb/issues/46779)@[jiyfhust](https://github.com/jiyfhust)
    - `EXCHANGE PARTITION`が失敗またはキャンセルされた場合に、パーティションテーブルに関する制限が元のテーブルに残る問題を修正する[#45920](https://github.com/pingcap/tidb/issues/45920) [#45791](https://github.com/pingcap/tidb/issues/45791)@[mjonss](https://github.com/mjonss)
    - `TIDB_INLJ`ヒントが2つのサブクエリを結合する際に効果がない問題を修正する[#46160](https://github.com/pingcap/tidb/issues/46160)@[qw4990](https://github.com/qw4990)
    - `DATETIME`または`TIMESTAMP`カラムを数値定数と比較する際のMySQLとの一貫性のない振る舞いを修正する[#38361](https://github.com/pingcap/tidb/issues/38361)@[yibin87](https://github.com/yibin87)
    - ハッシュコードが繰り返し計算される問題を修正する[#42788](https://github.com/pingcap/tidb/issues/42788)@[AilinKid](https://github.com/AilinKid)
    - `READ_FROM_STORAGE(TIFLASH[...])`ヒントが無視され、`Can't find a proper physical plan`エラーが発生するアクセスパスプルーニングロジックの問題を修正する[#40146](https://github.com/pingcap/tidb/issues/40146)@[AilinKid](https://github.com/AilinKid)
    - `cast(col)=range`条件が間違った場合にフルスキャンが発生する問題を修正する[#45199](https://github.com/pingcap/tidb/issues/45199)@[AilinKid](https://github.com/AilinKid)
    - `tmp-storage-quota`構成が効果を持たない問題を修正する[#45161](https://github.com/pingcap/tidb/issues/45161) [#26806](https://github.com/pingcap/tidb/issues/26806)@[wshwsh12](https://github.com/wshwsh12)
    - TiDBパーサーが状態を保持して解析に失敗する問題を修正する[#45898](https://github.com/pingcap/tidb/issues/45898)@[qw4990](https://github.com/qw4990)
    - MPP実行計画でUnionを介してAggregationが押し下げられた場合の結果が正しくない問題を修正する[#45850](https://github.com/pingcap/tidb/issues/45850)@[AilinKid](https://github.com/AilinKid)
    - `AUTO_ID_CACHE=1`が設定されている場合のTiDBがパニック後に遅い復旧を行う問題を修正する[#46454](https://github.com/pingcap/tidb/issues/46454)@[tiancaiamao](https://github.com/tiancaiamao)
    - スパイル処理中に`Sort`演算子がTiDBをクラッシュさせる可能性がある問題を修正する[#47538](https://github.com/pingcap/tidb/issues/47538)@[windtalker](https://github.com/windtalker)
    - `AUTO_ID_CACHE=1`で非クラスタ化されたインデックステーブルを復元する際の重複したプライマリキーの問題を修正する[#46093](https://github.com/pingcap/tidb/issues/46093)@[tiancaiamao](https://github.com/tiancaiamao)
    - 静的剪定モードでパーティションテーブルを照会し、実行計画が`IndexLookUp`を含むとクエリがエラーを報告する問題を修正する[#45757](https://github.com/pingcap/tidb/issues/45757)@[Defined2014](https://github.com/Defined2014)
    - パーティションテーブルにデータを挿入すると、パーティションテーブルと配置ポリシーを持つテーブルとの間でパーティションを交換後に失敗する問題を修正する[#45791](https://github.com/pingcap/tidb/issues/45791)@[mjonss](https://github.com/mjonss)
    - 時間フィールドを誤ったタイムゾーン情報でエンコードする問題を修正する[#46033](https://github.com/pingcap/tidb/issues/46033)@[tangenta](https://github.com/tangenta)
    - `tmp`ディレクトリが存在しない場合に、DDLステートメントが迅速にインデックスを追加するときに処理が停滞する問題を修正する[#45456](https://github.com/pingcap/tidb/issues/45456)@[tangenta](https://github.com/tangenta)
    - 複数のTiDBインスタンスを同時にアップグレードすると、アップグレードプロセスがブロックされる可能性のある問題を修正する[#46288](https://github.com/pingcap/tidb/issues/46228)@[zimulala](https://github.com/zimulala)
    - 不正なパラメータを使用してRegionを分割することで引き起こされるリージョン分散のバランス問題を修正する[#46135](https://github.com/pingcap/tidb/issues/46135)@[zimulala](https://github.com/zimulala)
- TiDB
    - TiDBの再起動後にDDL操作がスタックする可能性のある問題を修正 [#46751](https://github.com/pingcap/tidb/issues/46751) @[wjhuang2016](https://github.com/wjhuang2016)
    - 非整数のクラスタ化インデックスにおけるテーブル操作の分割を禁止する [#47350](https://github.com/pingcap/tidb/issues/47350) @[tangenta](https://github.com/tangenta)
    - 誤ったMDL処理によりDDL操作が永久的にブロックされる可能性のある問題を修正 [#46920](https://github.com/pingcap/tidb/issues/46920) @[wjhuang2016](https://github.com/wjhuang2016)
    - `RENAME TABLE`操作によってテーブル内で重複する列の問題を修正 [#47064](https://github.com/pingcap/tidb/issues/47064)@[jiyfhust](https://github.com/jiyfhust)
    - `client-go`における`batch-client`のパニックの問題を修正 [#47691](https://github.com/pingcap/tidb/issues/47691) @[crazycs520](https://github.com/crazycs520)
    - パーティションテーブルにおける統計収集がメモリ使用量が制限を超えた時に適切に終了されない問題を修正 [#45706](https://github.com/pingcap/tidb/issues/45706) @[hawkingrei](https://github.com/hawkingrei)
    - `UNHEX`条件を含むクエリが正確でないクエリ結果を返す問題を修正 [#45378](https://github.com/pingcap/tidb/issues/45378) @[qw4990](https://github.com/qw4990)
    - `GROUP_CONCAT`を含むクエリに対してTiDBが`Can't find column`を返す問題を修正 [#41957](https://github.com/pingcap/tidb/issues/41957) @[AilinKid](https://github.com/AilinKid)

+ TiKV
    - `ttl-check-poll-interval`構成項目がRawKV API V2に影響を与えない問題を修正 [#15142](https://github.com/tikv/tikv/issues/15142) @[pingyu](https://github.com/pingyu)
    - 連続して増加するraftstore-applyのデータエラーを修正 [#15371](https://github.com/tikv/tikv/issues/15371) @[Connor1996](https://github.com/Connor1996)
    - データレプリケーション自動同期モード下でsync-recoverフェーズでQPSがゼロに低下する問題を修正 [#14975](https://github.com/tikv/tikv/issues/14975) @[nolouch](https://github.com/nolouch)
    - TiKVノードが孤立し、別のノードが再起動された場合に発生するデータ不整合の問題を修正 [#15035](https://github.com/tikv/tikv/issues/15035) @[overvenus](https://github.com/overvenus)
    - オンラインUnsafeリカバリがマージの中止を処理できない問題を修正 [#15580](https://github.com/tikv/tikv/issues/15580) @[v01dstar](https://github.com/v01dstar)
    - PDとTiKV間のネットワーク障害がPITRをスタックさせる可能性のある問題を修正 [#15279](https://github.com/tikv/tikv/issues/15279) @[YuJuncen](https://github.com/YuJuncen)
    - `FLASHBACK`を実行した後にRegion Mergeがブロックされる可能性のある問題を修正 [#15258](https://github.com/tikv/tikv/issues/15258) @[overvenus](https://github.com/overvenus)
    - ストアハートビートのリトライ回数を減らすことによってハートビートストームの問題を修正 [#15184](https://github.com/tikv/tikv/issues/15184) @[nolouch](https://github.com/nolouch)
    - タイムアウトでオンラインUnsafeリカバリが中止されない問題を修正 [#15346](https://github.com/tikv/tikv/issues/15346) @[Connor1996](https://github.com/Connor1996)
    - 部分書き込み中にデータの暗号化がデータ破損を引き起こす可能性のある問題を修正 [#15080](https://github.com/tikv/tikv/issues/15080) @[tabokie](https://github.com/tabokie)
    - Regionのメタデータが正しくないことによって引き起こされるTiKVのパニックの問題を修正 [#13311](https://github.com/tikv/tikv/issues/13311) @[cfzjywxk](https://github.com/cfzjywxk)
    - オンラインワークロードが存在する状況でTiDB Lightningチェックサムコプロセッサのリクエストがタイムアウトする問題を修正 [#15565](https://github.com/tikv/tikv/issues/15565) @[lance6716](https://github.com/lance6716)
    - ピアの移動がフォロワーリードのパフォーマンスの低下を引き起こす可能性のある問題を修正 [#15468](https://github.com/tikv/tikv/issues/15468) @[YuJuncen](https://github.com/YuJuncen)

+ PD
    - ホットなRegionがv2スケジューラアルゴリズムにおいてスケジュールされない問題を修正 [#6645](https://github.com/tikv/pd/issues/6645) @[lhy1024](https://github.com/lhy1024)
    - 空のクラスタにおいてTLSハンドシェイクが高CPU使用率を引き起こす問題を修正 [#6913](https://github.com/tikv/pd/issues/6913) @[nolouch](https://github.com/nolouch)
    - PDノード間の注入エラーがPDパニックを引き起こす可能性のある問題を修正 [#6858](https://github.com/tikv/pd/issues/6858) @[HuSharp](https://github.com/HuSharp)
    - ストア情報の同期がPDリーダーの退出とスタックを引き起こす可能性のある問題を解消 [#6918](https://github.com/tikv/pd/issues/6918) @[rleungx](https://github.com/rleungx)
    - Flashback後にRegion情報が更新されない問題を修正 [#6912](https://github.com/tikv/pd/issues/6912) @[overvenus](https://github.com/overvenus)
    - PDが終了中にパニックする可能性のある問題を修正 [#7053](https://github.com/tikv/pd/issues/7053) @[HuSharp](https://github.com/HuSharp)
    - コンテキストタイムアウトが`lease timeout`エラーを引き起こす可能性のある問題を修正 [#6926](https://github.com/tikv/pd/issues/6926) @[rleungx](https://github.com/rleungx)
    - リーダーが適切にグループごとに散らばっていないことによってリーダーの分布が偏る可能性のある問題を解消 [#6962](https://github.com/tikv/pd/issues/6962) @[rleungx](https://github.com/rleungx)
    - `pd-ctl`を使用して更新中に孤立レベルラベルが同期されない問題を修正 [#7121](https://github.com/tikv/pd/issues/7121) @[rleungx](https://github.com/rleungx)
    - `evict-leader-scheduler`が構成を失う可能性のある問題を解消 [#6897](https://github.com/tikv/pd/issues/6897) @[HuSharp](https://github.com/HuSharp)
    - プラグインディレクトリとファイルの潜在的なセキュリティリスクを解消 [#7094](https://github.com/tikv/pd/issues/7094) @[HuSharp](https://github.com/HuSharp)
    - リソース制御を有効にした後にDDLが原子性を保証しない可能性のある問題を解消 [#45050](https://github.com/pingcap/tidb/issues/45050) @[glorv](https://github.com/glorv)
    - ルールチェッカーが選択するピアが不健康である場合に削除されない可能性のある問題を解消 [#6559](https://github.com/tikv/pd/issues/6559) @[nolouch](https://github.com/nolouch)
    - etcdが既に起動している状態でクライアントが接続されていない場合にPDをパニックさせる可能性のある問題を解消 [#6860](https://github.com/tikv/pd/issues/6860) @[HuSharp](https://github.com/HuSharp)
    - RU消費が0未満の場合にPDをクラッシュさせる可能性のある問題を解消 [#6973](https://github.com/tikv/pd/issues/6973) @[CabinfeverB](https://github.com/CabinfeverB)
    - クライアントが定期的に`min-resolved-ts`を更新することでPDが大規模なクラスタでOOMを引き起こす可能性のある問題を解消 [#46664](https://github.com/pingcap/tidb/issues/46664) @[HuSharp](https://github.com/HuSharp)

+ TiFlash
    - MemoryTrackerによって報告されるメモリ使用量が正確でない問題を解消 [#8128](https://github.com/pingcap/tiflash/issues/8128) @[JinheLin](https://github.com/JinheLin)
    - リージョンの無効な範囲キーによってTiFlashデータが不整合になる問題を解消 [#7762](https://github.com/pingcap/tiflash/issues/7762) @[lidezhu](https://github.com/lidezhu)
- `DATETIME`, `TIMESTAMP`, あるいは `TIME` データ型の `fsp` が変更された後にクエリが失敗する問題を修正する [#7809](https://github.com/pingcap/tiflash/issues/7809) @[JaySon-Huang](https://github.com/JaySon-Huang)
    - 同一の MPP タスク内に複数の HashAgg オペレータがある場合、MPP タスクのコンパイルに過度の時間がかかり、クエリのパフォーマンスに深刻な影響を与える問題を修正する [#7810](https://github.com/pingcap/tiflash/issues/7810) @[SeaRise](https://github.com/SeaRise)

+ ツール

    + バックアップ＆リストア（BR）

        - PITR を使用して暗黙のプライマリキーを回復すると競合が発生する可能性がある問題を修正する [#46520](https://github.com/pingcap/tidb/issues/46520) @[3pointer](https://github.com/3pointer)
        - PITR が GCS からデータを回復できない問題を修正する [#47022](https://github.com/pingcap/tidb/issues/47022) @[Leavrth](https://github.com/Leavrth)
        - RawKV モードにおけるファイングレインバックアップフェーズでの潜在的なエラーを修正する [#37085](https://github.com/pingcap/tidb/issues/37085) @[pingyu](https://github.com/pingyu)
        - PITR を使用してメタ kv を回復する際にエラーが発生する可能性がある問題を修正する [#46578](https://github.com/pingcap/tidb/issues/46578) @[Leavrth](https://github.com/Leavrth)
        - BR 統合テストケースのエラーを修正する [#45561](https://github.com/pingcap/tidb/issues/46561) @[purelind](https://github.com/purelind)
        - BR が使用するグローバルパラメータ `TableColumnCountLimit` および `IndexLimit` のデフォルト値を最大値に増やすことで、リストアに失敗する問題を修正する [#45793](https://github.com/pingcap/tidb/issues/45793) @[Leavrth](https://github.com/Leavrth)
        - リストアされたデータのスキャン中に br CLI クライアントが立ち往生する問題を修正する [#45476](https://github.com/pingcap/tidb/issues/45476) @[3pointer](https://github.com/3pointer)
        - PITR が `CREATE INDEX` DDL ステートメントの復元をスキップする可能性がある問題を修正する [#47482](https://github.com/pingcap/tidb/issues/47482) @[Leavrth](https://github.com/Leavrth)
        - 1 分以内に複数回 PITR を実行するとデータの損失が発生する可能性がある問題を修正する [#15483](https://github.com/tikv/tikv/issues/15483) @[YuJuncen](https://github.com/YuJuncen)

    + TiCDC

        - 異常な状態の複製タスクが上流の GC をブロックする問題を修正する [#9543](https://github.com/pingcap/tiflow/issues/9543) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - オブジェクトストレージにデータを複製するとデータの不整合が発生する可能性がある問題を修正する [#9592](https://github.com/pingcap/tiflow/issues/9592) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - `redo-resolved-ts` を有効にすると changefeed が失敗する可能性がある問題を修正する [#9769](https://github.com/pingcap/tiflow/issues/9769) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 特定のオペレーティングシステムで間違ったメモリ情報を取得することが OOM の問題を修正する [#9762](https://github.com/pingcap/tiflow/issues/9762) @[sdojjy](https://github.com/sdojjy)
        - `scale-out` が有効になっている場合に、書き込みキーがノード間で均等に分散されない問題を修正する [#9665](https://github.com/pingcap/tiflow/issues/9665) @[sdojjy](https://github.com/sdojjy)
        - ログに機密ユーザ情報が記録される問題を修正する [#9690](https://github.com/pingcap/tiflow/issues/9690) @[sdojjy](https://github.com/sdojjy)
        - TiCDC が誤ってリネーム DDL 操作を同期する可能性がある問題を修正する [#9488](https://github.com/pingcap/tiflow/issues/9488) [#9378](https://github.com/pingcap/tiflow/issues/9378) [#9531](https://github.com/pingcap/tiflow/issues/9531) @[asddongmen](https://github.com/asddongmen)
        - すべての changefeed が削除された後に上流 TiDB GC がブロックされる問題を修正する [#9633](https://github.com/pingcap/tiflow/issues/9633) @[sdojjy](https://github.com/sdojjy)
        - 特定のケースで TiCDC 複製タスクが失敗する可能性がある問題を修正する [#9685](https://github.com/pingcap/tiflow/issues/9685) [#9697](https://github.com/pingcap/tiflow/issues/9697) [#9695](https://github.com/pingcap/tiflow/issues/9695) [#9736](https://github.com/pingcap/tiflow/issues/9736) @[hicqu](https://github.com/hicqu) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - PD ノードのネットワーク分離によって TiCDC の複製遅延が高くなる問題を修正する [#9565](https://github.com/pingcap/tiflow/issues/9565) @[asddongmen](https://github.com/asddongmen)
        - PD のスケーリング中に TiCDC が無効な古いアドレスにアクセスする問題を修正する [#9584](https://github.com/pingcap/tiflow/issues/9584) @[fubinzh](https://github.com/fubinzh) @[asddongmen](https://github.com/asddongmen)
        - TiKV ノードの障害からの迅速な回復ができない状態で TiCDC が多くのリージョンを上流に持っている場合の問題を修正する [#9741](https://github.com/pingcap/tiflow/issues/9741) @[sdojjy](https://github.com/sdojjy)
        - CSV 形式を使用する際に TiCDC が `UPDATE` 操作を誤って `INSERT` に変更する問題を修正する [#9658](https://github.com/pingcap/tiflow/issues/9658) @[3AceShowHand](https://github.com/3AceShowHand)
        - 同一の DDL ステートメントで複数のテーブルがリネームされる場合に複製エラーが発生する問題を修正する [#9476](https://github.com/pingcap/tiflow/issues/9476) [#9488](https://github.com/pingcap/tiflow/issues/9488) @[CharlesCheung96](https://github.com/CharlesCheung96) @[asddongmen](https://github.com/asddongmen)
        - Kafka への同期時に複製書き込み競合が発生する可能性がある問題を修正する [#9504](https://github.com/pingcap/tiflow/issues/9504) @[3AceShowHand](https://github.com/3AceShowHand)
        - 上流のトランザクションで複数行のユニークキーが修正された場合に複製書き込み競合が発生する可能性がある問題を修正する [#9430](https://github.com/pingcap/tiflow/issues/9430) @[sdojjy](https://github.com/sdojjy)
        - 下流で一時的な障害が発生した場合に複製タスクが立ち往生する問題を修正する [#9542](https://github.com/pingcap/tiflow/issues/9542) [#9272](https://github.com/pingcap/tiflow/issues/9272) [#9582](https://github.com/pingcap/tiflow/issues/9582) [#9592](https://github.com/pingcap/tiflow/issues/9592) @[hicqu](https://github.com/hicqu)
        - 下流でエラーが発生しリトライが行われると複製タスクが立ち往生する問題を修正する [#9450](https://github.com/pingcap/tiflow/issues/9450) @[hicqu](https://github.com/hicqu)

    + TiDB データ移行（DM）

        - 変更された DDL がスキップされ、その後の DDL が実行されないと、DM によって返される複製遅延が増大し続ける問題を修正する [#9605](https://github.com/pingcap/tiflow/issues/9605) @[D3Hunter](https://github.com/D3Hunter)
        - 大文字小文字を区別しない照合において DM が競合を正しく処理できない問題を修正する [#9489](https://github.com/pingcap/tiflow/issues/9489) @[hihihuhu](https://github.com/hihihuhu)
        - DM バリデータのデッドロック問題を修正し、リトライを強化する [#9257](https://github.com/pingcap/tiflow/issues/9257) @[D3Hunter](https://github.com/D3Hunter)
        - 楽観的モードでタスクを再開する際にすべての DML をスキップする問題を修正する [#9588](https://github.com/pingcap/tiflow/issues/9588) @[GMHDBJD](https://github.com/GMHDBJD)
        - オンライン DDL をスキップする際に、DM が上流テーブルスキーマを正しくトラックできない問題を修正します [#9587](https://github.com/pingcap/tiflow/issues/9587) @[GMHDBJD](https://github.com/GMHDBJD)
        - 楽観的モードでのパーティション DDL をスキップする問題を修正します [#9788](https://github.com/pingcap/tiflow/issues/9788) @[GMHDBJD](https://github.com/GMHDBJD)

    + TiDB Lightning

        - `AUTO_ID_CACHE=1` を使用してテーブルをインポートする際に、誤った `row_id` が割り当てられる問題を修正します [#46100](https://github.com/pingcap/tidb/issues/46100) @[D3Hunter](https://github.com/D3Hunter)
        - `NEXT_GLOBAL_ROW_ID` を保存する際にデータ型が間違っている問題を修正します [#45427](https://github.com/pingcap/tidb/issues/45427) @[lyzx2001](https://github.com/lyzx2001)
        - `checksum = "optional"` の場合でも、checksum がエラーを報告し続ける問題を修正します [#45382](https://github.com/pingcap/tidb/issues/45382) @[lyzx2001](https://github.com/lyzx2001)
        - PD クラスタのアドレスが変更された場合にデータのインポートが失敗する問題を修正します [#43436](https://github.com/pingcap/tidb/issues/43436) @[lichunzhu](https://github.com/lichunzhu)
        - PD トポロジが変更された場合に TiDB Lightning が起動しない問題を修正します [#46688](https://github.com/pingcap/tidb/issues/46688) @[lance6716](https://github.com/lance6716)
        - CSV データをインポートする際にルートがパニックする可能性がある問題を修正します [#43284](https://github.com/pingcap/tidb/issues/43284) @[lyzx2001](https://github.com/lyzx2001)

    + TiDB Binlog

        - 1 GB よりも大きいトランザクションを輸送する際に Drainer が終了する問題を修正します [#28659](https://github.com/pingcap/tidb/issues/28659) @[jackysp](https://github.com/jackysp)