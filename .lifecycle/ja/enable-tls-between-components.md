---
title: TiDBコンポーネント間のTLSを有効にする
summary: TiDBコンポーネント間のTLS認証を有効にする方法について学びます。
aliases: ['/docs/dev/enable-tls-between-components/', '/docs/dev/how-to/secure/enable-tls-between-components/']
---

# TiDBコンポーネント間のTLSを有効にする

このドキュメントでは、TiDBクラスタ内のコンポーネント間の暗号化データ送信を有効にする方法について説明します。有効になると、次のコンポーネント間での暗号化送信が使用されます。

- TiDB、TiKV、PD、およびTiFlash間の通信
- TiDBコントロールとTiDB；TiKVコントロールとTiKV；PDコントロールとPD間の通信
- 各TiDB、TiKV、PD、およびTiFlashクラスタ内部の通信

現時点では、特定のコンポーネントのみで暗号化送信を有効にすることはサポートされていません。

## 暗号化データ送信の設定と有効化

1. 証明書を準備します。

    TiDB、TiKV、およびPDそれぞれにサーバー証明書を準備することを推奨します。これらのコンポーネントがお互いを認証できるようにしてください。TiDB、TiKV、およびPDのコントロールツールは1つのクライアント証明書を共有することを選択できます。

    `openssl`、`easy-rsa`、`cfssl`などのツールを使用して、自己署名証明書を生成できます。

    <CustomContent platform="tidb">

    `openssl`を選択する場合は、[自己署名証明書の生成](/generate-self-signed-certificates.md)を参照できます。

    </CustomContent>

    <CustomContent platform="tidb-cloud">

    `openssl`を選択する場合は、[自己署名証明書の生成](https://docs.pingcap.com/tidb/stable/generate-self-signed-certificates)を参照できます。

    </CustomContent>

2. 証明書を設定します。

    TiDBコンポーネント間で相互認証を有効にするには、TiDB、TiKV、およびPDの証明書を次のように設定します。

    - TiDB

        次の構成ファイルまたはコマンドライン引数で設定します：

        ```toml
        [security]
        # クラスタコンポーネントとの接続に使用されるSSL CAのリストを含むファイルのパス。
        cluster-ssl-ca = "/path/to/ca.pem"
        # PEM形式のX509証明書を含むファイルのパス。
        cluster-ssl-cert = "/path/to/tidb-server.pem"
        # PEM形式のX509キーを含むファイルのパス。
        cluster-ssl-key = "/path/to/tidb-server-key.pem"
        ```

    - TiKV

        次の構成ファイルまたはコマンドライン引数で設定し、対応するURLを`https`に設定します：

        ```toml
        [security]
        ## 証明書のパス。空の文字列はセキュア接続が無効であることを意味します。
        # 信頼されるSSL CAのリストを含むファイルのパス。設定する場合は、以下の設定`cert_path`および`key_path`も必要です。
        ca-path = "/path/to/ca.pem"
        # PEM形式のX509証明書を含むファイルのパス。
        cert-path = "/path/to/tikv-server.pem"
        # PEM形式のX509キーを含むファイルのパス。
        key-path = "/path/to/tikv-server-key.pem"
        ```

    - PD

        次の構成ファイルまたはコマンドライン引数で設定し、対応するURLを`https`に設定します：

        ```toml
        [security]
        ## 証明書のパス。空の文字列はセキュア接続が無効であることを意味します。
        # 信頼されるSSL CAのリストを含むファイルのパス。設定する場合は、以下の設定`cert_path`および`key_path`も必要です。
        cacert-path = "/path/to/ca.pem"
        # PEM形式のX509証明書を含むファイルのパス。
        cert-path = "/path/to/pd-server.pem"
        # PEM形式のX509キーを含むファイルのパス。
        key-path = "/path/to/pd-server-key.pem"
        ```

    - TiFlash（v4.0.5で新規）

        `tiflash.toml`ファイルで設定し、`http_port`項目を`https_port`に変更します：

        ```toml
        [security]
        ## 証明書のパス。空の文字列はセキュア接続が無効であることを意味します。
        # 信頼されるSSL CAのリストを含むファイルのパス。設定する場合は、以下の設定`cert_path`および`key_path`も必要です。
        ca_path = "/path/to/ca.pem"
        # PEM形式のX509証明書を含むファイルのパス。
        cert_path = "/path/to/tiflash-server.pem"
        # PEM形式のX509キーを含むファイルのパス。
        key_path = "/path/to/tiflash-server-key.pem"
        ```

        `tiflash-learner.toml`ファイルで設定します：

        ```toml
        [security]
        # 信頼されるSSL CAのリストを含むファイルのパス。設定する場合は、以下の設定`cert_path`および`key_path`も必要です。
        ca-path = "/path/to/ca.pem"
        # PEM形式のX509証明書を含むファイルのパス。
        cert-path = "/path/to/tiflash-server.pem"
        # PEM形式のX509キーを含むファイルのパス。
        key-path = "/path/to/tiflash-server-key.pem"
        ```

    - TiCDC

        構成ファイルで設定します：

        ```toml
        [security]
        ca-path = "/path/to/ca.pem"
        cert-path = "/path/to/cdc-server.pem"
        key-path = "/path/to/cdc-server-key.pem"
        ```

        または、コマンドライン引数で設定し、対応するURLを`https`に設定します：

        {{< copyable "shell-regular" >}}

        ```bash
        cdc server --pd=https://127.0.0.1:2379 --log-file=ticdc.log --addr=0.0.0.0:8301 --advertise-addr=127.0.0.1:8301 --ca=/path/to/ca.pem --cert=/path/to/ticdc-cert.pem --key=/path/to/ticdc-key.pem
        ```

        これで、TiDBコンポーネント間での暗号化送信が有効になります。

    > **注記:**
    >
    > TiDBクラスタで暗号化送信を有効にした後、tidb-ctl、tikv-ctl、またはpd-ctlを使用してクラスタに接続する場合は、クライアント証明書を指定する必要があります。例：

    {{< copyable "shell-regular" >}}

    ```bash
    ./tidb-ctl -u https://127.0.0.1:10080 --ca /path/to/ca.pem --ssl-cert /path/to/client.pem --ssl-key /path/to/client-key.pem
    ```

    {{< copyable "shell-regular" >}}

    ```bash
    tiup ctl:v<CLUSTER_VERSION> pd -u https://127.0.0.1:2379 --cacert /path/to/ca.pem --cert /path/to/client.pem --key /path/to/client-key.pem
    ```

    {{< copyable "shell-regular" >}}

    ```bash
    ./tikv-ctl --host="127.0.0.1:20160" --ca-path="/path/to/ca.pem" --cert-path="/path/to/client.pem" --key-path="/path/to/clinet-key.pem"
    ```

### コンポーネント呼び出し元の識別情報を検証する

一般的に、Calleは、呼び出し元の識別情報を検証する必要があります。つまり、呼び出し元が提供するキー、証明書、およびCAを検証するだけでなく、呼び出し元の識別情報も検証する必要があります。例えば、TiKVは正当な証明書を持っていても、TiDBからのアクセスしか許可されず、他の訪問者はブロックされます。

コンポーネント呼び出し元の識別情報を検証するには、証明書のユーザー識別情報に`Common Name`をマークし、Calleのために構成ファイル内で`Common Name`リストを設定して呼び出し元の識別情報をチェックする必要があります。

- TiDB

    次の構成ファイルまたはコマンドライン引数で設定します：

    ```toml
    [security]
    cluster-verify-cn = [
        "TiDB-Server",
        "TiKV-Control",
    ]
    ```

- TiKV

    次の構成ファイルまたはコマンドライン引数で設定します：

    ```toml
    [security]
    cert-allowed-cn = [
        "TiDB-Server", "PD-Server", "TiKV-Control", "RawKvClient1",
    ]
    ```

- PD

    次の構成ファイルまたはコマンドライン引数で設定します：

    ```toml
    [security]
    cert-allowed-cn = ["TiKV-Server", "TiDB-Server", "PD-Control"]
    ```

- TiFlash（v4.0.5で新規）

    `tiflash.toml`ファイルまたはコマンドライン引数で設定します：

    ```toml
    [security]
    cert_allowed_cn = ["TiKV-Server", "TiDB-Server"]
    ```

    `tiflash-learner.toml`ファイルで設定します：

    ```toml
    [security]
    cert-allowed-cn = ["PD-Server", "TiKV-Server", "TiFlash-Server"]
    ```

## 証明書をリロードします
- ローカルのデータセンターにTiDBクラスターが展開されている場合、TiDB、PD、TiKV、TiFlash、TiCDC、およびあらゆる種類のクライアントは、TiDBクラスターを再起動することなく、新しい接続が作成されるたびに現在の証明書と鍵ファイルを再読み込みします。

- TiDBクラスターが独自の管理クラウドに展開されている場合、TLS証明書の発行はクラウドプロバイダーの証明書管理サービスと統合されていることを確認してください。TiDB、PD、TiKV、TiFlash、およびTiCDCコンポーネントのTLS証明書は、TiDBクラスターを再起動することなく自動的にローテーションできます。

## 証明書の有効期限

TiDBクラスターの各コンポーネントのTLS証明書の有効期間をカスタマイズできます。たとえば、OpenSSLを使用してTLS証明書を発行および生成する場合、**days**パラメータを使用して有効期間を設定できます。詳細については、[自己署名証明書の生成](/generate-self-signed-certificates.md)を参照してください。

## 関連項目

- [TiDBクライアントとサーバー間のTLSを有効にする](/enable-tls-between-clients-and-servers.md)