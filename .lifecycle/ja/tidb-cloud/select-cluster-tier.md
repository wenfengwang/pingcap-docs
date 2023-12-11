---
title: クラスターティアの選択
summary: TiDB Cloud でクラスターティアを選択する方法について学びます。
aliases: ['/tidbcloud/developer-tier-cluster']
---

# クラスターティアの選択

クラスターティアは、クラスターのスループットとパフォーマンスを決定します。

TiDB Cloud では、以下の2つのクラスターティアのオプションが提供されています。クラスターを作成する前に、自分のニーズに合うオプションを検討する必要があります。

- [TiDB サーバーレス](#tidb-serverless)
- [TiDB デディケーテッド](#tidb-dedicated)

## TiDB サーバーレス

<!--To be confirmed-->
TiDB サーバーレスは、フルマネージドのマルチテナント TiDB オファリングです。インスタントでオートスケーリング可能なMySQL互換のデータベースを提供し、寛大な無料ティアと無料枠を超えた消費に基づく請求を提供します。

### 利用枠

TiDB Cloud の各組織では、デフォルトで最大5つの TiDB サーバーレスクラスターを作成できます。さらに TiDB サーバーレスクラスターを作成するには、クレジットカードを追加し、使用量に [支出制限](/tidb-cloud/tidb-cloud-glossary.md#spending-limit) を設定する必要があります。

組織内の最初の5つの TiDB サーバーレスクラスターに対して、TiDB Cloud はそれぞれ以下のような無料利用枠を提供します。

- 行ベースのストレージ：5 GiB
- [リクエストユニット（RU）](/tidb-cloud/tidb-cloud-glossary.md#request-unit)：1か月につき5000万 RU

リクエストユニット（RU）は、データベースへの単一のリクエストによって消費されるリソース量を表すための単位です。リクエストによって消費されるRUの量は、操作タイプや取得または変更されるデータの量など、さまざまな要因に依存します。

クラスターの無料利用枠を超えると、このクラスター上の読み取りおよび書き込み操作が制限され、使用量を [増やす](/tidb-cloud/manage-serverless-spend-limit.md#update-spending-limit) か、新しい月の開始時に使用量がリセットされるまで制限がかかります。例えば、クラスターのストレージが5 GiBを超えると、単一トランザクションの最大サイズ制限が10 MiBから1 MiBに減少します。

異なるリソース（読み取り、書き込み、SQL CPU、およびネットワークエグレスを含む）のRU消費量、価格の詳細、および制限情報については、[TiDB サーバーレス価格の詳細](https://www.pingcap.com/tidb-cloud-serverless-pricing-details) を参照してください。

### ユーザー名プレフィックス

<!--Important: Do not update the section name "User name prefix" because this section is referenced by TiDB backend error messages.-->

各 TiDB サーバーレスクラスターについて、TiDB Cloud はユニークなプレフィックスを生成して他のクラスターと区別します。

データベースのユーザー名を使用または設定する際は、ユーザー名にそのプレフィックスを含める必要があります。例えば、クラスターのプレフィックスが `3pTAoNNegb47Uc8` であるとします。

- クラスターに接続するには:

    ```shell
    mysql -u '3pTAoNNegb47Uc8.root' -h <host> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=<CA_root_path> -p
    ```

    > **注意:**
    >
    > TiDB サーバーレスは TLS 接続が必要です。システム上の CA ルートパスを見つけるには、[ルート証明書のデフォルトパス](/tidb-cloud/secure-connections-to-serverless-clusters.md#root-certificate-default-path) を参照してください。

- データベースユーザーを作成するには:

    ```sql
    CREATE USER '3pTAoNNegb47Uc8.jeffrey';
    ```

クラスターのプレフィックスを取得するには、以下の手順を実行してください:

1. [**クラスター**](https://tidbcloud.com/console/clusters) ページに移動します。
2. ターゲットクラスターの名前をクリックして概要ページに移動し、右上隅の **Connect** をクリックします。接続ダイアログが表示されます。
3. ダイアログで接続文字列からプレフィックスを取得します。

### TiDB サーバーレスの特別な条件と規約

一部の TiDB Cloud の機能は、TiDB サーバーレスで一部サポートされたりサポートされなかったりします。詳細については[TiDB サーバーレスの制限事項](/tidb-cloud/serverless-limitations.md) を参照してください。

## TiDB デディケーテッド

TiDB デディケーテッドは、クロスゾーン高可用性、水平スケーリング、および[HTAP](https://en.wikipedia.org/wiki/Hybrid_transactional/analytical_processing)の利点を活かした本番用途向けです。

TiDB デディケーテッドクラスターでは、TiDB、TiKV、TiFlashのクラスターサイズをビジネスニーズに応じて簡単にカスタマイズできます。各 TiKV ノードや TiFlash ノードでは、ノード上のデータが複製され、異なる可用性ゾーンに分散されて [高可用性](/tidb-cloud/high-availability-with-multi-az.md) を実珵しています。

TiDB デディケーテッドクラスターを作成するには、[支払方法を追加](/tidb-cloud/tidb-cloud-billing.md#payment-method) するか、[PoC（プルーフ オブ コンセプト）トライアルを申し込む](/tidb-cloud/tidb-cloud-poc.md) 必要があります。

> **注意:**
>
> クラスターを作成した後は、ノードのストレージを減少させることはできません。