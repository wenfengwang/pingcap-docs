---
title: DBeaverを使用してTiDBに接続する
summary: DBeaver Communityを使用してTiDBに接続する方法を学びます。

# DBeaverを使用してTiDBに接続する

TiDBはMySQL互換データベースであり、[DBeaver Community](https://dbeaver.io/download/)は開発者、データベース管理者、アナリスト、およびデータを扱う全ての人向けの無料クロスプラットフォームデータベースツールです。

このチュートリアルでは、DBeaver Communityを使用してTiDBクラスタに接続する方法を学ぶことができます。

> **注：**
>
> このチュートリアルはTiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと互換性があります。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- [DBeaver Community **23.0.3** 以上](https://dbeaver.io/download/)
- TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合は、次のように作成できます。**

- (推奨) [TiDB Serverless クラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[プロダクションTiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合は、次のように作成できます。**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[プロダクションTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## TiDBに接続する

接続するTiDBクラスタは、選択したTiDBデプロイメントオプションに応じて異なります。

<SimpleTab>
<div label="TiDB Serverless">

1. [クラスタ](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が操作環境に一致することを確認します。

    - **エンドポイントタイプ**が `Public` に設定されていること
    - **Connect With** が `JDBC` に設定されていること
    - **Operating System** が環境に一致していること

4. ランダムなパスワードを作成するには、**Create password**をクリックします。

    > **ヒント：**
    >
    > 以前にパスワードを作成している場合は、元のパスワードを使用するか、新しいパスワードを生成するために**Reset password**をクリックします。

5. DBeaverを起動し、左上隅の**New Database Connection**をクリックします。**Connect to a database**ダイアログで、リストから**TiDB**を選択して**Next**をクリックします。

    ![DBeaverでデータベースを選択](/media/develop/dbeaver-select-database.jpg)

6. TiDB Cloud接続ダイアログからJDBC文字列をコピーします。DBeaverで、**Connect by**に**URL**を選択し、JDBC文字列を**URL**フィールドに貼り付けます。文字列内の`<your_password>`プレースホルダを実際のパスワードに置換する必要はありません。なぜなら、DBeaverが**Authentication (Database Native)**セクションからユーザー名とパスワードを読み込むからです。

7. **Authentication (Database Native)**セクションに、**Username**および**Password**を入力します。例は次のようになります。

    ![TiDB Serverless用の接続設定を構成](/media/develop/dbeaver-connection-settings-serverless.jpg)

8. **Test Connection**をクリックしてTiDB Serverlessクラスタへの接続を検証します。

    **Download driver files**ダイアログが表示された場合は、ドライバーファイルを取得するために**Download**をクリックします。

    接続テストが成功した場合、以下のように**Connection test**ダイアログが表示されます。**OK**をクリックして閉じます。

    ![接続テスト結果](/media/develop/dbeaver-connection-test.jpg)

9. 接続構成を保存するには、**Finish**をクリックします。

</div>
<div label="TiDB Dedicated">

1. [クラスタ](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Anywhereからのアクセスを許可**をクリックし、次に**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. DBeaverを起動し、左上隅の**New Database Connection**をクリックします。**Connect to a database**ダイアログで、リストから**TiDB**を選択して**Next**をクリックします。

    ![DBeaverでデータベースを選択](/media/develop/dbeaver-select-database.jpg)

5. 適切な接続文字列をDBeaverの接続パネルにコピーして貼り付けます。DBeaverのフィールドとTiDB Dedicated接続文字列のマッピングは次の通りです。

    | DBeaverフィールド | TiDB Dedicated接続文字列 |
    |---------------| ------------------------------- |
    | サーバーホスト  | `{host}`                        |
    | ポート          | `{port}`                        |
    | ユーザー名      | `{user}`                        |
    | パスワード      | `{password}`                    |

    例は次のようになります。

    ![TiDB Dedicated用の接続設定を構成](/media/develop/dbeaver-connection-settings-dedicated.jpg)

6. **Test Connection**をクリックしてTiDB Dedicatedクラスタへの接続を検証します。

    **Download driver files**ダイアログが表示された場合は、ドライバーファイルを取得するために**Download**をクリックします。

    接続テストが成功した場合、以下のように**Connection test**ダイアログが表示されます。**OK**をクリックして閉じます。

    ![接続テスト結果](/media/develop/dbeaver-connection-test.jpg)

7. 接続構成を保存するには、**Finish**をクリックします。

</div>
<div label="TiDB Self-Hosted">

1. DBeaverを起動し、左上隅の**New Database Connection**をクリックします。**Connect to a database**ダイアログで、リストから**TiDB**を選択して**Next**をクリックします。

    ![DBeaverでデータベースを選択](/media/develop/dbeaver-select-database.jpg)

2. 次の接続パラメータを構成します。

    - **Server Host**：TiDB Self-HostedクラスタのIPアドレスまたはドメイン名
    - **Port**：TiDB Self-Hostedクラスタのポート番号
    - **Username**：TiDB Self-Hostedクラスタに接続するためのユーザー名
    - **Password**：ユーザー名のパスワード

    例は次のようになります。

    ![TiDB Self-Hosted用の接続設定を構成](/media/develop/dbeaver-connection-settings-self-hosted.jpg)

3. **Test Connection**をクリックしてTiDB Self-Hostedクラスタへの接続を検証します。

    **Download driver files**ダイアログが表示された場合は、ドライバーファイルを取得するために**Download**をクリックします。

    接続テストが成功した場合、以下のように**Connection test**ダイアログが表示されます。**OK**をクリックして閉じます。

    ![接続テスト結果](/media/develop/dbeaver-connection-test.jpg)

4. 接続構成を保存するには、**Finish**をクリックします。

</div>
</SimpleTab>

## 次のステップ

- [DBeaverのドキュメント](https://github.com/dbeaver/dbeaver/wiki)からDBeaverのさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一表の読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)などです。
- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格して[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

```
      Ask questions on the [Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc), or [create a support ticket](https://support.pingcap.com/).

</CustomContent>
```
```
      [Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、 [サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>
```