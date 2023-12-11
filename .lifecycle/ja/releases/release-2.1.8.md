---
title: TiDB 2.1.8 リリースノート
aliases: ['/docs/dev/releases/release-2.1.8/','/docs/dev/releases/2.1.8/']
---

# TiDB 2.1.8 リリースノート

リリース日: 2019年4月12日

TiDB バージョン: 2.1.8

TiDB Ansible バージョン: 2.1.8

## TiDB

- `GROUP_CONCAT` 関数の処理ロジックが MySQL と互換性がない場合に発生する問題を修正 [#9930](https://github.com/pingcap/tidb/pull/9930)
- `Distinct` モードにおける10進数値の等価チェックの問題を修正 [#9931](https://github.com/pingcap/tidb/pull/9931)
- `SHOW FULL COLUMNS` ステートメントにおける日付、日時、タイムスタンプ型の照合互換性の問題を修正
    - [#9938](https://github.com/pingcap/tidb/pull/9938)
    - [#10114](https://github.com/pingcap/tidb/pull/10114)
- フィルタリング条件に相関カラムが含まれる場合の行数推定が不正確である問題を修正 [#9937](https://github.com/pingcap/tidb/pull/9937)
- `DATE_ADD` および `DATE_SUB` 関数の互換性の問題を修正
    - [#9963](https://github.com/pingcap/tidb/pull/9963)
    - [#9966](https://github.com/pingcap/tidb/pull/9966)
- `STR_TO_DATE` 関数に `%H` フォーマットをサポートして互換性を向上させる問題を修正 [#9964](https://github.com/pingcap/tidb/pull/9964)
- `GROUP_CONCAT` 関数がユニークインデックスによってグループ化される場合に結果が誤っていた問題を修正 [#9969](https://github.com/pingcap/tidb/pull/9969)
- オプティマイザヒントに一致しないテーブル名が含まれている場合に警告を返すよう修正 [#9970](https://github.com/pingcap/tidb/pull/9970)
- 分析ツールを使用してログを収集するためにログ形式を統一するように修正 Unified Log Format
- 大量のNULL値が誤った統計推定を引き起こす問題を修正 [#9979](https://github.com/pingcap/tidb/pull/9979)
- TIMESTAMP 型のデフォルト値が境界値の場合にエラーが報告される問題を修正 [#9987](https://github.com/pingcap/tidb/pull/9987)
- `time_zone` の値を検証するように修正 [#10000](https://github.com/pingcap/tidb/pull/10000)
- `2019.01.01` 時刻フォーマットをサポートするように修正 [#10001](https://github.com/pingcap/tidb/pull/10001)
- `EXPLAIN` ステートメントによって返された結果において特定の場合に行数推定が誤って表示される問題を修正 [#10044](https://github.com/pingcap/tidb/pull/10044)
- 特定の場合に `KILL TIDB [session id]` がステートメントの実行を即座に停止できない問題を修正 [#9976](https://github.com/pingcap/tidb/pull/9976)
- 特定の場合に定数フィルタリング条件のプッシュダウン問題を修正 [#10049](https://github.com/pingcap/tidb/pull/10049)
- 特定の場合に読み取り専用ステートメントが正しく処理されない問題を修正 [#10048](https://github.com/pingcap/tidb/pull/10048)

## PD

- `regionScatterer` が無効な `OperatorStep` を生成する可能性がある問題を修正 [#1482](https://github.com/pingcap/pd/pull/1482)
- ホットストアがキーの不正確な統計を作成する問題を修正 [#1487](https://github.com/pingcap/pd/pull/1487)
- `MergeRegion` オペレーターのタイムアウトが短すぎる問題を修正 [#1495](https://github.com/pingcap/pd/pull/1495)
- PD サーバーが TSO リクエストを処理する際の経過時間メトリクスを追加 [#1502](https://github.com/pingcap/pd/pull/1502)

## TiKV

- 誤った読み取りトラフィックの統計情報の問題を修正 [#4441](https://github.com/tikv/tikv/pull/4441)
- 多数のリージョンが存在する場合の raftstore パフォーマンスの問題を修正 [#4484](https://github.com/tikv/tikv/pull/4484)
- level 0 SST ファイルの数が `level_zero_slowdown_writes_trigger/2` を超えた場合にファイルを挿入しないように修正 [#4464](https://github.com/tikv/tikv/pull/4464)

## Tools

- Lightning におけるテーブルのインポート順序を最適化し、大規模なテーブルの `Checksum` および `Analyze` の実行がインポートプロセス中にクラスターに影響を及ぼすことを減らし、 `Checksum` および `Analyze` の成功率を向上させるためにSQLパファイル内容を直接 `types.Datum` に解析することでエンコード SQL のパフォーマンスを50%改善 [#156](https://github.com/pingcap/tidb-lightning/pull/156)
- KV エンコーダの追加の解析作業を回避するために Lightning のデータソースファイルの内容を直接解析することでエンコード SQL のパフォーマンスを50%改善 [#145](https://github.com/pingcap/tidb-lightning/pull/145)
- ローカルストレージのディスクを非同期でフラッシュするための `storage.sync-log` 設定項目を TiDB Binlog Pump に追加 [#529](https://github.com/pingcap/tidb-binlog/pull/529)
- TiDB Binlog Pump と Drainer 間の通信のトラフィック圧縮をサポートするように修正 [#530](https://github.com/pingcap/tidb-binlog/pull/530)
- DDL クエリを解析するために異なる `sql-mode` を使用する機能をサポートするための `syncer.sql-mode` 設定項目を TiDB Binlog Drainer に追加 [#513](https://github.com/pingcap/tidb-binlog/pull/513)
- レプリケートされないテーブルをフィルタリングするための `syncer.ignore-table` 設定項目を TiDB Binlog Drainer に追加 [#526](https://github.com/pingcap/tidb-binlog/pull/526)

## TiDB Ansible

- オペレーティングシステムのバージョン制限を変更し、CentOS 7.0 以降および Red Hat 7.0 以降のみをサポートするように修正 [#734](https://github.com/pingcap/tidb-ansible/pull/734)
- 各 OS で `epollexclusive` がサポートされているかどうかをチェックする機能を追加 [#728](https://github.com/pingcap/tidb-ansible/pull/728)
- ローリングアップデートのためのバージョン制限を追加し、バージョン2.0.1以前からバージョン2.1以降へのアップグレードを禁止する [#728](https://github.com/pingcap/tidb-ansible/pull/728)