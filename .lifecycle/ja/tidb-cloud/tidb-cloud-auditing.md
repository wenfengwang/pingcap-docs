---
title: データベース監査ログ
summary: TiDB Cloud でクラスタを監査する方法について学びます。

# データベース監査ログ

TiDB Cloud では、データベース監査ログ機能が提供され、ログにユーザーアクセスの履歴 (実行された SQL ステートメントなど) を記録します。

> **注意:**
>
> 現在、データベース監査ログ機能は要求に基づいてのみ利用可能です。この機能を要求するには、[TiDB Cloud コンソール](https://tidbcloud.com) の右下の **?** をクリックし、**サポートを要求** をクリックしてください。次に、**説明** フィールドに "データベース監査ログの申請" を記入し、**送信** をクリックします。

組織のユーザーアクセスポリシーの有効性やその他の情報セキュリティ対策を評価するために、データベース監査ログの定期的な分析を行うことは、セキュリティの最善の実践です。

監査ログ機能はデフォルトで無効になっています。クラスタを監査するためには、まず監査ログを有効にし、次に監査フィルタのルールを指定する必要があります。

> **注意:**
>
> 監査ログはクラスタのリソースを消費するため、クラスタを監査するかどうかを慎重に検討してください。

## 前提条件

- TiDB 専用クラスタを使用していること。TiDB サーバーレスクラスタでは、監査ログは利用できません。
- 組織の `組織所有者` または `プロジェクト所有者` の役割にいること。そうでない場合は、TiDB Cloud コンソールでデータベースの監査に関連するオプションを表示できません。詳細については、[ユーザーの役割](/tidb-cloud/manage-user-access.md#user-roles) を参照してください。

## AWS や Google Cloud のための監査ログの有効化

TiDB Cloud が監査ログをクラウドバケットに書き込むように許可するためには、まず監査ログを有効にする必要があります。

### AWS のための監査ログの有効化

AWS のために監査ログを有効にするには、次の手順を実行します。

#### ステップ 1. Amazon S3 バケットの作成

企業所有の AWS アカウント内の Amazon S3 バケットを指定し、TiDB Cloud が監査ログを書き込む宛先として使用します。

詳細については、AWS ユーザーガイドの [Creating a bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html) を参照してください。

#### ステップ 2. Amazon S3 アクセスの構成

> **注意:**
>
> 1 つのプロジェクト内の 1 つのクラスタに対して Amazon S3 アクセスの構成が行われると、同じプロジェクト内のすべてのクラスタからの監査ログに対して同じバケットを使用できます。

1. データベースの監査ログを有効にしたい TiDB Cloud アカウント ID と TiDB クラスタの External ID を取得します。

    1. TiDB Cloud コンソールで、AWS に展開されたプロジェクトとクラスタを選択します。
    2. **設定** > **監査設定** を選択します。 **監査ログ** ダイアログが表示されます。
    3. **監査ログ** ダイアログで、**AWS IAM ポリシー設定を表示** をクリックします。対応する TiDB Cloud アカウント ID と TiDB クラスタの External ID が表示されます。
    4. 後で使用するために TiDB Cloud アカウント ID と External ID を記録します。

2. AWS マネジメントコンソールで、**IAM** > **アクセス管理** > **ポリシー** に移動し、`s3:PutObject` の書き込み専用権限を持つストレージバケットポリシーが存在するかどうかを確認します。

    - ある場合は、後で使用するために一致するストレージバケットポリシーを記録します。
    - 存在しない場合は、**IAM** > **アクセス管理** > **ポリシー** > **ポリシーの作成** に移動し、次のポリシーテンプレートに従ってバケットポリシーを定義します。

        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": "s3:PutObject",
                    "Resource": "<Your S3 bucket ARN>/*"
                }
            ]
        }
        ```

        テンプレート内の `<Your S3 bucket ARN>` は、監査ログファイルが書き込まれる S3 バケットの Amazon Resource Name (ARN) です。S3 バケットの **Bucket Overview** エリアで ARN の値を取得することができます。`"Resource"` フィールドで ARN の後に `/*` を追加する必要があります。たとえば、ARN が `arn:aws:s3:::tidb-cloud-test` の場合、`"Resource"` フィールドの値を `"arn:aws:s3:::tidb-cloud-test/*"` に構成する必要があります。

3. **IAM** > **アクセス管理** > **役割** に移動し、以前に記録した TiDB Cloud アカウント ID と External ID に対応するトラストエンティティを持つロールが既に存在するかどうかを確認します。

    - ある場合は、後で使用するために一致するロールを記録します。
    - 存在しない場合は、**役割の作成** をクリックし、トラストエンティティのタイプとして **Another AWS account** を選択し、**アカウント ID** フィールドに TiDB Cloud アカウント ID の値を入力します。次に、**Require External ID** オプションを選択し、**External ID** フィールドに TiDB Cloud External ID の値を入力します。

4. **IAM** > **アクセス管理** > **役割** に移動し、前の手順で作成したロール名をクリックして **概要** ページに移動し、次の手順を実行します:

    1. **権限** タブで、`s3:PutObject` の書き込み専用権限を持つ記録したポリシーがロールにアタッチされていないかどうかを確認します。アタッチされていない場合は、**ポリシーのアタッチ** を選択し、必要なポリシーを検索してから **ポリシーをアタッチ** をクリックします。
    2. **概要** ページに戻り、**ロール ARN** の値をクリップボードにコピーします。

#### ステップ 3. 監査ログの有効化

TiDB Cloud コンソールで、前述の TiDB Cloud アカウント ID および External ID の値を取得した **監査ログ** ダイアログボックスに戻り、次の手順を実行します:

1. **バケット URI** フィールドに、監査ログファイルが書き込まれる S3 バケットの URI を入力します。
2. **バケットのリージョン** ドロップダウンリストから、バケットの場所に応じた AWS リージョンを選択します。
3. **ロール ARN** フィールドに、[ステップ 2. Amazon S3 アクセスの構成](#step-2-configure-amazon-s3-access) でコピーしたロール ARN の値を入力します。
4. **接続のテスト** をクリックして、TiDB Cloud がバケットにアクセスして書き込むことができるかどうかを確認します。

    成功した場合、**合格** と表示されます。それ以外の場合は、アクセス構成を確認してください。

5. 右上隅で、監査設定を **オン** に切り替えます。

    TiDB Cloud は、指定されたクラスタの監査ログをあなたの Amazon S3 バケットに書き込む準備が整いました。

> **注意:**
>
> - 監査ログを有効にした後は、バケット URI、場所、または ARN に新しい変更を加えると、変更を有効にするために **再起動** をクリックして変更を読み込み、**接続のテスト** を再実行する必要があります。
> - TiDB Cloud から Amazon S3 アクセスを削除するには、追加したトラストポリシーを削除するだけです。

### Google Cloud のための監査ログの有効化

Google Cloud のために監査ログを有効にするには、次の手順を実行します。

#### ステップ 1. GCS バケットの作成

企業所有の Google Cloud アカウント内の Google Cloud Storage (GCS) バケットを指定し、TiDB Cloud が監査ログを書き込む宛先として使用します。

詳細については、Google Cloud Storage ドキュメントの [Creating storage buckets](https://cloud.google.com/storage/docs/creating-buckets) を参照してください。

#### ステップ 2. GCS アクセスの構成

> **注意:**
>
> 1 つのプロジェクト内の 1 つのクラスタに対して GCS アクセスの構成が行われると、同じプロジェクト内のすべてのクラスタからの監査ログに対して同じバケットを使用できます。

1. データベースの監査ログを有効にしたい TiDB クラスタの Google Cloud サービス アカウント ID を取得します。

    1. TiDB Cloud コンソールで、Google Cloud Platform に展開されたプロジェクトとクラスタを選択します。
    2. **設定** > **監査設定** を選択します。 **監査ログ** ダイアログボックスが表示されます。
    3. **Google Cloud サービス アカウント ID を表示** をクリックし、次にサービスアカウント ID をコピーします。後で使用するためにコピーしたサービスアカウント ID を記録します。

2. Google Cloud コンソールで、**IAM と管理** > **役割** に移動し、ストレージコンテナの以下の書き込み専用権限を持つロールが存在するかどうかを確認します。

    - storage.objects.create
    - storage.objects.delete

    ある場合は、TiDB クラスタ用の一致するロールを記録します。存在しない場合は、TiDB クラスタ用にロールを定義するために **IAM と管理** > **役割** > **ロールの作成** に移動します。

3. **Cloud Storage** > **ブラウザ** に移動し、TiDB Cloud がアクセスする GCS バケットを選択し、**情報パネルを表示** をクリックします。

    パネルが表示されます。

4. パネルで、**主体を追加** をクリックします。

    主体を追加するためのダイアログボックスが表示されます。

5. ダイアログボックスで、次の手順を実行します:

    1. **新しい主体** フィールドに TiDB クラスタの Google Cloud サービス アカウント ID を貼り付けます。
    2. **ロール** ドロップダウンリストから、対象の TiDB クラスタのロールを選択します。
    3. **保存** をクリックします。

#### ステップ 3. 監査ログの有効化

TiDB Cloud コンソールで、前述の TiDB Cloud アカウント ID の値を取得した **監査ログ** ダイアログボックスに戻り、次の手順を実行します:

1. **バケット URI** フィールドに、GCS バケットの完全な名前を入力します。
2. **バケットリージョン** フィールドで、バケットの場所に応じたGCSリージョンを選択してください。
3. **接続のテスト** をクリックして、TiDB Cloudがバケットにアクセスして書き込みできるかを確認してください。

    成功した場合は、**合格** が表示されます。それ以外の場合は、アクセス構成を確認してください。

4. 右上隅にありますが、監査設定を**オン**に切り替えてください。

    TiDB Cloudは特定のクラスターの監査ログをAmazon S3バケットに書き込む準備ができています。

> **注意:**
>
> - 監査ログを有効にした後、バケットURIや場所に新しい変更を加えた場合は、**再起動** をクリックして変更を読み込み、**接続のテスト** を再実行して変更を有効にしてください。
> - TiDB CloudからGCSアクセスを削除するには、追加したプリンシパルを単純に削除してください。

## 監査フィルタールールを指定する

監査ログを有効にした後、キャプチャして監査ログに書き込むユーザーアクセスイベントを制御する監査フィルタールールを指定する必要があります。フィルタールールが指定されていない場合、TiDB Cloudは何も記録しません。

クラスターに対する監査フィルタールールを指定するには、以下の手順を実行してください:

1. 監査ログを有効にする監査ログダイアログボックスで下にスクロールし、**フィルタールール**のセクションを見つけてください。
2. 1行につき1つのフィルタールールを追加し、各ルールでユーザーエクスプレッション、データベースエクスプレッション、テーブルエクスプレッション、およびアクセスタイプを指定してください。

> **注意:**
>
> - フィルタールールは正規表現であり、大文字と小文字が区別されます。ワイルドカードルール`.*`を使用すると、クラスター内のすべてのユーザー、データベース、またはテーブルイベントが記録されます。
> - 監査ログはクラスターリソースを消費するため、フィルタールールを指定する際には慎重に行ってください。消費量を最小限に抑えるためには、できる限り特定のデータベースオブジェクト、ユーザー、およびアクションに監査ログの範囲を限定するフィルタールールを指定することが推奨されています。

## 監査ログの表示

TiDB Cloudの監査ログは、クラスターID、Pod ID、およびログの作成日が完全修飾ファイル名に組み込まれた読み取り可能なテキストファイルです。

例として、`13796619446086334065/tidb-0/tidb-audit-2022-04-21T18-16-29.529.log`があります。この例では、`13796619446086334065`はクラスターIDを示し、`tidb-0`はPod IDを示します。

## 監査ログの無効化

クラスターの監査ログを取得しない場合は、クラスターのページに移動し、**設定** > **監査設定**をクリックして、右上隅にある監査設定を**オフ**に切り替えてください。

> **注意:**
>
> ログファイルのサイズが10MiBに達するたびに、ログファイルはクラウドストレージバケットにプッシュされます。したがって、監査ログを無効にした後は、サイズが10MiB未満のログファイルは自動的にクラウドストレージバケットにプッシュされません。このような状況でログファイルを取得するには、[PingCAPサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

## 監査ログのフィールド

監査ログの各データベースイベントレコードには、TiDBが以下のフィールドを提供しています。

> **注意:**
>
> 以下の表では、フィールドの空の最大長さは、このフィールドのデータ型が定義済みの定数長を持つことを意味します（たとえば、INTEGERの場合は4バイトです）。

| 列 # | フィールド名 | TiDBデータ型 | 最大長 | 説明 |
|---|---|---|---|---|
| 1 | N/A | N/A | N/A | 内部使用のために予約されています |
| 2 | N/A | N/A | N/A | 内部使用のために予約されています |
| 3 | N/A | N/A | N/A | 内部使用のために予約されています |
| 4 | ID       | INTEGER |  | ユニークなイベントID  |
| 5 | TIMESTAMP | TIMESTAMP |  | イベントの時間   |
| 6 | EVENT_CLASS | VARCHAR | 15 | イベントの種類     |
| 7 | EVENT_SUBCLASS     | VARCHAR | 15 | イベントのサブタイプ |
| 8 | STATUS_CODE | INTEGER |  | ステートメントの応答ステータス   |
| 9 | COST_TIME | FLOAT |  | ステートメントによって費やされる時間    |
| 10 | HOST | VARCHAR | 16 | サーバーIP    |
| 11 | CLIENT_IP         | VARCHAR | 16 | クライアントIP   |
| 12 | USER | VARCHAR | 17 | ログインユーザー名    |
| 13 | DATABASE | VARCHAR | 64 | イベントに関連するデータベース      |
| 14 | TABLES | VARCHAR | 64 | イベントに関連するテーブル名          |
| 15 | SQL_TEXT | VARCHAR | 64 KB | マスクされたSQLステートメント   |
| 16 | ROWS | INTEGER |  | 影響を受ける行の数（`0`は行が影響を受けていないことを示します）      |

TiDBによってEVENT_CLASSフィールドの値が設定された場合、監査ログのデータベースイベントレコードには以下の追加フィールドも含まれます:

- イベントクラスの値が`CONNECTION`である場合、データベースイベントレコードには以下のフィールドも含まれます:

    | 列 # | フィールド名 | TiDBデータ型 | 最大長 | 説明 |
    |---|---|---|---|---|
    | 17 | CLIENT_PORT | INTEGER |  | クライアントポート番号 |
    | 18 | CONNECTION_ID | INTEGER |  | 接続ID |
    | 19 | CONNECTION_TYPE  | VARCHAR | 12 | `socket`または`unix-socket`経由の接続 |
    | 20 | SERVER_ID | INTEGER |  | TiDBサーバーID |
    | 21 | SERVER_PORT | INTEGER |  | TiDBサーバーがMySQLプロトコルを介してクライアントとの通信に使用するポート |
    | 22 | SERVER_OS_LOGIN_USER | VARCHAR | 17 | TiDBプロセス起動システムのユーザー名  |
    | 23 | OS_VERSION | VARCHAR | N/A | TiDBサーバーが配置されているオペレーティングシステムのバージョン  |
    | 24 | SSL_VERSION | VARCHAR | 6 | TiDBの現在のSSLバージョン |
    | 25 | PID | INTEGER |  | TiDBプロセスのPID |

- イベントクラスの値が`TABLE_ACCESS`または`GENERAL`である場合、データベースイベントレコードには以下のフィールドも含まれます:

    | 列 # | フィールド名 | TiDBデータ型 | 最大長 | 説明 |
    |---|---|---|---|---|
    | 17 | CONNECTION_ID | INTEGER |  | 接続ID   |
    | 18 | COMMAND | VARCHAR | 14 | MySQLプロトコルのコマンドタイプ  |
    | 19 | SQL_STATEMENT  | VARCHAR | 17 | SQLステートメントタイプ |
    | 20 | PID | INTEGER |  | TiDBプロセスのPID  |