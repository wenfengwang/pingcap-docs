---
title: ZapierでTiDB Cloudを統合する
summary: Zapierを使用してTiDB Cloudを5000以上のアプリと統合する方法を学びます。

# ZapierでTiDB Cloudを統合する

[Zapier](https://zapier.com)は、数千のアプリやサービスを含むワークフローを簡単に作成できるノーコードの自動化ツールです。

[Zapier上のTiDB Cloudアプリ](https://zapier.com/apps/tidb-cloud/integrations)を使用すると、次のことができます。

- ローカルにビルドする必要がないMySQL互換のHTAPデータベースであるTiDBを使用します。
- TiDB Cloudを簡単に管理します。
- TiDB Cloudを5000以上のアプリに接続し、ワークフローを自動化します。

このガイドでは、Zapier上のTiDB Cloudアプリの概要とその使用例について説明します。

## テンプレートを使用したクイックスタート

[Zapテンプレート](https://platform.zapier.com/partners/zap-templates)は、公開されているZapier統合用のアプリやコアフィールドが事前に選択された統合済みの準備ができたZapsです。

このセクションでは、**Add new Github global events to TiDB rows**テンプレートを使用してワークフローを作成する例として説明します。このワークフローでは、GitHubアカウントから新しいグローバルイベント（[GitHubイベント](https://docs.github.com/en/developers/webhooks-and-events/events/github-event-types)があなたに送信または送信されるたびに）が作成されるたびに、ZapierがTiDB Cloudクラスターに新しい行を追加します。

### 前提条件

始める前に、次のものが必要です。

- [Zapierアカウント](https://zapier.com/app/login)。
- [GitHubアカウント](https://github.com/login)。
- [TiDB Cloudアカウント](https://tidbcloud.com/signup)およびTiDB Cloud上のTiDB Serverlessクラスター。詳細については、[TiDB Cloudクイックスタート](https://docs.pingcap.com/tidbcloud/tidb-cloud-quickstart#step-1-create-a-tidb-cluster)を参照してください。

### ステップ1: テンプレートを取得する

[Zapier上のTiDB Cloudアプリ](https://zapier.com/apps/tidb-cloud/integrations)に移動します。**Add new Github global events to TiDB rows**テンプレートを選択して **Try it** をクリックします。その後、エディターページに移動します。

### ステップ2: トリガーを設定する

エディターページでは、トリガーとアクションが表示されます。トリガーを設定するには以下の手順を実行します。

1. アプリとイベントを選択

    テンプレートはデフォルトでアプリとイベントを設定しているため、ここでは何もする必要はありません。**Continue** をクリックします。

2. アカウントを選択

    TiDB Cloudに接続したいGitHubアカウントを選択します。新しいアカウントを接続するか、既存のアカウントを選択できます。設定が完了したら **Continue** をクリックします。

3. トリガーを設定

    テンプレートはデフォルトでトリガーを設定しています。**Continue** をクリックします。

4. トリガーをテスト

    **Test trigger** をクリックします。トリガーが正常に設定された場合、GitHubアカウントから新しいグローバルイベントのデータが表示されます。**Continue** をクリックします。

### ステップ3: `Find Table in TiDB Cloud` アクションを設定する

1. アプリとイベントを選択

    テンプレートでデフォルト値 `Find Table` が設定されているため、**Continue** をクリックします。

2. アカウントを選択

    1. **Sign in** ボタンをクリックし、新しいログインページにリダイレクトされます。
    2. ログインページで、公開鍵とプライベートキーを入力します。TiDB Cloud APIキーを取得するには、[TiDB Cloud APIドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta#section/Authentication/API-Key-Management)の手順に従ってください。
    3. **Continue** をクリックします。

    ![Account](/media/tidb-cloud/zapier/zapier-tidbcloud-account.png)

3. アクションを設定

    このステップでは、TiDB Cloudクラスターにイベントデータを保存するためのテーブルを指定する必要があります。テーブルがまだ存在しない場合は、このステップで作成できます。

    1. ドロップダウンリストからプロジェクト名とクラスター名を選択します。クラスターの接続情報が自動的に表示されます。

        ![Set up project name and cluster name](/media/tidb-cloud/zapier/zapier-set-up-tidbcloud-project-and-cluster.png)

    2. パスワードを入力します。

    3. ドロップダウンリストからデータベースを選択します。

        ![Set up database name](/media/tidb-cloud/zapier/zapier-set-up-tidbcloud-databse.png)

        Zapierは入力したパスワードを使用してTiDB Cloudからデータベースをクエリします。クラスターにデータベースが見つからない場合は、パスワードを再度入力してページをリフレッシュします。

    4. **The table you want to search** ボックスに `github_global_event` を入力します。テーブルが存在しない場合は、テンプレートが次のDDLを使用してテーブルを作成します。**Continue** をクリックします。

        ![The create table DDL](/media/tidb-cloud/zapier/zapier-tidbcloud-create-table-ddl.png)

4. アクションをテスト

    **Test action** をクリックすると、Zapierがテーブルを作成します。テストをスキップしても構いません。このワークフローが最初に実行されるとテーブルが作成されます。

### ステップ4: `Create Row in TiDB Cloud` アクションを設定する

1. アプリとイベントを選択

    テンプレートでデフォルト値が設定されているため、**Continue** をクリックします。

2. アカウントを選択

    前のステップで設定した `Find Table in TiDB Cloud` アクションで選択したアカウントを選択します。**Continue** をクリックします。

    ![Choose account](/media/tidb-cloud/zapier/zapier-tidbcloud-choose-account.png)

3. アクションを設定

    1. **Project Name**、**Cluster Name**、**TiDB Password**、**Database Name**などを前のステップと同様に入力します。

    2. **Table Name** では、ドロップダウンリストから **github_global_event** テーブルを選択します。テーブルの列が表示されます。

        ![Table columns](/media/tidb-cloud/zapier/zapier-set-up-tidbcloud-columns.png)

    3. **Columns**ボックスでは、トリガーから対応するデータを選択します。すべての列を入力し、**Continue** をクリックします。

        ![Fill in Columns](/media/tidb-cloud/zapier/zapier-fill-in-tidbcloud-triggers-data.png)

4. アクションをテスト

    **Test action** をクリックして、テーブルに新しい行を作成します。TiDB Cloudクラスターを確認すると、データが正常に書き込まれていることがわかります。

   ```sql
   mysql> SELECT * FROM test.github_global_event;
   +-------------+-------------+------------+-----------------+----------------------------------------------+--------+---------------------+
   | id          | type        | actor      | repo_name       | repo_url                                     | public | created_at          |
   +-------------+-------------+------------+-----------------+----------------------------------------------+--------+---------------------+
   | 25324462424 | CreateEvent | shiyuhang0 | shiyuhang0/docs | https://api.github.com/repos/shiyuhang0/docs | True   | 2022-11-18 08:03:14 |
   +-------------+-------------+------------+-----------------+----------------------------------------------+--------+---------------------+
   1 row in set (0.17 sec)
   ```

### ステップ5: あなたのzapを公開する

**Publish** をクリックしてあなたのzapを公開します。[ホームページ](https://zapier.com/app/zaps)でzapが実行されているのを確認できます。

![Publish the zap](/media/tidb-cloud/zapier/zapier-tidbcloud-publish.png)

これで、このzapは自動的にあなたのGitHubアカウントからすべてのグローバルイベントをTiDB Cloudに記録します。

## トリガー＆アクション

[Zapier](https://zapier.com/how-it-works)のキーコンセプトであるトリガーやアクションを組み合わせることで、さまざまな自動化ワークフローを作成できます。

このセクションでは、Zapier上のTiDB Cloudアプリで提供されるトリガーとアクションについて紹介します。

### トリガー

TiDB Cloudアプリでサポートされているトリガーは次の表の通りです。

| トリガー                | 説明                                                                 |
| ---------------------- |-----------------------------------------------------------------------------|
| New Cluster            | 新しいクラスターが作成されたときにトリガーします。                                     |
| New Table              | 新しいテーブルが作成されたときにトリガーします。                                       |
| New Row                | 新しい行が作成されたときにトリガーします。最新の10000の新しい行のみを取得します。 |
| New Row (Custom Query) | 提供されたカスタムクエリから新しい行が返されたときにトリガーします。   |

### アクション

TiDB Cloudアプリでサポートされているアクションは次の表の通りです。一部のアクションには追加のリソースが必要であり、アクションを使用する前に対応するリソースを準備する必要があります。

| アクション | 説明 | リソース |
|---|---|---|
| Find Cluster | 既存のTiDB ServerlessまたはTiDB Dedicatedクラスターを検索します。 | なし |
| Create Cluster | 新しいクラスターを作成します。TiDB Serverlessクラスターの作成のみをサポートしています。 | なし |
| Find Database | 既存のデータベースを検索します。 | TiDB Serverlessクラスター |
| Create Database | 新しいデータベースを作成します。 | TiDB Serverlessクラスター |
| テーブルの検索 | 既存のテーブルを検索します。 | TiDBサーバーレスクラスターとデータベース |
| テーブルの作成 | 新しいテーブルを作成します。 | TiDBサーバーレスクラスターとデータベース |
| 行の作成 | 新しい行を作成します。 | TiDBサーバーレスクラスター、データベース、およびテーブル |
| 行の更新 | 既存の行を更新します。 | TiDBサーバーレスクラスター、データベース、およびテーブル |
| 行の検索 | ルックアップ列を使用してテーブル内の行を検索します。 | TiDBサーバーレスクラスター、データベース、およびテーブル |
| 行の検索（カスタムクエリ） | 提供されたカスタムクエリを使用してテーブル内の行を検索します。 | TiDBサーバーレスクラスター、データベース、およびテーブル |

## TiDB Cloudアプリのテンプレート

TiDB CloudはZapierで直接使用するためのいくつかのテンプレートを提供しています。[TiDB Cloudアプリ](https://zapier.com/apps/tidb-cloud/integrations)ページですべてのテンプレートを見つけることができます。

以下にいくつかの例を示します。

- [新しいTiDB Cloudの行をGoogle Sheetsで複製](https://zapier.com/apps/google-sheets/integrations/tidb-cloud/1134881/duplicate-new-tidb-cloud-rows-in-google-sheets)。
- [新しいカスタムTiDBクエリからGmail経由でメールを送信](https://zapier.com/apps/gmail/integrations/tidb-cloud/1134903/send-emails-via-gmail-from-new-custom-tidb-queries)。
- [新しくキャッチしたWebフックからTiDB Cloudに行を追加](https://zapier.com/apps/tidb-cloud/integrations/webhook/1134955/add-rows-to-tidb-cloud-from-newly-caught-webhooks)。
- [新しいSalesforceの連絡先をTiDBの行に保存](https://zapier.com/apps/salesforce/integrations/tidb-cloud/1134923/store-new-salesforce-contacts-on-tidb-rows)。
- [新しいGmailの履歴メールをTiDB Cloudの行に作成し、直接Slackに通知を送信](https://zapier.com/apps/gmail/integrations/slack/1135456/create-tidb-rows-for-new-gmail-emails-with-resumes-and-send-direct-slack-notifications)。

## よくある質問

### ZapierでTiDB Cloudアカウントを設定する方法は？

ZapierはTiDB Cloudアカウントに接続するために**TiDB Cloud APIキー**が必要です。ZapierはTiDB Cloudのログインアカウントは必要としません。

TiDB Cloud APIキーを取得するには、[TiDB Cloud APIドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta#section/Authentication/API-Key-Management)に従ってください。

### TiDB Cloudトリガーは重複排除をどのように行いますか？

Zapierトリガーは新しいデータを定期的にチェックするためのポーリングAPI呼び出しで動作します（間隔はZapierプランによって異なります）。

TiDB Cloudトリガーは多くの結果を返すポーリングAPI呼び出しを提供します。ただし、ほとんどの結果はZapierが以前に見たものであり、つまりほとんどの結果は重複しています。

API内のアイテムが複数の異なるポーリングに存在する場合に動作を複数回トリガーしたくないため、TiDB Cloudトリガーは`id`フィールドでデータを重複排除します。

`New Cluster`と`New Table`トリガーは単純に`cluster_id`または`table_id`を`id`フィールドとして使用して重複排除を行います。これらの2つのトリガーに対しては、特別な設定は不要です。

**New Rowトリガー**

`New Row`トリガーは、毎回のフェッチで10,000の結果に制限されています。そのため、10,000の結果に含まれていない新しい行はZapierをトリガーできません。

これを回避する方法の1つは、トリガーで`Order By`の設定を指定することです。たとえば、行を作成した時間で行を並べ替えると、新しい行は常に10,000の結果に含まれるようになります。

`New Row`トリガーは、次の順序で`id`フィールドを生成する柔軟な戦略を使用します。

1. 結果に`id`列が含まれている場合は、`id`列を使用します。
2. トリガー構成で`Dedupe Key`を指定した場合、`Dedupe Key`を使用します。
3. テーブルにプライマリキーがある場合は、プライマリキーを使用します。複数のプライマリキーがある場合は、最初の列を使用します。
4. テーブルにユニークキーがある場合は、ユニークキーを使用します。
5. テーブルの最初の列を使用します。

**New Row (Custom Query)トリガー**

`New Row (Custom Query)`トリガーは、毎回のフェッチで1,000,000の結果に制限されています。1,000,000は大きな数であり、システム全体を保護するために設定されています。クエリに`ORDER BY`と`LIMIT`が含まれるように推奨されます。

重複排除を実行するには、クエリ結果に一意のidフィールドが必要です。そうでないと、`You must return the results with id field`のエラーが発生します。

カスタムクエリが30秒未満で実行されることを確認してください。そうでないと、タイムアウトエラーが発生します。

### `find or create`アクションをどのように使用しますか？

`find or create`アクションを使用すると、リソースが存在しない場合にリソースを作成できます。以下に例を示します。

1. `Find Table`アクションを選択します。

2. `アクションの設定`ステップで、`Create TiDB Cloud Table if it doesn’t exist yet?`ボックスをチェックして`find and create`を有効にします。

   ![Find and create](/media/tidb-cloud/zapier/zapier-tidbcloud-find-and-create.png)

このワークフローでは、テーブルが存在しない場合にテーブルを作成します。テストアクションを実行すると、テーブルが直接作成されますので注意してください。