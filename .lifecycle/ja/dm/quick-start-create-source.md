---
title: TiDBデータ移行のためのデータソースを作成する
summary: データ移行（DM）のためのデータソースを作成する方法について学びます。

# TiDBデータ移行のためのデータソースを作成する

> **注：**
>
> データソースを作成する前に、[TiUPを使用してDMクラスタを展開する](/dm/deploy-a-dm-cluster-using-tiup.md)必要があります。

このドキュメントでは、TiDBデータ移行（DM）のデータ移行タスクのためのデータソースを作成する方法について説明します。

データソースには、上流移行タスクにアクセスするための情報が含まれています。データ移行タスクは、アクセスの構成情報を取得するために対応するデータソースを参照する必要があるため、データ移行タスクを作成する前にそのタスクのデータソースを作成する必要があります。特定のデータソースの管理コマンドについては、[データソースの構成を管理する](/dm/dm-manage-source.md)を参照してください。

## ステップ1：データソースを構成する

1. (任意) データソースのパスワードを暗号化する

    DMの構成ファイルでは、dmctlで暗号化されたパスワードを使用することを推奨しています。次の例に従って、後で構成ファイルに書き込むために使用できるデータソースの暗号化されたパスワードを取得できます。

    {{< copyable "shell-regular" >}}

    ```bash
    tiup dmctl encrypt 'abc!@#123'
    ```

    ```
    MKxn0Qo3m3XOyjCnhEMtsUCm83EhGQDZ/T4=
    ```

2. データソースの構成ファイルを作成する

    各データソースについて、作成するための個別の構成ファイルが必要です。次の例に従って、"mysql-01"というIDのデータソースを作成するための構成ファイル`./source-mysql-01.yaml`を作成できます。

    ```yaml
    source-id: "mysql-01"    # データソースのID。このsource-idをタスク構成やdmctlコマンドで対応するデータソースと関連付けることができます。

    from:
      host: "127.0.0.1"
      port: 3306
      user: "root"
      password: "MKxn0Qo3m3XOyjCnhEMtsUCm83EhGQDZ/T4=" # 上流データソースのユーザーパスワード。dmctlで暗号化されたパスワードを使用することを推奨します。
      security:                                        # 上流データソースのTLS構成。必要ない場合、削除してください。
        ssl-ca: "/path/to/ca.pem"
        ssl-cert: "/path/to/cert.pem"
        ssl-key: "/path/to/key.pem"
    ```

## ステップ2：データソースを作成する

次のコマンドを使用してデータソースを作成できます：

{{< copyable "shell-regular" >}}

```bash
tiup dmctl --master-addr <master-addr> operate-source create ./source-mysql-01.yaml
```

その他の構成パラメータについては、[上流データベースの構成ファイル](/dm/dm-source-configuration-file.md)を参照してください。

次のような結果が返されます：

{{< copyable "" >}}

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-01",
            "worker": "dm-worker-1"
        }
    ]
}
```

## ステップ3：作成したデータソースをクエリする

データソースを作成した後、次のコマンドを使用してデータソースをクエリできます：

- データソースの`source-id`を知っている場合、`dmctl config source <source-id>`コマンドを使用してデータソースの構成を直接確認できます：

    {{< copyable "shell-regular" >}}

    ```bash
    tiup dmctl --master-addr <master-addr> config source mysql-01
    ```

    ```
    {
      "result": true,
      "msg": "",
      "cfg": "enable-gtid: false
        flavor: mysql
        source-id: mysql-01
        from:
          host: 127.0.0.1
          port: 3306
          user: root
          password: '******'
    }
    ```

- `source-id`を知らない場合、`dmctl operate-source show`コマンドを使用してソースデータベースのリストを確認し、対応するデータソースを見つけることができます。

    {{< copyable "shell-regular" >}}

    ```bash
    tiup dmctl --master-addr <master-addr> operate-source show
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "source is added but there is no free worker to bound",
                "source": "mysql-02",
                "worker": ""
            },
            {
                "result": true,
                "msg": "",
                "source": "mysql-01",
                "worker": "dm-worker-1"
            }
        ]
    }
    ```