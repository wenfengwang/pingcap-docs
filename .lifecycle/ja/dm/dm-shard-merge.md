---
title: TiDBデータ移行シャードマージ
summary: DMのシャードマージ機能を学ぶ。
---

# TiDBデータ移行シャードマージ

TiDBデータ移行（DM）は、アップストリームのMySQL/MariaDBのシャードテーブルのDMLおよびDDLデータをマージし、マージされたデータをダウンストリームのTiDBテーブルに移行することをサポートしています。

MySQLの小さなデータセットのシャードをTiDBに移行してマージする必要がある場合は、[このチュートリアル](/migrate-small-mysql-shards-to-tidb.md)を参照してください。

## 制限事項

現在、シャードマージ機能は限られたシナリオのみでサポートされています。詳細については、[悲観的モードでのシャーディングDDL使用制限](/dm/feature-shard-merge-pessimistic.md#restrictions)および[楽観的モードでのシャードDDL使用制限](/dm/feature-shard-merge-optimistic.md#restrictions)を参照してください。

## パラメータの設定

タスク構成ファイルで、`shard-mode`を`pessimistic`に設定します：

```yaml
shard-mode: "pessimistic"
# シャードマージモード。""/ "pessimistic" / "optimistic" の任意のモードが利用可能です。""モードはデフォルトで使用され、シャーディングDDLマージは無効になります。タスクがシャードマージタスクの場合は、"pessimistic"モードに設定してください。"optimistic"モードの原則と制限を深く理解した後に、"optimistic"モードに設定できます。
```

## シャーディングDDLロックの手動ハンドリング

いくつかの異常なシナリオでは、[シャーディングDDLロックを手動で処理する必要があります](/dm/manually-handling-sharding-ddl-locks.md)。