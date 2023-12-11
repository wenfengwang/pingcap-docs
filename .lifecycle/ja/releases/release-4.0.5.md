---
title: TiDB 4.0.5 リリースノート
---

# TiDB 4.0.5 リリースノート

リリース日: 2020年8月31日

TiDB バージョン: 4.0.5

## 互換性の変更

+ TiDB

    - `drop partition` および `truncate partition` のジョブ引数を複数のパーティションのID配列をサポートするように変更 [#18930](https://github.com/pingcap/tidb/pull/18930)
    - `add partition` レプリカの確認に `delete-only` 状態を追加 [#18865](https://github.com/pingcap/tidb/pull/18865)

## 新機能

+ TiKV

    - エラーコードをエラーに定義する [#8387](https://github.com/tikv/tikv/pull/8387)

+ TiFlash

    - TiDBと統一されたログ形式をサポート

+ ツール

    + TiCDC

        - Kafka SSL接続をサポート [#764](https://github.com/pingcap/tiflow/pull/764)
        - 古い値を出力するサポートを追加 [#708](https://github.com/pingcap/tiflow/pull/708)
        - カラムフラグを追加するサポートを追加 [#796](https://github.com/pingcap/tiflow/pull/796)
        - 前のバージョンのDDLステートメントおよびテーブルスキーマを出力するサポートを追加 [#799](https://github.com/pingcap/tiflow/pull/799)

## 改善点

+ TiDB

    - 大規模な結合クエリの `DecodePlan` のパフォーマンスを最適化 [#18941](https://github.com/pingcap/tidb/pull/18941)
    - `Region cache miss` エラーが発生したときのGCロックスキャンの回数を減らす [#18876](https://github.com/pingcap/tidb/pull/18876)
    - 統計フィードバックがクラスターパフォーマンスに与える影響を緩和する [#18772](https://github.com/pingcap/tidb/pull/18772)
    - RPC応答が返される前に操作をキャンセルするサポートを追加する [#18580](https://github.com/pingcap/tidb/pull/18580)
    - TiDBメトリックプロファイルを生成するためのHTTP APIを追加するサポートを追加する [#18531](https://github.com/pingcap/tidb/pull/18531)
    - パーティションテーブルのメモリ使用量をGrafanaで詳細に表示するサポートを追加する [#18679](https://github.com/pingcap/tidb/pull/18679)
    - `EXPLAIN` の結果で `BatchPointGet` オペレータの詳細な実行時情報を表示するサポートを追加する [#18892](https://github.com/pingcap/tidb/pull/18892)
    - `EXPLAIN` の結果で `PointGet` オペレータの詳細な実行時情報を表示するサポートを追加する [#18817](https://github.com/pingcap/tidb/pull/18817)
    - `remove()` での `Consume` の潜在的なデッドロックを警告する [#18395](https://github.com/pingcap/tidb/pull/18395)
    - `StrToInt` および `StrToFloat` の動作を改善し、JSONを `date`、`time`、および `timestamp` のタイプに変換するサポートを追加する [#18159](https://github.com/pingcap/tidb/pull/18159)
    - `TableReader` オペレータのメモリ使用量を制限するサポートを追加する [#18392](https://github.com/pingcap/tidb/pull/18392)
    - `batch cop` リクエストをリトライする際のバックオフ回数を過度に減らすサポートを追加する [#18999](https://github.com/pingcap/tidb/pull/18999)
    - `ALTER TABLE` アルゴリズムの互換性を改善する [#19270](https://github.com/pingcap/tidb/pull/19270)
    - 単一のパーティションテーブルが内部側で `IndexJoin` をサポートするようにするサポートを追加する [#19151](https://github.com/pingcap/tidb/pull/19151)
    - ログに無効な行が含まれている場合でもログファイルを検索するサポートを追加する [#18579](https://github.com/pingcap/tidb/pull/18579)

+ PD

    - TiFlashなどの特殊なエンジンを持つストアでリージョンを散在させるサポートを追加する [#2706](https://github.com/tikv/pd/pull/2706)
    - 指定されたキー範囲のリージョンスケジューリングを優先するためのリージョンHTTP APIをサポートする [#2687](https://github.com/tikv/pd/pull/2687)
    - リージョンの散在後のリーダーの分布を改善する [#2684](https://github.com/tikv/pd/pull/2684)
    - TSOリクエスト用の追加のテストとログを追加する [#2678](https://github.com/tikv/pd/pull/2678)
    - リージョンのリーダーが変更された後に無効なキャッシュ更新を回避する [#2672](https://github.com/tikv/pd/pull/2672)
    - `store.GetLimit`が墓石ストアを返すようにするためのオプションを追加する [#2743](https://github.com/tikv/pd/pull/2743)
    - PDリーダーとフォロワーの間でリージョンリーダー変更の同期をサポートする [#2795](https://github.com/tikv/pd/pull/2795)
    - GCセーフポイントサービスを問い合わせるためのコマンドを追加する [#2797](https://github.com/tikv/pd/pull/2797)
    - パフォーマンスを改善するためにフィルタ内の `region.Clone` の呼び出しを置き換える [#2801](https://github.com/tikv/pd/pull/2801)
    - 大規模クラスターのパフォーマンスを改善するためにリージョンフローキャッシュの更新を無効にするオプションを追加する [#2848](https://github.com/tikv/pd/pull/2848)

+ TiFlash

    - CPU、I/O、RAMの使用率およびストレージエンジンのメトリックを表示するためのGrafanaパネルを追加する
    - Raftログの処理ロジックを最適化してI/O操作を減らす
    - ブロックされた `add partition` DDLステートメントのリージョンスケジューリングを加速する
    - DeltaTreeのデルタデータのコンパクションを最適化して読み書きの増幅を減らす
    - 複数のスレッドを使用してスナップショットを事前処理してリージョンのスナップショットの適用を最適化する
    - TiFlashの読み込み負荷が低いときにファイルディスクリプタの開数を最適化してシステムリソースの消費を減らす
    - TiFlashの再起動時に不要な小さなファイルが作成される回数を最適化する
    - データ保存のための暗号化をサポートする
    - データ転送のためのTLSをサポートする

+ ツール

    + TiCDC

        - TSOの取得頻度を下げる [#801](https://github.com/pingcap/tiflow/pull/801)

    + バックアップ＆リストア（BR）

        - 一部のログを最適化する [#428](https://github.com/pingcap/br/pull/428)

    + Dumpling

        - 接続が作成された後にFTWRLを解放してMySQLのロック時間を減らす [#121](https://github.com/pingcap/dumpling/pull/121)

    + TiDB Lightning

        - 一部のログを最適化する [#352](https://github.com/pingcap/tidb-lightning/pull/352)

## バグ修正

+ TiDB

    - `builtinCastRealAsDecimalSig` 関数で `ErrTruncate/Overflow` エラーが誤って処理されることによって発生する `should ensure all columns have the same length` エラーを修正する [#18967](https://github.com/pingcap/tidb/pull/18967)
    - パーティションテーブルで `pre_split_regions` テーブルオプションが機能しない問題を修正する [#18837](https://github.com/pingcap/tidb/pull/18837)
    - 大規模なトランザクションが予期せず早期に終了する可能性がある問題を修正する [#18813](https://github.com/pingcap/tidb/pull/18813)
    - `collation` 関数を使用した場合に誤ったクエリ結果が取得される問題を修正する [#18735](https://github.com/pingcap/tidb/pull/18735)
    - `getAutoIncrementID()` 関数が `tidb_snapshot` セッション変数を考慮しないために発生する `table not exist` エラーでダンプツールが失敗する可能性がある問題を修正する [#18692](https://github.com/pingcap/tidb/pull/18692)
    - `select a from t having t.a` のようなSQLステートメントで `unknown column error` を修正する [#18434](https://github.com/pingcap/tidb/pull/18434)
    - ハッシュパーティションテーブルに64ビット符号なしタイプを書き込むとオーバーフローが発生し、予期しない負の数が取得される問題を修正する [#18186](https://github.com/pingcap/tidb/pull/18186)
    - `char` 関数の誤った動作を修正する [#18122](https://github.com/pingcap/tidb/pull/18122)
    - `ADMIN REPAIR TABLE` ステートメントが範囲パーティションの式で整数を解析できない問題を修正する [#17988](https://github.com/pingcap/tidb/pull/17988)
    - `SET CHARSET` ステートメントの誤った動作を修正する [#17289](https://github.com/pingcap/tidb/pull/17289)
    - 間違った照合設定によって引き起こされたバグを修正し、`collation` 関数の結果が誤っている問題を修正 [#17231](https://github.com/pingcap/tidb/pull/17231)
    - `STR_TO_DATE`のフォーマットトークン'%r'、'%h'の処理がMySQLと一貫していない問題を修正 [#18727](https://github.com/pingcap/tidb/pull/18727)
    - `cluster_info` テーブルでのTiDBバージョン情報がPD/TiKVと一貫していない問題を修正 [#18413](https://github.com/pingcap/tidb/pull/18413)
    - 悲観的トランザクションのための既存のチェックを修正 [#19004](https://github.com/pingcap/tidb/pull/19004)
    - `union select for update` を実行すると同時実行競合が発生する可能性がある問題を修正 [#19006](https://github.com/pingcap/tidb/pull/19006)
    - `apply` が `PointGet` オペレーターの子を持つ場合に間違ったクエリ結果が発生する問題を修正 [#19046](https://github.com/pingcap/tidb/pull/19046)
    - `IndexLookUp`が `Apply` オペレーターの内側にある場合に発生する不正確な結果を修正 [#19496](https://github.com/pingcap/tidb/pull/19496)
    - `anti-semi-join` クエリの不正確な結果を修正 [#19472](https://github.com/pingcap/tidb/pull/19472)
    - 誤った `BatchPointGet` の使用によって引き起こされた不正確な結果を修正 [#19456](https://github.com/pingcap/tidb/pull/19456)
    - `UnionScan` が `Apply` オペレーターの内側にある場合に発生する不正確な結果を修正 [#19496](https://github.com/pingcap/tidb/pull/19496)
    - `EXECUTE` ステートメントを使用して高コストなクエリログを出力するとパニックが発生する問題を修正 [#17419](https://github.com/pingcap/tidb/pull/17419)
    - `ENUM` や `SET` の結合キーがある場合のインデックス結合エラーを修正 [#19235](https://github.com/pingcap/tidb/pull/19235)
    - インデックス列に `NULL` 値が存在する場合にクエリ範囲が構築されない問題を修正 [#19358](https://github.com/pingcap/tidb/pull/19358)
    - グローバル構成の更新によって引き起こされるデータ競合の問題を修正 [#17964](https://github.com/pingcap/tidb/pull/17964)
    - 大文字のスキーマで文字セットを変更するとパニックが発生する問題を修正 [#19286](https://github.com/pingcap/tidb/pull/19286)
    - ディスクスパイルアクション中に一時ディレクトリを変更することによって引き起こされる予期しないエラーを修正 [#18970](https://github.com/pingcap/tidb/pull/18970)
    - decimal 型に対する誤ったハッシュキーを修正 [#19131](https://github.com/pingcap/tidb/pull/19131)
    - `PointGet` および `BatchPointGet` オペレーターがパーティション選択構文を考慮せず、不正確な結果を返す問題を修正 [#19141](https://github.com/pingcap/tidb/issues/19141)
    - `UnionScan` オペレーターと一緒に `Apply` オペレーターを使用すると不正確な結果を修正 [#19104](https://github.com/pingcap/tidb/issues/19104)
    - インデックス付きの仮想生成列が誤った値を返すバグを修正 [#17989](https://github.com/pingcap/tidb/issues/17989)
    - 並行実行によって引き起こされるパニックを修正するため、ランタイム統計情報にロックを追加 [#18983](https://github.com/pingcap/tidb/pull/18983)

+ TiKV

    - Hibernate Region が有効になっている場合のリーダー選出を高速化 [#8292](https://github.com/tikv/tikv/pull/8292)
    - スケジューリング中のメモリリーク問題を修正 [#8357](https://github.com/tikv/tikv/pull/8357)
    - リーダーが速やかにハイバネート状態になることを防ぐための `hibernate-timeout` 構成項目を追加 [#8208](https://github.com/tikv/tikv/pull/8208)

+ PD

    - リーダーの変更時にTSOリクエストが失敗する可能性があるバグを修正 [#2666](https://github.com/tikv/pd/pull/2666)
    - プレイスメントルールが有効な場合、時々Regionレプリカが最適な状態にスケジュールされない問題を修正 [#2720](https://github.com/tikv/pd/pull/2720)
    - プレイスメントルールが有効な場合、`Balance Leader` が機能しない問題を修正 [#2726](https://github.com/tikv/pd/pull/2726)
    - 不健康なストアがストア負荷統計からフィルタリングされない問題を修正 [#2805](https://github.com/tikv/pd/pull/2805)

+ TiFlash

    - 以前のバージョンからアップグレードした後にTiFlashが特殊文字を含むデータベースやテーブル名を正常に起動できない問題を修正
    - 初期化中に例外がスローされた場合にTiFlashプロセスが終了しない問題を修正

+ Tools

    + Backup & Restore (BR)

        - バックアップサマリーログでの合計KVおよび合計バイトの重複した計算の問題を修正 [#472](https://github.com/pingcap/br/pull/472)
        - このモードに切り替えてから最初の5分間、インポートモードが機能しない問題を修正 [#473](https://github.com/pingcap/br/pull/473)

    + Dumpling

        - FTWRLロックが適切なタイミングで解放されない問題を修正 [#128](https://github.com/pingcap/dumpling/pull/128)

    + TiCDC

        - 失敗した `changefeed` を削除できない問題を修正 [#782](https://github.com/pingcap/tiflow/pull/782)
        - 1つのユニークなインデックスをハンドルインデックスとして選択することで無効な `delete` イベントを修正 [#787](https://github.com/pingcap/tiflow/pull/787)
        - GCセーフポイントが停止した `changefeed` のチェックポイントを超えて前進する問題を修正 [#797](https://github.com/pingcap/tiflow/pull/797)
        - ネットワークI/O待機がタスクの終了をブロックする問題を修正 [#825](https://github.com/pingcap/tiflow/pull/825)
        - 不要なデータが下流に誤って複製される可能性があるバグを修正 [#743](https://github.com/pingcap/tiflow/issues/743)

    + TiDB Lightning

        - TiDBバックエンドを使用する際に空のバイナリ/16進リテラルで構文エラーを修正 [#357](https://github.com/pingcap/tidb-lightning/pull/357)