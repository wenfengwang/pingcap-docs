---
title: TiDB 3.0.20 リリースノート
---

# TiDB 3.0.20 リリースノート

リリース日: 2020年12月25日

TiDB バージョン: 3.0.20

## 互換性の変更

+ TiDB

    - `enable-streaming` 構成項目を非推奨とします [#21054](https://github.com/pingcap/tidb/pull/21054)

## 改善点

+ TiDB

    - `LOAD DATA` ステートメントの準備中にエラーが発生した場合にエラーを発生させるようにする [#21222](https://github.com/pingcap/tidb/pull/21222)

+ TiKV

    - `end_point_slow_log_threshold` 構成項目を追加する [#9145](https://github.com/tikv/tikv/pull/9145)

## バグ修正

+ TiDB

    - 悲観的トランザクションのトランザクションステータスのキャッシュが正しくない問題を修正 [#21706](https://github.com/pingcap/tidb/pull/21706)
    - `INFORMATION_SCHEMA.TIDB_HOT_REGIONS` をクエリする際に発生する統計情報の不正確な問題を修正 [#21319](https://github.com/pingcap/tidb/pull/21319)
    - データベース名が純粋な小文字表現でない場合に `DELETE` がデータを正しく削除しない可能性がある問題を修正 [#21205](https://github.com/pingcap/tidb/pull/21205)
    - 再帰ビューのビルド時に発生するスタックオーバーフローの問題を修正 [#21000](https://github.com/pingcap/tidb/pull/21000)
    - TiKV クライアントでのゴルーチンリークの問題を修正 [#20863](https://github.com/pingcap/tidb/pull/20863)
    - `year` 型の誤ったデフォルトゼロ値の問題を修正 [#20828](https://github.com/pingcap/tidb/pull/20828)
    - インデックスルックアップ結合でのゴルーチンリークの問題を修正 [#20791](https://github.com/pingcap/tidb/pull/20791)
    - 悲観的トランザクションで `INSERT SELECT FOR UPDATE` を実行すると誤ったパケットが返される問題を修正 [#20681](https://github.com/pingcap/tidb/pull/20681)
    - 不明なタイムゾーン `'posixrules'` の問題を修正 [#20605](https://github.com/pingcap/tidb/pull/20605)
    - 符号なし整数型をビット型に変換する際に発生する問題を修正 [#20362](https://github.com/pingcap/tidb/pull/20362)
    - ビット型列の破損したデフォルト値を修正 [#20339](https://github.com/pingcap/tidb/pull/20339)
    - 等価条件の一つが `Enum` または `Set` 型である場合に潜在的に不正確な結果を修正 [#20296](https://github.com/pingcap/tidb/pull/20296)
    - `!= any()` の誤った動作を修正 [#20061](https://github.com/pingcap/tidb/pull/20061)
    - `BETWEEN...AND...` での型変換が無効な結果を返す問題を修正 [#21503](https://github.com/pingcap/tidb/pull/21503)
    - `ADDDATE` 関数との互換性の問題を修正 [#21008](https://github.com/pingcap/tidb/pull/21008)
    - 追加された `Enum` 列の正しいデフォルト値を設定する [#20999](https://github.com/pingcap/tidb/pull/20999)
    - `SELECT DATE_ADD('2007-03-28 22:08:28',INTERVAL "-2.-2" SECOND)` の結果を MySQL と互換性のあるものに修正 [#20627](https://github.com/pingcap/tidb/pull/20627)
    - 列の型を変更する際に発生する誤ったデフォルト値を修正 [#20532](https://github.com/pingcap/tidb/pull/20532)
    - 入力引数が `float` または `decimal` 型である場合に `timestamp` 関数が間違った結果を返す問題を修正 [#20469](https://github.com/pingcap/tidb/pull/20469)
    - 統計情報での潜在的なデッドロックの問題を修正 [#20424](https://github.com/pingcap/tidb/pull/20424)
    - 浮動小数点型データのオーバーフローが挿入される問題を修正 [#20251](https://github.com/pingcap/tidb/pull/20251)

+ TiKV

    - コミットされたトランザクションでキーがロックされて削除された場合にそのキーが存在するというエラーが返される問題を修正 [#8931](https://github.com/tikv/tikv/pull/8931)

+ PD

    - PD の起動時にスタージリージョンが多すぎる場合に過剰なログが出力される問題を修正 [#3064](https://github.com/pingcap/pd/pull/3064)