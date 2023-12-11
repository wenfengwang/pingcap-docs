---
title: Apache Kafka へのシンク (ベータ)
Summary: TiDB Cloud から Apache Kafka へデータをストリームする changefeed の作成方法について説明します。

# Apache Kafka へのシンク (ベータ)

このドキュメントでは、TiDB Cloud から Apache Kafka へデータをストリームする changefeed を作成する方法について説明します。

> **注:**
>
> - 現在、Kafka シンクは **ベータ** です。changefeed 機能を使用するには、TiDB Dedicated クラスタバージョンが v6.4.0 以降であることを確認してください。
> - [TiDB Serverless クラスタ](/tidb-cloud/select-cluster-tier.md#tidb-serverless)の場合、changefeed 機能は利用できません。

## 制限事項

- 各 TiDB Cloud クラスタにつき、最大 5 つの changefeed を作成できます。
- 現在、TiDB Cloud は自己署名 TLS 証明書を Kafka ブローカに接続するためにサポートしていません。
- TiDB Cloud は TiCDC を使用して changefeed を確立するため、TiCDC と同じ[制限事項](https://docs.pingcap.com/tidb/stable/ticdc-overview#unsupported-scenarios)が適用されます。
- レプリケートするテーブルに主キーまたは非 NULL のユニークインデックスがない場合、レプリケーション中に一意の制約が不足しているため、リトライシナリオで重複データが下流に挿入される可能性があります。

## 前提条件

Apache Kafka にデータをストリームする changefeed を作成する前に、以下の前提条件を満たす必要があります。

- ネットワーク接続の設定
- Kafka ACL 認可の権限を追加

### ネットワーク

TiDB クラスタが Apache Kafka サービスに接続できることを確認してください。

Apache Kafka サービスがインターネットアクセスのない AWS VPC にある場合は、次の手順を実行します。

1. Apache Kafka サービスの VPC と TiDB クラスタの VPC の間に[VPC ピアリング接続](/tidb-cloud/set-up-vpc-peering-connections.md)を設定します。
2. Apache Kafka サービスに関連付けられたセキュリティグループのインバウンドルールを変更します。

    Apache Kafka サービスが関連付けられているセキュリティグループに、TiDB Cloud クラスタのリージョンの CIDR を追加する必要があります。CIDR は**VPC ピアリング**ページで見つけることができます。これにより、TiDB クラスタから Kafka ブローカへのトラフィックが流れるようになります。

3. Apache Kafka URL がホスト名を含む場合は、TiDB Cloud が Apache Kafka ブローカの DNS ホスト名を解決できるようにする必要があります。

    1. [VPC ピアリング接続の DNS 解決を有効にする](https://docs.aws.amazon.com/vpc/latest/peering/modify-peering-connections.html#vpc-peering-dns)手順に従います。
    2. **受入側 DNS 解決**オプションを有効にします。

Apache Kafka サービスがインターネットアクセスのない Google Cloud VPC にある場合は、次の手順を実行します。

1. Apache Kafka サービスの VPC と TiDB クラスタの VPC の間に[VPC ピアリング接続](/tidb-cloud/set-up-vpc-peering-connections.md)を設定します。
2. Apache Kafka が存在する VPC のイングレスファイアウォールルールを変更します。

    Apache Kafka が存在する VPC のイングレスファイアウォールルールに、TiDB Cloud クラスタのリージョンの CIDR を追加する必要があります。CIDR は**VPC ピアリング**ページで見つけることができます。これにより、TiDB クラスタから Kafka ブローカへのトラフィックが流れるようになります。

### Kafka ACL 認可の権限を追加

TiDB Cloud changefeed が Apache Kafka にデータをストリームし、Kafka トピックを自動的に作成できるようにするには、Kafka に以下の権限が追加されていることを確認してください:

- Kafka のトピックリソースタイプに対する `Create` と `Write` の権限が追加されています。
- Kafka のクラスタリソースタイプに対する `DescribeConfigs` の権限が追加されています。

例えば、Kafka クラスタが Confluent Cloud にある場合は、より詳しい情報については Confluent のドキュメントの[リソース](https://docs.confluent.io/platform/current/kafka/authorization.html#resources)と[ACL の追加](https://docs.confluent.io/platform/current/kafka/authorization.html#adding-acls)を参照してください。

## ステップ 1. Apache Kafka の changefeed ページを開く

1. [TiDB Cloud コンソール](https://tidbcloud.com)にアクセスし、対象の TiDB クラスタのクラスタ概要ページに移動し、左側のナビゲーションペインで**Changefeed**をクリックします。
2. **Changefeed を作成**をクリックし、**Target Type** として **Kafka** を選択します。

## ステップ 2. changefeed ターゲットを構成する

1. **Brokers Configuration** の下で、Kafka ブローカのエンドポイントを入力します。複数のエンドポイントを区切るためにコンマ`,`を使用できます。
2. Kafka バージョンを選択します。それがわからない場合は、Kafka V2 を使用してください。
3. この changefeed のデータに対する希望の圧縮タイプを選択します。
4. Kafka が TLS 暗号化を有効にしており、Kafka 接続に TLS 暗号化を使用したい場合は、**TLS 暗号化**オプションを有効にします。
5. Kafka が認証を必要とする場合は、対応する認証タイプを選択し、認証用のユーザー名とパスワードを入力します。

    - Kafka が認証を必要としない場合は、デフォルトの**DISABLE**オプションを保持してください。
    - Kafka が認証を必要とする場合は、対応する認証タイプを選択し、認証用のユーザー名とパスワードを入力してください。

6. **次へ**をクリックして設定した構成を確認し、次のページに進みます。

## ステップ 3. changefeed を設定する

1. **Table Filter**をカスタマイズしてレプリケートするテーブルをフィルタリングします。ルールの構文については、[テーブルフィルタのルール](/table-filter.md)を参照してください。

    - **フィルタルールを追加**: この列でフィルタルールを設定できます。デフォルトでは、`*.*`というルールがあり、これはすべてのテーブルをレプリケートすることを意味します。新しいルールを追加すると、TiDB Cloud が TiDB のすべてのテーブルをクエリし、ルールに一致するテーブルのみを**レプリケートするテーブル**列に表示します。
    - **レプリケートするテーブル**: この列にはレプリケートするテーブルが表示されますが、未来にレプリケートされる新しいテーブルや完全にレプリケートされるスキーマは表示されません。
    - **有効なキーがないテーブル**: この列には一意の主キーやユニークインデックスがないテーブルが表示されます。これらのテーブルに対しては、下流システムで重複イベントを処理するためのユニークな識別子が使用されないため、レプリケーション中にデータが不整合になる可能性があります。このような問題を回避するためには、レプリケーションの前にこれらのテーブルに一意のキーまたは主キーを追加するか、フィルタルールを設定してこれらのテーブルを除外することをお勧めします。例えば、"!test.tbl1"を使用してテーブル`test.tbl1`をフィルタリングできます。

2. **Data Format** 領域で、Kafka メッセージの希望のフォーマットを選択します。

   - Avro はリッチなデータ構造を持つコンパクトで高速なバイナリデータ形式であり、様々なフローシステムで広く使用されています。詳細については、[Avro データフォーマット](https://docs.pingcap.com/tidb/stable/ticdc-avro-protocol)を参照してください。
   - Canal-JSON はプレーンな JSON テキスト形式であり、解析が容易です。詳細については、[Canal-JSON データフォーマット](https://docs.pingcap.com/tidb/stable/ticdc-canal-json)を参照してください。

3. **TiDB Extension**オプションを有効にすると、Kafka メッセージボディに TiDB 拡張フィールドを追加できます。

    TiDB 拡張フィールドの詳細については、[Avro データフォーマットの TiDB 拡張フィールド](https://docs.pingcap.com/tidb/stable/ticdc-avro-protocol#tidb-extension-fields)および[Canal-JSON データフォーマットの TiDB 拡張フィールド](https://docs.pingcap.com/tidb/stable/ticdc-canal-json#tidb-extension-field)を参照してください。

4. データフォーマットとして **Avro** を選択した場合、ページにいくつかの Avro 固有の構成が表示されます。以下のようにこれらの構成を入力できます。

    - **Decimal** と **Unsigned BigInt** の構成では、Kafka メッセージでの Decimal および Unsigned BigInt データ型の処理方法を指定します。
    - **Schema Registry** 領域では、スキーマレジストリのエンドポイントを入力します。**HTTP 認証**を有効にすると、ユーザー名とパスワードのフィールドが表示され、TiDB クラスタのエンドポイントとパスワードが自動的に入力されます。

5. **Topic Distribution** 領域で、配布モードを選択し、モードに応じてトピック名の構成を入力します。

    データフォーマットとして **Avro** を選択した場合、**Distribution Mode** ドロップダウンリストで **Distribute changelogs by table to Kafka Topics** モードしか選択できません。

    配布モードは changefeed が Kafka トピックを作成する方法を制御し、テーブル毎、データベース毎、または全ての変更ログ用に1つのトピックを作成します。

   - **Distribute changelogs by table to Kafka Topics**

        changefeed が各テーブルに専用の Kafka トピックを作成するようにしたい場合は、このモードを選択します。その後、テーブルのすべての Kafka メッセージが専用の Kafka トピックに送信されます。テーブルの変更ログ以外の非行イベント（例: スキーマ作成イベント）のために、**Default Topic Name**フィールドでトピック名を指定できます。changefeed はこれに応じてトピックを作成します。

   - **Distribute changelogs by database to Kafka Topics**

```
        各データベースごとに専用のKafkaトピックを作成する場合は、このモードを選択してください。すると、データベースのすべてのKafkaメッセージが専用のKafkaトピックに送信されます。データベースのトピック名は、トピックプレフィックスとサフィックスを設定することでカスタマイズできます。

        Resolved Ts Eventなどの非行イベントのchangelogの場合は、「デフォルトトピック名」フィールドにトピック名を指定できます。changelogがそのようなトピック名に従って作成されます。

   - **すべてのchangelogを指定されたKafkaトピックに送信**

        すべてのchangelogのために1つのKafkaトピックを作成する場合は、このモードを選択してください。その後、changelogのすべてのKafkaメッセージが1つのKafkaトピックに送信されます。**トピック名**フィールドでトピック名を定義できます。

6. **パーティション分布**エリアでは、Kafkaメッセージをどのパーティションに送信するかを決定できます。

   - **行のインデックス値によるchangelogのKafkaパーティションへの分散**

        テーブルのKafkaメッセージを異なるパーティションに送信することを希望する場合は、この分散方法を選択してください。行のchangelogのインデックス値によって、changelogが送信されるパーティションが決まります。この分散方法はより良いパーティションバランスを提供し、行レベルの順序性を確保します。

   - **テーブルごとのchangelogのKafkaパーティションへの分散**

        テーブルのKafkaメッセージを1つのKafkaパーティションに送信することを希望する場合は、この分散方法を選択してください。行のchangelogのテーブル名によって、changelogが送信されるパーティションが決まります。この分散方法はテーブルの整合性を確保しますが、パーティションの不均衡を引き起こす可能性があります。

7. **トピック構成**エリアでは、以下の数値を設定できます。changelogは数値に従って自動的にKafkaトピックを作成します。

   - **レプリケーションファクター**：各Kafkaメッセージが複製されるKafkaサーバーの数を制御します。
   - **パーティション数**：トピック内に存在するパーティションの数を制御します。

8. **次へ**をクリックします。

## Step 4. changefeed仕様を設定する

1. **Changefeed Specification**エリアにて、chagefeedで使用するReplication Capacity Units（RCU）の数を指定してください。
2. **Changefeed Name**エリアにて、chagefeedの名前を指定してください。
3. **次へ**をクリックして、設定を確認し、次のページに進んでください。

## Step 5. 設定の確認

このページでは、設定したすべてのchagefeed設定を確認できます。

エラーがある場合は戻って修正できます。エラーがない場合は、一番下のチェックボックスをクリックし、**作成**をクリックしてchagefeedを作成できます。
```