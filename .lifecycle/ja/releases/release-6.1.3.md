---
title: TiDB 6.1.3 のリリースノート
---

# TiDB 6.1.3 のリリースノート

リリース日: 2022年12月5日

TiDB バージョン: 6.1.3

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.1/quick-start-with-tidb) | [本番環境への展開](https://docs.pingcap.com/tidb/v6.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.1.3#version-list)

## 互換性の変更

- ツール

    - TiCDC

        - [`transaction-atomicity`](/ticdc/ticdc-sink-to-mysql.md#configure-sink-uri-for-mysql-or-tidb) のデフォルト値を `table` から `none` に変更しました。これにより、レプリケーションの遅延が減少し、OOM のリスクが低減され、単一トランザクションのサイズが 1024 行を超える場合にのみ、すべてのトランザクションではなく、わずかなトランザクションが分割されることが保証されます [#7505](https://github.com/pingcap/tiflow/issues/7505) [#5231](https://github.com/pingcap/tiflow/issues/5231) @[asddongmen](https://github.com/asddongmen)

## 改善点

- PD

    - ロックの粒度を最適化して、ロックの競合を減らし、高い並行性でハートビートを処理する能力を向上させました [#5586](https://github.com/tikv/pd/issues/5586) @[rleungx](https://github.com/rleungx)

- ツール

    - TiCDC

        - デフォルトで TiCDC の changefeed でトランザクションを分割し、セーフモードを無効にし、パフォーマンスを向上させました [#7505](https://github.com/pingcap/tiflow/issues/7505) @[asddongmen](https://github.com/asddongmen)
        - Kafka プロトコルのエンコーダのパフォーマンスを向上させました [#7540](https://github.com/pingcap/tiflow/issues/7540), [#7532](https://github.com/pingcap/tiflow/issues/7532), [#7543](https://github.com/pingcap/tiflow/issues/7543) @[sdojjy](https://github.com/sdojjy) @[3AceShowHand](https://github.com/3AceShowHand)

- その他

    - TiDB の Go コンパイラのバージョンを go1.18 から [go1.19](https://go.dev/doc/go1.19) にアップグレードしました。これにより、Go 環境変数 [`GOMEMLIMIT`](https://pkg.go.dev/runtime@go1.19#hdr-Environment_Variables) が導入され、TiDB のメモリ使用量がある閾値以下に保たれます。これにより、ほとんどの OOM の問題が緩和されます。詳細については、[`GOMEMLIMIT` の構成による OOM の問題の緩和](/configure-memory-usage.md#mitigate-oom-issues-by-configuring-gomemlimit) を参照してください。

## バグ修正

+ TiDB

    - `mysql.tables_priv` テーブルの `grantor` フィールドが欠落している問題を修正しました [#38293](https://github.com/pingcap/tidb/issues/38293) @[CbcWestwolf](https://github.com/CbcWestwolf)
    - 誤ってプッシュダウンされた条件が Join Reorder によって破棄されると、間違ったクエリ結果が発生する問題を修正しました [#38736](https://github.com/pingcap/tidb/issues/38736) @[winoros](https://github.com/winoros)
    - `get_lock()` で取得したロックが 10 分以上維持できない問題を修正しました [#38706](https://github.com/pingcap/tidb/issues/38706) @[tangenta](https://github.com/tangenta)
    - オートインクリメント列がチェック制約と使用できない問題を修正しました [#38894](https://github.com/pingcap/tidb/issues/38894) @[YangKeao](https://github.com/YangKeao)
    - gPRC ログが間違ったファイルに出力される問題を修正しました [#38941](https://github.com/pingcap/tidb/issues/38941) @[xhebox](https://github.com/xhebox)
    - TiFlash のテーブルの同期ステータスが切り捨てられたり削除された場合に、その情報が etcd から削除されない問題を修正しました [#37168](https://github.com/pingcap/tidb/issues/37168) @[CalvinNeo](https://github.com/CalvinNeo)
    - データファイルがデータソース名のインジェクションによって制限なくアクセスできる問題を修正しました (CVE-2022-3023) [#38541](https://github.com/pingcap/tidb/issues/38541) @[lance6716](https://github.com/lance6716)
    - `str_to_date` 関数が `NO_ZERO_DATE` SQL モードで誤った結果を返す問題を修正しました [#39146](https://github.com/pingcap/tidb/issues/39146) @[mengxin9014](https://github.com/mengxin9014)
    - バックグラウンドでの統計収集タスクがいくつかのシナリオでパニックを引き起こす可能性がある問題を修正しました [#35421](https://github.com/pingcap/tidb/issues/35421) @[lilinghai](https://github.com/lilinghai)
    - 一部のシナリオで悲観的なロックが一意でないセカンダリインデックスに誤って追加される問題を修正しました [#36235](https://github.com/pingcap/tidb/issues/36235) @[ekexium](https://github.com/ekexium)

- PD

    - 不正確なストリームタイムアウトを修正し、リーダーの切り替えを加速しました [#5207](https://github.com/tikv/pd/issues/5207) @[CabinfeverB](https://github.com/CabinfeverB)

+ TiKV

    - スナップショットの取得中に有効期限が切れたリースによる異常なリージョン競合を修正しました [#13553](https://github.com/tikv/tikv/issues/13553) @[SpadeA-Tang](https://github.com/SpadeA-Tang)

+ TiFlash

    - 論理演算子が `UInt8` 型の引数で誤った結果を返す問題を修正しました [#6127](https://github.com/pingcap/tiflash/issues/6127) @[xzhangxian1008](https://github.com/xzhangxian1008)
    - `CAST(value AS DATETIME)` に誤ったデータ入力がある場合に高い TiFlash システム CPU が発生する問題を修正しました [#5097](https://github.com/pingcap/tiflash/issues/5097) @[xzhangxian1008](https://github.com/xzhangxian1008)
    - 大量の書き込みによってデルタレイヤに多くの列ファイルが生成される問題を修正しました [#6361](https://github.com/pingcap/tiflash/issues/6361) @[lidezhu](https://github.com/lidezhu)
    - TiFlash を再起動した後にデルタレイヤの列ファイルがコンパクト化されない問題を修正しました [#6159](https://github.com/pingcap/tiflash/issues/6159) @[lidezhu](https://github.com/lidezhu)

+ ツール

    + バックアップ＆リストア（BR）

        - 旧しフレームワークを使用してデータベースやテーブルの整理ルールを復元する際に失敗する問題を修正しました [#39150](https://github.com/pingcap/tidb/issues/39150) @[MoCuishle28](https://github.com/MoCuishle28)

    + TiCDC

        - DDL ステートメントを最初に実行し、その後 changefeed を一時停止して再開するシナリオでデータ損失が発生する問題を修正しました [#7682](https://github.com/pingcap/tiflow/issues/7682) @[asddongmen](https://github.com/asddongmen)
        - 下流のネットワークが利用できなくなった場合に、sink コンポーネントが詰まる問題を修正しました [#7706](https://github.com/pingcap/tiflow/issues/7706) @[hicqu](https://github.com/hicqu)

    + TiDB データ移行（DM）

        - `collation_compatible` が `"strict"` に設定されている場合、DM が重複した整理ルールを生成する問題を修正しました [#6832](https://github.com/pingcap/tiflow/issues/6832) @[lance6716](https://github.com/lance6716)
        - DM タスクが `Unknown placement policy` エラーで停止する問題を修正しました [#7493](https://github.com/pingcap/tiflow/issues/7493) @[lance6716](https://github.com/lance6716)
        - 一部の場合でリレーログが再度上流から取得される問題を修正しました [#7525](https://github.com/pingcap/tiflow/issues/7525) @[liumengya94](https://github.com/liumengya94)
        - 新しい DM ワーカーが既存のワーカーが終了する前にスケジュールされると、データが複数回複製される問題を修正しました [#7658](https://github.com/pingcap/tiflow/issues/7658) @[GMHDBJD](https://github.com/GMHDBJD)