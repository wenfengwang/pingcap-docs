---
title: DM レプリケーションシナリオにおけるデータチェック
summary: DM-master から特定の `task-name` 設定を行ってデータチェックを実行する方法について学びます。

# DM レプリケーションシナリオにおけるデータチェック

[TiDB データ移行](/dm/dm-overview.md)などのレプリケーションツールを使用する場合、レプリケーションプロセスの前後でデータの整合性を確認する必要があります。`DM-master` から特定の `task-name` 設定を行ってデータチェックを実行することができます。

以下はシンプルな構成の例です。完全な構成については、[Sync-diff-inspector ユーザーガイド](/sync-diff-inspector/sync-diff-inspector-overview.md)を参照してください。

```toml
# Diff Configuration.

######################### グローバル設定 #########################

# データをチェックするために作成されたゴルーチンの数。上流と下流のデータベース間の接続数は、この値よりわずかに大きくなります。
check-thread-count = 4

# 有効になっている場合、SQL 文が不整合なテーブルを修正するためにエクスポートされます。
export-fix-sql = true

# データの代わりにテーブル構造のみを比較します。
check-struct-only = false

# DM-master の IP アドレス、フォーマットは "http://127.0.0.1:8261" です。
dm-addr = "http://127.0.0.1:8261"

# DM の `task-name` を指定します。
dm-task = "test"

######################### タスクの設定 #########################
[task]
    output-dir = "./output"

    # 比較する下流データベースのテーブル。各テーブルには、スキーマ名とテーブル名が含まれており、'.' で区切られている必要があります。
    target-check-tables = ["hb_test.*"]
```

この例では、dm-task = "test" に設定されており、"test" タスクの hb_test スキーマのすべてのテーブルをチェックしています。これにより、上流と下流のデータベース間のスキーマの正規の一致を自動的に取得し、DM レプリケーション後のデータ整合性を検証します。