```yaml
---
title: tiupクラスターパッチ
---

# tiupクラスターパッチ

クラスターが実行中の状態でサービスのバイナリを動的に置き換えたい（つまり、置換プロセス中にクラスターを利用可能な状態に保ちたい）場合は、`tiup cluster patch`コマンドを使用できます。コマンド実行後、TiUPは以下のことを行います。

- 置換するバイナリパッケージをターゲットマシンにアップロードします。
- ターゲットサービスがTiKV、TiFlash、またはTiDB Binlogなどのストレージサービスである場合、TiUPは最初に関連ノードをAPI経由でオフラインにします。
- ターゲットサービスを停止します。
- バイナリパッケージを展開し、サービスを置き換えます。
- ターゲットサービスを起動します。

## 構文

```shell
tiup cluster patch <クラスター名> <パッケージパス> [フラグ]
```

- `<クラスター名>`：操作されるクラスターの名前。
- `<パッケージパス>`：置換に使用されるバイナリパッケージへのパス。

### 準備

`tiup cluster patch`コマンドを実行する前に、必要なバイナリパッケージをパックする必要があります。次の手順を実行してください。

1. 次の変数を決定します：

    - `${component}`：置換されるコンポーネントの名前（`tidb`、`tikv`、または`pd`など）。
    - `${version}`：コンポーネントのバージョン（`v7.4.0`または`v6.5.3`など）。
    - `${os}`：オペレーティングシステム（`linux`）。
    - `${arch}`：コンポーネントが実行されるプラットフォーム（`amd64`、`arm64`）。

2. 次のコマンドを使用して現在のコンポーネントパッケージをダウンロードします：

    ```shell
    wget https://tiup-mirrors.pingcap.com/${component}-${version}-${os}-${arch}.tar.gz -O /tmp/${component}-${version}-${os}-${arch}.tar.gz
    ```

3. 一時ディレクトリを作成し、ファイルをパックするためにそれに移動します：

    ```shell
    mkdir -p /tmp/package && cd /tmp/package
    ```

4. 元のバイナリパッケージを展開します：

    ```shell
    tar xf /tmp/${component}-${version}-${os}-${arch}.tar.gz
    ```

5. 一時ディレクトリ内のファイル構造を確認します：

    ```shell
    find .
    ```

6. 一時ディレクトリ内の対応する場所にバイナリファイルまたは構成ファイルをコピーします。
7. 一時ディレクトリ内のすべてのファイルをパックします：

    ```shell
    tar czf /tmp/${component}-hotfix-${os}-${arch}.tar.gz *
    ```

上記の手順を完了したら、`/tmp/${component}-hotfix-${os}-${arch}.tar.gz`を`tiup cluster patch`コマンドの`<パッケージパス>`として使用できます。

## オプション

### --上書き

- 特定のコンポーネント（TiDBまたはTiKVなど）をパッチ適用した後、tiup clusterがコンポーネントをスケールアウトさせるとき、デフォルトでTiUPは元のコンポーネントバージョンを使用します。将来、クラスターがスケールアウトする際にパッチ適用したバージョンを使用するには、コマンドでオプション `--overwrite` を指定する必要があります。
- データ型：`BOOLEAN`
- このオプションはデフォルトで無効であり、`false`の値です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

### --transfer-timeout

- PDまたはTiKVサービスを再起動する際、最初にTiKV/PDは再起動するノードのリーダーを他のノードに転送します。転送プロセスには時間がかかるため、オプション`--transfer-timeout`を使用して最大待機時間（秒単位）を設定できます。タイムアウト後、TiUPはサービスを直接再起動します。
- データ型：`UINT`
- このオプションが指定されていない場合、TiUPは`600`秒待機後に直接サービスを再起動します。

> **注意：**
>
> タイムアウト後にTiUPがサービスを直接再起動すると、サービスのパフォーマンスに乱れが生じる可能性があります。

### -N、--node

- 置換するノードを指定します。このオプションの値は、ノードIDのコンマ区切りリストです。`tiup cluster display`コマンドが返す[クラスターステータステーブル](/tiup/tiup-component-cluster-display.md)の最初の列からノードIDを取得できます。
- データ型：`STRINGS`
- このオプションが指定されていない場合、TiUPはデフォルトで置換するノードを選択しません。

> **注意：**
>
> オプション`-R、--role`を同時に指定した場合、TiUPは`-N、--node`と`-R、--role`の両方の要件に一致するサービスノードを置換します。

### -R、--role

- 置換するロールを指定します。このオプションの値は、ノードの役割のコンマ区切りリストです。`tiup cluster display`コマンドが返す[クラスターステータステーブル](/tiup/tiup-component-cluster-display.md)の2番目の列からノードに展開されたロールを取得できます。
- データ型：`STRINGS`
- このオプションが指定されていない場合、TiUPはデフォルトで置換するロールを選択しません。

> **注意：**
>
> オプション`-N、--node`を同時に指定した場合、TiUPは`-N、--node`と`-R、--role`の両方の要件に一致するサービスノードを置換します。

### --offline

- 現在のクラスターが稼働していないことを宣言します。このオプションを指定すると、TiUPはサービスリーダーを他のノードに移動したりサービスを再起動することなく、クラスターコンポーネントのバイナリファイルを置き換えるのみです。
- データ型：`BOOLEAN`
- このオプションはデフォルトで無効であり、`false`の値です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

### -h、--help

- ヘルプ情報を出力します。
- データ型：`BOOLEAN`
- このオプションはデフォルトで無効であり、`false`の値です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

## 出力

tiup-clusterの実行ログ。

[<<前のページに戻る - TiUPクラスターコマンドリスト](/tiup/tiup-component-cluster.md#command-list)
```