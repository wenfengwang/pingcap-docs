---
title: TiDB 6.1.1 リリースノート
---

# TiDB 6.1.1 リリースノート

リリース日: 2022年9月1日

TiDB バージョン: 6.1.1

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.1/quick-start-with-tidb) | [本番展開](https://docs.pingcap.com/tidb/v6.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.1.1#version-list)

## 互換性の変更

+ TiDB

    - `SHOW DATABASES LIKE …` ステートメントを大文字小文字を区別しないように変更 [#34766](https://github.com/pingcap/tidb/issues/34766) @[e1ijah1](https://github.com/e1ijah1)
    - デフォルト値の [`tidb_enable_outer_join_reorder`](/system-variables.md#tidb_enable_outer_join_reorder-new-in-v610) を `1` から `0` に変更し、デフォルトで Join Reorder の Outer Join サポートを無効にする。

+ 診断

    - Continuous Profiling 機能をデフォルトで無効にし、この機能を有効にした場合に発生する可能性のある TiFlash クラッシュの問題を回避します。詳細については、[#5687](https://github.com/pingcap/tiflash/issues/5687) を参照してください @[mornyx](https://github.com/mornyx)

## その他の変更

- `TiDB-community-toolkit` バイナリパッケージに以下の内容を追加しました。詳細については、[TiDB インストールパッケージ](/binary-package.md) を参照してください。

    - `server-{version}-linux-amd64.tar.gz`
    - `grafana-{version}-linux-amd64.tar.gz`
    - `alertmanager-{version}-linux-amd64.tar.gz`
    - `prometheus-{version}-linux-amd64.tar.gz`
    - `blackbox_exporter-{version}-linux-amd64.tar.gz`
    - `node_exporter-{version}-linux-amd64.tar.gz`

- オペレーティングシステムと CPU アーキテクチャの組み合わせに対する異なる品質基準のマルチレベルサポートを導入しました。[OS and platform requirements](https://docs.pingcap.com/tidb/v6.1/hardware-and-software-requirements#os-and-platform-requirements) を参照してください。

## 改善

+ TiDB

    - `EXISTS` クエリのパフォーマンスを改善するために新しい optimizer `SEMI_JOIN_REWRITE` を追加 [#35323](https://github.com/pingcap/tidb/issues/35323) @[winoros](https://github.com/winoros)

+ TiKV

    - gzip を使用してメトリクス応答を圧縮し、HTTP ボディサイズを削減するサポートを提供します [#12355](https://github.com/tikv/tikv/issues/12355) @[winoros](https://github.com/winoros)
    - [`server.simplify-metrics`](https://docs.pingcap.com/tidb/v6.1/tikv-configuration-file#simplify-metrics-new-in-v611) 構成項目を使用して、一部のメトリクスをフィルタリングして各リクエストで返されるデータ量を削減するサポートを提供します [#12355](https://github.com/tikv/tikv/issues/12355) @[glorv](https://github.com/glorv)
    - RocksDB (`rocksdb.max-sub-compactions`) で同時に実行される sub-compaction 操作の数を動的に変更するサポートを提供します [#13145](https://github.com/tikv/tikv/issues/13145) @[ethercflow](https://github.com/ethercflow)

+ PD

    - 特定の段階での Balance Region のスケジューリング速度を改善しました [#4990](https://github.com/tikv/pd/issues/4990) @[bufferflies](https://github.com/bufferflies)

+ ツール

    + TiDB Lightning

        - `stale command` のようなエラーに対するリトライメカニズムを追加し、インポート成功率を向上させました [#36877](https://github.com/pingcap/tidb/issues/36877) @[D3Hunter](https://github.com/D3Hunter)

    + TiDB データ移行（DM）

        - ユーザーは lightning loader の並行性の量を手動で設定できます [#5505](https://github.com/pingcap/tiflow/issues/5505) @[buchuitoudegou](https://github.com/buchuitoudegou)

    + TiCDC

        - 大規模トランザクションのメモリ消費と遅延を大幅に削減するために、`transaction-atomicity` シンク URI パラメータを追加しました [#5231](https://github.com/pingcap/tiflow/issues/5231) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 複数のリージョンシナリオでのランタイムコンテキスト切り替えによるパフォーマンスオーバーヘッドを削減しました [#5610](https://github.com/pingcap/tiflow/issues/5610) @[hicqu](https://github.com/hicqu)
        - MySQL シンクを自動的にセーフモードをオフにすることで強化しました [#5611](https://github.com/pingcap/tiflow/issues/5611) @[overvenus](https://github.com/overvenus)

## バグ修正

+ TiDB

    - `LIMIT` と一緒に使用された場合に `INL_HASH_JOIN` がハングアップする問題を修正しました [#35638](https://github.com/pingcap/tidb/issues/35638) @[guo-shaoge](https://github.com/guo-shaoge)
    - `UPDATE` ステートメントを実行すると TiDB がパニックする可能性がある問題を修正しました [#32311](https://github.com/pingcap/tidb/issues/32311) @[Yisaer](https://github.com/Yisaer)
    - `SHOW COLUMNS` ステートメントを実行すると TiDB が coprocessor リクエストを送信する可能性があるバグを修正しました [#36496](https://github.com/pingcap/tidb/issues/36496) @[tangenta](https://github.com/tangenta)
    - `SHOW WARNINGS` ステートメントを実行すると `invalid memory address or nil pointer dereference` エラーが返される可能性があるバグを修正しました [#31569](https://github.com/pingcap/tidb/issues/31569) @[zyguan](https://github.com/zyguan)
    - 集約条件を持つ SQL ステートメントがテーブルが空の場合に誤った結果を返す可能性がある問題を修正しました（静的パーティションプルーニングモード） [#35295](https://github.com/pingcap/tidb/issues/35295) @[tiancaiamao](https://github.com/tiancaiamao)
    - Join Reorder 操作が誤って Outer Join 条件をプッシュダウンする問題を修正しました [#37238](https://github.com/pingcap/tidb/issues/37238) @[winoros](https://github.com/winoros)
    - CTE-schema ハッシュコードが誤ってクローンされ、CTE が複数回参照された場合に `Can't find column ... in schema ...` エラーが発生する問題を修正しました [#35404](https://github.com/pingcap/tidb/issues/35404) @[AilinKid](https://github.com/AilinKid)
    - 特定の右外部結合シナリオでの誤った Join Reorder がクエリ結果を誤らせる問題を修正しました [#36912](https://github.com/pingcap/tidb/issues/36912) @[winoros](https://github.com/winoros)
    - EqualAll ケースで TiFlash `firstrow` 集約関数の null フラグが正しく推論されていない問題を修正しました [#34584](https://github.com/pingcap/tidb/issues/34584) @[fixdb](https://github.com/fixdb)
    - バインディングが `IGNORE_PLAN_CACHE` ヒントで作成された場合に Plan Cache が機能しない問題を修正しました [#34596](https://github.com/pingcap/tidb/issues/34596) @[fzzf678](https://github.com/fzzf678)
    - ハッシュパーティションウィンドウと単一パーティションウィンドウ間に `EXCHANGE` オペレーターが不足している問題を修正しました [#35990](https://github.com/pingcap/tidb/issues/35990) @[LittleFall](https://github.com/LittleFall)
    - パーティション化されたテーブルがいくつかのケースでインデックスを完全に使用できない問題を修正しました [#33966](https://github.com/pingcap/tidb/issues/33966) @[mjonss](https://github.com/mjonss)
    - 集約がプッシュダウンされた後に誤ったデフォルト値が設定された場合に誤ったクエリ結果が返される問題を修正しました [#35295](https://github.com/pingcap/tidb/issues/35295) @[tiancaiamao](https://github.com/tiancaiamao)
    - クエリ実行時に異なるコラートがクエリパーティションテーブルと異なる場合にパーティションが誤ってプルーニングされる可能性がある問題を修正しました [#32749](https://github.com/pingcap/tidb/issues/32749) @[mjonss](https://github.com/mjonss)
  - TiDB Binlogを有効にした場合、`ALTER SEQUENCE`ステートメントを実行すると誤ったメタデータバージョンが発生し、Drainerが終了する可能性のある問題を修正しました [#36276](https://github.com/pingcap/tidb/issues/36276) @[AilinKid](https://github.com/AilinKid)
    - 一部の極端なケースで起動時に発生する可能性のあるTiDBステータスの不正な問題を修正しました [#36791](https://github.com/pingcap/tidb/issues/36791) @[xhebox](https://github.com/xhebox)
    - TiDB Dashboardでパーティションテーブルの実行計画をクエリする際に発生する`UnknownPlanID`の問題を修正しました [#35153](https://github.com/pingcap/tidb/issues/35153) @[time-and-fate](https://github.com/time-and-fate)
    - `LOAD DATA`ステートメントにおいてカラムリストが機能しない問題を修正しました [#35198](https://github.com/pingcap/tidb/issues/35198) @[SpadeA-Tang](https://github.com/SpadeA-Tang)
    - TiDB Binlogを有効にした状態で重複した値を挿入すると`data and columnID count not match`エラーが発生する問題を修正しました [#33608](https://github.com/pingcap/tidb/issues/33608) @[zyguan](https://github.com/zyguan)
    - `tidb_gc_life_time`の制限を削除しました [#35392](https://github.com/pingcap/tidb/issues/35392) @[TonsnakeLin](https://github.com/TonsnakeLin)
    - 空のフィールド終端記号が使用された場合に`LOAD DATA`ステートメントがデッドループする問題を修正しました [#33298](https://github.com/pingcap/tidb/issues/33298) @[zyguan](https://github.com/zyguan)
    - 可用性を向上させるため、不健康なTiKVノードにリクエストを送信しないようにしました [#34906](https://github.com/pingcap/tidb/issues/34906) @[sticnarf](https://github.com/sticnarf)

+ TiKV

    - Raftstoreがビジーな場合にRegionsが重複する可能性があるバグを修正しました [#13160](https://github.com/tikv/tikv/issues/13160) @[5kbpers](https://github.com/5kbpers)
    - Regionのハートビートが中断された後にPDがTiKVに再接続しない問題を修正しました [#12934](https://github.com/tikv/tikv/issues/12934) @[bufferflies](https://github.com/bufferflies)
    - 空の文字列の型変換を実行する際にTiKVがパニックを起こす問題を修正しました [#12673](https://github.com/tikv/tikv/issues/12673) @[wshwsh12](https://github.com/wshwsh12)
    - TiKVとPDの間でRegionサイズ設定が不一致になる問題を修正しました [#12518](https://github.com/tikv/tikv/issues/12518) @[5kbpers](https://github.com/5kbpers)
    - Raft Engineが有効になっているとき、暗号化キーがクリーンアップされない問題を修正しました [#12890](https://github.com/tikv/tikv/issues/12890) @[tabokie](https://github.com/tabokie)
    - ピアが分割されて同時に破壊されているときに発生する可能性のあるパニック問題を修正しました [#12825](https://github.com/tikv/tikv/issues/12825) @[BusyJay](https://github.com/BusyJay)
    - リージョンのマージプロセスでスナップショットによるログの追いつき中に発生する可能性のあるパニック問題を修正しました [#12663](https://github.com/tikv/tikv/issues/12663) @[BusyJay](https://github.com/BusyJay)
    - PDクライアントがエラーに遭遇した際に頻繁に再接続する問題を修正しました [#12345](https://github.com/tikv/tikv/issues/12345) @[Connor1996](https://github.com/Connor1996)
    - Raft Engineの並行回復が有効な場合に発生する可能性のあるパニックを修正しました [#13123](https://github.com/tikv/tikv/issues/13123) @[tabokie](https://github.com/tabokie)
    - 新しいRegionのコミットログの持続時間が高すぎてQPSが低下する問題を修正しました [#13077](https://github.com/tikv/tikv/issues/13077) @[Connor1996](https://github.com/Connor1996)
    - Raft Engineが有効な場合に発生する稀なパニックを修正しました [#12698](https://github.com/tikv/tikv/issues/12698) @[tabokie](https://github.com/tabokie)
    - procfsが見つからない場合に冗長なログ警告を出さないようにしました [#13116](https://github.com/tikv/tikv/issues/13116) @[tabokie](https://github.com/tabokie)
    - ダッシュボードで`Unified Read Pool CPU`の間違った表現を修正しました [#13086](https://github.com/tikv/tikv/issues/13086) @[glorv](https://github.com/glorv)
    - リージョンが大きい場合、デフォルトの [`region-split-check-diff`](/tikv-configuration-file.md#region-split-check-diff) がバケットサイズよりも大きくなってしまう問題を修正しました [#12598](https://github.com/tikv/tikv/issues/12598) @[tonyxuqqi](https://github.com/tonyxuqqi)
    - スナップショットの適用が中止された際にTiKVがパニックする可能性のある問題を修正しました [#12470](https://github.com/tikv/tikv/issues/12470) @[tabokie](https://github.com/tabokie)
    - PDクライアントがデッドロックを起こす可能性のある問題を修正しました [#13191](https://github.com/tikv/tikv/issues/13191) @[bufferflies](https://github.com/bufferflies) [#12933](https://github.com/tikv/tikv/issues/12933) @[BurtonQin](https://github.com/BurtonQin)

+ PD

    - クラスタノードのラベル設定が無効な場合にオンラインの進捗が不正確になる問題を修正しました [#5234](https://github.com/tikv/pd/issues/5234) @[rleungx](https://github.com/rleungx)
    - `enable-forwarding`が有効な場合、gRPCがエラーを不適切に処理してPDがパニックする問題を修正しました [#5373](https://github.com/tikv/pd/issues/5373) @[bufferflies](https://github.com/bufferflies)
    - `/regions/replicated`が誤ったステータスを返す可能性がある問題を修正しました [#5095](https://github.com/tikv/pd/issues/5095) @[rleungx](https://github.com/rleungx)

+ TiFlash

    - クラスタ内でIPv6を使用する場合、TiFlashが動作しない可能性があるバグを修正しました [#5247](https://github.com/pingcap/tiflash/issues/5247) @[solotzg](https://github.com/solotzg)
    - クラスタ化されたインデックスを持つテーブルのカラムを削除した後、TiFlashがクラッシュする問題を修正しました [#5154](https://github.com/pingcap/tiflash/issues/5154) @[hongyunyan](https://github.com/hongyunyan)
    - `format`関数が`Data truncated`エラーを返す可能性がある問題を修正しました [#4891](https://github.com/pingcap/tiflash/issues/4891) @[xzhangxian1008](https://github.com/xzhangxian1008)
    - 一部の状況で過去のデータがストレージに残り、削除できない問題を修正しました [#5659](https://github.com/pingcap/tiflash/issues/5659) @[lidezhu](https://github.com/lidezhu)
    - 一部の状況で不要なCPU使用率が発生する問題を修正しました [#5409](https://github.com/pingcap/tiflash/issues/5409) @[breezewish](https://github.com/breezewish)
    - パラレル集約中にTiFlashがエラーおよびスレッドリソースのリークを引き起こす可能性がある問題を修正しました [#5356](https://github.com/pingcap/tiflash/issues/5356) @[gengliqi](https://github.com/gengliqi)
    - `MinTSOScheduler`クエリエラーが発生した場合にスレッドリソースがリークする可能性がある問題を修正しました [#5556](https://github.com/pingcap/tiflash/issues/5556) @[windtalker](https://github.com/windtalker)

+ Tools

    + TiDB Lightning

        - TiDBがIPv6ホストを使用する場合にTiDB Lightningが接続に失敗する問題を修正しました [#35880](https://github.com/pingcap/tidb/issues/35880) @[D3Hunter](https://github.com/D3Hunter)
        - `read index not ready`エラーを追加のリトライ機構で修正しました [#36566](https://github.com/pingcap/tidb/issues/36566) @[D3Hunter](https://github.com/D3Hunter)
        - サーバーモードでログに含まれる機密情報を出力する問題を修正しました [#36374](https://github.com/pingcap/tidb/issues/36374) @[lichunzhu](https://github.com/lichunzhu)
      - TiDB Lightningの問題を修正し、Parquetファイル内でスラッシュ、数字、または非ASCII文字で始まる列に対応させる[#36980](https://github.com/pingcap/tidb/issues/36980) @[D3Hunter](https://github.com/D3Hunter)
      - 重複排除が極端なケースでTiDB Lightningをパニック状態にする可能性のある問題を修正する[#34163](https://github.com/pingcap/tidb/issues/34163) @[ForwardStar](https://github.com/ForwardStar)

    + TiDBデータ移行（DM）

      - `txn-entry-size-limit`構成項目がDMで効果を発揮しない問題を修正する[#6161](https://github.com/pingcap/tiflow/issues/6161) @[ForwardStar](https://github.com/ForwardStar)
      - `check-task`コマンドが特殊文字を処理できない問題を修正する[#5895](https://github.com/pingcap/tiflow/issues/5895) @[Ehco1996](https://github.com/Ehco1996)
      - `query-status`で可能性のあるデータ競合の問題を修正する[#4811](https://github.com/pingcap/tiflow/issues/4811) @[lyzx2001](https://github.com/lyzx2001)
      - `operate-schema`コマンドの出力形式が異なる問題を修正する[#5688](https://github.com/pingcap/tiflow/issues/5688) @[ForwardStar](https://github.com/ForwardStar)
      - リレーでエラーが発生した時のゴルーチンリークを修正する[#6193](https://github.com/pingcap/tiflow/issues/6193) @[lance6716](https://github.com/lance6716)
      - DM WorkerがDB Connを取得する際にスタックする可能性がある問題を修正する[#3733](https://github.com/pingcap/tiflow/issues/3733) @[lance6716](https://github.com/lance6716)
      - TiDBがIPv6ホストを使用する場合にDMが起動できない問題を修正する[#6249](https://github.com/pingcap/tiflow/issues/6249) @[D3Hunter](https://github.com/D3Hunter)

    + TiCDC

      - 誤った最大互換性バージョン番号を修正する[#6039](https://github.com/pingcap/tiflow/issues/6039) @[hi-rustin](https://github.com/hi-rustin)
      - cdcサーバが完全に起動する前にHTTPリクエストを受信するとパニックを引き起こす可能性のあるバグを修正する[#5639](https://github.com/pingcap/tiflow/issues/5639) @[asddongmen](https://github.com/asddongmen)
      - changefeed sync-pointが有効になっている場合にddl sinkがパニックを引き起こす問題を修正する[#4934](https://github.com/pingcap/tiflow/issues/4934) @[asddongmen](https://github.com/asddongmen)
      - sync-pointが有効な場合に、特定のシナリオでchangefeedがスタックする問題を修正する[#6827](https://github.com/pingcap/tiflow/issues/6827) @[hicqu](https://github.com/hicqu)
      - cdcサーバが再起動後にchangefeed APIが正しく機能しないバグを修正する[#5837](https://github.com/pingcap/tiflow/issues/5837) @[asddongmen](https://github.com/asddongmen)
      - black hole sinkでのデータ競合の問題を修正する[#6206](https://github.com/pingcap/tiflow/issues/6206) @[asddongmen](https://github.com/asddongmen)
      - `enable-old-value = false`を設定した場合のTiCDCのパニック問題を修正する[#6198](https://github.com/pingcap/tiflow/issues/6198) @[hi-rustin](https://github.com/hi-rustin)
      - redoログ機能が有効な場合のデータ整合性の問題を修正する[#6189](https://github.com/pingcap/tiflow/issues/6189) [#6368](https://github.com/pingcap/tiflow/issues/6368) [#6277](https://github.com/pingcap/tiflow/issues/6277) [#6456](https://github.com/pingcap/tiflow/issues/6456) [#6695](https://github.com/pingcap/tiflow/issues/6695) [#6764](https://github.com/pingcap/tiflow/issues/6764) [#6859](https://github.com/pingcap/tiflow/issues/6859) @[asddongmen](https://github.com/asddongmen)
      - redoイベントを非同期で書き込むことによるredoログのパフォーマンスの問題を修正する[#6011](https://github.com/pingcap/tiflow/issues/6011) @[CharlesCheung96](https://github.com/CharlesCheung96)
      - MySQL sinkがIPv6アドレスに接続できない問題を修正する[#6135](https://github.com/pingcap/tiflow/issues/6135) @[hi-rustin](https://github.com/hi-rustin)

    + バックアップとリストア（BR）

      - RawKVモードでBRが`ErrRestoreTableIDMismatch`を報告するバグを修正する[#35279](https://github.com/pingcap/tidb/issues/35279) @[3pointer](https://github.com/3pointer)
      - 大規模クラスタのバックアップでS3のレート制限によるバックアップ失敗を修正するためにバックアップデータディレクトリ構造を調整する[#30087](https://github.com/pingcap/tidb/issues/30087) @[MoCuishle28](https://github.com/MoCuishle28)
      - サマリーログ内の不正確なバックアップ時間を修正する[#35553](https://github.com/pingcap/tidb/issues/35553) @[ixuh12](https://github.com/ixuh12)

    + Dumpling

      - GetDSNがIPv6をサポートしていない問題を修正する[#36112](https://github.com/pingcap/tidb/issues/36112) @[D3Hunter](https://github.com/D3Hunter)

    + TiDB Binlog

      - `compressor`を`gzip`に設定した場合に、DrainerがPumpに正しくリクエストを送信できないバグを修正する[#1152](https://github.com/pingcap/tidb-binlog/issues/1152) @[lichunzhu](https://github.com/lichunzhu)