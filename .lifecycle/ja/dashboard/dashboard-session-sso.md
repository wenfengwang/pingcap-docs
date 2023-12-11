---
title: TiDB ダッシュボードのSSOを構成する
summary: TiDB ダッシュボードにサインインするためのSSOを有効にする方法を学ぶ

# TiDB ダッシュボードのSSOを構成する

TiDB ダッシュボードは[OIDC](https://openid.net/connect/)ベースのSingle Sign-On (SSO)をサポートしています。TiDB ダッシュボードのSSO機能を有効にした後、構成されたSSOサービスを使用してサインイン認証を行い、その後はSQLユーザーパスワードを入力せずにTiDBダッシュボードにアクセスできます。

## OIDC SSOを構成する

### SSOを有効にする

1. TiDB ダッシュボードにサインインします。

2. 左サイドバーのユーザー名をクリックして構成ページにアクセスします。

3. **Single Sign-On** セクションで **Enable to use SSO when sign into TiDB Dashboard** を選択します。

4. フォームの **OIDC Client ID** および **OIDC Discovery URL** のフィールドを入力します。

    一般的に、2つのフィールドはSSOサービスプロバイダから取得できます:

    - OIDC Client ID は OIDC Token Issuer とも呼ばれます。
    - OIDC Discovery URL は OIDC Token Audience とも呼ばれます。

5. **Authorize Impersonation** をクリックし、SQLパスワードを入力します。

    TiDBダッシュボードはこのSQLパスワードを保存し、SSOサインインが完了した後に通常のSQLサインインを模倣するために使用します。

    ![Sample Step](/media/dashboard/dashboard-session-sso-enable-1.png)

    > **注記:**
    >
    > 入力したパスワードは暗号化されて保存されます。SQLユーザーのパスワードが変更されると、SSOサインインが失敗します。この場合は、再度パスワードを入力してSSOを再開できます。

6. **Authorize and Save** をクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-sso-enable-2.png)

7. **Update** (更新) をクリックして構成を保存します。

    ![Sample Step](/media/dashboard/dashboard-session-sso-enable-3.png)

これでTiDBダッシュボードのSSOサインインが有効になりました。

> **注記:**
>
> セキュリティの観点から、一部のSSOサービスは、信頼されたサインインおよびサインアウトのURIなど、SSOサービスの追加構成を要求する場合があります。詳細についてはSSOサービスのドキュメントを参照してください。

### SSOを無効にする

保存されたSQLパスワードを完全に削除するためにSSOを無効にできます:

1. TiDB ダッシュボードにサインインします。

2. 左サイドバーのユーザー名をクリックして構成ページにアクセスします。

3. **Single Sign-On** セクションで **Enable to use SSO when sign into TiDB Dashboard** の選択を解除します。

4. **Update** (更新) をクリックして構成を保存します。

    ![Sample Step](/media/dashboard/dashboard-session-sso-disable.png)

### パスワードの変更後にパスワードを再入力する

SQLユーザーのパスワードが変更されると、SSOサインインは失敗します。この場合は、SQLパスワードを再入力することでSSOサインインを再開できます:

1. TiDB ダッシュボードにサインインします。

2. 左サイドバーのユーザー名をクリックして構成ページにアクセスします。

3. **Single Sign-On** セクションで**Authorize Impersonation** をクリックし、更新されたSQLパスワードを入力します。

    ![Sample Step](/media/dashboard/dashboard-session-sso-reauthorize.png)

4. **Authorize and Save** をクリックします。

## SSO経由でサインインする

TiDBダッシュボードにSSOが構成されたら、次の手順でSSO経由でサインインできます:

1. TiDBダッシュボードのサインインページで、 **Sign in via Company Account** をクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-sso-signin.png)

2. 構成されたSSOサービスでシステムにサインインします。

3. TiDBダッシュボードにリダイレクトされ、サインインを完了します。

## 例1: TiDBダッシュボードのSSOサインインにOktaを使用する

[Okta](https://www.okta.com/)はOIDC SSOアイデンティティサービスであり、TiDBダッシュボードのSSO機能と互換性があります。以下の手順は、OktaとTiDBダッシュボードを構成して、OktaをTiDBダッシュボードのSSOプロバイダとして使用する方法を示しています。

### ステップ1: Oktaを構成する

まず、Oktaアプリケーション統合を作成してSSOを統合します。

1. Okta管理サイトにアクセスします。

2. 左サイドバーから **Applications** > **Applications** に移動します。

3. **Create App Integration** をクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-sso-okta-1.png)

4. ポップアップダイアログで、**Sign-in method** で **OIDC - OpenID Connect** を選択します。

5. **Application Type** で **Single-Page Application** を選択します。

6. **Next** ボタンをクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-sso-okta-2.png)

7. **Sign-in redirect URIs** を次のように入力します:

    ```
    http://DASHBOARD_IP:PORT/dashboard/?sso_callback=1
    ```

    ブラウザでTiDBダッシュボードにアクセスする際の実際のドメイン（またはIPアドレス）とポートで `DASHBOARD_IP:PORT` を置き換えます。

8. **Sign-out redirect URIs** を次のように入力します:

    ```
    http://DASHBOARD_IP:PORT/dashboard/
    ```

    同様に、`DASHBOARD_IP:PORT` を実際のドメイン（またはIPアドレス）とポートに置き換えます。

    ![Sample Step](/media/dashboard/dashboard-session-sso-okta-3.png)

9. 組織のユーザータイプをSSOサインインに許可するように**Assignments**フィールドを構成し、構成を保存するために**Save**をクリックします。

    ![Sample Step](/media/dashboard/dashboard-session-sso-okta-4.png)

### ステップ2: OIDC情報を取得し、TiDBダッシュボードに入力する

Oktaで作成したアプリケーション統合で **Sign On** をクリックします。

    ![Sample Step 1](/media/dashboard/dashboard-session-sso-okta-info-1.png)

2. **OpenID Connect ID Token** セクションで **Issuer** と **Audience** フィールドの値をコピーします。

    ![Sample Step 2](/media/dashboard/dashboard-session-sso-okta-info-2.png)

3. TiDBダッシュボード構成ページを開き、**OIDC Client ID** フィールドを前の手順で取得した**Issuer**で、**OIDC Discovery URL** フィールドを**Audience**で入力します。その後、承認を完了し、構成を保存します。例:

    ![Sample Step 3](/media/dashboard/dashboard-session-sso-okta-info-3.png)

これで、TiDBダッシュボードはOkta SSOを使用するように構成されました。

## 例2: TiDBダッシュボードのSSOサインインにAuth0を使用する

Oktaと同様に、[Auth0](https://auth0.com/)もOIDC SSOアイデンティティサービスを提供しています。以下の手順では、Auth0とTiDBダッシュボードを構成して、Auth0をTiDBダッシュボードのSSOプロバイダとして使用する方法を説明します。

### ステップ1: Auth0を構成する

1. Auth0管理サイトにアクセスします。

2. 左サイドバーから **Applications** > **Applications** に移動します。

3. **Create App Integration** をクリックします。

    ![Create Application](/media/dashboard/dashboard-session-sso-auth0-create-app.png)

    ポップアップダイアログで、**Name** を入力し、「TiDBダッシュボード」とします。**Choose an application type** で **Single Page Web Applications** を選択して **Create** をクリックします。

4. **Settings** をクリックします。

    ![Settings](/media/dashboard/dashboard-session-sso-auth0-settings-1.png)

5. **Allowed Callback URLs** を次のように入力します:

    ```
    http://DASHBOARD_IP:PORT/dashboard/?sso_callback=1
    ```

    ブラウザでTiDBダッシュボードにアクセスする際の実際のドメイン（またはIPアドレス）とポートで `DASHBOARD_IP:PORT` を置き換えます。

6. **Allowed Logout URLs** を次のように入力します:

    ```
    http://DASHBOARD_IP:PORT/dashboard/
    ```

    同様に、`DASHBOARD_IP:PORT` を実際のドメイン（またはIPアドレス）とポートに置き換えます。

    ![Settings](/media/dashboard/dashboard-session-sso-auth0-settings-2.png)

7. 他の設定についてはデフォルト値のままにして、**Save Changes** をクリックします。

### ステップ2: OIDC情報を取得し、TiDBダッシュボードに入力する

1. TiDBダッシュボードの **Basic Information** の**Settings**タブで **Client ID** を**OIDC Client ID**に入力します。

2. **Domain** フィールドの値に`https://`をプレフィックスとして、末尾に`/`を付けたものを**OIDC Discovery URL** に入力します。例: `https://example.us.auth0.com/`。承認を完了し、構成を保存します。

    ![Settings](/media/dashboard/dashboard-session-sso-auth0-settings-3.png)

これでTiDBダッシュボードはAuth0 SSOを使用するように構成されました。

## 例3: TiDBダッシュボードのSSOサインインにCasdoorを使用する

[Casdoor](https://casdoor.org/)は、独自のホストに展開できるオープンソースのSSOプラットフォームであり、TiDBダッシュボードのSSO機能と互換性があります。以下の手順では、CasdoorとTiDBダッシュボードを構成して、CasdoorをTiDBダッシュボードのSSOプロバイダとして使用する方法を説明します。

### ステップ1: Casdoorを構成する

1. Casdoor管理サイトを展開してアクセスします。
```
2. サイドバーの上部から**Applications**に移動します。

3. **Applications - Add**をクリックします。
    ![Settings](/media/dashboard/dashboard-session-sso-casdoor-settings-1.png)

4. **Name**と**Display name**を入力します。例えば、**TiDB Dashboard**とします。

5. **Redirect URLs**を以下のように追加します：

    ```
    http://DASHBOARD_IP:PORT/dashboard/?sso_callback=1
    ```

    `DASHBOARD_IP:PORT`を、ブラウザでTiDB Dashboardにアクセスする際に使用する実際のドメイン（またはIPアドレス）とポートに置き換えます。

    ![Settings](/media/dashboard/dashboard-session-sso-casdoor-settings-2.png)

6. 他の設定のデフォルト値を保持し、**Save & Exit**をクリックします。

7. ページに表示されている**Client ID**を保存します。

### Step 2: OIDC情報を取得し、TiDB Dashboardに入力します

1. TiDBダッシュボードの**OIDC Client ID**に、前の手順で保存した**Client ID**を入力します。

2. **OIDC Discovery URL**に、**Domain**フィールドの値に`https://`を前置し、`/`を後置したものを入力します。例えば、`https://casdoor.example.com/`とします。認証を完了し、設定を保存します。

    ![Settings](/media/dashboard/dashboard-session-sso-casdoor-settings-3.png)

これで、TiDB DashboardはCasdoor SSOを使用するように設定されました。
```