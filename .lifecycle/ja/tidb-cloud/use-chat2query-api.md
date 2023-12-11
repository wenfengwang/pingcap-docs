---
title: Chat2Query APIの利用を開始する
summary: TiDB Cloud Chat2Query APIを利用して、AIを使用してSQLステートメントを生成し実行する方法を学びます。
---

# Chat2Query APIの利用を開始する

TiDB CloudはChat2Query APIを提供しており、このRESTfulインターフェイスを使用して指示を提供することでAIを使用してSQLステートメントを生成し実行することができます。その後、APIはクエリ結果を返します。

Chat2Query APIはHTTPSを介してのみアクセスでき、ネットワーク経由で送信されるすべてのデータがTLSを使用して暗号化されることを保証します。

> **注意:**
>
> Chat2Query APIは[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタで利用可能です。 [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタでChat2Query APIを使用するには、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)にお問い合わせください。

## 開始する前に

Chat2Query APIを使用する前に、TiDBクラスタを作成し、[AIを使用してSQLクエリを生成](/tidb-cloud/explore-data-with-chat2query.md)するように有効にしていることを確認してください。TiDBクラスタがない場合は、[TiDB Serverlessクラスタの作成](/tidb-cloud/create-tidb-cluster-serverless.md)または[TiDB Dedicatedクラスタの作成](/tidb-cloud/create-tidb-cluster.md)の手順に従って作成してください。

## 手順1 Chat2Query APIを有効にする

Chat2Query APIを有効にするには、次の手順を実行します。

1. プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動します。

    > **ヒント:**
    >
    > 複数のプロジェクトがある場合は、左下隅の<MDSvgIcon name="icon-left-projects" />をクリックして別のプロジェクトに切り替えることができます。

2. クラスタ名をクリックし、左側のナビゲーションペインで**Chat2Query**をクリックします。
3. Chat2Queryの右上隅にある**...**をクリックし、**Settings**を選択します。
4. **DataAPI**を有効にし、Chat2Query Data Appが作成されます。

    > **注意:**
    >
    > 1つのTiDBクラスタでDataAPIが有効になると、同じプロジェクトのすべてのTiDBクラスタでChat2Query APIが使用できるようになります。

5. メッセージの**Data Service**リンクをクリックして、Chat2Query APIにアクセスします。

    左側のペインに**Chat2Query System** [Data App](/tidb-cloud/tidb-cloud-glossary.md#data-app) およびその**Chat2Data** [endpoint](/tidb-cloud/tidb-cloud-glossary.md#endpoint) が表示されます。

## 手順2 APIキーを作成する

エンドポイントを呼び出す前に、APIキーを作成する必要があります。 Chat2Query Data AppのためにAPIキーを作成するには、次の手順を実行します。

1. [**Data Service**](https://tidbcloud.com/console/data-service)の左側のペインで、**Chat2Query System**の名前をクリックして詳細を表示します。
2. **Authentication**エリアで、**Create API Key**をクリックします。
3. **Create API Key**ダイアログボックスで、説明を入力し、APIキーのロールを選択します。

    ロールは、APIキーがデータAppにリンクされたクラスタに対してデータを読み取るか書き込むかを制御するために使用されます。 `ReadOnly`または`ReadAndWrite`のロールを選択できます:

    - `ReadOnly`: APIキーが`SELECT`、`SHOW`、`USE`、`DESC`、`EXPLAIN`ステートメントなどのデータを読み取ることのみを許可します。
    - `ReadAndWrite`: APIキーはデータの読み書きを許可します。このAPIキーを使用して、すべてのSQLステートメント、DMLおよびDDLステートメントなどを実行できます。

4. **Next**をクリックします。公開キーと秘密キーが表示されます。

    安全な場所に秘密キーをコピーして保存してください。このページを離れた後では、再び完全な秘密キーを取得することはできません。

5. **Done**をクリックします。

## 手順3 Chat2Dataエンドポイントを呼び出す

[**Data Service**](https://tidbcloud.com/console/data-service)ページの左側のペインで、**Chat2Query** > **/chat2data**をクリックしてエンドポイントの詳細を表示します。 Chat2Dataの**Properties**が表示されます:

- **Endpoint Path**: （読み取り専用） Chat2Dataエンドポイントのパス、つまり`/chat2data`です。

- **Endpoint URL**: （読み取り専用） Chat2DataエンドポイントのURLで、エンドポイントを呼び出すために使用されます。 例えば、 `https://<region>.data.tidbcloud.com/api/v1beta/app/chat2query-<ID>/endpoint/chat2data` です。

- **Request Method**: （読み取り専用） Chat2DataエンドポイントのHTTPメソッドで、`POST`です。

- **Timeout(ms)**: Chat2Dataエンドポイントのタイムアウト（ミリ秒）です。

- **Max Rows**: Chat2Dataエンドポイントが返す行の最大数です。

TiDB Cloudはエンドポイントを呼び出すためのコード例を生成します。コードを取得して実行する手順は次のとおりです。

1. 現在の**Chat2Data**ページで、**Endpoint URL**の右側にある**Code Example**をクリックします。 **Code Example**ダイアログボックスが表示されます。
2. ダイアログボックスで、エンドポイントを呼び出すために使用するクラスタとデータベースを選択し、コード例をコピーします。
3. コード例をアプリケーションに貼り付けて実行します。

    - `<Public Key>`および`<Private Key>`のプレースホルダーをAPIキーで置き換えます。
    - `<your instruction>`プレースホルダーをAIにSQLステートメントを生成し実行するための指示に置き換えます。
    - `<your table name, optional>`プレースホルダーをクエリしたいテーブル名に置き換えます。テーブル名を指定しない場合、AIはデータベース内のすべてのテーブルをクエリします。

> **注意:**
>
> 各Chat2Query Data Appには、1日あたり100件のリクエストのレート制限があります。 レート制限を超えると、APIは`429`エラーを返します。 より多くのクォータをご希望の場合は、[リクエストを送信](https://support.pingcap.com/hc/en-us/requests/new?ticket_form_id=7800003722519)してください。

以下のコード例は、`sample_data.github_events`テーブルから最も人気のあるGitHubリポジトリを見つけるために使用されます:

```bash
curl --digest --user '<Public Key>:<Private Key>' \
  --request POST 'https://<region>.data.tidbcloud.com/api/v1beta/app/chat2query-<ID>/endpoint/chat2data' \
  --header 'content-type: application/json' \
  --data-raw '{
      "cluster_id": "12345678912345678960",
      "database": "sample_data",
      "tables": ["github_events"],
      "instruction": "Find the most popular repo from GitHub events"
      }'
```

上記の例では、リクエスト本文は次のプロパティを持つJSONオブジェクトです:

- `cluster_id`: _string_. TiDBクラスタのユニークな識別子。
- `database`: _string_. データベースの名前。
- `tables`: _array_. （オプション）クエリするテーブル名のリスト。
- `instruction`: _string_. 欲しいクエリを記述した自然言語の指示。

レスポンスは次のようになります:

```json
{
    "type": "chat2data_endpoint",
    "data": {
        "columns": [
            {
                "col": "repo_name",
                "data_type": "VARCHAR",
                "nullable": false
            },
            {
                "col": "count",
                "data_type": "BIGINT",
                "nullable": false
            }
        ],
        "rows": [
            {
                "count": "2390",
                "repo_name": "pytorch/pytorch"
            }
        ],
        "result": {
            "code": 200,
            "message": "Query OK!",
            "start_ms": 1678965476709,
            "end_ms": 1678965476839,
            "latency": "130ms",
            "row_count": 1,
            "row_affect": 0,
            "limit": 50,
            "sql": "SELECT sample_data.github_events.`repo_name`, COUNT(*) AS count FROM sample_data.github_events GROUP BY sample_data.github_events.`repo_name` ORDER BY count DESC LIMIT 1;",
            "ai_latency": "30ms"
        }
    }
}
```

APIの呼び出しが成功しない場合は、`200`以外のステータスコードが返されます。 `500`ステータスコードの例は以下の通りです:

```json
{
    "type": "chat2data_endpoint",
    "data": {
        "columns": [],
        "rows": [],
        "result": {
            "code": 500,
            "message": "internal error! defaultPermissionHelper: rpc error: code = DeadlineExceeded desc = context deadline exceeded",
            "start_ms": "",
            "end_ms": "",
            "latency": "",
            "row_count": 0,
            "row_affect": 0,
            "limit": 0
        }
    }
}
```

## 詳細を学ぶ

- [APIキーの管理](/tidb-cloud/data-service-api-key.md)
- [Data Serviceの応答およびステータスコード](/tidb-cloud/data-service-response-and-status-code.md)