---
title: アップストリームおよびダウンストリームクラスタのデータ検証とスナップショット読み取り
summary: TiDBのアップストリームおよびダウンストリームクラスタのデータを確認する方法を学ぶ。
aliases: ['/docs/dev/sync-diff-inspector/upstream-downstream-diff/','/docs/dev/reference/tools/sync-diff-inspector/tidb-diff/', '/tidb/dev/upstream-downstream-diff']
---

# アップストリームおよびダウンストリームクラスタのデータ検証とスナップショット読み取り

TiCDCを使用してTiDBのアップストリームおよびダウンストリームクラスタを構築する際には、レプリケーションを停止することなくアップストリームおよびダウンストリームのデータ整合的なスナップショット読み取りまたはデータ整合性検証を実行する必要があります。通常のレプリケーションモードでは、TiCDCはデータが最終的に整合することしか保証せず、レプリケーションプロセス中にデータが整合していることを保証することはできません。そのため、動的に変化するデータの整合的な読み取りを実行することは困難です。このようなニーズを満たすために、TiCDCはSyncpoint機能を提供しています。

SyncpointはTiDBのスナップショット機能を使用し、TiCDCにアップストリームおよびダウンストリーム間の整合性を保持する`ts-map`を維持させることで、レプリケーションプロセス中においてアップストリームおよびダウンストリームスナップショットの整合性を定期的に整合させ、動的データの整合性検証の問題を静的スナップショットデータの整合性検証の問題に変換します。これにより、ほぼリアルタイムの検証効果が達成されます。

## Syncpointの有効化

Syncpoint機能を有効にした後は、[Consistent snapshot read](#consistent-snapshot-read) および [データ整合性検証](#data-consistency-validation) を使用できます。

Syncpoint機能を有効にするには、レプリケーションタスクの作成時にTiCDC構成項目 `enable-sync-point` の値を `true` に設定します。Syncpointを有効にした後、TiCDCは以下の情報をダウンストリームのTiDBクラスタに書き込みます。

1. レプリケーション中、TiCDCは定期的に（`sync-point-interval` で構成）アップストリームとダウンストリームのスナップショットを整合させ、アップストリームとダウンストリームのTSOの対応関係をダウンストリームの `tidb_cdc.syncpoint_v1` テーブルに保存します。
2. レプリケーション中、TiCDCは定期的に（`sync-point-interval` で構成） `SET GLOBAL tidb_external_ts = @@tidb_current_ts` を実行し、レプリケートされたバックアップクラスタで一貫したスナップショットポイントを設定します。

以下のTiCDC構成例はレプリケーションタスクの作成時にSyncpointを有効にしています。

```toml
# SyncPointを有効にします。
enable-sync-point = true

# アップストリームとダウンストリームのスナップショットを毎5分整合させます
sync-point-interval = "5m"

# ダウンストリームのtidb_cdc.syncpoint_v1テーブル内のts-mapデータを1時間ごとにクリーンアップします
sync-point-retention = "1h"
```

## 一貫したスナップショット読み取り

> **注意:**
>
> 一貫したスナップショット読み取りを行う前に、[Syncpoint機能を有効にして](#enable-syncpoint) いることを確認してください。

バックアップクラスタからデータをクエリする際、アプリケーションがバックアップクラスタでトランザクション的に整合したデータを取得するために `SET GLOBAL|SESSION tidb_enable_external_ts_read = ON;` を設定できます。

さらに、`ts-map` をクエリして以前の時間点を選択してスナップショット読み取りを実行することもできます。

## データ整合性検証

> **注意:**
>
> データ整合性検証を実行する前に、[Syncpoint機能を有効にして](#enable-syncpoint) いることを確認してください。

アップストリームおよびダウンストリームクラスタのデータを検証するには、`sync-diff-inspector`で `snapshot` を構成するだけです。

### ステップ1: `ts-map`の取得

ダウンストリームのTiDBクラスタで以下のSQLステートメントを実行して、アップストリームTSO（`primary_ts`）およびダウンストリームTSO（`secondary_ts`）を取得できます。

```sql
select * from tidb_cdc.syncpoint_v1;
+------------------+----------------+--------------------+--------------------+---------------------+
| ticdc_cluster_id | changefeed     | primary_ts         | secondary_ts       | created_at          |
+------------------+----------------+--------------------+--------------------+---------------------+
| default          | test-2 | 435953225454059520 | 435953235516456963 | 2022-09-13 08:40:15 |
+------------------+----------------+--------------------+--------------------+---------------------+
```

前述の `syncpoint_v1` テーブルのフィールドは次のとおりです。

- `ticdc_cluster_id`: このレコードのTiCDCクラスタのIDです。
- `changefeed`: このレコードのchangefeedのIDです。異なるTiCDCクラスタで同じ名前のchangefeedを持つ可能性があるため、`ts-map`がTiCDCクラスタIDおよびchangefeed IDによって挿入されたことを確認する必要があります。
- `primary_ts`: アップストリームデータベースのスナップショットのタイムスタンプです。
- `secondary_ts`: ダウンストリームデータベースのスナップショットのタイムスタンプです。
- `created_at`: このレコードが挿入された時刻です。

### ステップ2: スナップショットの構成

次に、[ステップ1](#step-1-obtain-ts-map) で取得した`ts-map`情報を使用して、アップストリームおよびダウンストリームデータベースのスナップショット情報を構成します。

以下は `Datasource config` セクションの構成例です。

```toml
######################### Datasource config ########################
[data-sources.uptidb]
    host = "172.16.0.1"
    port = 4000
    user = "root"
    password = ""
    snapshot = "435953225454059520"

[data-sources.downtidb]
    host = "172.16.0.2"
    port = 4000
    user = "root"
    snapshot = "435953235516456963"
```

## 注意事項

- TiCDCがchangefeedを作成する前に、TiCDC構成項目 `enable-sync-point` の値を `true` に設定していることを確認してください。これにより、Syncpointが有効化され、`ts-map`がダウンストリームに保存されます。完全な構成については、[TiCDCタスク構成ファイル](/ticdc/ticdc-changefeed-config.md) を参照してください。
- Syncpointを使用してデータ検証を実行する場合、TiKVのGarbage Collection（GC）時間を変更して、データ検証中にスナップショットに対応する過去のデータがGCによって収集されないようにする必要があります。データの確認後、GC時間を1時間に変更してから、設定を元に戻すことをお勧めします。
- 上記の例は`Datasource config`の一部のみを示しています。完全な構成については、[sync-diff-inspectorユーザーガイド](/sync-diff-inspector/sync-diff-inspector-overview.md) を参照してください。
- v6.4.0以降、`SYSTEM_VARIABLES_ADMIN`または`SUPER`権限を持つchangefeedのみがTiCDC Syncpoint機能を使用できます。