---
title: TiDB 2.1.10 リリースノート
aliases: ['/docs/dev/releases/release-2.1.10/','/docs/dev/releases/2.1.10/']
---

# TiDB 2.1.10 リリースノート

リリース日: 2019年5月22日

TiDB バージョン: 2.1.10

TiDB Ansible バージョン: 2.1.10

## TiDB

- `tidb_snapshot` を使用して履歴データを読む際に、一部の異常が原因で間違ったテーブルスキーマが発生する問題を修正しました [#10359](https://github.com/pingcap/tidb/pull/10359)
- 特定のケースで `NOT` 関数が誤った読み取り結果を引き起こす問題を修正しました [#10363](https://github.com/pingcap/tidb/pull/10363)
- `Replace` や `Insert on duplicate update` 文における `Generated Column` の誤った動作を修正しました [#10385](https://github.com/pingcap/tidb/pull/10385)
- `DATE`/`DATETIME` の比較において `BETWEEN` 関数に関するバグを修正しました [#10407](https://github.com/pingcap/tidb/pull/10407)
- `SLOW_QUERY` テーブルを使用して遅いクエリをクエリする際に、1行が長すぎる場合にエラーが発生する問題を修正しました [#10412](https://github.com/pingcap/tidb/pull/10412)
- `DATETIME` に `INTERVAL` を加算した結果がMySQLと一致しない場合がある問題を修正しました [#10416](https://github.com/pingcap/tidb/pull/10416), [#10418](https://github.com/pingcap/tidb/pull/10418)
- 閏年の2月の無効な日付に対するチェックを追加しました [#10417](https://github.com/pingcap/tidb/pull/10417)
- クラスタの初期化時に大量の競合エラーが発生するのを避けるために、DDL オーナーのみが内部初期化操作の制限を実行するように修正しました [#10426](https://github.com/pingcap/tidb/pull/10426)
- 出力タイムスタンプ列のデフォルト値が `default current_timestamp on update current_timestamp` の場合に、`DESC` がMySQLと互換性がない問題を修正しました [#10337](https://github.com/pingcap/tidb/issues/10337)
- `Update` 文における権限チェック中にエラーが発生する問題を修正しました [#10439](https://github.com/pingcap/tidb/pull/10439)
- `CHAR` 列において `RANGE` の誤った計算が特定のケースで誤った結果を引き起こす問題を修正しました [#10455](https://github.com/pingcap/tidb/pull/10455)
- `SHARD_ROW_ID_BITS` を減少させた後にデータが上書きされる可能性がある問題を修正しました [#9868](https://github.com/pingcap/tidb/pull/9868)
- `ORDER BY RAND()` がランダムな数値を返さない問題を修正しました [#10064](https://github.com/pingcap/tidb/pull/10064)
- 小数の精度を修正する `ALTER` 文を禁止しました [#10458](https://github.com/pingcap/tidb/pull/10458)
- `TIME_FORMAT` 関数のMySQLとの互換性の問題を修正しました [#10474](https://github.com/pingcap/tidb/pull/10474)
- `PERIOD_ADD` のパラメータの有効性をチェックするようにしました [#10430](https://github.com/pingcap/tidb/pull/10430)
- TiDB における無効な `YEAR` 文字列の挙動がMySQLと互換性がない問題を修正しました [#10493](https://github.com/pingcap/tidb/pull/10493)
- `ALTER DATABASE` 構文をサポートするようにしました [#10503](https://github.com/pingcap/tidb/pull/10503)
- 遅いクエリをクエリする際に `SLOW_QUERY` メモリエンジンがステートメント内に `;` が存在しない場合にエラーが発生する問題を修正しました [#10536](https://github.com/pingcap/tidb/pull/10536)
- 分割されたテーブルにおける `Add index` 操作が特定のケースでキャンセルできない問題を修正しました [#10533](https://github.com/pingcap/tidb/pull/10533)
- 特定のケースでOOMパニックが回復しない問題を修正しました [#10545](https://github.com/pingcap/tidb/pull/10545)
- DDL 操作がテーブルメタデータを書き換える際のセキュリティを向上しました [#10547](https://github.com/pingcap/tidb/pull/10547)

## PD

- リーダーの優先度が影響を与えない問題を修正しました [#1533](https://github.com/pingcap/pd/pull/1533)

## TiKV

- 直近で構成が変更されたリージョンのリーダーの転送を拒否して転送の失敗を避けるように修正しました [#4684](https://github.com/tikv/tikv/pull/4684)
- Coprocessor メトリクス用の優先度ラベルを追加しました [#4643](https://github.com/tikv/tikv/pull/4643)
- リーダーを転送する際の可能な読み取り不整合問題を修正しました [#4724](https://github.com/tikv/tikv/pull/4724)
- `CommitMerge` がTiKVの再起動に失敗する問題を修正しました [#4615](https://github.com/tikv/tikv/pull/4615)
- 不明なログを修正しました [#4730](https://github.com/tikv/tikv/pull/4730)

## Tools

- TiDB Lightning
    - `tidb_snapshot` fails to send data to `importer` の再試行機能を追加しました [#176](https://github.com/pingcap/tidb-lightning/pull/176)
- TiDB Binlog
    - Pumpストレージログを最適化してトラブルシューティングを容易にしました [#607](https://github.com/pingcap/tidb-binlog/pull/607)

## TiDB Ansible

- TiDB Lightningの構成ファイルを更新し、`tidb_lightning_ctl` スクリプトを追加しました [#d3a4a368](https://github.com/pingcap/tidb-ansible/commit/d3a4a368810a421c49980899a286cf010569b4c7)