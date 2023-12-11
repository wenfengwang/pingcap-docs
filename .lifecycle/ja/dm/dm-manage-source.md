---
title: TiDBデータマイグレーションでのデータソース構成の管理
summary: TiDBデータマイグレーションにおける上流のMySQLインスタンスの管理方法について学んでください。
---

# TiDBデータマイグレーションでのデータソース構成の管理

このドキュメントでは、MySQLのパスワードを暗号化する方法、データソースの操作、および上流のMySQLインスタンスとDMワーカーの間のバインディングを変更する方法を紹介します。[dmctl](/dm/dmctl-introduction.md) を使用します。

## データベースのパスワードを暗号化する

DMの構成ファイルでは、dmctlで暗号化されたパスワードを使用することを推奨します。1つの元のパスワードに対して、それぞれの暗号化が異なります。

{{< copyable "shell-regular" >}}

```bash
./dmctl -encrypt 'abc!@#123'
```

```
MKxn0Qo3m3XOyjCnhEMtsUCm83EhGQDZ/T4=
```

## データソースの操作

`operate-source` コマンドを使用して、DMクラスタにデータソース構成をロード、リスト、または削除できます。

{{< copyable "" >}}

```bash
help operate-source
```

```
`create`/`stop`/`show` upstream MySQL/MariaDB source.

Usage:
  dmctl operate-source <operate-type> [config-file ...] [--print-sample-config] [flags]

Flags:
  -h, --help                  operate-sourceのヘルプ
  -p, --print-sample-config   ソースのサンプルの設定ファイルを表示

Global Flags:
  -s, --source strings   MySQLソースID
```

### フラグの説明

+ `create`: 1つ以上の上流データベースソースを作成します。複数のデータソースの作成に失敗した場合、DMはコマンドが実行されていない状態にロールバックします。

+ `stop`: 1つ以上の上流データベースソースを停止します。複数のデータソースの停止に失敗した場合、一部のデータソースが停止されることがあります。

+ `show`: 追加されたデータソースとそれに対応するDMワーカーを表示します。

+ `config-file`: `source.yaml` のファイルパスを指定し、複数のファイルパスを渡すことができます。

+ `--print-sample-config`: サンプルの設定ファイルを表示します。このパラメータは他のパラメータを無視します。

### 使用例

次の `operate-source` コマンドを使用して、ソース構成ファイルを作成します。

{{< copyable "" >}}

```bash
operate-source create ./source.yaml
```

`source.yaml` の構成については、[上流データベース構成ファイルの紹介](/dm/dm-source-configuration-file.md) を参照してください。

以下は返される結果の例です。

{{< copyable "" >}}

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "dm-worker-1"
        }
    ]
}
```

### データソース構成の確認

> **注意:**
>
> `config` コマンドは、DM v6.0 以降のバージョンでのみサポートされています。以前のバージョンでは、`get-config` コマンドを使用する必要があります。

`source-id` を知っている場合は、`dmctl --master-addr <master-addr> config source <source-id>` を実行してデータソースの構成を取得できます。

{{< copyable "" >}}

```bash
config source mysql-replica-01
```

```
{
  "result": true,
    "msg": "",
    "cfg": "enable-gtid: false
      flavor: mysql
      source-id: mysql-replica-01
      from:
        host: 127.0.0.1
        port: 8407
        user: root
        password: '******'
}
```

`source-id` を知らない場合は、まずすべてのデータソースをリストアップするために `dmctl --master-addr <master-addr> operate-source show` を実行できます。

{{< copyable "" >}}

```bash
operate-source show
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "source is added but there is no free worker to bound",
            "source": "mysql-replica-02",
            "worker": ""
        },
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "dm-worker-1"
        }
    ]
}
```

## 上流のMySQLインスタンスとDMワーカーのバインディングの変更

`transfer-source` コマンドを使用して、上流のMySQLインスタンスとDMワーカーのバインディングを変更できます。

{{< copyable "" >}}

```bash
help transfer-source
```

```
上流のMySQL/MariaDBソースを空いているワーカーに移行します。
使用方法:
  dmctl transfer-source <source-id> <worker-id> [flags]
Flags:
  -h, --help   transfer-sourceのヘルプ
Global Flags:
  -s, --source strings   MySQLソースID
```

移行前に、DMは解除されるワーカーに実行中のタスクがあるかどうかをチェックします。ワーカーに実行中のタスクがある場合は、まず [タスクを一時停止](/dm/dm-pause-task.md) し、バインディングを変更してから [タスクを再開](/dm/dm-resume-task.md) する必要があります。

### 使用例

DMワーカーのバインディングを知らない場合は、`dmctl --master-addr <master-addr> list-member --worker` を実行して、すべてのワーカーの現在のバインディングをリストアップできます。

{{< copyable "" >}}

```bash
list-member --worker
```

```
{
    "result": true,
    "msg": "",
    "members": [
        {
            "worker": {
                "msg": "",
                "workers": [
                    {
                        "name": "dm-worker-1",
                        "addr": "127.0.0.1:8262",
                        "stage": "bound",
                        "source": "mysql-replica-01"
                    },
                    {
                        "name": "dm-worker-2",
                        "addr": "127.0.0.1:8263",
                        "stage": "free",
                        "source": ""
                    }
                ]
            }
        }
    ]
}
```

上記の例では、`mysql-replica-01` が `dm-worker-1` にバインドされています。以下のコマンドで `mysql-replica-01` のバインディングを `dm-worker-2` に移行します。

{{< copyable "" >}}

```bash
transfer-source mysql-replica-01 dm-worker-2
```

```
{
    "result": true,
    "msg": ""
}
```

コマンドが効果を持っているかどうかを確認するために、`dmctl --master-addr <master-addr> list-member --worker` を実行してください。

{{< copyable "" >}}

```bash
list-member --worker
```

```
{
    "result": true,
    "msg": "",
    "members": [
        {
            "worker": {
                "msg": "",
                "workers": [
                    {
                        "name": "dm-worker-1",
                        "addr": "127.0.0.1:8262",
                        "stage": "free",
                        "source": ""
                    },
                    {
                        "name": "dm-worker-2",
                        "addr": "127.0.0.1:8263",
                        "stage": "bound",
                        "source": "mysql-replica-01"
                    }
                ]
            }
        }
    ]
}
```