---
title: TiDBデータ移行用の自己署名証明書の生成
summary: `openssl`を使用して自己署名証明書を生成します。
---

# TiDBデータ移行用の自己署名証明書の生成

このドキュメントでは、TiDBデータ移行（DM）用の自己署名証明書を生成するために`openssl`を使用した例を提供します。また、要件に応じて証明書とキーを生成することも可能です。

インスタンスクラスターのトポロジーが以下のようであると仮定します。

| 名前  | ホストIP      | サービス   |
| ----- | -----------  | ---------- |
| node1 | 172.16.10.11 | DM-master1 |
| node2 | 172.16.10.12 | DM-master2 |
| node3 | 172.16.10.13 | DM-master3 |
| node4 | 172.16.10.14 | DM-worker1 |
| node5 | 172.16.10.15 | DM-worker2 |
| node6 | 172.16.10.16 | DM-worker3 |

## OpenSSLのインストール

- DebianまたはUbuntu OSの場合:

    {{< copyable "shell-regular" >}}

    ```bash
    apt install openssl
    ```

- RedHatまたはCentOS OSの場合:

    {{< copyable "shell-regular" >}}

    ```bash
    yum install openssl
    ```

インストールについては、OpenSSLの公式の[ダウンロードドキュメント](https://www.openssl.org/source/)を参照することもできます。

## CA証明書の生成

証明書機関（CA）はデジタル証明書を発行する信頼されたエンティティです。実際には、管理者に証明書の発行を依頼するか、信頼できるCAを使用してください。CAは複数の証明書ペアを管理します。ここでは、次のように元の証明書ペアを生成するだけです。

1. CAキーを生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl genrsa -out ca-key.pem 4096
    ```

2. CA証明書を生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl req -new -x509 -days 1000 -key ca-key.pem -out ca.pem
    ```

3. CA証明書を検証します:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl x509 -text -in ca.pem -noout
    ```

## 個々のコンポーネントの証明書の発行

### クラスターで使用される証明書

- DM-masterが他のコンポーネントを認証するために使用する`master`証明書。
- DM-workerが他のコンポーネントを認証するために使用する`worker`証明書。
- dmctlがDM-masterおよびDM-workerを認証するために使用する`client`証明書。

### DM-masterのための証明書の発行

DM-masterインスタンスに証明書を発行するには、次の手順を実行します。

1. 証明書に対応するプライベートキーを生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl genrsa -out master-key.pem 2048
    ```

2. OpenSSL構成テンプレートファイルのコピーを作成します（テンプレートファイルの実際の場所は異なることがあるため、実際の場所を参照してください）:

    {{< copyable "shell-regular" >}}

    ```bash
    cp /usr/lib/ssl/openssl.cnf .
    ```

    実際の場所がわからない場合は、ルートディレクトリで検索してください:

    ```bash
    find / -name openssl.cnf
    ```

3. `openssl.cnf`を編集し、`[ req ]`フィールドの下に`req_extensions = v3_req`を追加し、`[ v3_req ]`フィールドの下に`subjectAltName = @alt_names`を追加します。最後に、新しいフィールドを作成し、クラスターのトポロジーの説明に従って`Subject Alternative Name`（SAN）の情報を編集します。

    ```
    [ alt_names ]
    IP.1 = 127.0.0.1
    IP.2 = 172.16.10.11
    IP.3 = 172.16.10.12
    IP.4 = 172.16.10.13
    ```

    現在サポートされているSANの確認項目は以下のとおりです:

    - `IP`
    - `DNS`
    - `URI`

    > **注意:**
    >
    > 接続や通信に`0.0.0.0`などの特別なIPを使用する場合は、それを`alt_names`に追加する必要があります。

4. `openssl.cnf`ファイルを保存し、証明書リクエストファイルを生成します（`Common Name（e.g. server FQDN or YOUR name) []:`に入力を与えると、証明書にCommon Name（CN）を割り当てます。例えば`dm`などです。サーバーはこれを使用してクライアントの正体を検証します。各コンポーネントはデフォルトで検証を有効にしていませんが、構成ファイルで有効にできます）。

    {{< copyable "shell-regular" >}}

    ```bash
    openssl req -new -key master-key.pem -out master-cert.pem -config openssl.cnf
    ```

5. 証明書を発行し生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl x509 -req -days 365 -CA ca.pem -CAkey ca-key.pem -CAcreateserial -in master-cert.pem -out master-cert.pem -extensions v3_req -extfile openssl.cnf
    ```

6. 証明書にSANフィールドが含まれていることを検証します（オプション）:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl x509 -text -in master-cert.pem -noout
    ```

7. 現在のディレクトリに以下のファイルが存在していることを確認します:

    ```
    ca.pem
    master-cert.pem
    master-key.pem
    ```

> **注意:**
>
> DM-workerインスタンスに証明書を発行する手順は類似しており、このドキュメントで繰り返す必要はありません。

### クライアント（dmctl）のための証明書の発行

クライアント（dmctl）に証明書を発行するには、次の手順を実行します:

1. 証明書に対志するプライベートキーを生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl genrsa -out client-key.pem 2048
    ```

2. 証明書リクエストファイルを生成します（このステップで、証明書にCommon Nameを割り当てることもできます。これはサーバーがクライアントの正体を検証するために使用されます。各コンポーネントはデフォルトで検証を有効にしていませんが、構成ファイルで有効にできます）:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl req -new -key client-key.pem -out client-cert.pem
    ```

3. 証明書を発行し生成します:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl x509 -req -days 365 -CA ca.pem -CAkey ca-key.pem -CAcreateserial -in client-cert.pem -out client-cert.pem
    ```