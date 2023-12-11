---
title: TiDB専用クラスターの一時停止または再開
summary: TiDB専用クラスターを一時停止または再開する方法を学びます。

# TiDB専用クラスターの一時停止または再開

TiDB Cloudでは、常に稼働していないTiDB専用クラスターを簡単に一時停止および再開することができます。

一時停止は、クラスターに保存されているデータに影響を与えませんが、モニタリング情報の収集とコンピューティングリソースの消費を停止します。一時停止後、いつでもクラスターを再開できます。

バックアップとリストアと比較して、クラスターの一時停止と再開はより短い時間がかかり、クラスターバージョン、クラスター構成、TiDBユーザーアカウントを含むクラスター情報が保持されます。

> **注:**
>
> [TiDB Serverlessクラスター](/tidb-cloud/select-cluster-tier.md#tidb-serverless)は一時停止できません。

## 制限事項

- クラスターは**利用可能**な状態のときのみ一時停止できます。クラスターが**変更中**などの他の状態にある場合は、クラスターを一時停止する前に現在の操作が完了するのを待たなければなりません。
- データのインポートタスクが実行中のときはクラスターを一時停止できません。インポートタスクが完了するのを待つか、インポートタスクをキャンセルするしかありません。
- バックアップジョブが実行中のときはクラスターを一時停止できません。現在のバックアップジョブが完了するのを待つか、実行中のバックアップジョブを[削除します](/tidb-cloud/backup-and-restore.md#delete-a-running-backup-job)。
- [changefeeds](/tidb-cloud/changefeed-overview.md)が存在する場合はクラスターを一時停止できません。クラスターを一時停止する前に既存のchangefeedsを[削除する必要があります](/tidb-cloud/changefeed-overview.md#delete-a-changefeed)。

## TiDBクラスターの一時停止

クラスターが一時停止されると、以下の点に注意してください：

- TiDB Cloudはクラスターのモニタリング情報の収集を停止します。
- クラスターからデータの読み取りや書き込みはできません。
- データのインポートやバックアップを行えません。
- 費用としては以下のみが請求されます：

    - ノードのストレージ費用
    - データバックアップ費用

- TiDB Cloudはクラスターの[自動バックアップ](/tidb-cloud/backup-and-restore.md#automatic-backup)を停止します。

クラスターを一時停止するには、次の手順を実行します：

1. TiDB Cloudコンソールで、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動します。
2. 一時停止したいクラスターの行で**...**をクリックします。

    > **ヒント:**
    >
    > 代わりに、**Clusters**ページで一時停止したいクラスターの名前をクリックし、その後、右上隅の**...**をクリックすることもできます。

3. ドロップダウンメニューで**Pause**をクリックします。

    **クラスターを一時停止**ダイアログが表示されます。

4. ダイアログで**Pause**をクリックして選択を確認します。

    **Pause**をクリックすると、クラスターはまず**一時停止中**の状態になります。一時停止操作が完了すると、クラスターは**一時停止**の状態に移行します。

TiDB Cloud APIを使用してクラスターを一時停止することもできます。現在、TiDB Cloud APIはベータ版です。詳細については、[TiDB Cloud APIドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta)を参照してください。

## TiDBクラスターの再開

一時停止したクラスターを再開すると、以下の点に注意してください：

- TiDB Cloudはクラスターのモニタリング情報の収集を再開し、クラスターからデータの読み取りや書き込みが可能になります。
- TiDB Cloudはコンピューティングとストレージの費用を再開します。
- TiDB Cloudはクラスターの[自動バックアップ](/tidb-cloud/backup-and-restore.md#automatic-backup)を再開します。

一時停止したクラスターを再開するには、次の手順を実行します：

1. TiDB Cloudコンソールで、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動します。
2. 再開したいクラスターで**再開**をクリックします。**クラスターを再開**ダイアログが表示されます。

    > **注:**
    >
    > **一時停止中**のクラスターを再開することはできません。

3. ダイアログで**再開**をクリックして選択を確認します。クラスターの状態は**再開中**になります。

クラスターのサイズに応じて、クラスターを再開するのに数分かかる場合があります。クラスターが再開されると、クラスターの状態が**再開中**から**利用可能**に変わります。

TiDB Cloud APIを使用してクラスターを再開することもできます。現在、TiDB Cloud APIはベータ版です。詳細については、[TiDB Cloud APIドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta)を参照してください。