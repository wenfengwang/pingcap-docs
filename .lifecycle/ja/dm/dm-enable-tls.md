---
title: DM接続のTLSを有効にする
summary: DM接続のTLSを有効にする方法について説明します。

# DM接続のTLSを有効にする

このドキュメントでは、DM接続間でのデータ送信の暗号化を有効にする方法について説明します。DM-master、DM-worker、dmctl間の接続およびDMと上流または下流データベース間の接続を含みます。

## DM-master、DM-worker、およびdmctl間のデータ送信の暗号化を有効にする

このセクションでは、DM-master、DM-worker、およびdmctl間のデータ送信の暗号化を有効にする方法について紹介します。

### 暗号化されたデータ送信を構成して有効にする

1. 証明書の準備

    DM-masterとDM-workerそれぞれにサーバー証明書を準備することをお勧めします。両方のコンポーネントがお互いを認証できるようにしてください。dmctl用の1つのクライアント証明書を共有することもできます。

    自己署名証明書を生成するには、`openssl`、`cfssl`、および`easy-rsa`などの`openssl`ベースのツールを使用できます。

    `openssl`を選択する場合は、[自己署名証明書の生成](/dm/dm-generate-self-signed-certificates.md)を参照してください。

2. 証明書の構成

    > **注意：**
    >
    > DM-master、DM-worker、およびdmctlを同じ証明書セットを使用するように構成できます。

    - DM-master

        構成ファイルまたはコマンドライン引数で構成:

        ```toml
        ssl-ca = "/path/to/ca.pem"
        ssl-cert = "/path/to/master-cert.pem"
        ssl-key = "/path/to/master-key.pem"
        ```

    - DM-worker

        構成ファイルまたはコマンドライン引数で構成:

        ```toml
        ssl-ca = "/path/to/ca.pem"
        ssl-cert = "/path/to/worker-cert.pem"
        ssl-key = "/path/to/worker-key.pem"
        ```

    - dmctl

        DMクラスタで暗号化された送信を有効にした後、dmctlを使用してクラスタに接続する必要がある場合は、クライアント証明書を指定します。例:

        {{< copyable "shell-regular" >}}

        ```bash
        ./dmctl --master-addr=127.0.0.1:8261 --ssl-ca /path/to/ca.pem --ssl-cert /path/to/client-cert.pem --ssl-key /path/to/client-key.pem
        ```

### コンポーネント呼び出し元のアイデンティティの確認

一般的には、呼び出し先は、呼び出し元のキー、証明書、およびCAの提供者が提供するエイリアスだけでなく、呼び出し元のアイデンティティを確認する必要があります。例えば、DM-workerはDM-masterからのみアクセスでき、他の訪問者は正当な証明書を持っていてもブロックされます。

コンポーネント呼び出し元の確認のために、証明書のユーザーのアイデンティティに`Common Name` (CN) を付けて証明書を生成し、呼び出し先に対して`Common Name`リストを構成して呼び出し元のアイデンティティをチェックする必要があります。

- DM-master

    構成ファイルまたはコマンドライン引数で構成:

    ```toml
    cert-allowed-cn = ["dm"]
    ```

- DM-worker

    構成ファイルまたはコマンドライン引数で構成:

    ```toml
    cert-allowed-cn = ["dm"]
    ```

### 証明書のリロード

証明書と鍵をリロードするには、DM-master、DM-worker、およびdmctlは新しい接続が作成されるたびに現在の証明書とキーファイルを再読み込みします。

`ssl-ca`、`ssl-cert`、または`ssl-key`で指定されたファイルが更新されると、DMコンポーネントを再起動して証明書とキーファイルを再読み込みし、お互いに再接続する必要があります。

## DMコンポーネントと上流または下流データベース間のデータ送信の暗号化を有効にする

このセクションでは、DMコンポーネントと上流または下流データベース間のデータ送信の暗号化を有効にする方法について紹介します。

### 上流データベースの暗号化されたデータ送信を有効にする

1. 上流データベースを構成し、暗号化サポートを有効にし、サーバー証明書を設定します。詳細な操作については、[暗号化接続の使用](https://dev.mysql.com/doc/refman/8.0/en/using-encrypted-connections.html)を参照してください。

2. ソース構成ファイルにMySQLクライアント証明書を設定:

    > **注意：**
    >
    > DM-masterとDM-workerのすべてのコンポーネントが、指定されたパス経由で証明書とキーファイルを読み取れることを確認してください。

    ```yaml
    from:
        security:
            ssl-ca: "/path/to/mysql-ca.pem"
            ssl-cert: "/path/to/mysql-cert.pem"
            ssl-key: "/path/to/mysql-key.pem"
    ```

### 下流のTiDBの暗号化されたデータ送信を有効にする

1. 下流のTiDBを暗号化接続を使用するように構成します。詳細な操作については、[TiDBサーバーを安全な接続に使用するように構成](/enable-tls-between-clients-and-servers.md#configure-tidb-server-to-use-secure-connections)を参照してください。

2. タスク構成ファイルにTiDBクライアント証明書を設定:

    > **注意：**
    >
    > DM-masterとDM-workerのすべてのコンポーネントが、指定されたパス経由で証明書とキーファイルを読み取れることを確認してください。

    ```yaml
    target-database:
        security:
            ssl-ca: "/path/to/tidb-ca.pem"
            ssl-cert: "/path/to/tidb-client-cert.pem"
            ssl-key: "/path/to/tidb-client-key.pem"
    ```