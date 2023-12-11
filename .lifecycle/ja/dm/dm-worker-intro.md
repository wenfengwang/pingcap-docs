---
title: DM-workerの紹介
summary: DM-workerの機能を学ぶ
aliases: ['/docs/tidb-data-migration/dev/dm-worker-intro/']
---

# DM-workerの紹介

DM-workerはMySQL/MariaDBからTiDBへのデータ移行に使用されるツールです。

以下の機能があります:

- 任意のMySQLまたはMariaDBインスタンスのセカンダリデータベースとして機能します
- MySQL/MariaDBのbinlogイベントを読み込み、それらをローカルストレージに保持します
- 単一のDM-workerは、1つのMySQL/MariaDBインスタンスのデータを複数のTiDBインスタンスに移行できます
- 複数のDM-workersは、複数のMySQL/MariaDBインスタンスのデータを1つのTiDBインスタンスに移行できます

## DM-worker処理ユニット

DM-workerのタスクには、リレーログ、ダンプ処理ユニット、ロード処理ユニット、およびバイナリログレプリケーションなど、複数の論理ユニットが含まれています。

### リレーログ

リレーログは、上流のMySQL/MariaDBからのbinlogデータを永続的に保持し、binlogレプリケーションのためのbinlogイベントへのアクセス機能を提供します。

その根拠や機能はMySQLのリレーログと類似しています。詳細については、[MySQLリレーログ](https://dev.mysql.com/doc/refman/8.0/en/replica-logs-relaylog.html)を参照してください。

### ダンプ処理ユニット

ダンプ処理ユニットは、上流のMySQL/MariaDBからデータを完全にダンプしてローカルディスクに保存します。

### ロード処理ユニット

ロード処理ユニットは、ダンプ処理ユニットのダンプファイルを読み込み、それらのファイルを下流のTiDBにロードします。

### バイナリログレプリケーション/同期処理ユニット

バイナリログレプリケーション/同期処理ユニットは、上流のMySQL/MariaDBのbinlogイベントまたはリレーログのbinlogイベントを読み込み、これらのイベントをSQLステートメントに変換し、それらのステートメントを下流のTiDBに適用します。

## DM-workerが必要とする権限

このセクションでは、DM-workerが必要とする上流および下流のデータベースユーザーの権限、および各処理ユニットが必要とするユーザー権限について説明します。

### 上流データベースユーザーの権限

上流データベース（MySQL/MariaDB）ユーザーは、以下の権限を持っている必要があります:

| 権限 | スコープ |
|:----|:----|
| `SELECT` | テーブル |
| `RELOAD` | グローバル |
| `REPLICATION SLAVE` | グローバル |
| `REPLICATION CLIENT` | グローバル |

`db1`からTiDBへデータを移行する必要がある場合は、次の`GRANT`ステートメントを実行してください:

```sql
GRANT RELOAD,REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'your_user'@'your_wildcard_of_host'
GRANT SELECT ON db1.* TO 'your_user'@'your_wildcard_of_host';
```

他のデータベースからもデータをTiDBに移行する必要がある場合は、それらのデータベースのユーザーに同じ権限が付与されていることを確認してください。

### 下流データベースユーザーの権限

下流データベース（TiDB）ユーザーは、以下の権限を持っている必要があります:

| 権限 | スコープ |
|:----|:----|
| `SELECT` | テーブル |
| `INSERT` | テーブル |
| `UPDATE` | テーブル |
| `DELETE` | テーブル |
| `CREATE` | データベース、テーブル |
| `DROP` | データベース、テーブル |
| `ALTER` | テーブル |
| `INDEX` | テーブル |

移行する必要のあるデータベースまたはテーブルに対して次の`GRANT`ステートメントを実行してください:

```sql
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER,INDEX ON db.table TO 'your_user'@'your_wildcard_of_host';
GRANT ALL ON dm_meta.* TO 'your_user'@'your_wildcard_of_host';
```

### 各処理ユニットが必要とする最小限の権限

| 処理ユニット | 最小限の上流権限 (MySQL/MariaDB) | 最小限の下流権限 (TiDB) | 最小限のシステム権限 |
|:----|:--------------------|:------------|:----|
| リレーログ | `REPLICATION SLAVE` (binlogの読み取り)<br/>`REPLICATION CLIENT` (`show master status`、`show slave status`) | NULL | ローカルファイルの読み書き |
| ダンプ | `SELECT`<br/>`RELOAD` (テーブルのフラッシュとロック解除）| NULL | ローカルファイルへの書き込み |
| ロード | NULL | `SELECT` (チェックポイント履歴のクエリ)<br/>`CREATE` (データベース/テーブルの作成)<br/>`DELETE` (チェックポイントの削除)<br/>`INSERT` (ダンプデータの挿入) | ローカルファイルの読み書き |
| バイナリログレプリケーション | `REPLICATION SLAVE` (binlogの読み取り)<br/>`REPLICATION CLIENT` (`show master status`、`show slave status`) | `SELECT` (インデックスとカラムの表示)<br/>`INSERT` (DML)<br/>`UPDATE` (DML)<br/>`DELETE` (DML)<br/>`CREATE` (データベース/テーブルの作成)<br/>`DROP` (データベース/テーブルの削除)<br/>`ALTER` (テーブルの変更)<br/>`INDEX` (インデックスの作成/削除) | ローカルファイルの読み書き |

> **注意:**
>
> これらの権限は不変ではなく、リクエストに応じて変更されます。