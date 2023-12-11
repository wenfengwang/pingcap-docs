---
title: クラウドストレージへのシンク
Summary: TiDB Dedicated クラスタからクラウドストレージ（Amazon S3、GCS など）にデータをストリームする changefeed の作成方法を学びます。

# クラウドストレージへのシンク

このドキュメントでは、TiDB Cloud からクラウドストレージにデータをストリームする changefeed の作成方法について説明します。現在、Amazon S3 と GCS がサポートされています。

> **注記:**
>
> - データをクラウドストレージにストリームするには、TiDB クラスタのバージョンが v7.1.1 以降であることを確認してください。TiDB Dedicated クラスタを v7.1.1 以降にアップグレードするには、[TiDB Cloud サポートに連絡](/tidb-cloud/tidb-cloud-support.md)してください。
> - [TiDB サーバーレス](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタでは、changefeed 機能は利用できません。

## 制限事項

- 各 TiDB Cloud クラスタには、最大 5 つの changefeed を作成できます。
- TiDB Cloud は TiCDC を使用して changefeed を確立するため、TiCDC と同じ[制限事項](https://docs.pingcap.com/tidb/stable/ticdc-overview#unsupported-scenarios)が適用されます。
- レプリケートするテーブルに主キーまたはユニークなインデックスが存在しない場合、レプリケーション中に一意の制約がないことにより、再試行のシナリオで下流に重複したデータが挿入される可能性があります。

## ステップ 1. 宛先を構成する

対象の TiDB クラスタのクラスタ概要ページに移動します。左側のナビゲーションペインで **Changefeed** をクリックし、**Create Changefeed** をクリックし、宛先として **Amazon S3** または **GCS** を選択します。宛先によって、構成プロセスが異なります。

<SimpleTab>
<div label="Amazon S3">

**Amazon S3** の場合、以下の情報を入力します: **S3 URI**、**Access Key ID**、**Secret Access Key**。S3 バケットは TiDB クラスタと同じリージョンにある必要があります。

![s3_endpoint](/media/tidb-cloud/changefeed/sink-to-cloud-storage-s3-endpoint.jpg)

</div>
<div label="GCS">

**GCS** の場合、**GCS Endpoint** を入力する前に、まず GCS バケットへのアクセスを許可する必要があります。以下の手順に従ってください:

1. TiDB Cloud コンソールで、TiDB Cloud が GCS バケットにアクセスするために使用される **Service Account ID** を記録します。

    ![gcs_endpoint](/media/tidb-cloud/changefeed/sink-to-cloud-storage-gcs-endpoint.png)

2. Google Cloud コンソールで、GCS バケットに対する IAM ロールを作成します。

    1. [Google Cloud コンソール](https://console.cloud.google.com/)にサインインします。
    2. [Roles](https://console.cloud.google.com/iam-admin/roles) ページに移動し、**Create role** をクリックします。

        ![Create a role](/media/tidb-cloud/changefeed/sink-to-cloud-storage-gcs-create-role.png)

    3. ロールの名前、説明、ID、およびロールの開始段階を入力します。ロール名は作成後に変更できません。
    4. **Add permissions** をクリックします。次の読み取り専用権限をロールに追加してから、**Add** をクリックします。

        - storage.buckets.get
        - storage.objects.create
        - storage.objects.delete
        - storage.objects.get
        - storage.objects.list
        - storage.objects.update

    ![Add permissions](/media/tidb-cloud/changefeed/sink-to-cloud-storage-gcs-assign-permission.png)

3. [Bucket](https://console.cloud.google.com/storage/browser) ページに移動し、TiDB Cloud がアクセスする GCS バケットを選択します。TiDB クラスタと同じリージョンにある必要があります。

4. **Bucket の詳細** ページで、**Permissions** タブをクリックし、**Grant access** をクリックします。

    ![Grant Access to the bucket ](/media/tidb-cloud/changefeed/sink-to-cloud-storage-gcs-grant-access-1.png)

5. 次の情報を入力してバケットへのアクセスを許可します。その後、**Save** をクリックします。

    - **New Principals** フィールドに、前述で記録した対象 TiDB クラスタの **Service Account ID** を貼り付けます。
    - **Select a role** ドロップダウンリストに、作成した IAM ロールの名前を入力し、フィルタ結果から名前を選択します。

    > **注記:**
    >
    > TiDB Cloud へのアクセスを取り消すには、単に付与したアクセスを取り外します。

6. **Bucket の詳細** ページで、**Objects** タブをクリックします。

    - バケットの gsutil URI を取得するには、コピー ボタンをクリックして、プレフィックスとして `gs://` を追加します。たとえば、バケット名が `test-sink-gcs` の場合、URI は `gs://test-sink-gcs/` になります。

        ![Get bucket URI](/media/tidb-cloud/changefeed/sink-to-cloud-storage-gcs-uri01.png)

    - フォルダの gsutil URI を取得するには、フォルダを開いてコピー ボタンをクリックし、プレフィックスとして `gs://` を追加します。たとえば、バケット名が `test-sink-gcs` でフォルダ名が `changefeed-xxx` の場合、URI は `gs://test-sink-gcs/changefeed-xxx` になります。

        ![Get bucket URI](/media/tidb-cloud/changefeed/sink-to-cloud-storage-gcs-uri02.png)

7. TiDB Cloud コンソールで、Changefeed の **Configure Destination** ページに移動し、**bucket gsutil URI** フィールドを入力します。

</div>
</SimpleTab>

**Next** をクリックして、TiDB Dedicated クラスタから Amazon S3 または GCS への接続を確立します。TiDB Cloud は自動的に接続のテストを行い、成功したかどうかを検証します。

- 成功した場合、次の構成ステップに進みます。
- 失敗した場合、接続エラーが表示され、エラーを解決した後に **Next** をクリックして再試行します。

## ステップ 2. レプリケーションを構成する

1. **Table Filter** をカスタマイズして、レプリケートするテーブルをフィルタリングします。ルール構文については、[table filter rules](https://docs.pingcap.com/tidb/stable/ticdc-filter#changefeed-log-filters)を参照してください。

    ![the table filter of changefeed](/media/tidb-cloud/changefeed/sink-to-s3-02-table-filter.jpg)

    - **Filter Rules**: このカラムにフィルタールールを設定できます。デフォルトでは、`*.*` というルールがあり、すべてのテーブルをレプリケートします。新しいルールを追加すると、TiDB Cloud は TiDB のすべてのテーブルをクエリし、ルールに一致するテーブルのみを右側のボックスに表示します。
    - **Tables with valid keys**: このカラムは、主キーまたはユニークなインデックスを持つテーブルを表示します。
    - **Tables without valid keys**: このカラムは、主キーまたはユニークなキーがないテーブルを表示します。これらのテーブルは重複したイベントを処理する際にデータの不整合を引き起こす可能性があるため、確実なデータ整合性を保つために、これらのテーブルに重複しないキーまたは主キーを追加することをお勧めします。または、これらのテーブルを除外するためにフィルタールールを使用できます。たとえば、ルール`"!test.tbl1"`を使用してテーブル `test.tbl1` を除外できます。

2. **Start Replication Position** エリアで、次のいずれかのレプリケーション位置を選択します:

    - 今後のレプリケーションを開始
    - 特定の [TSO](https://docs.pingcap.com/tidb/stable/glossary#tso) からレプリケーションを開始
    - 特定の時間からレプリケーションを開始

3. **Data Format** エリアで、**CSV** または **Canal-JSON** 形式を選択します。

    <SimpleTab>
    <div label="CSV 形式を構成する">

    **CSV** 形式を構成するには、以下のフィールドに入力します:

    - **Binary Encode Method**: バイナリデータのエンコード方法。**base64**（デフォルト）または **hex** を選択できます。AWS DMS と統合する場合は **hex** を使用します。
    - **Date Separator**: 年、月、日に基づいてデータをローテーションするか、全くローテーションしないかを選択します。
    - **Delimiter**: CSV ファイル内の値を区切るために使用する文字を指定します。カンマ（`,`）が最も一般的に使用されるデリミタです。
    - **Quote**: 区切り文字や特殊文字を含む値を囲むために使用する文字を指定します。通常、二重引用符 (`"`) がクォート文字として使用されます。
    - **Null/Empty Values**: CSV ファイルでのヌル値または空の値の表現方法を指定します。データの適切な処理と解釈のために重要です。
    - **Include Commit Ts**: CSV 行に [`commit-ts`](https://docs.pingcap.com/tidb/stable/ticdc-sink-to-cloud-storage#replicate-change-data-to-storage-services) を含めるかどうかを制御します。

    </div>
    <div label="Canal-JSON 形式を構成する">

    Canal-JSON はプレインな JSON テキスト形式です。以下のフィールドに入力します:

    - **Date Separator**: 年、月、日に基づいてデータをローテーションするか、全くローテーションしないかを選択します。
- **TiDBエクステンションを有効にする**: このオプションを有効にすると、TiCDCは[WATERMARKイベント](https://docs.pingcap.com/tidb/stable/ticdc-canal-json#watermark-event)を送信し、Canal-JSONメッセージに[TiDBエクステンションフィールド](https://docs.pingcap.com/tidb/stable/ticdc-canal-json#tidb-extension-field)を追加します。

    </div>
    </SimpleTab>

4. **Flush Parameters**エリアでは、次の2つのアイテムを構成できます:

    - **フラッシュ間隔**: デフォルトでは60秒に設定されており、2秒から10分の範囲で調整可能です;
    - **ファイルサイズ**: デフォルトでは64MBに設定されており、1MBから512MBの範囲で調整可能です。

    ![フラッシュパラメータ](/media/tidb-cloud/changefeed/sink-to-cloud-storage-flush-parameters.jpg)

    > **注意:**
    >
    > これらの2つのパラメータは、各個々のデータベーステーブルで生成されるオブジェクトの数に影響を与えます。テーブルが多い場合、同じ構成を使用すると生成されるオブジェクトの数が増え、それに伴ってクラウドストレージAPIの呼び出しコストが上がります。そのため、復旧目標（RPO）とコストの要件に基づいてこれらのパラメータを適切に構成することをお勧めします。

## Step 3. 仕様を構成する

**次へ**をクリックして、chagefeedの仕様を構成します。

1. **Changefeed Specification**エリアで、chagefeedによって使用されるレプリケーション・キャパシティ・ユニット（RCU）の数を指定します。
2. **Changefeed Name**エリアで、chagefeedの名前を指定します。

## ステップ4. 構成を確認し、レプリケーションを開始する

**次へ**をクリックして、chagefeedの構成を確認します。

- すべての構成が正しいことを確認した場合は、**作成**をクリックして、chagefeedの作成を進めます。
- いずれかの構成を変更する必要がある場合は、**前へ**をクリックして必要な変更を行います。

シンクはまもなく開始され、シンクの状態が**作成中**から**実行中**に変わることが観察されます。

chagefeedの名前をクリックして、その詳細ページに移動します。このページでは、チェックポイントの状態、レプリケーションの遅延など、chagefeedに関するより詳細な情報を表示できます。