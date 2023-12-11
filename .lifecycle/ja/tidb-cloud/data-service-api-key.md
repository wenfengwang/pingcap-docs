---
title: データサービスのAPIキー
summary: データアプリのAPIキーの作成、編集、削除方法を学びます。

# データサービスのAPIキー

TiDB Cloud Data APIは、[Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication)と[Digest Authentication](https://en.wikipedia.org/wiki/Digest_access_authentication)の両方をサポートしています。

- [Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication)は、非暗号化のbase64エンコーディングを使用して公開キーと秘密キーを送信します。HTTPSは送信セキュリティを確保します。詳細については[RFC 7617 - The 'Basic' HTTP Authentication Scheme](https://datatracker.ietf.org/doc/html/rfc7617)を参照してください。
- [Digest Authentication](https://en.wikipedia.org/wiki/Digest_access_authentication)は、公開キー、秘密キー、サーバー提供のノンス値、HTTPメソッド、要求されたURIをハッシュ化してネットワーク送信前に追加のセキュリティレイヤーを提供します。これにより、秘密キーが平文で送信されることを防ぎます。詳細については[RFC 7616 - HTTP Digest Access Authentication](https://datatracker.ietf.org/doc/html/rfc7616)を参照してください。

> **注意:**
>
> データサービスのデータAPIキーと[TiDB Cloud API](https://docs.pingcap.com/tidbcloud/api/v1beta#section/Authentication)で使用されるキーは異なります。データAPIキーはTiDB Cloudクラスタ内のデータにアクセスするために使用され、一方、TiDB Cloud APIキーはプロジェクト、クラスタ、バックアップ、リストア、インポートなどのリソースを管理するために使用されます。

## APIキーの概要

- APIキーには、ユーザー名と認証に必要なパスワードである公開キーと秘密キーが含まれます。秘密キーはキー作成時にのみ表示されます。
- 各APIキーは1つのデータアプリにのみ属し、TiDB Cloudクラスタ内のデータにアクセスするために使用されます。
- すべてのリクエストで正しいAPIキーを提供する必要があります。それ以外の場合、TiDB Cloudは`401`エラーで応答します。

## レート制限

リクエストクォータは次のようにレート制限されます:

- APIキーあたり1分間に100リクエスト（rpm）
- Chat2Queryデータアプリごとに1日に100リクエスト

制限を超えると、APIは`429`エラーを返します。クォータを増やすには、サポートチームに[リクエストを送信](https://support.pingcap.com/hc/en-us/requests/new?ticket_form_id=7800003722519)することができます。

## APIキーの管理

以下のセクションで、データアプリのAPIキーの作成、編集、削除方法について説明します。

### APIキーの作成

データアプリのAPIキーを作成するには、次の手順を実行します:

1. プロジェクトの[**データサービス**](https://tidbcloud.com/console/data-service)ページに移動します。
2. 左ペインで対象のデータアプリの名前をクリックして詳細を表示します。
3. **認証**エリアで**APIキーの作成**をクリックします。
4. **APIキーの作成**ダイアログボックスで、説明を入力し、APIキーのロールを選択します。

    ロールは、APIキーがデータアプリにリンクされたクラスタに対してデータを読み取るか書き込むかを制御するために使用されます。`ReadOnly`または`ReadAndWrite`ロールを選択できます:

    - `ReadOnly`: `SELECT`、`SHOW`、`USE`、`DESC`、`EXPLAIN`などのデータを読み取ることのみを許可します。
    - `ReadAndWrite`: データを読み書きすることができます。DMLやDDLなどのすべてのSQLステートメントを実行するためにこのAPIキーを使用できます。

5. **次へ**をクリックします。公開キーと秘密キーが表示されます。

    ページを離れる前に、秘密キーをコピーして安全な場所に保存しておいてください。それ以降、完全な秘密キーを取得することはできません。

6. **完了**をクリックします。

### APIキーの編集

APIキーの説明を編集するには、次の手順を実行します:

1. プロジェクトの[**データサービス**](https://tidbcloud.com/console/data-service)ページに移動します。
2. 左ペインで対象のデータアプリの名前をクリックして詳細を表示します。
3. **APIキー**エリアで、変更したいAPIキーの**アクション**列を見つけ、そのAPIキー行の**...** > **編集**をクリックします。
4. APIキーの説明またはロールを更新します。
5. **更新**をクリックします。

### APIキーの削除

> **注意:**
>
> APIキーを削除する前に、APIキーがどのデータアプリにも使用されていないことを確認してください。

データアプリのAPIキーを削除するには、次の手順を実行します:

1. プロジェクトの[**データサービス**](https://tidbcloud.com/console/data-service)ページに移動します。
2. 左ペインで対象のデータアプリの名前をクリックして詳細を表示します。
3. **APIキー**エリアで、削除したいAPIキーの**アクション**列を見つけ、そのAPIキー行の**...** > **削除**をクリックします。
4. 表示されるダイアログボックスで、削除を確認します。