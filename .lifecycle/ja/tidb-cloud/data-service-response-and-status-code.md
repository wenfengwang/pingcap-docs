---
title: データサービスのレスポンスとHTTPステータスコード
summary: この文書はTiDB CloudのData ServiceのレスポンスとHTTPステータスコードについて説明しています。

# データサービスのレスポンスとHTTPステータスコード

[データサービス](/tidb-cloud/data-service-overview.md) で定義されたAPIエンドポイントを呼び出すと、データサービスはHTTPレスポンスを返します。このレスポンスの構造とステータスコードの意味を理解することは、Data Serviceエンドポイントから返されるデータを解釈するために不可欠です。

この文書はTiDB CloudのData Serviceのレスポンスとステータスコードについて説明しています。

## レスポンス

Data ServiceはJSONボディを含むHTTPレスポンスを返します。

> **注意:**
>
> 複数のSQLステートメントを含むエンドポイントを呼び出すと、Data Serviceはステートメントを一つずつ実行しますが、HTTPレスポンスでは最後のステートメントの実行結果のみを返します。

レスポンスボディには以下のフィールドが含まれます:

- `type`: _string_. このエンドポイントのタイプ。値は`"sql_endpoint"`または`"chat2data_endpoint"`のいずれかです。異なるエンドポイントは異なるタイプのレスポンスを返します。
- `data`: _object_. 実行結果であり以下の三つの部分からなります:

    - `columns`: _array_. 返されたフィールドのスキーマ情報。
    - `rows`: _array_. `key:value`形式で返された結果。

        エンドポイントに**バッチ操作**が有効になっており、エンドポイントの最後のSQLステートメントが`INSERT`、`UPDATE`、または`DELETE`操作である場合、次の点に注意してください:

        - エンドポイントの返された結果には、各行の応答とステータスを示す`"message"`と`"success"`フィールドも含まれます。
        - 対象テーブルの主キーカラムが`auto_increment`として構成されている場合、エンドポイントの返された結果には各行ごとに`"auto_increment_id"`フィールドも含まれます。このフィールドの値は`INSERT`操作の場合の自動増分IDであり、`UPDATE`や`DELETE`などの他の操作では`null`です。

    - `result`: _object_. 成功/失敗のステータス、実行時間、返された行数、ユーザー設定などのSQLステートメントに関連する情報。

以下に例を示します:

<SimpleTab>
<div label="SQLエンドポイント">

```json
{
    "type": "sql_endpoint",
    "data": {
        "columns": [],
        "rows": [
            {
                "auto_increment_id": "270001",
                "index": "0",
                "message": "Row insert successfully",
                "success": "true"
            },
            {
                "auto_increment_id": "270002",
                "index": "1",
                "message": "Row insert successfully",
                "success": "true"
            }
        ],
        "result": {
            "code": 200,
            "message": "Query OK, 2 rows affected (8.359 sec)",
            "start_ms": 1689593360560,
            "end_ms": 1689593368919,
            "latency": "8.359s",
            "row_count": 2,
            "row_affect": 2,
            "limit": 500
        }
    }
}
```

</div>

<div label="Chat2Dataエンドポイント">

```json
{
  "type": "chat2data_endpoint",
  "data": {
    "columns": [
      {
        "col": "id",
        "data_type": "BIGINT",
        "nullable": false
      },
      {
        "col": "type",
        "data_type": "VARCHAR",
        "nullable": false
      }
    ],
    "rows": [
      {
        "id": "20008295419",
        "type": "CreateEvent"
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
      "limit": 50
      "sql": "select id,type from sample_data.github_events limit 1;",
      "ai_latency": "30ms"
    }
  }
}
```

</div>
</SimpleTab>

## ステータスコード

### 200

HTTPステータスコードが`200`であり、`data.result.code`フィールドも`200`を示す場合は、SQLステートメントが正常に実行されたことを示します。それ以外の場合は、TiDB Cloudはエンドポイントで定義されたSQLステートメントを実行できません。詳細な情報については`code`および`message`フィールドを確認してください。

以下に例を示します:

```json
{
    "type": "sql_endpoint",
    "data": {
        "columns": [],
        "rows": [],
        "result": {
            "code": 1146,
            "message": "table not found",
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

### 400

このステータスコードは、パラメータのチェックが失敗したことを示します。

以下に例を示します:

```json
{
    "type": "sql_endpoint",
    "data": {
        "columns": [],
        "rows": [],
        "result": {
            "code": 400,
            "message": "param check failed! {detailed error}",
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

### 401

このステータスコードは、権限が不足して認証に失敗したことを示します。

以下に例を示します:

```json
{
    "type": "sql_endpoint",
    "data": {
        "columns": [],
        "rows": [],
        "result": {
            "code": 401,
            "message": "auth failed",
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

### 404

このステータスコードは、指定されたエンドポイントが見つからず、認証に失敗したことを示します。

以下に例を示します:

```json
{
    "type": "sql_endpoint",
    "data": {
        "columns": [],
        "rows": [],
        "result": {
            "code": 404,
            "message": "endpoint not found",
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

### 405

このステータスコードは、許可されていないメソッドが使用されたことを示します。Data Serviceは`GET`および`POST`のみをサポートしていることに注意してください。

以下に例を示します:

```json
{
    "type": "sql_endpoint",
    "data": {
        "columns": [],
        "rows": [],
        "result": {
            "code": 405,
            "message": "method not allowed",
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

### 408

このステータスコードは、リクエストがエンドポイントのタイムアウト期間を超えたことを示します。エンドポイントのタイムアウトを変更するには、[プロパティを構成](/tidb-cloud/data-service-manage-endpoint.md#configure-properties)してください。

以下に例を示します:

```json
{
    "type": "sql_endpoint",
    "data": {
        "columns": [],
        "rows": [],
        "result": {
            "code": 408,
            "message": "request timeout.",
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

### 429

このステータスコードは、リクエストがAPIキーの使用率制限を超えたことを示します。より多くのクォータをご希望の場合は、[リクエストを送信](https://support.pingcap.com/hc/en-us/requests/new?ticket_form_id=7800003722519)してください。

以下に例を示します:

<SimpleTab>
<div label="SQLエンドポイント">

```json
{
  "type": "",
  "data": {
    "columns": [],
    "rows": [],
    "result": {
      "code": 49900007,
      "message": "The request exceeded the limit of 100 times per apikey per minute. For more quota, please contact us: https://support.pingcap.com/hc/en-us/requests/new?ticket_form_id=7800003722519",
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

</div>

<div label="Chat2Dataエンドポイント">

```json
{
  "type": "chat2data_endpoint",
  "data": {
    "columns": [],
    "rows": [],
    "result": {
      "code": 429,
```json
{
    "メッセージ": "1日あたりのAIリクエストが100回を超えました。さらにクォータをご希望の場合は、お問い合わせください：https://support.pingcap.com/hc/en-us/requests/new?ticket_form_id=7800003722519",
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

</div>
</SimpleTab>

### 500

このステータスコードは、リクエストが内部エラーに遭遇したことを示します。このエラーにはさまざまな原因が考えられます。

一つの可能性としては、認証サーバーに接続できなかったため認証に失敗したことが考えられます。

次のようなレスポンスがあります。

```json
{
    "type": "sql_endpoint",
    "data": {
        "columns": [],
        "rows": [],
        "result": {
            "code": 500,
            "message": "内部エラー！デフォルトのパーミッションヘルパー：RPCエラー：コード=DeadlineExceeded desc =コンテキストの締め切りが過ぎました",
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

これはTiDB Cloudクラスターに接続できない可能性もあります。トラブルシューティングには、`message`を参照する必要があります。

```json
{
    "type": "sql_endpoint",
    "data": {
        "columns": [],
        "rows": [],
        "result": {
            "code": 500,
            "message": "内部エラー！{詳細なエラー}",
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