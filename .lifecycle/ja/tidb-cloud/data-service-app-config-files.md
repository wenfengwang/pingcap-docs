---
title: データアプリ構成ファイル
summary: この文書はTiDB Cloudのデータアプリの構成ファイルについて説明します。

# データアプリ構成ファイル

この文書はTiDB Cloudにおける[データアプリ](/tidb-cloud/tidb-cloud-glossary.md#data-app)の構成ファイルについて説明します。

GitHubにデータアプリを接続しましたら、指定したディレクトリ内にデータアプリの構成ファイルを以下のようにGitHubで見つけることができます。

```
├── <Your Data App directory>
│   ├── data_sources
│   │   └── cluster.json
│   ├── dataapp_config.json
│   ├── http_endpoints
│   │   ├── config.json
│   │   └── sql
│   │       ├── <method>-<endpoint-path1>.sql
│   │       ├── <method>-<endpoint-path2>.sql
│   │       └── <method>-<endpoint-path3>.sql
```

## データソース構成

データアプリのデータソースはリンクされたTiDBクラスタから取得されます。`data_sources/cluster.json`内でデータソース構成を見つけることができます。

```
├── <Your Data App directory>
│   ├── data_sources
│   │   └── cluster.json
```

各データアプリにつき、1つまたは複数のTiDBクラスタにリンクすることができます。

以下は`cluster.json`の構成の例です。この例では、このデータアプリには2つのリンクされたクラスタがあります。

```json
[
  {
    "cluster_id": <Cluster ID1>
  },
  {
    "cluster_id": <Cluster ID2>
  }
]
```

各フィールドの説明は以下の通りです。

| フィールド   | タイプ    | 説明  |
|---------|---------|--------------|
| `cluster_id` | Integer | TiDBクラスタのID。クラスタのURLから取得できます。たとえば、クラスタのURLが `https://tidbcloud.com/console/clusters/1234567891234567890/overview` の場合、クラスタIDは `1234567891234567890` です。 |

## データアプリ構成

データアプリのプロパティにはApp ID、名前、およびタイプが含まれています。これらのプロパティは`dataapp_config.json`ファイルに見つけることができます。

```
├── <Your Data App directory>
│   ├── dataapp_config.json
```

以下は`dataapp_config.json`の構成の例です。

```json
{
  "app_id": "<データアプリID>",
  "app_name": "<データアプリ名>",
  "app_type": "dataapi",
  "app_version": "<データアプリバージョン>",
  "description": "<データアプリの説明>"
}
```

各フィールドの説明は以下の通りです。

| フィールド      | タイプ   | 説明        |
|------------|--------|--------------------|
| `app_id`   | String | データアプリID。`dataapp_config.json`ファイルが他のデータアプリからコピーされ、現在のデータアプリのIDに更新したい場合を除き、このフィールドを変更しないでください。それ以外の場合、この変更によって引き起こされるデプロイは失敗します。 |
| `app_name` | String | データアプリの名前。 |
| `app_type` | String | データアプリのタイプ。`"dataapi"` のみを指定できます。 |
| `app_version` | String | データアプリのバージョン。「<major>.<minor>.<patch>」の形式です。たとえば、`"1.0.0"` です。 |
| `description` | String | データアプリの説明。 |

## HTTP エンドポイント構成

データアプリディレクトリ内では、エンドポイントの構成を`http_endpoints/config.json`およびSQLファイルを`http_endpoints/sql/<method>-<endpoint-name>.sql`で見つけることができます。

```
├── <Your Data App directory>
│   ├── http_endpoints
│   │   ├── config.json
│   │   └── sql
│   │       ├── <method>-<endpoint-path1>.sql
│   │       ├── <method>-<endpoint-path2>.sql
│   │       └── <method>-<endpoint-path3>.sql
```

### エンドポイント構成

データアプリごとに、1つまたは複数のエンドポイントが存在する可能性があります。データアプリのすべてのエンドポイントの構成は`http_endpoints/config.json`で見つけることができます。

以下は`config.json`の構成の例です。この例では、このデータアプリには2つのエンドポイントがあります。

```json
[
  {
    "name": "<エンドポイント名1>",
    "description": "<エンドポイント説明1>",
    "method": "<HTTPメソッド1>",
    "endpoint": "<エンドポイントパス1>",
    "data_source": {
      "cluster_id": <Cluster ID1>
    },
    "params": [],
    "settings": {
      "timeout": <エンドポイントタイムアウト>,
      "row_limit": <最大行数>,
      "enable_pagination": <0 | 1>,
      "cache_enabled": <0 | 1>,
      "cache_ttl": <キャッシュの有効期間>
    },
    "tag": "Default",
    "batch_operation": <0 | 1>,
    "sql_file": "<SQLファイルディレクトリ1>",
    "type": "sql_endpoint",
    "return_type": "json"
  },
  {
    "name": "<エンドポイント名2>",
    "description": "<エンドポイント説明2>",
    "method": "<HTTPメソッド2>",
    "endpoint": "<エンドポイントパス2>",
    "data_source": {
      "cluster_id": <Cluster ID2>
    },
    "params": [
      {
        "name": "<パラメータ名>",
        "type": "<パラメータタイプ>",
        "required": <0 | 1>,
        "default": "<パラメータデフォルト値>",
        "description": "<パラメータ説明>"
      }
    ],
    "settings": {
      "timeout": <エンドポイントタイムアウト>,
      "row_limit": <最大行数>,
      "enable_pagination": <0 | 1>,
      "cache_enabled": <0 | 1>,
      "cache_ttl": <キャッシュの有効期間>
    },
    "tag": "Default",
    "batch_operation": <0 | 1>,
    "sql_file": "<SQLファイルディレクトリ2>",
    "type": "sql_endpoint",
    "return_type": "json"
  }
]
```

各フィールドの説明は以下の通りです。

| フィールド         | タイプ   | 説明 |
|---------------|--------|-------------|
| `name`        | String | エンドポイント名。            |
| `description` | String | (オプション) エンドポイントの説明。          |
| `method`      | String | エンドポイントのHTTPメソッド。データの取得には `GET` を使用し、データの作成または挿入には `POST` を使用し、データの更新または変更には `PUT` を使用し、データの削除には `DELETE` を使用します。 |
| `endpoint`    | String | データアプリ内のエンドポイントの固有のパス。パスには、文字、数字、アンダースコア (`_`)、およびスラッシュ (`/`) のみを使用でき、パスはスラッシュ (`/`) で始まり、文字、数字、またはアンダースコア (`_`) で終了する必要があります。たとえば、`/my_endpoint/get_id`。パスの長さは64文字未満である必要があります。|
| `cluster_id`  | String | エンドポイントのTiDBクラスタのID。クラスタのURLから取得できます。たとえば、クラスタのURLが `https://tidbcloud.com/console/clusters/1234567891234567890/overview` の場合、クラスタIDは `1234567891234567890` です。 |
| `params` | Array | エンドポイントで使用されるパラメータ。パラメータを定義することで、エンドポイントを通じてクエリのパラメータ値を動的に置き換えることができます。`params` で、1つまたは複数のパラメータを定義できます。各パラメータについては、その `name`、`type`、`required`、`default` フィールドを定義する必要があります。エンドポイントがパラメータを必要としない場合は、`"params": []` のように `params` を空にしておくことができます。 |
| `params.name` | String | パラメータの名前。名前には文字、数字、およびアンダースコア (`_`) のみを含めることができ、文字またはアンダースコア (`_`) で始める必要があります。このパラメータ名には `page` と `page_size` を使用しないでください。これらはリクエスト結果のページネーションのために予約されています。 |
| `params.type` | String | パラメータのデータタイプ。サポートされる値は `string`、`number`、`integer`、`boolean` です。`string` タイプのパラメータを使用する場合、引用符 (`'` や `"`) を追加する必要はありません。たとえば、`foo` は `string` タイプの場合に有効であり、 `"foo"` と処理されます。一方で、`"foo"` は `"\"foo\""` と処理されます。 |
| `params.required` | Integer | リクエストでパラメータが必要かどうかを指定します。サポートされる値は `0` (不要) および `1` (必要) です。デフォルト値は `0` です。  |
| `params.default` | String | パラメータのデフォルト値。値が指定したパラメータタイプと一致することを確認してください。それ以外の場合、エンドポイントはエラーを返します。 |
| `params.description` | String | パラメータの説明。 |
| `settings.timeout`     | Integer | エンドポイントのタイムアウト時間 (ミリ秒単位)。デフォルトでは `30000` です。 `1` から `30000` の整数に設定できます。  |
| `settings.row_limit`   | Integer  | エンドポイントが操作または返すことができる最大行数であり、デフォルトでは `1000` です。 `batch_operation` が `0` に設定されているときは、`1` から `2000` までの整数に設定できます。 `batch_operation` が `1` に設定されているときは、`1` から `100` までの整数に設定できます。  |
| `settings.enable_pagination`   | Integer  | リクエストによって返される結果のページネーションを有効にするかどうかを制御します。サポートされる値は `0`（無効）および `1`（有効）です。デフォルト値は `0` です。 |
| `settings.cache_enabled`   | Integer  | `GET` リクエストが指定された time-to-live（TTL）期間内に応答をキャッシュするかどうかを制御します。サポートされる値は `0`（無効）および `1`（有効）です。デフォルト値は `0` です。 |
| `settings.cache_ttl`   | Integer  | `settings.cache_enabled` が `1` に設定されている場合のキャッシュされた応答の生存期間（TTL）を秒単位で指定します。30 から 600 までの整数に設定できます。TTL 期間中、同じ `GET` リクエストを再度行うと、データサービスはデータベースからデータを再取得する代わりにキャッシュされた応答を直接返し、クエリのパフォーマンスを向上させます。 |
| `tag`    | String | エンドポイントのタグ。デフォルト値は `"Default"` です。 |
| `batch_operation`    | Integer | エンドポイントがバッチモードで操作できるかどうかを制御します。サポートされる値は `0`（無効）および `1`（有効）です。これが `1` に設定されている場合は、単一のリクエストで複数の行を操作できます。このオプションを有効にするには、リクエストメソッドを `POST`、`PUT`、または `DELETE` に設定してください。 |
| `sql_file`    | String | エンドポイントの SQL ファイルディレクトリ。例：`"sql/GET-v1.sql"`。 |
| `type`        | String | エンドポイントのタイプ。`"sql_endpoint"` のみ指定できます。          |
| `return_type` | String | エンドポイントのレスポンス形式。`"json"` のみ指定できます。             |

### SQL ファイルの構成

エンドポイントの SQL ファイルは、エンドポイントを介してデータをクエリするための SQL ステートメントを指定します。Data App のエンドポイント SQL ファイルは `http_endpoints/sql/` ディレクトリ内で見つけることができます。各エンドポイントには、対応する SQL ファイルが必要です。

SQL ファイルの名前は `<method>-<endpoint-path>.sql` 形式であり、`<method>` と `<endpoint-path>` は [`http_endpoints/config.json`](#endpoint-configuration) での `method` および `endpoint` 構成と一致する必要があります。

SQL ファイルでは、テーブル結合クエリ、複雑なクエリ、および集計関数などのステートメントを記述できます。以下は SQL ファイルの例です。

```sql
/* 初めてお試しの場合：
SQL ステートメントを入力する前に、"USE {データベース};" を入力してください。
タイプ "--your question" + Enter で TiDB Cloud コンソールで AI 生成の SQL クエリを試すことができます。
"Where id = ${arg}" のようにパラメーターを宣言します。
*/
USE sample_data;
SELECT
  rank,
  company_name,
FROM
  global_fortune_500_2018_2022
WHERE
  country = ${country};
```

SQL ファイルを作成する際は、以下に注意してください：

- SQL ファイルの先頭に、SQL ステートメントでデータベースを指定する必要があります。例: `USE database_name;`。

- エンドポイントのパラメータを定義するには、SQL ステートメントに `${variable-name}` のように変数プレースホルダーとして挿入できます。

    先述の例では、`${country}` がエンドポイントのパラメータとして使用されています。このパラメータを使用すると、エンドポイントの curl コマンドでクエリする国を指定できます。

    > **注意：**
    >
    > - パラメータ名は大文字と小文字が区別されます。
    > - パラメータはテーブル名または列名にすることはできません。
    > - SQL ファイル内のパラメータ名は [`http_endpoints/config.json`](#endpoint-configuration) で構成されたパラメータ名と一致する必要があります。