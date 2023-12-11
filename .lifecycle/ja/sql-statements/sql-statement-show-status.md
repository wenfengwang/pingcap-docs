---
title: SHOW [GLOBAL|SESSION] STATUS | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのSHOW [GLOBAL|SESSION] STATUSの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-show-status/','/docs/dev/reference/sql/statements/show-status/']
---

# SHOW [GLOBAL|SESSION] STATUS

このステートメントはMySQLとの互換性のために含まれています。これはTiDBに影響を与えず、代わりに`SHOW STATUS`の代わりにPrometheusとGrafanaを使用して集中メトリクス収集を行います。

## 概要

**ShowStmt:**

![ShowStmt](/media/sqlgram/ShowStmt.png)

**ShowTargetFilterable:**

![ShowTargetFilterable](/media/sqlgram/ShowTargetFilterable.png)

**GlobalScope:**

![GlobalScope](/media/sqlgram/GlobalScope.png)

## 例

```sql
mysql> show status;
+--------------------+--------------------------------------+
| Variable_name      | Value                                |
+--------------------+--------------------------------------+
| Ssl_cipher_list    |                                      |
| server_id          | 93e2e07d-6bb4-4a1b-90b7-e035fae154fe |
| ddl_schema_version | 141                                  |
| Ssl_verify_mode    | 0                                    |
| Ssl_version        |                                      |
| Ssl_cipher         |                                      |
+--------------------+--------------------------------------+
6 rows in set (0.01 sec)

mysql> show global status;
+--------------------+--------------------------------------+
| Variable_name      | Value                                |
+--------------------+--------------------------------------+
| Ssl_cipher         |                                      |
| Ssl_cipher_list    |                                      |
| Ssl_verify_mode    | 0                                    |
| Ssl_version        |                                      |
| server_id          | 93e2e07d-6bb4-4a1b-90b7-e035fae154fe |
| ddl_schema_version | 141                                  |
+--------------------+--------------------------------------+
6 rows in set (0.00 sec)
```

## MySQL互換性

* このステートメントはMySQLとの互換性のために含まれています。

## 関連項目

* [FLUSH STATUS](/sql-statements/sql-statement-flush-status.md)