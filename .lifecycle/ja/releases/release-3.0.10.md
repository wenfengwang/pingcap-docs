---
title: TiDB 3.0.10リリースノート
aliases: ['/docs/dev/releases/release-3.0.10/','/docs/dev/releases/3.0.10/']
---

# TiDB 3.0.10リリースノート

リリース日: 2020年2月20日

TiDBバージョン: 3.0.10

TiDB Ansibleバージョン: 3.0.10

> **警告:**
>
> このバージョンでいくつかの既知の問題が見つかっており、これらの問題は新しいバージョンで修正されています。最新の3.0.xバージョンを使用することをお勧めします。

## TiDB

- `IndexLookUpJoin`が`OtherCondition`を使用して`InnerRange`を構築する際に間違った`Join`結果を修正します[#14599](https://github.com/pingcap/tidb/pull/14599)
- `tidb_pprof_sql_cpu`構成項目を削除し、`tidb_pprof_sql_cpu`変数を追加します[#14416](https://github.com/pingcap/tidb/pull/14416)
- グローバル権限を持っている場合にのみユーザーがすべてのデータベースをクエリできる問題を修正します[#14386](https://github.com/pingcap/tidb/pull/14386)
- `PointGet`操作を実行する際にトランザクションのタイムアウトによりデータの可視性が期待通りでない問題を修正します[#14480](https://github.com/pingcap/tidb/pull/14480)
- 悲観的トランザクションのアクティベーションのタイミングを遅延アクティベーションに変更し、楽観的トランザクションモードと一貫性を持たせます[#14474](https://github.com/pingcap/tidb/pull/14474)
- `unixtimestamp`式がテーブルパーティションのタイムゾーンを計算する際のタイムゾーンの不正確な結果を修正します[#14476](https://github.com/pingcap/tidb/pull/14476)
- デッドロック検出時間を監視するための`tidb_session_statement_deadlock_detect_duration_seconds`監視項目を追加します[#14484](https://github.com/pingcap/tidb/pull/14484)
- GCワーカーの一部のロジックエラーによって引き起こされるシステムパニック問題を修正します[#14439](https://github.com/pingcap/tidb/pull/14439)
- `IsTrue`関数の式名を修正します[#14516](https://github.com/pingcap/tidb/pull/14516)
- 一部のメモリ使用量が正確にカウントされない問題を修正します[#14533](https://github.com/pingcap/tidb/pull/14533)
- CM-Sketch統計の初期化時における不正確な処理ロジックによって引き起こされるシステムパニック問題を修正します[#14470](https://github.com/pingcap/tidb/pull/14470)
- パーティションされたテーブルをクエリする際の不正確なパーティションプルーニングの問題を修正します[#14546](https://github.com/pingcap/tidb/pull/14546)
- SQLバインディング内のSQLステートメントのデフォルトデータベース名が誤って設定される問題を修正します[#14548](https://github.com/pingcap/tidb/pull/14548)
- `json_key`がMySQLと互換性がない問題を修正します[#14561](https://github.com/pingcap/tidb/pull/14561)
- パーティションされたテーブルの統計を自動的に更新する機能を追加します[#14566](https://github.com/pingcap/tidb/pull/14566)
- `PointGet`操作を実行した際のプランIDの変更を修正します（プランIDは常に`1`になることが期待されます）[#14595](https://github.com/pingcap/tidb/pull/14595)
- SQLバインディングが完全に一致しない場合に引き起こされる不正確な処理ロジックによるシステムパニック問題を修正します[#14263](https://github.com/pingcap/tidb/pull/14263)
- 悲観的トランザクションのロック失敗後のリトライ回数を監視するための`tidb_session_statement_pessimistic_retry_count`監視項目を追加します[#14619](https://github.com/pingcap/tidb/pull/14619)
- `show binding`ステートメントのための誤った特権チェックを修正します[#14618](https://github.com/pingcap/tidb/pull/14618)
- `backoff`ロジックに`killed`タグのチェックが含まれていないためクエリをキャンセルできない問題を修正します[#14614](https://github.com/pingcap/tidb/pull/14614)
- 内部ロックを保持する時間を短縮することでステートメントのサマリーのパフォーマンスを向上させる問題を修正します[#14627](https://github.com/pingcap/tidb/pull/14627)
- TiDBの文字列を時刻に解析する結果がMySQLと互換性がない問題を修正します[#14570](https://github.com/pingcap/tidb/pull/14570)
- ユーザーログインの失敗を監査ログに記録します[#14620](https://github.com/pingcap/tidb/pull/14620)
- 悲観的トランザクションのロックキーの数を監視するための`tidb_session_statement_lock_keys_count`監視項目を追加します[#14634](https://github.com/pingcap/tidb/pull/14634)
- JSON内の文字（`&`、`<`、`>`など）が誤ってエスケープされる問題を修正します[#14637](https://github.com/pingcap/tidb/pull/14637)
- `HashJoin`操作がハッシュテーブルを構築する際にメモリ使用量が過剰なために引き起こされるシステムパニック問題を修正します[#14642](https://github.com/pingcap/tidb/pull/14642)
- SQLバインディングが不正なレコードを処理する際の不正確な処理ロジックによって引き起こされるパニック問題を修正します[#14645](https://github.com/pingcap/tidb/pull/14645)
- 小数の除算計算で切り捨てエラーを検出することでMySQLとの非互換性問題を修正します[#14673](https://github.com/pingcap/tidb/pull/14673)
- 存在しないテーブルに対してユーザーに権限を成功裏に付与する問題を修正します[#14611](https://github.com/pingcap/tidb/pull/14611)

## TiKV

+ Raftstore
    - リージョンのマージ失敗によって引き起こされるシステムパニック問題＃6460またはデータの損失問題＃598を修正します[#6481](https://github.com/tikv/tikv/pull/6481)
    - スケジューリングの公平性を最適化するための`yield`をサポートし、リーダーの事前転送をサポートすることでリーダーのスケジューリングの安定性を向上させます[#6563](https://github.com/tikv/tikv/pull/6563)

## PD

- システムのトラフィックが変化すると自動的にリージョンキャッシュ情報を更新することで無効なキャッシュ問題を修正します[#2103](https://github.com/pingcap/pd/pull/2103)
- リーダーリース時間を使用してTSOサービスの有効性を決定します[#2117](https://github.com/pingcap/pd/pull/2117)

## Tools

+ TiDB Binlog
    - Drainerで中継ログをサポートします[#893](https://github.com/pingcap/tidb-binlog/pull/893)
+ TiDB Lightning
    - 構成ファイルが不足している場合に一部の構成項目がデフォルト値を使用するようにします[#255](https://github.com/pingcap/tidb-lightning/pull/255)
    - 非サーバーモードでWebインターフェースが開けない問題を修正します[#259](https://github.com/pingcap/tidb-lightning/pull/259)

## TiDB Ansible

- 一部のシナリオでPDリーダーを取得できないためにコマンドの実行が失敗する問題を修正します[#1121](https://github.com/pingcap/tidb-ansible/pull/1121)
- TiDBダッシュボードに`Deadlock Detect Duration`モニタリング項目を追加します[#1127](https://github.com/pingcap/tidb-ansible/pull/1127)
- TiDBダッシュボードに`Statement Lock Keys Count`モニタリング項目を追加します[#1132](https://github.com/pingcap/tidb-ansible/pull/1132)
- TiDBダッシュボードに`Statement Pessimistic Retry Count`モニタリング項目を追加します[#1133](https://github.com/pingcap/tidb-ansible/pull/1133)