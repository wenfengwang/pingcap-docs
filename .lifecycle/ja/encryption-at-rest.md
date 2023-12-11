---
title: データの安全な保管
summary: 機密データを保護するためにデータの安全な保管を有効にする方法を学びます。
aliases: ['/docs/dev/encryption at rest/']
---

# データの安全な保管

> **注意:**
>
> クラスターがAWS上に展開されておりEBSストレージを使用している場合は、EBS暗号化を使用することをお勧めします。[AWSドキュメント - EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html)を参照してください。AWSの非EBSストレージ（ローカルNVMeストレージなど）を使用している場合は、このドキュメントで紹介されているデータの安全な保管を使用することをお勧めします。

データの安全な保管とは、データを保存する際にデータが暗号化されることを意味します。データベースの場合、この機能はTDE（transparent data encryption）とも呼ばれます。これは、データの安全な保管（例えばSSDドライブ、ファイルシステム、およびクラウドベンダーなどによって行われる可能性がありますが）、TiKVがストレージ前に暗号化を行うことで、攻撃者がデータにアクセスするためにデータベースと認証する必要があることを保証するのに役立ちます。例えば、攻撃者が物理マシンにアクセスした場合、データはディスク上のファイルをコピーしてアクセスすることはできません。

## TiDBコンポーネントにおける暗号化のサポート

TiDBクラスターでは、さまざまなコンポーネントが異なる暗号化方法を使用します。このセクションでは、TiKV、TiFlash、PD、およびバックアップとリストア（BR）などの異なるTiDBコンポーネントでの暗号化のサポートについて紹介します。

TiDBクラスターが展開されると、ユーザーデータの大部分はTiKVおよびTiFlashノードに保存されます。一部のメタデータはPDノードに保存されます（例えば、TiKVリージョンの境界として使用されるセカンダリインデックスキーなど）。データの安全な保管の完全なメリットを得るには、すべてのコンポーネントで暗号化を有効にする必要があります。暗号化を実装する際には、バックアップ、ログファイル、およびネットワーク経由で送信されるデータも考慮する必要があります。

### TiKV

TiKVはデータの安全な保管をサポートしています。この機能により、TiKVは[AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)または[SM4](https://en.wikipedia.org/wiki/SM4_(cipher))を[CTR](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation)モードで使用してデータファイルを透過的に暗号化することができます。データの安全な保管を有効にするには、ユーザーによって暗号化キー（マスターキーと呼ばれます）を提供する必要があります。TiKVは実際のデータファイルを暗号化するために使用するデータキーを自動的に回転させます。マスターキーの手動ローテーションも定期的に行うことができます。なお、データの安全な保管はデータが安全な状態である（つまりディスク上にある）時のみ（ネットワーク経由でデータを転送する際には）データを暗号化します。データの安全な保管と併用してTLSを使用することをお勧めします。

必要に応じて、クラウドとセルフホスト型の展開の双方でAWS KMSを使用することができます。また、平文のマスターキーをファイルで提供することもできます。

TiKVは現在、コアダンプから暗号化キーとユーザーデータを除外していません。データの安全な保管を使用する場合は、TiKVプロセスのコアダンプを無効にすることをお勧めします。現在、TiKV自体はこれを処理していません。

TiKVは暗号化されたデータファイルをファイルの絶対パスで追跡します。そのため、TiKVノードのデータファイルのパス構成（`storage.data-dir`、`raftstore.raftdb-path`、`rocksdb.wal-dir`、および`raftdb.wal-dir`など）を変更しないようにする必要があります。

SM4暗号化は、TiKVのv6.3.0以降でのみサポートされています。v6.3.0より前のTiKVバージョンでは、AES暗号化のみがサポートされています。SM4暗号化を使用すると、スループットが50％から80％低下する可能性があります。

### TiFlash

TiFlashはデータの安全な保管をサポートしています。データキーはTiFlashによって生成されます。TiFlashに書き込まれるすべてのファイル（データファイル、スキーマファイル、一時ファイルを含む）は、現在のデータキーを使用して暗号化されます。TiFlashでサポートされている暗号化アルゴリズム、暗号化構成（TiFlashでサポートされている[`tiflash-learner.toml`ファイル](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)）、および監視メトリクスの意味は、TiKVと一致しています。

Grafanaを使用してTiFlashを展開している場合は、**TiFlash-Proxy-Details** -> **Encryption**パネルを確認することができます。

SM4暗号化は、TiFlashのv6.4.0以降でのみサポートされています。v6.4.0より前のTiFlashバージョンでは、AES暗号化のみがサポートされています。

### PD

PDの暗号化は実験的な機能であり、TiKVと同じ方法で構成されます。

### BRによるバックアップ

データをS3にバックアップする際に、BRはS3サーバーサイド暗号化（SSE）をサポートしています。S3サーバーサイド暗号化と一緒に、顧客所有のAWS KMSキーを使用することもできます。詳細については、[BR S3 server-side encryption](/encryption-at-rest.md#br-s3-server-side-encryption)を参照してください。

### ロギング

TiKV、TiDB、およびPDの情報ログにはデバッグ目的でユーザーデータが含まれている場合があります。情報ログおよびそれに含まれるデータは暗号化されていません。[ログの改ざん](/log-redaction.md)を有効にすることをお勧めします。

## TiKVのデータの安全な保管

### 概要

TiKVは現在、AES128、AES192、AES256、またはSM4をCTRモードで使用してデータを暗号化する機能をサポートしています。TiKVはエンベロープ暗号化を使用しています。その結果、TiKVが暗号化を有効にする際に2種類のキーが使用されます。

* マスターキー。マスターキーはユーザーによって提供され、TiKVが生成するデータキーを暗号化するために使用されます。マスターキーの管理はTiKVの外部で行われます。
* データキー。データキーはTiKVによって生成され、実際にデータを暗号化するために使用されるキーです。

同じマスターキーは複数のTiKVインスタンスで共有できます。本番環境でマスターキーを提供する推奨される方法は、AWS KMSを介して提供することです。AWS KMSを介してカスタマーマスターキー（CMK）を作成し、それからCMKキーIDをTiKVに構成ファイルで提供します。TiKVプロセスは実行中にKMS CMKにアクセスする必要があります。これは[IAMロール](https://aws.amazon.com/iam/)を使用して行うことができます。TiKVがKMS CMKへのアクセスを取得できない場合、起動または再起動に失敗します。[KMS](https://docs.aws.amazon.com/kms/index.html)および[IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)の使用方法についてはAWSドキュメントを参照してください。

また、カスタムキーを使用したい場合は、マスターキーをファイルで提供することもサポートされています。ファイルには、256ビット（または32バイト）のキーが16進文字列としてエンコードされており、改行（つまり、`\n`）で終わり、それ以外のものは含まれていない必要があります。ただし、ディスク上にキーを保存することでキーが漏洩するため、キーファイルは`tmpfs`のRAMにのみ保存することが適しています。

データキーは基礎ストレージエンジン（RocksDB）に渡されます。RocksDBによって書き込まれるすべてのファイル（SSTファイル、WALファイル、およびMANIFESTファイルを含む）は、現在のデータキーで暗号化されます。TiKVが使用する他のユーザーデータを含む可能性がある一時ファイルも、同じデータキーを使用して暗号化されます。データキーはデフォルトでは1週間ごとにTiKVによって自動的に回転されますが、この期間は構成可能です。キーの回転時にTiKVはすべての既存のファイルを置き換えるためのものではありませんが、RocksDBの圧縮は古いデータを新しいデータファイルに書き換えることを期待しており、最新のデータキーを使用します。TiKVは各ファイルを暗号化したキーと暗号化方式を追跡し、読み取り時にその情報を使用してコンテンツを復号化します。

データの暗号化方法に関わらず、データキーは追加の認証のためにGCMモードでAES256を使用して暗号化されます。このため、ファイルから渡す際にはマスターキーが256ビット（32バイト）である必要があります。

### キーの作成

AWSでキーを作成するには、次の手順に従います：

1. AWSコンソールの[AWS KMS](https://console.aws.amazon.com/kms)に移動します。
2. コンソールの右上隅で正しいリージョンが選択されていることを確認します。
3. 「キーの作成」をクリックし、キーの種類として「Symmetric」を選択します。
4. キーにエイリアスを設定します。

AWS CLIを使用して操作を行うこともできます：

```shell
aws --region us-west-2 kms create-key
aws --region us-west-2 kms create-alias --alias-name "alias/tidb-tde" --target-key-id 0987dcba-09fe-87dc-65ba-ab0987654321
```

2つ目のコマンドで入力する`--target-key-id`は、最初のコマンドの出力にあります。

### 暗号化の構成

暗号化を有効にするには、TiKVとPDの構成ファイルに暗号化セクションを追加できます：

```
[security.encryption]
data-encryption-method = "aes128-ctr"
data-key-rotation-period = "168h" # 7 days
```

`data-encryption-method` の可能な値には "aes128-ctr"、"aes192-ctr"、"aes256-ctr"、"sm4-ctr"（v6.3.0以降のバージョンのみ）および "plaintext" があります。デフォルト値は "plaintext" で、これは暗号化がオフになっていることを意味します。`data-key-rotation-period` は TiKV がデータキーを回転させる頻度を定義します。暗号化は新しい TiKV クラスター、または既存の TiKV クラスターの両方に対してオンにできますが、暗号化が有効になってから書き込まれたデータのみが暗号化されることが保証されます。暗号化を無効にするには、設定ファイルの `data-encryption-method` を削除するか、"plaintext" にリセットして TiKV を再起動します。暗号化方式を変更するには、設定ファイルで `data-encryption-method` を更新して TiKV を再起動します。暗号化アルゴリズムを変更するには、`data-encryption-method` をサポートされている暗号化アルゴリズムに置き換えてから TiKV を再起動します。置換後、新しいデータが書き込まれると、以前の暗号化アルゴリズムによって生成された暗号化ファイルは徐々に新しい暗号化アルゴリズムによって生成されたファイルに書き換えられます。

暗号化が有効になっている場合（つまり `data-encryption-method` が "plaintext" ではない場合）、マスターキーを指定する必要があります。AWS KMS CMK をマスターキーとして指定する場合は、`encryption.master-key` セクションを `encryption` セクションの後に追加します：

```
[security.encryption.master-key]
type = "kms"
key-id = "0987dcba-09fe-87dc-65ba-ab0987654321"
region = "us-west-2"
endpoint = "https://kms.us-west-2.amazonaws.com"
```

`key-id` は KMS CMK のキー ID を指定します。`region` は KMS CMK のAWSリージョン名です。`endpoint` はオプションで、通常は指定する必要はありませんが、AWSベンダー以外のAWS KMS互換サービスを使用している場合や [VPCエンドポイント for KMS](https://docs.aws.amazon.com/kms/latest/developerguide/kms-vpc-endpoint.html) を使用する必要がある場合に指定します。

AWSでは [マルチリージョンキー](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html) も使用できます。この場合、特定のリージョンにプライマリキーを設定し、必要なリージョンにレプリカキーを追加する必要があります。

ファイルに保存されているマスターキーを指定する場合、マスターキーコンフィギュレーションは次のようになります：

```
[security.encryption.master-key]
type = "file"
path = "/path/to/key/file"
```

ここで `path` はキーファイルへのパスです。ファイルには16バイトの16進数文字列としてエンコードされた256ビットのキーが含まれており、最後は改行文字(`\n`)で終わり、他には何も含まれていません。ファイル内容の例：

```
3b5896b5be691006e0f71c3040a29495ddcad20b14aff61806940ebd780d3c62
```

### マスターキーの回転

マスターキーを回転させるには、新しいマスターキーと古いマスターキーの両方を設定して、TiKVを再起動する必要があります。新しいマスターキーを指定するには `security.encryption.master-key` を使用し、古いマスターキーを指定するには `security.encryption.previous-master-key` を使用します。`security.encryption.previous-master-key` の設定形式は `encryption.master-key` と同じです。再起動時、TiKV は古いマスターキーと新しいマスターキーの両方にアクセスする必要がありますが、TiKV が起動して実行されると、TiKV は新しいキーにだけアクセスするようになります。それ以降、`encryption.previous-master-key` 設定を設定ファイルに残しておくことができます。また、再起動時にも、TiKV は既存のデータを新しいマスターキーを使用して復号化できない場合、古いキーを使用しようとはしません。

現在、オンラインでのマスターキーの回転はサポートされていませんので、TiKVを再起動する必要があります。稼働中のクエリを提供する実行中のTiKVクラスターをローリング再起動することをお勧めします。

以下は、KMS CMKを回転させる設定の例です：

```
[security.encryption.master-key]
type = "kms"
key-id = "50a0c603-1c6f-11e6-bb9e-3fadde80ce75"
region = "us-west-2"

[security.encryption.previous-master-key]
type = "kms"
key-id = "0987dcba-09fe-87dc-65ba-ab0987654321"
region = "us-west-2"
```

### モニタリングとデバッグ

TiKVをGrafanaと一緒に展開している場合、**TiKV-Details** ダッシュボードの **Encryption** パネルでデータの暗号化をモニタリングできます。確認すべきいくつかのメトリクスがあります：

* Encryption initialized: TiKVの起動中に暗号化が初期化された場合は1、それ以外の場合は0。マスターキーの回転後、暗号化が初期化されると、TiKVは以前のマスターキーにアクセスする必要がありません。
* Encryption data keys: 存在するデータキーの数。データキーが回転した後、この番号は1回のデータキーの回転ごとに1つ増加します。データキーの回転が期待どおりに機能しているかを監視するためにこのメトリックを使用します。
* Encrypted files: 現在存在する暗号化されたデータファイルの数。以前に暗号化されていないクラスターの暗号化を有効にするときに、この番号をデータディレクトリ内の既存のデータファイルと比較することで、暗号化されるデータの割合を推定します。
* Encryption meta file size: 暗号化メタデータファイルのサイズ。
* Read/Write encryption meta duration: 暗号化のメタデータの操作にかかる追加オーバーヘッドの時間。

デバッグのために、`tikv-ctl` コマンドを使用してデータを暗号化するために使用された暗号化方法やデータキーIDなどの暗号化メタデータをダンプできます。この操作は機密データを公開する可能性があるため、本番環境で使用することはお勧めしません。詳細については、[TiKV Control](/tikv-control.md#dump-encryption-metadata) ドキュメントを参照してください。

### TiKVバージョン間の互換性

TiKVが暗号化メタデータを処理するときのI/Oとミューテックスの競合によるオーバーヘッドを減らすために、TiKV v4.0.9 で最適化が導入され、`security.encryption.enable-file-dictionary-log` をTiKVの構成ファイルで制御できます。この構成パラメーターはTiKV v4.0.9以降でのみ有効です。

有効にすると（デフォルトでは）、暗号化メタデータのデータ形式はTiKV v4.0.8以前のバージョンでは認識できなくなります。たとえば、TiKV v4.0.9以降を使用して暗号化を行い、デフォルトの `enable-file-dictionary-log` 構成を使用しているとします。クラスターをTiKV v4.0.8以前にダウングレードする場合、TiKVは次の情報のような情報ログに類似したエラーで開始に失敗します：

```
[2020/12/07 07:26:31.106 +08:00] [ERROR] [mod.rs:110] ["encryption: failed to load file dictionary."]
[2020/12/07 07:26:33.598 +08:00] [FATAL] [lib.rs:483] ["called `Result::unwrap()` on an `Err` value: Other(\"[components/encryption/src/encrypted_file/header.rs:18]: unknown version 2\")"]
```

上記のエラーを回避するためには、まず `security.encryption.enable-file-dictionary-log` を `false` に設定し、TiKV v4.0.9以降でTiKVを起動します。TiKVが正常に起動した後は、暗号化メタデータのデータ形式が以前のTiKVバージョンで認識できる形式にダウングレードされます。この時点では、TiKVクラスターを以前のバージョンにダウングレードすることができます。

## TiFlash暗号化

### 概要

TiFlashで現在サポートされている暗号化アルゴリズムは、AES128、AES192、AES256、およびSM4（v6.4.0以降のバージョンのみ）、CTRモードでTiKVでサポートされているアルゴリズムと一致しています。TiFlashもエンベロープ暗号化を使用しています。したがって、TiFlashで暗号化が有効になっている場合、TiFlashで2つのタイプのキーが使用されます。

* マスターキー：マスターキーはユーザーによって提供され、TiFlashが生成したデータキーを暗号化するために使用されます。マスターキーの管理は、TiFlashの外部にあります。
* データキー：データキーはTiFlashによって生成され、実際にデータを暗号化するために使用されます。

同じマスターキーは複数のTiFlashインスタンスで共有されることができ、またTiFlashとTiKVの間でも共有されることができます。本番環境でマスターキーを提供する推奨される方法は、AWS KMSを介して提供することです。カスタムキーを使用したい場合、マスターキーをファイル経由で提供する方法もサポートされています。マスターキーを生成する特定の方法とマスターキーの形式はTiKVと同じです。

TiFlashはデータキーを使用してディスク上に配置されたすべてのデータ（データファイル、スキーマファイル、および計算中に生成された一時データファイルを含む）を暗号化します。データキーはデフォルトでTiFlashによって自動的に毎週回転し、その期間は設定可能です。キーが回転する際、TiFlashはすべての既存のファイルを新しいキーに置き換えるために書き換えないが、バックグラウンドでコンパクションタスクが古いデータを新しいデータファイルに書き換えることが期待されます。TiFlashは、各ファイルを暗号化したキーと暗号化方法で追跡し、それらの情報を使用して読み込み時に内容を復号化します。

### キーの作成

AWSでキーを作成する手順については、TiKVにキーを作成する手順を参照してください。

### 暗号化の設定

暗号化を有効にするには、`tiflash-learner.toml` 設定ファイルに `security.encryption` セクションを追加できます：

```
[security.encryption]
data-encryption-method = "aes128-ctr"
```yaml
server_configs:
  tiflash-learner:
    security.encryption.data-key-rotation-period: "168h" # 7 days
```

```
server_configs:
  tiflash-learner:
    security.encryption.data-encryption-method: "aes128-ctr"
    security.encryption.data-key-rotation-period: "168h" # 7 days
```
- URIに`encryption-key`を含め、AES256暗号化キーに設定します。キーに`&`や`%`などのURI予約文字が含まれる場合は、まずパーセントエンコードする必要があります。

    ```shell
    ./br backup full --pd <pd-address> --storage "azure://<bucket>/<prefix>?encryption-key=<aes256-key>"
    ```

- `AZURE_ENCRYPTION_KEY`環境変数をAES256暗号化キーに設定します。実行前に、環境変数の暗号化キーを忘れないようにするために確認してください。

    ```shell
    export AZURE_ENCRYPTION_KEY=<aes256-key>
    ./br backup full --pd <pd-address> --storage "azure://<bucket>/<prefix>"
    ```

詳細については、Azureドキュメントを参照してください：[Blob storageへのリクエストで暗号化キーを指定する](https://learn.microsoft.com/en-us/azure/storage/blobs/encryption-customer-provided-keys)。

バックアップを復元する際には、暗号化キーを指定する必要があります。例：

- `restore`コマンドに`--azblob.encryption-key`オプションを含めます：

    ```shell
    ./br restore full --pd <pd-address> --storage "azure://<bucket>/<prefix>" --azblob.encryption-key <aes256-key>
    ```

- URIに`encryption-key`を含めます：

    ```shell
    ./br restore full --pd <pd-address> --storage "azure://<bucket>/<prefix>?encryption-key=<aes256-key>"
    ```

- `AZURE_ENCRYPTION_KEY`環境変数を設定します：

    ```shell
    export AZURE_ENCRYPTION_KEY=<aes256-key>
    ./br restore full --pd <pd-address> --storage "azure://<bucket>/<prefix>"
    ```