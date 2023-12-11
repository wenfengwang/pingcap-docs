---
title: TiDB 2.1 RC4 リリースノート
aliases: ['/docs/dev/releases/release-2.1-rc.4/','/docs/dev/releases/21rc4/']
---

# TiDB 2.1 RC4 リリースノート

2018年10月23日、TiDB 2.1 RC4 がリリースされました。TiDB 2.1 RC3 と比較して、このリリースでは安定性、SQL オプティマイザ、統計情報、および実行エンジンの大幅な改善が行われています。

## TiDB

+ SQL オプティマイザ
    - `UnionAll` の列の剪定が一部のケースで正しくない問題を修正しました [#7941](https://github.com/pingcap/tidb/pull/7941)
    - `UnionAll` 演算子の結果が一部のケースで正しくない問題を修正しました [#8007](https://github.com/pingcap/tidb/pull/8007)
+ SQL 実行エンジン
    - `AVG` 関数の精度の問題を修正しました [#7874](https://github.com/pingcap/tidb/pull/7874)
    - クエリ実行プロセス中に各演算子の実行時間と返された行数などの実行時統計をチェックするために `EXPLAIN ANALYZE` ステートメントをサポートしました [#7925](https://github.com/pingcap/tidb/pull/7925)
    - 結果セット内でテーブルの列が複数回表示された場合に `PointGet` 演算子のパニック問題を修正しました [#7943](https://github.com/pingcap/tidb/pull/7943)
    - `Limit` のサブ句における大きすぎる値によって引き起こされるパニック問題を修正しました [#8002](https://github.com/pingcap/tidb/pull/8002)
    - 特定のケースにおける `AddDate`/`SubDate` ステートメントの実行プロセス中に発生するパニック問題を修正しました [#8009](https://github.com/pingcap/tidb/pull/8009)
+ 統計
    - 結合インデックスのヒストグラムの下限のプレフィックスを範囲外と判断する問題を修正しました [#7856](https://github.com/pingcap/tidb/pull/7856)
    - 統計収集によって引き起こされるメモリリーク問題を修正しました [#7873](https://github.com/pingcap/tidb/pull/7873)
    - ヒストグラムが空の場合のパニック問題を修正しました [#7928](https://github.com/pingcap/tidb/pull/7928)
    - 統計がアップロード中にヒストグラムの境界が範囲外になる問題を修正しました [#7944](https://github.com/pingcap/tidb/pull/7944)
    - 統計サンプリングプロセス中の値の最大長を制限しました [#7982](https://github.com/pingcap/tidb/pull/7982)
+ サーバー
    - トランザクションの競合の誤判断を回避し、並行トランザクションの実行パフォーマンスを改善するために Latch をリファクタリングしました [#7711](https://github.com/pingcap/tidb/pull/7711)
    - 特定のケースにおける遅いクエリの収集によって発生するパニック問題を修正しました [#7874](https://github.com/pingcap/tidb/pull/7847)
    - `LOAD DATA` ステートメントにおける `ESCAPED BY` が空の文字列である場合に発生するパニック問題を修正しました [#8005](https://github.com/pingcap/tidb/pull/8005)
    - “coprocessor エラー” ログ情報を完了しました [#8006](https://github.com/pingcap/tidb/pull/8006)
+ 互換性
    - クエリが空の場合に `SHOW PROCESSLIST` 結果の `Command` フィールドを `Sleep` に設定しました [#7839](https://github.com/pingcap/tidb/pull/7839)
+ Expressions
    - `SYSDATE` 関数の定数畳み込みの問題を修正しました [#7895](https://github.com/pingcap/tidb/pull/7895)
    - 特定のケースにおいて `SUBSTRING_INDEX` がパニックする問題を修正しました [#7897](https://github.com/pingcap/tidb/pull/7897)
+ DDL
    - `invalid ddl job type` エラーをスローしたことによるスタックオーバーフロー問題を修正しました [#7958](https://github.com/pingcap/tidb/pull/7958)
    - 特定のケースにおいて `ADMIN CHECK TABLE` の結果が正しくない問題を修正しました [#7975](https://github.com/pingcap/tidb/pull/7975)

## PD

- Tombstone TiKV が Grafana から削除されない問題を修正しました [#1261](https://github.com/pingcap/pd/pull/1261)
- grpc-go がステータスを設定する際にデータ競合問題を修正しました [#1265](https://github.com/pingcap/pd/pull/1265)
- etcd の起動失敗によって PD サーバーが立ち往生する問題を修正しました [#1267](https://github.com/pingcap/pd/pull/1267)
- リーダー切り替え中にデータ競合が発生する可能性がある問題を修正しました [#1273](https://github.com/pingcap/pd/pull/1273)
- TiKV が Tombstone になった時に余分な警告ログが出力される可能性がある問題を修正しました [#1280](https://github.com/pingcap/pd/pull/1273)

## TiKV

- スナップショットの適用によって引き起こされる RocksDB の書き込みスタールの問題を最適化しました [#3606](https://github.com/tikv/tikv/pull/3606)
- raftstore `tick` メトリクスを追加しました [#3657](https://github.com/tikv/tikv/pull/3657)
- RocksDB をアップグレードし、`IngestExternalFile` を実行する際に書き込みブロックの問題とソースファイルが書き込み操作で破損する可能性がある問題を修正しました [#3661](https://github.com/tikv/tikv/pull/3661)
- grpcio をアップグレードし、“too many pings” が誤って報告される問題を修正しました [#3650](https://github.com/tikv/tikv/pull/3650)
