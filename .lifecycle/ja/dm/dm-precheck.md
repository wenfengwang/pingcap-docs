---
title: マイグレーションタスクの事前チェック
summary: マイグレーションタスクを開始する前にDMが実行する事前チェックについて学びます。
aliases: ['/docs/tidb-data-migration/dev/precheck/']
---

# マイグレーションタスクの事前チェック

上流から下流へのデータ移行にDMを使用する前に、事前チェックを実行して上流データベースの構成エラーを検出し、移行がスムーズに進むことを確認します。このドキュメントでは、DMの事前チェック機能、使用シナリオ、チェック項目、引数について紹介します。

## 使用シナリオ

データ移行タスクをスムーズに実行するために、DMはタスクの開始時に自動的に事前チェックをトリガーし、チェック結果を返します。DMは事前チェックに合格した後にのみ移行を開始します。

手動で事前チェックをトリガーするには、`check-task`コマンドを実行します。

例：

{{< copyable "" >}}

```bash
tiup dmctl check-task ./task.yaml
```

## チェック項目の説明

タスクのための事前チェックがトリガーされると、DMは移行モード構成に応じて対応する項目をチェックします。

このセクションでは、すべての事前チェック項目をリストします。

> **注意:**
>
> このドキュメントでは、合格する必要のあるチェック項目には"(Mandatory)"のラベルが付いています。

> - 必須のチェック項目が合格しない場合、DMはチェック後にエラーを返し、マイグレーションタスクを進めません。この場合は、エラーメッセージに従って構成を変更し、事前チェック要件を満たした後にタスクを再試行してください。

> - 必須でないチェック項目が合格しない場合、DMはチェック後に警告を返します。チェック結果に警告のみが含まれている場合は、DMは自動的に移行タスクを開始しますが、エラーは発生しません。

### 共通のチェック項目

選択した移行モードに関係なく、事前チェックには常に以下の共通のチェック項目が含まれます。

- データベースバージョン

    - MySQLバージョン > 5.5
    - MariaDBバージョン >= 10.1.2

    > **警告:**
    >
    > - DMを使用してMySQL 8.0からTiDBへのデータ移行は実験的な機能です (DM v2.0から導入)。本番環境での使用はお勧めしません。
    > - DMを使用してMariaDBからTiDBへのデータ移行は実験的な機能です。本番環境での使用はお勧めしません。

- 上流MySQLテーブルスキーマの互換性

    - 上流テーブルに外部キーがあるかどうかをチェックし、TiDBでサポートされていない場合は警告が返されます。
    - 上流テーブルがTiDBと互換性のない文字セットを使用していないかをチェックします。詳細については、[TiDB サポート文字セット](/character-set-and-collation.md)を参照してください。
    - 上流テーブルに主キーやユニークキーの制約があるかどうかをチェックします（v1.0.7以降で導入）。

    > **警告:**
    >
    > - 上流で互換性のない文字セットが使用されている場合、下流でutf8mb4文字セットのテーブルを作成することでレプリケーションを継続できます。ただし、この方法はお勧めしません。上流で使用されている非互換の文字セットを、下流でサポートされている他の文字セットに置き換えることをお勧めします。
    > - 上流テーブルに主キーやユニークキーの制約がない場合、同じデータ行が複数回下流にレプリケートされ、レプリケーションのパフォーマンスにも影響する可能性があります。本番環境では、上流テーブルの主キーやユニークキーの制約を指定することをお勧めします。

### 完全なデータ移行のチェック項目

完全なデータ移行モード (`task-mode: full`)の場合、[共通のチェック項目](#common-check-items)に加えて、事前チェックには以下のチェック項目も含まれます:

* (Mandatory) 上流データベースのダンプ権限

    - INFORMATION_SCHEMAおよびダンプテーブルへのSELECT権限
    - `consistency=flush`の場合、RELOAD権限
    - `consistency=flush/lock`の場合、ダンプテーブルへのLOCK TABLES権限

* (Mandatory) 上流MySQLマルチインスタンスのシャーディングテーブルの一貫性

    - 悲観的モードでは、以下の項目ですべてのシャーディングテーブルのテーブルスキーマが一貫しているかどうかをチェックします:

        - 列の数
        - 列名
        - 列の順序
        - 列のタイプ
        - 主キー
        - ユニークインデックス

    - 楽観的モードでは、すべてのシャーディングテーブルのスキーマが [楽観的互換性](https://github.com/pingcap/tiflow/blob/master/dm/docs/RFCS/20191209_optimistic_ddl.md#modifying-column-types) を満たしているかどうかをチェックします。

    - `start-task`コマンドによって移行タスクが正常に開始された場合、このタスクの事前チェックでは一貫性チェックがスキップされます。

* シャーディングテーブルの自動インクリメント主キー

    - シャーディングテーブルに自動インクリメント主キーが存在する場合、事前チェックは警告を返します。自動インクリメント主キーに競合がある場合は、解決策については[自動インクリメント主キーの競合処理](/dm/shard-merge-best-practices.md#handle-conflicts-of-auto-increment-primary-key)を参照してください。

#### 物理インポートのチェック項目

タスク構成で`import-mode: "physical"`を設定した場合、以下のチェック項目が追加され、「[物理インポート](/tidb-lightning/tidb-lightning-physical-import-mode.md)が正常に実行されることを確認します。プロンプトに従って、これらのチェック項目の要件を満たすことが難しい場合は、[論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode.md)を使用することを検討できます。

* 下流データベース内の空のリージョン

    - 空のリージョンの数が `max(1000, 3 * テーブル数)` ("1000" および "テーブル数の3倍" の大きい方) を上回る場合、事前チェックは警告を返します。関連するPDパラメータを調整して空のリージョンのマージを高速化し、空のリージョンの数が減少するのを待つことができます。[PDスケジューリングのベストプラクティス - リージョンマージが遅い](/best-practices/pd-scheduling-best-practices.md#region-merge-is-slow)を参照してください。

* 下流データベースでのリージョン分布

    - 異なるTiKVノード上のリージョン数をチェックします。リージョン数が最も少ないTiKVノードが `a` リージョンを持ち、最も多いTiKVノードが `b` リージョンを持つと仮定すると、`a / b` が 0.75 より小さい場合、事前チェックは警告を返します。関連するPDパラメータを調整してリージョンのスケジューリングを高速化し、リージョンの数が変化するのを待つことができます。[PDスケジューリングのベストプラクティス - リーダーリージョンの分布が均等でない](/best-practices/pd-scheduling-best-practices.md#leadersregions-are-not-evenly-distributed)を参照してください。

* 下流データベースのTiDB、PD、TiKVのバージョン

    - 物理インポートはTiDB、PD、TiKVのインタフェースを呼び出す必要があります。バージョンが互換性がない場合、事前チェックはエラーを返します。

* 下流データベースのフリースペース

    - 上流データベースの許可リスト内のすべてのテーブルの合計サイズを見積もります (`source_size`)。下流データベースのフリースペースが `source_size` よりも少ない場合、事前チェックはエラーを返します。下流データベースのフリースペースが TiKVレプリカ数 \* `source_size` \* 2 よりも少ない場合、事前チェックは警告を返します。

* 下流データベースでの物理インポートと互換性のないタスクの実行

    - 現在、物理インポートは下流データベースで [TiCDC](/ticdc/ticdc-overview.md) および [PITR](/br/br-pitr-guide.md) タスクと互換性がありません。これらのタスクが下流データベースで実行されている場合、事前チェックはエラーを返します。

### 増分データ移行のチェック項目

増分データ移行モード (`task-mode: incremental`)の場合、[共通のチェック項目](#common-check-items)に加えて、事前チェックには以下のチェック項目も含まれます:

* (Mandatory) 上流データベースのREPLICATION権限

    - REPLICATION CLIENT権限
    - REPLICATION SLAVE権限

* データベースのプライマリ・セカンダリ構成

    - プライマリ-セカンダリレプリケーションの障害を避けるために、上流データベースのデータベースID `server_id` を指定することがお勧めされます (AWS Aurora環境ではGTIDが推奨されます)。

* (Mandatory) MySQL binlogの設定

    - binlogが有効になっているかをチェックします (DMが必要とする)。
    - `binlog_format=ROW` が設定されているかをチェックします (DMはROWフォーマットのbinlogの移行のみをサポートしています)。
    - `binlog_row_image=FULL` が設定されているかをチェックします (DMは `binlog_row_image=FULL` のみをサポートしています)。
    - `binlog_do_db` や `binlog_ignore_db` が設定されている場合、移行するデータベーステーブルが `binlog_do_db` および `binlog_ignore_db` の条件を満たしているかをチェックします。

* (Mandatory) 上流データベースが [Online-DDL](/dm/feature-online-ddl.md) プロセス中かどうかをチェックします（`ghost`テーブルが作成されているが、`rename`フェーズがまだ実行されていない状態）。上流がオンライン-DDLプロセス中の場合、事前チェックはエラーを返します。この場合は、DDLが完了するまで待機し、再試行してください。

### 完全および増分データ移行のチェック項目
完全および増分データ移行モード（`task-mode: all`）の場合、[一般的なチェック項目](#common-check-items)に加えて、事前チェックには[完全データ移行チェック項目](#check-items-for-full-data-migration)と[増分データ移行チェック項目](#check-items-for-incremental-data-migration)も含まれます。

### 無視可能なチェック項目

事前チェックでは、環境の潜在的なリスクを発見することができます。チェック項目を無視することは推奨されていません。データ移行タスクに特別な要件がある場合は、[`ignore-checking-items`構成項目](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)を使用して、一部のチェック項目をスキップすることができます。

| チェック項目                | 説明               |
| :-------------------------- | :----------------- |
| `dump_privilege`            | 上流MySQLインスタンスのユーザーのダンプ権限をチェックします。 |
| `replication_privilege`     | 上流MySQLインスタンスのユーザーのレプリケーション権限をチェックします。 |
| `version`                   | 上流データベースのバージョンをチェックします。 |
| `server_id`                 | 上流データベースでserver_idが設定されているかをチェックします。 |
| `binlog_enable`             | 上流データベースでbinlogが有効になっているかをチェックします。 |
| `table_schema`              | 上流MySQLテーブルのテーブルスキーマの互換性をチェックします。 |
| `schema_of_shard_tables`    | 上流MySQLマルチインスタンスのシャードのテーブルスキーマの整合性をチェックします。 |
| `auto_increment_ID`         | 上流MySQLマルチインスタンスのシャードで、auto-incrementプライマリーキーの競合をチェックします。 |
| `online_ddl`                | 上流が[オンライン-DDL](/dm/feature-online-ddl.md)のプロセス中かどうかをチェックします。 |
| `empty_region`              | 物理インポート用の下流データベースにおける空のリージョンの数をチェックします。 |
| `region_distribution`       | 物理インポート用の下流データベースにおけるリージョンの分布をチェックします。 |
| `downstream_version`        | 下流データベースにおけるTiDB、PD、TiKVのバージョンをチェックします。 |
| `free_space`                | 下流データベースの空き領域をチェックします。 |
| `downstream_mutex_features` | 下流データベースが物理インポートと互換性のないタスクを実行しているかをチェックします。 |

> **注意:**
>
> バージョンv6.0より前では、より多くの無視可能なチェック項目がサポートされています。v6.0からは、DMではデータの安全性に関連する一部のチェック項目を無視することは許可されません。たとえば、`binlog_row_image`パラメーターを誤って設定すると、レプリケーション中にデータが失われる可能性があります。

## 事前チェック引数を構成する

移行タスクの事前チェックは並列処理をサポートしています。シャード化されたテーブルの行数が100万行に達しても、事前チェックは数分で完了することができます。

事前チェックのスレッド数を指定するには、移行タスク構成ファイルの`mydumpers`フィールドの`threads`引数を構成できます。

```yaml
mydumpers:                           # ダンプ処理ユニットの構成引数
  global:                            # 構成名
    threads: 4                       # ダンプ処理ユニットが上流から事前チェックを実行し、上流データベースからデータをエクスポートする際にアクセスするスレッドの数（デフォルトは4）
    chunk-filesize: 64               # ダンプ処理ユニットによって生成されるファイルのサイズ（デフォルトは64 MB）
    extra-args: "--consistency none" # ダンプ処理ユニットのその他の引数。`extra-args`で`table-list`を手動で構成する必要はありません。なぜなら、DMが自動的に生成するからです。

```

> ** 注意: **
>
> `threads`の値は、上流データベースとDM間の物理的な接続数を決定します。過度に大きな`threads`の値は、上流の負荷を増加させる可能性があります。したがって、適切な値に`threads`を設定する必要があります。