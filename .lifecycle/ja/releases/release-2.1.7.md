---
title: TiDB 2.1.7 リリースノート
aliases: ['/docs/dev/releases/release-2.1.7/','/docs/dev/releases/2.1.7/']
---

# TiDB 2.1.7 リリースノート

リリース日: 2019年3月28日

TiDB バージョン: 2.1.7

TiDB Ansible バージョン: 2.1.7

## TiDB

- アップグレードプログラムによってキャンセルされたDDL操作による起動時間の長さの問題を修正 [#9768](https://github.com/pingcap/tidb/pull/9768)
- `config.example.toml`ファイルで`check-mb4-value-in-utf8`構成項目が間違った位置にある問題を修正 [#9852](https://github.com/pingcap/tidb/pull/9852)
- `str_to_date` 組み込み関数のMySQL互換性を向上 [#9817](https://github.com/pingcap/tidb/pull/9817)
- `last_day` 組み込み関数の互換性問題を修正 [#9750](https://github.com/pingcap/tidb/pull/9750)
- SQLステートメントを使用して`tidb_table_id`列を`infoschema.tables`に追加し、`tidb_indexes`システムテーブルを追加してTableとIndexの関係を管理する機能を提供 [#9862](https://github.com/pingcap/tidb/pull/9862)
- テーブルパーティションのNULL定義に関するチェックを追加 [#9663](https://github.com/pingcap/tidb/pull/9663)
- `Truncate Table`に必要な権限を`Delete`から`Drop`に変更し、MySQLと一貫性を持たせ [#9876](https://github.com/pingcap/tidb/pull/9876)
- `DO`ステートメントでのサブクエリ使用をサポート [#9877](https://github.com/pingcap/tidb/pull/9877)
- `week`関数で`default_week_format`変数が効果をもたない問題を修正 [#9753](https://github.com/pingcap/tidb/pull/9753)
- プラグインフレームワークをサポート [#9880](https://github.com/pingcap/tidb/pull/9880), [#9888](https://github.com/pingcap/tidb/pull/9888)
- `log_bin`システム変数を使用してbinlogの有効化状態の確認をサポート [#9634](https://github.com/pingcap/tidb/pull/9634)
- SQLステートメントを使用してPump/Drainerの状態を確認する機能をサポート [#9896](https://github.com/pingcap/tidb/pull/9896)
- TiDBのアップグレード時にutf8でのmb4文字の確認に関する互換性問題を修正 [#9887](https://github.com/pingcap/tidb/pull/9887)
- 一部の場合において、集約関数がJSONデータを計算する際のパニック問題を修正 [#9927](https://github.com/pingcap/tidb/pull/9927)

## PD

- レプリカ数が1の場合において、balance-regionにおいてleaderステップが作成されない問題を修正 [#1462](https://github.com/pingcap/pd/pull/1462)

## Tools

- binlogを使用して生成された列をレプリケートする機能をサポート

## TiDB Ansible

Prometheus監視データのデフォルト保持期間を30日に変更