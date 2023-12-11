---
title: TiDB 6.5.4 リリースノート
summary: TiDB 6.5.4 における互換性の変更、改善、およびバグ修正についての情報をご覧ください。
---

# TiDB 6.5.4 リリースノート

リリース日: 2023年8月28日

TiDB バージョン: 6.5.4

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.5/quick-start-with-tidb) | [プロダクションデプロイ](https://docs.pingcap.com/tidb/v6.5/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.5.4#version-list)

## 互換性の変更

- `Cursor Fetch` を使用して大量の結果セットを取得する際に TiDB が過剰なメモリを消費する問題を修正し、結果セットをディスクに自動的に書き込んでメモリを解放するようになりました [#43233](https://github.com/pingcap/tidb/issues/43233) @[YangKeao](https://github.com/YangKeao)
- デフォルトで RocksDB の定期的なコンパクションを無効にし、アップグレード後に大量のコンパクションによる潜在的なパフォーマンスへの影響を防ぐため、TiKV RocksDB のデフォルトの動作が v6.5.0 以前のバージョンと一貫するようになりました。また、TiKV では新しい設定項目 [`rocksdb.[defaultcf|writecf|lockcf].periodic-compaction-seconds`](https://docs.pingcap.com/tidb/v6.5/tikv-configuration-file#periodic-compaction-seconds-new-in-v654) と [`rocksdb.[defaultcf|writecf|lockcf].ttl`](https://docs.pingcap.com/tidb/v6.5/tikv-configuration-file#ttl-new-in-v654) を導入し、RocksDB の定期的なコンパクションを手動で構成できるようになりました [#15355](https://github.com/tikv/tikv/issues/15355) @[LykxSassinator](https://github.com/LykxSassinator)

## 改善点

+ TiDB

    - 代入式を含む `LOAD DATA` ステートメントのパフォーマンスを最適化しました [#46081](https://github.com/pinggap/tidb/issues/46081) @[gengliqi](https://github.com/gengliqi)
    - ディスクからダンプされたチャンクの読み取りパフォーマンスを最適化しました [#45125](https://github.com/pingcap/tidb/issues/45125) @[YangKeao](https://github.com/YangKeao)
    - PD スケジューリングを一時停止する `halt-scheduling` 設定項目を追加しました [#6493](https://github.com/tikv/pd/issues/6493) @[JmPotato](https://github.com/JmPotato)

+ TiKV

    - `check_leader` リクエストに対して gzip 圧縮を使用してトラフィックを削減しました [#14553](https://github.com/tikv/tikv/issues/14553) @[you06](https://github.com/you06)
    - `tikv-ctl get-region-read-progress` コマンドを導入し、`Max gap of safe-ts` および `Min safe ts region` メトリクスを追加して解決済み TS および safe-ts の状態をよりよく観察および診断できるようにしました [#15082](https://github.com/tikv/tikv/issues/15082) @[ekexium](https://github.com/ekexium)
    - TTL や定期的なコンパクションなどの機能を無効にできるように、TiKV でいくつかの RocksDB 設定を公開しました [#14873](https://github.com/tikv/tikv/issues/14873) @[LykxSassinator](https://github.com/LykxSassinator)
    - 他のスレッドに影響を及ぼさないように、Titan マニフェストファイルの書き込み時にミューテックスを保持しないよう修正しました [#15351](https://github.com/tikv/tikv/issues/15351) @[Connor1996](https://github.com/Connor1996)

+ PD 

    - Swagger サーバーが無効になっている場合にデフォルトで Swagger API をブロックするサポートを追加しました [#6786](https://github.com/tikv/pd/issues/6786) @[bufferflies](https://github.com/bufferflies)
    - etcd の高可用性を向上させました [#6554](https://github.com/tikv/pd/issues/6554) [#6442](https://github.com/tikv/pd/issues/6442) @[lhy1024](https://github.com/lhy1024)
    - `GetRegions` リクエストのメモリ消費を削減しました [#6835](https://github.com/tikv/pd/issues/6835) @[lhy1024](https://github.com/lhy1024)
    - HTTP 接続の再利用をサポートするようにしました [#6913](https://github.com/tikv/pd/issues/6913) @[nolouch](https://github.com/nolouch)

+ TiFlash 

    - IO バッチの最適化により TiFlash の書き込みパフォーマンスを向上させました [#7735](https://github.com/pingcap/tiflash/issues/7735) @[lidezhu](https://github.com/lidezhu)
    - 不要な fsync 操作を削除することで TiFlash の書き込みパフォーマンスを向上させました [#7736](https://github.com/pingcap/tiflash/issues/7736) @[lidezhu](https://github.com/lidezhu)
    - TiFlash のサービス可用性に影響を与える待機タスクの過剰なキューイングを避けるため、TiFlash コプロセッサタスクキューの最大長を制限しました [#7747](https://github.com/pingcap/tiflash/issues/7747) @[LittleFall](https://github.com/LittleFall)

+ ツール

    + Backup & Restore (BR)

        - HTTP クライアントの `MaxIdleConns` および `MaxIdleConnsPerHost` パラメータを設定することで接続の再利用を強化しました [#46011](https://github.com/pingcap/tidb/issues/46011) @[Leavrth](https://github.com/Leavrth)
        - PD または外部 S3 ストレージに接続できない場合の BR の耐障害性を向上させました [#42909](https://github.com/pingcap/tidb/issues/42909) @[Leavrth](https://github.com/Leavrth)
        - 新しい復元パラメータ `WaitTiflashReady` を追加し、このパラメータを有効にすると TiFlash レプリカが正常にレプリケートされた後に復元操作が完了するようになります [#43828](https://github.com/pingcap/tidb/issues/43828) [#46302](https://github.com/pingcap/tidb/issues/46302) @[3pointer](https://github.com/3pointer)

    + TiCDC

        - 失敗後に TiCDC がリトライする際のステータスメッセージを改良しました [#9483](https://github.com/pingcap/tiflow/issues/9483) @[asddongmen](https://github.com/asddongmen)
        - Kafka に同期する際にリミットを超えるメッセージを扱う TiCDC の方法を最適化し、下流にプライマリキーのみを送信するようにしました [#9574](https://github.com/pingcap/tiflow/issues/9574) @[3AceShowHand](https://github.com/3AceShowHand)
        - HEX 形式のデータに対応するために、Storage Sink が 16 進数エンコーディングをサポートし、AWS DMS 形式仕様と互換性があるようになりました [#9373](https://github.com/pingcap/tiflow/issues/9373) @[CharlesCheung96](https://github.com/CharlesCheung96)

    + TiDB データ移行 (DM) 
    
        - 互換性のない DDL ステートメントのための厳格な楽観的モードをサポートしました [#9112](https://github.com/pingcap/tiflow/issues/9112) @[GMHDBJD](https://github.com/GMHDBJD)

    + Dumpling 

        - `-sql` パラメータを使用してデータをエクスポートする際にすべてのデータベースとテーブルのクエリをスキップすることで、エクスポートのオーバーヘッドを削減しました [#45239](https://github.com/pingcap/tidb/issues/45239) @[lance6716](https://github.com/lance6716)

## バグ修正

+ TiDB 

    - `STREAM_AGG()` 演算子をプッシュダウンする際に `index out of range` エラーが報告される可能性がある問題を修正しました [#40857](https://github.com/pingcap/tidb/issues/40857) @[Dousir9](https://github.com/Dousir9)
    - `CREATE TABLE` ステートメントにサブパーティション定義が含まれている場合に、TiDB がすべてのパーティション情報を無視してパーティション化されていないテーブルを作成してしまう問題を修正しました [#41198](https://github.com/pingcap/tidb/issues/41198) [#41200](https://github.com/pingcap/tidb/issues/41200) @[mjonss](https://github.com/mjonss)
    - 不正な `stale_read_ts` 設定により `PREPARE stmt` がデータを誤って読み取る可能性がある問題を修正しました [#43044](https://github.com/pingcap/tidb/issues/43044) @[you06](https://github.com/you06) 
    - `ActivateTxn` でのデータ競合の可能性がある問題を修正しました [#42092](https://github.com/pingcap/tidb/issues/42092) @[hawkingrei](https://github.com/hawkingrei)
    - バッチクライアントが適切に再接続しない可能性がある問題を修正しました [#44431](https://github.com/pingcap/tidb/issues/44431) @[crazycs520](https://github.com/crazycs520)
- SQLのコンパイルエラーログが適切に隠蔽されない問題を修正する [#41831](https://github.com/pingcap/tidb/issues/41831) @[lance6716](https://github.com/lance6716)
- CTEと相関副問い合わせを同時に使用すると、クエリ結果が誤ってなったりパニックが発生する可能性がある問題を修正する [#44649](https://github.com/pingcap/tidb/issues/44649) [#38170](https://github.com/pingcap/tidb/issues/38170) [#44774](https://github.com/pingcap/tidb/issues/44774) @[winoros](https://github.com/winoros) @[guo-shaoge](https://github.com/guo-shaoge)
- TTLタスクが統計情報の更新を適切なタイミングでトリガーできない問題を修正する [#40109](https://github.com/pingcap/tidb/issues/40109) @[YangKeao](https://github.com/YangKeao)
- GC Resolve Locksステップが悲観的なロックをいくつか見落とす可能性がある問題を修正する [#45134](https://github.com/pingcap/tidb/issues/45134) @[MyonKeminta](https://github.com/MyonKeminta)
- メモリトラッカーにおける潜在的なメモリリークの問題を修正する [#44612](https://github.com/pingcap/tidb/issues/44612) @[wshwsh12](https://github.com/wshwsh12)
- `INFORMATION_SCHEMA.DDL_JOBS`テーブルの「QUERY」列のデータ長が列の定義を超える可能性がある問題を修正する [#42440](https://github.com/pingcap/tidb/issues/42440) @[tiancaiamao](https://github.com/tiancaiamao)
- 大量のリージョンがあるが、一部の仮想テーブルをクエリする際に表IDがプッシュされない場合にPD OOMの問題を修正する `Prepare`または`Execute`を使用する[#39605](https://github.com/pingcap/tidb/issues/39605) @[djshow832](https://github.com/djshow832)
- パーティションテーブルに新しいインデックスを追加した後に、統計自動収集が正しくトリガーされない問題を修正する[#41638](https://github.com/pingcap/tidb/issues/41638) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
- 極端なケースで統計情報におけるSQL実行の詳細の過剰なメモリ消費がTiDB OOMを引き起こす問題を修正する [#44047](https://github.com/pingcap/tidb/issues/44047) @[wshwsh12](https://github.com/wshwsh12)
- バッチコプロセッサのリトライによって誤ったリージョン情報が生成され、クエリの失敗を引き起こす可能性がある問題を修正する [#44622](https://github.com/pingcap/tidb/issues/44622) @[windtalker](https://github.com/windtalker)
- `indexMerge`を使用するクエリが中断されたときに発生するハングアップの問題を修正する [#45279](https://github.com/pingcap/tidb/issues/45279) @[xzhangxian1008](https://github.com/xzhangxian1008)
- システムテーブル`INFORMATION_SCHEMA.TIKV_REGION_STATUS`をクエリすると、一部の場合に正しくない結果が返される問題を修正する[#45531](https://github.com/pingcap/tidb/issues/45531)@[Defined2014](https://github.com/Defined2014)
- `tidb_enable_parallel_apply`が有効になっている場合、`MPP`モードでクエリ結果が正しくない問題を修正する[#45299](https://github.com/pingcap/tidb/issues/45299)@[windtalker](https://github.com/windtalker)
- `tidb_opt_agg_push_down`が有効になっている場合、クエリが正しくない結果を返す可能性がある問題を修正する[#44795](https://github.com/pingcap/tidb/issues/44795)@[AilinKid](https://github.com/AilinKid)
- 仮想列によって適切な物理プランが見つからない問題を修正する[#41014](https://github.com/pingcap/tidb/issues/41014)@[AilinKid](https://github.com/AilinKid)
- 空の`processInfo`によってパニックが発生する問題を修正する[#43829](https://github.com/pingcap/tidb/issues/43829)@[zimulala](https://github.com/zimulala)
- `SELECT CAST(n AS CHAR)`ステートメントの`n`が負の数の場合にクエリ結果が不正確になる問題を修正する[#44786](https://github.com/pingcap/tidb/issues/44786)@[xhebox](https://github.com/xhebox)
- MySQLカーソルフェッチプロトコルを使用した場合に、結果セットのメモリ消費が`tidb_mem_quota_query`制限を超えてしまい、TiDBがOOMを起こす問題を修正する。修正後は、TiDBはメモリを解放するために自動的に結果セットをディスクに書き込む[#43233](https://github.com/pingcap/tidb/issues/43233)@[YangKeao](https://github.com/YangKeao)
- `AUTO_ID_CACHE=1`を使用してテーブルをリストアする際に発生する`duplicate entry`エラーを修正する[#44716](https://github.com/pingcap/tidb/issues/44716)@[tiancaiamao](https://github.com/tiancaiamao)
- テーブルのパーティション定義が`FLOOR()`関数を使用してパーティション列を丸める場合、`SELECT`ステートメントがエラーを返す問題を修正する[#42323](https://github.com/pingcap/tidb/issues/42323)@[jiyfhust](https://github.com/jiyfhust)
- 並行ビューがDDL操作をブロックする可能性がある問題を修正する[#40352](https://github.com/pingcap/tidb/issues/40352)@[zeminzhou](https://github.com/zeminzhou)
- 統計情報収集タスクが不適切な`datetime`値によって失敗する問題を修正する[#39336](https://github.com/pingcap/tidb/issues/39336)@[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
- クラスターのPDノードが置換された後、一部のDDLステートメントが一時的にスタックする問題を修正する[#33908](https://github.com/pingcap/tidb/issues/33908)
- 一度にPDの時間が急変したときに`resolve lock`がハングアップする可能性がある問題を修正する[#44822](https://github.com/pingcap/tidb/issues/44822)@[zyguan](https://github.com/zyguan)
- インデックススキャンで潜在的なデータ競合問題を修正する[#45126](https://github.com/pingcap/tidb/issues/45126)@[wshwsh12](https://github.com/wshwsh12)
- `FormatSQL()`メソッドが入力の非常に長いSQLステートメントを適切に切り詰められない問題を修正する[#44542](https://github.com/pingcap/tidb/issues/44542)@[hawkingrei](https://github.com/hawkingrei)
- 権限がなくてもユーザーが`INFORMATION_SCHEMA.TIFLASH_REPLICA`テーブルの情報を表示できる問題を修正する[#45320](https://github.com/pingcap/tidb/issues/45320)@[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)
- `DATETIME`または`TIMESTAMP`列を数値定数と比較する際の振る舞いがMySQLと一貫していない問題を修正する[#38361](https://github.com/pingcap/tidb/issues/38361)@[yibin87](https://github.com/yibin87)
- インデックスジョインでのエラーがクエリのスタックを引き起こす可能性がある問題を修正する[#45716](https://github.com/pingcap/tidb/issues/45716)@[wshwsh12](https://github.com/wshwsh12)
- 接続を終了させるとゴルーチンがリークする可能性がある問題を修正する[#46034](https://github.com/pingcap/tidb/issues/46034)@[pingyu](https://github.com/pingyu)
- `tmp-storage-quota`構成が効果を発揮しない問題を修正する[#45161](https://github.com/pingcap/tidb/issues/45161) [#26806](https://github.com/pingcap/tidb/issues/26806)@[wshwsh12](https://github.com/wshwsh12)
- クラスター内のTiFlashノードがダウンしたときにTiFlashレプリカが利用できなくなる問題を修正する[#38484](https://github.com/pingcap/tidb/issues/38484)@[hehechen](https://github.com/hehechen)
- `Config.Lables`を同時に読み書きする際のデータ競合の可能性があるため、TiDBがクラッシュする問題を修正する[#45561](https://github.com/pingcap/tidb/issues/45561)@[genliqi](https://github.com/gengliqi)
- クラスターが大きい場合に、`client-go`が定期的に`min-resolved-ts`を更新することがPD OOMを引き起こす可能性がある問題を修正する[#46664](https://github.com/pingcap/tidb/issues/46664)@[HuSharp](https://github.com/HuSharp)

+ TiKV
- `ttl-check-poll-interval`の設定項目がRawKV API V2で効果がない問題を修正 [#15142](https://github.com/tikv/tikv/issues/15142) @[pingyu](https://github.com/pingyu)
    - オンライン不安全リカバリがタイムアウトした場合に中止しない問題を修正 [#15346](https://github.com/tikv/tikv/issues/15346) @[Connor1996](https://github.com/Connor1996)
    - `FLASHBACK`を実行した後にRegion Mergeがブロックされる可能性がある問題を修正 [#15258](https://github.com/tikv/tikv/issues/15258) @[overvenus](https://github.com/overvenus)
    - TiKVノードが隔離され、別のノードが再起動された場合に発生するかもしれないデータの不一致問題を修正 [#15035](https://github.com/tikv/tikv/issues/15035) @[overvenus](https://github.com/overvenus)
    - Data Replication Auto Synchronousモードで、sync-recoverフェーズでQPSがゼロに下がる問題を修正 [#14975](https://github.com/tikv/tikv/issues/14975) @[nolouch](https://github.com/nolouch)
    - 部分書き込み時に暗号化がデータの破損を引き起こす可能性がある問題を修正 [#15080](https://github.com/tikv/tikv/issues/15080) @[tabokie](https://github.com/tabokie)
    - ストアハートビートの再試行回数を減らすことでハートビートストームの問題を修正 [#15184](https://github.com/tikv/tikv/issues/15184) @[nolouch](https://github.com/nolouch)
    - 保留中のコンパクションバイトが多い場合にトラフィックコントロールが機能しない可能性がある問題を修正 [#14392](https://github.com/tikv/tikv/issues/14392) @[Connor1996](https://github.com/Connor1996)
    - PDとTiKVの間のネットワーク障害がPITRが停止する原因になる可能性がある問題を修正 [#15279](https://github.com/tikv/tikv/issues/15279) @[YuJuncen](https://github.com/YuJuncen)
    - TiCDCのOld Value機能が有効になっている場合にTiKVがより多くのメモリを消費する可能性がある問題を修正 [#14815](https://github.com/tikv/tikv/issues/14815) @[YuJuncen](https://github.com/YuJuncen)

+ PD

    - etcdが既に起動しているがクライアントがまだ接続していない場合、クライアントを呼び出すことがPANICを引き起こす可能性がある問題を修正 [#6860](https://github.com/tikv/pd/issues/6860) @[HuSharp](https://github.com/HuSharp)
    - リーダーが長時間終了しない問題を修正 [#6918](https://github.com/tikv/pd/issues/6918) @[bufferflies](https://github.com/bufferflies)
    - プレースメントルールが `LOCATION_LABLES` を使用すると、SQLとRule Checkerが互換性がない問題を修正 [#38605](https://github.com/pingcap/tidb/issues/38605) @[nolouch](https://github.com/nolouch)
    - PDが予期しなく複数のLearnerをRegionに追加する可能性がある問題を修正 [#5786](https://github.com/tikv/pd/issues/5786) @[HunDunDM](https://github.com/HunDunDM)
    - ルールチェッカーがピアを選択する際に不健康なピアを削除できない問題を修正 [#6559](https://github.com/tikv/pd/issues/6559) @[nolouch](https://github.com/nolouch)
    - `unsafe recovery`で失敗したLearnerピアが`auto-detect`モードで無視される問題を修正 [#6690](https://github.com/tikv/pd/issues/6690) @[v01dstar](https://github.com/v01dstar)

+ TiFlash 

    - `fsp`が`DATETIME`、`TIMESTAMP`、または`TIME`のデータ型の変更後にクエリが失敗する問題を修正 [#7809](https://github.com/pingcap/tiflash/issues/7809) @[JaySon-Huang](https://github.com/JaySon-Huang)
    - リージョンの無効な範囲キーによりTiFlashデータが不一致になる問題を修正 [#7762](https://github.com/pingcap/tiflash/issues/7762) @[lidezhu](https://github.com/lidezhu)
    - 同じMPPタスク内に複数のHashAgg演算子がある場合、MPPタスクのコンパイルに過度に長い時間がかかり、クエリのパフォーマンスに影響を与える問題を修正 [#7810](https://github.com/pingcap/tiflash/issues/7810) @[SeaRise](https://github.com/SeaRise)
    - オンライン不安全なリカバリ後にTiFlashが再起動するのに時間がかかりすぎる問題を修正 [#7671](https://github.com/pingcap/tiflash/issues/7671) @[hongyunyan](https://github.com/hongyunyan)
    - `DECIMAL`の結果を不正確に丸めるTiFlashの問題を修正 [#6462](https://github.com/pingcap/tiflash/issues/6462) @[LittleFall](https://github.com/LittleFall)

+ Tools

    + Backup & Restore（BR）

        - BRが使用するグローバルパラメーター`TableColumnCountLimit`と`IndexLimit`のデフォルト値を最大値に増やすことで、リストアの失敗を修正 [#45793](https://github.com/pingcap/tidb/issues/45793) @[Leavrth](https://github.com/Leavrth)
        - PITRでDDLメタ情報の処理中にリライトの失敗を修正 [#43184](https://github.com/pingcap/tidb/issues/43184) @[Leavrth](https://github.com/Leavrth)
        - PITR実行中に関数の戻り値をチェックしないことによるPANICの問題を修正 [#45853](https://github.com/pingcap/tidb/issues/45853) @[Leavrth](https://github.com/Leavrth)
        - Amazon S3以外の互換性のあるS3ストレージを使用する際に無効なリージョンIDを取得する問題を修正 [#41916](https://github.com/pingcap/tidb/issues/41916) [#42033](https://github.com/pingcap/tidb/issues/42033) @[3pointer](https://github.com/3pointer)
        - RawKVモードの細かく分割されたバックアップ段階での潜在的なエラーの修正 [#37085](https://github.com/pingcap/tidb/issues/37085) @[pingyu](https://github.com/pingyu)
        - TiDBクラスターにPITRバックアップタスクがない場合に`resolve lock`の頻度が高すぎる問題を和らげる [#40759](https://github.com/pingcap/tidb/issues/40759) @[joccau](https://github.com/joccau)
        - リージョンのリーダーシップの移行時にPITRログバックアップの遅延が増加する問題を緩和する [#13638](https://github.com/tikv/tikv/issues/13638) @[YuJuncen](https://github.com/YuJuncen)

    + TiCDC

        - 下流でエラーが発生しリトライするとレプリケーションタスクがスタックする可能性がある問題を修正 [#9450](https://github.com/pingcap/tiflow/issues/9450) @[hicqu](https://github.com/hicqu)
        - Kafkaへの同期時にリトライ間隔が短すぎてレプリケーションタスクが失敗する問題を修正 [#9504](https://github.com/pingcap/tiflow/issues/9504) @[3AceShowHand](https://github.com/3AceShowHand)
        - 上流で1つのトランザクションで複数のユニークキー行を変更する際にTiCDCが同期書き込みの競合を引き起こす可能性がある問題を修正 [#9430](https://github.com/pingcap/tiflow/issues/9430) @[sdojjy](https://github.com/sdojjy)
        - TiCDCが間違ってリネームDDL操作を同期する可能性がある問題を修正 [#9488](https://github.com/pingcap/tiflow/issues/9488) [#9378](https://github.com/pingcap/tiflow/issues/9378) [#9531](https://github.com/pingcap/tiflow/issues/9531) @[asddongmen](https://github.com/asddongmen)
        - 下流で一時的な障害が発生したときにレプリケーションタスクがスタックする可能性がある問題を修正 [#9542](https://github.com/pingcap/tiflow/issues/9542) [#9272](https://github.com/pingcap/tiflow/issues/9272)[#9582](https://github.com/pingcap/tiflow/issues/9582) [#9592](https://github.com/pingcap/tiflow/issues/9592) @[hicqu](https://github.com/hicqu)
        - TiCDCノードの状態が変化するとパニックが発生する可能性がある問題を修正 [#9354](https://github.com/pingcap/tiflow/issues/9354) @[sdojjy](https://github.com/sdojjy)
        - Kafka Sinkでエラーが発生した場合、changefeedの進行が無期限にブロックされる可能性がある問題を修正 [#9309](https://github.com/pingcap/tiflow/issues/9309) @[hicqu](https://github.com/hicqu)
        - 下流がKafkaの場合、TiCDCは下流のメタデータを頻繁にクエリし、下流で過剰なワークロードを引き起こす問題を修正 [#8957](https://github.com/pingcap/tiflow/issues/8957) [#8959](https://github.com/pingcap/tiflow/issues/8959) @[hi-rustin](https://github.com/hi-rustin)
        - 一部のTiCDCノードがネットワークから切断されるとデータの不整合が発生する可能性がある問題を修正 [#9344](https://github.com/pingcap/tiflow/issues/9344) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - リプレイケーションタスクがリドログが有効になっており、下流で例外が発生した場合にタスクがスタックする問題を修正 [#9172](https://github.com/pingcap/tiflow/issues/9172) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - PDの一時的な利用不可によりchangefeedsが失敗する問題を修正 [#9294](https://github.com/pingcap/tiflow/issues/9294) @[asddongmen](https://github.com/asddongmen)
        - TiDBまたはMySQLにデータをレプリケートする際に、下流の双方向レプリケーション関連変数を頻繁に設定することにより、下流ログが多すぎる問題を修正 [#9180](https://github.com/pingcap/tiflow/issues/9180) @[asddongmen](https://github.com/asddongmen)
        - Avroプロトコルが`Enum`タイプの値を誤って識別する問題を修正 [#9259](https://github.com/pingcap/tiflow/issues/9259) @[3AceShowHand](https://github.com/3AceShowHand)

    + TiDBデータ移行（DM）

        - ユニークキーの列名がnullの場合にパニックが発生する問題を修正 [#9247](https://github.com/pingcap/tiflow/issues/9247) @[lance6716](https://github.com/lance6716)
        - 検証プログラムがエラーを誤って処理し、リトライメカニズムを最適化することで発生するデッドロックの問題を修正 [#9257](https://github.com/pingcap/tiflow/issues/9257) @[D3Hunter](https://github.com/D3Hunter)
        - ソーティングキーを計算する際に照合が考慮されていない問題を修正 [#9489](https://github.com/pingcap/tiflow/issues/9489) @[hihihuhu](https://github.com/hihihuhu)

    + TiDB Lightning

        - エンジンがデータをインポートしている際にディスククォータチェックがブロックされる可能性がある問題を修正 [#44867](https://github.com/pingcap/tidb/issues/44867) @[D3Hunter](https://github.com/D3Hunter)
        - SSLがターゲットクラスタで有効になっている場合にチェックサムが`Region is unavailable`と報告する問題を修正 [#45462](https://github.com/pingcap/tidb/issues/45462) @[D3Hunter](https://github.com/D3Hunter)
        - エンコーディングエラーを正しくログに記録できない問題を修正 [#44321](https://github.com/pingcap/tidb/issues/44321) @[lyzx2001](https://github.com/lyzx2001)
        - CSVデータをインポートする際にルートがパニックする可能性がある問題を修正 [#43284](https://github.com/pingcap/tidb/issues/43284) @[lyzx2001](https://github.com/lyzx2001)
        - 論理インポートモードでテーブルAをインポートする際に、誤ってテーブルBが存在しないと報告される問題を修正 [#44614](https://github.com/pingcap/tidb/issues/44614) @[dsdashun](https://github.com/dsdashun)
        - `NEXT_GLOBAL_ROW_ID`を保存する際にデータ型が誤っている問題を修正 [#45427](https://github.com/pingcap/tidb/issues/45427) @[lyzx2001](https://github.com/lyzx2001)
        - `checksum = "optional"`の場合でもチェックサムがエラーを報告し続ける問題を修正 [#45382](https://github.com/pingcap/tidb/issues/45382) @[lyzx2001](https://github.com/lyzx2001)
        - PDクラスタのアドレスが変更された場合にデータのインポートに失敗する問題を修正 [#43436](https://github.com/pingcap/tidb/issues/43436) @[lichunzhu](https://github.com/lichunzhu)
        - 一部のPDノードが失敗した際にデータのインポートに失敗する問題を修正 [#43400](https://github.com/pingcap/tidb/issues/43400) @[lichunzhu](https://github.com/lichunzhu)
        - 自動増分列を持つテーブルが`AUTO_ID_CACHE=1`を設定した場合、IDアロケータの基本値が正しくない問題を修正 [#46100](https://github.com/pingcap/tidb/issues/46100) @[D3Hunter](https://github.com/D3Hunter)

    + Dumpling

        - Amazon S3にエクスポートする際に未処理のファイルライターのクローズエラーによりエクスポートファイルが失われる問題を修正 [#45353](https://github.com/pingcap/tidb/issues/45353) @[lichunzhu](https://github.com/lichunzhu)

    + TiDB Binlog

        - 起動時にetcdクライアントが最新のノード情報を自動的に同期しない問題を修正 [#1236](https://github.com/pingcap/tidb-binlog/issues/1236) @[lichunzhu](https://github.com/lichunzhu)