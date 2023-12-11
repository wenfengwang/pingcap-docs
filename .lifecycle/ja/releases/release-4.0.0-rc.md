---
title: TiDB 4.0 RC リリースノート
aliases: ['/docs/dev/releases/release-4.0.0-rc/','/docs/dev/releases/4.0.0-rc/']
---

# TiDB 4.0 RC リリースノート

リリース日: 2020年4月8日

TiDB バージョン: 4.0.0-rc

TiUP バージョン: 0.0.3

> **警告:**
>
> このバージョンにはいくつかの既知の問題がありますが、これらの問題は新しいバージョンで修正されています。最新の 4.0.x バージョンを使用することをお勧めします。

## 互換性の変更

+ TiDB

    - `tidb-server` ステータスポートが占有されている場合、アラートログを返す代わりに起動を拒否するように修正 [#15177](https://github.com/pingcap/tidb/pull/15177)

+ TiKV

    - ピベリスティックトランザクションで `pipelined` 機能をサポートし、TPC-C のパフォーマンスを20%向上させます。リスクとして、トランザクションのコミットが実行中のロックの失敗によって失敗する可能性があります [#6984](https://github.com/tikv/tikv/pull/6984)
    - 新しいクラスタでは`unify-read-pool` 設定項目をデフォルトで有効にし、古いクラスタでは以前の設定を使用するように修正 [#7059](https://github.com/tikv/tikv/pull/7059)

+ ツール

    - TiDB Binlog

        * Common Name の検証を行うための構成項目を追加 [#934](https://github.com/pingcap/tidb-binlog/pull/934)

## 重要なバグ修正

+ TiDB

    - `PREPARE` 文を使用して DDL ジョブが実行されると、内部レコードの不正確なジョブクエリによって、上流と下流間のレプリケーションが誤って行われる問題を修正 [#15435](https://github.com/pingcap/tidb/pull/15435)
    - `Read Commited` 分離レベルにおける不正確なサブクエリの結果の問題を修正 [#15471](https://github.com/pingcap/tidb/pull/15471)
    - インラインプロジェクションの最適化によって引き起こされる不正確な結果の問題を修正 [#15411](https://github.com/pingcap/tidb/pull/15411)
    - SQL ヒント `INL_MERGE_JOIN` が一部のケースで誤って実行される問題を修正 [#15515](https://github.com/pingcap/tidb/pull/15515)
    - `AutoRandom` 属性を持つ列に負の数値が明示的に書き込まれた場合に、列が再ベースされる問題を修正 [#15397](https://github.com/pingcap/tidb/pull/15397)

## 新機能

+ TiDB

    - 大文字小文字を区別しない照合を追加し、ユーザーが新しいクラスタで `utf8mb4_general_ci` と `utf8_general_ci` を有効にできるようにする [#33](https://github.com/pingcap/tidb/projects/33)
    - `RECOVER TABLE` 構文を強化して切り捨てられたテーブルを回復するようにサポートを追加 [#15398](https://github.com/pingcap/tidb/pull/15398)
    - `tidb-server` ステータスポートが占有されている場合、アラートログを返す代わりに起動を拒否するように修正 [#15177](https://github.com/pingcap/tidb/pull/15177)
    - シーケンスをデフォルトの列値として使用する書き込みパフォーマンスを最適化 [#15216](https://github.com/pingcap/tidb/pull/15216)
    - DDL ジョブの詳細をクエリするための `DDLJobs` システムテーブルを追加 [#14837](https://github.com/pingcap/tidb/pull/14837)
    - `aggFuncSum` パフォーマンスを最適化 [#14887](https://github.com/pingcap/tidb/pull/14887)
    - `EXPLAIN` の出力を最適化 [#15507](https://github.com/pingcap/tidb/pull/15507)

+ TiKV

    - ピベリスティックトランザクションで `pipelined` 機能をサポートし、TPC-C のパフォーマンスを20%向上させます。リスクとして、トランザクションのコミットが実行中のロックの失敗によって失敗する可能性があります [#6984](https://github.com/tikv/tikv/pull/6984)
    - HTTP ポートで TLS をサポート [#5393](https://github.com/tikv/tikv/pull/5393)
    - 新しいクラスタでは `unify-read-pool` 設定項目をデフォルトで有効にし、古いクラスタでは以前の設定を使用するように修正 [#7059](https://github.com/tikv/tikv/pull/7059)

+ PD

    - HTTP API を介してデフォルトの PD 構成情報を取得するサポートを追加 [#2258](https://github.com/pingcap/pd/pull/2258)

+ ツール

    - TiDB Binlog

        * Common Name の検証を行うための構成項目を追加 [#934](https://github.com/pingcap/tidb-binlog/pull/934)

    - TiDB Lightning

        * TiDB Lightning のパフォーマンスを最適化 [#281](https://github.com/pingcap/tidb-lightning/pull/281) [#275](https://github.com/pingcap/tidb-lightning/pull/275)

## バグ修正

+ TiDB

    - `PREPARE` 文を使用して DDL ジョブが実行されると、内部レコードの不正確なジョブクエリによって、上流と下流間のレプリケーションが誤って行われる問題を修正 [#15435](https://github.com/pingcap/tidb/pull/15435)
    - `Read Commited` 分離レベルにおける不正確なサブクエリの結果の問題を修正 [#15471](https://github.com/pingcap/tidb/pull/15471)
    - `INSERT ... VALUES` を使用して `BIT(N)` データ型を指定した場合の可能な誤った動作の問題を修正 [#15350](https://github.com/pingcap/tidb/pull/15350)
    - DDL ジョブの内部リトライが `ErrorCount` の値が正しく合計されないため、期待される結果が完全に実現されない問題を修正 [#15373](https://github.com/pingcap/tidb/pull/15373)
    - TiDB が TiFlash に接続すると、Garbage Collection が異常に機能する可能性がある問題を修正 [#15505](https://github.com/pingcap/tidb/pull/15505)
    - インラインプロジェクションの最適化によって引き起こされる不正確な結果の問題を修正 [#15411](https://github.com/pingcap/tidb/pull/15411)
    - SQL ヒント `INL_MERGE_JOIN` が一部のケースで誤って実行される問題を修正 [#15515](https://github.com/pingcap/tidb/pull/15515)
    - `AutoRandom` 属性を持つ列に負の数値が明示的に書き込まれた場合に、列が再ベースされる問題を修正 [#15397](https://github.com/pingcap/tidb/pull/15397)

+ TiKV

    - フォロワーリード機能が有効になっている場合、リーダーの転送によって発生する可能性のあるパニックを修正 [#7101](https://github.com/tikv/tikv/pull/7101)

+ ツール

    - TiDB Lightning

        * バックエンドが TiDB の場合に文字変換のエラーによってデータエラーが発生する問題を修正 [#283](https://github.com/pingcap/tidb-lightning/pull/283)

    - TiCDC

        * MySQL シンクが DDL ステートメントを実行すると、下流に `test` スキーマが存在しない場合にエラーが返される問題を修正 [#353](https://github.com/pingcap/tiflow/pull/353)
        * CDC cli でリアルタイムインタラクティブモードをサポート [#351](https://github.com/pingcap/tiflow/pull/351)
        * データレプリケーション中に上流のテーブルがレプリケート可能かどうかをチェックするサポートを追加 [#368](https://github.com/pingcap/tiflow/pull/368)
        * Kafka への非同期書き込みをサポート [#344](https://github.com/pingcap/tiflow/pull/344)