---
title: TiDB 2.1.11 リリースノート
aliases: ['/docs/dev/releases/release-2.1.11/','/docs/dev/releases/2.1.11/']
---

# TiDB 2.1.11 リリースノート

リリース日: 2019年6月3日

TiDB バージョン: 2.1.11

TiDB Ansible バージョン: 2.1.11

## TiDB

- `delete from join` で不正なスキーマが使用される問題を修正 [#10595](https://github.com/pingcap/tidb/pull/10595)
- 組み込みの `CONVERT()` が間違ったフィールドタイプを返す可能性がある問題を修正 [#10263](https://github.com/pingcap/tidb/pull/10263)
- バケツカウントの更新時に重複しないフィードバックをマージするように修正 [#10569](https://github.com/pingcap/tidb/pull/10569)
- `unix_timestamp()-unix_timestamp(now())` の計算エラーを修正 [#10491](https://github.com/pingcap/tidb/pull/10491)
- MySQL 8.0 との `period_diff` の非互換性の問題を修正 [#10501](https://github.com/pingcap/tidb/pull/10501)
- 例外を回避するため、統計情報の収集時に `Virtual Column` をスキップするよう修正 [#10628](https://github.com/pingcap/tidb/pull/10628)
- `SHOW OPEN TABLES` ステートメントをサポート [#10374](https://github.com/pingcap/tidb/pull/10374)
- 特定のケースでゴルーチンリークが発生する可能性がある問題を修正 [#10656](https://github.com/pingcap/tidb/pull/10656)
- 特定のケースで `tidb_snapshot` 変数を設定すると、時間形式の解析が誤る可能性がある問題を修正 [#10637](https://github.com/pingcap/tidb/pull/10637)

## PD

- `balance-region` に起因するホットリージョンのスケジュールが失敗する可能性がある問題を修正 [#1551](https://github.com/pingcap/pd/pull/1551)
- ホットスポットに関連するスケジューリングの優先度を高く設定 [#1551](https://github.com/pingcap/pd/pull/1551)
- 2つの構成項目を追加 [#1551](https://github.com/pingcap/pd/pull/1551)
    - 同時に実行されるホットスポットスケジューリングタスクの最大数を制御するための `hot-region-schedule-limit`
    - ホットリージョンを識別するための `hot-region-cache-hits-threshold`

## TiKV

- リーダーとリーダーが1つだけの場合にリーダーが空のインデックスを読み込む問題を修正 [#4751](https://github.com/tikv/tikv/pull/4751)
- `ScanLock` および `ResolveLock` を通常の優先度のコマンドへの影響を減らすために、スレッドプールで高い優先度で処理するよう修正 [#4791](https://github.com/tikv/tikv/pull/4791)
- 受信したスナップショットのすべてのファイルを同期 [#4811](https://github.com/tikv/tikv/pull/4811)

## ツール

- TiDB Binlog
    - `WritePause` によるQPSの低下を避けるため、GC中のデータ削除速度を制限 [#620](https://github.com/pingcap/tidb-binlog/pull/620)

## TiDB Ansible

- Drainer パラメータを追加 [#760](https://github.com/pingcap/tidb-ansible/pull/760)