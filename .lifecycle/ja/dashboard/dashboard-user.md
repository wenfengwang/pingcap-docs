---
title: TiDBダッシュボードユーザー管理
summary: TiDBダッシュボードにアクセスするためのSQLユーザーの作成方法を学びます。
aliases: ['/docs/dev/dashboard/dashboard-user/']
---

# TiDBダッシュボードユーザー管理

TiDBダッシュボードはTiDBと同じユーザー権限システムとサインイン認証を使用しています。TiDBダッシュボードへのアクセスを制御および管理するためにTiDB SQLユーザーを制限することができます。このドキュメントでは、TiDB SQLユーザーがTiDBダッシュボードにアクセスするために必要な最小限の権限および最小限の権限を持つSQLユーザーを作成し、RBACを介して認可する方法について説明します。

TiDB SQLユーザーを制御および管理する方法の詳細については、[TiDBユーザーアカウント管理](/user-account-management.md)を参照してください。

## 必要な権限

- TiDBサーバーにセキュリティ強化モード（SEM）(詳細は[System Variables](/system-variables.md#tidb_enable_enhanced_security)を参照)が有効でない場合、TiDBダッシュボードにアクセスするには、SQLユーザーは以下の**すべて**の権限を持つ必要があります:

    - PROCESS
    - SHOW DATABASES
    - CONFIG
    - DASHBOARD_CLIENT

- TiDBサーバーにセキュリティ強化モード（SEM）(詳細は[System Variables](/system-variables.md#tidb_enable_enhanced_security)を参照)が有効な場合、SQLユーザーは以下の**すべて**の権限を持つ必要があります:

    - PROCESS
    - SHOW DATABASES
    - CONFIG
    - DASHBOARD_CLIENT
    - RESTRICTED_TABLES_ADMIN
    - RESTRICTED_STATUS_ADMIN
    - RESTRICTED_VARIABLES_ADMIN

- TiDBダッシュボードにサインイン後にインターフェースの設定を変更するには、SQLユーザーは以下の権限も持つ必要があります:

    - SYSTEM_VARIABLES_ADMIN

- TiDBダッシュボードにサインイン後に[ファストバインド実行計画](/dashboard/dashboard-statement-details.md#fast-plan-binding)機能を使用するには、SQLユーザーは以下の権限も持つ必要があります:

    - SYSTEM_VARIABLES_ADMIN
    - SUPER

> **注意:**
>
> 「すべての権限」や「SUPER」などの高権限を持つユーザーもTiDBダッシュボードにサインインできます。したがって、最小権限の原則に従うためにも、意図しない操作を防ぐために必要な権限のみを持つユーザーを作成することを強くお勧めします。これらの権限に関する詳細については、[権限管理](/privilege-management.md)を参照してください。

前述の権限要件を満たさないSQLユーザーの場合、以下のようにTiDBダッシュボードにサインインできません。

![権限が不足](/media/dashboard/dashboard-user-insufficient-privileges.png)

## 例: TiDBダッシュボードにアクセスするための最小権限のSQLユーザーの作成

- 接続されているTiDBサーバーでセキュリティ強化モード（SEM）(詳細は[System Variables](/system-variables.md#tidb_enable_enhanced_security)を参照)が有効でない場合、TiDBダッシュボードにサインインできるSQLユーザー「dashboardAdmin」を作成するには、次のSQLステートメントを実行します:

    ```sql
    CREATE USER 'dashboardAdmin'@'%' IDENTIFIED BY '<YOUR_PASSWORD>';
    GRANT PROCESS, CONFIG ON *.* TO 'dashboardAdmin'@'%';
    GRANT SHOW DATABASES ON *.* TO 'dashboardAdmin'@'%';
    GRANT DASHBOARD_CLIENT ON *.* TO 'dashboardAdmin'@'%';

    -- TiDBダッシュボードにサインイン後にインターフェースの設定項目を変更するためには、ユーザー定義のSQLユーザーに以下の権限を付与する必要があります。
    GRANT SYSTEM_VARIABLES_ADMIN ON *.* TO 'dashboardAdmin'@'%';
    
    -- [ファストバインド実行計画](/dashboard/dashboard-statement-details.md#fast-plan-binding)機能をTiDBダッシュボードにサインイン後に使用するためには、ユーザー定義のSQLユーザーに以下の権限を付与する必要があります。
    GRANT SYSTEM_VARIABLES_ADMIN ON *.* TO 'dashboardAdmin'@'%';
    GRANT SUPER ON *.* TO 'dashboardAdmin'@'%';
    ```

- 接続されているTiDBサーバーでセキュリティ強化モード（SEM）(詳細は[System Variables](/system-variables.md#tidb_enable_enhanced_security)を参照)が有効である場合、SEMを無効にしてから次のSQLステートメントを実行して、TiDBダッシュボードにサインインできるSQLユーザー「dashboardAdmin」を作成します。ユーザーを作成した後は、SEMを再度有効にします:

    ```sql
    CREATE USER 'dashboardAdmin'@'%' IDENTIFIED BY '<YOUR_PASSWORD>';
    GRANT PROCESS, CONFIG ON *.* TO 'dashboardAdmin'@'%';
    GRANT SHOW DATABASES ON *.* TO 'dashboardAdmin'@'%';
    GRANT DASHBOARD_CLIENT ON *.* TO 'dashboardAdmin'@'%';
    GRANT RESTRICTED_STATUS_ADMIN ON *.* TO 'dashboardAdmin'@'%';
    GRANT RESTRICTED_TABLES_ADMIN ON *.* TO 'dashboardAdmin'@'%';
    GRANT RESTRICTED_VARIABLES_ADMIN ON *.* TO 'dashboardAdmin'@'%';

    -- TiDBダッシュボードにサインイン後にインターフェースの設定項目を変更するためには、ユーザー定義のSQLユーザーに以下の権限を付与する必要があります。
    GRANT SYSTEM_VARIABLES_ADMIN ON *.* TO 'dashboardAdmin'@'%';
    
    -- [ファストバインド実行計画](/dashboard/dashboard-statement-details.md#fast-plan-binding)機能をTiDBダッシュボードにサインイン後に使用するためには、ユーザー定義のSQLユーザーに以下の権限を付与する必要があります。
    GRANT SYSTEM_VARIABLES_ADMIN ON *.* TO 'dashboardAdmin'@'%';
    GRANT SUPER ON *.* TO 'dashboardAdmin'@'%';
    ```

## 例: RBACを介してTiDBダッシュボードへのSQLユーザーの認証

次の例では、[ロールベースのアクセス制御（RBAC）](/role-based-access-control.md)メカニズムを使用してTiDBダッシュボードにアクセスするためのロールとユーザーを作成する方法を示します。

1. TiDBダッシュボードのすべての権限要件を満たす「dashboard_access」ロールを作成します:

    ```sql
    CREATE ROLE 'dashboard_access';
    GRANT PROCESS, CONFIG ON *.* TO 'dashboard_access'@'%';
    GRANT SHOW DATABASES ON *.* TO 'dashboard_access'@'%';
    GRANT DASHBOARD_CLIENT ON *.* TO 'dashboard_access'@'%';
    GRANT SYSTEM_VARIABLES_ADMIN ON *.* TO 'dashboard_access'@'%';
    GRANT SUPER ON *.* TO 'dashboardAdmin'@'%';    
    ```

2. `dashboard_access`ロールを他のユーザーに付与し、`dashboard_access`をデフォルトロールに設定します:

    ```sql
    CREATE USER 'dashboardAdmin'@'%' IDENTIFIED BY '<YOUR_PASSWORD>';
    GRANT 'dashboard_access' TO 'dashboardAdmin'@'%';
    -- ユーザーに対してデフォルトロールとしてdashboard_accessを設定する必要があります
    SET DEFAULT ROLE dashboard_access to 'dashboardAdmin'@'%';
    ```

上記の手順を完了すると、`dashboardAdmin`ユーザーを使用してTiDBダッシュボードにサインインできます。

## TiDBダッシュボードにサインイン

TiDBダッシュボードの権限要件を満たすSQLユーザーを作成した後は、このユーザーを使用して[TiDBダッシュボードにサインイン](/dashboard/dashboard-access.md#sign-in)することができます。