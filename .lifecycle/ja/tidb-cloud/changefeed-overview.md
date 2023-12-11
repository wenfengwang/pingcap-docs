---
title: Changefeed
summary: TiDB Cloudのchangefeedは、TiDB Cloudから他のデータサービスにデータをストリーム転送できます。

# Changefeed

TiDB Cloudのchangefeedは、TiDB Cloudから他のデータサービスにデータをストリーム転送できます。現在、TiDB CloudはApache Kafka、MySQL、TiDB Cloud、およびクラウドストレージへのデータストリーミングをサポートしています。

> **Note:**
>
> - changefeed機能を使用するには、TiDB Dedicatedクラスターバージョンがv6.4.0以降であることを確認してください。
> - 現在、TiDB Cloudはクラスターごとに最大5つのchangefeedを許可しています。
> - [TiDB Serverlessクラスター](/tidb-cloud/select-cluster-tier.md#tidb-serverless)の場合、changefeed機能を使用できません。

changefeed機能にアクセスするには、TiDBクラスターのクラスターオーバービューページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。 changefeedページが表示されます。

changefeedページでは、changefeedを作成したり、既存のchangefeedのリストを表示したり、既存のchangefeed（スケーリング、一時停止、再開、編集、削除）を操作したりすることができます。

## changefeedの作成

changefeedを作成するには、以下のチュートリアルを参照してください：

- [Apache Kafkaへのストリーミング](/tidb-cloud/changefeed-sink-to-apache-kafka.md) (ベータ)
- [MySQLへのストリーミング](/tidb-cloud/changefeed-sink-to-mysql.md)
- [TiDB Cloudへのストリーミング](/tidb-cloud/changefeed-sink-to-tidb-cloud.md)
- [クラウドストレージへのストリーミング](/tidb-cloud/changefeed-sink-to-cloud-storage.md)

## Changefeed RCUsのクエリ

1. 対象のTiDBクラスターのクラスターオーバービューページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. クエリしたい対応するchangefeedを見つけ、**...** > **View**をクリックします。
3. ページの**Specification**エリアで、現在のTiCDCレプリケーションキャパシティユニット（RCU）を確認できます。

## changefeedのスケーリング

changefeedのTiCDC Replication Capacity Units（RCU）をスケールアップまたはスケールダウンして、changfeedを変更できます。

> **Note:**
>
> - クラスターのchangefeedをスケーリングするには、このクラスターのすべてのchangefeedが2023年3月28日以降に作成されていることを確認してください。
> - クラスターには、2023年3月28日以前に作成されたchangefeedと、その後に作成されたchangefeedのスケールアップまたはスケールダウンをサポートしていません。

1. 対象のTiDBクラスターのクラスターオーバービューページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. スケーリングしたい対応するchangefeedを見つけ、**...** > **Scale Up/Down**をクリックします。
3. 新しい仕様を選択します。
4. **Submit**をクリックします。

スケーリングプロセスは約10分かかります（この間、changefeedは通常動作します）、新しい仕様に切り替わるのには数秒かかります（この間、changefeedは自動的に一時停止および再開されます）。

## changefeedの一時停止または再開

1. 対象のTiDBクラスターのクラスターオーバービューページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. 一時停止または再開したい対忈するchangefeedを見つけ、**...** > **Pause/Resume**をクリックします。

## changefeedの編集

> **Note:**
>
> TiDB Cloudでは現在、一時停止状態のchangefeedの編集のみが許可されています。

1. 対象のTiDBクラスターのクラスターオーバービューページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. 編集したいchangefeedを見つけ、**...** > **Pause**をクリックします。
3. changefeedの状態が`Paused`に変更されたら、対応するchangefeedを編集するために**...** > **Edit**をクリックします。

    TiDB Cloudはデフォルトでchangefeed構成を補完します。以下の構成を修正できます：

    - MySQLシンク：**MySQL接続**および**テーブルフィルター**
    - Kafkaシンク：すべての構成

4. 構成を編集した後、対応するchangefeedを再開するために**...** > **Resume**をクリックします。

## changefeedの削除

1. 対象のTiDBクラスターのクラスターオーバービューページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. 削除したい対応するchangefeedを見つけ、**...** > **Delete**をクリックします。

## Changefeedの課金

TiDB Cloudでのchangefeedの課金については、[Changefeed billing](/tidb-cloud/tidb-cloud-billing-ticdc-rcu.md)をご覧ください。

## Changefeedの状態

レプリケーションタスクの状態は、レプリケーションタスクの実行状態を表します。実行プロセス中、レプリケーションタスクはエラーで失敗したり、手動で一時停止や再開したり、指定した`TargetTs`に到達したりします。これらの動作により、レプリケーションタスクの状態が変わることがあります。

状態は以下のように説明されます：

- `CREATING`：レプリケーションタスクが作成中です。
- `RUNNING`：レプリケーションタスクは正常に実行され、チェックポイント-tsが正常に進行しています。
- `EDITING`：レプリケーションタスクが編集中です。
- `PAUSING`：レプリケーションタスクが一時停止中です。
- `PAUSED`：レプリケーションタスクが一時停止しています。
- `RESUMING`：レプリケーションタスクが再開中です。
- `DELETING`：レプリケーションタスクが削除中です。
- `DELETED`：レプリケーションタスクが削除されました。
- `WARNING`：レプリケーションタスクが警告を返しました。リカバラブルなエラーが原因でレプリケーションが続行できない場合があります。この状態のchangefeedは、状態が`RUNNING`に変わるまで再開し続けます。この状態のchangefeedは[GC operations](https://docs.pingcap.com/tidb/stable/garbage-collection-overview)をブロックします。
- `FAILED`：レプリケーションタスクが失敗しました。リカバリできないエラーが原因で、レプリケーションタスクを再開できず、回復できません。この状態のchangefeedは、GC operationsをブロックしません。