---
title: データサービスを始めよう
summary: TiDB Cloudデータサービスを使用してHTTPSリクエストでデータにアクセスする方法を学びます。

# データサービスを始めよう

データサービス（ベータ版）を使用すると、カスタムAPIエンドポイントを使用してHTTPSリクエストでTiDB Cloudデータにアクセスでき、HTTPSに対応した任意のアプリケーションやサービスとシームレスに統合できます。

> **ヒント：**
>
> TiDB CloudではTiDBクラスターのためのChat2Query APIを提供しています。有効になった後、TiDB Cloudは自動的にシステムデータアプリケーション**Chat2Query**とChat2Dataエンドポイントを作成します。これらのエンドポイントを呼び出すことで、AIによってSQLステートメントを生成し実行することができます。
>
> 詳細については、[Chat2Query APIを使用する](/tidb-cloud/use-chat2query-api.md)を参照してください。

このドキュメントでは、TiDB Cloudデータサービス（ベータ版）の使用を始めるために、データアプリケーションの作成、開発、テスト、展開、およびエンドポイントの呼び出しについて紹介します。

## 開始する前に

データアプリケーションを作成する前に、[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターを作成していることを確認してください。まだ作成していない場合は、[TiDB Serverlessクラスターを作成する](/tidb-cloud/create-tidb-cluster-serverless.md)の手順に従って作成してください。

## ステップ1. データアプリケーションを作成する

データアプリケーションは、特定のアプリケーションのデータにアクセスするためのエンドポイントのコレクションです。データアプリケーションを作成するには、次の手順を実行してください。

1. [TiDB Cloudコンソール](https://tidbcloud.com)で、左側のナビゲーションペインで<MDSvgIcon name="icon-left-data-service" /> **データサービス**をクリックします。

2. **データサービス**ページで、**データアプリケーションを作成**をクリックします。

3. **データアプリケーションを作成**ダイアログボックスで、名前と説明を入力し、データアプリケーションからアクセスしたいクラスターを選択します。

4. （オプション）データアプリケーションのエンドポイントを自動的に選択したGitHubリポジトリとブランチに展開する場合は、**GitHubに接続**を有効にして、次の手順を実行します。

    1. **GitHubにインストール**をクリックし、画面の指示に従ってターゲットリポジトリに**TiDB Cloudデータサービス**をアプリケーションとしてインストールします。
    2. **Authorize**をクリックして、GitHub上のアプリケーションへのアクセスを承認します。
    3. データアプリケーションの構成ファイルを保存したいターゲットリポジトリ、ブランチ、およびディレクトリを指定します。

    > **ノート：**
    >
    > - ディレクトリはスラッシュ（`/`）で始まる必要があります。例えば、`/mydata`です。指定したディレクトリがターゲットリポジトリとブランチに存在しない場合、自動的に作成されます。
    > - リポジトリ、ブランチ、およびディレクトリの組み合わせは、データアプリケーション間でユニークである必要があります。指定したパスが他のデータアプリケーションで既に使用されている場合は、新しいパスを指定する必要があります。そうでなければ、TiDB Cloudコンソールで構成されたエンドポイントは、指定したパスのファイルを上書きします。

5. **データアプリケーションを作成**をクリックします。[**データサービス**](https://tidbcloud.com/console/data-service)の詳細ページが表示されます。

6. データアプリケーションをGitHubに接続するように構成した場合、指定したGitHubディレクトリを確認してください。そこに[tidb-cloud-data-service](/tidb-cloud/data-service-app-config-files.md)によってデータアプリケーションの構成ファイルがコミットされていることがわかります。これは、データアプリケーションがGitHubに正常に接続されていることを意味します。

    新しいデータアプリケーションには、**Auto Sync & Deployment**と**Review Draft**がデフォルトで有効になっているため、TiDB CloudコンソールとGitHubの間でデータアプリケーションの変更を簡単に同期し、展開前に変更を確認できます。GitHubインテグレーションの詳細については、[GitHubを使用してデータアプリケーションの変更を自動的に展開する](/tidb-cloud/data-service-manage-github-connection.md)を参照してください。

## ステップ2. エンドポイントを開発する

エンドポイントは、SQLステートメントを実行するためにカスタマイズできるWeb APIです。

データアプリケーションを作成した後、デフォルトの`untitled endpoint`が自動的に作成されます。このデフォルトのエンドポイントを使用してTiDB Cloudクラスターにアクセスできます。

新しいエンドポイントを作成したい場合は、新たに作成されたデータアプリケーションを見つけ、アプリケーション名の右側にある**+** **Create Endpoint**をクリックします。

### プロパティを設定する

右側のペインで、**プロパティ**タブをクリックし、エンドポイントのプロパティ（例：）を設定します。

- **Path**: ユーザーがエンドポイントにアクセスするために使用するパス。要求メソッドとパスの組み合わせはデータアプリケーション内で一意である必要があります。

- **エンドポイントURL**: （読み取り専用）URLは、対応するクラスターが存在するリージョン、データアプリケーションのサービスURL、およびエンドポイントのパスに基づいて自動的に生成されます。例えば、エンドポイントのパスが`/my_endpoint/get_id`の場合、エンドポイントURLは`https://<region>.data.tidbcloud.com/api/v1beta/app/<App ID>/endpoint/my_endpoint/get_id`となります。

- **リクエストメソッド**: エンドポイントのHTTPメソッド。`GET`を使用してデータを取得し、`POST`を使用してデータを作成または挿入し、`PUT`を使用してデータを更新または修正し、`DELETE`を使用してデータを削除できます。

その他のエンドポイントプロパティについての詳細は、[プロパティを設定する](/tidb-cloud/data-service-manage-endpoint.md#configure-properties)を参照してください。

### SQLステートメントを作成する

SQLエディターでエンドポイントのためにSQLステートメントをカスタマイズできます。

1. クラスターを選択します。

    > **ノート：**
    >
    > データアプリケーションにリンクされているクラスターのみが、ドロップダウンリストに表示されます。リンクされたクラスターを管理するには、[リンクされたクラスターを管理する](/tidb-cloud/data-service-manage-data-app.md#manage-linked-data-sources)を参照してください。

    SQLエディターの上部で、ドロップダウンリストからSQLステートメントを実行したいクラスターを選択します。その後、右側のペインの**Schema**タブでこのクラスターのすべてのデータベースを表示できます。

2. SQLステートメントを書きます。

    データをクエリしたり変更する前に、SQLステートメントでまずデータベースを指定する必要があります。例えば、`USE database_name;`のようになります。

    SQLエディターでは、テーブル結合クエリ、複雑なクエリ、集計関数などのステートメントを書くことができます。また、単純に`--`に続けて指示を記述することで、AIによって自動的にSQLステートメントを生成することもできます。

    パラメータを定義するには、SQLステートメント内で`${ID}`のような変数プレースホルダーとして挿入します。例えば、`SELECT * FROM table_name WHERE id = ${ID}`となります。その後、右側のペインの**Params**タブをクリックしてパラメータの定義やテスト値を変更することができます。

    > **ノート：**
    >
    > - パラメータ名は大文字と小文字を区別します。
    > - パラメータはテーブル名や列名として使用できません。

    - **Definition**セクションでは、クライアントがエンドポイントを呼び出す際にパラメータが必要かどうか、データ型（`STRING`、`NUMBER`、`INTEGER`、または`BOOLEAN`）、パラメータのデフォルト値を指定できます。`STRING`型のパラメータを使用する場合、引用符（`'`または`"`）を追加する必要はありません。例えば、`foo`は`STRING`型の値として有効であり、`"foo"`として処理され、一方`"foo"`は`"\"foo\""`として処理されます。
    - **Test Values**セクションでは、パラメータのテスト値を設定できます。これらのテスト値はSQLステートメントを実行したりエンドポイントをテストする際に使用されます。テスト値を設定しない場合は、デフォルト値が使用されます。
    - 詳細については、[パラメータを設定する](/tidb-cloud/data-service-manage-endpoint.md#configure-parameters)を参照してください。

3. SQLステートメントを実行します。

    SQLステートメントにパラメータを挿入した場合は、右側のペインの**Params**タブでパラメータのテスト値またはデフォルト値を設定してください。そうしないと、エラーが返されます。

    SQLステートメントを実行するには、カーソルを使ってSQLの行を選択し、**Run** > **Run at cursor**をクリックします。

    SQLエディターですべてのSQLステートメントを実行するには、**Run**をクリックします。この場合、最後のSQLの結果のみが返されます。

    ステートメントを実行した後、ページの下部にある**Result**タブでクエリ結果をすぐに確認できます。

## ステップ3. エンドポイントをテストする（オプション）

エンドポイントを設定した後、展開する前にエンドポイントをテストして正常に動作するか確認できます。

エンドポイントをテストするには、ページの右上にある**Test**をクリックするか、**F5**を押してください。

その後、ページの下部にある**HTTP Response**タブで応答を確認できます。詳細については、[エンドポイントの応答](/tidb-cloud/data-service-manage-endpoint.md#response)を参照してください。

## ステップ4. エンドポイントを展開する

エンドポイントを展開するには、次の手順を実行してください。

1. エンドポイントの詳細ページで、右上にある**展開**をクリックします。

2. 展開を確認するために**展開**をクリックします。エンドポイントが正常に展開された場合は、**エンドポイントが展開されました**のプロンプトが表示されます。

    エンドポイントの詳細ページの右側のペインで、**展開**タブをクリックして展開履歴を表示できます。

## ステップ5. エンドポイントを呼び出す

データアプリケーションのAPIキーを取得する前に、HTTPSリクエストを送信することでエンドポイントを呼び出すことができます。

### 1. APIキーを作成する

1. [**データサービス**](https://tidbcloud.com/console/data-service) ページの左ペインで、データアプリの名前をクリックして、その詳細を表示します。
3. **認証** 領域で、**APIキーの作成** をクリックします。
3. **APIキーの作成** ダイアログボックスで、説明を入力し、APIキーのためのロールを選択します。

    ロールは、データアプリにリンクされたクラスタに対してデータの読み取りや書き込みを制御するために使用されます。`ReadOnly` または `ReadAndWrite` のロールを選択できます：

    - `ReadOnly`：APIキーが `SELECT`、`SHOW`、`USE`、`DESC`、`EXPLAIN` 文などのデータの読み取りを許可します。
    - `ReadAndWrite`：APIキーがデータの読み書きを許可します。DML や DDL 文など、すべてのSQL文の実行にこのAPIキーを使用できます。

4. **次へ** をクリックします。公開キーとプライベートキーが表示されます。

    安全な場所にプライベートキーをコピーして保存してください。このページを離れると、完全なプライベートキーを再取得することはできなくなります。

5. **完了** をクリックします。

### 2. コード例を取得する

TiDB Cloudは、エンドポイントの呼び出しをサポートするためのコード例を生成します。コード例を取得するには、次の手順を実行してください：

1. [**データサービス**](https://tidbcloud.com/console/data-service) ページの左ペインで、エンドポイントの名前をクリックして、右上隅の**...** > **コード例** をクリックします。**コード例** ダイアログボックスが表示されます。

2. ダイアログボックスで、エンドポイントを呼び出すために使用するクラスタとデータベースを選択して、コード例をコピーします。

    curlコード例の例は次のとおりです：

    <SimpleTab>
    <div label="テスト環境">

    エンドポイントのドラフトバージョンを呼び出すには、`endpoint-type: draft` ヘッダを追加する必要があります：

    ```bash
    curl --digest --user '<Public Key>:<Private Key>' \
      --request GET 'https://<region>.data.tidbcloud.com/api/v1beta/app/<App ID>/endpoint/<Endpoint Path>' \
      --header 'endpoint-type: draft'
    ```

    </div>

    <div label="オンライン環境">

    オンライン環境でコード例を確認する前に、エンドポイントを展開する必要があります。

    現在のオンラインバージョンのエンドポイントを呼び出すには、次のコマンドを使用してください：

    ```bash
    curl --digest --user '<Public Key>:<Private Key>' \
      --request GET 'https://<region>.data.tidbcloud.com/api/v1beta/app/<App ID>/endpoint/<Endpoint Path>'
    ```

    </div>
    </SimpleTab>

    > **注意:**
    >
    > - リージョナルドメイン `<region>.data.tidbcloud.com` をリクエストすることで、TiDBクラスタが配置されたリージョン内のエンドポイントに直接アクセスできます。
    > - またはリージョンを指定せずにグローバルドメイン `data.tidbcloud.com` をリクエストすることもできます。この場合、TiDB Cloudはリクエストを対象のリージョンに内部的にリダイレクトしますが、これには追加のレイテンシが発生する可能性があります。この方法を選択する場合は、エンドポイントの呼び出し時にcurlコマンドに `--location-trusted` オプションを追加することを確認してください。

### 3. コード例を使用する

コード例をアプリケーションに貼り付けて実行します。その後、エンドポイントの応答を取得できます。

- `<Public Key>` と `<Private Key>` のプレースホルダをAPIキーで置き換える必要があります。
- エンドポイントにパラメータが含まれている場合は、エンドポイントを呼び出す際にパラメータの値を指定してください。

エンドポイントを呼び出した後、JSON形式で応答が表示されます。次に示すのはその例です：

```json
{
  "type": "sql_endpoint",
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
    }
  }
}
```

応答の詳細については、[エンドポイントの応答](/tidb-cloud/data-service-manage-endpoint.md#response)を参照してください。

## 詳細を学ぶ

- [データサービスの概要](/tidb-cloud/data-service-overview.md)
- [Chat2Query APIの始め方](/tidb-cloud/use-chat2query-api.md)
- [データアプリの管理](/tidb-cloud/data-service-manage-data-app.md)
- [エンドポイントの管理](/tidb-cloud/data-service-manage-endpoint.md)