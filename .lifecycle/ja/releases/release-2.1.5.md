---
title: TiDB 2.1.5 リリースノート
aliases: ['/docs/dev/releases/release-2.1.5/','/docs/dev/releases/2.1.5/']
---

# TiDB 2.1.5 リリースノート

2019年2月28日、TiDB 2.1.5 がリリースされました。対応する TiDB Ansible 2.1.5 もリリースされています。このリリースは TiDB 2.1.4 と比較して、安定性、SQL オプティマイザー、統計情報、実行エンジンの大幅な改善が行われています。

## TiDB

+ SQL オプティマイザ/エグゼキュータ
    - `SHOW CREATE TABLE` がテーブルの文字セット情報と同じ場合に列の文字セット情報を表示しないようにし、`SHOW CREATE TABLE` の互換性を向上させました [#9306](https://github.com/pingcap/tidb/pull/9306)
    - 演算の簡略化のために `Sort` 演算子から `ScalarFunc` を抽出して `Projection` 演算子に変更し、一部のケースでのpanicや誤った結果を修正しました [#9319](https://github.com/pingcap/tidb/pull/9319)
    - `Sort` 演算子内の定数値での並び替えフィールドを削除しました [#9335](https://github.com/pingcap/tidb/pull/9335), [#9440](https://github.com/pingcap/tidb/pull/9440)
    - 符号なし整数の列にデータを挿入する際のデータオーバーフローの問題を修正しました [#9339](https://github.com/pingcap/tidb/pull/9339)
    - ターゲットのバイナリの長さが `max_allowed_packet` を超える場合に `cast_as_binary` を `NULL` に設定するようにしました [#9349](https://github.com/pingcap/tidb/pull/9349)
    - `IF` と `IFNULL` の定数折りたたみ処理を最適化しました [#9351](https://github.com/pingcap/tidb/pull/9351)
    - 簡単なクエリの安定性を向上するために、TiDB のインデックス選択をスカイライン剪定を使用して最適化しました [#9356](https://github.com/pingcap/tidb/pull/9356)
    - `DNF` 式の選択度を計算するサポートを追加しました [#9405](https://github.com/pingcap/tidb/pull/9405)
    - 一部のケースでの `!=ANY()` および `=ALL()` の誤ったSQLクエリ結果を修正しました [#9403](https://github.com/pingcap/tidb/pull/9403)
    - `Merge Join` 操作が実行される2つのテーブルの結合キーの型が異なる場合のpanicや誤った結果を修正しました [#9438](https://github.com/pingcap/tidb/pull/9438)
    - `RAND()` 関数の結果がMySQLと互換性がない問題を修正しました [#9446](https://github.com/pingcap/tidb/pull/9446)
    - `NULL` および空の結果セットの処理の論理をリファクタリングして、正しい結果を取得し、MySQLとの互換性を向上させました [#9449](https://github.com/pingcap/tidb/pull/9449)
+ サーバー
    - `INSERT` 文を実行する際にデータの一意性制約をチェックするための `tidb_constraint_check_in_place` システム変数を追加しました [#9401](https://github.com/pingcap/tidb/pull/9401)
    - `tidb_force_priority` システム変数の値が構成ファイルで設定された値と異なる問題を修正しました [#9347](https://github.com/pingcap/tidb/pull/9347)
    - 一般的なログに使用中のデータベース名を表示するための `current_db` フィールドを追加しました [#9346](https://github.com/pingcap/tidb/pull/9346)
    - テーブルIDで表情報を取得するためのHTTP APIを追加しました [#9408](https://github.com/pingcap/tidb/pull/9408)
    - 一部のケースで `LOAD DATA` が誤ったデータを読み込む問題を修正しました [#9414](https://github.com/pingcap/tidb/pull/9414)
    - MySQLクライアントとTiDBの間で接続を構築するのに長い時間がかかる問題を修正しました [#9451](https://github.com/pingcap/tidb/pull/9451)
+ DDL
    - `DROP COLUMN` 操作をキャンセルする際のいくつかの問題を修正しました [#9352](https://github.com/pingcap/tidb/pull/9352)
    - パーティション化されたテーブルの`DROP`または`ADD`操作をキャンセルする際のいくつかの問題を修正しました [#9376](https://github.com/pingcap/tidb/pull/9376)
    - `ADMIN CHECK TABLE` が一部のケースでデータインデックスの整合性を誤って報告する問題を修正しました [#9399](https://github.com/pingcap/tidb/pull/9399)
    - `TIMESTAMP`デフォルト値のタイムゾーンの問題を修正しました [#9108](https://github.com/pingcap/tidb/pull/9108)

## PD

- `GetAllStores` インタフェースで `Tombstone` ストアを返される結果から削除するための `exclude_tombstone_stores` オプションを提供しました [#1444](https://github.com/pingcap/pd/pull/1444)

## TiKV

- 一部のケースで Importer がデータをインポートできない問題を修正しました [#4223](https://github.com/tikv/tikv/pull/4223)
- 一部のケースで `KeyNotInRegion` エラーを修正しました [#4125](https://github.com/tikv/tikv/pull/4125)
- 一部のケースでリージョンのマージによるpanicの問題を修正しました [#4235](https://github.com/tikv/tikv/pull/4235)
- 詳細な `StoreNotMatch` エラーメッセージを追加しました [#3885](https://github.com/tikv/tikv/pull/3885)

## Tools

+ Lightning
    - クラスター内に Tombstone ストアが存在する場合にエラーを報告せずに終了するようにしました [#4223](https://github.com/tikv/tikv/pull/4223)
+ TiDB Binlog
    - DDLイベントのレプリケーションの正確性を保証するために、DDL binlog レプリケーションプランを更新しました [#9304](https://github.com/pingcap/tidb/issues/9304)