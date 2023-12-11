---
title: TiDB 2.1.1 リリースノート
aliases: ['/docs/dev/releases/release-2.1.1/','/docs/dev/releases/2.1.1/']
---

# TiDB 2.1.1 リリースノート

2018年12月12日にTiDB 2.1.1がリリースされました。このリリースはTiDB 2.1.0と比較して、安定性、SQLオプティマイザ、統計情報、および実行エンジンに大幅な改善があります。

## TiDB

+ SQLオプティマイザ/エグゼキュータ
    - 負の日付の丸め誤差を修正 [#8574](https://github.com/pingcap/tidb/pull/8574)
    - `uncompress` 関数がデータ長をチェックしない問題を修正 [#8606](https://github.com/pingcap/tidb/pull/8606)
    - `execute` コマンドの後に `prepare` ステートメントのバインド引数をリセットするよう修正 [#8652](https://github.com/pingcap/tidb/pull/8652)
    - パーティションテーブルの統計情報を自動的に収集するようサポート [#8649](https://github.com/pingcap/tidb/pull/8649)
    - `abs` 関数のプッシュダウン時に誤って構成された整数型を修正 [#8628](https://github.com/pingcap/tidb/pull/8628)
    - JSON列のデータ競合を修正 [#8660](https://github.com/pingcap/tidb/pull/8660)
+ サーバ
    - PDがダウンしたときにトランザクションが誤ったTSOを取得する問題を修正 [#8567](https://github.com/pingcap/tidb/pull/8567)
    - ANSI標準に準拠しないステートメントによるブートストラップの失敗を修正 [#8576](https://github.com/pingcap/tidb/pull/8576)
    - トランザクションのリトライで誤ったパラメータが使用される問題を修正 [#8638](https://github.com/pingcap/tidb/pull/8638)
+ DDL
    - テーブルのデフォルトの文字セットと照合順序を `utf8mb4` に変更 [#8590](https://github.com/pingcap/tidb/pull/8590)
    - インデックスの追加速度を制御するために `ddl_reorg_batch_size` 変数を追加 [#8614](https://github.com/pingcap/tidb/pull/8614)
    - DDLの文字セットと照合順序オプションの内容を大文字/小文字を区別しないように修正 [#8611](https://github.com/pingcap/tidb/pull/8611)
    - 生成された列にインデックスを追加する問題を修正 [#8655](https://github.com/pingcap/tidb/pull/8655)

## PD

- いくつかの設定項目を構成ファイルで `0` に設定できない問題を修正 [#1334](https://github.com/pingcap/pd/pull/1334)
- PDを起動する際に未定義の設定をチェックするように修正 [#1362](https://github.com/pingcap/pd/pull/1362)
- 新たに作成されたピアにリーダーを転送しないように、可能な遅延を最適化するための修正 [#1339](https://github.com/pingcap/pd/pull/1339)
- デッドロックにより `RaftCluster` を停止できなくなる問題を修正 [#1370](https://github.com/pingcap/pd/pull/1370)

## TiKV

- 新たに作成されたピアにリーダーを転送しないように、可能な遅延を最適化するための修正 [#3878](https://github.com/tikv/tikv/pull/3878)

## Tools

+ Lightning
    - インポートされたテーブルの `analyze` メカニズムを最適化してインポート速度を向上させる
    - チェックポイント情報をローカルファイルに保存するようサポート
+ TiDB Binlog
    - プライマリキー列のみを持つテーブルはpbイベントを生成できないのバグを修正