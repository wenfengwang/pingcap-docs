---
title: ログのマスキング
summary: TiDBコンポーネントでのログのマスキング方法について学びます。

# ログのマスキング

TiDBは詳細なログ情報を提供する際、ログに機密データ（たとえば、ユーザーデータ）を出力することがあります。これによりデータのセキュリティリスクが発生します。このようなリスクを避けるために、各コンポーネント（TiDB、TiKV、およびPD）はユーザーデータの値を保護するためのログのマスキングを可能にする構成項目を提供しています。

## TiDB側でのログのマスキング

TiDB側でログのマスキングを有効にするには、[`global.tidb_redact_log`](/system-variables.md#tidb_redact_log)の値を`1`に設定します。この構成値のデフォルトは`0`であり、ログのマスキングは無効になっています。

グローバル変数`tidb_redact_log`を設定するために`set`構文を使用できます：

{{< copyable "sql" >}}

```sql
set @@global.tidb_redact_log=1;
```

設定後、新しいセッションで生成されたすべてのログがマスキングされます：

```sql
create table t (a int, unique key (a));
Query OK, 0 rows affected (0.00 sec)

insert into t values (1),(1);
ERROR 1062 (23000): Duplicate entry '1' for key 't.a'
```

上記の`INSERT`ステートメントのエラーログは次のように出力されます：

```
[2020/10/20 11:45:49.539 +08:00] [INFO] [conn.go:800] ["command dispatched failed"] [conn=5] [connInfo="id:5, addr:127.0.0.1:57222 status:10, collation:utf8_general_ci,  user:root"] [command=Query] [status="inTxn:0, autocommit:1"] [sql="insert into t values ( ? ) , ( ? )"] [txn_mode=OPTIMISTIC] [err="[kv:1062]Duplicate entry '?' for key 't.a'"]
```

上記のエラーログから、`tidb_redact_log`が有効になると、すべての機密情報が`?`を使用して保護されることがわかります。これにより、データのセキュリティリスクを回避できます。

## TiKV側でのログのマスキング

TiKV側でログのマスキングを有効にするには、[`security.redact-info-log`](/tikv-configuration-file.md#redact-info-log-new-in-v408)の値を`true`に設定します。この構成値のデフォルトは`false`であり、ログのマスキングは無効になっています。

## PD側でのログのマスキング

PD側でログのマスキングを有効にするには、[`security.redact-info-log`](/pd-configuration-file.md#redact-info-log-new-in-v50)の値を`true`に設定します。この構成値のデフォルトは`false`であり、ログのマスキングは無効になっています。

## TiFlash側でのログのマスキング

TiFlash側でログのマスキングを有効にするには、tiflash-serverにある[`security.redact_info_log`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)の値とtiflash-learnerにある[`security.redact-info-log`](/tiflash/tiflash-configuration.md#configure-the-tiflash-learnertoml-file)の値をどちらも`true`に設定します。これらの構成値のデフォルトは`false`であり、ログのマスキングは無効になっています。
