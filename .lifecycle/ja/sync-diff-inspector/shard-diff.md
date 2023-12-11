---
title: シャーディングシナリオにおけるデータチェック
summary: シャーディングシナリオにおけるデータチェックを学ぶ。
aliases: ['/docs/dev/sync-diff-inspector/shard-diff/','/docs/dev/reference/tools/sync-diff-inspector/shard-diff/']
---

# シャーディングシナリオにおけるデータチェック

sync-diff-inspectorはシャーディングシナリオにおけるデータチェックをサポートしています。複数のMySQLインスタンスからTiDBにデータを複製するために[TiDBデータ移行](/dm/dm-overview.md)ツールを使用する場合、sync-diff-inspectorを使用して上流と下流のデータをチェックできます。

上流のシャードされたテーブルの数が少なく、シャードされたテーブルの命名規則に以下のようなパターンがないシナリオでは、`Datasource config`を使用して`table-0`を構成し、対応する`rules`を設定し、上流と下流のデータベース間のマッピング関係を持つテーブルを構成できます。この構成方法には、すべてのシャードされたテーブルを設定する必要があります。

![shard-table-replica-1](/media/shard-table-replica-1.png)

以下はsync-diff-inspector構成の完全な例です。

``` toml
# Diff Configuration.

######################### Global config #########################

# データをチェックするために作成されたゴルーチンの数。上流と下流のデータベース間の接続数は、わずかにこの値よりも大きくなります
check-thread-count = 4

# 有効になっている場合、SQLステートメントが不整合なテーブルを修正するためにエクスポートされます
export-fix-sql = true

# データの代わりにテーブル構造のみを比較する
check-struct-only = false

######################### Datasource config #########################
[data-sources.mysql1]
    host = "127.0.0.1"
    port = 3306
    user = "root"
    password = ""

    route-rules = ["rule1"]

[data-sources.mysql2]
    host = "127.0.0.1"
    port = 3306
    user = "root"
    password = ""

    route-rules = ["rule2"]

[data-sources.tidb0]
    host = "127.0.0.1"
    port = 4000
    user = "root"
    password = ""

########################### Routes ###########################
[routes.rule1]
schema-pattern = "test"        # データソースのスキーマ名に一致します。ワイルドカード"*"および"?"がサポートされています
table-pattern = "table-[1-2]"  # データソースのテーブル名に一致します。ワイルドカード"*"および"?"がサポートされています
target-schema = "test"         # ターゲットデータベースのスキーマ名
target-table = "table-0"       # ターゲットテーブルの名前

[routes.rule2]
schema-pattern = "test"      # データソースのスキーマ名に一致します。ワイルドカード"*"および"?"がサポートされています
table-pattern = "table-3"    # データソースのテーブル名に一致します。ワイルドカード"*"および"?"がサポートされています
target-schema = "test"       # ターゲットデータベースのスキーマ名
target-table = "table-0"     # ターゲットテーブルの名前

######################### Task config #########################
[task]
    output-dir = "./output"

    source-instances = ["mysql1", "mysql2"]

    target-instance = "tidb0"

    # 比較対象となる下流データベースのテーブル。各テーブルには、スキーマ名とテーブル名が"."で区切られている必要があります
    target-check-tables = ["test.table-0"]
```

上流のシャードされたテーブルの数が多く、すべてのシャードされたテーブルの命名規則にパターンがある場合は、以下に示すように構成するために`table-rules`を使用できます。

![shard-table-replica-2](/media/shard-table-replica-2.png)

以下はsync-diff-inspector構成の完全な例です。

```toml
# Diff Configuration.
######################### Global config #########################

# データをチェックするために作成されたゴルーチンの数。上流と下流のデータベース間の接続数は、わずかにこの値よりも大きくなります。
check-thread-count = 4

# 有効になっている場合、SQLステートメントが不整合なテーブルを修正するためにエクスポートされます。
export-fix-sql = true

# データの代わりにテーブル構造のみを比較します。
check-struct-only = false

######################### Datasource config #########################
[data-sources.mysql1]
    host = "127.0.0.1"
    port = 3306
    user = "root"
    password = ""

[data-sources.mysql2]
    host = "127.0.0.1"
    port = 3306
    user = "root"
    password = ""

[data-sources.tidb0]
    host = "127.0.0.1"
    port = 4000
    user = "root"
    password = ""

########################### Routes ###########################
[routes.rule1]
schema-pattern = "test"      # データソースのスキーマ名に一致します。ワイルドカード"*"および"?"がサポートされています
table-pattern = "table-*"    # データソースのテーブル名に一致します。ワイルドカード"*"および"?"がサポートされています
target-schema = "test"       # ターゲットデータベースのスキーマ名
target-table = "table-0"     # ターゲットテーブルの名前

######################### Task config #########################
[task]
    output-dir = "./output"
    source-instances = ["mysql1", "mysql2"]

    target-instance = "tidb0"

    # 比較対象となる下流データベースのテーブル。各テーブルには、スキーマ名とテーブル名が"."で区切られている必要があります
    target-check-tables = ["test.table-0"]
```

## 注意

もし上流データベースに`test.table-0`が存在する場合、下流データベースもこのテーブルを比較します。