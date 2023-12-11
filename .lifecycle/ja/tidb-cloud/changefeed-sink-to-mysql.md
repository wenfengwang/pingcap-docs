---
title: MySQLへのシンク
Summary: TiDB CloudからMySQLへデータをストリーミングするchangefeedの作成方法を学びます。

# MySQLへのシンク

このドキュメントでは、**MySQLへのシンク** changefeedを使用してTiDB CloudからMySQLへデータをストリーミングする方法について説明します。

> **注意:**
>
> - changefeed機能を使用するには、TiDB専用クラスタのバージョンがv6.4.0以降であることを確認してください。
> - [TiDB Serverlessクラスタ](/tidb-cloud/select-cluster-tier.md#tidb-serverless)の場合、changefeed機能は使用できません。

## 制限事項

- 各TiDB Cloudクラスタには、最大で5つのchangefeedを作成できます。
- TiDB CloudはTiCDCを使用してchangefeedを確立するため、TiCDCと同じ[制限事項](https://docs.pingcap.com/tidb/stable/ticdc-overview#unsupported-scenarios)が適用されます。
- 複製するテーブルに主キーまたは非NULL一意のインデックスがない場合、複製中の一部のリトライシナリオにおいて一意の制約がないことが原因で、複製データが下流に挿入され重複する可能性があります。

## 前提条件

changefeedを作成する前に、以下の前提条件を満たす必要があります。

- ネットワーク接続の設定
- MySQLへの既存データのエクスポートとロード（オプション）
- 既存データのみをロードせずに増分データをMySQLに複製する場合は、MySQLに対応するターゲットテーブルを作成する

### ネットワーク

TiDBクラスタがMySQLサービスに接続できるようにしてください。

MySQLサービスがパブリックインターネットアクセスのないAWS VPCにある場合は、次の手順を実行してください。

1. MySQLサービスのVPCとTiDBクラスタのVPCの間に[VPCピアリング接続を設定](/tidb-cloud/set-up-vpc-peering-connections.md)します。
2. MySQLサービスに関連付けられたセキュリティグループのインバウンドルールを変更します。

    [TiDB Cloudクラスタが置かれているリージョンのCIDR](/tidb-cloud/set-up-vpc-peering-connections.md#prerequisite-set-a-project-cidr)をインバウンドルールに追加する必要があります。これにより、TiDBクラスタからMySQLインスタンスへのトラフィックが許可されます。

3. MySQL URLにホスト名が含まれる場合は、TiDB CloudがMySQLサービスのDNSホスト名を解決できるようにする必要があります。

    1. [VPCピアリング接続のDNS解決を有効にする](https://docs.aws.amazon.com/vpc/latest/peering/modify-peering-connections.html#vpc-peering-dns)手順に従います。
    2. **Accepter DNS resolution**オプションを有効にします。

MySQLサービスがパブリックインターネットアクセスのないGoogle Cloud VPCにある場合は、次の手順を実行してください。

1. MySQLサービスがGoogle Cloud SQLの場合、Googleが開発した[**Cloud SQL Auth proxy**](https://cloud.google.com/sql/docs/mysql/sql-proxy)を使用して、Google Cloud SQLインスタンスの関連VPCにMySQLエンドポイントを公開する必要があります。
2. MySQLサービスのVPCとTiDBクラスタのVPCの間に[VPCピアリング接続を設定](/tidb-cloud/set-up-vpc-peering-connections.md)します。
3. MySQLが配置されているVPCのイングレスファイアウォールルールを変更します。

    [TiDB Cloudクラスタが置かれているリージョンのCIDR](/tidb-cloud/set-up-vpc-peering-connections.md#prerequisite-set-a-project-cidr)をイングレスファイアウォールルールに追加する必要があります。これにより、TiDBクラスタからMySQLエンドポイントへのトラフィックが許可されます。

### 既存データのロード（オプション）

**MySQLへのシンク**コネクタは、特定のタイムスタンプ以降のTiDBクラスタからの増分データのみをMySQLに複製できます。既にTiDBクラスタにデータがある場合は、**MySQLへのシンク**を有効にする前にTiDBクラスタの既存データをMySQLにエクスポートしてロードすることができます。

既存データをロードするには:

1. [tidb_gc_life_time](https://docs.pingcap.com/tidb/stable/system-variables#tidb_gc_life_time-new-in-v50)を、以下の2つの操作の合計時間よりも長い期間に設定してください。これにより、その期間中の歴史データがTiDBによってガベージ収集されなくなります。

    - 既存データのエクスポートとインポートにかかる時間
    - **MySQLへのシンク**の作成にかかる時間

    例:

    {{< copyable "sql" >}}

    ```sql
    SET GLOBAL tidb_gc_life_time = '720h';
    ```

2. [Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview)を使用してTiDBクラスタからデータをエクスポートし、その後、myloaderなどのコミュニティツールを使用してデータをMySQLサービスにロードします。

3. [Dumplingのエクスポートファイル](https://docs.pingcap.com/tidb/stable/dumpling-overview#format-of-exported-files)から、MySQLシンクの開始位置をメタデータファイルから取得します。

    次は、メタデータファイルの一部です。`SHOW MASTER STATUS`の`Pos`は、既存データのTSOであり、MySQLシンクの開始位置でもあります。

    ```
    Started dump at: 2020-11-10 10:40:19
    SHOW MASTER STATUS:
            Log: tidb-binlog
            Pos: 420747102018863124
    Finished dump at: 2020-11-10 10:40:20
    ```

### MySQLでターゲットテーブルを作成

既存データをロードしない場合は、TiDBから増分データを保存するための対応するターゲットテーブルをMySQLで手動で作成する必要があります。それ以外の場合、データは複製されません。

## MySQLシンクの作成

前提条件を満たしたら、データをMySQLにシンクすることができます。

1. 対象のTiDBクラスタのクラスタ概要ページに移動し、左側のナビゲーションペインで **Changefeed** をクリックします。

2. **Changefeedを作成** をクリックし、**Target Type** で **MySQL** を選択します。

3. **MySQL Connection**で MySQLエンドポイント、ユーザー名、パスワードを入力します。

4. **Next** をクリックして、TiDBがMySQLに正常に接続できるかどうかをテストします:

    - 成功した場合、次の構成手順に進みます。
    - 失敗した場合は、接続エラーが表示され、エラーを対処する必要があります。エラーが解決されたら、再度 **Next** をクリックします。

5. **Table Filter** をカスタマイズして、複製したいテーブルをフィルタリングします。ルール構文については、[table filter rules](/table-filter.md)を参照してください。

    - **フィルタールールの追加**: この列でフィルタールールを設定できます。デフォルトでは、すべてのテーブルを複製する ** `*.*`というルールがあります。新しいルールを追加すると、TiDB CloudはTiDBのすべてのテーブルをクエリし、右側のボックスにルールに一致するテーブルのみを表示します。
    - **複製するテーブル**: この列には複製するテーブルが表示されます。ただし、将来的に新しいテーブルや完全に複製されるスキーマは表示されません。
    - **有効なキーのないテーブル**: この列にはユニークキーや主キーのないテーブルが表示されます。これらのテーブルでは、下流システムが重複するイベントを処理するためのユニークな識別子がないため、複製中にデータが不一致になる可能性があります。このような問題を回避するためには、複製前にこれらのテーブルに一意のキーまたは主キーを追加するか、フィルタールールを設定してこれらのテーブルをフィルタリングすることをお勧めします。たとえば、「!test.tbl1」を使用してテーブル `test.tbl1` をフィルタリングすることができます。

6. **Start Position**で、MySQLシンクの開始位置を構成します。

    - Dumplingを使用して[既存データをロード](#load-existing-data-optional)した場合は、**特定のTSOから複製を開始**を選択し、Dumplingがエクスポートしたメタデータファイルから取得したTSOを入力します。
    - 上流のTiDBクラスタにデータがない場合は、**今から複製を開始**を選択します。
    - それ以外の場合は、**特定の時刻から複製を開始**を選択して開始時刻をカスタマイズできます。

7. **Next** をクリックして、changefeedの仕様を構成します。

    - **Changefeed Specification**エリアでは、changefeedで使用するReplication Capacity Units（RCU）の数を指定します。
    - **Changefeed Name**エリアでは、changefeedの名前を指定します。

8. **Next** をクリックして、changefeedの構成を確認します。

    すべての構成が正しいことを確認した場合は、クロスリージョン複製の適合性を確認し、**Create**をクリックします。

    一部の構成を変更したい場合は、前の構成ページに戻るために**Previous**をクリックします。

9. シンクがすぐに開始され、シンクのステータスが"**Creating**"から"**Running**"に変わります。

    changefeedの名前をクリックすると、チェックポイント、複製の遅延、その他のメトリクスなど、changefeedに関する詳細情報が表示されます。

10. [既存データをロード](#load-existing-data-optional)している場合、シンクを作成した後にGC時間を元の値（デフォルト値は `10m`）に戻す必要があります:

{{< copyable "sql" >}}

```sql
SET GLOBAL tidb_gc_life_time = '10m';
```