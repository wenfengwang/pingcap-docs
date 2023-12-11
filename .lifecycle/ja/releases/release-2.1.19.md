---
title: TiDB 2.1.19 リリースノート
aliases: ['/docs/dev/releases/release-2.1.19/','/docs/dev/releases/2.1.19/']
---

# TiDB 2.1.19 リリースノート

リリース日: 2019年12月27日

TiDB バージョン: 2.1.19

TiDB Ansible バージョン: 2.1.19

## TiDB

+ SQL オプティマイザ
    - `select max(_tidb_rowid) from t` のシナリオを最適化し、フルテーブルスキャンを回避 [#13294](https://github.com/pingcap/tidb/pull/13294)
    - クエリ内のユーザ変数への誤った値の割り当てやプッシュダウンによる誤った結果を修正 [#13230](https://github.com/pingcap/tidb/pull/13230)
    - 統計情報の更新時にデータ競合が発生し、統計情報が正確でない問題を修正 [#13690](https://github.com/pingcap/tidb/pull/13690)
    - `UPDATE` ステートメントにサブクエリとストアド生成列の両方が含まれる場合の結果が正しくない問題を修正；異なるデータベースから2つの同じ名前のテーブルが含まれる場合に `UPDATE` ステートメントの実行エラーを修正 [#13357](https://github.com/pingcap/tidb/pull/13357)
    - `PhysicalUnionScan` オペレーターが統計情報を誤って設定しているため、クエリプランが誤って選択される問題を修正 [#14134](https://github.com/pingcap/tidb/pull/14134)
    - 自動的な `ANALYZE` をよりタイムリーに行うために `minAutoAnalyzeRatio` 制約を削除 [#14013](https://github.com/pingcap/tidb/pull/14013)
    - `WHERE` 句がユニークキーに対する等しい条件を含んでいる場合に推定行数が `1` を超える問題を修正 [#13385](https://github.com/pingcap/tidb/pull/13385)
+ SQL 実行エンジン
    - `int64` を `unit64` の中間結果として使用する際の精度オーバーフローを修正 [#13036](https://github.com/pingcap/tidb/pull/13036)
    - クエリ内に `SLEEP` 関数が含まれている場合に、クエリ内の `sleep(1)` が無効になる問題を修正 [#13039](https://github.com/pingcap/tidb/pull/13039)
    - `INSERT ON DUPLICATE UPDATE` ステートメントで `Chunk` を再利用してメモリオーバーヘッドを削減 [#12999](https://github.com/pingcap/tidb/pull/12999)
    - `slow_query` テーブルのためにより多くのトランザクション関連のフィールドを追加 [#13129](https://github.com/pingcap/tidb/pull/13129):
        - `Prewrite_time`
        - `Commit_time`
        - `Get_commit_ts_time`
        - `Commit_backoff_time`
        - `Backoff_types`
        - `Resolve_lock_time`
        - `Local_latch_wait_time`
        - `Write_key`
        - `Write_size`
        - `Prewrite_region`
        - `Txn_retry`
    - `UPDATE` ステートメント内のサブクエリが誤って変換される問題を修正；`WHERE` 句がサブクエリを含んでいる場合の `UPDATE` 実行エラーを修正 [#13120](https://github.com/pingcap/tidb/pull/13120)
    - パーティション化されたテーブルで `ADMIN CHECK TABLE` を実行できるようにする [#13143](https://github.com/pingcap/tidb/pull/13143)
    - `ON UPDATE CURRENT_TIMESTAMP` がカラム属性として使用され、浮動小数点の精度が指定された場合の `SHOW CREATE TABLE` ステートメントなどの精度が不完全である問題を修正 [#12462](https://github.com/pingcap/tidb/pull/12462)
    - `foreign key` がチェックされずに列を削除、変更、または変更する場合に、`SELECT * FROM information_schema.KEY_COLUMN_USAGE` ステートメントの実行時にパニックが発生する問題を修正 [#14162](https://github.com/pingcap/tidb/pull/14162)
    - TiDB で `Streaming` が有効になっている場合に戻されるデータが重複する可能性がある問題を修正 [#13255](https://github.com/pingcap/tidb/pull/13255)
    - 夏時間によって発生する `Invalid time format` エラーを修正 [#13624](https://github.com/pingcap/tidb/pull/13624)
    - 整数が符号なし浮動小数点数や10進数の型に変換される際に精度が失われるためにデータが正しくない問題を修正 [#13756](https://github.com/pingcap/tidb/pull/13756)
    - `Quote` 関数が `NULL` 値を処理する際に誤った型の値が返される問題を修正 [#13681](https://github.com/pingcap/tidb/pull/13681)
    - 文字列から `gotime.Local` を使用して日付を解析した後のタイムゾーンが正しくない問題を修正 [#13792](https://github.com/pingcap/tidb/pull/13792)
    - `builtinIntervalRealSig` の実装で `binSearch` 関数がエラーを返さないために結果が正しくない可能性がある問題を修正 [#13768](https://github.com/pingcap/tidb/pull/13768)
    - `INSERT` ステートメントの実行において文字列型を浮動小数点型に変換する際にエラーが発生する可能性がある問題を修正 [#14009](https://github.com/pingcap/tidb/pull/14009)
    - `sum(distinct)` 関数から返される結果が正しくない問題を修正 [#13041](https://github.com/pingcap/tidb/pull/13041)
    - 同じ場所の `union` のデータをマージされたタイプに `CAST` 変換する際に `jsonUnquoteFunction` 関数の返される長さの型が誤っていたために `data too long` が返される問題を修正 [#13645](https://github.com/pingcap/tidb/pull/13645)
    - 権限チェックが厳格すぎてパスワードが設定できない問題を修正 [#13805](https://github.com/pingcap/tidb/pull/13805)
+ サーバー
    - `KILL CONNECTION` がゴルーチンリークを引き起こす可能性がある問題を修正 [#13252](https://github.com/pingcap/tidb/pull/13252)
    - HTTP API の `info/all` インタフェースを介してすべての TiDB ノードの binlog ステータスを取得するサポートを追加 [#13188](https://github.com/pingcap/tidb/pull/13188)
    - Windows 上で TiDB プロジェクトのビルドに失敗する問題を修正 [#13650](https://github.com/pingcap/tidb/pull/13650)
    - TiDB サーバーのバージョンを制御および変更するための `server-version` 構成項目を追加 [#13904](https://github.com/pingcap/tidb/pull/13904)
    - Go1.13 でコンパイルされたバイナリの `plugin` が正常に実行されない問題を修正 [#13527](https://github.com/pingcap/tidb/pull/13527)
+ DDL
    - テーブルが `COLLATE` を含む場合に、テーブルが作成され、テーブル内に `COLLATE` が含まれる場合にテーブルの `COLLATE` ではなく、システムのデフォルト文字セットを使用する問題を修正 [#13190](https://github.com/pingcap/tidb/pull/13190)
    - テーブルを作成する際にインデックス名の長さを制限 [#13311](https://github.com/pingcap/tidb/pull/13311)
    - テーブルの名前の長さがリネームする際にチェックされない問題を修正 [#13345](https://github.com/pingcap/tidb/pull/13345)
    - `BIT` 列の幅範囲を確認 [#13511](https://github.com/pingcap/tidb/pull/13511)
    - `change/modify column` から出力されるエラー情報をより理解しやすくするために変更 [#13798](https://github.com/pingcap/tidb/pull/13798)
    - ダウンストリームのDrainerが未処理の状態で `drop column` 操作を実行した場合、ダウンストリームは影響を受ける列のない DML 操作を受け取る問題を修正 [#13974](https://github.com/pingcap/tidb/pull/13974)

## TiKV

+ Raftstore
    - TiKV を再起動し、リージョンのマージとCompact logの適用中に `is_merging` に誤った値が与えられた場合にパニックが発生する問題を修正 [#5884](https://github.com/tikv/tikv/pull/5884)
+ Importer
    - gRPC メッセージ長の制限を削除 [#5809](https://github.com/tikv/tikv/pull/5809)

## PD

- すべてのリージョンを取得するための HTTP API のパフォーマンスを改善 [#1988](https://github.com/pingcap/pd/pull/1988)
- etcd をアップグレードし、etcd PreVote がリーダーを選出できない問題を修正 (ダウングレードはサポートされていません) [#2052](https://github.com/pingcap/pd/pull/2052)

## Tools

+ TiDB ビンログ
    - binlogctl を通じたノードステータス情報の出力を最適化 [#777](https://github.com/pingcap/tidb-binlog/pull/777)
    - Drainerフィルター構成で`nil`値によって発生したパニックを修正する[#802](https://github.com/pingcap/tidb-binlog/pull/802)
    - Pumpの`Graceful`な終了を最適化する[#825](https://github.com/pingcap/tidb-binlog/pull/825)
    - Pumpがbinlogデータを書き込む際により詳細なモニタリングメトリクスを追加する[#830](https://github.com/pingcap/tidb-binlog/pull/830)
    - Drainerのロジックを最適化し、DrainerがDDL操作を実行した後にテーブル情報を更新する[#836](https://github.com/pingcap/tidb-binlog/pull/836)
    - Pumpがこのbinlogを受信しない場合に、DDL操作のコミットbinlogが無視される問題を修正する[#855](https://github.com/pingcap/tidb-binlog/pull/855)

## TiDB Ansible

- TiDBサービスの`Uncommon Error OPM`モニタリング項目を`Write Binlog Error`に名前変更し、対応するアラートメッセージを追加する[#1038](https://github.com/pingcap/tidb-ansible/pull/1038)
- TiSparkを2.1.8にアップグレードする[#1063](https://github.com/pingcap/tidb-ansible/pull/1063)