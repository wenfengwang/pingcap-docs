---
title: TiDB 4.0.6 リリースノート
---

# TiDB 4.0.6 リリースノート

リリース日: 2020年9月15日

TiDB バージョン: 4.0.6

## 新機能

+ TiFlash

    - TiFlashブロードキャスト結合でのアウタージョインサポート

+ TiDB ダッシュボード

    - クエリエディターおよび実行UIを追加（実験的）[#713](https://github.com/pingcap-incubator/tidb-dashboard/pull/713)
    - ストアの場所トポロジーの視覚化をサポート [#719](https://github.com/pingcap-incubator/tidb-dashboard/pull/719)
    - クラスタ構成UIを追加（実験的） [#733](https://github.com/pingcap-incubator/tidb-dashboard/pull/733)
    - 現在のセッションを共有するサポート [#741](https://github.com/pingcap-incubator/tidb-dashboard/pull/741)
    - SQLステートメントリストに実行計画の数を表示するサポート [#746](https://github.com/pingcap-incubator/tidb-dashboard/pull/746)

+ ツール

    + TiCDC (v4.0.6 でGA)

        - `maxwell`形式でデータを出力するサポート [#869](https://github.com/pingcap/tiflow/pull/869)

## 改善点

+ TiDB

    - エラーコードとメッセージを標準エラーに置き換える [#19888](https://github.com/pingcap/tidb/pull/19888)
    - パーティション化されたテーブルの書き込みパフォーマンスを改善する [#19649](https://github.com/pingcap/tidb/pull/19649)
    - `Cop Runtime`統計情報でより多くのRPCランタイム情報を記録する [#19264](https://github.com/pingcap/tidb/pull/19264)
    - `metrics_schema`および`performance_schema`でのテーブル作成を禁止するサポート [#19792](https://github.com/pingcap/tidb/pull/19792)
    - union executorの並行性を調整するサポート [#19886](https://github.com/pingcap/tidb/pull/19886)
    - ブロードキャスト結合でのアウタージョインをサポート [#19664](https://github.com/pingcap/tidb/pull/19664)
    - プロセスリストのためのSQLダイジェストを追加するサポート [#19829](https://github.com/pingcap/tidb/pull/19829)
    - オートコミットステートメントのリトライに対して悲観的トランザクションモードに切り替えるサポート [#19796](https://github.com/pingcap/tidb/pull/19796)
    - `Str_to_date()`で`%r`および`%T`データフォーマットをサポートする [#19693](https://github.com/pingcap/tidb/pull/19693)
    - `SELECT INTO OUTFILE`を使用するためのファイル権限を要求するサポート [#19577](https://github.com/pingcap/tidb/pull/19577)
    - `stddev_pop`関数をサポートする [#19541](https://github.com/pingcap/tidb/pull/19541)
    - `TiDB-Runtime`ダッシュボードを追加する [#19396](https://github.com/pingcap/tidb/pull/19396)
    - `ALTER TABLE`アルゴリズムの互換性を改善する [#19364](https://github.com/pingcap/tidb/pull/19364)
    - スローログの`plan`フィールドに`insert`/`delete`/`update`プランをエンコードする [#19269](https://github.com/pingcap/tidb/pull/19269)

+ TiKV

    - `DropTable`または`TruncateTable`が実行されているときのQPS低下を軽減する [#8627](https://github.com/tikv/tikv/pull/8627)
    - エラーコードのメタファイルの生成をサポートする [#8619](https://github.com/tikv/tikv/pull/8619)
    - cfスキャンの詳細に対するパフォーマンス統計を追加するサポート [#8618](https://github.com/tikv/tikv/pull/8618)
    - Grafanaデフォルトテンプレートで`rocksdb perf context`パネルを追加する [#8467](https://github.com/tikv/tikv/pull/8467)

+ PD

    - TiDBダッシュボードをv2020.09.08.1に更新する [#2928](https://github.com/pingcap/pd/pull/2928)
    - リージョンおよびストアハートビートのための追加のメトリクスを追加する [#2891](https://github.com/tikv/pd/pull/2891)
    - 低スペース閾値を制御するための元の方法に戻すサポートを追加する [#2875](https://github.com/pingcap/pd/pull/2875)
    - 標準エラーコードのサポート
        - [#2918](https://github.com/tikv/pd/pull/2918) [#2911](https://github.com/tikv/pd/pull/2911) [#2913](https://github.com/tikv/pd/pull/2913) [#2915](https://github.com/tikv/pd/pull/2915) [#2912](https://github.com/tikv/pd/pull/2912)
        - [#2907](https://github.com/tikv/pd/pull/2907) [#2906](https://github.com/tikv/pd/pull/2906) [#2903](https://github.com/tikv/pd/pull/2903) [#2806](https://github.com/tikv/pd/pull/2806) [#2900](https://github.com/tikv/pd/pull/2900) [#2902](https://github.com/tikv/pd/pull/2902)

+ TiFlash

    - データ複製（`apply Region snapshots`および`ingest SST files`）のためのGrafanaパネルを追加する
    - `write stall`のためのGrafanaパネルを追加する
    - `dt_segment_force_merge_delta_rows`および`dt_segment_force_merge_delta_deletes`を追加して`write stall`の閾値を調整する
    - TiFlash-Proxyで`raftstore.snap-handle-pool-size`を`0`に設定して、データ複製中のメモリ消費を削減するために複数スレッドでRegionスナップショットの適用を無効にするサポート
    - `https_port`および`metrics_port`でのCNチェックをサポートする

+ ツール

    + TiCDC

        - pullerの初期化中に解決済みロックをスキップするサポート [#910](https://github.com/pingcap/tiflow/pull/910)
        - PDへの書き込み頻度を減らすサポート [#937](https://github.com/pingcap/tiflow/pull/937)

    + バックアップ＆リストア（BR）

        - サマリーログにリアルタイムコストを追加する [#486](https://github.com/pingcap/br/issues/486)

    + Dumpling

        - カラム名付きで`INSERT`を出力するサポート [#135](https://github.com/pingcap/dumpling/pull/135)
        - `--filesize`および`--statement-size`の定義をmydumperと統一するサポート [#142](https://github.com/pingcap/dumpling/pull/142)

    + TiDB Lightning

        - より正確なサイズでリージョンを分割し取り込むサポート [#369](https://github.com/pingcap/tidb-lightning/pull/369)

    + TiDB Binlog

        - `go time`パッケージ形式でGC時間を設定するサポート [#996](https://github.com/pingcap/tidb-binlog/pull/996)

## バグ修正

+ TiDB

    - メトリックプロファイルでの`tikv_cop_wait`時間の収集に関する問題を修正する [#19881](https://github.com/pingcap/tidb/pull/19881)
    - `SHOW GRANTS`の誤った結果を修正する [#19834](https://github.com/pingcap/tidb/pull/19834)
    - `!= ALL (subq)`の不正確なクエリ結果を修正する [#19831](https://github.com/pingcap/tidb/pull/19831)
    - `enum`および`set`タイプの変換のバグを修正する [#19778](https://github.com/pingcap/tidb/pull/19778)
    - `SHOW STATS_META`および`SHOW STATS_BUCKET`の権限チェックを追加する [#19760](https://github.com/pingcap/tidb/pull/19760)
    - `builtinGreatestStringSig`および`builtinLeastStringSig`による不一致した列長のエラーを修正する [#19758](https://github.com/pingcap/tidb/pull/19758)
    - 不要なエラーまたは警告が発生した場合、ベクトル化制御式をスカラー実行にフォールバックする [#19749](https://github.com/pingcap/tidb/pull/19749)
    - 相関カラムのタイプが`Bit`の場合の`Apply`演算子のエラーを修正する [#19692](https://github.com/pingcap/tidb/pull/19692)
    - MySQL 8.0クライアントで`processlist`および`cluster_log`をクエリするときに発生する問題を修正する [#19690](https://github.com/pingcap/tidb/pull/19690)
    - 同じタイプのプランに異なるプランダイジェストがある問題を修正する [#19684](https://github.com/pingcap/tidb/pull/19684)
    - `Decimal`から`Int`への列タイプの変更を禁止する [#19682](https://github.com/pingcap/tidb/pull/19682)
- `SELECT ... INTO OUTFILE` のランタイムエラーが返される問題を修正しました [#19672](https://github.com/pingcap/tidb/pull/19672)
    - `builtinRealIsFalseSig` の誤った実装を修正しました [#19670](https://github.com/pingcap/tidb/pull/19670)
    - パーティション式のチェックで括弧式を見落とす問題を修正しました [#19614](https://github.com/pingcap/tidb/pull/19614)
    - `HashJoin` 上の `Apply` 演算子によるクエリエラーを修正しました [#19611](https://github.com/pingcap/tidb/pull/19611)
    - `Real` を `Time` としてキャストした際の不正な結果を修正しました [#19594](https://github.com/pingcap/tidb/pull/19594)
    - `SHOW GRANTS` 文が存在しないユーザーの権限を表示するバグを修正しました [#19588](https://github.com/pingcap/tidb/pull/19588)
    - `IndexLookupJoin` 上の `Apply` 実行時のクエリエラーを修正しました [#19566](https://github.com/pingcap/tidb/pull/19566)
    - パーティションテーブルにおける `Apply` の `HashJoin` への変換時の誤った結果を修正しました [#19546](https://github.com/pingcap/tidb/pull/19546)
    - `Apply` の内部に `IndexLookUp` 実行時の不正な結果を修正しました [#19508](https://github.com/pingcap/tidb/pull/19508)
    - ビューを使用した際の予期しないパニックを修正しました [#19491](https://github.com/pingcap/tidb/pull/19491)
    - `anti-semi-join` クエリの不正な結果を修正しました [#19477](https://github.com/pingcap/tidb/pull/19477)
    - 統計が削除されない `TopN` 統計のバグを修正しました [#19465](https://github.com/pingcap/tidb/pull/19465)
    - バッチポイント取得の誤った使用による誤った結果を修正しました [#19460](https://github.com/pingcap/tidb/pull/19460)
    - 仮想列の生成された列を持つ `indexLookupJoin` で列が見つからないバグを修正しました [#19439](https://github.com/pingcap/tidb/pull/19439)
    - `select` と `update` クエリの異なるプランが `datum` を比較した際の誤った挙動を修正しました [#19403](https://github.com/pingcap/tidb/pull/19403)
    - リージョンキャッシュ内の TiFlash ワークインデックスにおけるデータ競合を修正しました [#19362](https://github.com/pingcap/tidb/pull/19362)
    - `logarithm` 関数が警告を表示しないバグを修正しました [#19291](https://github.com/pingcap/tidb/pull/19291)
    - TiDB がデータをディスクに永続化する際に発生する予期しないエラーを修正しました [#19272](https://github.com/pingcap/tidb/pull/19272)
    - `index join` 内部の単一のパーティションテーブルの使用をサポートしました [#19197](https://github.com/pingcap/tidb/pull/19197)
    - 10 進数に対して生成された間違ったハッシュキー値を修正しました [#19188](https://github.com/pingcap/tidb/pull/19188)
    - テーブルの `endKey` およびリージョンの `endKey` が同じ場合に TiDB が `no regions` エラーを返す問題を修正しました [#19895](https://github.com/pingcap/tidb/pull/19895)
    - `alter partition` の予期しない成功を修正しました [#19891](https://github.com/pingcap/tidb/pull/19891)
    - プッシュダウン式に対して許可されるデフォルトの最大パケット長の値を修正しました [#19876](https://github.com/pingcap/tidb/pull/19876)
    - `ENUM`/`SET` 列に対する `Max`/`Min` 関数の誤った挙動を修正しました [#19869](https://github.com/pingcap/tidb/pull/19869)
    - `tiflash_segments` および `tiflash_tables` システムテーブルからの読み取り失敗を修正しました [#19748](https://github.com/pingcap/tidb/pull/19748)
    - `Count(col)` 集約関数の間違った結果を修正しました [#19628](https://github.com/pingcap/tidb/pull/19628)
    - `TRUNCATE` 操作のランタイムエラーを修正しました [#19445](https://github.com/pingcap/tidb/pull/19445)
    - `PREPARE statement FROM @Var` において `Var` に大文字が含まれる場合に失敗する問題を修正しました [#19378](https://github.com/pingcap/tidb/pull/19378)
    - 大文字スキーマでのスキーマ文字セットの変更がパニックを引き起こすバグを修正しました [#19302](https://github.com/pingcap/tidb/pull/19302)
    - `information_schema.statements_summary` と `explain` のプランの不一致を修正しました（情報が `tikv/tiflash` を含んでいる場合） [#19159](https://github.com/pingcap/tidb/pull/19159)
    - `select into outfile` でファイルが存在しないテスト内のエラーを修正しました [#19725](https://github.com/pingcap/tidb/pull/19725)
    - `INFORMATION_SCHEMA.CLUSTER_HARDWARE` が RAID デバイス情報を持たない問題を修正しました [#19457](https://github.com/pingcap/tidb/pull/19457)
    - `add index` 操作が `case-when` 式で生成された列を持つ場合にパースエラーが発生した際に正常に終了できるようにしました [#19395](https://github.com/pingcap/tidb/pull/19395)
    - DDL 操作のリトライに時間がかかりすぎるバグを修正しました [#19488](https://github.com/pingcap/tidb/pull/19488)
    - `alter table db.t1 add constraint fk foreign key (c2) references t2(c1)` の実行時に最初に `use db` を実行せずに実行できるようにしました [#19471](https://github.com/pingcap/tidb/pull/19471)
    - サーバログファイルの `Error` から `Info` メッセージへのディスパッチエラーを修正しました [#19454](https://github.com/pingcap/tidb/pull/19454)

+ TiKV

    - ソートが有効になっている場合に非インデックス列の見積もりエラーを修正しました [#8620](https://github.com/tikv/tikv/pull/8620)
    - リージョン転送のプロセス中に Green GC がロックを見落とす可能性がある問題を修正しました [#8460](https://github.com/tikv/tikv/pull/8460)
    - Raft メンバーシップの変更中に TiKV が非常に遅い場合に発生するパニックの問題を修正しました [#8497](https://github.com/tikv/tikv/pull/8497)
    - PD クライアントスレッドと他のスレッドの間で発生するデッドロックの問題を修正しました [#8612](https://github.com/tikv/tikv/pull/8612)
    - ヒュージページでのメモリ割り当ての問題を解消するために jemalloc を v5.2.1 にアップグレードしました [#8463](https://github.com/tikv/tikv/pull/8463)
    - 長時間実行されるクエリに対して統一スレッドプールが停止する問題を解消しました [#8427](https://github.com/tikv/tikv/pull/8427)

+ PD

    - ブートストラップ中に異なるクラスタが相互に通信しないようにするために `initial-cluster-token` 構成を追加しました [#2922](https://github.com/pingcap/pd/pull/2922)
    - モードが `auto` の場合にストア制限率の単位を修正しました [#2826](https://github.com/pingcap/pd/pull/2826)
    - いくつかのスケジューラがエラーを解決せずに構成を永続化する問題を修正しました [#2818](https://github.com/tikv/pd/pull/2818)
    - スケジューラにおける空の HTTP レスポンスを修正しました [#2871](https://github.com/tikv/pd/pull/2871) [#2874](https://github.com/tikv/pd/pull/2874)

+ TiFlash

    - 以前のバージョンで主キーカラムの名前を変更した後、TiFlash が v4.0.4/v4.0.5 にアップグレードした後に起動できなくなる問題を修正しました
    - 列の `nullable` 属性を変更した後に発生する例外を修正しました
    - テーブルの複製ステータスを計算する際に発生するクラッシュを修正しました
    - サポートされていない DDL 操作を適用した後に TiFlash がデータ読み取りに利用できなくなる問題を修正しました
    - `utf8mb4_bin` として扱われるサポートされていない照合を修正しました
    - TiFlash コプロセッサエグゼキュータの QPS パネルが常に `0` を表示する問題を修正しました
    - `NULL` を入力とした場合に `FROM_UNIXTIME` 関数の間違った結果を修正しました

+ Tools

    + TiCDC

        - いくつかの場合において TiCDC がメモリリークする問題を修正しました [#942](https://github.com/pingcap/tiflow/pull/942)
        - TiCDC が Kafka Sink でパニックする可能性がある問題を修正しました [#912](https://github.com/pingcap/tiflow/pull/912)
        - CommitTsまたはResolvedTs（CRTs）がpuller [#927](https://github.com/pingcap/tiflow/pull/927)で`resolvedTs`よりも小さい場合の問題を修正
        - `changefeed`がMySQL driver [#936](https://github.com/pingcap/tiflow/pull/936)によってブロックされる可能性がある問題を修正
        - TiCDCのResolved Ts間隔が正しくない問題を修正 [#8573](https://github.com/tikv/tikv/pull/8573)

    + バックアップとリストア（BR）

        - チェックサム中に発生する可能性があるパニックを修正 [#479](https://github.com/pingcap/br/pull/479)
        - PDリーダーの変更後に発生する可能性があるパニックを修正 [#496](https://github.com/pingcap/br/pull/496)

    + Dumpling

        - バイナリタイプの`NULL`値が適切に処理されない問題を修正 [#137](https://github.com/pingcap/dumpling/pull/137)

    + TiDB Lightning

        - すべての失敗した書き込みとインジェストの操作が誤って成功した操作として表示される問題を修正 [#381](https://github.com/pingcap/tidb-lightning/pull/381)
        - TiDB Lightningが終了する前にいくつかのチェックポイント更新がデータベースに書き込まれない可能性がある問題を修正 [#386](https://github.com/pingcap/tidb-lightning/pull/386)