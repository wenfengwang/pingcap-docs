---
title: TiDB 7.1.1 リリースノート
summary: TiDB 7.1.1 での互換性の変更、改善、およびバグ修正について学びます。

# TiDB 7.1.1 リリースノート

リリース日: 2023年7月24日

TiDB バージョン: 7.1.1

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v7.1/quick-start-with-tidb) | [本番環境デプロイ](https://docs.pingcap.com/tidb/v7.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v7.1.1#version-list)

## 互換性の変更

- TiDB は新しいシステム変数 `tidb_lock_unchanged_keys` を導入し、変更されていないキーをロックするかどうかを制御します [#44714](https://github.com/pingcap/tidb/issues/44714) @[ekexium](https://github.com/ekexium)

## 改善

+ TiDB

    - プランキャッシュは、200以上のパラメータを持つクエリをサポートします [#44823](https://github.com/pingcap/tidb/issues/44823) @[qw4990](https://github.com/qw4990)
    - ディスクからダンプしたチャンクを読み込むパフォーマンスを最適化します [#45125](https://github.com/pingcap/tidb/issues/45125) @[YangKeao](https://github.com/YangKeao)
    - インデックススキャン範囲の構築ロジックを最適化し、複雑な条件をインデックススキャン範囲に変換できるようにサポートします [#41572](https://github.com/pingcap/tidb/issues/41572) [#44389](https://github.com/pingcap/tidb/issues/44389) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
    - ステールリードのリトライリーダーがロックを遭遇した場合、TiDB はロックを解決した後にリーダーで強制的にリトライし、不要なオーバーヘッドを回避します [#43659](https://github.com/pingcap/tidb/issues/43659) @[you06](https://github.com/you06)

+ PD

    - PD は、Swagger サーバーが無効化されている場合、Swagger API をデフォルトでブロックします [#6786](https://github.com/tikv/pd/issues/6786) @[bufferflies](https://github.com/bufferflies)

+ ツール

    + TiCDC

        - TiCDC がデータをオブジェクトストレージサービスにレプリケートする際のバイナリフィールドのエンコーディング形式を最適化します [#9373](https://github.com/pingcap/tiflow/issues/9373) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - Kafka へのレプリケーションシナリオで OAUTHBEARER 認証をサポートします [#8865](https://github.com/pingcap/tiflow/issues/8865) @[hi-rustin](https://github.com/hi-rustin)

    + TiDB Lightning

        - PD `ClientTSOStreamClosed` エラーのチェックサムフェーズ中に TiDB Lightning のリトライロジックを改善します [#45301](https://github.com/pingcap/tidb/issues/45301) @[lance6716](https://github.com/lance6716)
        - インポート後の SQL によるチェックサム検証を改善し、安定性を向上させます [#41941](https://github.com/pingcap/tidb/issues/41941) @[GMHDBJD](https://github.com/GMHDBJD)

    + Dumpling

        - `--sql` パラメータを使用すると、Dumpling はテーブルクエリを実行せず、エクスポートのオーバーヘッドを軽減します [#45239](https://github.com/pingcap/tidb/issues/45239) @[lance6716](https://github.com/lance6716)

    + TiDB Binlog

        - テーブル情報の取得方法を最適化し、Drainer の初期化時間とメモリ使用量を削減します [#1137](https://github.com/pingcap/tidb-binlog/issues/1137) @[lichunzhu](https://github.com/lichunzhu)

## バグ修正

+ TiDB

    - GC Resolve Locks のステップで一部の悲観的ロックを見逃す可能性がある問題を修正します [#45134](https://github.com/pingcap/tidb/issues/45134) @[MyonKeminta](https://github.com/MyonKeminta)
    - 新しいセッションが作成された場合に Stats Collector がデッドロックを引き起こす可能性がある問題を修正します [#44502](https://github.com/pingcap/tidb/issues/44502) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
    - メモリトラッカーに潜在的なメモリリークがある可能性がある問題を修正します [#44612](https://github.com/pingcap/tidb/issues/44612) @[wshwsh12](https://github.com/wshwsh12)
    - バッチコプロセッサーリトライが不正確なリージョン情報を生成し、クエリの失敗を引き起こす可能性がある問題を修正します [#44622](https://github.com/pingcap/tidb/issues/44622) @[windtalker](https://github.com/windtalker)
    - インデックススキャンに潜在的なデータ競合問題を修正します [#45126](https://github.com/pingcap/tidb/issues/45126) @[wshwsh12](https://github.com/wshwsh12)
    - `tidb_enable_parallel_apply` が有効になっている場合に MPP モードでクエリ結果が不正確になる問題を修正します [#45299](https://github.com/pingcap/tidb/issues/45299) @[windtalker](https://github.com/windtalker)
    - `indexMerge` が含まれるクエリをキルした際に発生する待ち状態の問題を修正します [#45279](https://github.com/pingcap/tidb/issues/45279) @[xzhangxian1008](https://github.com/xzhangxian1008)
    - 統計の SQL 実行詳細の過剰なメモリ消費が極端な場合に TiDB OOM を発生させる問題を修正します [#44047](https://github.com/pingcap/tidb/issues/44047) @[wshwsh12](https://github.com/wshwsh12)
    - `FormatSQL()` メソッドが極端に長い SQL 文を適切に切り捨てられない問題を修正します [#44542](https://github.com/pingcap/tidb/issues/44542) @[hawkingrei](https://github.com/hawkingrei)
    - クラスターのアップグレード中に DDL 操作がスタックする問題を修正し、アップグレードの失敗を回避します [#44158](https://github.com/pingcap/tidb/issues/44158) @[zimulala](https://github.com/zimulala)
    - 1 つの TiDB ノードでの障害後に他の TiDB ノードが TTL タスクを引き継がない問題を修正します [#45022](https://github.com/pingcap/tidb/issues/45022) @[lcwangchao](https://github.com/lcwangchao)
    - MySQL カーソルフェッチプロトコルを使用している場合に、結果セットのメモリ消費が `tidb_mem_quota_query` 制限を超え、TiDB OOM を引き起こす問題を修正します。修正後、TiDB は自動的に結果セットをディスクに書き込んでメモリを解放します [#43233](https://github.com/pingcap/tidb/issues/43233) @[YangKeao](https://github.com/YangKeao)
    - 権限がなくてもユーザーが `INFORMATION_SCHEMA.TIFLASH_REPLICA` テーブルの情報を表示できる問題を修正します [#45320](https://github.com/pingcap/tidb/issues/45320) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)
    - `ADMIN SHOW DDL JOBS` ステートメントが返す `ROW_COUNT` が正確でない問題を修正します [#44044](https://github.com/pingcap/tidb/issues/44044) @[tangenta](https://github.com/tangenta)
    - Range COLUMNS でパーティション分割されたテーブルのクエリがエラーになる問題を修正します [#43459](https://github.com/pingcap/tidb/issues/43459) @[mjonss](https://github.com/mjonss)
    - 一時停止している DDL タスクを再開すると失敗する問題を修正します [#44217](https://github.com/pingcap/tidb/issues/44217) @[dhysum](https://github.com/dhysum)
    - インメモリの悲観的ロックが `FLASHBACK` の失敗とデータの不整合を引き起こす問題を修正します [#44292](https://github.com/pingcap/tidb/issues/44292) @[JmPotato](https://github.com/JmPotato)
    - 削除されたテーブルが `INFORMATION_SCHEMA` から読み取れる問題を修正します [#43714](https://github.com/pingcap/tidb/issues/43714) @[tangenta](https://github.com/tangenta)
    - アップグレード前に一時停止している DDL 操作がある場合にクラスターアップグレードが失敗する問題を修正します [#44225](https://github.com/pingcap/tidb/issues/44225) @[zimulala](https://github.com/zimulala)
    - `AUTO_ID_CACHE=1` を使用してテーブルを復元する際に `duplicate entry` エラーが発生する問題を修正します [#44716](https://github.com/pingcap/tidb/issues/44716) @[tiancaiamao](https://github.com/tiancaiamao)
- 複数のDDLオーナーの切り替えによって引き起こされるデータインデックスの不整合の問題を修正 [#44619](https://github.com/pingcap/tidb/issues/44619) @[tangenta](https://github.com/tangenta)
    - `none`ステータスで`ADD INDEX`のDDLタスクをキャンセルすると、このタスクがバックエンドタスクキューから削除されないためメモリリークが発生する問題を修正 [#44205](https://github.com/pingcap/tidb/issues/44205) @[tangenta](https://github.com/tangenta)
    - 特定の誤ったデータ処理時にプロキシプロトコルが`Header read timeout`エラーを報告する問題を修正 [#43205](https://github.com/pingcap/tidb/issues/43205) @[blacktear23](https://github.com/blacktear23)
    - PDアイソレーションが実行中のDDLをブロックする可能性がある問題を修正 [#44267](https://github.com/pingcap/tidb/issues/44267) @[wjhuang2016](https://github.com/wjhuang2016)
    - `SELECT CAST(n AS CHAR)`ステートメントのクエリ結果が、ステートメント内の`n`が負の数の場合に不正確である問題を修正 [#44786](https://github.com/pingcap/tidb/issues/44786) @[xhebox](https://github.com/xhebox)
    - 空のパーティションテーブルを大量に作成した後に、過剰なメモリ使用量の問題を修正 [#44308](https://github.com/pingcap/tidb/issues/44308) @[hawkingrei](https://github.com/hawkingrei)
    - Join Reorderが不正な外部結合結果を引き起こす可能性がある問題を修正 [#44314](https://github.com/pingcap/tidb/issues/44314) @[AilinKid](https://github.com/AilinKid)
    - Common Table Expressions (CTE)を含むクエリがディスクスペース不足を引き起こす可能性がある問題を修正 [#44477](https://github.com/pingcap/tidb/issues/44477) @[guo-shaoge](https://github.com/guo-shaoge)
    - データベースを削除すると、GCの進行が遅くなる問題を修正 [#33069](https://github.com/pingcap/tidb/issues/33069) @[tiancaiamao](https://github.com/tiancaiamao)
    - インジェストモードでインデックスを追加しようとすると失敗する問題を修正 [#44137](https://github.com/pingcap/tidb/issues/44137) @[tangenta](https://github.com/tangenta)
    - テーブルパーティション定義が`FLOOR()`関数を使用してパーティション化された列を丸める場合に、`SELECT`ステートメントがパーティション化されたテーブルのエラーを返す問題を修正 [#42323](https://github.com/pingcap/tidb/issues/42323) @[jiyfhust](https://github.com/jiyfhust)
    - フォロワーリードがフラッシュバックエラーを処理しないため、クエリエラーが発生する問題を修正 [#43673](https://github.com/pingcap/tidb/issues/43673) @[you06](https://github.com/you06)
    - カーソルフェッチ時に`memTracker`を使用するとメモリリークが発生する問題を修正 [#44254](https://github.com/pingcap/tidb/issues/44254) @[YangKeao](https://github.com/YangKeao)
    - `SHOW PROCESSLIST`ステートメントが、サブクエリの実行時間が長い場合にトランザクションの`TxnStart`を表示できない問題を修正 [#40851](https://github.com/pingcap/tidb/issues/40851) @[crazycs520](https://github.com/crazycs520)
    - `LEADING`ヒントがブロックエイリアスのクエリをサポートしていない問題を修正 [#44645](https://github.com/pingcap/tidb/issues/44645) @[qw4990](https://github.com/qw4990)
    - `PREPARE stmt FROM "ANALYZE TABLE xxx"`を使用すると`tidb_mem_quota_query`によって処理が中断される可能性がある問題を修正 [#44320](https://github.com/pingcap/tidb/issues/44320) @[chrysan](https://github.com/chrysan)
    - 空の`processInfo`によって引き起こされるパニックの問題を修正 [#43829](https://github.com/pingcap/tidb/issues/43829) @[zimulala](https://github.com/zimulala)
    - `ON UPDATE`ステートメントが主キーを正しく更新しない場合に、データとインデックスが一貫していない問題を修正 [#44565](https://github.com/pingcap/tidb/issues/44565) @[zyguan](https://github.com/zyguan)
    - `tidb_opt_agg_push_down`が有効な場合、クエリが不正確な結果を返す可能性がある問題を修正 [#44795](https://github.com/pingcap/tidb/issues/44795) @[AilinKid](https://github.com/AilinKid)
    - CTEと相関サブクエリを同時に使用すると、不正確なクエリ結果またはパニックが発生する可能性がある問題を修正 [#44649](https://github.com/pingcap/tidb/issues/44649) [#38170](https://github.com/pingcap/tidb/issues/38170) [#44774](https://github.com/pingcap/tidb/issues/44774) @[winoros](https://github.com/winoros) @[guo-shaoge](https://github.com/guo-shaoge)
    - ロールバック状態でDDLタスクをキャンセルすると、関連するメタデータでエラーが発生する問題を修正 [#44143](https://github.com/pingcap/tidb/issues/44143) @[wjhuang2016](https://github.com/wjhuang2016)
    - `UPDATE`ステートメントの実行中に、外部キー制約のチェックによってエラーが発生する問題を修正 [#44848](https://github.com/pingcap/tidb/issues/44848) @[crazycs520](https://github.com/crazycs520)

+ PD

    - リソースマネージャーがデフォルトのリソースグループを繰り返し初期化する問題を修正 [#6787](https://github.com/tikv/pd/issues/6787) @[glorv](https://github.com/glorv)
    - SQLの配置ルールで設定された`location-labels`が期待どおりにスケジュールされない場合がある問題を修正 [#6662](https://github.com/tikv/pd/issues/6662) @[rleungx](https://github.com/rleungx)
    - 一部の例外的なケースで冗長なレプリカが自動修復されない問題を修正 [#6573](https://github.com/tikv/pd/issues/6573) @[nolouch](https://github.com/nolouch)

+ TiFlash

    - 分散ストレージとコンピュートアーキテクチャモードでTiFlashコンピュートノードが正確でないCPUコア情報を取得する問題を修正 [#7436](https://github.com/pingcap/tiflash/issues/7436) @[guo-shaoge](https://github.com/guo-shaoge)
    - オンライン非安全リカバリを使用した後にTiFlashが再起動に時間がかかる問題を修正 [#7671](https://github.com/pingcap/tiflash/issues/7671) @[hongyunyan](https://github.com/hongyunyan)

+ Tools

    + Backup & Restore (BR)

        - 一部のケースで`checksum mismatch`が誤って報告される問題を修正 [#44472](https://github.com/pingcap/tidb/issues/44472) @[Leavrth](https://github.com/Leavrth)

    + TiCDC

        - PDの例外がレプリケーションタスクがスタックする可能性がある問題を修正 [#8808](https://github.com/pingcap/tiflow/issues/8808) [#9054](https://github.com/pingcap/tiflow/issues/9054) @[asddongmen](https://github.com/asddongmen) @[fubinzh](https://github.com/fubinzh)
        - オブジェクトストレージサービスにレプリケートする際の過剰なメモリ消費の問題を修正 [#8894](https://github.com/pingcap/tiflow/issues/8894) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - リドゥログが有効になっており、下流で例外が発生するとレプリケーションタスクがスタックする可能性がある問題を修正 [#9172](https://github.com/pingcap/tiflow/issues/9172) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 下流に障害が発生した場合、TiCDCが継続的にリトライしすぎてリトライ時間が長すぎる問題を修正 [#9272](https://github.com/pingcap/tiflow/issues/9272) @[asddongmen](https://github.com/asddongmen)
        - 下流のメタデータを頻繁に読み取ることで過剰な圧力が生じる問題を修正 [#8959](https://github.com/pingcap/tiflow/issues/8959) @[hi-rustin](https://github.com/hi-rustin)
        - 下流がKafkaの場合、TiCDCが下流のメタデータを過剰にクエリし、下流で過剰な作業量が発生する問題を修正 [#8957](https://github.com/pingcap/tiflow/issues/8957) [#8959](https://github.com/pingcap/tiflow/issues/8959) @[hi-rustin](https://github.com/hi-rustin)
- 特定のシナリオでソーターのメモリ使用量が過剰であることによるOOM問題を修正 [#8974](https://github.com/pingcap/tiflow/issues/8974) @[hicqu](https://github.com/hicqu) 
- AvroまたはCSVプロトコルを使用する場合に`UPDATE`操作が古い値を出力できない問題を修正[#9086](https://github.com/pingcap/tiflow/issues/9086) @[3AceShowHand](https://github.com/3AceShowHand) 
- データをストレージサービスにレプリケートする際、ダウンストリームのDDLステートメントに対応するJSONファイルがテーブルフィールドのデフォルト値を記録しない問題を修正 [#9066](https://github.com/pingcap/tiflow/issues/9066) @[CharlesCheung96](https://github.com/CharlesCheung96) 
- データをTiDBまたはMySQLにレプリケートする際に、ダウンストリームのバイダイレクショナルレプリケーション関連変数を頻繁に設定することによって、過剰なダウンストリームログが発生する問題を修正[#9180](https://github.com/pingcap/tiflow/issues/9180) @[asddongmen](https://github.com/asddongmen) 
- Kafkaメッセージがサイズオーバーとなりレプリケーションエラーが発生した場合に、メッセージ本文がログに記録される問題を修正 [#9031](https://github.com/pingcap/tiflow/issues/9031) @[darraes](https://github.com/darraes) 
- PDがネットワーク隔離やPDオーナーノードの再起動などによって障害が発生した際、TiCDCが立ち往生する問題を修正 [#8808](https://github.com/pingcap/tiflow/issues/8808) [#8812](https://github.com/pingcap/tiflow/issues/8812) [#8877](https://github.com/pingcap/tiflow/issues/8877) @[asddongmen](https://github.com/asddongmen)
- Avroプロトコルが`Enum`タイプの値を誤って識別する問題を修正 [#9259](https://github.com/pingcap/tiflow/issues/9259) @[3AceShowHand](https://github.com/3AceShowHand) 

+ TiDBデータ移行（DM）

    - 移行対象テーブルにユニークインデックスが空の列を含む場合に、DM-masterが異常終了する問題を修正 [#9247](https://github.com/pingcap/tiflow/issues/9247) @[lance6716](https://github.com/lance6716)

+ TiDB Lightning

    - TiDB LightningとPDの間の接続が失敗した際に、リトライされない問題を修正し、インポート成功率を向上させる[#43400](https://github.com/pingcap/tidb/issues/43400) @[lichunzhu](https://github.com/lichunzhu) 
    - TiDB LightningがTiKVへのデータ書き込みで空き領域エラーが返された際に正しくエラーメッセージを表示しない問題を修正[#44733](https://github.com/pingcap/tidb/issues/44733) @[lance6716](https://github.com/lance6716) 
    - チェックサム操作中に`Region is unavailable`エラーが報告される問題を修正 [#45462](https://github.com/pingcap/tidb/issues/45462) @[D3Hunter](https://github.com/D3Hunter) 
    - `experimental.allow-expression-index` が有効になっており、デフォルト値がUUIDである場合にTiDB Lightningがパニックする問題を修正 [#44497](https://github.com/pingcap/tidb/issues/44497) @[lichunzhu](https://github.com/lichunzhu) 
    - ディスククォータが競合条件により正確でない可能性がある問題を修正 [#44867](https://github.com/pingcap/tidb/issues/44867) @[D3Hunter](https://github.com/D3Hunter) 
    - 論理インポートモードで、インポート中にダウンストリームのテーブルを削除するとTiDB Lightningのメタデータが適時に更新されない問題を修正 [#44614](https://github.com/pingcap/tidb/issues/44614) @[dsdashun](https://github.com/dsdashun)

+ Dumpling

    - `--sql`のクエリ結果セットが空の場合にDumplingが異常終了する問題を修正 [#45200](https://github.com/pingcap/tidb/issues/45200) @[D3Hunter](https://github.com/D3Hunter)

+ TiDB Binlog

    - PDアドレスの完全な変更後に`SHOW PUMP STATUS`または`SHOW DRAINER STATUS`経由でTiDBが正確にBinlogノードの状態をクエリできない問題を修正 [#42643](https://github.com/pingcap/tidb/issues/42643) @[lichunzhu](https://github.com/lichunzhu) 
    - PDアドレスの完全な変更後にTiDBがBinlogを正しく書き込めない問題を修正 [#42643](https://github.com/pingcap/tidb/issues/42643) @[lance6716](https://github.com/lance6716) 
    - 初期化中にetcdクライアントが最新のノード情報を自動的に同期しない問題を修正 [#1236](https://github.com/pingcap/tidb-binlog/issues/1236) @[lichunzhu](https://github.com/lichunzhu)