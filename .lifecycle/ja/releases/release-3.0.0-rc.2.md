---
title: TiDB 3.0.0-rc.2 リリースノート
aliases: ['/docs/dev/releases/release-3.0.0-rc.2/', '/docs/dev/releases/3.0.0-rc.2/']
---

# TiDB 3.0.0-rc.2 リリースノート

リリース日: 2019年5月28日

TiDB バージョン: 3.0.0-rc.2

TiDB Ansible バージョン: 3.0.0-rc.2

## 概要

2019年5月28日、TiDB 3.0.0-rc.2 がリリースされました。対応する TiDB Ansible バージョンは 3.0.0-rc.2 です。このリリースでは、TiDB 3.0.0-rc.1 と比較して、安定性、使いやすさ、機能、SQL オプティマイザー、統計情報、実行エンジンが大幅に改善されています。

## TiDB

+ SQL オプティマイザー
    - より多くのシナリオでの Index Join のサポート [#10540](https://github.com/pingcap/tidb/pull/10540)
    - 履歴統計情報のエクスポートのサポート [#10291](https://github.com/pingcap/tidb/pull/10291)
    - 単調増加インデックス列に対する増分の `Analyze` 操作のサポート [#10355](https://github.com/pingcap/tidb/pull/10355)
    - `Order By` 句における NULL 値の無視 [#10488](https://github.com/pingcap/tidb/pull/10488)
    - 列情報の簡素化時に `UnionAll` 論理演算子のスキーマ情報計算が誤っている問題の修正 [#10384](https://github.com/pingcap/tidb/pull/10384)
    - `Not` 演算子のプッシュダウン時に元の式を変更しないようにする問題の修正 [#10363](https://github.com/pingcap/tidb/pull/10363/files)
    - ヒストグラムの `dump`/`load` の相関のサポート [#10573](https://github.com/pingcap/tidb/pull/10573)

+ 実行エンジン
    - `batchChecker` で重複する行を取得する際に一意インデックスを正しく扱う問題の修正 [#10370](https://github.com/pingcap/tidb/pull/10370)
    - `CHAR` 列のスキャン範囲計算の問題の修正 [#10124](https://github.com/pingcap/tidb/pull/10124)
    - `PointGet` が負の数を誤って処理する問題の修正 [#10113](https://github.com/pingcap/tidb/pull/10113)
    - 同じ名前の `Window` 関数をマージして実行効率を向上させる [#9866](https://github.com/pingcap/tidb/pull/9866)
    - `Window` 関数の `RANGE` フレームが `OrderBy` 句を含まない場合に対応 [#10496](https://github.com/pingcap/tidb/pull/10496)

+ サーバー
    - TiKV で障害が発生した場合に TiDB が引き続き新しい接続を作成し続ける問題の修正 [#10301](https://github.com/pingcap/tidb/pull/10301)
    - `tidb_disable_txn_auto_retry` がライトコンフリクトエラーのみでなくすべてのリトライ可能なエラーに影響するようにする問題の修正 [#10339](https://github.com/pingcap/tidb/pull/10339)
    - パラメータのない DDL ステートメントを `prepare`/`execute` を使用して実行できるようにする [#10144](https://github.com/pingcap/tidb/pull/10144)
    - バックオフ時間を制御するための `tidb_back_off_weight` 変数の追加 [#10266](https://github.com/pingcap/tidb/pull/10266)
    - デフォルト条件下で TiDB が自動的にコミットされないトランザクションをリトライしないように、`tidb_disable_txn_auto_retry` のデフォルト値を `on` に設定する問題の修正 [#10266](https://github.com/pingcap/tidb/pull/10266)
    - `RBAC` における `role` のデータベース権限判断の修正 [#10261](https://github.com/pingcap/tidb/pull/10261)
    - ペシミスティックトランザクションモードのサポート (実験的) [#10297](https://github.com/pingcap/tidb/pull/10297)
    - 一部の場合におけるロック競合処理の待ち時間の短縮 [#10006](https://github.com/pingcap/tidb/pull/10006)
    - リーダーノードで障害が発生した場合にリージョンキャッシュがフォロワーノードにアクセスできるようにする [#10256](https://github.com/pingcap/tidb/pull/10256)
    - TSO のバッチ取得数を制御し、データの一貫性がそれほど厳密に必要とされないシナリオに適応するためにトランザクションが TSO を取得する回数を減らすための `tidb_low_resolution_tso` 変数の追加 [#10428](https://github.com/pingcap/tidb/pull/10428)

+ DDL
    - 旧バージョンの TiDB のストレージにおける文字セット名の大文字の問題の修正 [#10272](https://github.com/pingcap/tidb/pull/10272)
    - テーブルの作成時にテーブルリージョンを事前に割り当てる `preSplit` のサポート。これにより、テーブル作成後のライトホットスポットを回避するためのテーブルリージョンの事前確保 [#10221](https://github.com/pingcap/tidb/pull/10221)
    - 特定のケースで PD でバージョン情報が誤って更新される問題の修正 [#10324](https://github.com/pingcap/tidb/pull/10324)
    - `ALTER DATABASE` ステートメントを使用して文字セットと整理順序を変更する機能のサポート [#10393](https://github.com/pingcap/tidb/pull/10393)
    - 指定したテーブルのインデックスと範囲に基づいてリージョンを分割し、ホットスポットの問題を緩和する機能のサポート [#10203](https://github.com/pingcap/tidb/pull/10203)
    - `alter table` ステートメントを使用して小数列の精度を変更することを禁止する制限の追加 [#10433](https://github.com/pingcap/tidb/pull/10433)
    - ハッシュパーティションに対する式と関数の制限の修正 [#10273](https://github.com/pingcap/tidb/pull/10273)
    - パーティションを含むテーブルにインデックスを追加することで TiDB がいくつかのケースでパニックを引き起こす問題の修正 [#10475](https://github.com/pingcap/tidb/pull/10475)
    - 無効なテーブルスキーマを回避するためにDDL の実行前にテーブル情報を検証する機能の追加 [#10464](https://github.com/pingcap/tidb/pull/10464)
    - ハッシュパーティションをデフォルトで有効化します; また、パーティション定義に列が1つしかない場合は range columns パーティションを有効にします[#9936](https://github.com/pingcap/tidb/pull/9936)

## PD

- リージョンのメタデータを保存するためにデフォルトでリージョンストレージを有効化[#1524](https://github.com/pingcap/pd/pull/1524)
- ホットリージョンスケジューリングが別のスケジューラによって優先される問題の修正[#1522](https://github.com/pingcap/pd/pull/1522)
- リーダーの優先度が影響を受けない問題の修正[#1533](https://github.com/pingcap/pd/pull/1533)
- `ScanRegions` のための gRPC インターフェースの追加[#1535](https://github.com/pingcap/pd/pull/1535)
- アクティブにオペレータをプッシュ[#1536](https://github.com/pingcap/pd/pull/1536)
- 各ストアのオペレータの速度を個別に制御するためのストア制限メカニズムの追加[#1474](https://github.com/pingcap/pd/pull/1474)
- 不一致のConfigステータスの問題の修正[#1476](https://github.com/pingcap/pd/pull/1476)

## TiKV

+ エンジン
    - 複数の列ファミリーがブロックキャッシュを共有するサポート[#4563](https://github.com/tikv/tikv/pull/4563)

+ サーバー
    - `TxnScheduler` の削除[#4098](https://github.com/tikv/tikv/pull/4098)
    - ペシミスティックロックトランザクションのサポート[#4698](https://github.com/tikv/tikv/pull/4698)

+ Raftstore
    - はHibernateリージョンをサポートしてraftstore CPUの消費を減らす[#4591](https://github.com/tikv/tikv/pull/4591)
    - リーダーは、learnerの `ReadIndex` のリクエストに対して応答しない問題の修正[#4653](https://github.com/tikv/tikv/pull/4653)
    - いくつかのケースでリーダーの移行に失敗する問題の修正[#4684](https://github.com/tikv/tikv/pull/4684)
    - 一部のケースでダーティリードの問題の修正[#4688](https://github.com/tikv/tikv/pull/4688)
    - 一部のケースでスナップショットが適用されたデータを失う可能性がある問題の修正[#4716](https://github.com/tikv/tikv/pull/4716)

+ Coprocessor
    - さらにRPN関数の追加
        - `LogicalOr` [#4691](https://github.com/tikv/tikv/pull/4601)
        - `LTReal` [#4602](https://github.com/tikv/tikv/pull/4602)
- `LEReal` [#4602](https://github.com/tikv/tikv/pull/4602)
        - `GTReal` [#4602](https://github.com/tikv/tikv/pull/4602)
        - `GEReal` [#4602](https://github.com/tikv/tikv/pull/4602)
        - `NEReal` [#4602](https://github.com/tikv/tikv/pull/4602)
        - `EQReal` [#4602](https://github.com/tikv/tikv/pull/4602)
        - `IsNull` [#4720](https://github.com/tikv/tikv/pull/4720)
        - `IsTrue` [#4720](https://github.com/tikv/tikv/pull/4720)
        - `IsFalse` [#4720](https://github.com/tikv/tikv/pull/4720)
        - `Int`に対する比較演算のサポート [#4625](https://github.com/tikv/tikv/pull/4625)
        - `Decimal`に対する比較演算のサポート [#4625](https://github.com/tikv/tikv/pull/4625)
        - `String`に対する比較演算のサポート [#4625](https://github.com/tikv/tikv/pull/4625)
        - `Time`に対する比較演算のサポート [#4625](https://github.com/tikv/tikv/pull/4625)
        - `Duration`に対する比較演算のサポート [#4625](https://github.com/tikv/tikv/pull/4625)
        - `Json`に対する比較演算のサポート [#4625](https://github.com/tikv/tikv/pull/4625)
        - `Int`に対するプラス演算のサポート [#4733](https://github.com/tikv/tikv/pull/4733)
        - `Real`に対するプラス演算のサポート [#4733](https://github.com/tikv/tikv/pull/4733)
        - `Decimal`に対するプラス演算のサポート [#4733](https://github.com/tikv/tikv/pull/4733)
        - `Int`に対するMOD関数のサポート [#4727](https://github.com/tikv/tikv/pull/4727)
        - `Real`に対するMOD関数のサポート [#4727](https://github.com/tikv/tikv/pull/4727)
        - `Decimal`に対するMOD関数のサポート [#4727](https://github.com/tikv/tikv/pull/4727)
        - `Int`に対するマイナス演算のサポート [#4746](https://github.com/tikv/tikv/pull/4746)
        - `Real`に対するマイナス演算のサポート [#4746](https://github.com/tikv/tikv/pull/4746)
        - `Decimal`に対するマイナス演算のサポート [#4746](https://github.com/tikv/tikv/pull/4746)

## ツール

+ TiDB Binlog
    - データレプリケーションの遅延を追跡するためのメトリクスを追加 [#594](https://github.com/pingcap/tidb-binlog/pull/594)

+ TiDB Lightning

    - シャードされたデータベースとテーブルのマージをサポート [#95](https://github.com/pingcap/tidb-lightning/pull/95)
    - KV書き込みの失敗のリトライ機構を追加 [#176](https://github.com/pingcap/tidb-lightning/pull/176)
    - `table-concurrency`のデフォルト値を6に更新 [#175](https://github.com/pingcap/tidb-lightning/pull/175)
    - 提供されていない場合は自動的に`tidb.pd-addr`と`tidb.port`を検出して必要な構成項目を減らす [#173](https://github.com/pingcap/tidb-lightning/pull/173)