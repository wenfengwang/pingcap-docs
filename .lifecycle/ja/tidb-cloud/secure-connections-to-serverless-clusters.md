---
title: TiDB ServerlessへのTLS接続
summary: TiDB ServerlessでのTLS接続を紹介します。
aliases: ['/tidbcloud/secure-connections-to-serverless-tier-clusters']
---

# TiDB ServerlessへのTLS接続

クライアントとTiDB Serverlessクラスタ間の安全なTLS接続を確立することは、データベースへの接続における基本的なセキュリティプラクティスの1つです。TiDB Serverlessのサーバ証明書は、独立した第三者の証明書プロバイダによって発行されます。サーバサイドのデジタル証明書をダウンロードせずに、簡単にTiDB Serverlessクラスタに接続することができます。

## 前提条件

- [パスワード認証](/tidb-cloud/tidb-cloud-password-authentication.md)または[SSO認証](/tidb-cloud/tidb-cloud-sso-authentication.md)を使用してTiDB Cloudにログインします。
- [TiDB Serverlessクラスタを作成](/tidb-cloud/tidb-cloud-quickstart.md)します。

## TiDB ServerlessクラスタへのTLS接続

[TiDB Cloudコンソール](https://tidbcloud.com/)では、次の方法でTiDB Serverlessクラスタに接続するための異なる接続方法の例を取得し、以下のようにTiDB Serverlessクラスタに接続することができます：

1. プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、クラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。ダイアログが表示されます。

3. ダイアログで、エンドポイントの種類を`Public`としてデフォルト設定のままにし、希望する接続方法とオペレーティングシステムを選択します。

    - サポートされている接続方法: MySQL CLI、MyCLI、JDBC、Python、Go、Node.js。
    - サポートされているオペレーティングシステム: MacOS、Debian、CentOS/RedHat/Fedora、Alpine、OpenSUSE、Windows。

4. パスワードをまだ設定していない場合は、TiDB Serverlessクラスタに簡単に接続するためにランダムなパスワードを生成するために**パスワードの作成**をクリックします。

    > **注意:**
    >
    > - ランダムなパスワードは大文字と小文字の英字、数字、特殊文字を含む16文字からなります。
    > - ダイアログを閉じた後は、生成されたパスワードは再び表示されないため、パスワードを安全な場所に保存する必要があります。忘れた場合は、このダイアログで**パスワードのリセット**をクリックしてリセットできます。
    > - TiDB Serverlessクラスタはインターネット経由でアクセスできます。別の場所でパスワードを使用する必要がある場合、データベースのセキュリティを保証するために、リセットすることをお勧めします。

5. 接続文字列を使用してクラスタに接続します。

    > **注意:**
    >
    > TiDB Serverlessクラスタに接続する際は、ユーザ名にクラスタの接頭辞を含め、名前を引用符で括る必要があります。詳細は[ユーザ名の接頭辞](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

以下の例では、MySQL CLI、MyCLI、JDBC、Python、Go、Node.jsでの接続文字列を示しています。オペレーティングシステムの`<CA_root_path>`の取得方法については、[ルート証明書管理](#root-certificate-management)を参照してください。

<SimpleTab>
<div label="MySQL CLI">

MySQL CLIクライアントはデフォルトでTLS接続を確立しようとします。TiDB Serverlessクラスタに接続する場合は、`ssl-mode`と`ssl-ca`を設定する必要があります。

```shell
mysql --connect-timeout 15 -u <username> -h <host> -P 4000 --ssl-mode=VERIFY_IDENTITY --ssl-ca=<CA_root_path> -D test -p
```

- `--ssl-mode=VERIFY_IDENTITY`を使用すると、MySQL CLIクライアントはTLSを強制し、TiDB Serverlessクラスタを検証します。
- `--ssl-ca=<CA_root_path>`を使用して、システム上のCAルートパスを設定します。

</div>

<div label="MyCLI">

[MyCLI](https://www.mycli.net/)は、TLS関連のパラメータを使用する場合に自動的にTLSを有効にします。TiDB Serverlessクラスタに接続する場合は、`ssl-ca`と`ssl-verify-server-cert`を設定する必要があります。

```shell
mycli -u <username> -h <host> -P 4000 -D test --ssl-ca=<CA_root_path> --ssl-verify-server-cert
```

- `--ssl-ca=<CA_root_path>`を使用して、システム上のCAルートパスを設定します。
- `--ssl-verify-server-cert`を使用して、TiDB Serverlessクラスタを検証します。

</div>

<div label="JDBC">

[MySQL Connector/J](https://dev.mysql.com/doc/connector-j/en/)のTLS接続構成をここで例示します。

```
jdbc:mysql://<host>:4000/test?user=<username>&password=<your_password>&sslMode=VERIFY_IDENTITY&enabledTLSProtocols=TLSv1.2,TLSv1.3
```

- `sslMode=VERIFY_IDENTITY`を設定して、TLSを有効にし、TiDB Serverlessクラスタを検証します。JDBCはデフォルトでシステムのCAルート証明書を信頼するため、証明書の構成は必要ありません。
- `enabledTLSProtocols=TLSv1.2,TLSv1.3`を設定して、TLSプロトコルのバージョンを制限します。

</div>

<div label="Python">

[mysqlclient](https://pypi.org/project/mysqlclient/)のTLS接続構成をここで例示します。

```
host="<host>", user="<username>", password="<your_password>", port=4000, database="test", ssl_mode="VERIFY_IDENTITY", ssl={"ca": "<CA_root_path>"}
```

- `ssl_mode="VERIFY_IDENTITY"`を設定して、TLSを有効にし、TiDB Serverlessクラスタを検証します。
- `ssl={"ca": "<CA_root_path>"}`を使用して、システム上のCAルートパスを設定します。

</div>

<div label="Go">

[Go-MySQL-Driver](https://github.com/go-sql-driver/mysql)のTLS接続構成をここで例示します。

```
mysql.RegisterTLSConfig("tidb", &tls.Config{
  MinVersion: tls.VersionTLS12,
  ServerName: "<host>",
})

db, err := sql.Open("mysql", "<usename>:<your_password>@tcp(<host>:4000)/test?tls=tidb")
```

- `tls.Config`を登録してTLSを有効にし、TiDB Serverlessクラスタを検証します。Go-MySQL-DriverはデフォルトでシステムのCAルート証明書を使用するため、証明書の構成は必要ありません。
- `MinVersion: tls.VersionTLS12`を設定して、TLSプロトコルのバージョンを制限します。
- `ServerName: "<host>"`を設定して、TiDB Serverlessのホスト名を検証します。
- 新しいTLS構成を登録したくない場合は、接続文字列で`tls=true`を設定するだけで済みます。

</div>

<div label="Node.js">

[Mysql2](https://www.npmjs.com/package/mysql2/)のTLS接続構成をここで例示します。

```
host: '<host>', port: 4000,user: '<username>', password: '<your_password>', database: 'test', ssl: {minVersion: 'TLSv1.2', rejectUnauthorized: true}
```

- `ssl: {minVersion: 'TLSv1.2'}`を設定して、TLSプロトコルのバージョンを制限します。
- `ssl: {rejectUnauthorized: true}`を設定して、TiDB Serverlessクラスタを検証します。Mysql2はデフォルトでシステムのCAルート証明書を使用するため、証明書の構成は必要ありません。

</div>
</SimpleTab>

## ルート証明書の管理

### ルート証明書の発行と有効性

TiDB Serverlessは、クライアントとTiDB Serverlessクラスタ間のTLS接続のための証明書として[Let's Encrypt](https://letsencrypt.org/)の証明機関（CA）の証明書を使用します。TiDB Serverless証明書が期限切れになった場合、クラスタの正常な動作と確立されたTLS安全な接続に影響を与えることなく自動的にローテーションされます。

> **注意:**
>
> TiDB ServerlessはCAルート証明書のダウンロードを提供しないため、将来的に異なるCAが証明書を発行する可能性があるため、CAルート証明書が変更されることがあります。

クライアントがデフォルトでシステムのルートCAストアを使用する場合（JavaやGoなど）、CAルートのパスを指定せずにTiDB Serverlessクラスタに安全に接続できます。TiDB ServerlessクラスタのCA証明書を取得したい場合は、単一のCA証明書の代わりに[Mozilla CA Certificate bundle](https://curl.se/docs/caextract.html)をダウンロードして使用できます。

ただし、一部のドライバやORMはシステムのルートCAストアを使用しない場合があります。その場合は、PythonのmacOSでTiDB Serverlessクラスタに接続する際、[mysqlclient](https://github.com/PyMySQL/mysqlclient)を使用する場合などは、ドライバやORMのCAルートパスをシステムのルートCAストアに構成する必要があります。

証明書の多重化されたファイルを受け入れないGUIクライアント（DBeaverなど）を使用している場合は、[ISRG Root X1](https://letsencrypt.org/certs/isrgrootx1.pem.txt)証明書をダウンロードする必要があります。

### ルート証明書のデフォルトパス

異なるオペレーティングシステムでは、ルート証明書のデフォルトの保存パスは次のとおりです：

**MacOS**

```
/etc/ssl/cert.pem
```

**Debian / Ubuntu / Arch**

```
/etc/ssl/certs/ca-certificates.crt
```

**RedHat / Fedora / CentOS / Mageia**

```
/etc/pki/tls/certs/ca-bundle.crt
```

**Alpine**

```
/etc/ssl/cert.pem
```

**OpenSUSE**

```
/etc/ssl/ca-bundle.pem
```

**Windows**

Windowsでは、CAルートの特定のパスは提供されていません。代わりに、証明書を格納するために[レジストリ](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/local-machine-and-current-user-certificate-stores)が使用されます。このため、WindowsでCAルートのパスを指定するには、次の手順を実行してください：

1. [Mozilla CA Certificate bundle](https://curl.se/docs/caextract.html)をダウンロードし、`<path_to_mozilla_ca_cert_bundle>`など好きなパスに保存します。
2. TiDB Serverlessクラスタに接続する際に、このパス（`<path_to_mozilla_ca_cert_bundle>`）をCAルートパスとして使用してください。

## FAQs

### TiDB Serverlessクラスタに接続するためにサポートされているTLSバージョンは何ですか？

セキュリティ上の理由から、TiDB ServerlessはTLS 1.2とTLS 1.3のみをサポートし、TLS 1.0およびTLS 1.1のバージョンはサポートしていません。詳細については、IETFの[Deprecating TLS 1.0 and TLS 1.1](https://datatracker.ietf.org/doc/rfc8996/)を参照してください。

### 接続クライアントとTiDB Serverless間の双方向TLS認証はサポートされていますか？

いいえ。

TiDB Serverlessは単方向TLS認証のみをサポートしており、つまり、クライアントは公開鍵を使用してTiDB Cloudクラスタ証明書の秘密鍵の署名を検証しますが、クラスタはクライアントを検証しません。

### 安全な接続を確立するためにTiDB ServerlessをTLSで構成する必要がありますか？

標準接続では、TiDB ServerlessはTLS接続のみを許可し、非SSL/TLS接続を禁止しています。その理由は、インターネットを介してTiDB Serverlessクラスタに接続する際にデータがインターネットに露出するリスクを減らすためにSSL/TLSが最も基本的なセキュリティ対策の1つであるためです。

プライベートエンドポイント接続の場合、TiDB Cloudサービスへの高度に安全で単方向のアクセスをサポートし、データを公共のインターネットに公開しないため、TLSの構成は任意です。