---
title: TiDB 2.1.14のリリースノート
aliases: ['/docs/dev/releases/release-2.1.14/','/docs/dev/releases/2.1.14/']
---

# TiDB 2.1.14のリリースノート

リリース日: 2019年7月4日

TiDBバージョン: 2.1.14

TiDB Ansibleバージョン: 2.1.14

## TiDB

- 一部の場合におけるカラム削除による誤ったクエリ結果が修正されました [#11019](https://github.com/pingcap/tidb/pull/11019)
- `show processlist`の`db`と`info`カラムに誤って表示される情報が修正されました [#11000](https://github.com/pingcap/tidb/pull/11000)
- `MAX_EXECUTION_TIME`がSQLヒントおよびグローバル変数として一部の場合に機能しない問題が修正されました [#10999](https://github.com/pingcap/tidb/pull/10999)
- 負荷に基づいて自動的に割り当てられた自動インクリメントIDによる増分ギャップをサポートするようになりました [#10997](https://github.com/pingcap/tidb/pull/10997)
- クエリ終了時に`MemTracker`の`Distsql`メモリ情報を正しくクリーンアップしない問題が修正されました [#10971](https://github.com/pingcap/tidb/pull/10971)
- `information_schema.processlist`テーブルに`MEM`カラムが追加され、クエリのメモリ使用を記述するようになりました [#10896](https://github.com/pingcap/tidb/pull/10896)
- `max_execution_time`グローバルシステム変数がクエリの最大実行時間を制御するために追加されました [#10940](https://github.com/pingcap/tidb/pull/10940)
- サポートされていない集約関数を使用することによって発生するパニックが修正されました [#10911](https://github.com/pingcap/tidb/pull/10911)
- `load data`ステートメントが失敗した際に最後のトランザクションの自動ロールバック機能が追加されました [#10862](https://github.com/pingcap/tidb/pull/10862)
- `OOMAction`構成項目が`Cancel`に設定された場合にTiDBが一部の場合に誤った結果を返す問題が修正されました [#11016](https://github.com/pingcap/tidb/pull/11016)
- TiDBパニックの問題を避けるために`TRACE`ステートメントを無効化しました [#11039](https://github.com/pingcap/tidb/pull/11039)
- 特定の関数をCoprocessorにプッシュダウンするかどうかを動的に有効または無効にする`mysql.expr_pushdown_blacklist`システムテーブルが追加されました [#10998](https://github.com/pingcap/tidb/pull/10998)
- `ONLY_FULL_GROUP_BY`モードで`ANY_VALUE`関数が機能しない問題が修正されました [#10994](https://github.com/pingcap/tidb/pull/10994)
- 文字列型のユーザ変数を評価する際に深いコピーを行わないことによって引き起こされる評価の不正確さが修正されました [#11043](https://github.com/pingcap/tidb/pull/11043)

## TiKV

- 不要なメッセージを送信することを避けるために、Raftstoreメッセージの処理中に空コールバックを最適化しました [#4682](https://github.com/tikv/tikv/pull/4682)

## PD

- 無効な構成項目を読み込む際にログの出力レベルを`Error`から`Warning`に調整しました [#1577](https://github.com/pingcap/pd/pull/1577)

## Tools

TiDB Binlog

- Reparo
    - `safe-mode`構成項目を追加し、この構成項目が有効になってから重複データをインポートするサポートが追加されました [#662](https://github.com/pingcap/tidb-binlog/pull/662)
- Pump
    - 利用可能なバイナリログスペースが構成値よりも少ない場合にPumpでのバイナリログファイルの書き込みを停止する`stop-write-at-available-space`構成項目が追加されました [#659](https://github.com/pingcap/tidb-binlog/pull/659)
    - LevelDB L0ファイルの数が0の場合に時々Garbage Collectorが機能しない問題が修正されました [#648](https://github.com/pingcap/tidb-binlog/pull/648)
    - 空き容量を解放する処理の高速化のためにログファイルの削除アルゴリズムを最適化しました [#648](https://github.com/pingcap/tidb-binlog/pull/648)
- Drainer
    - 下流で`BIT`列を更新できない問題が修正されました [#655](https://github.com/pingcap/tidb-binlog/pull/655)

## TiDB Ansible

- `ansible`コマンドとその`jmespath`および`jinja2`依存パッケージのための事前チェック機能が追加されました [#807](https://github.com/pingcap/tidb-ansible/pull/807)
- Pumpに10 GiB（デフォルト値）の`stop-write-at-available-space`パラメータが追加され、利用可能なディスク容量がこのパラメータ値よりも少ない場合はPumpでバイナリログファイルの書き込みを停止します [#807](https://github.com/pingcap/tidb-ansible/pull/807)