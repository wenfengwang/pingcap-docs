---
title: TiDB 3.1.0 GA リリースノート
aliases: ['/docs/dev/releases/release-3.1.0-ga/','/docs/dev/releases/3.1.0-ga/']
---

# TiDB 3.1.0 GA リリースノート

リリース日: 2020年4月16日

TiDB バージョン: 3.1.0 GA

TiDB Ansible バージョン: 3.1.0 GA

## 互換性の変更

+ TiDB

    - `report-status` 設定項目が有効な場合に、HTTP リッスンポートが利用できない場合に TiDB の直接停止開始をサポート [#16291](https://github.com/pingcap/tidb/pull/16291)

+ ツール

    - バックアップ＆リストア（BR）

        * BR は 3.1 GA より前の TiKV クラスタからのデータリストアをサポートしていません [#233](https://github.com/pingcap/br/pull/233)

## 新機能

+ TiDB

    - `explain format = "dot"` で Coprocessor タスクの情報を表示するサポート [#16125](https://github.com/pingcap/tidb/pull/16125)
    - `disable-error-stack` 設定項目を使用して、ログの冗長なスタック情報を削減するサポート [#16182](https://github.com/pingcap/tidb/pull/16182)

+ Placement Driver（PD）

    - ホットリージョンのスケジューリングを最適化するサポート [#2342](https://github.com/pingcap/pd/pull/2342)

+ TiFlash

    - DeltaTree エンジンの読み書きワークロードに関連するメトリクスレポートを追加する
    - `fromUnixTime` および `dateFormat` 関数をプッシュダウンするサポート
    - ラフセットフィルタをデフォルトで無効にする

+ TiDB Ansible

    - TiFlash モニタを追加するサポート [#1253](https://github.com/pingcap/tidb-ansible/pull/1253) [#1257](https://github.com/pingcap/tidb-ansible/pull/1257)
    - TiFlash の構成パラメータを最適化するサポート [#1262](https://github.com/pingcap/tidb-ansible/pull/1262) [#1265](https://github.com/pingcap/tidb-ansible/pull/1265) [#1271](https://github.com/pingcap/tidb-ansible/pull/1271)
    - TiDB の起動スクリプトを最適化するサポート [#1268](https://github.com/pingcap/tidb-ansible/pull/1268)

## バグ修正

+ TiDB

    - 特定のシナリオで発生するマージ結合操作によるパニックの問題を修正 [#15920](https://github.com/pingcap/tidb/pull/15920)
    - セレクティビティ計算で一部の式が繰り返しカウントされる問題を修正 [#16052](https://github.com/pingcap/tidb/pull/16052)
    - 極端な場合に統計情報の読み込みで発生するパニックの問題を修正 [#15710](https://github.com/pingcap/tidb/pull/15710)
    - SQL クエリで等価式を認識できない場合に一部のケースでエラーが返される問題を修正 [#16015](https://github.com/pingcap/tidb/pull/16015)
    - 特定の場合に別のデータベースの `view` をクエリする際にエラーが返される問題を修正 [#15867](https://github.com/pingcap/tidb/pull/15867)
    - `fast analyze` を使用してカラムを処理する際に発生するパニックの問題を修正 [#16080](https://github.com/pingcap/tidb/pull/16080)
    - `current_role` の印字結果の不正な文字セットを修正 [#16084](https://github.com/pingcap/tidb/pull/16084)
    - MySQL 接続ハンドシェイクエラーのログを洗練する [#15799](https://github.com/pingcap/tidb/pull/15799)
    - 監査プラグインが読み込まれた後のポートプロービングにより発生するパニックの問題を修正 [#16065](https://github.com/pingcap/tidb/pull/16065)
    - `sort` 演算子における `TypeNull` クラスが可変長タイプと誤って認識されるために左結合でのパニックの問題を修正 [#15739](https://github.com/pingcap/tidb/pull/15739)
    - モニタリングセッションのリトライエラーの不正確なカウントの問題を修正 [#16120](https://github.com/pingcap/tidb/pull/16120)
    - `ALLOW_INVALID_DATES` モードにおける `weekday` の誤った結果の問題を修正 [#16171](https://github.com/pingcap/tidb/pull/16171)
    - TiFlash ノードを持つクラスタにおいて Garbage Collection（GC）が正常に動作しない可能性の問題を修正 [#15761](https://github.com/pingcap/tidb/pull/15761)
    - ハッシュパーティションテーブルの大きなパーティション数を設定したユーザが TiDB がメモリ不足（OOM）になる問題を修正 [#16219](https://github.com/pingcap/tidb/pull/16219)
    - 警告がエラーと誤解され、`UNION` 文が `SELECT` 文と同じ動作をするようにする問題を修正 [#16138](https://github.com/pingcap/tidb/pull/16138)
    - `TopN` が mocktikv にプッシュダウンされた際の実行エラーを修正 [#16200](https://github.com/pingcap/tidb/pull/16200)
    - 不要な `runtime.growslice` のオーバーヘッドを避けるために `chunk.column.nullBitMap` の初期長を増やす修正 [#16142](https://github.com/pingcap/tidb/pull/16142)

+ TiKV

    - レプリカ読み取りによって発生するパニックの問題を修正 [#7418](https://github.com/tikv/tikv/pull/7418) [#7369](https://github.com/tikv/tikv/pull/7369)
    - 復元プロセスが空のリージョンを作成する問題を修正 [#7419](https://github.com/tikv/tikv/pull/7419)
    - 繰り返しリゾルブロックリクエストが悲観的トランザクションの原子性に影響を与える可能性がある問題を修正 [#7389](https://github.com/tikv/tikv/pull/7389)

+ TiFlash

    - TiDB からスキーマレプリケーション時の `rename table` 操作の潜在的な問題を修正
    - 複数のデータパス構成下での `rename table` 操作によるデータ損失の問題を修正
    - 一部のシナリオで TiFlash が誤ったストレージ容量を報告する問題を修正
    - Region Merge が有効の場合に TiFlash からの読み取りによって発生する潜在的な問題を修正

+ ツール

    - TiDB ビンロッグ

        * TiFlash 関連の DDL ジョブが Drainer のレプリケーションを中断する可能性のある問題を修正 [#948](https://github.com/pingcap/tidb-binlog/pull/948) [#942](https://github.com/pingcap/tidb-binlog/pull/942)

    - バックアップ＆リストア（BR）

        * 無効化されているにも関わらず `checksum` 操作が実行される問題を修正 [#223](https://github.com/pingcap/br/pull/223)
        * TiDB が `auto-random` または `alter-pk` を有効にすると増分バックアップが失敗する問題を修正 [#230](https://github.com/pingcap/br/pull/230) [#231](https://github.com/pingcap/br/pull/231)