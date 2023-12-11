---
title: 異なるスキーマ名やテーブル名を持つテーブルのデータチェック
summary: 異なるデータベース名やテーブル名のためのデータチェック方法を学びます。
aliases: ['/docs/dev/sync-diff-inspector/route-diff/','/docs/dev/reference/tools/sync-diff-inspector/route-diff/']
---

# 異なるスキーマ名やテーブル名を持つテーブルのデータチェック

[TiDB Data Migration](/dm/dm-overview.md)などのレプリケーションツールを使用する際には、`route-rules`を設定してデータを指定したテーブルにレプリケートすることができます。sync-diff-inspectorを使用すると、`rules`を設定することで異なるスキーマ名やテーブル名を持つテーブルを検証できます。

以下は簡単な設定例です。完全な設定については、[Sync-diff-inspectorユーザーガイド](/sync-diff-inspector/sync-diff-inspector-overview.md)を参照してください。

```toml
######################### データソースの設定 #########################
[data-sources.mysql1]
    host = "127.0.0.1"
    port = 3306
    user = "root"
    password = ""
    route-rules = ["rule1"]

[data-sources.tidb0]
    host = "127.0.0.1"
    port = 4000
    user = "root"
    password = ""
########################### ルート ###########################
[routes.rule1]
schema-pattern = "test_1"      # データソースのスキーマ名と一致します。ワイルドカード"*"および"?"をサポートします
table-pattern = "t_1"          # データソースのテーブル名と一致します。ワイルドカード"*"および"?"をサポートします
target-schema = "test_2"       # ターゲットデータベースのスキーマ名
target-table = "t_2"           # ターゲットテーブルの名前
```

この設定を使用すると、下流の`test_2.t_2`と`mysql1`インスタンスの`test_1.t_1`をチェックできます。

異なるスキーマ名やテーブル名を持つ多くのテーブルをチェックする場合、`rules`を使用してマッピング関係を設定することで構成を簡略化できます。スキーマまたはテーブル、またはその両方のマッピング関係を構成できます。例えば、上流の`test_1`データベースのすべてのテーブルを下流の`test_2`データベースにレプリケートする場合、次の構成を使用してチェックできます。

```toml
######################### データソースの設定 #########################
[data-sources.mysql1]
    host = "127.0.0.1"
    port = 3306
    user = "root"
    password = ""
    route-rules = ["rule1"]

[data-sources.tidb0]
    host = "127.0.0.1"
    port = 4000
    user = "root"
    password = ""
########################### ルート ###########################
[routes.rule1]
schema-pattern = "test_1"      # データソースのスキーマ名と一致します。ワイルドカード"*"および"?"をサポートします
table-pattern = "*"            # データソースのテーブル名と一致します。ワイルドカード"*"および"?"をサポートします
target-schema = "test_2"       # ターゲットデータベースのスキーマ名
target-table = "t_2"           # ターゲットテーブルの名前
```

## 注意

もし上流データベースに`test_2`.`t_2`が存在する場合、下流データベースもこのテーブルを比較します。