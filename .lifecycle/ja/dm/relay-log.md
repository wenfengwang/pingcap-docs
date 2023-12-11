---
title: データ移行リレーログ
summary: ディレクトリ構造、初回移行ルール、DMリレーログのデータ削除方法を学びます。
aliases: ['/docs/tidb-data-migration/dev/relay-log/']
---

# データ移行リレーログ

データ移行（DM）リレーログは、データベース変更を記述する複数の番号付きファイルセットと、使用されるすべてのリレーログファイルの名前を含むインデックスファイルから構成されます。

リレーログを有効にした後、DMワーカーは自動的にアップストリームのbinlogをローカルの構成ディレクトリに移行します（DMがTiUPを使用して展開される場合、デフォルトの移行ディレクトリは `<deploy_dir>/<relay_log>` です）。 `<relay_log>` のデフォルト値は `relay-dir` であり、[アップストリームデータベース構成ファイル](/dm/dm-source-configuration-file.md) で変更できます。v5.4.0以降、[DMワーカーコンフィグレーションファイル](/dm/dm-worker-configuration-file.md) で `relay-dir` を使用してローカル構成ディレクトリを構成でき、これはアップストリームデータベースのコンフィグレーションファイルより優先されます。

## ユーザーシナリオ

MySQLでは、ストレージスペースが限られているため、最大保持時間に達するとbinlogは自動的にパージされます。アップストリームデータベースがbinlogをパージした後、DMはパージされたbinlogを取得できず、移行タスクが失敗します。各移行タスクについて、DMはbinlogを取得するためにアップストリームに接続を作成します。多くの接続は、アップストリームデータベースに負荷をかける可能性があります。

リレーログを有効にすると、同じアップストリームデータベースを持つ複数の移行タスクは、ローカルディスクに取り込まれたリレーログを再利用できます。これにより、**アップストリームデータベースにかかる負荷が軽減**されます。

完全および増分データ移行タスク（`task-mode=all`）の場合、DMはまず完全データを移行し、その後binlogに基づいて増分移行を行う必要があります。完全移行段階が長引くと、アップストリームのbinlogがパージされる可能性があり、結果的に増分移行が失敗することがあります。この状況を避けるために、リレーログ機能を有効にしておくことで、DMは自動的にローカルディスクに十分なログを保持し、**増分移行タスクを正常に行えるようにします**。

通常、リレーログを有効にすることが推奨されていますが、以下の潜在的な問題に注意してください。

リレーログはディスクに書き込む必要があるため、外部IOとCPUリソースを消費します。これにより、全体のデータレプリケーションプロセスが遅延し、データレプリケーションの遅延が増加します。**遅延に敏感な**シナリオでは、リレーログを有効にすることをお勧めしません。

> **注意:**
> 
> DM v2.0.7以降のバージョンでは、リレーログの書き込みが最適化されています。遅延とCPUリソースの消費は比較的低いです。

## リレーログの利用

このセクションでは、リレーログを有効および無効にする方法、リレーログのステータスをクエリする方法、リレーログをクリーンアップする方法について説明します。

### リレーログの有効および無効

<SimpleTab>

<div label="v5.4.0およびそれ以降のバージョン">

v5.4.0およびそれ以降のバージョンでは、 `enable-relay` を `true` に設定することでリレーログを有効にできます。v5.4.0以降、アップストリームデータソースをバインドする際、DMワーカーはデータソースの構成で `enable-relay` 項目を確認します。 `enable-relay` が `true` の場合、このデータソースでリレーログ機能が有効になります。

詳細な構成方法については、[アップストリームデータベース構成ファイル](/dm/dm-source-configuration-file.md) を参照してください。

また、 `start-relay` または `stop-relay` コマンドを使用して、データソースの `enable-relay` 構成を動的に調整したり、リレーログを時に有効または無効にできます。

{{< copyable "shell-regular" >}}

```bash
start-relay -s mysql-replica-01
```

```
{
    "result": true,
    "msg": ""
}
```

</div>

<div label="v2.0.2（含む）以上およびv5.3.0（含む）未満のバージョン">

> **注意:**
> 
> DM v2.0.2以降のバージョンおよびv5.3.0では、ソース構成ファイルの `enable-relay` 項目は無効になり、リレーログを有効または無効にするには、 `start-relay` および `stop-relay` を使用する必要があります。DMがデータソース構成を[ロード](/dm/dm-manage-source.md#operate-data-source)する際に `enable-relay` が `true` に設定されていると、次のメッセージが出力されます：

> ```
> Please use `start-relay` to specify which workers should pull relay log of relay-enabled sources.
> ```

> **警告:**
> 
> このスタートアップ手法はv6.1で非推奨とマークされ、将来のリリースで削除される可能性があります。関連コマンドの出力に次のプロンプトが表示されます： `start-relay/stop-relay with worker name will be deprecated soon. You can try stopping relay first and use start-relay without worker name instead`。

`start-relay`コマンドでは、指定したデータソースのリレーログをマイグレーションするための1つまたは複数のDMワーカーを構成できますが、パラメータで指定されたDMワーカーは無料であるか、またはすでにアップストリームデータソースにバインドされている必要があります。次の例を参照してください。

{{< copyable "" >}}

```bash
start-relay -s mysql-replica-01 worker1 worker2
```

```
{
    "result": true,
    "msg": ""
}
```

{{< copyable "" >}}

```bash
stop-relay -s mysql-replica-01 worker1 worker2
```

```
{
    "result": true,
    "msg": ""
}
```

</div>

<div label="v2.0.2未満のバージョン">

v2.0.2より前のDMバージョン（v2.0.2を含まない）では、DMはアップストリームデータソースのDMワーカーにバインドする際に、ソース構成ファイルの `enable-relay` 項目を確認します。 `enable-relay` が `true` に設定されている場合、DMはそのデータソースのリレーログ機能を有効にします。

`enable-relay` 項目の設定方法については、[アップストリームデータベース構成ファイル](/dm/dm-source-configuration-file.md) を参照してください。

</div>
</SimpleTab>

### リレーログステータスのクエリ

`query-status -s`コマンドを使用してリレーログのステータスをクエリできます：

```bash
query-status -s mysql-replica-01
```

<details>
<summary>期待される出力</summary>

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "no sub task started",
            "sourceStatus": {
                "source": "mysql-replica-01",
                "worker": "worker2",
                "result": null,
                "relayStatus": {
                    "masterBinlog": "(mysql-bin.000005, 916)",
                    "masterBinlogGtid": "09bec856-ba95-11ea-850a-58f2b4af5188:1-28",
                    "relaySubDir": "09bec856-ba95-11ea-850a-58f2b4af5188.000001",
                    "relayBinlog": "(mysql-bin.000005, 4)",
                    "relayBinlogGtid": "09bec856-ba95-11ea-850a-58f2b4af5188:1-28",
                    "relayCatchUpMaster": false,
                    "stage": "Running",
                    "result": null
                }
            },
            "subTaskStatus": [
            ]
        },
        {
            "result": true,
            "msg": "no sub task started",
            "sourceStatus": {
                "source": "mysql-replica-01",
                "worker": "worker1",
                "result": null,
                "relayStatus": {
                    "masterBinlog": "(mysql-bin.000005, 916)",
                    "masterBinlogGtid": "09bec856-ba95-11ea-850a-58f2b4af5188:1-28",
                    "relaySubDir": "09bec856-ba95-11ea-850a-58f2b4af5188.000001",
                    "relayBinlog": "(mysql-bin.000005, 916)",
                    "relayBinlogGtid": "",
                    "relayCatchUpMaster": true,
                    "stage": "Running",
                    "result": null
                }
            },
            "subTaskStatus": [
            ]
        }
    ]
}
```

</details>

### リレーログの一時停止と再開

`pause-relay`コマンドを使用してリレーログの取得プロセスを一時停止し、`resume-relay`コマンドを使用してプロセスを再開できます。これらの2つのコマンドを実行する際には、実行時にアップストリームデータソースの `source-id` を指定する必要があります。次の例を参照してください：

```bash
pause-relay -s mysql-replica-01 -s mysql-replica-02
```

<details>
<summary>期待される出力</summary>

```
{
    "op": "PauseRelay",
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "worker1"
        },
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-02",
            "worker": "worker2"
        }
    ]
}
```

</details>

```
```
resume-relay -s mysql-replica-01
```

<details>
<summary>出力結果</summary>

```
{
    "op": "ResumeRelay",
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "worker1"
        }
    ]
}
```

</details>

### リレーログのパージ

DMには、リレーログをパージするための2つの方法が用意されています。手動パージと自動パージの両方とも、アクティブなリレーログはパージされません。

> **注記:**
>
> - アクティブなリレーログ: リレーログがデータ移行タスクによって使用されています。アクティブなリレーログは現在、更新および書き込みのみがSyncerユニットで行われています。All モードでのデータ移行タスクがフルエクスポート/インポートに時間を費やす場合、この時間がデータソースのパージ構成の設定された有効期限を超えた場合、リレーログはパージされません。
>
> - 有効期限切れのリレーログ: リレーログファイルの最終更新時刻と現在時刻の差が、設定ファイルの`expires` フィールドの値よりも大きい場合。

#### 自動パージ

自動パージを有効にし、ソース設定ファイルでその戦略を構成することができます。以下に示す例を参照してください。

```yaml
# リレーログパージ戦略
purge:
    interval: 3600
    expires: 24
    remain-space: 15
```

+ `purge.interval`
    - バックグラウンドでの自動パージのインターバル（秒単位）。
    - デフォルトは "3600" で、バックグラウンドでのパージタスクは 3600 秒ごとに実行されます。

+ `purge.expires`
    - パージされるまでの時間の長さ（リレーログは以前にリレープロセッシングユニットに書き込まれたものであり、現在実行中のデータ移行タスクによって使用されていない、または後で読み取られることはないリレーログ）。
    - デフォルトは "0" で、リレーログは更新時刻に応じてパージされません。

+ `purge.remain-space`
    - DM-worker マシンで安全にパージできるリレーログをパージしようとする残りのディスク容量（単位: GB）。
    - デフォルトは "15" で、利用可能なディスク容量が 15GB 未満の場合、DM-master はリレーログを安全にパージしようとします。

#### 手動パージ

手動パージとは、`dmctl` が提供する `purge-relay` コマンドを使用して、`subdir` と binlog 名を指定して、指定された binlog の**前の**すべてのリレーログをパージすることを意味します。`-subdir` オプションがコマンドで指定されていない場合、現在のリレーログサブディレクトリの**前の**すべてのリレーログがパージされます。

現在のリレーログのディレクトリ構造が次のような場合を想定しています:

```
$ tree .
.
|-- deb76a2b-09cc-11e9-9129-5242cf3bb246.000001
|   |-- mysql-bin.000001
|   |-- mysql-bin.000002
|   |-- mysql-bin.000003
|   `-- relay.meta
|-- deb76a2b-09cc-11e9-9129-5242cf3bb246.000003
|   |-- mysql-bin.000001
|   `-- relay.meta
|-- e4e0e8ab-09cc-11e9-9220-82cc35207219.000002
|   |-- mysql-bin.000001
|   `-- relay.meta
`-- server-uuid.index

$ cat server-uuid.index
deb76a2b-09cc-11e9-9129-5242cf3bb246.000001
e4e0e8ab-09cc-11e9-9220-82cc35207219.000002
deb76a2b-09cc-11e9-9129-5242cf3bb246.000003
```

+ 次の `purge-relay` コマンドを `dmctl` で実行すると、**前の** `e4e0e8ab-09cc-11e9-9220-82cc35207219.000002/mysql-bin.000001` のすべてのリレーログファイル（`deb76a2b-09cc-11e9-9129-5242cf3bb246.000001` のすべてのリレーログファイル）がパージされます。

    {{< copyable "" >}}

    ```bash
    purge-relay -s mysql-replica-01 --filename mysql-bin.000001 --sub-dir e4e0e8ab-09cc-11e9-9220-82cc35207219.000002
    ```

+ 次の `purge-relay` コマンドを `dmctl` で実行すると、現在の（`deb76a2b-09cc-11e9-9129-5242cf3bb246.000003`）ディレクトリの `mysql-bin.000001` の**前の**すべてのリレーログファイル（`deb76a2b-09cc-11e9-9129-5242cf3bb246.000001` と `e4e0e8ab-09cc-11e9-9220-82cc35207219.000002` のすべてのリレーログファイル）がパージされます。

    {{< copyable "" >}}

    ```bash
    purge-relay -s mysql-replica-01 --filename mysql-bin.000001
    ```

## リレーログの内部メカニズム

このセクションでは、リレーログの内部メカニズムについて紹介します。

### ディレクトリ構造

リレーログのローカルストレージのディレクトリ構造の例:

```
<deploy_dir>/<relay_log>/
|-- 7e427cc0-091c-11e9-9e45-72b7c59d52d7.000001
|   |-- mysql-bin.000001
|   |-- mysql-bin.000002
|   |-- mysql-bin.000003
|   |-- mysql-bin.000004
|   `-- relay.meta
|-- 842965eb-091c-11e9-9e45-9a3bff03fa39.000002
|   |-- mysql-bin.000001
|   `-- relay.meta
`-- server-uuid.index
```

- `subdir`:

    - DM-worker は、上流データベースから移行された binlog を同じディレクトリにストアします。各ディレクトリは `subdir` です。

    - `subdir` は `<上流データベースUUID>.<ローカルサブディレクトリの連番>` の形式で命名されます。

    - 上流側でプライマリとセカンダリインスタンスが切り替わった後、DM-worker は新しい `subdir` ディレクトリを増分の連番で生成します。

    - 上記の例では、`7e427cc0-091c-11e9-9e45-72b7c59d52d7.000001` ディレクトリについて、`7e427cc0-091c-11e9-9e45-72b7c59d52d7` が上流データベース UUID であり、`000001` がローカルの `subdir` シリアル番号です。

- `server-uuid.index`: 現在利用可能な `subdir` ディレクトリのリストを記録します。

- `relay.meta`: 各 `subdir` で移行された binlog の情報を保持します。例:

    ```bash
    cat c0149e17-dff1-11e8-b6a8-0242ac110004.000001/relay.meta
    ```

    ```
    binlog-name = "mysql-bin.000010"                            # 現在移行された binlog の名前。
    binlog-pos = 63083620                                       # 現在移行された binlog の位置。
    binlog-gtid = "c0149e17-dff1-11e8-b6a8-0242ac110004:1-3328" # 現在移行された binlog の GTID。
    ```

    また、複数の GTID も存在する場合があります:

    ```bash
    cat 92acbd8a-c844-11e7-94a1-1866daf8accc.000001/relay.meta
    ```

    ```
    binlog-name = "mysql-bin.018393"
    binlog-pos = 277987307
```yaml
binlog-gtid = "3ccc475b-2343-11e7-be21-6c0b84d59f30:1-14,406a3f61-690d-11e7-87c5-6c92bf46f384:1-94321383,53bfca22-690d-11e7-8a62-18ded7a37b78:1-495,686e1ab6-c47e-11e7-a42c-6c92bf46f384:1-34981190,03fc0263-28c7-11e7-a653-6c0b84d59f30:1-7041423,05474d3c-28c7-11e7-8352-203db246dd3d:1-170,10b039fc-c843-11e7-8f6a-1866daf8d810:1-308290454"

### DMがbinlogを受信する位置

- DMは保存されたチェックポイント（デフォルトではダウンストリームの `dm_meta` スキーマ）から各マイグレーションタスクが必要とする最も初期の位置を取得します。この位置が以下の位置のいずれよりも後ろであれば、DMはこの位置からマイグレーションを開始します。

- もしローカルのリレーログが有効であれば、つまりリレーログに有効な `server-uuid.index`、`subdir`、`relay.meta` ファイルが含まれていれば、DM-workerは`relay.meta` に記録された位置からマイグレーションを復旧します。

- もし有効なローカルリレーログがなく、かつ上流データソースの設定ファイルが `relay-binlog-name` または `relay-binlog-gtid` を指定している場合：

    - GTID モードでは、`relay-binlog-gtid` が指定されている場合、DM-workerは指定されたGTIDからマイグレーションを開始します。 
    - GTID モードでない場合、`relay-binlog-name` が指定されている場合、DM-workerは指定されたbinlogファイルからマイグレーションを開始します。

- もし有効なローカルリレーログがなく、かつDMの設定ファイルに `relay-binlog-name` または `relay-binlog-gtid` が指定されていない場合：

    - GTID モードでない場合、各サブタスクがマイグレーションを開始する最も初期のbinlogから最新のbinlogまでをDM-workerはマイグレーションを開始します。
    - GTID モードの場合、各サブタスクがマイグレーションを開始する最も初期のGTIDから最新のGTIDまでをDM-workerはマイグレーションを開始します。

    > **注意：**
    >
    > もし上流のリレーログが削除された場合、エラーが発生します。この場合、マイグレーションの開始位置を指定するために [`relay-binlog-gtid`](/dm/dm-source-configuration-file.md#global-configuration) を構成する必要があります。
```