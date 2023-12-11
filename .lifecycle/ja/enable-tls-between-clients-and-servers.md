---
title: TiDBクライアントとサーバー間のTLSを有効にする
summary: 暗号化された接続を使用してデータのセキュリティを確保します。
aliases: ['/docs/dev/enable-tls-between-clients-and-servers/','/docs/dev/how-to/secure/enable-tls-clients/','/docs/dev/encrypted-connections-with-tls-protocols/']
---

# TiDBクライアントとサーバー間でのTLSを有効にする

TiDBのサーバーとクライアント間の非暗号化接続はデフォルトで許可されており、これによりチャネルトラフィックを監視する第三者がクエリコンテンツやクエリ結果を含む、サーバーとクライアント間で送受信されるデータを知ることができます。チャネルが信頼できない場合（たとえばクライアントが公共ネットワークを介してTiDBサーバーに接続している場合）、非暗号化接続は情報漏洩の危険があるため、セキュリティ上の理由から暗号化された接続を要求することが推奨されています。

TiDBサーバーは、TLS（トランスポートレイヤーセキュリティ）に基づく暗号化された接続をサポートしています。このプロトコルはMySQLの暗号化接続と一貫しており、MySQLクライアント（MySQL Client、MySQL Shell、MySQLドライバなど）でも直接サポートされています。TLSは時々SSL（Secure Sockets Layer）と呼ばれます。SSLプロトコルには [既知のセキュリティ脆弱性](https://en.wikipedia.org/wiki/Transport_Layer_Security) があるため、TiDBはSSLをサポートしていません。TiDBは次のプロトコルをサポートしています：TLSv1.0、TLSv1.1、TLSv1.2、TLSv1.3。

暗号化された接続を使用する場合、接続には以下のセキュリティプロパティがあります：

- 機密性：トラフィックの平文は盗聴を避けるために暗号化されます
- 完全性：トラフィックの平文は改ざんできません
- 認証：（オプション）クライアントはサーバーの識別を検証し、サーバーはクライアントの識別を検証し、中間者攻撃を回避します

TLSで保護された接続を使用するには、まずTiDBサーバーを設定してTLSを有効にする必要があります。その後、クライアントアプリケーションをTLSを使用するように設定する必要があります。ほとんどのクライアントライブラリは、サーバーが正しくTLSサポートが構成されている場合、自動でTLSを有効にします。

MySQLと同様に、TiDBは同じTCPポートでTLSおよび非TLS接続を許可します。TLSが有効なTiDBサーバーの場合、暗号化された接続を介してTiDBサーバーに安全に接続するか、または非暗号化接続を使用するかを選択できます。安全な接続の使用を要求するには、以下の方法を使用できます：

+ システム変数 `require_secure_transport` を構成して、すべてのユーザーに対してTiDBサーバーへの安全な接続を要求します。
+ ユーザーの作成（`create user`）時または既存のユーザーの修正（`alter user`）時に `REQUIRE SSL` を指定して、指定されたユーザーが暗号化された接続を使用してTiDBにアクセスする必要があることを指定します。以下はユーザーの作成の例です：

    {{< copyable "sql" >}}

    ```sql
    CREATE USER 'u1'@'%' IDENTIFIED BY 'my_random_password' REQUIRE SSL;
    ```

> **注記:**
>
> ログインユーザーが [ログイン認証のためのTiDB証明書ベースの認証](/certificate-authentication.md#configure-the-user-certificate-information-for-login-verification) を構成した場合、ユーザーは暗号化された接続をTiDBに有効にする必要が暗黙的に要求されます。

## TiDBサーバーを安全な接続に設定する

安全な接続を有効にする関連パラメータに関する以下の説明を参照してください：

- [`auto-tls`](/tidb-configuration-file.md#auto-tls)：自動証明書生成を有効にします（v5.2.0以降）
- [`ssl-cert`](/tidb-configuration-file.md#ssl-cert)：SSL証明書のファイルパスを指定します
- [`ssl-key`](/tidb-configuration-file.md#ssl-key)：証明書に対応するプライベートキーを指定します
- [`ssl-ca`](/tidb-configuration-file.md#ssl-ca)：（オプション）信頼できるCA証明書のファイルパスを指定します
- [`tls-version`](/tidb-configuration-file.md#tls-version)：（オプション）最小TLSバージョンを指定します。例："TLSv1.2"

`auto-tls` は安全な接続を可能にしますが、クライアント証明書の検証を提供しません。証明書の検証および証明書の生成方法を制御するためのアドバイスについては、以下の `ssl-cert`、`ssl-key`、および `ssl-ca` 変数の構成に関するアドバイスを参照してください。

TiDBサーバーで独自の証明書を使用して安全な接続を有効にするには、TiDBサーバーを起動する際に構成ファイルで `ssl-cert` および `ssl-key` パラメータの両方を指定する必要があります。クライアントの認証のために `ssl-ca` パラメータを指定することもできます（[認証を有効にする](#enable-authentication) を参照）。

各パラメータで指定されたファイルは、PEM（Privacy Enhanced Mail）形式です。現在、TiDBはパスワードで保護されたプライベートキーのインポートをサポートしていないため、パスワードで保護されたプライベートキーファイルを提供する必要があります。証明書またはプライベートキーが無効な場合、TiDBサーバーは通常通り起動しますが、クライアントは暗号化された接続を介してTiDBサーバーに接続することができません。

証明書パラメータが正しい場合、TiDBは起動時に `secure connection is enabled` を出力します。それ以外の場合、`secure connection is NOT ENABLED` を出力します。

v5.2.0より前のTiDBバージョンでは、`mysql_ssl_rsa_setup --datadir=./certs` を使用して証明書を生成できます。`mysql_ssl_rsa_setup` ツールはMySQL Serverの一部です。

## MySQLクライアントを暗号化された接続に設定する

MySQL 5.7以降のバージョンのクライアントはデフォルトで暗号化された接続を確立しようとします。サーバーが暗号化された接続をサポートしていない場合、自動的に非暗号化接続に戻ります。MySQL 5.7より前のバージョンのクライアントはデフォルトで非暗号化接続を使用します。

クライアントの接続動作を変更するには、次の `--ssl-mode` パラメータを使用できます：

- `--ssl-mode=REQUIRED`：クライアントは暗号化された接続を要求します。サーバー側が暗号化された接続をサポートしていない場合、接続を確立できません。
- `--ssl-mode` パラメータがない場合：クライアントは暗号化された接続を試みますが、サーバー側が暗号化された接続をサポートしていない場合、暗号化された接続を確立することができません。その場合、クライアントは非暗号化接続を使用します。
- `--ssl-mode=DISABLED`：クライアントは非暗号化接続を使用します。

MySQL 8.0クライアントには、このパラメータに加えて2つのSSLモードがあります：

- `--ssl-mode=VERIFY_CA`：`--ssl-ca` を必要とするサーバーからの証明書を検証します。
- `--ssl-mode=VERIFY_IDENTITY`：`VERIFY_CA` と同じですが、接続しようとするホスト名が証明書と一致することも検証します。

詳細については、MySQLの[クライアント側暗号化接続の構成](https://dev.mysql.com/doc/refman/8.0/en/using-encrypted-connections.html#using-encrypted-connections-client-side-configuration) を参照してください。

## 認証を有効にする

TiDBサーバーまたはMySQLクライアントで `ssl-ca` パラメータが指定されていない場合、デフォルトではクライアントまたはサーバーが認証を実行せず、中間者攻撃を防ぐことができません。たとえば、クライアントは偽装されたクライアントに「安全に」接続する可能性があります。サーバーおよびクライアントの認証のために `ssl-ca` パラメータを構成できます。通常、サーバーを認証するだけで十分ですが、さらにセキュリティを強化するためにクライアントを認証することもできます。

+ MySQLクライアントからTiDBサーバーへの認証を行うには：
  1. TiDBサーバーで `ssl-cert` および `ssl-key` パラメータを指定します。
  2. MySQLクライアントで `--ssl-ca` パラメータを指定します。
  3. MySQLクライアントで少なくとも `--ssl-mode` を `VERIFY_CA` に指定します。
  4. TiDBサーバーで構成された証明書（`ssl-cert`）がクライアントで指定された `--ssl-ca` パラメータによって指定されたCAによって署名されていることを確認してください。そうでない場合、認証に失敗します。

+ TiDBサーバーからMySQLクライアントへの認証を行うには：
  1. TiDBサーバーで `ssl-cert`、`ssl-key`、および `ssl-ca` パラメータを指定します。
  2. クライアントで `--ssl-cert` および `--ssl-key` パラメータを指定します。
  3. サーバーで構成された証明書およびクライアントで構成された証明書が両方ともサーバーが指定した `ssl-ca` によって署名されていることを確認してください。

- 相互認証を実行するには、上記の要件を満たしてください。

デフォルトでは、サーバーからクライアントへの認証はオプションです。TLSハンドシェイク中にクライアントの識別証明書を提出しなくても、TLS接続を確立できます。また、`require x509` を指定してユーザーの作成（`create user`）、権限の付与（`grant`）、または既存のユーザーの修正（`alter user`）時に、クライアントの認証を必要とすることができます。以下はユーザーの作成の例です：

{{< copyable "sql" >}}

```sql
create user 'u1'@'%'  require x509;
```

> **注記:**
>
> ログインユーザーが [ログイン認証のためのTiDB証明書ベースの認証](/certificate-authentication.md#configure-the-user-certificate-information-for-login-verification) を構成した場合、ユーザーは暗号化された接続をTiDBに有効にする必要が暗黙的に要求されます。

## 現在の接続が暗号化を使用しているか確認する

`SHOW STATUS LIKE "%Ssl%";` 文を使用して、現在の接続の詳細を取得し、暗号化が使用されているか、暗号化された接続で使用される暗号化プロトコルやTLSバージョン番号を含む詳細を取得できます。

以下の例は暗号化された接続の結果の例です。クライアントでサポートされているさまざまなTLSバージョンや暗号化プロトコルに応じて結果が変わります。

```
mysql> SHOW STATUS LIKE "%Ssl%";
......
| Ssl_verify_mode | 5                            |
| Ssl_version     | TLSv1.2                      |
| Ssl_cipher      | ECDHE-RSA-AES128-GCM-SHA256  |
......

公式のMySQLクライアントでは、接続状態を確認するために `STATUS` または `\s` ステートメントを使用することもできます。

```
mysql> \s
...
SSL: Cipher in use is ECDHE-RSA-AES128-GCM-SHA256
...
```

## サポートされているTLSバージョン、キー交換プロトコル、および暗号化アルゴリズム

TiDBでサポートされているTLSバージョン、キー交換プロトコル、および暗号化アルゴリズムは、公式のGolangライブラリによって決定されます。

使用しているオペレーティングシステムとクライアントライブラリの暗号ポリシーも、サポートされているプロトコルと暗号スイートのリストに影響を与える可能性があります。

### サポートされているTLSバージョン

- TLSv1.0 (デフォルトでは無効)
- TLSv1.1
- TLSv1.2
- TLSv1.3

`tls-version` 構成オプションを使用して使用できるTLSバージョンを制限できます。

実際に使用できるTLSバージョンは、OSの暗号ポリシー、MySQLクライアントのバージョン、およびクライアントが使用するSSL/TLSライブラリに依存します。

### サポートされているキー交換プロトコルと暗号化アルゴリズム

- TLS_RSA_WITH_AES_128_CBC_SHA
- TLS_RSA_WITH_AES_256_CBC_SHA
- TLS_RSA_WITH_AES_128_CBC_SHA256
- TLS_RSA_WITH_AES_128_GCM_SHA256
- TLS_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
- TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
- TLS_AES_128_GCM_SHA256
- TLS_AES_256_GCM_SHA384
- TLS_CHACHA20_POLY1305_SHA256

## 証明書、キー、およびCAのリロード

証明書、キー、またはCAを置き換えるには、まず対応するファイルを置き換え、その後実行中のTiDBインスタンスで [`ALTER INSTANCE RELOAD TLS`](/sql-statements/sql-statement-alter-instance.md) ステートメントを実行して、元の構成パスから証明書（[`ssl-cert`](/tidb-configuration-file.md#ssl-cert)）、キー（[`ssl-key`](/tidb-configuration-file.md#ssl-key)）、およびCA（[`ssl-ca`](/tidb-configuration-file.md#ssl-ca)）を再読み込みします。この方法で、TiDBインスタンスを再起動する必要はありません。

新しく読み込まれた証明書、キー、およびCAは、ステートメントが正常に実行された後に確立された接続に影響します。ステートメントの実行前に確立された接続は影響を受けません。

## モニタリング

TiDB v5.2.0以降、`Ssl_server_not_after` と `Ssl_server_not_before` ステータス変数を使用して、証明書の有効期間の開始日と終了日をモニタリングできます。

```sql
SHOW GLOBAL STATUS LIKE 'Ssl\_server\_not\_%';
```

```
+-----------------------+--------------------------+
| Variable_name         | Value                    |
+-----------------------+--------------------------+
| Ssl_server_not_after  | Nov 28 06:42:32 2021 UTC |
| Ssl_server_not_before | Aug 30 06:42:32 2021 UTC |
+-----------------------+--------------------------+
2 rows in set (0.0076 sec)
```

## 関連項目

- [TiDBコンポーネント間のTLSを有効にする](/enable-tls-between-components.md)