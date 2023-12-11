---
title: TiDB 2.1.6 リリースノート
aliases: ['/docs/dev/releases/release-2.1.6/','/docs/dev/releases/2.1.6/']
---

# TiDB 2.1.6 リリースノート

2019年3月15日、TiDB 2.1.6がリリースされました。対応するTiDB Ansible 2.1.6もリリースされています。このリリースはTiDB 2.1.5と比較して、安定性、SQLオプティマイザ、統計情報、実行エンジンが大幅に改善されています。

## TiDB

+ SQLオプティマイザ/実行エンジン
    - `TIDB_INLJ`のHintで両方のテーブルが指定された場合、コストに基づいて外部テーブルを選択するようにプランナーを最適化 [#9615](https://github.com/pingcap/tidb/pull/9615)
    - 特定の場合に`IndexScan`が正しく選択されない問題を修正 [#9587](https://github.com/pingcap/tidb/pull/9587)
    - サブクエリの`agg`関数でMySQLとの非互換性を修正 [#9551](https://github.com/pingcap/tidb/pull/9551)
    - 有効な列のみを出力するように`show stats_histograms`を修正してパニックを回避 [#9502](https://github.com/pingcap/tidb/pull/9502)

+ サーバ
    - `log_bin`変数をサポートしてBinlogの有効化/無効化を可能にする [#9634](https://github.com/pingcap/tidb/pull/9634)
    - トランザクションのための健全性チェックを追加して誤ったトランザクションのコミットを回避 [#9559](https://github.com/pingcap/tidb/pull/9559)
    - 変数の設定がパニックを引き起こす可能性のある問題を修正 [#9539](https://github.com/pingcap/tidb/pull/9539)

+ DDL
    - 特定の場合に`Create Table Like`ステートメントがパニックを引き起こす問題を修正 [#9652](https://github.com/pingcap/tidb/pull/9652)
    - etcdクライアントの`AutoSync`機能を有効にして、TiDBとetcdの接続問題を回避するための対応 [#9600](https://github.com/pingcap/tidb/pull/9600)

## TiKV

- 特定の場合に`protobuf`の解析の失敗が`StoreNotMatch`エラーを引き起こす問題を修正 [#4303](https://github.com/tikv/tikv/pull/4303)

## Tools

+ Lightning
    - インポーターのデフォルトの`region-split-size`を512 MiBに変更 [#4369](https://github.com/tikv/tikv/pull/4369)
    - メモリにキャッシュされていた中間SSTをローカルディスクに保存してメモリ使用量を削減 [#4369](https://github.com/tikv/tikv/pull/4369)
    - RocksDBのメモリ使用量を制限する対応 [#4369](https://github.com/tikv/tikv/pull/4369)
    - スケジュールが完了する前にリージョンが散在する問題を修正 [#4369](https://github.com/tikv/tikv/pull/4369)
    - 大きなテーブルのデータとインデックスのインポートを分離し、バッチでのインポート時間を効果的に短縮する対応 [#132](https://github.com/pingcap/tidb-lightning/pull/132)
    - CSVのサポート [#111](https://github.com/pingcap/tidb-lightning/pull/111)
    - スキーマ名に非英数字文字が含まれているためのインポート失敗のエラーを修正 [#9547](https://github.com/pingcap/tidb/pull/9547)