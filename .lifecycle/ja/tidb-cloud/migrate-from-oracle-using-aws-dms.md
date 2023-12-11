---
title:  AWS DMSを使用してAmazon RDS for OracleからTiDB Cloudに移行する方法
summary: Amazon RDS for OracleからTiDB Serverlessにデータを移行する方法について、AWS Database Migration Service（AWS DMS）を使用してステップバイステップの例を説明します。

# AWS DMSを使用してAmazon RDS for OracleからTiDB Cloudに移行する方法

このドキュメントでは、AWS Database Migration Service（AWS DMS）を使用して、Amazon RDS for Oracleから[TiDB Serverless](https://tidbcloud.com/console/clusters/create-cluster)にデータを移行する手順の例について説明します。

TiDB CloudとAWS DMSについて詳しく学びたい場合は、以下を参照してください。

- [TiDB Cloud](https://docs.pingcap.com/tidbcloud/)
- [TiDB 開発者ガイド](https://docs.pingcap.com/tidbcloud/dev-guide-overview)
- [AWS DMSのドキュメント](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.html)

## AWS DMSを使用する理由

AWS DMSは、リレーショナルデータベース、データウェアハウス、NoSQLデータベース、その他のタイプのデータストアを移行することが可能なクラウドサービスです。

PostgreSQL、Oracle、SQL Serverなどの異種データベースからTiDB Cloudにデータを移行したい場合は、AWS DMSを使用することをお勧めします。

## デプロイメントアーキテクチャ

高レベルでは、以下の手順に従います。

1. Amazon RDS for Oracleのソースをセットアップします。
2. [ターゲットのTiDB Serverless](https://tidbcloud.com/console/clusters/create-cluster)をセットアップします。
3. AWS DMSを使用してデータ移行（フルロード）をセットアップします。

以下の図は、高レベルのアーキテクチャを示しています。

![アーキテクチャ](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-0.png)

## 前提条件

開始する前に、以下の前提条件をお読みください。

- [AWS DMSの前提条件](/tidb-cloud/migrate-from-mysql-using-aws-dms.md#prerequisites)
- [AWSクラウドアカウント](https://aws.amazon.com)
- [TiDB Cloudアカウント](https://tidbcloud.com)
- [DBeaver](https://dbeaver.io/)

次に、AWS DMSを使用してAmazon RDS for OracleからTiDB Cloudにデータを移行する方法を学びます。

## ステップ1. VPCを作成する

[AWSコンソール](https://console.aws.amazon.com/vpc/home#vpcs:)にログインし、AWS VPCを作成します。後でこのVPCでOracle RDSおよびDMSインスタンスを作成する必要があります。

VPCの作成方法についての手順については、[VPCの作成](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html#Create-VPC)を参照してください。

![VPCの作成](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-1.png)

## ステップ2. Oracle DBインスタンスを作成する

作成したVPC内でOracle DBインスタンスを作成し、パスワードを覚えておいてパブリックアクセスを許可します。AWS Schema Conversion Toolを使用するためにパブリックアクセスを有効にする必要があります。なお、本番環境でのパブリックアクセスの許可は推奨されていません。

Oracle DBインスタンスの作成手順については、[Oracle DBインスタンスの作成およびOracle DBインスタンスのデータベースへの接続](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.Oracle.html)を参照してください。

![Oracle RDSの作成](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-2.png)

## ステップ3. Oracleのテーブルデータを準備する

以下のスクリプトを使用して、github_eventsテーブルに10000行のデータを作成および追加します。[GH Archive](https://gharchive.org/)からgithub eventのデータセットを使用しダウンロードします。10000行のデータが含まれています。以下のSQLスクリプトを使用してOracleで実行してください。

- [table_schema_oracle.sql](https://github.com/pingcap-inc/tidb-integration-script/blob/main/aws-dms/oracle_table_schema.sql)
- [oracle_data.sql](https://github.com/pingcap-inc/tidb-integration-script/blob/main/aws-dms/oracle_data.sql)

SQLスクリプトの実行が完了したら、Oracleでデータを確認してください。以下の例では、[DBeaver](https://dbeaver.io/)を使用してデータをクエリします。

![Oracle RDSのデータ](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-3.png)

## ステップ4. TiDB Serverlessクラスタを作成する

1. [TiDB Cloudコンソール](https://tidbcloud.com/console/clusters)にログインします。

2. [TiDB Serverlessクラスタを作成](/tidb-cloud/tidb-cloud-quickstart.md)します。

3. [**クラスタ**](https://tidbcloud.com/console/clusters) ページで、対象のクラスタ名をクリックして概要ページに移動します。

4. 右上隅にある **Connect** をクリックします。

5. **パスワードの作成** をクリックしてパスワードを生成し、生成されたパスワードをコピーします。

6. 好みの接続方法とオペレーティングシステムを選択して、表示される接続文字列を使用してクラスタに接続します。

## ステップ5. AWS DMSレプリケーションインスタンスを作成する

1. AWS DMSコンソールで、[レプリケーションインスタンス](https://console.aws.amazon.com/dms/v2/home#replicationInstances) ページに移動し、対応するリージョンに切り替えます。

2. VPC内で `dms.t3.large` を使用してAWS DMSレプリケーションインスタンスを作成します。

    ![AWS DMSインスタンスの作成](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-8.png)

## ステップ6. DMSエンドポイントを作成する

1. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home)で、左ペインの `エンドポイント` メニューアイテムをクリックします。

2. OracleのソースエンドポイントとTiDBのターゲットエンドポイントを作成します。

    以下のスクリーンショットは、ソースエンドポイントの構成を示しています。

    ![AWS DMSソースエンドポイントの作成](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-9.png)

    以下のスクリーンショットは、ターゲットエンドポイントの構成を示しています。

    ![AWS DMSターゲットエンドポイントの作成](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-10.png)

## ステップ7. スキーマを移行する

この例では、AWS DMSはスキーマを自動的に処理します。

AWS Schema Conversion Toolを使用してスキーマを移行する場合は、[AWS SCTのインストール](https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/CHAP_Installing.html#CHAP_Installing.Procedure)を参照してください。

詳細については、[AWS SCTを使用したソーススキーマをターゲットデータベースに移行する](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.SCT.html)を参照してください。

## ステップ8. データベース移行タスクを作成する

1. AWS DMSコンソールで、[データ移行タスク](https://console.aws.amazon.com/dms/v2/home#tasks) ページに移動します。リージョンを切り替えた後、ウィンドウの右上隅にある **タスクの作成** をクリックします。

    ![タスクの作成](/media/tidb-cloud/aws-dms-to-tidb-cloud-create-task.png)

2. データベース移行タスクを作成し、**選択ルール** を指定します：

    ![AWS DMS移行タスクの作成](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-11.png)

    ![AWS DMS移行タスクの選択ルール](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-12.png)

3. タスクを作成し、開始し、タスクの完了を待ちます。

4. **表の統計** をクリックしてテーブルをチェックします。スキーマ名は `ADMIN` です。

    ![AWS DMS移行タスクの確認](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-13.png)

## ステップ9. 下流のTiDBクラスタでデータを確認する

[TiDB Serverlessクラスタ](https://tidbcloud.com/console/clusters/create-cluster)に接続し、 `admin.github_event` テーブルのデータを確認します。以下のスクリーンショットに示すように、DMSは`github_events` のテーブルと10000行のデータを正常に移行しました。

![TiDBでデータを確認](/media/tidb-cloud/aws-dms-from-oracle-to-tidb-14.png)

## 要約

AWS DMSを使用すると、このドキュメントの例に従って、どのようなアップストリームAWS RDSデータベースからもデータを正常に移行できます。

移行中に問題や失敗が発生した場合、問題をトラブルシューティングするために[CloudWatch](https://console.aws.amazon.com/cloudwatch/home)でのログ情報を確認できます。

![トラブルシューティング](/media/tidb-cloud/aws-dms-to-tidb-cloud-troubleshooting.png)

## 関連リンク

- [AWS DMSを使用してMySQL互換のデータベースから移行する](/tidb-cloud/migrate-from-mysql-using-aws-dms.md)