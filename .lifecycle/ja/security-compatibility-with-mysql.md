---
title: MySQLとのセキュリティ互換性
summary: TiDBのMySQLとのセキュリティ互換性について学びます。
aliases: ['/docs/dev/security-compatibility-with-mysql/','/docs/dev/reference/security/compatibility/']
---

# MySQLとのセキュリティ互換性

TiDBは、MySQL 5.7と類似したセキュリティ機能をサポートしており、一部のMySQL 8.0のセキュリティ機能もサポートしています。TiDBのセキュリティ機能は、MySQLと異なる実装となっています。

## サポートされていないセキュリティ機能

- カラムレベルの権限。
- 次の権限属性：`max_questions`、`max_updates`、`max_user_connections`。
- パスワード確認ポリシー（変更時に現在のパスワードを確認する必要があるポリシー）。
- 二重パスワードポリシー。
- ランダムパスワード生成。
- マルチファクタ認証。

## MySQLとの違い

### パスワード有効期限ポリシー

TiDBとMySQLのパスワード有効期限ポリシーには、以下の違いがあります。

- MySQLはv5.7とv8.0でパスワード有効期限ポリシーをサポートしています。
- TiDBはv6.5.0からパスワード有効期限ポリシーをサポートしています。

TiDBの有効期限メカニズムは、以下の点でMySQLと異なります。

- MySQL v5.7およびv8.0では、クライアントとサーバーの構成が組み合わさり、「サンドボックスモード」をクライアント接続に有効にするかどうかを決定します。
- TiDBでは、[`security.disconnect-on-expired-password`](/tidb-configuration-file.md#disconnect-on-expired-password-new-in-v650)の構成項目のみが、クライアント接続の「サンドボックスモード」を有効にするかどうかを決定します。

### パスワード複雑性ポリシー

TiDBとMySQLのパスワード複雑性ポリシーには、以下の違いがあります。

- MySQL v5.7は`validate_password`プラグインを使用してパスワード複雑性ポリシーを実装しています。
- MySQL v8.0は`validate_password`コンポーネントを使用してパスワード複雑性ポリシーを再実装しています。
- TiDBはv6.5.0から組み込みのパスワード複雑性管理機能を導入しています。

機能の実装には、以下の違いがあります。

- 機能の有効化:

    - MySQL v5.7では、`validate_password`プラグインを使用して機能を実装します。プラグインをインストールすることで機能を有効にできます。
    - MySQL v8.0では、`validate_password`コンポーネントを使用して機能を実装します。コンポーネントをインストールすることで機能を有効にできます。
    - TiDBでは、この機能は組み込みのものです。システム変数[`validate_password.enable`](/system-variables.md#validate_passwordenable-new-in-v650)を使用して機能を有効にできます。

- 辞書チェック:

    - MySQL v5.7では、`validate_password_dictionary_file`変数を使用してファイルパスを指定できます。ファイルにはパスワードに存在してはいけない単語のリストが含まれています。
    - MySQL v8.0では、`validate_password.dictionary_file`変数を使用してファイルパスを指定できます。ファイルにはパスワードに存在してはいけない単語のリストが含まれています。
    - TiDBでは、[`validate_password.dictionary`](/system-variables.md#validate_passworddictionary-new-in-v650)システム変数を使用して文字列を指定できます。文字列にはパスワードに存在してはいけない単語のリストが含まれています。

### パスワード失敗追跡ポリシー

TiDBとMySQLのパスワード失敗追跡ポリシーには、以下の違いがあります。

- MySQL v5.7はパスワード失敗追跡をサポートしていません。
- MySQL v8.0はパスワード失敗追跡をサポートしています。
- TiDBはv6.5.0からパスワード失敗追跡をサポートしています。

失敗した試行の数とアカウントのロック状態をグローバルに一貫させる必要があるため、分散データベースであるTiDBでは、MySQLと異なる実装メカニズムがあります。

- 自動的にロックされないユーザーに対して、失敗した試行の数は以下のシナリオでリセットされます:

    + MySQL 8.0:

        - サーバーが再起動されると、すべてのアカウントの失敗した試行の数がリセットされます。
        - `FLUSH PRIVILEGES`が実行されると、すべてのアカウントの失敗した試行の数がリセットされます。
        - `ALTER USER ... ACCOUNT UNLOCK`が実行され、アカウントがロック解除されると、数がリセットされます。
        - アカウントが成功したログイン時に、数がリセットされます。

    + TiDB:

        - `ALTER USER ... ACCOUNT UNLOCK`を実行してアカウントをロック解除すると、数がリセットされます。
        - アカウントが成功したログイン時に、数がリセットされます。

- 自動的にロックされるユーザーに対して、失敗した試行の数は以下のシナリオでリセットされます:

    + MySQL 8.0:

        - サーバーが再起動されると、すべてのアカウントの一時的なロックがリセットされます。
        - `FLUSH PRIVILEGES`が実行されると、すべてのアカウントの一時的なロックがリセットされます。
        - アカウントのロック時間が終了すると、次のログイン試行でアカウントの一時的なロックがリセットされます。
        - `ALTER USER ... ACCOUNT UNLOCK`が実行され、アカウントがロック解除されると、アカウントの一時的なロックがリセットされます。

    + TiDB:

        - アカウントのロック時間が終了すると、次のログイン試行でアカウントの一時的なロックがリセットされます。
        - `ALTER USER ... ACCOUNT UNLOCK`を実行してアカウントをロック解除すると、アカウントの一時的なロックがリセットされます。

### パスワード再利用ポリシー

TiDBとMySQLのパスワード再利用ポリシーには、以下の違いがあります。

- MySQL v5.7はパスワードの再利用管理をサポートしていません。
- MySQL v8.0はパスワードの再利用管理をサポートしています。
- TiDBはv6.5.0からパスワードの再利用管理をサポートしています。

実装メカニズムはTiDBとMySQLで一貫しています。両方ともパスワードの再利用管理機能を実装するために`mysql.password_history`システムテーブルを使用します。ただし、`mysql.user`システムテーブルに存在しないユーザーを削除する際、TiDBとMySQLでは異なる動作があります。

- シナリオ: ユーザー(`user01`)が通常の方法で作成されず、代わりに`INSERT INTO mysql.password_history VALUES (...)`ステートメントを使用して`user01`のレコードを`mysql.password_history`システムテーブルに追加した場合、`mysql.user`システムテーブルに`user01`のレコードが存在しないため、`DROP USER`を`user01`に実行した際、TiDBとMySQLでは異なる動作がします。

    - MySQL: `DROP USER user01`を実行すると、MySQLは`user01`を`mysql.user`と`mysql.password_history`で検索しようとします。どちらかのシステムテーブルに`user01`が含まれている場合、`DROP USER`ステートメントが正常に実行され、エラーは報告されません。
    - TiDB: `DROP USER user01`を実行すると、TiDBは`mysql.user`でのみ`user01`を検索しようとします。関連するレコードが見つからない場合、`DROP USER`ステートメントは失敗し、エラーが報告されます。ステートメントを正常に実行して`mysql.password_history`から`user01`のレコードを削除するには、代わりに`DROP USER IF EXISTS user01`を使用してください。

## 認証プラグインの状態

TiDBは複数の認証メソッドをサポートしています。これらのメソッドは[`CREATE USER`](/sql-statements/sql-statement-create-user.md)と[`ALTER USER`](/sql-statements/sql-statement-create-user.md)を使用してユーザーごとに指定できます。これらの方法はMySQLの同じ名前の認証メソッドと互換性があります。

以下のサポートされている認証メソッドのいずれかをテーブルで使用できます。サーバーとクライアントの接続が確立されている際にサーバーが広告するデフォルトのメソッドを指定するには、[`default_authentication_plugin`](/system-variables.md#default_authentication_plugin)変数を設定します。`tidb_sm3_password`はTiDBでのみサポートされているSM3認証メソッドです。したがって、このメソッドを使用して認証するには、[TiDB-JDBC](https://github.com/pingcap/mysql-connector-j/tree/release/8.0-sm3)を使用してTiDBに接続する必要があります。`tidb_auth_token`はTiDB Cloudでのみ使用されるJSON Web Token（JWT）ベースの認証メソッドです。

<CustomContent platform="tidb">

TLS認証のサポートは異なる方法で構成されます。詳細については、[TiDBクライアントとサーバー間でTLSを有効にする](/enable-tls-between-clients-and-servers.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

TLS認証のサポートは異なる方法で構成されます。詳細については、[TiDBクライアントとサーバー間でTLSを有効にする](https://docs.pingcap.com/tidb/stable/enable-tls-between-clients-and-servers)を参照してください。

</CustomContent>

| 認証メソッド                 | サポート |
| :---------------------------| :------- |
| `mysql_native_password`      | はい     |
| `sha256_password`            | いいえ   |
| `caching_sha2_password`      | はい、v5.2.0以降 |
| `auth_socket`                | はい、v5.3.0以降 |
| `tidb_sm3_password`          | はい、v6.3.0以降 |
| `tidb_auth_token`            | はい、v6.4.0以降 |
| `authentication_ldap_sasl`   | はい、v7.1.0以降 |
| `authentication_ldap_simple` | はい、v7.1.0以降 |
| TLS証明書                    | はい     |
| LDAP                         | はい、v7.1.0以降 |
| PAM                          | いいえ   |
| ed25519（MariaDB）             | いいえ   |
| GSSAPI（MariaDB）              | いいえ   |
| FIDO                         | いいえ   |