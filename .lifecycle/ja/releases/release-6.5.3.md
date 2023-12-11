---
title: TiDB 6.5.3リリースノート
summary: TiDB 6.5.3における互換性の変更、改善、およびバグ修正について学ぶ。
---

# TiDB 6.5.3リリースノート

リリース日: 2023年6月14日

TiDBバージョン: 6.5.3

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.5/quick-start-with-tidb) | [本番運用展開](https://docs.pingcap.com/tidb/v6.5/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.5.3#version-list)

## 改善

+ TiDB

    - パートションテーブルにおける`TRUNCATE`のパフォーマンスを改善 [#43070](https://github.com/pingcap/tidb/issues/43070) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)
    - ロックの解決後に無効なステールリードのリトライを避ける [#43659](https://github.com/pingcap/tidb/issues/43659) @[you06](https://github.com/you06)
    - ステールリードが`DataIsNotReady`エラーに遭遇した場合にリーダーリードを使用してレイテンシを低減する [#765](https://github.com/tikv/client-go/pull/765) @[Tema](https://github.com/Tema)
    - ステールリード使用時のヒット率およびトラフィックを追跡するための`Stale Read OPS`および`Stale Read MBps`メトリクスを追加する [#43325](https://github.com/pingcap/tidb/issues/43325) @[you06](https://github.com/you06)

+ TiKV

    - `check_leader`リクエストを圧縮するためにgzipを使用してトラフィックを低減する [#14839](https://github.com/tikv/tikv/issues/14839) @[cfzjywxk](https://github.com/cfzjywxk)

+ PD

    - 他のリクエストへの影響を防ぐためにPDリーダー選出用の独立したgRPC接続を使用する [#6403](https://github.com/tikv/pd/issues/6403) @[rleungx](https://github.com/rleungx)

+ ツール

    + TiCDC

        - DDLを処理する方法を最適化し、DDLが関連しない他のDMLイベントの使用をブロックせず、メモリ使用量を削減する [#8106](https://github.com/pingcap/tiflow/issues/8106) @[asddongmen](https://github.com/asddongmen)
        - デコーダーインターフェースを最適化し、新しい`AddKeyValue`メソッドを追加する [#8861](https://github.com/pingcap/tiflow/issues/8861) @[3AceShowHand](https://github.com/3AceShowHand)
        - データをオブジェクトストレージにレプリケートするシナリオでDDLイベントが発生した際のディレクトリ構造を最適化する [#8890](https://github.com/pingcap/tiflow/issues/8890) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - Pulsarダウンストリームにデータをレプリケートするサポートを追加する [#8892](https://github.com/pingcap/tiflow/issues/8892) @[hi-rustin](https://github.com/hi-rustin)
        - データをKafkaにレプリケートする際の検証にOAuthプロトコルの使用をサポートする [#8865](https://github.com/pingcap/tiflow/issues/8865) @[hi-rustin](https://github.com/hi-rustin)
        - AvroまたはCSVプロトコルを使用してデータをレプリケートする際の`UPDATE`ステートメントの処理方法を最適化し、`UPDATE`を`DELETE`および`INSERT`ステートメントに分割することで、`DELETE`ステートメントから古い値を取得できるようにする [#9086](https://github.com/pingcap/tiflow/issues/9086) @[3AceShowHand](https://github.com/3AceShowHand)
        - TLSを有効にしているシナリオにおける認証アルゴリズムの設定を制御するための`insecure-skip-verify`構成項目を追加する [#8867](https://github.com/pingcap/tiflow/issues/8867) @[hi-rustin](https://github.com/hi-rustin)
        - DDLレプリケーション操作を最適化し、ダウンストリームのレイテンシに対するDDL操作の影響を緩和する [#8686](https://github.com/pingcap/tiflow/issues/8686) @[hi-rustin](https://github.com/hi-rustin)
        - TiCDCレプリケーションタスクが失敗した際の上流のGC TLS設定方法を最適化する [#8403](https://github.com/pingcap/tiflow/issues/8403) @[charleszheng44](https://github.com/charleszheng44)

    + TiDB Binlog

        - ドレイナーの初期化時間とメモリ使用量を削減するためにテーブル情報を取得する方法を最適化する [#1137](https://github.com/pingcap/tidb-binlog/issues/1137) @[lichunzhu](https://github.com/lichunzhu)

## バグ修正

+ TiDB

    - `min, max`クエリ結果が正しくない問題を修正する [#43805](https://github.com/pingcap/tidb/issues/43805) @[wshwsh12](https://github.com/wshwsh12)
    - ウィンドウ関数をTiFlashにプッシュダウンする際の実行計画が正しくない問題を修正する [#43922](https://github.com/pingcap/tidb/issues/43922) @[gengliqi](https://github.com/gengliqi)
    - CTEを使用したクエリによりTiDBがフリーズする問題を修正する [#43749](https://github.com/pingcap/tidb/issues/43749) [#36896](https://github.com/pingcap/tidb/issues/36896) @[guo-shaoge](https://github.com/guo-shaoge)
    - `AES_DECRYPT`式を使用した際に`runtime error: index out of range`エラーを報告する問題を修正する [#43063](https://github.com/pingcap/tidb/issues/43063) @[lcwangchao](https://github.com/lcwangchao)
    - `SHOW PROCESSLIST`ステートメントが長いサブクエリ時間を持つステートメントのトランザクションのTxnStartを表示できない問題を修正する [#40851](https://github.com/pingcap/tidb/issues/40851) @[crazycs520](https://github.com/crazycs520)
    - PDの分離が実行中のDDLをブロックする可能性がある問題を修正する [#44014](https://github.com/pingcap/tidb/issues/44014) [#43755](https://github.com/pingcap/tidb/issues/43755) [#44267](https://github.com/pingcap/tidb/issues/44267) @[wjhuang2016](https://github.com/wjhuang2016)
    - `UNION`を使用した場合にTiDBが一時テーブルと結合ビューのクエリでパニックを起こす問題を修正する [#42563](https://github.com/pingcap/tidb/issues/42563) @[lcwangchao](https://github.com/lcwangchao)
    - パーティションされたテーブルのPlacement Rulesに関する動作の問題を修正し、削除されたパーティションのPlacement Rulesが正しく設定およびリサイクルされるようにする [#44116](https://github.com/pingcap/tidb/issues/44116) @[lcwangchao](https://github.com/lcwangchao)
    - パーティションのTRUNCATEが原因でパーティションのPlacement Ruleが無効になる問題を修正する [#44031](https://github.com/pingcap/tidb/issues/44031) @[lcwangchao](https://github.com/lcwangchao)
    - TiCDCがテーブルのリネーム中に一部の行変更を失う可能性がある問題を修正する [#43338](https://github.com/pingcap/tidb/issues/43338) @[tangenta](https://github.com/tangenta)
    - BRを使用してテーブルをインポートした際にDDLジョブ履歴が失われる問題を修正する [#43725](https://github.com/pingcap/tidb/issues/43725) @[tangenta](https://github.com/tangenta)
    - 一部のケースで`JSON_OBJECT`がエラーを報告する問題を修正する [#39806](https://github.com/pingcap/tidb/issues/39806) @[YangKeao](https://github.com/YangKeao)
    - クラスタがIPv6環境で一部のシステムビューをクエリできない問題を修正する [#43286](https://github.com/pingcap/tidb/issues/43286) @[Defined2014](https://github.com/Defined2014)
    - PDメンバーアドレスが変更された際に`AUTO_INCREMENT`列のID割り当てが長時間ブロックされる問題を修正する [#42643](https://github.com/pingcap/tidb/issues/42643) @[tiancaiamao](https://github.com/tiancaiamao)
    - パーティションルールのリサイクル中にTiDBがPDに重複したリクエストを送信し、PDログに多数の`full config reset`エントリを引き起こす問題を修正する [#33069](https://github.com/pingcap/tidb/issues/33069) @[tiancaiamao](https://github.com/tiancaiamao)
- `SHOW PRIVILEGES`ステートメントが不完全な権限リストを返す問題を修正する[#40591](https://github.com/pingcap/tidb/issues/40591) @[CbcWestwolf](https://github.com/CbcWestwolf)を修正する
- `ADMIN SHOW DDL JOBS LIMIT`が正しくない結果を返す問題を修正する[#42298](https://github.com/pingcap/tidb/issues/42298) @[CbcWestwolf](https://github.com/CbcWestwolf)を修正する
- `tidb_auth_token`ユーザーが作成されない問題を修正する(パスワードの複雑さチェックが有効になっている場合) [#44098](https://github.com/pingcap/tidb/issues/44098) @[CbcWestwolf](https://github.com/CbcWestwolf)を修正する
- 動的プルーニングモードでの内部結合時にパーティションが見つからない問題を修正する[#43686](https://github.com/pingcap/tidb/issues/43686) @[mjonss](https://github.com/mjonss)を修正する
- パーティションテーブルで`MODIFY COLUMN`を実行する際に`Data Truncated`警告が発生する問題を修正する[#41118](https://github.com/pingcap/tidb/issues/41118) @[mjonss](https://github.com/mjonss)を修正する
- IPv6環境でTiDBアドレスが正しくない問題を修正する[#43260](https://github.com/pingcap/tidb/issues/43260) @[nexustar](https://github.com/nexustar)を修正する
- CTE結果が不正な場合がある問題を修正する(プレディケートのプッシュダウン時) [#43645](https://github.com/pingcap/tidb/issues/43645) @[winoros](https://github.com/winoros)を修正する
- 非相関サブクエリを含むステートメントで共通テーブル式(CTE)を使用すると不正な結果が返される可能性がある問題を修正する[#44051](https://github.com/pingcap/tidb/issues/44051) @[winoros](https://github.com/winoros)を修正する
- Join Reorderが不正な外部結合結果を引き起こす可能性がある問題を修正する[#44314](https://github.com/pingcap/tidb/issues/44314) @[AilinKid](https://github.com/AilinKid)を修正する
- 悲観的トランザクションの最初のステートメントがリトライされた場合にそのトランザクションのロック解除がトランザクションの正確性に影響を与える可能性がある問題を修正する[#42937](https://github.com/pingcap/tidb/issues/42937) @[MyonKeminta](https://github.com/MyonKeminta)を修正する
- 悲観的トランザクションの残留悲観ロックがGCがロック解除を行う際、データの正確性に影響を与える可能性がある問題を修正する[#43243](https://github.com/pingcap/tidb/issues/43243) @[MyonKeminta](https://github.com/MyonKeminta)を修正する
- `batch cop`の実行中にスキャンの詳細情報が不正確になる可能性がある問題を修正する[#41582](https://github.com/pingcap/tidb/issues/41582) @[you06](https://github.com/you06)を修正する
- Stale Readと`PREPARE`ステートメントを同時に使用した場合、TiDBがデータ更新を読み取れない問題を修正する[#43044](https://github.com/pingcap/tidb/issues/43044) @[you06](https://github.com/you06)を修正する
- `LOAD DATA`ステートメントを実行する際に「assertion failed」エラーが誤って報告される可能性がある問題を修正する[#43849](https://github.com/pingcap/tidb/issues/43849) @[you06](https://github.com/you06)を修正する
- `region data not ready`エラーが発生した際にcoprocessorがリーダーにフォールバックできない問題を修正する[#43365](https://github.com/pingcap/tidb/issues/43365) @[you06](https://github.com/you06)を修正する

+ TiKV

    - TiKVノードが失敗した際に対応するリージョンのピアが誤ってハイバーネートする問題を修正する[#14547](https://github.com/tikv/tikv/issues/14547) @[hicqu](https://github.com/hicqu)を修正する
    - 連続プロファイリングでファイルハンドルが漏れる問題を修正する[#14224](https://github.com/tikv/tikv/issues/14224) @[tabokie](https://github.com/tabokie)を修正する
    - PDクラッシュがPITRの進行を妨げる可能性がある問題を修正する[#14184](https://github.com/tikv/tikv/issues/14184) @[YuJuncen](https://github.com/YuJuncen)を修正する
    - 暗号化キーIDの競合が古いキーの削除を引き起こす可能性がある問題を修正する[#14585](https://github.com/tikv/tikv/issues/14585) @[tabokie](https://github.com/tabokie)を修正する
    - オートコミットとポイントゲットレプリカ読み取りが直線性を壊す可能性がある問題を修正する[#14715](https://github.com/tikv/tikv/issues/14715) @[cfzjywxk](https://github.com/cfzjywxk)を修正する
    - クラスターを前のバージョンからv6.5またはそれ以降のバージョンにアップグレードした際に蓄積されたロックレコードがパフォーマンス劣化を引き起こす問題を修正する[#14780](https://github.com/tikv/tikv/issues/14780) @[MyonKeminta](https://github.com/MyonKeminta)を修正する
    - TiDB LightningがSSTファイルの漏洩を引き起こす可能性がある問題を修正する[#14745](https://github.com/tikv/tikv/issues/14745) @[YuJuncen](https://github.com/YuJuncen)を修正する
    - 暗号化キーとraftログファイルの削除との間に競合が生じTiKVが起動に失敗する可能性がある問題を修正する[#14761](https://github.com/tikv/tikv/issues/14761) @[Connor1996](https://github.com/Connor1996)を修正する

+ TiFlash

    - リージョン転送時のパーティションTableScan演算子のパフォーマンス劣化問題を修正する[#7519](https://github.com/pingcap/tiflash/issues/7519) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)を修正する
    - `GENERATED`タイプのフィールドが`TIMESTAMP`または`TIME`タイプと一緒に存在する場合、TiFlashクエリでエラーが報告される可能性がある問題を修正する[#7468](https://github.com/pingcap/tiflash/issues/7468) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)を修正する
    - 大規模な更新トランザクションがTiFlashで繰り返しエラーを報告し、再起動する可能性がある問題を修正する[#7316](https://github.com/pingcap/tiflash/issues/7316) @[JaySon-Huang](https://github.com/JaySon-Huang)を修正する
    - `INSERT SELECT`ステートメントでTiFlashからデータを読み取る際に「Truncate error cast decimal as decimal」エラーが発生する問題を修正する[#7348](https://github.com/pingcap/tiflash/issues/7348) @[windtalker](https://github.com/windtalker)を修正する
    - クエリが不要なメモリを消費する可能性がある問題を修正する(Join build sideのデータが非常に大きい場合で、多くの小さな文字列型のカラムを含む場合) [#7416](https://github.com/pingcap/tiflash/issues/7416) @[yibin87](https://github.com/yibin87)を修正する

+ Tools

    + Backup & Restore (BR)

        - バックアップに失敗した際にBRのエラーメッセージが実際のエラー情報を隠している問題を修正する（「resolve lock timeout」エラーメッセージが誤解を招く）[#43236](https://github.com/pingcap/tidb/issues/43236) @[YuJuncen](https://github.com/YuJuncen)を修正する

    + TiCDC

        - 5万個のテーブルが存在する場合に発生する可能性があるOOM問題を修正する[#7872](https://github.com/pingcap/tiflow/issues/7872) @[sdojjy](https://github.com/sdojjy)を修正する
        - upstream TiDBでOOMが発生した際にTiCDCがスタックする問題を修正する[#8561](https://github.com/pingcap/tiflow/issues/8561) @[overvenus](https://github.com/overvenus)を修正する
        - PDのネットワーク隔離やPD Ownerノードの再起動などの際にTiCDCがスタックする問題を修正する[#8808](https://github.com/pingcap/tiflow/issues/8808) [#8812](https://github.com/pingcap/tiflow/issues/8812) [#8877](https://github.com/pingcap/tiflow/issues/8877) @[asddongmen](https://github.com/asddongmen)を修正する
        - TiCDCのタイムゾーン設定の問題を修正する[#8798](https://github.com/pingcap/tiflow/issues/8798) @[hi-rustin](https://github.com/hi-rustin)を修正する
        - upstream TiKVノードの1つがクラッシュした場合にチェックポイントのラグが増加する問題を修正する[#8858](https://github.com/pingcap/tiflow/issues/8858) @[hicqu](https://github.com/hicqu)を修正する
        - upstream TiDBで`FLASHBACK CLUSTER TO TIMESTAMP`ステートメントが実行された後に下流のMySQLにデータをレプリケートする際にレプリケーションエラーが発生する問題を修正する[#8040](https://github.com/pingcap/tiflow/issues/8040) @[asddongmen](https://github.com/asddongmen)を修正する
        - データをオブジェクトストレージに複製する際に、上流の`EXCHANGE PARTITION`操作が正しく下流にレプリケートされない問題を修正 [#8914](https://github.com/pingcap/tiflow/issues/8914) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 特定の特殊シナリオでソーターのコンポーネントの過剰なメモリ使用によって発生するOOM問題を修正 [#8974](https://github.com/pingcap/tiflow/issues/8974) @[hicqu](https://github.com/hicqu)
        - 下流がKafkaの場合、TiCDCが下流のメタデータを過度に頻繁にクエリし、下流で過剰なワークロードを引き起こす問題を修正 [#8957](https://github.com/pingcap/tiflow/issues/8957) [#8959](https://github.com/pingcap/tiflow/issues/8959) @[hi-rustin](https://github.com/hi-rustin)
        - オーバーサイズのKafkaメッセージによるレプリケーションエラーが発生した場合、メッセージ本文がログに記録される問題を修正 [#9031](https://github.com/pingcap/tiflow/issues/9031) @[darraes](https://github.com/darraes)
        - 下流のKafkaシンクがローリング再起動される際に発生するTiCDCノードのパニックを修正 [#9023](https://github.com/pingcap/tiflow/issues/9023) @[asddongmen](https://github.com/asddongmen)
        - データをストレージサービスにレプリケートする際に、下流のDDLステートメントに対応するJSONファイルがテーブルフィールドのデフォルト値を記録しない問題を修正 [#9066](https://github.com/pingcap/tiflow/issues/9066) @[CharlesCheung96](https://github.com/CharlesCheung96)

    + TiDB Lightning

        - 広いテーブルをインポートする際にOOMが発生する可能性がある問題を修正 [#43728](https://github.com/pingcap/tidb/issues/43728) @[D3Hunter](https://github.com/D3Hunter)
        - 大量のデータをインポートする際に`write to tikv with no leader returned`の問題を修正 [#43055](https://github.com/pingcap/tidb/issues/43055) @[lance6716](https://github.com/lance6716)
        - データファイルに未閉じの区切り記号がある場合に発生する可能性のあるOOM問題を修正 [#40400](https://github.com/pingcap/tidb/issues/40400) @[buchuitoudegou](https://github.com/buchuitoudegou)
        - データのインポート中に`unknown RPC`エラーに遭遇した場合のリトライメカニズムを追加 [#43291](https://github.com/pingcap/tidb/issues/43291) @[D3Hunter](https://github.com/D3Hunter)

    + TiDB Binlog

        - `CANCELED`のDDLステートメントに遭遇した際にTiDB Binlogがエラーを報告する問題を修正 [#1228](https://github.com/pingcap/tidb-binlog/issues/1228) @[okJiang](https://github.com/okJiang)