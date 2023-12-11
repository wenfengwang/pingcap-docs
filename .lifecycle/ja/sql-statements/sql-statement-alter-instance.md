---
title: ALTER INSTANCE
summary: TiDBでの`ALTER INSTANCE`の使用概要を学びます。
aliases: ['/docs/dev/sql-statements/sql-statement-alter-instance/','/docs/dev/reference/sql/statements/alter-instance/']
---

# ALTER INSTANCE

`ALTER INSTANCE`ステートメントは、単一のTiDBインスタンスに変更を加えるために使用されます。現在、TiDBは`RELOAD TLS`句のみをサポートしています。

> **Note:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)はTLS証明書を自動的にリフレッシュできるため、この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタには適用されません。

## RELOAD TLS

<CustomContent platform="tidb">

`ALTER INSTANCE RELOAD TLS`ステートメントを実行して、元の構成パスから証明書([`ssl-cert`](/tidb-configuration-file.md#ssl-cert))、キー([`ssl-key`](/tidb-configuration-file.md#ssl-key))、およびCA([`ssl-ca`](/tidb-configuration-file.md#ssl-ca))をリロードできます。

</CustomContent>

<CustomContent platform="tidb-cloud">

`ALTER INSTANCE RELOAD TLS`ステートメントを実行して、元の構成パスから証明書([`ssl-cert`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#ssl-cert))、キー([`ssl-key`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#ssl-key))、およびCA([`ssl-ca`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#ssl-ca))をリロードできます。

</CustomContent>

新しく読み込まれた証明書、キー、およびCAは、ステートメントが正常に実行された後に確立された接続に影響します。このステートメントの実行前に確立された接続には影響しません。

リロード中にエラーが発生した場合、デフォルトではエラーメッセージが返され、前のキーと証明書が引き続き使用されます。ただし、オプションの`NO ROLLBACK ON ERROR`を追加した場合、エラーが発生してもエラーは返されず、TLSセキュリティ接続が無効になったまま後続のリクエストが処理されます。

## 構文図

**AlterInstanceStmt:**

```ebnf+diagram
AlterInstanceStmt ::=
    'ALTER' 'INSTANCE' InstanceOption

InstanceOption ::=
    'RELOAD' 'TLS' ('NO' 'ROLLBACK' 'ON' 'ERROR')?
```

## 例

{{< copyable "sql" >}}

```sql
ALTER INSTANCE RELOAD TLS;
```

## MySQL互換性

`ALTER INSTANCE RELOAD TLS`ステートメントは、元の構成パスからのリロードのみをサポートしています。TiDBの起動時に動的にロードパスを変更したり、TLS暗号化接続機能を動的に有効にしたりすることはサポートされていません。この機能は、TiDBを再起動した際にデフォルトで無効になっています。

## 関連項目

<CustomContent platform="tidb">

[TiDBクライアントとサーバー間でTLSを有効にする](/enable-tls-between-clients-and-servers.md)。

</CustomContent>

<CustomContent platform="tidb-cloud">

[TiDBクライアントとサーバー間でTLSを有効にする](https://docs.pingcap.com/tidb/stable/enable-tls-between-clients-and-servers)。

</CustomContent>