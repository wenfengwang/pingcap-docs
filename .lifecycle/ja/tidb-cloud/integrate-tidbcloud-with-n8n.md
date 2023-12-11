---
title: n8nでTiDB Cloudを統合する
summary: n8nでのTiDB Cloudノードの使用方法を学ぶ。

# n8nでTiDB Cloudを統合する

[n8n](https://n8n.io/)は拡張可能なワークフロー自動化ツールです。[fair-code](https://faircode.io/)の配布モデルを採用しており、n8nは常に可視なソースコードを持ち、自己ホストが可能であり、カスタム関数、ロジック、およびアプリを追加できます。

このドキュメントでは、自動ワークフローを構築する方法について紹介します：TiDB Serverlessクラスタを作成し、Hacker NewsのRSSを取得してTiDBに保存し、ブリーフィングメールを送信します。

## 前提条件: TiDB Cloud APIキーを取得

1. TiDB Cloudのダッシュボードにアクセスします。
2. <MDSvgIcon name="icon-top-organization" />をクリックし、左下の**Organization Settings**をクリックします。
3. **API Keys**タブをクリックします。
4. **Create API Key**ボタンをクリックして新しいAPIキーを作成します。
5. 作成したAPIキーをn8nで後で使用するために保存します。

詳細は、[TiDB Cloud API Overview](/tidb-cloud/api-overview.md)を参照してください。

## ステップ1：n8nのインストール

自己ホスティングのn8nをインストールする方法は2つあります。お好きな方法を選択してください。

<SimpleTab>
<div label="npm">

1. ワークスペースに[node.js](https://nodejs.org/en/download/)をインストールします。
2. 以下のコマンドを使用してn8nをダウンロードして起動します。

    ```shell
    npx n8n
    ```

</div>
<div label="Docker">

1. ワークスペースに[Docker](https://www.docker.com/products/docker-desktop)をインストールします。
2. 以下のコマンドを使用してn8nをダウンロードして起動します。

    ```shell
    docker run -it --rm --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
    ```

</div>
</SimpleTab>

n8nを起動した後、[localhost:5678](http://localhost:5678)にアクセスしてn8nを試すことができます。

## ステップ2：n8nでTiDB Cloudノードをインストールする

TiDB Cloudノードはnpmリポジトリで`n8n-nodes-tidb-cloud`という名前です。このノードを手動でインストールしてn8nでTiDB Cloudを制御する必要があります。

1. [localhost:5678](http://localhost:5678)ページで、自己ホストn8nの所有者アカウントを作成します。
2. **Settings** > **Community nodes**に移動します。
3. **Install a community node**をクリックします。
4. **npm Package Name**フィールドに`n8n-nodes-tidb-cloud`を入力します。
5. **Install**をクリックします。

その後、**Workflow** > 検索バーで**TiDB Cloud**ノードを検索し、ワークスペースにドラッグして使用できます。

## ステップ3：ワークフローの構築

このステップでは、**Execute**ボタンをクリックするとTiDBにデータを挿入する新しいワークフローを作成します。

この例のワークフローでは、以下のノードが使用されます：

- [スケジュールトリガー](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.scheduletrigger/)
- [RSS Read](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.rssfeedread/)
- [Code](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.code/)
- [Gmail](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.gmail/)
- [TiDB Cloud node](https://www.npmjs.com/package/n8n-nodes-tidb-cloud)

最終ワークフローは以下の画像のようになります。

![img](/media/tidb-cloud/integration-n8n-workflow-rss.jpg)

### (オプション) TiDB Serverlessクラスタを作成する

TiDB Serverlessクラスタがない場合は、このノードを使用して作成することができます。それ以外の場合は、この操作をスキップしてください。

1. **Workflows**パネルに移動し、**Add workflow**をクリックします。
2. 新しいワークフローのワークスペースで、右上の**+**をクリックして**All**フィールドを選択します。
3. `TiDB Cloud`を検索してワークスペースにドラッグします。
4. TiDB Cloudノードの資格情報、つまりTiDB Cloud APIキーを入力します。
5. **Project**リストでプロジェクトを選択します。
6. **Operation**リストで`Create Serverless Cluster`を選択します。
7. **Cluster Name**ボックスにクラスタ名を入力します。
8. **Region**リストからリージョンを選択します。
9. **Password**ボックスにTiDBクラスタにログインするためのパスワードを入力します。
10. ノードを実行するために**Execute Node**をクリックします。

> **注意:**
>
> 新しいTiDB Serverlessクラスタを作成するには数秒かかります。

### ワークフローの作成

#### マニュアルトリガーをワークフローの起動コンポーネントとして使用する

1. ワークフローがまだない場合は、**Workflows**パネルに移動し、**Start from scratch**をクリックします。それ以外の場合は、この手順をスキップしてください。
2. 右上の**+**をクリックし、`schedule trigger`を検索します。
3. マニュアルトリガーノードをワークスペースにドラッグし、ノードをダブルクリックします。**Parameters**ダイアログが表示されます。
4. ルールを以下のように構成します：

    - **Trigger Interval**: `Days`
    - **Days Between Triggers**: `1`
    - **Trigger at Hour**: `8am`
    - **Trigger at Minute**: `0`

このトリガーは毎朝8時にワークフローを実行します。

#### データを挿入するために使用されるテーブルを作成

1. マニュアルトリガーノードの右側に**+**をクリックします。
2. `TiDB Cloud`を検索して追加します。
3. **Parameters**ダイアログに、TiDB Cloudノードの資格情報、つまりTiDB Cloud APIキーを入力します。
4. **Project**リストでプロジェクトを選択します。
5. **Operation**リストで`Execute SQL`を選択します。
6. クラスタを選択します。新しいクラスタが一覧に表示されていない場合は、クラスタの作成が完了するまで数分待つ必要があります。
7. **User**リストでユーザを選択します。TiDB Cloudは常にデフォルトユーザを作成するため、手動で作成する必要はありません。
8. **Database**ボックスに`test`を入力します。
9. データベースのパスワードを入力します。
10. **SQL**ボックスに次のSQLを入力します：

    ```sql
    CREATE TABLE IF NOT EXISTS hacker_news_briefing (creator VARCHAR (200), title TEXT,  link VARCHAR(200), pubdate VARCHAR(200), comments VARCHAR(200), content TEXT, guid VARCHAR (200), isodate VARCHAR(200));
    ```

11. テーブルを作成するために**Execute node**をクリックします。

#### Hacker NewsのRSSを取得

1. TiDB Cloudノードの右側に**+**をクリックします。
2. `RSS Read`を検索して追加します。
3. **URL**ボックスに`https://hnrss.org/frontpage`を入力します。

#### TiDBにデータを挿入

1. RSS Readノードの右側に**+**をクリックします。
2. `TiDB Cloud`を検索して追加します。
3. 前のTiDB Cloudノードで入力した資格情報を選択します。
4. **Project**リストでプロジェクトを選択します。
5. **Operation**リストで`Insert`を選択します。
6. **Cluster**、**User**、**Database**、**Password**ボックスにそれぞれ対応する値を入力します。
7. **Table**ボックスに`hacker_news_briefing`と入力します。
8. **Columns**ボックスに`creator, title, link, pubdate, comments, content, guid, isodate`と入力します。

#### メッセージを構築

1. RSS Feed Readノードの右側に**+**をクリックします。
2. `code`を検索して追加します。
3. `Run Once for All Items`モードを選択します。
4. **JavaScript**ボックスに、以下のコードをコピーして貼り付けます。

    ```javascript
    let message = "";

    // 入力アイテムをループする
    for (item of items) {
      message += `
          <h3>${item.json.title}</h3>
          <br>
          ${item.json.content}
          <br>
          `
    }

    let response =
        `
          <!DOCTYPE html>
          <html>
          <head>
          <title>Hacker News Briefing</title>
        </head>
        <body>
            ${message}
        </body>
        </html>
        `
    // メッセージを返す
    return [{json: {response}}];
    ```

#### Gmailでメッセージを送信

1. codeノードの右側に**+**をクリックします。
2. `gmail`を検索して追加します。
3. Gmailノードの資格情報を入力します。詳細な手順については、[n8n documentation](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/)を参照してください。
4. **Resource**リストで`Message`を選択します。
5. **Operation**リストで`Send`を選択します。
6. **宛先**ボックスにあなたのメールアドレスを入力してください。
7. **件名**ボックスに `ハッカーニュースの概要` を入力してください。
8. **メールのタイプ**ボックスで `HTML` を選択してください。
9. **メッセージ**ボックスで `Expression` をクリックし、`{{ $json["response"] }}` を入力してください。

    > **注:**
    >
    > **メッセージ**ボックスにカーソルを合わせ、**Expression**パターンを選択する必要があります。

## ステップ 4: ワークフローを実行する

ワークフローを構築した後、**ワークフローを実行**をクリックしてテスト実行できます。

ワークフローが期待どおりに実行されると、ハッカーニュースの概要メールを受け取ることができます。これらのニュースコンテンツはTiDB Serverlessクラスターにログが記録されるため、失う心配はありません。

以上で、このワークフローを**ワークフロー**パネルでアクティブにすることができます。このワークフローによって、毎日ハッカーニュースのトップページの記事を取得することができます。

## TiDB Cloud ノードコア

### サポートされている操作

TiDB Cloudノードは、[通常のノード](https://docs.n8n.io/workflows/nodes/#regular-nodes)として機能し、以下の5つの操作のみがサポートされています:

- **Serverlessクラスターを作成**: TiDB Serverlessクラスターを作成します。
- **SQLを実行**: TiDBでSQLステートメントを実行します。
- **削除**: TiDBで行を削除します。
- **挿入**: TiDBに行を挿入します。
- **更新**: TiDBで行を更新します。

### フィールド

異なる操作を行うには、異なる必須フィールドに入力する必要があります。次に、対応する操作のためのフィールドの説明を示します。

<SimpleTab>
<div label="Serverlessクラスターを作成">

- **TiDB Cloud APIの認証情報**: TiDB Cloud APIキーのみがサポートされています。APIキーの作成方法については、[TiDB Cloud APIキーの取得](#prerequisites-get-tidb-cloud-api-key)を参照してください。
- **プロジェクト**: TiDB Cloudプロジェクト名。
- **Operation**: このノードの操作。サポートされているすべての操作については、[サポートされている操作](#supported-operations)を参照してください。
- **クラスター**: TiDB Cloudクラスター名。新しいクラスターの名前を入力してください。
- **地域**: 地域名。クラスターを展開する地域を選択してください。通常、アプリケーションの展開に最も近い地域を選択してください。
- **パスワード**: ルートパスワード。新しいクラスターのパスワードを設定してください。

</div>
<div label="SQLを実行">

- **TiDB Cloud APIの認証情報**: TiDB Cloud APIキーのみがサポートされています。APIキーの作成方法については、[TiDB Cloud APIキーの取得](#prerequisites-get-tidb-cloud-api-key)を参照してください。
- **プロジェクト**: TiDB Cloudプロジェクト名。
- **Operation**: このノードの操作。サポートされているすべての操作については、[サポートされている操作](#supported-operations)を参照してください。
- **クラスター**: TiDB Cloudクラスター名。既存のクラスターを選択してください。
- **パスワード**: TiDB Cloudクラスターのパスワード。
- **ユーザー**: TiDB Cloudクラスターのユーザー名。
- **データベース**: データベース名。
- **SQL**: 実行するSQLステートメント。

</div>
<div label="Delete">

- **TiDB Cloud APIの認証情報**: TiDB Cloud APIキーのみがサポートされています。APIキーの作成方法については、[TiDB Cloud APIキーの取得](#prerequisites-get-tidb-cloud-api-key)を参照してください。
- **プロジェクト**: TiDB Cloudプロジェクト名。
- **Operation**: このノードの操作。サポートされているすべての操作については、[サポートされている操作](#supported-operations)を参照してください。
- **クラスター**: TiDB Cloudクラスター名。既存のクラスターを選択してください。
- **パスワード**: TiDB Cloudクラスターのパスワード。
- **ユーザー**: TiDB Cloudクラスターのユーザー名。
- **データベース**: データベース名。
- **テーブル**: テーブル名。`From list`モードを使用して選択するか、`Name`モードを使用してテーブル名を手動で入力することができます。
- **削除キー**: データベースの行を削除する際にどのプロパティ名を使用するかを決定する項目の名前。項目とは、1つのノードから別のノードに送信されるデータのことです。n8nにおける項目の詳細については、[n8nのドキュメント](https://docs.n8n.io/workflows/items/)を参照してください。

</div>
<div label="Insert">

- **TiDB Cloud APIの認証情報**: TiDB Cloud APIキーのみがサポートされています。APIキーの作成方法については、[TiDB Cloud APIキーの取得](#prerequisites-get-tidb-cloud-api-key)を参照してください。
- **Project**: TiDB Cloudプロジェクト名。
- **Operation**: このノードの操作。サポートされているすべての操作については、[サポートされている操作](#supported-operations)を参照してください。
- **クラスター**: TiDB Cloudクラスター名。既存のクラスターを選択してください。
- **Password**: TiDB Cloudクラスターのパスワード。
- **User**: TiDB Cloudクラスターのユーザー名。
- **データベース**: データベース名。
- **テーブル**: テーブル名。`From list`モードを使用して選択するか、`Name`モードを使用してテーブル名を手動で入力することができます。
- **Columns**: 新しい行の列として使用される入力アイテムのプロパティのコンマ区切りリスト。項目とは、1つのノードから別のノードに送信されるデータのことです。n8nにおける項目の詳細については、[n8nのドキュメント](https://docs.n8n.io/workflows/items/)を参照してください。

</div>
<div label="Update">

- **TiDB Cloud APIの認証情報**: TiDB Cloud APIキーのみがサポートされています。APIキーの作成方法については、[TiDB Cloud APIキーの取得](#prerequisites-get-tidb-cloud-api-key)を参照してください。
- **プロジェクト**: TiDB Cloudプロジェクト名。
- **Operation**: このノードの操作。サポートされているすべての操作については、[サポートされている操作](#supported-operations)を参照してください。
- **クラスター**: TiDB Cloudクラスター名。既存のクラスターを選択してください。
- **パスワード**: TiDB Cloudクラスターのパスワード。
- **ユーザー**: TiDB Cloudクラスターのユーザー名。
- **データベース**: データベース名。
- **テーブル**: テーブル名。`From list`モードを使用して選択するか、`Name`モードを使用してテーブル名を手動で入力することができます。
- **更新キー**: データベースの行を更新する際にどのプロパティ名を使用するかを決定する項目の名前。項目とは、1つのノードから別のノードに送信されるデータのことです。n8nにおける項目の詳細については、[n8nのドキュメント](https://docs.n8n.io/workflows/items/)を参照してください。
- **Columns**: 更新される行の列として使用される入力アイテムのプロパティのコンマ区切りリスト。

</div>
</SimpleTab>

### 制限事項

- **Execute SQL**操作では通常、1つのSQLステートメントのみが許可されます。1つの操作で複数のステートメントを実行したい場合は、手動で[`tidb_multi_statement_mode`](https://docs.pingcap.com/tidbcloud/system-variables#tidb_multi_statement_mode-new-in-v4011)を有効にする必要があります。
- **Delete**および**Update**操作では、1つのフィールドをキーとして指定する必要があります。たとえば、`Delete Key` が `id` に設定されており、これは `DELETE FROM table WHERE id = ${item.id}` を実行するのと等価です。現在、**Delete**および**Update**操作では1つのキーのみを指定することができます。
- **Insert**および**Update**操作では、**Columns**フィールドにコンマ区切りのリストを指定する必要があります。また、フィールド名は入力アイテムのプロパティと同じである必要があります。