---
title: tiup クラスタのスケールイン
---

# tiup クラスタのスケールイン

`tiup cluster scale-in` コマンドは、クラスタをスケールインするために使用されます。これにより、指定されたノードのサービスがオフラインになり、指定されたノードがクラスタから削除され、それらのノードから残されたファイルが削除されます。

## コンポーネントのオフライン操作の処理

TiKV、TiFlash、TiDB Binlog コンポーネントは非同期でオフラインになります(このために TiUP がまず API を介してノードを削除する必要があり)、停止プロセスには長い時間がかかります(このために TiUP がノードが正常にオフラインになったかどうかを継続的にチェックする必要があります)。そのため、TiKV、TiFlash、TiDB Binlog コンポーネントは以下のように特に扱われます。

- TiKV、TiFlash、TiDB Binlog コンポーネントの場合:

    1. TiUP クラスタは API を介してノードをオフラインにし、プロセスの完了を待たずに直ちに終了します。
    2. スケールインされているノードのステータスを確認するには、`tiup cluster display` コマンドを実行し、ステータスが `Tombstone` になるまで待機する必要があります。
    3. `Tombstone` ステータスのノードをクリーンアップするには、`tiup cluster prune` コマンドを実行する必要があります。`tiup cluster prune` コマンドは以下の操作を実行します:

        - オフラインになったノードのサービスを停止します。
        - オフラインになったノードのデータファイルをクリーンアップします。
        - クラスタのトポロジを更新し、オフラインになったノードを削除します。

その他のコンポーネントの場合:

- PD コンポーネントをオフラインにする場合、TiUP クラスタは急いで指定されたノードを API を介してクラスタから削除し、指定された PD ノードのサービスを停止し、その後ノードから関連するデータファイルを削除します。
- その他のコンポーネントをダウンさせる場合、TiUP クラスタは直ちにノードのサービスを停止し、指定されたノードから関連するデータファイルを削除します。

## 構文

```shell
tiup cluster scale-in <cluster-name> [flags]
```

`<cluster-name>` はスケールインするクラスタの名前です。クラスタ名を忘れた場合は、[`tiup cluster list`](/tiup/tiup-component-cluster-list.md) コマンドを使用して確認できます。

## オプション

### -N、--node

- オフラインにするノードを指定します。複数のノードはカンマで区切ります。
- データタイプ: `STRING`
- デフォルト値はありません。このオプションは必須であり、値は null であってはいけません。

### --force

- 指定されたノードをクラスタから強制的に削除するかどうかを制御します。場合によっては、オフラインにするノードのホストがダウンしており、そのノードに SSH 経由で操作することができないため、`--force` オプションを使用してノードをクラスタから強制的に削除することができます。
- データタイプ: `BOOLEAN`
- このオプションはデフォルトで無効になっており、値が `false` です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true` の値を渡すか、値を渡さないようにします。

> **警告:**
>
> このオプションを使用してサービス中または未完了の TiKV または TiFlash ノードを強制的に削除すると、これらのノードが即座に削除され、データの移行を待つことなく削除されます。これは非常に高いデータ損失リスクをもたらします。メタデータが配置されているリージョンでデータ損失が発生した場合、クラスタ全体が利用できなくなり、回復不可能になります。

### --transfer-timeout

- PD または TiKV ノードを削除する際には、最初にそのノードのリージョンリーダを別のノードに移動します。移動プロセスにはしばらく時間がかかるため、`--transfer-timeout` を構成して最大待機時間(秒単位)を設定できます。タイムアウト後、`tiup cluster scale-in` コマンドは待機をスキップしてスケーリングインを直接開始します。
- データタイプ: `UINT`
- このオプションはデフォルトで有効になっており、渡される `600` 秒(デフォルト値)が設定されています。

> **注意:**
>
> PD または TiKV ノードをリーダの移動が完了するのを待たずにオフラインに取ると、サービスパフォーマンスが不安定になる可能性があります。

### -h、--help

- ヘルプ情報を出力します。
- データタイプ: `BOOLEAN`
- このオプションはデフォルトで無効になっており、値が `false` です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true` の値を渡すか、値を渡さないようにします。

## 出力

スケールインプロセスのログを表示します。

[<< 前のページに戻る - TiUP クラスタ コマンドリスト](/tiup/tiup-component-cluster.md#command-list)