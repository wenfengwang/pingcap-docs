---
title: binlogctl
summary: `binlogctl`の使用方法を学びます。
aliases: ['/docs/dev/tidb-binlog/binlog-control/']
---

# binlogctl

[Binlog Control](https://github.com/pingcap/tidb-binlog/tree/master/binlogctl)（略して`binlogctl`）は、TiDB Binlog向けのコマンドラインツールです。`binlogctl`を使用して、TiDB Binlogクラスタを管理できます。

`binlogctl`を使用して、次の操作を実行できます。

* PumpまたはDrainerの状態を確認する
* PumpまたはDrainerを一時停止または閉じる
* PumpまたはDrainerの異常な状態を処理する

以下は、`binlogctl`の使用シナリオです。

* データレプリケーション中にエラーが発生した場合、またはPumpまたはDrainerの実行状態を確認する必要がある場合
* クラスタのメンテナンス時にPumpまたはDrainerを一時停止または閉じる必要がある場合
* PumpまたはDrainerプロセスが異常終了し、ノードの状態が更新されていないか予期しない状態である場合。これによりデータレプリケーションタスクに影響が生じます。

## `binlogctl`のダウンロード

`binlogctl`はTiDB Toolkitに含まれています。TiDB Toolkitをダウンロードするには、[TiDBツールのダウンロード](/download-ecosystem-tools.md)を参照してください。

## 説明

コマンドラインパラメータ：

```
binlogctlの使用方法：
  -V    バージョンを表示して終了
  -cmd string
        オペレーター: "generate_meta"、"pumps"、"drainers"、"update-pump"、"update-drainer"、"pause-pump"、"pause-drainer"、"offline-pump"、"offline-drainer"、"encrypt"（デフォルトは"pumps"）
  -data-dir string
        メタディレクトリのパス（デフォルトは"binlog_position"）
  -node-id string
        ノードのID、update-pump、update-drainer、pause-pump、pause-drainer、offline-pump、offline-drainerの操作でいくつかのノードを更新するために使用
  -pd-urls string
        PDエンドポイントのコンマ区切りリスト（デフォルトは"http://127.0.0.1:2379"）
  -show-offline-nodes
        pumps/drainersを問い合わせる際にオフラインノードを含める
  -ssl-ca string
        クラスタコンポーネントへの接続に信頼されるSSL CAの一覧を含むファイルのパス
  -ssl-cert string
        クラスタコンポーネントへの接続に使用するX509証明書のPEM形式ファイルのパス
  -ssl-key string
        クラスタコンポーネントへの接続に使用するX509キーのPEM形式ファイルのパス
  -state string
        ノードの状態を設定し、オンライン、一時停止、一時停止、クローズ、またはオフラインに設定できます。
  -text string
        暗号化コマンドを使用するときに暗号化されるテキスト
  -time-zone Asia/Shanghai
        セーブポイントファイルに時刻情報を保存する場合にタイムゾーンを設定します。たとえば、CST時間の場合は、Asia/Shanghai、ローカル時間の場合は`Local`を設定します
```

コマンドの例：

- すべてのPumpまたはDrainerノードの状態を確認します。

    `cmd`を`pumps`または`drainers`に設定します。例：

    {{< copyable "shell-regular" >}}

    ```bash
    bin/binlogctl -pd-urls=http://127.0.0.1:2379 -cmd pumps
    ```

    ```
    [2019/04/28 09:29:59.016 +00:00] [INFO] [nodes.go:48] ["query node"] [type=pump] [node="{NodeID: 1.1.1.1:8250, Addr: pump:8250, State: online, MaxCommitTS: 408012403141509121, UpdateTime: 2019-04-28 09:29:57 +0000 UTC}"]
    ```

    {{< copyable "shell-regular" >}}

    ```bash
    bin/binlogctl -pd-urls=http://127.0.0.1:2379 -cmd drainers
    ```

    ```
    [2019/04/28 09:29:59.016 +00:00] [INFO] [nodes.go:48] ["query node"] [type=drainer] [node="{NodeID: 1.1.1.1:8249, Addr: 1.1.1.1:8249, State: online, MaxCommitTS: 408012403141509121, UpdateTime: 2019-04-28 09:29:57 +0000 UTC}"]
    ```

- PumpまたはDrainerを一時停止または閉じます。

    次のコマンドを使用して、サービスを一時停止または閉じることができます：

    | コマンド             | 説明           | 例                                                                                             |
    | :--------------- | :------------- | :------------------------------------------------------------------------------------------------|
    | pause-pump      | Pumpの一時停止 | `bin/binlogctl -pd-urls=http://127.0.0.1:2379 -cmd pause-pump -node-id ip-127-0-0-1:8250`       |
    | pause-drainer   | Drainerの一時停止 | `bin/binlogctl -pd-urls=http://127.0.0.1:2379 -cmd pause-drainer -node-id ip-127-0-0-1:8249`    |
    | offline-pump    | Pumpの閉じる | `bin/binlogctl -pd-urls=http://127.0.0.1:2379 -cmd offline-pump -node-id ip-127-0-0-1:8250`     |
    | offline-drainer | Drainerの閉じる | `bin/binlogctl -pd-urls=http://127.0.0.1:2379 -cmd offline-drainer -node-id ip-127-0-0-1:8249`  |

    `binlogctl`はHTTPリクエストをPumpまたはDrainerノードに送信します。リクエストを受け取った後、ノードは対応する手順を実行します。

- 異常状態のPumpまたはDrainerノードの状態を修正します。

    PumpまたはDrainerノードが正常に実行される場合、または通常のプロセスで一時停止または閉じられた場合、それは通常の状態にあります。異常な状態では、PumpまたはDrainerノードは状態を正しく保つことができません。これがデータレプリケーションタスクに影響を与えます。この場合、`binlogctl`を使用して状態情報を修復します。

    PumpまたはDrainerノードの状態を更新するには、`cmd`を`update-pump`または`update-drainer`に設定します。状態は`paused`または`offline`にすることができます。例：

    {{< copyable "shell-regular" >}}

    ```bash
    bin/binlogctl -pd-urls=http://127.0.0.1:2379 -cmd update-pump -node-id ip-127-0-0-1:8250 -state paused
    ```

    > **注意:**
    >
    > PumpまたはDrainerノードが正常に実行されている場合、それは定期的にその状態をPDに更新します。上記のコマンドは、PDに保存されているPumpまたはDrainerの状態を直接修正するため、PumpまたはDrainerノードが正常に実行されている場合にはこのコマンドを使用しないでください。詳細については、[TiDB Binlog FAQ](/tidb-binlog/tidb-binlog-faq.md)を参照してください。