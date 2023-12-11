---
title: SQLシェル経由で接続
summary: TiDBクラスターにSQLシェルを使用して接続する方法を学びます。

# SQLシェル経由で接続

TiDB CloudのSQLシェルでは、TiDB SQLを試し、MySQLとの互換性を迅速にテストし、データベースユーザーの権限を管理できます。

> **注意:**
>
> [TiDB Serverlessクラスター](/tidb-cloud/select-cluster-tier.md#tidb-serverless)にはSQLシェルを使用して接続できません。TiDB Serverlessクラスターに接続するには、[TiDB Serverlessクラスターに接続](/tidb-cloud/connect-to-tidb-cluster-serverless.md)を参照してください。

TiDBクラスターにSQLシェルを使用して接続するには、次の手順を実行します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にログインし、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動します。

    > **ヒント:**
    >
    > 複数のプロジェクトをお持ちの場合は、左下隅の<MDSvgIcon name="icon-left-projects" />をクリックして、別のプロジェクトに切り替えることができます。

2. ターゲットクラスターの名前をクリックして、そのクラスター概要ページに移動し、右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. ダイアログで、**Web SQL Shell**タブを選択し、**Open SQL Shell**をクリックします。

4. 表示される**Enter password**行に、現在のクラスターのrootパスワードを入力します。その後、アプリケーションはTiDBクラスターに接続されます。