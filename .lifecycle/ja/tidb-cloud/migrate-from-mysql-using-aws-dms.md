---
title: MySQL互換データベースからTiDB Cloudへの移行をAWS DMSを使用して行う
summary: AWS Database Migration Service（AWS DMS）を使用してMySQL互換データベースからTiDB Cloudにデータを移行する方法について学びます。

# MySQL互換データベースからTiDB Cloudへの移行をAWS DMSを使用して行う

異種のデータベース（例: PostgreSQL、Oracle、SQL Server）をTiDB Cloudに移行する場合は、AWS Database Migration Service（AWS DMS）を使用することをお勧めします。

AWS DMSは、関係データベース、データウェアハウス、NoSQLデータベースなどのさまざまな種類のデータストアを簡単に移行できるクラウドサービスです。AWS DMSを使用して、データをTiDB Cloudに移行できます。

このドキュメントでは、Amazon RDSを使用して、AWS DMSを使用してデータをTiDB Cloudに移行する手順を示します。この手順は、自己ホスト型のMySQLデータベースやAmazon AuroraからTiDB Cloudにデータを移行する場合にも適用されます。

この例では、データソースはAmazon RDSであり、データの送信先はTiDB Cloud内のTiDB専用クラスターです。上流および下流のデータベースは同じリージョン内にあります。

## 前提条件

移行を開始する前に、以下を確認してください。

- ソースデータベースがAmazon RDSまたはAmazon Auroraの場合、「binlog_format」パラメータを「ROW」に設定する必要があります。データベースがデフォルトのパラメータグループを使用している場合、「binlog_format」パラメータはデフォルトで「MIXED」になっており、変更できません。この場合、「newset」といった新しいパラメータグループを[作成](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.Prerequisites.html#CHAP_GettingStarted.Prerequisites.params)してその「binlog_format」を「ROW」に設定する必要があります。その後、デフォルトのパラメータグループを「newset」に[変更](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithDBInstanceParamGroups.html#USER_WorkingWithParamGroups.Modifying)する必要があります。パラメータグループを変更すると、データベースが再起動します。
- ソースデータベースがTiDBと互換性のある照合を使用していることを確認し、必要に応じて修正してください。TiDBのutf8mb4文字セットのデフォルト照合は「utf8mb4_bin」ですが、MySQL 8.0ではデフォルトの照合が「utf8mb4_0900_ai_ci」です。上流のMySQLがデフォルトの照合を使用している場合、TiDBは「utf8mb4_0900_ai_ci」と互換性がないため、AWS DMSはターゲットテーブルをTiDBに作成できず、データを移行できません。この問題を解決するには、移行前にソースデータベースの照合を「utf8mb4_bin」に変更する必要があります。TiDBがサポートする文字セットと照合の完全なリストについては、「Character Set and Collation」を参照してください。
- TiDBにはデフォルトで次のシステムデータベースが含まれています: `INFORMATION_SCHEMA`、`PERFORMANCE_SCHEMA`、`mysql`、`sys`、および `test`。AWS DMS移行タスクを作成する際には、これらのシステムデータベースを選択するためにデフォルトの「%」を使用せず、特定のデータベースおよびテーブル名を記入する必要があります。そうしないと、AWS DMSはこれらのシステムデータベースをソースデータベースからターゲットのTiDBに移行しようとし、タスクが失敗する可能性があります。この問題を回避するために、特定のデータベースおよびテーブル名を入力することをお勧めします。
- AWS DMSの公開およびプライベートネットワークIPアドレスをソースおよびターゲットデータベースのIPアクセスリストに追加してください。そうしないと、ネットワーク接続が一部のシナリオで失敗する可能性があります。
- [VPCピアリング](/tidb-cloud/set-up-vpc-peering-connections.md#set-up-vpc-peering-on-aws)または[プライベートエンドポイント接続](/tidb-cloud/set-up-private-endpoint-connections.md)を使用して、AWS DMSとTiDBクラスターを接続してください。
- AWS DMSとTiDBクラスターで同じリージョンを使用することをお勧めします。これにより、データ書き込みのパフォーマンスが向上します。
- AWS DMS `dms.t3.large` (2 vCPUおよび8 GiBメモリ)またはそれ以上のインスタンスクラスを使用することをお勧めします。小さなインスタンスクラスではメモリ不足(OOM)のエラーが発生する可能性があります。
- AWS DMSは自動的にターゲットデータベースに`awsdms_control`データベースを作成します。

## 制限

AWS DMSは`DROP TABLE`のレプリケーションをサポートしていません。

## ステップ1. AWS DMSレプリケーションインスタンスを作成する

1. AWS DMSコンソールの[Replication instances](https://console.aws.amazon.com/dms/v2/home#replicationInstances)ページに移動し、対応するリージョンに切り替えます。AWS DMSとTiDB Cloudで同じリージョンを使用することをお勧めします。このドキュメントでは、上流および下流のデータベースおよびDMSインスタンスはすべて**us-west-2**リージョンにあります。

2. **Create replication instance**をクリックします。

    ![Create replication instance](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-create-instance.png)

3. インスタンス名、ARN、および説明を入力します。

4. インスタンス構成を入力します:
    - **Instance class**: 適切なインスタンスクラスを選択します。パフォーマンスを向上させるためには、`dms.t3.large`またはそれ以上のインスタンスクラスを使用することをお勧めします。
    - **Engine version**: デフォルトの構成を使用します。
    - **Multi-AZ**: ビジネスの必要に合わせて**Single-AZ**または**Multi-AZ**を選択します。

5. **Allocated storage (GiB)**フィールドにストレージを構成します。デフォルトの構成を使用します。

6. 接続性とセキュリティを構成します。
    - **Network type - new**: **IPv4**を選択します。
    - **Virtual private cloud (VPC) for IPv4**: 必要なVPCを選択します。ネットワーク構成を簡略化するために上流データベースと同じVPCを使用することをお勧めします。
    - **Replication subnet group**: レプリケーションインスタンス用のサブネットグループを選択します。
    - **Public accessible**: デフォルトの構成を使用します。

7. 必要に応じて、**Advanced settings**、**Maintenance**、および**Tags**を構成します。インスタンスの作成を完了するために **Create replication instance**をクリックします。

## ステップ2. ソースデータベースエンドポイントを作成する

1. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home)で、作成したばかりのレプリケーションインスタンスをクリックします。次のスクリーンショットに示すように、公開およびプライベートネットワークIPアドレスをコピーします。

    ![Copy the public and private network IP addresses](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-copy-ip.png)

2. Amazon RDSのセキュリティグループルールを構成します。この例では、AWS DMSインスタンスの公開およびプライベートIPアドレスをセキュリティグループに追加します。

    ![Configure the security group rules](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-rules.png)

3. **Create endpoint**をクリックしてソースデータベースエンドポイントを作成します。

    ![Click Create endpoint](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-endpoint.png)

4. この例では、**Select RDS DB instance**をクリックして、ソースRDSインスタンスを選択します。ソースデータベースが自己ホスト型のMySQLの場合、この手順はスキップして次の手順で必要な情報を入力します。

    ![Select RDS DB instance](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-select-rds.png)

5. 以下の情報を構成します:
   - **Endpoint identifier**: 次のタスク構成で識別するためにソースエンドポイントのラベルを作成します。
   - **Descriptive Amazon Resource Name (ARN) - optional**: デフォルトのDMS ARNのためのフレンドリーな名前を作成します。
   - **Source engine**: **MySQL**を選択します。
   - **Access to endpoint database**: **手動でアクセス情報を提供**を選択します。
   - **Server name**: データプロバイダーのデータサーバーの名前を入力します。データベースコンソールからコピーできます。Amazon RDSまたはAmazon Auroraが上流の場合、名前は自動的に入力されます。ドメイン名を持たない自己ホスト型のMySQLの場合はIPアドレスを入力します。
   - ソースデータベースの**Port**、**Username**、および **Password**を入力します。
   - **Secure Socket Layer (SSL) mode**: 必要に応じてSSLモードを有効にします。

    ![Fill in the endpoint configurations](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-endpoint-config.png)

6. **Endpoint settings**、**KMS key**、および**Tags**にはデフォルト値を使用します。「Test endpoint connection (optional)」セクションでは、ネットワーク構成を簡略化するためにソースデータベースと同じVPCを選択することをお勧めします。対応するレプリケーションインスタンスを選択し、**Run test**をクリックします。ステータスは**successful**である必要があります。

7. **Create endpoint**をクリックします。

    ![Click Create endpoint](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-connection.png)

## ステップ3. ターゲットデータベースエンドポイントを作成する

1. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home)で、作成したばかりのレプリケーションインスタンスをクリックします。次のスクリーンショットに示すように、公開およびプライベートネットワークIPアドレスをコピーします。

    ![Copy the public and private network IP addresses](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-copy-ip.png)

2. TiDB Cloudコンソールで、[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスター名をクリックしてから、右上隅の**Connect**をクリックしてTiDB Cloudデータベース接続情報を取得します。

3. ダイアログで**Step 1: Create traffic filter**の下で、**Edit**をクリックし、AWS DMSコンソールからコピーしたパブリックおよびプライベートネットワークIPアドレスを入力し、**Update Filter**をクリックします。AWS DMSがTiDBクラスターに接続できない場合があるため、AWS DMSレプリケーションインスタンスのパブリックIPアドレスとプライベートIPアドレスを同時にTiDBクラスタートラフィックフィルタに追加することが推奨されています。

4. **Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。次に、ダイアログで**Step 3: Connect with a SQL client**の下にある接続文字列の`-u`、`-h`、および`-P`情報を後で使用するためにメモしておきます。

5. ダイアログで**VPC Peering**タブをクリックし、**Step 1: Set up VPC**の下で**Add**をクリックして、TiDBクラスターとAWS DMSのためのVPC Peering接続を作成します。

6. 対応する情報を構成します。[VPC Peering接続の設定](/tidb-cloud/set-up-vpc-peering-connections.md)を参照してください。

7. TiDBクラスターのターゲットエンドポイントを構成します。
    - **エンドポイントのタイプ**: **ターゲットエンドポイント**を選択します。
    - **エンドポイント識別子**: エンドポイントの名前を入力します。
    - **Descriptive Amazon Resource Name (ARN) - オプション**: デフォルトのDMS ARNのための分かりやすい名前を作成します。
    - **対象エンジン**: **MySQL**を選択します。

    ![ターゲットエンドポイントの構成](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-target-endpoint.png)

8. [AWS DMSコンソール](https://console.aws.amazon.com/dms/v2/home) で**Create endpoint**をクリックして、ターゲットデータベースエンドポイントを作成し、次に以下の情報を構成します。
    - **サーバー名**: TiDBクラスターのホスト名を入力します。これは、メモした`-h`情報です。
    - **ポート**: TiDBクラスターのポート番号を入力します。これは、メモした`-P`情報です。TiDBクラスターのデフォルトポートは4000です。
    - **ユーザー名**: TiDBクラスターのユーザー名を入力します。これは、メモした`-u`情報です。
    - **パスワード**: TiDBクラスターのパスワードを入力します。
    - **Secure Socket Layer (SSL) モード**: **Verify-ca**を選択します。
    - **Add new CA certificate**をクリックして、前の手順でTiDB CloudコンソールからダウンロードしたCAファイルをインポートします。

    ![ターゲットエンドポイント情報の入力](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-target-endpoint2.png)

9. CAファイルをインポートします。

    ![CAのアップロード](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-upload-ca.png)

10. **エンドポイントの設定**、**KMSキー**、および**タグ**のデフォルト値を使用します。**エンドポイント接続のテスト (オプション)**セクションで、ソースデータベースと同じVPCを選択します。対応するレプリケーションインスタンスを選択し、**Run test**をクリックします。ステータスは**successful**である必要があります。

11. **Create endpoint**をクリックします。

    ![Create endpointをクリック](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-target-endpoint3.png)

## Step 4.データベース移行タスクを作成する

1. AWS DMSコンソールで、[Data migration tasks](https://console.aws.amazon.com/dms/v2/home#tasks)ページに移動します。リージョンを切り替えます。その後、ウィンドウの右上隅にある**Create task**をクリックします。

    ![タスクの作成](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-create-task.png)

2. 次の情報を構成します。
    - **Task identifier**: タスクに名前を入力します。覚えやすい名前を使用することをお勧めします。
    - **Descriptive Amazon Resource Name (ARN) - optional**: デフォルトのDMS ARNのための分かりやすい名前を作成します。
    - **Replication instance**: 作成したばかりのAWS DMSインスタンスを選択します。
    - **Source database endpoint**: 作成したソースデータベースエンドポイントを選択します。
    - **Target database endpoint**: 作成したターゲットデータベースエンドポイントを選択します。
    - **Migration type**: 必要に応じた移行タイプを選択します。この例では、**Migrate existing data and replicate ongoing changes**を選択します。

    ![タスクの構成](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-task-config.png)

3. 次の情報を構成します。
    - **Editing mode**: **Wizard**を選択します。
    - **Custom CDC stop mode for source transactions**: デフォルト設定を使用します。
    - **Target table preparation mode**: **Do nothing**または必要な他のオプションを選択します。この例では、**Do nothing**を選択します。
    - **Stop task after full load completes**: デフォルト設定を使用します。
    - **Include LOB columns in replication**: **Limited LOB mode**を選択します。
    - **Maximum LOB size in (KB)**: デフォルト値**32**を使用します。
    - **Turn on validation**: 必要に応じて選択します。
    - **Task logs**: 今後のトラブルシューティングのために**Turn on CloudWatch logs**を選択します。関連する構成についてはデフォルト設定を使用します。

    ![タスクの設定](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-task-settings.png)

4. **Table mappings**セクションで、移行するデータベースを指定します。

    スキーマ名はAmazon RDSインスタンス内のデータベース名です。**Source name**のデフォルト値は"%"で、これはAmazon RDSのすべてのデータベースがTiDBに移行されることを意味します。結果として、Amazon RDS内の`mysql`や`sys`などのシステムデータベースがTiDBクラスターに移行され、タスクの失敗につながる可能性があります。したがって、特定のデータベース名を入力するか、すべてのシステムデータベースをフィルタリングすることが推奨されます。例えば、以下のスクリーンショットの設定に従い、データベース名が`franktest`であるデータベースとそのデータベース内のすべてのテーブルのみが移行されます。

    ![テーブルのマッピング](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-table-mappings.png)

5. 右下隅にある**Create task**をクリックします。

6. [Data migration tasks](https://console.aws.amazon.com/dms/v2/home#tasks)ページに戻ります。リージョンを切り替えます。タスクのステータスや進捗状況を確認できます。

    ![タスクのステータス](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-task-status.png)

移行中に問題や失敗が発生した場合は、問題をトラブルシューティングするために[CloudWatch](https://console.aws.amazon.com/cloudwatch/home)でログ情報を確認できます。

![トラブルシューティング](/media/tidb-cloud/aws-dms-tidb-cloud/aws-dms-to-tidb-cloud-troubleshooting.png)

## 関連情報

- Aurora MySQLやAmazon Relational Database Service (RDS)などのMySQL互換データベースからTiDB Cloudに移行する場合は、[TiDB Cloudでのデータ移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を使用することをお勧めします。

- Amazon RDS for OracleからTiDB Serverless Using AWS DMSに移行したい場合は、[AWS DMSを使用したAmazon RDS for OracleからTiDB Serverlessへの移行](/tidb-cloud/migrate-from-oracle-using-aws-dms.md)を参照してください。