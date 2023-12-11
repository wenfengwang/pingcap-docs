---
title: TiDB 3.0.16のリリースノート
aliases: ['/docs/dev/releases/release-3.0.16/']
---

# TiDB 3.0.16のリリースノート

リリース日: 2020年7月3日

TiDBバージョン: 3.0.16

## 改善点

+ TiDB

    - `is null`フィルタ条件をハッシュパーティションプルーニングでサポート [#17308](https://github.com/pingcap/tidb/pull/17308)
    - 複数のリージョンリクエストが同時に失敗した場合のSQLタイムアウトの問題を避けるため、それぞれのリージョンに異なる`Backoffer`を割り当てるように修正  [#17583](https://github.com/pingcap/tidb/pull/17583)
    - 追加されたパーティションのために別々のリージョンを分割するように修正  [#17668](https://github.com/pingcap/tidb/pull/17668)
    - `delete`または`update`ステートメントから生成されたフィードバックを破棄するように修正 [#17841](https://github.com/pingcap/tidb/pull/17841)
    - `json.Unmarshal`の`job.DecodeArgs`の使用法を修正し、将来のGoバージョンと互換性があるように修正  [#17887](https://github.com/pingcap/tidb/pull/17887)
    - 遅いクエリログとステートメントのサマリーテーブルから機密情報を削除  [#18128](https://github.com/pingcap/tidb/pull/18128)
    - `DateTime`のデリミタのMySQLとの動作を合わせるように修正  [#17499](https://github.com/pingcap/tidb/pull/17499)
    - MySQLの動作と一致するように日付フォーマット内の`%h`を処理  [#17496](https://github.com/pingcap/tidb/pull/17496)

+ TiKV

    - スナップショットを受信した後、PDにストアハートビートを送信しないように修正  [#8145](https://github.com/tikv/tikv/pull/8145)
    - PDクライアントログを改善  [#8091](https://github.com/tikv/tikv/pull/8091)

## バグ修正

+ TiDB

    - 他のトランザクションによって解決された書かれた後削除されたプライマリキーのロックによって発生したデータ不整合の問題を修正  [#18248](https://github.com/pingcap/tidb/pull/18248)
    - PDサーバーサイドフォロワーで`Got too many pings` gRPCエラーログが発生する問題を修正  [#17944](https://github.com/pingcap/tidb/pull/17944)
    - HashJoinの子で`TypeNull`列を返すときに発生する可能性のあるパニックの問題を修正  [#17935](https://github.com/pingcap/tidb/pull/17935)
    - アクセスが拒否された際のエラーメッセージを修正  [#17722](https://github.com/pingcap/tidb/pull/17722)
    - `int`および`float`タイプのJSON比較の問題を修正  [#17715](https://github.com/pingcap/tidb/pull/17715)
    - データ競合を引き起こすfailpointをアップデート  [#17710](https://github.com/pingcap/tidb/pull/17710)
    - テーブルを作成する際にタイムアウトプリスプリットしたリージョンが機能しない可能性のある問題を修正  [#17617](https://github.com/pingcap/tidb/pull/17617)
    - 送信失敗後のあいまいなエラーメッセージによって引き起こされるパニックを修正  [#17378](https://github.com/pingcap/tidb/pull/17378)
    - `FLASHBACK TABLE`が特定のケースで失敗する可能性がある問題を修正  [#17165](https://github.com/pingcap/tidb/pull/17165)
    - 文のみが文字列列を持つ場合の正確な範囲計算結果の問題を修正  [#16658](https://github.com/pingcap/tidb/pull/16658)
    - `only_full_group_by` SQLモードが設定されている場合に発生するクエリエラーを修正  [#16620](https://github.com/pingcap/tidb/pull/16620)
    - `case when`関数から返される結果のフィールド長が不正確な問題を修正  [#16562](https://github.com/pingcap/tidb/pull/16562)
    - `count`集約関数の10進数プロパティの型推論を修正  [#17702](https://github.com/pingcap/tidb/pull/17702)

+ TiKV

    - 取り込まれたファイルから誤った結果が読み取られる可能性がある問題を修正  [#8039](https://github.com/tikv/tikv/pull/8039)
    - 複数のマージ処理中にストアが孤立している場合にそのピアを削除できない問題を修正  [#8005](https://github.com/tikv/tikv/pull/8005)

+ PD

    - PDコントロールでRegionキーをクエリする際の`404`エラーを修正  [#2577](https://github.com/pingcap/pd/pull/2577)