---
title: 自己署名証明書の生成
summary: `openssl` を使用して自己署名証明書を生成します。
aliases: ['/docs/dev/generate-self-signed-certificates/','/docs/dev/how-to/secure/generate-self-signed-certificates/']
---

# 自己署名証明書の生成

> **注意:**
>
> クライアントとサーバー間のTLSを有効にするには、`auto-tls` の設定のみが必要です。

本書は、`openssl` を使用して自己署名証明書を生成する例を示します。また、要件に応じた証明書とキーの生成もできます。

インスタンスクラスターのトポロジーが以下のようであると仮定します:

| 名前  | ホストIP      | サービス   |
| ----- | -----------  | ---------- |
| node1 | 172.16.10.11 | PD1, TiDB1 |
| node2 | 172.16.10.12 | PD2        |
| node3 | 172.16.10.13 | PD3        |
| node4 | 172.16.10.14 | TiKV1      |
| node5 | 172.16.10.15 | TiKV2      |
| node6 | 172.16.10.16 | TiKV3      |

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

インストールについては、OpenSSLの公式の[ダウンロードドキュメント](https://www.openssl.org/source/)を参照してください。

## CA証明書の生成

証明書認証局（CA）は信頼されるエンティティであり、デジタル証明書を発行します。実際には、管理者に証明書の発行を依頼するか、信頼できるCAを使用します。CAは複数の証明書ペアを管理します。ここでは、以下のとおりに元の証明書ペアを生成する必要があります。

1. ルートキーの生成:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl genrsa -out root.key 4096
    ```

2. ルート証明書の生成:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl req -new -x509 -days 1000 -key root.key -out root.crt
    ```

3. ルート証明書の検証:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl x509 -text -in root.crt -noout
    ```

## 個々のコンポーネントのための証明書の発行

このセクションでは、個々のコンポーネントのための証明書の発行方法について説明します。

### クラスターで使用される可能性がある証明書

- tidb-server証明書: TiDBが他のコンポーネントおよびクライアントを認証するためにTiDBによって使用されます。
- tikv-server証明書: TiKVが他のコンポーネントおよびクライアントを認証するためにTiKVによって使用されます。
- pd-server証明書: PDが他のコンポーネントおよびクライアントを認証するためにPDによって使用されます。
- クライアント証明書: `pd-ctl`、`tikv-ctl`など、PD、TiKV、TiDBからクライアントを認証するために使用されます。

### TiKVインスタンス向けの証明書の発行

TiKVインスタンス向けの証明書を発行するには、以下の手順を実行します:

1. 証明書に対応するプライベートキーの生成:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl genrsa -out tikv.key 2048
    ```

2. OpenSSL構成テンプレートファイルのコピーを作成します（テンプレートファイルの実際の場所を参照してください。場所によっては複数箇所に存在する場合があります）:

    {{< copyable "shell-regular" >}}

    ```bash
    cp /usr/lib/ssl/openssl.cnf .
    ```

    実際の場所がわからない場合は、ルートディレクトリから探します:

    ```bash
    find / -name openssl.cnf
    ```

3. `openssl.cnf`を編集し、`[ req ]`フィールドの下に `req_extensions = v3_req` を追加し、`[ v3_req ]`フィールドの下に `subjectAltName = @alt_names` を追加します。最後に、新しいフィールドを作成し、SAN（代替名）の情報を編集します。

    ```
    [ alt_names ]
    IP.1 = 127.0.0.1
    IP.2 = 172.16.10.14
    IP.3 = 172.16.10.15
    IP.4 = 172.16.10.16
    ```

4. `openssl.cnf`ファイルを保存し、証明書リクエストファイルを生成します（このステップでは、証明書にCommon Nameを割り当てることもできます。これはサーバーがクライアントの正体を検証するために使用される。各コンポーネントはデフォルトで検証を有効にしておらず、設定ファイルで有効にできます）:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl req -new -key tikv.key -out tikv.csr -config openssl.cnf
    ```

5. 証明書の発行と生成:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl x509 -req -days 365 -CA root.crt -CAkey root.key -CAcreateserial -in tikv.csr -out tikv.crt -extensions v3_req -extfile openssl.cnf
    ```

6. 証明書にSANフィールドが含まれているか確認します（オプション）:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl x509 -text -in tikv.crt -noout
    ```

7. 現在のディレクトリに以下のファイルが存在することを確認します:

    ```
    root.crt
    tikv.crt
    tikv.key
    ```

他のTiDBコンポーネントに対しても証明書の発行手順は類似しており、この文書では繰り返しません。

### クライアント用の証明書の発行

クライアントに証明書を発行するには、以下の手順を実行します:

1. 証明書に対応するプライベートキーの生成:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl genrsa -out client.key 2048
    ```

2. 証明書リクエストファイルを生成します（このステップでは、証明書にCommon Nameを割り当てることもできます。これはサーバーがクライアントの正体を検証するために使用される。各コンポーネントはデフォルトで検証を有効にしておらず、設定ファイルで有効にできます）:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl req -new -key client.key -out client.csr
    ```

3. 証明書の発行と生成:

    {{< copyable "shell-regular" >}}

    ```bash
    openssl x509 -req -days 365 -CA root.crt -CAkey root.key -CAcreateserial -in client.csr -out client.crt
    ```