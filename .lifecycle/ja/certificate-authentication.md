---
title: ログイン用の証明書ベース認証
summary: ログイン用の証明書ベース認証について学ぶ。
aliases: ['/docs/dev/certificate-authentication/','/docs/dev/reference/security/cert-based-authentication/']
---

# ログイン用の証明書ベース認証

TiDBは、ユーザーがTiDBにログインするための証明書ベースの認証方法をサポートしています。この方法では、TiDBは異なるユーザーに証明書を発行し、データを転送する際に暗号化された接続を使用し、ユーザーがログインする際に証明書を検証します。このアプローチは、MySQLユーザーがよく使用する伝統的なパスワードベースの認証方法よりもセキュリティが高く、それによりますます多くのユーザーに採用されています。

証明書ベースの認証を使用するには、次の操作を実行する必要があります。

+ セキュリティキーと証明書を作成する
+ TiDBとクライアントのために証明書を構成する
+ ユーザーの証明書情報をログイン時に検証するために構成する
+ 証明書を更新および置換する

ここからは、これらの操作を詳細に紹介します。

## セキュリティキーと証明書を作成する

<CustomContent platform="tidb">

セキュリティキーや証明書を作成する際に[OpenSSL](https://www.openssl.org/)の使用をお勧めします。証明書の生成プロセスは[クライアントとサーバーの間のTLSを有効にする](/enable-tls-between-clients-and-servers.md)で説明されているプロセスと類似しています。以下の段落では、証明書で検証する必要のある追加属性フィールドを構成する方法を示します。

</CustomContent>

<CustomContent platform="tidb-cloud">

セキュリティキーや証明書を作成する際に[OpenSSL](https://www.openssl.org/)の使用をお勧めします。証明書の生成プロセスは[クライアントとサーバーの間のTLSを有効にする](https://docs.pingcap.com/tidb/stable/enable-tls-between-clients-and-servers)で説明されているプロセスと類似しています。以下の段落では、証明書で検証する必要のある追加属性フィールドを構成する方法を示します。

</CustomContent>

### CAキーと証明書を生成する

1. 次のコマンドを実行してCAキーを生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl genrsa 2048 > ca-key.pem
    ```

    上記コマンドの出力:

    ```
    RSAプライベートキーを生成中、2048ビットの長さのモジュラス（2つの素数）
    ....................+++++
    ...............................................+++++
    eは65537（0x010001）
    ```

2. 次のコマンドを実行して、CAキーに対応する証明書を生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl req -new -x509 -nodes -days 365000 -key ca-key.pem -out ca-cert.pem
    ```

3. 詳細な証明書情報を入力します。例:

    {{< copyable "shell-regular" >}}

    ```bash
    Country Name (2 letter code) [AU]:US
    State or Province Name (full name) [Some-State]:California
    Locality Name (e.g. city) []:San Francisco
    Organization Name (e.g. company) [Internet Widgits Pty Ltd]:PingCAP Inc.
    Organizational Unit Name (e.g. section) []:TiDB
    Common Name (e.g. server FQDN or YOUR name) []:TiDB admin
    Email Address []:s@pingcap.com
    ```

    > **注記:**
    >
    > 上記の証明書詳細では、`:`の後ろのテキストが入力された情報です。

### サーバーキーと証明書を生成する

1. 次のコマンドを実行してサーバーキーを生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl req -newkey rsa:2048 -days 365000 -nodes -keyout server-key.pem -out server-req.pem
    ```

2. 詳細な証明書情報を入力します。例:

    {{< copyable "shell-regular" >}}

    ```bash
    Country Name (2 letter code) [AU]:US
    State or Province Name (full name) [Some-State]:California
    Locality Name (e.g. city) []:San Francisco
    Organization Name (e.g. company) [Internet Widgits Pty Ltd]:PingCAP Inc.
    Organizational Unit Name (e.g. section) []:TiKV
    Common Name (e.g. server FQDN or YOUR name) []:TiKV Test Server
    Email Address []:k@pingcap.com

    下記の「追加」属性を入力します
    証明書リクエストと共に送信するための
    チャレンジパスワード []:
    オプションの会社名 []:
    ```

3. 次のコマンドを実行してサーバーのRSAキーを生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl rsa -in server-key.pem -out server-key.pem
    ```

    上記コマンドの出力:

    ```bash
    RSAキーを書き込んでいます
    ```

4. CA証明書署名を使用してサーバー証明書を生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl x509 -req -in server-req.pem -days 365000 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem
    ```

    上記コマンドの出力（例）:

    ```bash
    Signature ok
    subject=C = US, ST = California, L = San Francisco, O = PingCAP Inc., OU = TiKV, CN = TiKV Test Server, emailAddress = k@pingcap.com
    Getting CA Private Key
    ```

    > **注記:**
    >
    > ログイン時、TiDBは上記出力の`subject`セクションの情報が一致しているかどうかを確認します。

### クライアントキーと証明書を生成する

サーバーキーと証明書を生成した後、クライアントのキーと証明書を生成する必要があります。異なるユーザーに対して異なるキーと証明書を生成することがしばしば必要です。

1. 次のコマンドを実行して、クライアントキーを生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl req -newkey rsa:2048 -days 365000 -nodes -keyout client-key.pem -out client-req.pem
    ```

2. 詳細な証明書情報を入力します。例:

    {{< copyable "shell-regular" >}}

    ```bash
    Country Name (2 letter code) [AU]:US
    State or Province Name (full name) [Some-State]:California
    Locality Name (e.g. city) []:San Francisco
    Organization Name (e.g. company) [Internet Widgits Pty Ltd]:PingCAP Inc.
    Organizational Unit Name (e.g. section) []:TiDB
    Common Name (e.g. server FQDN or YOUR name) []:tpch-user1
    Email Address []:zz@pingcap.com

    下記の「追加」属性を入力します
    証明書リクエストと共に送信するための
    チャレンジパスワード []:
    オプションの会社名 []:
    ```

3. 次のコマンドを実行してクライアントのRSAキーを生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl rsa -in client-key.pem -out client-key.pem
    ```

    上記コマンドの出力:

    ```bash
    RSAキーを書き込んでいます
    ```

4. CA証明書署名を使用してクライアント証明書を生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl x509 -req -in client-req.pem -days 365000 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem
    ```

    上記コマンドの出力（例）:

    ```bash
    Signature ok
    subject=C = US, ST = California, L = San Francisco, O = PingCAP Inc., OU = TiDB, CN = tpch-user1, emailAddress = zz@pingcap.com
    Getting CA Private Key
    ```

    > **注記:**
    >
    > 上記出力の`subject`セクションの情報は、[ログイン検証のための証明書構成](#configure-the-user-certificate-information-for-login-verification)で使用されます。

### 証明書を検証する

証明書を検証するために、次のコマンドを実行します:

{{< copyable "shell-regular" >}}

```bash
openssl verify -CAfile ca-cert.pem server-cert.pem client-cert.pem
```

証明書が検証されると、次の結果が表示されます:

```
server-cert.pem: OK
client-cert.pem: OK
```

## TiDBとクライアントの証明書を構成する

証明書を生成した後、TiDBサーバーとクライアントを対応するサーバー証明書またはクライアント証明書を使用するように構成する必要があります。

### サーバー証明書の使用をTiDBに構成

TiDB構成ファイルの`[security]`セクションを修正します。このステップでは、CA証明書、サーバーキー、サーバー証明書が格納されているディレクトリが指定されます。`path/to/server-cert.pem`、`path/to/server-key.pem`、`path/to/ca-cert.pem`は、独自のディレクトリに置き換えてください。

{{< copyable "" >}}

```
[security]
ssl-cert ="path/to/server-cert.pem"
ssl-key ="path/to/server-key.pem"
ssl-ca="path/to/ca-cert.pem"
```

TiDBを起動し、ログを確認します。次の情報がログに表示されたら、構成は成功です:

```
```plaintext
[INFO] [server.go:264] ["secure connection is enabled"] ["client verification enabled"=true]
```

### クライアントをクライアント証明書を使用するように構成

クライアントがクライアントキーと証明書を使用してログインするようにクライアントを構成します。

MySQLクライアントを例に取ると、新しく作成されたクライアント証明書、クライアントキー、CAを`ssl-cert`、`ssl-key`、`ssl-ca`で指定することができます。

{{< copyable "shell-regular" >}}

```bash
mysql -utest -h0.0.0.0 -P4000 --ssl-cert /path/to/client-cert.new.pem --ssl-key /path/to/client-key.new.pem --ssl-ca /path/to/ca-cert.pem
```

> **Note:**
>
> `/path/to/client-cert.new.pem`、`/path/to/client-key.new.pem`、`/path/to/ca-cert.pem`はCA証明書、クライアントキー、クライアント証明書のディレクトリです。独自のディレクトリに置き換えてください。

## ログイン検証のためのユーザ証明書情報を構成

まず、クライアントを使用してTiDBに接続し、ログイン検証を構成します。そして、検証するユーザ証明書情報を取得し構成します。

### ユーザ証明書情報を取得

ユーザ証明書情報は`require subject`、`require issuer`、`require san`、`require cipher`によって指定され、X509証明書属性のチェックに使用されます。

+ `require subject`: ログインする際のクライアント証明書の`subject`情報を指定します。このオプションが指定されている場合、`require ssl`またはx509を構成する必要はありません。指定する情報は[クライアントキーと証明書を生成](#generate-client-key-and-certificate)で入力される`subject`情報と一致しています。

    このオプションを取得するには、次のコマンドを実行してください：

    {{< copyable "shell-regular" >}}

    ```bash
    openssl x509 -noout -subject -in client-cert.pem | sed 's/.\{8\}//'  | sed 's/, /\//g' | sed 's/ = /=/g' | sed 's/^/\//'
    ```

+ `require issuer`: ユーザ証明書を発行するCA証明書の`subject`情報を指定します。指定する情報は[CAキーと証明書を生成](#generate-ca-key-and-certificate)で入力される`subject`情報と一致しています。

    このオプションを取得するには、次のコマンドを実行してください：

    {{< copyable "shell-regular" >}}

    ```bash
    openssl x509 -noout -subject -in ca-cert.pem | sed 's/.\{8\}//'  | sed 's/, /\//g' | sed 's/ = /=/g' | sed 's/^/\//'
    ```

+ `require san`: ユーザ証明書を発行するCA証明書の`Subject Alternative Name`情報を指定します。指定する情報は[openssl.cnf構成ファイルの`alt_names`](https://docs.pingcap.com/tidb/stable/generate-self-signed-certificates)で使用される情報と一致している必要があります。

    + 生成済み証明書の`require san`アイテムの情報を取得するには、次のコマンドを実行します：

        {{< copyable "shell-regular" >}}

        ```shell
        openssl x509 -noout -extensions subjectAltName -in client.crt
        ```

    + `require san`は現在、次の`Subject Alternative Name`チェックアイテムをサポートしています：

        - URI
        - IP
        - DNS

    + 複数のチェックアイテムは、カンマで接続した後に構成することができます。たとえば、`u1`ユーザでは`require san`を次のように構成します：

        {{< copyable "sql" >}}

        ```sql
        create user 'u1'@'%' require san 'DNS:d1,URI:spiffe://example.org/myservice1,URI:spiffe://example.org/myservice2';
        ```

        上記の構成では、`u1`ユーザが`spiffe://example.org/myservice1`または`spiffe://example.org/myservice2`のURI項目と`d1`のDNS項目を持つ証明書を使用してのみTiDBにログインすることができます。

+ `require cipher`: クライアントがサポートする暗号方式をチェックします。次のステートメントを使用して、サポートされる暗号方式の一覧をチェックしてください：

    {{< copyable "sql" >}}

    ```sql
    SHOW SESSION STATUS LIKE 'Ssl_cipher_list';
    ```

### ユーザ証明書情報を構成

ユーザ証明書情報（`require subject`、`require issuer`、`require san`、`require cipher`）を取得したら、これらの情報をユーザの作成、権限の付与、またはユーザの変更時に検証されるように構成してください。次の文の対応する情報を`<replaceable>`で置き換えてください。

スペースまたは`and`を区切り記号として使用して、1つまたは複数のオプションを構成できます。

+ ユーザの作成時にユーザ証明書を構成する場合（`create user`）：

    {{< copyable "sql" >}}

    ```sql
    create user 'u1'@'%' require issuer '<replaceable>' subject '<replaceable>' san '<replaceable>' cipher '<replaceable>';
    ```

+ 権限の付与時にユーザ証明書を構成する場合：

    {{< copyable "sql" >}}

    ```sql
    grant all on *.* to 'u1'@'%' require issuer '<replaceable>' subject '<replaceable>' san '<replaceable>' cipher '<replaceable>';
    ```

+ ユーザの変更時にユーザ証明書を構成する場合：

    {{< copyable "sql" >}}

    ```sql
    alter user 'u1'@'%' require issuer '<replaceable>' subject '<replaceable>' san '<replaceable>' cipher '<replaceable>';
    ```

上記の構成後、次の項目がログイン時に検証されます：

+ SSLが使用されており、クライアント証明書を発行するCAがサーバーで構成されたCAと一致している。
+ クライアント証明書の`issuer`情報が`require issuer`で指定された情報と一致している。
+ クライアント証明書の`subject`情報が`require cipher`で指定された情報と一致している。
+ クライアント証明書の`Subject Alternative Name`情報が`require san`で指定された情報と一致している。

上記のいずれかが検証されない場合、`ERROR 1045 (28000): Access denied`エラーが返されます。TLSバージョン、暗号アルゴリズム、および現在の接続がログインに証明書を使用しているかどうかを確認するには、次のコマンドを使用してください。

MySQLクライアントに接続し、次のステートメントを実行します：

{{< copyable "sql" >}}

```sql
\s
```

出力：

```
--------------
mysql  Ver 15.1 Distrib 10.4.10-MariaDB, for Linux (x86_64) using readline 5.1

Connection id:       1
Current database:    test
Current user:        root@127.0.0.1
SSL:                 Cipher in use is TLS_AES_256_GCM_SHA384
```

次に、次のステートメントを実行してください：

{{< copyable "sql" >}}

```sql
show variables like '%ssl%';
```

出力：

```
+---------------+----------------------------------+
| Variable_name | Value                            |
+---------------+----------------------------------+
| ssl_cert      | /path/to/server-cert.pem         |
| ssl_ca        | /path/to/ca-cert.pem             |
| have_ssl      | YES                              |
| have_openssl  | YES                              |
| ssl_key       | /path/to/server-key.pem          |
+---------------+----------------------------------+
6 rows in set (0.067 sec)
```

## 証明書の更新および置換

キーと証明書は定期的に更新されます。次のセクションでは、キーと証明書の更新方法について説明します。

CA証明書はクライアントとサーバー間の相互検証の基礎です。CA証明書を置換するには、新しいCA証明書を使用して古い証明書および新しい証明書の両方をサポートする統合された証明書を生成します。最初に、クライアントおよびサーバーで古いCA証明書を置換し、次にクライアント/サーバーキーと証明書を置換します。

### CAキーと証明書を更新

1. 古いCAキーおよび証明書をバックアップする（`ca-key.pem`が盗まれた場合を想定）

    {{< copyable "shell-regular" >}}

    ```bash
    mv ca-key.pem ca-key.old.pem && \
    mv ca-cert.pem ca-cert.old.pem
    ```

2. 新しいCAキーを生成

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl genrsa 2048 > ca-key.pem
    ```

3. 新しく生成されたCAキーを使用して新しいCA証明書を生成

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl req -new -x509 -nodes -days 365000 -key ca-key.pem -out ca-cert.new.pem
    ```

    > **Note:**
    >
    > 新しいCA証明書を生成することは、クライアントおよびサーバー上のキーおよび証明書を置換し、オンラインユーザーに影響を与えないようにするため、上記のコマンドで追加される情報は`require issuer`情報と一致している必要があります。

4. 統合されたCA証明書を生成する

    {{< copyable "shell-regular" >}}

    ```bash
    cat ca-cert.new.pem ca-cert.old.pem > ca-cert.pem
    ```

上記の操作後、新しく生成された統合されたCA証明書を使用してTiDBサーバーを再起動します。それから、サーバーは新しいCA証明書および古いCA証明書の両方を受け入れます。
```
### クライアントキーと証明書を更新する

> **注意：**
>
> 古いCA証明書をクライアントとサーバー上で結合されたCA証明書に置き換えた後に、以下の手順を実行してください。

1. クライアントの新しいRSAキーを生成します：

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl req -newkey rsa:2048 -days 365000 -nodes -keyout client-key.new.pem -out client-req.new.pem && \
    sudo openssl rsa -in client-key.new.pem -out client-key.new.pem
    ```

    > **注意：**
    >
    上記のコマンドは、クライアントのキーと証明書を置換し、オンラインユーザーに影響を与えないようにするためのものです。したがって、上記のコマンドに付加される情報は、`必要な主体`情報と整合している必要があります。

2. 結合された証明書と新しいCAキーを使用して、新しいクライアント証明書を生成します：

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl x509 -req -in client-req.new.pem -days 365000 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.new.pem
    ```

3. クライアント（例：MySQL）を、新しいクライアントキーと証明書を使用してTiDBに接続します：

    {{< copyable "shell-regular" >}}

    ```bash
    mysql -utest -h0.0.0.0 -P4000 --ssl-cert /path/to/client-cert.new.pem --ssl-key /path/to/client-key.new.pem --ssl-ca /path/to/ca-cert.pem
    ```

    > **注意：**
    >
    `/path/to/client-cert.new.pem`、`/path/to/client-key.new.pem`、`/path/to/ca-cert.pem` は、CA証明書、クライアントキー、クライアント証明書のディレクトリを指定しています。これらは独自のディレクトリに置き換えることができます。

### サーバーキーと証明書を更新する

1. サーバーの新しいRSAキーを生成します：

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl req -newkey rsa:2048 -days 365000 -nodes -keyout server-key.new.pem -out server-req.new.pem && \
    sudo openssl rsa -in server-key.new.pem -out server-key.new.pem
    ```

2. 結合されたCA証明書と新しいCAキーを使用して、新しいサーバー証明書を生成します：

    {{< copyable "shell-regular" >}}

    ```bash
    sudo openssl x509 -req -in server-req.new.pem -days 365000 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.new.pem
    ```

3. TiDBサーバーを新しいサーバーキーと証明書を使用するように構成します。詳細は[Configure TiDB server](#configure-tidb-and-the-client-to-use-certificates)を参照してください。