---
title: TiDB 5.0.4 リリースノート
---

# TiDB 5.0.4 リリースノート

リリース日: 2021年9月27日

TiDB バージョン: 5.0.4

## 互換性の変更

+ TiDB

    - 新しいセッションで `SHOW VARIABLES` を実行すると遅い問題を修正しました。この修正により、[#19341](https://github.com/pingcap/tidb/pull/19341) で行われた一部の変更が取り消され、互換性の問題が発生する可能性があります。[#24326](https://github.com/pingcap/tidb/issues/24326)
    - `tidb_stmt_summary_max_stmt_count` 変数のデフォルト値を `200` から `3000` に変更しました [#25873](https://github.com/pingcap/tidb/pull/25873)
    + 以下のバグ修正により、実行結果が変わる可能性があり、アップグレードの非互換性が生じるかもしれません：
        - `UNION` の子に `NULL` 値が含まれる場合、TiDB が誤った結果を返す問題を修正しました [#26559](https://github.com/pingcap/tidb/issues/26559)
        - `greatest(datetime) union null` が空の文字列を返す問題を修正しました [#26532](https://github.com/pingcap/tidb/issues/26532)
        - `SQL MODE` で `last_day` 関数の動作が互換性のない問題を修正しました [#26000](https://github.com/pingcap/tidb/pull/26000)
        - `having` 句が正しく機能しない可能性がある問題を修正しました [#26496](https://github.com/pingcap/tidb/issues/26496)
        - `between` 式周辺の照合順序が異なる場合に、誤った実行結果が発生する問題を修正しました [#27146](https://github.com/pingcap/tidb/issues/27146)
        - `group_concat` 関数の列が非バイナリ照合順序を持つ場合に、誤った実行結果が発生する問題を修正しました [#27429](https://github.com/pingcap/tidb/issues/27429)
        - 新しい照合順序が有効になっている場合、複数の列に `count(distinct)` 式を使用すると誤った結果が返る問題を修正しました [#27091](https://github.com/pingcap/tidb/issues/27091)
        - `extract` 関数の引数が負の期間の場合に発生する誤った結果を修正しました [#27236](https://github.com/pingcap/tidb/issues/27236)
        - `SQL_MODE` が 'STRICT_TRANS_TABLES' の場合、無効な日付を挿入してもエラーを報告しない問題を修正しました [#26762](https://github.com/pingcap/tidb/issues/26762)
        - `SQL_MODE` が 'NO_ZERO_IN_DATE' の場合、無効なデフォルト日付を使用してもエラーを報告しない問題を修正しました [#26766](https://github.com/pingcap/tidb/issues/26766)
        - 接頭辞インデックスのクエリ範囲に関するバグを修正しました [#26029](https://github.com/pingcap/tidb/issues/26029)
        - `LOAD DATA` ステートメントが非UTF-8データを異常に取り込む場合の問題を修正しました [#25979](https://github.com/pingcap/tidb/issues/25979)
        - 二次インデックスがプライマリキーと同じ列を持つ場合、`insert ignore on duplicate update` が誤ったデータを挿入する問題を修正しました [#25809](https://github.com/pingcap/tidb/issues/25809)
        - パーティション化されたテーブルにクラスタ化インデックスがある場合、`insert ignore duplicate update` が誤ったデータを挿入する問題を修正しました [#25846](https://github.com/pingcap/tidb/issues/25846)
        - キーが `ENUM` 型の場合に、`POINT GET` または `BATCH POINT GET` でクエリ結果が誤る問題を修正しました [#24562](https://github.com/pingcap/tidb/issues/24562)
        - `BIT` 型の値を除算する場合に、誤った結果が発生する問題を修正しました [#23479](https://github.com/pingcap/tidb/issues/23479)
        - `prepared` ステートメントと直接クエリの結果が一貫性がない可能性がある問題を修正しました [#22949](https://github.com/pingcap/tidb/issues/22949)
        - `YEAR` 型が文字列または整数型と比較される場合に、クエリ結果が誤る可能性がある問題を修正しました [#23262](https://github.com/pingcap/tidb/issues/23262)

## 機能強化

+ TiDB

    - `tidb_enforce_mpp=1` を設定して、オプティマイザの推定を無視して強制的に MPP モードを使用できるようにサポートを追加しました [#26382](https://github.com/pingcap/tidb/pull/26382)

+ TiKV

    - TiCDC 設定を動的に変更できるようにサポートを追加しました [#10645](https://github.com/tikv/tikv/issues/10645)

+ PD

    - TiDB ダッシュボードの OIDC ベースの SSO サポートを追加しました [#3884](https://github.com/tikv/pd/pull/3884)

+ TiFlash

    - DAG リクエストで `HAVING()` 関数をサポート
    - `DATE()` 関数をサポート
    - インスタンスごとの書き込みスループットのための Grafana パネルを追加

## 改善

+ TiDB

    - ヒストグラム行数に基づいて自動分析をトリガーするように変更しました [#24237](https://github.com/pingcap/tidb/issues/24237)
    - TiFlash ノードに一定期間リクエストを送信しないようにし、ノードが失敗して再起動した場合に [#26757](https://github.com/pingcap/tidb/pull/26757)
    - `split table` および `presplit` を安定化するために `split region` の上限を増やしました [#26657](https://github.com/pingcap/tidb/pull/26657)
    - MPP クエリのリトライをサポートしました [#26483](https://github.com/pingcap/tidb/pull/26483)
    - MPP クエリを開始する前に、TiFlash の可用性をチェックするサポートを追加しました [#1807](https://github.com/pingcap/tics/issues/1807)
    - クエリ結果をより安定させるために、安定結果モードをサポートしました [#26084](https://github.com/pingcap/tidb/pull/26084)
    - MySQL システム変数 `init_connect` およびそれに関連する機能をサポートしました [#18894](https://github.com/pingcap/tidb/issues/18894)
    - MPP モードで `COUNT(DISTINCT)` 集約関数を徹底的に押し下げました [#25861](https://github.com/pingcap/tidb/pull/25861)
    - `EXPLAIN` 文で集約関数を押し下げられない場合にログ警告を出力する機能を追加しました [#25736](https://github.com/pingcap/tidb/pull/25736)
    - Grafana ダッシュボードの `TiFlashQueryTotalCounter` にエラーラベルを追加しました [#25327](https://github.com/pingcap/tidb/pull/25327)
    - HTTP API を使用してクラスタ化インデックステーブルのセカンダリインデックス経由で MVCC データを取得するサポートを追加しました [#24209](https://github.com/pingcap/tidb/issues/24209)
    - パーサーでの `prepared` ステートメントのメモリ割り当てを最適化しました [#24371](https://github.com/pingcap/tidb/pull/24371)

+ TiKV

    - 読み込み準備完了と書き込み準備完了を別々に処理して、読み込み遅延を減らしました [#10475](https://github.com/tikv/tikv/issues/10475)
    - ネットワーク帯域を節約するために、Resolved TS メッセージのサイズを減らしました [#2448](https://github.com/pingcap/tiflow/issues/2448)
    - スローガーのスレッドが過負荷になりキューが満タンになった場合、ログを破棄してスレッドをブロックしないようにしました [#10841](https://github.com/tikv/tikv/issues/10841)
    - TiKV 共同処理の遅いログは、リクエスト処理にかかる時間のみを考慮するようにしました [#10841](https://github.com/tikv/tikv/issues/10841)
    - 不確定なエラーの発生確率を減らすために、可能な限り prewrite を冪等性として実行するようにしました [#10587](https://github.com/tikv/tikv/pull/10587)
    - 低い書き込みフローの場合に誤った「GC が動作できない」アラートを回避するようにしました [#10662](https://github.com/tikv/tikv/pull/10662)
    - バックアップ時に復元されるデータベースが常に元のクラスタサイズに一致するようにしました [#10643](https://github.com/tikv/tikv/pull/10643)
    - パニック出力がログにフラッシュされることを保証しました [#9955](https://github.com/tikv/tikv/pull/9955)

+ PD

    - PD 間でのリージョン情報の同期のパフォーマンスを改善しました [#3993](https://github.com/tikv/pd/pull/3993)

+ ツール

    + Dumpling

        - `START TRANSACTION ... WITH CONSISTENT SNAPSHOT` や `SHOW CREATE TABLE` 構文をサポートしない MySQL 互換のデータベースのバックアップをサポートしました [#309](https://github.com/pingcap/dumpling/issues/309)

    + TiCDC

        - Unified Sorter がメモリを使用してソートする際のメモリ管理を最適化しました [#2553](https://github.com/pingcap/tiflow/issues/2553)
        - メジャーまたはマイナーバージョンをまたいで TiCDC クラスタを操作することを禁止しました [#2598](https://github.com/pingcap/tiflow/pull/2598)
- {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
  - {R}
# The technical documentation has been translated into Japanese as follows:

- TiDB

    - パーティション化されたテーブルをクエリしたときに、パーティションキーに `IS NULL` 条件がある場合、TiDBがパニックする可能性がある問題を修正しました [#23802](https://github.com/pingcap/tidb/issues/23802)
    - `FLOAT64` 型のオーバーフローチェックが MySQL のものと異なる問題を修正しました [#23897](https://github.com/pingcap/tidb/issues/23897)
    - `case when` 式の文字セットと照合順序が誤っていた問題を修正しました [#26662](https://github.com/pingcap/tidb/issues/26662)
    - 悲観的トランザクションのコミットが書き込み競合を引き起こす可能性がある問題を修正しました [#25964](https://github.com/pingcap/tidb/issues/25964)
    - 悲観的トランザクションでインデックスキーが繰り返しコミットされる可能性があるバグを修正しました [#26359](https://github.com/pingcap/tidb/issues/26359) [#10600](https://github.com/tikv/tikv/pull/10600)
    - 非同期コミットロックを解決する際にTiDBがパニックする可能性がある問題を修正しました [#25778](https://github.com/pingcap/tidb/issues/25778)
    - `INDEX MERGE` を使用した場合に列が見つからない可能性があるバグを修正しました [#25045](https://github.com/pingcap/tidb/issues/25045)
    - `ALTER USER REQUIRE SSL` がユーザーの `authentication_string` をクリアする問題を修正しました [#25225](https://github.com/pingcap/tidb/issues/25225)
    - 新しいクラスタで `tidb_gc_scan_lock_mode` グローバル変数の値が実際のデフォルトモード "LEGACY" ではなく "PHYSICAL" と表示される問題を修正しました [#25100](https://github.com/pingcap/tidb/issues/25100)
    - `TIKV_REGION_PEERS` システムテーブルが正しい `DOWN` ステータスを表示しない問題を修正しました [#24879](https://github.com/pingcap/tidb/issues/24879)
    - HTTP APIを使用した際のメモリリークの問題を修正しました [#24649](https://github.com/pingcap/tidb/pull/24649)
    - ビューが `DEFINER` をサポートしていない問題を修正しました [#24414](https://github.com/pingcap/tidb/issues/24414)
    - `tidb-server --help` を実行するとコード `2` で終了する問題を修正しました [#24046](https://github.com/pingcap/tidb/issues/24046)
    - グローバル変数 `dml_batch_size` を設定しても効果がない問題を修正しました [#24709](https://github.com/pingcap/tidb/issues/24709)
    - `read_from_storage` とパーティション化されたテーブルを同時に使用するとエラーが発生する問題を修正しました [#20372](https://github.com/pingcap/tidb/issues/20372)
    - プロジェクション演算子を実行した際にTiDBがパニックする可能性がある問題を修正しました [#24264](https://github.com/pingcap/tidb/issues/24264)
    - 統計情報がクエリをパニックさせる可能性がある問題を修正しました [#24061](https://github.com/pingcap/tidb/pull/24061)
    - `BIT` 列で `approx_percentile` 関数を使用するとTiDBがパニックする可能性がある問題を修正しました [#23662](https://github.com/pingcap/tidb/issues/23662)
    - Grafanaの**Coprocessor Cache**パネルのメトリクスが正しくない問題を修正しました [#26338](https://github.com/pingcap/tidb/issues/26338)
    - 同じパーティションを同時に切り捨てるとDDLステートメントがスタックする問題を修正しました [#26229](https://github.com/pingcap/tidb/issues/26229)
    - セッション変数を `GROUP BY` 項目として使用した場合に誤ったクエリ結果が発生する問題を修正しました [#27106](https://github.com/pingcap/tidb/issues/27106)
    - `VARCHAR` とタイムスタンプの間の誤った暗黙の変換を修正しました [#25902](https://github.com/pingcap/tidb/issues/25902)
    - 関連するサブクエリステートメントの誤った結果を修正しました [#27233](https://github.com/pingcap/tidb/issues/27233)

- TiKV

    - 破損したスナップショットファイルによる潜在的なディスクフルの問題を修正しました [#10813](https://github.com/tikv/tikv/issues/10813)
    - TiKVがTitanを有効にした状態から5.0未満のバージョンからアップグレードするときに発生するTiKVパニックの問題を修正しました [#10843](https://github.com/tikv/tikv/pull/10843)
    - 新しいバージョンのTiKVをv5.0.xにロールバックできない問題を修正しました [#10843](https://github.com/tikv/tikv/pull/10843)
    - 5.0以降のTiKVにアップグレードするときに、古いバージョンのTiKV v3.xからアップグレードしたクラスタが問題に遭遇する可能性がある問題を修正しました [#10774](https://github.com/tikv/tikv/issues/10774)
    - 左の悲観的ロックによる解析の失敗を修正しました [#26404](https://github.com/pingcap/tidb/issues/26404)
    - 特定のプラットフォームでのデュレーションの計算時に発生するパニックを修正しました [#10571](https://github.com/tikv/tikv/pull/10571)
    - Load Base Splitの`batch_get_command`のキーがエンコードされていない問題を修正しました [#10542](https://github.com/tikv/tikv/issues/10542)

- PD

    - PDがダウンしたペアを適切なタイミングで修復しない問題を修正しました [#4077](https://github.com/tikv/pd/issues/4077)
    - `replication.max-replicas` が更新された後もデフォルトの配置ルールのレプリカ数が一定のままになる問題を修正しました [#3886](https://github.com/tikv/pd/issues/3886)
    - TiKVをスケールアウトするときにPDがパニックする可能性がある問題を修正しました [#3868](https://github.com/tikv/pd/issues/3868)
    - 複数のスケジューラが同時に実行されているとスケジューリングの競合が発生する問題を修正しました [#3807](https://github.com/tikv/pd/issues/3807)
    - 削除したはずのスケジューラが再度表示される問題を修正しました [#2572](https://github.com/tikv/pd/issues/2572)

- TiFlash

    - テーブルスキャンタスクを実行する際に発生する潜在的なパニックの問題を修正しました
    - MPPタスクを実行する際にメモリリークが発生する可能性がある問題を修正しました
    - DAQリクエストを処理する際にTiFlashが `duplicated region` エラーを発生させるバグを修正しました
    - 集約関数 `COUNT` や `COUNT DISTINCT` を実行する際に予期しない結果が表示される問題を修正しました
    - MPPタスクを実行する際に潜在的なパニックの問題を修正しました
    - 複数のディスクに展開された場合、TiFlashがデータを復元できない可能性がある問題を修正しました
    - `SharedQueryBlockInputStream` を分解する際に潜在的なパニックの問題を修正しました
    - `MPPTask` を分解する際に潜在的なパニックの問題を修正しました
    - MPP接続の確立に失敗したときに予期しない結果が表示される問題を修正しました
    - ロックを解決する際に潜在的なパニックの問題を修正しました
    - 重い書き込み時にメトリクスのストアサイズが正確でない問題を修正しました
    - `CONSTANT`、`<`、`<=`、`>`、`>=`、または `COLUMN` のようなフィルタを含むクエリで不正確な結果が表示される問題を修正しました
    - 長時間実行した後にDeltaデータをガベージコレクトできない可能性がある問題を修正しました
    - メトリクスが間違った値を表示する可能性がある問題を修正しました
    - 複数のディスクに展開された場合のデータの不統一性の可能性がある問題を修正しました

- Tools

    + Dumpling

        - MySQL 8.0.3以降で `show table status` の実行がスタックする問題を修正しました [#322](https://github.com/pingcap/dumpling/issues/322)

    + TiCDC

        - `mysql.TypeString`、`mysql.TypeVarString`、`mysql.TypeVarchar` などのデータ型をJSONにエンコードする際にプロセスがパニックする問題を修正しました [#2758](https://github.com/pingcap/tiflow/issues/2758)
- 同じテーブルに複数のプロセッサがデータを書き込もうとすることで発生するデータ不整合の問題を修正する [#2417](https://github.com/pingcap/tiflow/pull/2417)
        - TiCDCがあまりにも多くの領域をキャプチャするとOOMが発生するため、gRPCウィンドウサイズを減らす
[#2724](https://github.com/pingcap/tiflow/pull/2724)
        - メモリ圧力が高いとgRPC接続が頻繁に切断されるエラーを修正する
[#2202](https://github.com/pingcap/tiflow/issues/2202)
        - 符号なしの`TINYINT`型でTiCDCがパニックを引き起こすバグを修正する [#2648](https://github.com/pingcap/tiflow/issues/2648)
        - Upstreamでトランザクションの挿入と同じ行のデータを削除する際にTiCDC Open Protocolが空の値を出力する問題を修正する
[#2612](https://github.com/pingcap/tiflow/issues/2612)
        - Changefeedがスキーマ変更の終了TSで開始するとDDL処理が失敗するバグを修正する [#2603](https://github.com/pingcap/tiflow/issues/2603)
        - 古いオーナーで非応答のダウンストリームがタイムアウトするまでレプリケーションタスクを中断する問題を修正する
[#2295](https://github.com/pingcap/tiflow/issues/2295)
        - メタデータ管理のバグを修正する [#2558](https://github.com/pingcap/tiflow/pull/2558)
        - TiCDCオーナー切り替え後に発生するデータ不整合の問題を修正する
[#2230](https://github.com/pingcap/tiflow/issues/2230)
        - `capture list`コマンドの出力に古いキャプチャが表示される問題を修正する
[#2388](https://github.com/pingcap/tiflow/issues/2388)
        - 統合テストでDDLジョブの重複が発生した場合に発生する`ErrSchemaStorageTableMiss`エラーを修正する
[#2422](https://github.com/pingcap/tiflow/issues/2422)
        - `ErrGCTTLExceeded`エラーが発生した場合にchangefeedを削除できないバグを修正する [#2391](https://github.com/pingcap/tiflow/issues/2391)
        - 大きなテーブルをcdclogにレプリケートできないバグを修正する
[#1259](https://github.com/pingcap/tiflow/issues/1259) [#2424](https://github.com/pingcap/tiflow/issues/2424)
        - CLIの後方互換性の問題を修正する [#2373](https://github.com/pingcap/tiflow/issues/2373)
        - `SinkManager`のマップへの不安全な同時アクセスの問題を修正する
[#2299](https://github.com/pingcap/tiflow/pull/2299)
        - オーナーがDDLステートメントを実行中にクラッシュした際の潜在的なDDL損失の問題を修正する [#1260](https://github.com/pingcap/tiflow/issues/1260)
        - リージョンが初期化された直後にロックがすぐ解除される問題を修正する
[#2188](https://github.com/pingcap/tiflow/issues/2188)
        - パーティションテーブルを追加する際に追加のパーティションディスパッチが発生する問題を修正する
[#2263](https://github.com/pingcap/tiflow/pull/2263)
        - 削除されたchangefeedについてTiCDCが繰り返し警告を出す問題を修正する
[#2156](https://github.com/pingcap/tiflow/issues/2156)