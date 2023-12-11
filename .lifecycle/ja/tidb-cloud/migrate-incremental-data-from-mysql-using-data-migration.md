---
title: MySQL互換データベースからTiDB Cloudへの増分データのみのマイグレーションにデータマイグレーションを使用する
summary: MySQL互換データベース（Amazon Aurora MySQL、Amazon Relational Database Service（RDS）、Google Cloud SQL for MySQL、またはローカルMySQLインスタンス）にホストされている増分データを、TiDB Cloudコンソールのデータマイグレーション機能を使用して、TiDB Cloudへマイグレーションする方法について説明します。

# MySQL互換データベースからTiDB Cloudへの増分データのみのマイグレーションにデータマイグレーションを使用する

本書は、クラウドプロバイダーのMySQL互換データベース（Amazon Aurora MySQL、Amazon Relational Database Service（RDS）、またはGoogle Cloud SQL for MySQL）、または自己ホスト型のソースデータベースから、TiDB Cloudへの増分データのマイグレーション方法について説明します。その際、TiDB Cloudコンソールのデータマイグレーション機能を使用します。

既存データのマイグレーションや既存データと増分データの両方のマイグレーション方法については、[MySQL互換データベースからTiDB Cloudへのデータマイグレーション](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を参照してください。

## 制限事項

> **注意**：
>
> このセクションには増分データマイグレーションに関する制限事項のみが含まれています。一般的な制限事項についてもお読みいただくことをお勧めします。[制限事項](/tidb-cloud/migrate-from-mysql-using-data-migration.md#limitations)を参照してください。

- ターゲットテーブルがまだターゲットデータベースに作成されていない場合、マイグレーションジョブは以下のようなエラーを報告し、失敗します。この場合、ターゲットテーブルを手動で作成してから、マイグレーションジョブを再試行する必要があります。

    ```sql
    startLocation: [position: (mysql_bin.000016, 5122), gtid-set:
    00000000-0000-0000-0000-00000000000000000], endLocation:
    [position: (mysql_bin.000016, 5162), gtid-set: 0000000-0000-0000
    0000-0000000000000:0]: cannot fetch downstream table schema of
    zm`.'table1' to initialize upstream schema 'zm'.'table1' in sschema
    tracker Raw Cause: Error 1146: Table 'zm.table1' doesn't exist
    ```

- 上流でいくつかの行が削除または更新され、下流に対応する行がない場合、マイグレーションジョブは、上流から`DELETE`および`UPDATE`のDML操作をレプリケートする際に削除または更新用の行が利用できないことを検出します。

増分データのマイグレーションの開始位置としてGTIDを指定する場合は、次の制限事項に注意してください。

- ソースデータベースでGTIDモードが有効になっていることを確認してください。
- ソースデータベースがMySQLの場合、MySQLバージョンは5.6以降である必要があり、ストレージエンジンはInnoDBである必要があります。
- マイグレーションジョブが上流のセカンダリデータベースに接続する場合、「REPLICATE CREATE TABLE ... SELECT」イベントはマイグレーションできません。これは、そのステートメントが2つのトランザクション（`CREATE TABLE`および`INSERT`）に分割され、同じGTIDが割り当てられるためです。その結果、セカンダリデータベースで`INSERT`ステートメントが無視されます。

## 前提条件

> **注意**：
>
> このセクションには増分データマイグレーションに関する前提条件のみが含まれています。一般的な前提条件についてもお読みいただくことをお勧めします。[一般的な前提条件](/tidb-cloud/migrate-from-mysql-using-data-migration.md#prerequisites)を参照してください。

マイグレーションジョブの開始位置を指定するためにGTIDを使用する場合は、ソースデータベースでGTIDが有効になっていることを確認してください。データベースの種類によって操作が異なります。

### Amazon RDSおよびAmazon Aurora MySQLの場合

Amazon RDSおよびAmazon Aurora MySQLの場合は、新しい変更可能なパラメーターグループ（デフォルトのパラメーターグループではない）を作成し、次のパラメータを変更してから、インスタンスアプリケーションを再起動する必要があります。

- `gtid_mode`
- `enforce_gtid_consistency`

次のSQLステートメントを実行して、GTIDモードが正常に有効になっているかどうかを確認できます。

```sql
SHOW VARIABLES LIKE 'gtid_mode';
```

結果が`ON`または`ON_PERMISSIVE`の場合、GTIDモードが正常に有効になっています。

詳細については、[GTIDベースのレプリケーションのパラメーター](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/mysql-replication-gtid.html#mysql-replication-gtid.parameters)を参照してください。

### Google Cloud SQL for MySQLの場合

Google Cloud SQL for MySQLの場合、GTIDモードはデフォルトで有効になっています。次のSQLステートメントを実行して、GTIDモードが正常に有効になっているかどうかを確認できます。

```sql
SHOW VARIABLES LIKE 'gtid_mode';
```

結果が`ON`または`ON_PERMISSIVE`の場合、GTIDモードが正常に有効になっています。

### 自己ホスト型のMySQLインスタンスの場合

> **注意**：
>
> 具体的な手順やコマンドは、MySQLのバージョンや構成によって異なる場合があります。GTIDの有効化の影響を理解し、本番環境ではない環境で適切にテストおよび検証を行ってから、この操作を実行してください。

自己ホスト型のMySQLインスタンスでGTIDモードを有効にするには、次の手順に従ってください。

1. 適切な権限を持つMySQLクライアントを使用して、MySQLサーバーに接続します。

2. 次のSQLステートメントを実行して、GTIDモードを有効にします。

    ```sql
    -- GTIDモードを有効にする
    SET GLOBAL gtid_mode = ON;

    -- `enforce_gtid_consistency`を有効にする
    SET GLOBAL enforce_gtid_consistency = ON;

    -- GTIDの設定をリロードする
    RESET MASTER;
    ```

3. 構成変更が有効になるように、MySQLサーバーを再起動します。

4. 次のSQLステートメントを実行して、GTIDモードが正常に有効になっているかどうかを確認できます。

    ```sql
    SHOW VARIABLES LIKE 'gtid_mode';
    ```

    結果が`ON`または`ON_PERMISSIVE`の場合、GTIDモードが正常に有効になっています。

## ステップ1：**データマイグレーション**ページに移動する

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にログインし、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動します。

    > **ヒント：**
    >
    > 複数のプロジェクトを所有している場合は、左下の< MDSvgIcon name="icon-left-projects" />をクリックして、別のプロジェクトに切り替えることができます。

2. ターゲットクラスターの名前をクリックしてその概要ページに移動し、左のナビゲーションペインで**データマイグレーション**をクリックします。

3. **データマイグレーション**ページで、右上隅の**マイグレーションジョブの作成**をクリックします。**マイグレーションジョブの作成**ページが表示されます。

## ステップ2：ソースとターゲットの接続を構成する

**マイグレーションジョブの作成**ページで、ソースとターゲットの接続を構成します。

1. 文字で始まり、60文字未満である必要があるジョブ名を入力します。大文字（A-Z、a-z）、数字（0-9）、アンダースコア（_）、ハイフン（-）が許可されています。

2. ソース接続プロファイルに入力します。

   - **データソース**：データソースの種類。
   - **リージョン**：データソースのリージョン。クラウドデータベースの場合に必要です。
   - **接続方式**：データソースの接続方式。現在は、パブリックIP、VPCピアリング、またはプライベートリンクのいずれかを選択できます。
   - **ホスト名またはIPアドレス**（パブリックIPおよびVPCピアリングの場合）：データソースのホスト名またはIPアドレス。
   - **サービス名**（プライベートリンクの場合）：エンドポイントサービス名。
   - **ポート**：データソースのポート番号。
   - **ユーザー名**：データソースのユーザー名。
   - **パスワード**：ユーザー名のパスワード。
   - **SSL/TLS**：SSL/TLSを有効にする場合、次のいずれかを含むデータソースの証明書をアップロードする必要があります。
        - CA証明書のみ
        - クライアント証明書とクライアントキー
        - CA証明書、クライアント証明書、およびクライアントキー

3. ターゲット接続プロファイルに入力します。

   - **ユーザー名**：TiDB Cloudのターゲットクラスターのユーザー名を入力します。
   - **パスワード**：TiDB Cloudのユーザー名のパスワードを入力します。

4. **接続を検証して次へ**をクリックして、入力した情報を検証します。

5. 表示されるメッセージに従ってアクションを実行します。

    - パブリックIPまたはVPCピアリングを使用する場合は、データマイグレーションサービスのIPアドレスをソースデータベースおよびファイアウォール（存在する場合）のIP Accessリストに追加する必要があります。
    - AWSプライベートリンクを使用する場合は、エンドポイントリクエストを受け入れるように指示されます。[AWS VPCコンソール](https://us-west-2.console.aws.amazon.com/vpc/home)に移動し、**エンドポイントサービス**をクリックして、エンドポイントリクエストを受け入れます。

## ステップ3：マイグレーションジョブの種類を選択する

ソースデータベースの増分データのみをTiDB Cloudにマイグレーションするには、**増分データマイグレーション**を選択し、**既存データマイグレーション**を選択しないようにします。これにより、マイグレーションジョブはソースデータベースの進行中の変更のみをTiDB Cloudにマイグレーションします。

**開始位置**エリアで、増分データマイグレーションの開始位置として次のいずれかのタイプを指定できます。

- 増分マイグレーションジョブの開始時刻
- GTID
- Binlogファイル名と位置

マイグレーションジョブが開始すると、開始位置を変更することはできません。

### 増分マイグレーションジョブの開始時刻

このオプションを選択すると、マイグレーションジョブは開始された後に生成された増分データのみをマイグレーションします。

### GTIDの指定


このオプションを選択して、ソースデータベースのGTIDを指定します。たとえば、 `3E11FA47-71CA-11E1-9E33-C80AA9429562:1-23` のように指定します。移行ジョブは、指定されたGTIDセットを除外してトランザクションをレプリケートし、ソースデータベースの変更をTiDB Cloudに移行します。

次のコマンドを実行して、ソースデータベースのGTIDを確認できます。

```sql
SHOW MASTER STATUS;
```

GTIDを有効にする方法の詳細については、[前提条件](#prerequisites)を参照してください。

### binlogファイル名と位置を指定

このオプションを選択して、ソースデータベースのbinlogファイル名（たとえば、`binlog.000001`）とbinlog位置（たとえば、`1307`）を指定します。移行ジョブは、指定されたbinlogファイル名と位置から開始して、ソースデータベースの進行中の変更をTiDB Cloudに移行します。

次のコマンドを実行して、ソースデータベースのbinlogファイル名と位置を確認できます。

```sql
SHOW MASTER STATUS;
```

ターゲットデータベースにデータがある場合は、binlogの位置が正しいことを確認してください。それ以外の場合、既存のデータと増分データとの間に競合が発生する可能性があります。競合が発生した場合、移行ジョブは失敗します。競合したレコードをソースデータベースのデータで置き換えたい場合は、移行ジョブを再開できます。

## ステップ4: 移行するオブジェクトを選択

1. **移行するオブジェクトを選択** ページで、移行するオブジェクトを選択します。すべてのオブジェクトを選択するには **全て選択** をクリックします、または **カスタマイズ** をクリックして、オブジェクト名の横にあるチェックボックスをクリックしてオブジェクトを選択します。

2. **次へ** をクリックします。

## ステップ5: 事前チェック

**事前チェック** ページでは、事前チェックの結果を確認できます。事前チェックに失敗した場合は、**失敗** または **警告** の詳細に従って操作し、再チェックをクリックして再度チェックを行います。

一部のチェック項目で警告のみが表示される場合は、リスクを評価し、警告を無視するかどうかを検討できます。すべての警告が無視されると、移行ジョブは自動的に次のステップに進みます。

エラーや解決策の詳細については、[事前チェックエラーと解決策](/tidb-cloud/tidb-cloud-dm-precheck-and-troubleshooting.md#precheck-errors-and-solutions)を参照してください。

事前チェック項目の詳細については、[マイグレーションタスクの事前チェック](https://docs.pingcap.com/tidb/stable/dm-precheck)を参照してください。

すべてのチェック項目が **合格** を表示している場合は、**次へ** をクリックします。

## ステップ6: 仕様を選択して移行を開始

**仕様を選択して移行を開始** ページでは、パフォーマンス要件に応じた適切な移行仕様を選択します。仕様についての詳細については、[データ移行の仕様](/tidb-cloud/tidb-cloud-billing-dm.md#specifications-for-data-migration)を参照してください。

適切な仕様を選択したら、**ジョブを作成して開始** をクリックして移行を開始します。

## ステップ7: 移行の進捗を確認

移行ジョブが作成された後、**移行ジョブの詳細** ページで移行の進捗を確認できます。移行の進捗は **ステージとステータス** 領域に表示されます。

実行中の移行ジョブを一時停止または削除できます。

移行ジョブが失敗した場合は、問題を解決した後に再開できます。

任意のステータスで移行ジョブを削除できます。

移行中に問題が発生した場合は、[移行のエラーと解決策](/tidb-cloud/tidb-cloud-dm-precheck-and-troubleshooting.md#migration-errors-and-solutions)を参照してください。