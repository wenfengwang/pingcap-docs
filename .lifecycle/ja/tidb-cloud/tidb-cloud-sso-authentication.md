---
title: SSO認証
summary: Google、GitHub、またはMicrosoftアカウントを使用してTiDB Cloudコンソールにログインする方法について学びます。

# SSO認証

このドキュメントでは、基本的なシングルサインオン（SSO）認証を使用して[TiDB Cloudコンソール](https://tidbcloud.com/)にログインする方法について説明します。これは迅速かつ便利です。

TiDB CloudはGoogle、GitHub、およびMicrosoftアカウントのSSO認証をサポートしています。SSO認証を使用してTiDB Cloudにログインする場合、IDと認証情報はサードパーティのGoogle、GitHub、およびMicrosoftプラットフォームに保存されるため、アカウントのパスワードを変更したり、TiDBコンソールを使用して多要素認証（MFA）を有効にしたりすることはできません。

> **注意：**
>
> ユーザー名とパスワードでTiDB Cloudにログインする場合は、[パスワード認証](/tidb-cloud/tidb-cloud-password-authentication.md)を参照してください。

## Google SSOでサインインする

Googleアカウントでサインインするには、以下の手順を実行してください。

1. TiDB Cloudの[ログイン](https://tidbcloud.com/)ページに移動します。

2. **Googleでサインイン**をクリックします。Googleのログインページに移動します。

3. 画面の指示に従い、Googleのユーザー名とパスワードを入力します。

    ログインが成功すると、TiDB Cloudコンソールに移動します。

    > **注意：**
    >
    > - Googleでの初回サインインの場合、TiDB Cloudの規約に同意するかどうかを尋ねられます。規約を読んで同意すると、TiDB Cloudのウェルカムページが表示され、その後TiDB Cloudコンソールに移動します。
    > - Googleアカウントに二段階認証（または二要素認証とも呼ばれる）を有効にしている場合、ユーザー名とパスワードを入力した後に検証コードを入力する必要があります。

## GitHub SSOでサインインする

GitHubアカウントでサインインするには、以下の手順を実行してください。

1. TiDB Cloudの[ログイン](https://tidbcloud.com/)ページに移動します。

2. **GitHubでサインイン**をクリックします。GitHubのログインページに移動します。

3. 画面の指示に従い、GitHubのユーザー名とパスワードを入力します。

    ログインが成功すると、TiDB Cloudコンソールに移動します。

    > **注意：**
    >
    > - GitHubでの初回サインインの場合、TiDB Cloudの規約に同意するかどうかを尋ねられます。規約を読んで同意すると、TiDB Cloudのウェルカムページが表示され、その後TiDB Cloudコンソールに移動します。
    > - GitHubアカウントに二要素認証を構成している場合、ユーザー名とパスワードを入力した後に検証コードを入力する必要があります。

## Microsoft SSOでサインインする

Microsoftアカウントでサインインするには、以下の手順を実行してください。

1. TiDB Cloudの[ログイン](https://tidbcloud.com/)ページに移動します。

2. **Microsoftでサインイン**をクリックします。Microsoftのログインページに移動します。

3. 画面の指示に従い、Microsoftのユーザー名とパスワードを入力します。

    ログインが成功すると、TiDB Cloudコンソールに移動します。

    > **注意：**
    >
    > - Microsoftでの初回サインインの場合、TiDB Cloudの規約に同意するかどうかを尋ねられます。規約を読んで同意すると、TiDB Cloudのウェルカムページが表示され、その後TiDB Cloudコンソールに移動します。
    > - Microsoftアカウントに二段階認証を設定している場合、ユーザー名とパスワードを入力した後に検証コードを入力する必要があります。
