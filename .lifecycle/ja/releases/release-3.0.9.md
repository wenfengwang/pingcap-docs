---
title: TiDB 3.0.9リリースノート
aliases: ['/docs/dev/releases/release-3.0.9/','/docs/dev/releases/3.0.9/']
---

# TiDB 3.0.9リリースノート

リリース日: 2020年1月14日

TiDBバージョン: 3.0.9

TiDB Ansibleバージョン: 3.0.9

> **警告:**
>
> このバージョンでいくつかの既知の問題が見つかっており、これらの問題は新しいバージョンで修正されています。最新の3.0.xバージョンをご利用いただくことをお勧めします。

## TiDB

+ Executor
    - `ENUM`列やコレクション列に集約関数が適用された場合に、結果が正しくない問題を修正 [#14364](https://github.com/pingcap/tidb/pull/14364)
+ Server
    - `auto_increment_increment`および`auto_increment_offset`システム変数をサポート [#14396](https://github.com/pingcap/tidb/pull/14396)
    - 10分間のTTLを持つ悲観的トランザクションの数を監視するための`tidb_tikvclient_ttl_lifetime_reach_total`監視メトリクスを追加 [#14300](https://github.com/pingcap/tidb/pull/14300)
    - SQLクエリの実行中にSQL情報をログに出力するように修正 [#14322](https://github.com/pingcap/tidb/pull/14322)
    - 実行中の`plan`および`plan`シグネチャを記録するためのステートメントサマリーテーブルに`plan`および`plan_digest`フィールドを追加 [#14285](https://github.com/pingcap/tidb/pull/14285)
    - `stmt-summary.max-stmt-count`構成項目のデフォルト値を`100`から`200`に調整 [#14285](https://github.com/pingcap/tidb/pull/14285)
    - `plan`シグネチャを記録するための遅いクエリテーブルに`plan_digest`フィールドを追加 [#14292](https://github.com/pingcap/tidb/pull/14292)
+ DDL
    - `alter table ... add index`で`primary`列を使用して匿名インデックスが作成された場合の結果がMySQLと一貫していない問題を修正 [#14310](https://github.com/pingcap/tidb/pull/14310)
    - `drop table`構文によって`VIEW`が誤って削除される問題を修正 [#14052](https://github.com/pingcap/tidb/pull/14052)
+ Planner
    - `select max(a), min(a) from t`などのステートメントのパフォーマンスを最適化。`a`列にインデックスが存在する場合、ステートメントを`select * from (select a from t order by a desc limit 1) as t1, (select a from t order by a limit 1) as t2`に最適化し、全テーブル走査を回避 [#14410](https://github.com/pingcap/tidb/pull/14410)

## TiKV

+ Raftstore
    - 構成変更を高速化してリージョンの散らばりを速めるように修正 [#6421](https://github.com/tikv/tikv/pull/6421)
+ Transaction
    - `tikv_lock_manager_waiter_lifetime_duration`、`tikv_lock_manager_detect_duration`、および`Wait`テーブルのステータスを監視するための`tikv_lock_manager_detect_duration`監視メトリクスを追加 [#6392](https://github.com/tikv/tikv/pull/6392)
    - リージョンリーダーの変更や極端な状況でのデッドロック検出器のリーダーの変更によるトランザクション実行の遅延を減少させるために、以下の構成項目を最適化 [#6429](https://github.com/tikv/tikv/pull/6429)
        - `wait-for-lock-time`のデフォルト値を`3s`から`1s`に変更
        - `wake-up-delay-duration`のデフォルト値を`100ms`から`20ms`に変更
    - リージョンマージプロセス中にデッドロック検出器のリーダーが正しくない可能性がある問題を修正 [#6431](https://github.com/tikv/tikv/pull/6431)

## PD

+ 位置ラベル名でバックスラッシュ`/`を使用するサポートを追加 [#2083](https://github.com/pingcap/pd/pull/2083)
+ タイムスタンプストアが誤ってラベルカウンタに含まれるための不正確な統計を修正 [#2067](https://github.com/pingcap/pd/pull/2067)

## ツール

+ TiDB Binlog
    - Drainerによって出力されるバイナリログプロトコルにユニークキー情報を追加 [#862](https://github.com/pingcap/tidb-binlog/pull/862)
    - Drainerでデータベース接続に暗号化されたパスワードを使用するサポートを追加 [#868](https://github.com/pingcap/tidb-binlog/pull/868)

## TiDB Ansible

+ TiDB Lightningのデプロイを最適化するためにディレクトリを自動的に作成するサポートを追加 [#1105](https://github.com/pingcap/tidb-ansible/pull/1105)