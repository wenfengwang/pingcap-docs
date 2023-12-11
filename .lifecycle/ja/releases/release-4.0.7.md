---
title: TiDB 4.0.7 リリースノート
---

# TiDB 4.0.7 リリースノート

リリース日: 2020年9月29日

TiDB バージョン: 4.0.7

## 新機能

+ PD

    - PDクライアントに`GetAllMembers`関数を追加して、PDメンバー情報を取得する[#2980](https://github.com/pingcap/pd/pull/2980)

+ TiDB Dashboard

    - メトリクスの関係グラフを生成する機能をサポート[#760](https://github.com/pingcap-incubator/tidb-dashboard/pull/760)

## 改善点

+ TiDB

    - `join`演算子のためにランタイム情報を追加[#20093](https://github.com/pingcap/tidb/pull/20093)
    - `EXPLAIN ANALYZE`でcoprocessorキャッシュのヒット率情報をサポート[#19972](https://github.com/pingcap/tidb/pull/19972)
    - `ROUND`関数をTiFlashにプッシュダウンするサポート[#19967](https://github.com/pingcap/tidb/pull/19967)
    - `ANALYZE`のための`CMSketch`のデフォルト値を追加[#19927](https://github.com/pingcap/tidb/pull/19927)
    - エラーメッセージの機密化を改善[#20004](https://github.com/pingcap/tidb/pull/20004)
    - MySQL 8.0のコネクタを使用するクライアントからの接続を受け入れる[#19959](https://github.com/pingcap/tidb/pull/19959)

+ TiKV

    - JSONログフォーマットをサポート[#8382](https://github.com/tikv/tikv/pull/8382)

+ PD

    - スケジュールされたオペレーターが終了した際にカウントするように変更[#2983](https://github.com/pingcap/pd/pull/2983)
    - `make-up-replica`オペレーターを高優先度に設定[#2977](https://github.com/pingcap/pd/pull/2977)

+ TiFlash

    - 読み込み中に発生するリージョンメタの変更のエラーハンドリングを改善

+ Tools

    + TiCDC

        - 旧バリュー機能が有効な場合、MySQL sinkで実行効率の良いSQLステートメントをより多く翻訳するサポート[#955](https://github.com/pingcap/tiflow/pull/955)

    + Backup & Restore (BR)

        - バックアップ中に接続が切断された際の接続再試行を追加[#508](https://github.com/pingcap/br/pull/508)

    + TiDB Lightning

        - HTTPインタフェース経由でログレベルを動的に更新する機能をサポート[#393](https://github.com/pingcap/tidb-lightning/pull/393)

## バグ修正

+ TiDB

    - `and`/`or`/`COALESCE`からのベクトル化バグを修正[#20092](https://github.com/pingcap/tidb/pull/20092)
    - 異なるタイプのcopタスクストアの場合にプランダイジェストが同じである問題を修正[#20076](https://github.com/pingcap/tidb/pull/20076)
    - `!= any()`関数の誤った動作を修正[#20062](https://github.com/pingcap/tidb/pull/20062)
    - `slow-log`ファイルが存在しない場合に発生するクエリエラーを修正[#20051](https://github.com/pingcap/tidb/pull/20051)
    - コンテクストがキャンセルされた際にRegionリクエストが継続的にリトライする問題を修正[#20031](https://github.com/pingcap/tidb/pull/20031)
    - ストリーミングリクエストで`cluster_slow_query`テーブルの時間型をクエリーした際にエラーが発生する可能性がある問題を修正[#19943](https://github.com/pingcap/tidb/pull/19943)
    - `case when`を使用したDMLステートメントがスキーマ変更を引き起こす可能性がある問題を修正[#20095](https://github.com/pingcap/tidb/pull/20095)
    - スローログ内の`prev_stmt`情報が機密化されない問題を修正[#20048](https://github.com/pingcap/tidb/pull/20048)
    - tidb-serverが異常終了した際にテーブルロックを解放しない問題を修正[#20020](https://github.com/pingcap/tidb/pull/20020)
    - `ENUM`および`SET`タイプのデータを挿入する際に発生する不正確なエラーメッセージを修正[#19950](https://github.com/pingcap/tidb/pull/19950)
    - 特定の状況で`IsTrue`関数の誤った動作を修正[#19903](https://github.com/pingcap/tidb/pull/19903)
    - PDがスケールインまたはスケールアウトした後に`CLUSTER_INFO`システムテーブルが正常に動作しない可能性がある問題を修正[#20026](https://github.com/pingcap/tidb/pull/20026)
    - `control`式で定数を折り畳んでOut of Memory (OOM)を回避するための統計情報の更新方法を更新[#19910](https://github.com/pingcap/tidb/pull/19910)
    - 無駄な警告やエラーが発生しないように統計情報の更新方法を更新[#20013](https://github.com/pingcap/tidb/pull/20013)

+ TiKV

    - TLSハンドシェイクが失敗した際に利用できないStatus APIの問題を修正[#8649](https://github.com/tikv/tikv/pull/8649)
    - 未定義の動作が発生する可能性がある問題を修正[#7782](https://github.com/tikv/tikv/pull/7782)
    - `UnsafeDestroyRange`を実行中にスナップショットの生成がパニックを起こす可能性がある問題を修正[#8681](https://github.com/tikv/tikv/pull/8681)

+ PD

    - `balance-region`が有効な場合にいくつかのリージョンにリーダーが存在しないと、PDがパニックする可能性があるバグを修正[#2994](https://github.com/pingcap/pd/pull/2994)
    - リージョンマージ後のサイズとリージョンキーの統計の偏差を修正[#2985](https://github.com/pingcap/pd/pull/2985)
    - 不正確なホットスポット統計を修正[#2991](https://github.com/pingcap/pd/pull/2991)
    - `redirectSchedulerDelete`に`nil`チェックが行われていない問題を修正[#2974](https://github.com/pingcap/pd/pull/2974)

+ TiFlash

    - 右外部結合の結果が誤っていた問題を修正

+ Tools

    + Backup & Restore (BR)

        - リストアプロセス後にTiDB構成が変更されてしまうバグを修正[#509](https://github.com/pingcap/br/pull/509)

    + Dumpling

        - 一部の変数が`NULL`の場合にDumplingがメタデータを解析できない問題を修正[#150](https://github.com/pingcap/dumpling/pull/150)