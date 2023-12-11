---
title: ロード統計
summary: TiDBデータベースのLOAD STATSの使用概要。
aliases: ['/docs/dev/sql-statements/sql-statement-load-stats/']
---

# LOAD STATS

`LOAD STATS`ステートメントは統計情報をTiDBにロードするために使用されます。

> **注意：**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

## 概要

```ebnf+diagram
LoadStatsStmt ::=
    'LOAD' 'STATS' stringLit
```

## 例

TiDBインスタンスの統計情報をダウンロードするには、アドレス`http://${tidb-server-ip}:${tidb-server-status-port}/stats/dump/${db_name}/${table_name}`にアクセスできます。

また、特定の統計情報ファイルをロードするには、`LOAD STATS ${stats_path}`を使用できます。

`${stats_path}`は絶対パスまたは相対パスで指定できます。相対パスを使用する場合、対応するファイルは`tidb-server`の起動パスから検索されます。以下に例を示します：

{{< copyable "sql" >}}

```sql
LOAD STATS '/tmp/stats.json';
```

```
クエリ OK、0行が影響を受けました (0.00 秒)
```

## MySQL互換性

このステートメントはMySQL構文のTiDB拡張です。

## 関連項目

* [統計情報](/statistics.md)