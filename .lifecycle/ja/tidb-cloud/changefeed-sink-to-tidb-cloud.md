---
title: TiDBクラウドへのシンク
Summary: TiDB DedicatedクラスターからTiDB Serverlessクラスターへのデータストリームの作成方法について学びます。

# TiDBクラウドへのシンク

このドキュメントでは、TiDB DedicatedクラスターからTiDB Serverlessクラスターへのデータストリームの作成方法について説明します。

> **注意:**
>
> Changefeed機能を使用するには、TiDB Dedicatedクラスターバージョンがv6.4.0以降であることを確認してください。

## 制限事項

- 各TiDBクラウドクラスターにつき、最大5つのchangefeedを作成できます。
- TiDBクラウドはTiCDCを使用してchangefeedを確立するため、TiCDCと同じ[制限事項](https://docs.pingcap.com/tidb/stable/ticdc-overview#unsupported-scenarios)が適用されます。
- レプリケート対象のテーブルに主キーまたはノンヌルなユニークインデックスがない場合、リトライシナリオにおいてユニーク制約の欠如により下流に重複データが挿入される可能性があります。
- **TiDBクラウドへのシンク** 機能は、以下のAWSリージョンにある、2022年11月9日以降に作成されたTiDB Dedicatedクラスターでのみ利用可能です。

    - AWS オレゴン（us-west-2）
    - AWS フランクフルト（eu-central-1）
    - AWS シンガポール（ap-southeast-1）
    - AWS 東京（ap-northeast-1）

- ソースのTiDB Dedicatedクラスターと対象のTiDB Serverlessクラスターは、同じプロジェクトおよび同じリージョンにある必要があります。
- **TiDBクラウドへのシンク** 機能はプライベートエンドポイントを介したネットワーク接続のみをサポートしています。TiDB DedicatedクラスターからTiDB Serverlessクラスターへのデータストリームの作成時、TiDBクラウドは自動的に2つのクラスター間でプライベートエンドポイント接続を確立します。

## 必要条件

**TiDBクラウドへのシンク** コネクタは、ある特定の[TSO](https://docs.pingcap.com/tidb/stable/glossary#tso)以降のTiDB DedicatedクラスターからTiDB Serverlessクラスターへの増分データのみをレプリケートできます。

Changefeedを作成する前に、ソースのTiDB Dedicatedクラスターから既存データをエクスポートし、そのデータを対象のTiDB Serverlessクラスターにロードする必要があります。

1. [tidb_gc_life_time](https://docs.pingcap.com/tidb/stable/system-variables#tidb_gc_life_time-new-in-v50)を、次の2つの操作の合計時間よりも長い時間に設定し、その時間内の履歴データがTiDBによってガベージコレクトされないようにします。

    - 既存データのエクスポートとインポートにかかる時間
    - **TiDBクラウドへのシンク** の作成にかかる時間

    例:

    ```sql
    SET GLOBAL tidb_gc_life_time = '720h';
    ```

2. TiDB Dedicatedクラスターからデータを[バックアップ](/tidb-cloud/backup-and-restore.md#backup)し、その後、[mydumper/myloader](https://centminmod.com/mydumper.html)などのコミュニティツールを使用して、対象のTiDB Serverlessクラスターにデータをロードします。

3. [Dumplingによるエクスポートファイル](https://docs.pingcap.com/tidb/stable/dumpling-overview#format-of-exported-files)から、TiDBクラウドへのシンクの開始位置をメタデータファイルから取得します:

    以下は、例のメタデータファイルの一部です。 `SHOW MASTER STATUS` の `Pos` が、既存データのTSOであり、TiDBクラウドへのシンクの開始位置でもあります。

    ```
    Started dump at: 2023-03-28 10:40:19
    SHOW MASTER STATUS:
            Log: tidb-binlog
            Pos: 420747102018863124
    Finished dump at: 2023-03-28 10:40:20
    ```

## TiDBクラウドシンクの作成

前提条件を満たしたら、データを対象のTiDB Serverlessクラスターにシンクできます。

1. 対象のTiDBクラスターのクラスター概要ページに移動し、左側のナビゲーションペインで **Changefeed** をクリックします。

2. **Changefeedを作成** をクリックし、宛先として **TiDBクラウド** を選択します。

3. **TiDBクラウド接続** 領域で、対象のTiDB Serverlessクラスターを選択し、次に、宛先クラスターのユーザー名とパスワードを入力します。

4. 2つのTiDBクラスター間の接続を確立し、changefeedが正常に接続できるかをテストするために **次へ** をクリックします:

    - 正常に接続された場合、次の設定ステップに移動します。
    - 接続されない場合、接続エラーが表示され、エラーを処理する必要があります。エラーが解消されたら、再度 **次へ** をクリックします。

5. **Table Filter** をカスタマイズして、レプリケートしたいテーブルをフィルタリングします。ルールの構文については、[table filter rules](/table-filter.md) を参照してください。

    - **フィルタールール**: この列でフィルタールールを設定できます。デフォルトでは、`*.*` というルールがあり、すべてのテーブルがレプリケートされます。新しいルールを追加すると、TiDBクラウドはTiDBのすべてのテーブルをクエリし、ルールに一致するテーブルのみを右側のボックスに表示します。
    - **レプリケートするテーブル**: この列にはレプリケートするテーブルが表示されますが、将来的にレプリケートする新しいテーブルや完全にレプリケートするスキーマは表示されません。
    - **有効なキーのないテーブル**: この列には、ユニークキーとプライマリキーがないテーブルが表示されます。これらのテーブルについては、下流システムで重複イベントを処理するためのユニーク識別子がないため、そのデータはレプリケーション中に不整合になる可能性があります。このような問題を回避するためには、レプリケーションの前にこれらのテーブルにユニークキーまたはプライマリキーを追加するか、フィルタールールを設定してこれらのテーブルをフィルタリングすることをお勧めします。たとえば、`!test.tbl1` を使用してテーブル `test.tbl1` をフィルタリングできます。

6. **Start Replication Position** 領域で、Dumplingでエクスポートしたメタデータファイルから取得したTSOを入力します。

7. **次へ** をクリックして、changefeedの仕様を構成します。

    - **Changefeed Specification** 領域で、Changefeedが使用するReplication Capacity Units（RCU）の数を指定します。
    - **Changefeed Name** 領域で、changefeedに名前を指定します。

8. **次へ** をクリックして、Changefeedの構成を確認します。

    すべての構成が正しいことを確認し、クロスリージョンレプリケーションの準拠を確認したら、 **作成** をクリックします。

    いくつかの構成を変更する場合は、前の構成ページに戻るために **前へ** をクリックします。

9. シンクがすぐに開始され、状態が **Creating** から **Running** に変化するのを確認できます。

    changefeed名をクリックすると、チェックポイント、レプリケーション遅延、その他のメトリクスなどのchangefeedに関する詳細が表示されます。

10. シンクが作成されたら、[tidb_gc_life_time](https://docs.pingcap.com/tidb/stable/system-variables#tidb_gc_life_time-new-in-v50)を元の値（デフォルト値は `10m`）に復元します:

    ```sql
    SET GLOBAL tidb_gc_life_time = '10m';
    ```