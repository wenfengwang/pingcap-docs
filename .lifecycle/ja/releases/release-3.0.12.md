---
title: TiDB 3.0.12のリリースノート
aliases: ['/docs/dev/releases/release-3.0.12/','/docs/dev/releases/3.0.12/']
---

# TiDB 3.0.12のリリースノート

リリース日: 2020年3月16日

TiDBバージョン: 3.0.12

TiDB Ansibleバージョン: 3.0.12

> **警告:**
>
> このバージョンにはいくつかの既知の問題が見つかっており、これらの問題は新しいバージョンで修正されています。最新の3.0.xバージョンを使用することをお勧めします。

## 互換性の変更

+ TiDB
    - スロークエリログの事前書き込みビンロッグのタイミングが正確でない問題を修正しました。元のタイミングフィールドは `Binlog_prewrite_time` と呼ばれていました。この修正後、名前が `Wait_prewrite_binlog_time` に変更されます。 [#15276](https://github.com/pingcap/tidb/pull/15276)

## 新機能

+ TiDB
    - `alter instance` ステートメントを使用して、置換証明書ファイルの動的ロードをサポートしました。 [#15080](https://github.com/pingcap/tidb/pull/15080) [#15292](https://github.com/pingcap/tidb/pull/15292)
    - `cluster-verify-cn` 構成項目を追加しました。設定後、対応するCN証明書がないとステータスサービスを使用できません。 [#15164](https://github.com/pingcap/tidb/pull/15164)
    - 各TiDBサーバでDDLリクエストのフロー制限機能を追加し、DDLリクエストの競合のエラー報告頻度を減らしました。 [#15148](https://github.com/pingcap/tidb/pull/15148)
    - ビンロッグの書き込みが失敗した場合にTiDBサーバを終了するサポートを追加しました。 [#15339](https://github.com/pingcap/tidb/pull/15339)

+ ツール
    - TiDB Binlog
        - Drainerに `kafka-client-id` 構成項目を追加しました。これにより、Kafkaクライアントへの接続にクライアントIDを構成することができます。 [#929](https://github.com/pingcap/tidb-binlog/pull/929)

## バグ修正

+ TiDB
    - 複数のユーザを修正する際に `GRANT`、 `REVOKE` が原子性を保証するように変更しました。 [#15092](https://github.com/pingcap/tidb/pull/15092)
    - パーティションテーブルの悲観的ロックのロックに失敗し、正しい行をロックしなかった問題を修正しました。 [#15114](https://github.com/pingcap/tidb/pull/15114)
    - インデックス長が制限を超える場合、構成の `max-index-length` の値に応じてエラーメッセージを表示するように修正しました。 [#15130](https://github.com/pingcap/tidb/pull/15130)
    - `FROM_UNIXTIME` 関数の不正確な小数点の問題を修正しました。 [#15270](https://github.com/pingcap/tidb/pull/15270)
    - トランザクションで自分自身が書き込んだレコードを削除することによって引き起こされる競合検出の失敗またはデータインデックスの不整合の問題を修正しました。 [#15176](https://github.com/pingcap/tidb/pull/15176)

+ TiKV
    - 一致性チェックパラメータの無効化時に既存のキーをトランザクションに挿入してすぐに削除することで引き起こされる競合検出の失敗またはデータインデックスの不整合の問題を修正しました。 [#7054](https://github.com/tikv/tikv/pull/7054)
    - Raftstoreでフローコントロールメカニズムを導入し、フローコントロールがないと、追跡が遅くなり、クラスタが停滞し、トランザクションサイズがTiKV接続の頻繁な再接続を引き起こす可能性がある問題を解決しました。 [#7072](https://github.com/tikv/tikv/pull/7072) [#6993](https://github.com/tikv/tikv/pull/6993)

+ PD
    - PDがRegionのハートビートを処理する際のデータ競合によって引き起こされる不正確なRegion情報の問題を修正しました。 [#2233](https://github.com/pingcap/pd/pull/2233)

+ TiDB Ansible
    - クラスタ内に複数のGrafana/Prometheus/Alertmanagerをデプロイすることをサポートしました。 [#1198](https://github.com/pingcap/tidb-ansible/pull/1198)