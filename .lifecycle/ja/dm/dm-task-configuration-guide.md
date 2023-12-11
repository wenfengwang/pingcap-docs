---
title: データ移行タスク構成ガイド
summary: データ移行（DM）でデータ移行タスクを構成する方法について学びます。

# データ移行タスクの構成ガイド

このドキュメントでは、データ移行（DM）でデータ移行タスクを構成する方法について紹介します。

## 移行するデータソースを構成する

タスクのために移行するデータソースを構成する前に、DMが対応するデータソースの構成ファイルを読み込んでいることを最初に確認する必要があります。以下はいくつかの操作の参照です：

- データソースを表示するには、[データソース構成を確認する](/dm/dm-manage-source.md#check-data-source-configurations)を参照してください。
- データソースを作成するには、[データソースを作成する](/dm/migrate-data-using-dm.md#step-3-create-data-source)を参照してください。
- データソース構成ファイルを生成するには、[ソース構成ファイルの紹介](/dm/dm-source-configuration-file.md)を参照してください。

以下の`mysql-instances`の例は、データ移行タスクのために移行する必要があるデータソースをどのように構成するかを示しています：

```yaml
---

## ********* 基本設定 *********
name: test             # タスクの名前。グローバルに一意である必要があります。

## ******** データソース構成 **********
mysql-instances:
  - source-id: "mysql-replica-01"  # `source-id`が`mysql-replica-01`であるデータソースからデータを移行します。
  - source-id: "mysql-replica-02"  # `source-id`が`mysql-replica-02`であるデータソースからデータを移行します。
```

## ダウンストリームTiDBクラスタを構成する

以下の`target-database`の例は、データ移行タスクのために移行先のTiDBクラスタをどのように構成するかを示しています：

```yaml
---

## ********* 基本設定 *********
name: test             # タスクの名前。グローバルに一意である必要があります。

## ******** データソース構成 **********
mysql-instances:
  - source-id: "mysql-replica-01"  # `source-id`が`mysql-replica-01`であるデータソースからデータを移行します。
  - source-id: "mysql-replica-02"  # `source-id`が`mysql-replica-02`であるデータソースからデータを移行します。

## ******** ダウンストリームTiDBデータベースの構成 **********
target-database:       # 移行先のTiDBデータベースの構成。
  host: "127.0.0.1"
  port: 4000
  user: "root"
  password: ""         # パスワードが空でない場合は、dmctlで暗号化されたパスワードを使用することを推奨します。
```

## 移行するテーブルを構成する

> **注意：**
>
> 特定のテーブルをフィルタリングしたり、特定のテーブルを移行したりする必要がない場合は、この構成をスキップしてください。

データ移行タスクのためにデータソーステーブルのブロックと許可リストを構成するには、以下の手順を実行してください：

1. タスク構成ファイルでのブロックと許可リストのグローバルフィルタルールセットを構成します。

    ```yaml
    block-allow-list:
      bw-rule-1:                           # ブロックと許可リストルールの名前。
        do-dbs: ["test.*", "user"]         # 移行する上流スキーマの許可リスト。ワイルドカード文字（*?）がサポートされています。`do-dbs`または`ignore-dbs`のいずれかを構成する必要があります。両方のフィールドが構成されている場合は、`do-dbs`のみが有効です。
        # ignore-dbs: ["mysql", "account"] # 移行する上流スキーマのブロックリスト。ワイルドカード文字（*?）がサポートされています。
        do-tables:                         # 移行する上流テーブルの許可リスト。`do-tables`または`ignore-tables`のいずれかを構成する必要があります。両方のフィールドが構成されている場合は、`do-tables`のみが有効です。
        - db-name: "test.*"
          tbl-name: "t.*"
        - db-name: "user"
          tbl-name: "information"
      bw-rule-2:                          # ブロックと許可リストルールの名前。
        ignore-tables:                    # 移行する必要のあるデータソーステーブルのブロックリスト。
        - db-name: "user"
          tbl-name: "log"
    ```

    詳しい構成ルールについては、[ブロックおよび許可テーブルリスト](/dm/dm-block-allow-table-lists.md)を参照してください。

2. テーブルをフィルタリングするために、データソース構成でブロックと許可リストルールを参照します。

    ```yaml
    mysql-instances:
      - source-id: "mysql-replica-01"  # `source-id`が`mysql-replica-01`であるデータソースからデータを移行します。
        block-allow-list:  "bw-rule-1" # ブロックおよび許可リストルールの名前。DMバージョンがv2.0.0-beta.2より前の場合は、`black-white-list`を使用してください。
      - source-id: "mysql-replica-02"  # `source-id`が`mysql-replica-02`であるデータソースからデータを移行します。
        block-allow-list:  "bw-rule-2" # ブロックおよび許可リストルールの名前。DMバージョンがv2.0.0-beta.2より前の場合は、`black-white-list`を使用してください。
    ```

## 移行するビンログイベントを構成する

> **注意：**
>
> 特定のスキーマまたはテーブルの特定のビンログイベントをフィルタリングする必要がない場合は、この構成をスキップしてください。

データ移行タスクのためにビンログイベントのフィルタを構成するには、以下の手順を実行してください：

1. タスク構成ファイルでビンログイベントのグローバルフィルタルールセットを構成します。

    ```yaml
    filters:                                        # データソースのビンログイベントのフィルタルールセット。同時に複数のルールを設定できます。
      filter-rule-1:                                # フィルタリングルールの名前。
        schema-pattern: "test_*"                    # データソーススキーマ名のパターン。ワイルドカード文字（*?）がサポートされています。
        table-pattern: "t_*"                        # データソーステーブル名のパターン。ワイルドカード文字（*?）がサポートされています。
        events: ["truncate table", "drop table"]    # `schema-pattern`または`table-pattern`に一致するスキーマまたはテーブルでフィルタリングされるイベントの種類。
        action: Ignore                              # フィルタリングルールに一致するビンログを移行（Do）するか、無視（Ignore）するか。
      filter-rule-2:
        schema-pattern: "test"
        events: ["all dml"]
        action: Do
    ```

    詳しい構成ルールについては、[ビンログイベントフィルタ](/dm/dm-binlog-event-filter.md)を参照してください。

2. データソース構成で指定したテーブルまたはスキーマの特定のビンログイベントをフィルタリングするために、ビンログイベントのフィルタリングルールを参照します。

    ```yaml
    mysql-instances:
      - source-id: "mysql-replica-01"    # `source-id`が`mysql-replica-01`であるデータソースからデータを移行します。
        block-allow-list:  "bw-rule-1"   # ブロックおよび許可リストルールの名前。DMバージョンがv2.0.0-beta.2より前の場合は、`black-white-list`を使用してください。
        filter-rules: ["filter-rule-1"]  # データソースの特定のビンログイベントをフィルタリングするルールの名前。ここでは複数のルールを構成できます。
      - source-id: "mysql-replica-02"    # `source-id`が`mysql-replica-02`であるデータソースからデータを移行します。
        block-allow-list:  "bw-rule-2"   # ブロックおよび許可リストルールの名前。DMバージョンがv2.0.0-beta.2より前の場合は、`black-white-list`を使用してください。
        filter-rules: ["filter-rule-2"]  # データソースの特定のビンログイベントをフィルタリングするルールの名前。ここでは複数のルールを構成できます。
    ```

## データソーステーブルの移行先TiDBテーブルへのマッピングを構成する

> **注意：**
>
> - データソースの特定のテーブルを移行先のTiDBインスタンスの名前が異なるテーブルに移行する必要がない場合は、この構成をスキップしてください。
>
> - シャードマージタスクの場合は、タスク構成ファイルでマッピングルールを設定する必要が**あります**。

データソーステーブルを指定された移行先のTiDBテーブルに移行するためのルーティングマッピングルールを構成するには、以下の手順を実行してください：

1. グローバルルーティングマッピングルールセットをタスク構成ファイルで構成します。

    ```yaml
    routes:                           # データソーステーブルと移行先TiDBテーブルの間のルーティングマッピングルールセット。同時に複数のルールを設定できます。
      route-rule-1:                   # ルーティングマッピングルールの名前。
        schema-pattern: "test_*"      # 上流スキーマ名のパターン。ワイルドカード文字（*?）がサポートされています。
        table-pattern: "t_*"          # 上流テーブル名のパターン。ワイルドカード文字（*?）がサポートされています。
        target-schema: "test"         # 移行先TiDBスキーマ名。
        target-table: "t"             # 移行先TiDBテーブル名。
      route-rule-2:
        schema-pattern: "test_*"
        target-schema: "test"
    ```

    詳しい構成ルールについては、[テーブルルーティング](/dm/dm-table-routing.md)を参照してください。

2. テーブルをフィルタリングするために、データソース構成でルーティングマッピングルールを参照します。

    ```yaml
    mysql-instances:
      - source-id: "mysql-replica-01"                     # `source-id` が `mysql-replica-01` のデータソースからデータを移行します。
        block-allow-list:  "bw-rule-1"                    # block allow リストルールの名前。DMのバージョンが v2.0.0-beta.2 より前の場合は、`black-white-list` を使用してください。
        filter-rules: ["filter-rule-1"]                   # データソースの特定の binlog イベントをフィルタリングするルールの名前。複数のルールをここで構成することができます。
        route-rules: ["route-rule-1", "route-rule-2"]     # ルーティングマッピングルールの名前。複数のルールをここで構成することができます。
      - source-id: "mysql-replica-02"                     # `source-id` が `mysql-replica-02` のデータソースからデータを移行します。
        block-allow-list:  "bw-rule-2"                    # block allow リストルールの名前。DMのバージョンが v2.0.0-beta.2 より前の場合は、`black-white-list` を使用してください。
        filter-rules: ["filter-rule-2"]                   # データソースの特定の binlog イベントをフィルタリングするルールの名前。複数のルールをここで構成することができます。
    ```

## シャードマージタスクを構成する

> **注意:**
>
> - シャードマージシナリオでシャーディングDDLステートメントを移行する必要がある場合は、`shard-mode` フィールドを明示的に構成する必要があります。そうでない場合は、`shard-mode` を全く構成しないでください。
>
> - シャーディングDDLステートメントを移行することで、多くの問題が発生する可能性があります。この機能を使用する前に、DMがDDLステートメントを移行する原則と制限を理解し、この機能を慎重に使用する必要があります。

次の例は、タスクをシャードマージタスクとして構成する方法を示しています:

```yaml
---

## ********* 基本情報 *********
name: test                      # タスクの名前。グローバルに一意である必要があります。
shard-mode: "pessimistic"       # シャードマージモード。オプションのモードは ""/"pessimistic"/"optimistic" です。""モードは、デフォルトで、つまりシャーディングDDLマージが無効になります。タスクがシャードマージタスクの場合、"pessimistic" モードに設定してください。"optimistic"モードの原則と制限を十分理解した後で、"optimistic" モードに設定できます。
```

## その他の構成

以下は、このドキュメントのタスクの全体的な構成例です。完全なタスク構成テンプレートは、[DMタスク構成ファイル全体の紹介](/dm/task-configuration-file-full.md) で見つけることができます。

```yaml
---

## ********* 基本構成 *********
name: test                      # タスクの名前。グローバルに一意である必要があります。
shard-mode: "pessimistic"       # シャードマージモード。オプションのモードは ""/"pessimistic"/"optimistic" です。""モードは、デフォルトで、つまりシャーディングDDLマージが無効になります。タスクがシャードマージタスクの場合、"pessimistic" モードに設定してください。"optimistic"モードの原則と制限を十分理解した後で、"optimistic" モードに設定できます。
task-mode: all                  # タスクモード。`full`（完全データのみを移行）/`incremental`（binlog を同期的にレプリケート）/`all`（完全および増分の両方のbinlog をレプリケート） に設定できます。
timezone: "UTC"               # SQL セッションで使用されるタイムゾーン。デフォルトでは、DM はターゲットクラスタのグローバルなタイムゾーン設定を使用し、自動的に正確性を確保します。カスタマイズされたタイムゾーンはデータ移行に影響しませんが、不要です。

## ******** データソースの構成 **********
mysql-instances:
  - source-id: "mysql-replica-01"                   # `source-id` が `mysql-replica-01` のデータソースからデータを移行します。
    block-allow-list:  "bw-rule-1"                  # block allow リストルールの名前。DMのバージョンが v2.0.0-beta.2 より前の場合は、`black-white-list` を使用してください。
    filter-rules: ["filter-rule-1"]                 # データソースの特定の binlog イベントをフィルタリングするルールの名前。複数のルールをここで構成することができます。
    route-rules: ["route-rule-1", "route-rule-2"]   # ルーティングマッピングルールの名前。複数のルールをここで構成することができます。
  - source-id: "mysql-replica-02"                   # `source-id` が `mysql-replica-02` のデータソースからデータを移行します。
    block-allow-list:  "bw-rule-2"                  # block allow リストルールの名前。DMのバージョンが v2.0.0-beta.2 より前の場合は、`black-white-list` を使用してください。
    filter-rules: ["filter-rule-2"]                 # データソースの特定の binlog イベントをフィルタリングするルールの名前。複数のルールをここで構成することができます。
    route-rules: ["route-rule-2"]                   # ルーティングマッピングルールの名前。複数のルールをここで構成することができます。

## ******** ターゲット TiDB インスタンスの構成 **********
target-database:       # 下流データベースインスタンスの構成。
  host: "127.0.0.1"
  port: 4000
  user: "root"
  password: ""         # パスワードが空でない場合は、dmctl で暗号化されたパスワードを使用することを推奨します。

## ******** 機能構成セット **********
# 上流データベースインスタンスからの移行対象テーブルのフィルタールールセット。同時に複数のルールを設定できます。
block-allow-list:                      # DMのバージョンが v2.0.0-beta.2 より前の場合は、black-white-list を使用してください。
  bw-rule-1:                           # block allow リストルールの名前。
    do-dbs: ["test.*", "user"]         # 移行する上流スキーマの allow リスト。ワイルドカード文字（*?）がサポートされます。`do-dbs` または `ignore-dbs` のどちらかを構成する必要があります。両方のフィールドが構成されている場合は、`do-dbs` のみが影響します。
    # ignore-dbs: ["mysql", "account"] # 移行する上流スキーマのブロックリスト。ワイルドカード文字（*?）がサポートされます。
    do-tables:                         # 移行する上流テーブルの allow リスト。`do-tables` または `ignore-tables` のどちらかを構成する必要があります。両方のフィールドが構成されている場合は、`do-tables` のみが影響します。
    - db-name: "test.*"
      tbl-name: "t.*"
    - db-name: "user"
      tbl-name: "information"
  bw-rule-2:                         # block allow リストルールの名前。
    ignore-tables:                   # データソーステーブルのブロックリスト。
    - db-name: "user"
      tbl-name: "log"

# データソースbinlog イベントのフィルタールールセット。
filters:                                        # 同時に複数のルールを設定できます。
  filter-rule-1:                                # フィルタリングルールの名前。
    schema-pattern: "test_*"                    # データソーススキーマ名のパターン。ワイルドカード文字（*?）がサポートされます。
    table-pattern: "t_*"                        # データソーステーブル名のパターン。ワイルドカード文字（*?）がサポートされます。
    events: ["truncate table", "drop table"]    # `schema-pattern` または `table-pattern` に一致するスキーマまたはテーブルで除外するイベントのタイプ。
    action: Ignore                              # フィルタリングルールに一致するbinlog を移行（Do）するか無視（Ignore）するか。
  filter-rule-2:
    schema-pattern: "test"
    events: ["all dml"]
    action: Do

# データソースとターゲット TiDB インスタンステーブル間のルーティングマッピングルールセット。
routes:                           # 同時に複数のルールを設定できます。
  route-rule-1:                   # ルーティングマッピングルールの名前。
    schema-pattern: "test_*"      # データソーススキーマ名のパターン。ワイルドカード文字（*?）がサポートされます。
    table-pattern: "t_*"          # データソーステーブル名のパターン。ワイルドカード文字（*?）がサポートされます。
    target-schema: "test"         # 下流の TiDB スキーマの名前。
    target-table: "t"             # 下流の TiDB テーブルの名前。
  route-rule-2:
    schema-pattern: "test_*"
    target-schema: "test"
```