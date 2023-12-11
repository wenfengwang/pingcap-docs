---
title: TiDBデータ移行を使用して移行するテーブルのテーブルスキーマを管理する
summary: DMを使用して移行するテーブルのスキーマを管理する方法を学ぶ
---

# TiDBデータ移行を使用して移行するテーブルのテーブルスキーマを管理する

このドキュメントでは、[dmctl](/dm/dmctl-introduction.md)を使用して移行中のDMのテーブルのスキーマを管理する方法について説明します。

DMは増分レプリケーションを実行する際には、まず上流のバイナリログを読み取り、次にSQLステートメントを生成し、それを下流で実行します。ただし、上流のバイナリログには完全なテーブルスキーマが含まれていません。したがって、SQLステートメントを生成するために、DMは移行対象のテーブルのスキーマ情報を内部で維持します。これを内部テーブルスキーマと呼びます。

特定の特別な場合に対処するか、テーブルスキーマの不一致による移行の中断を処理するために、DMは `binlog-schema` コマンドを提供して、内部テーブルスキーマを取得、変更、削除します。

## 実装原則

内部テーブルスキーマは次のソースから取得されます。

- 完全データ移行(`task-mode=all`)の場合、移行タスクは3つの段階を経ます: ダンプ/ロード/同期 これは完全エクスポート、完全インポート、増分レプリケーションを意味します。ダンプ段階では、DMはデータと共にテーブルスキーマ情報をエクスポートし、自動的に下流で対応するテーブルを作成します。同期段階では、このテーブルスキーマが増分レプリケーションの開始テーブルスキーマとして使用されます。
- 同期段階では、DMが`ALTER TABLE`などのDDLステートメントを処理する際に、同時に内部テーブルスキーマを更新します。
- 増分移行(`task-mode=incremental`)の場合、下流で移行するテーブルが作成された後、DMは下流データベースからテーブルスキーマ情報を取得します。この動作はDMのバージョンによって異なります。

増分レプリケーションでは、スキーマの維持は複雑です。全体のデータレプリケーション中に、次の4つのテーブルスキーマが関係します。これらのスキーマは一致するか一致しないかの場合があります。

![schema](/media/dm/operate-schema.png)

* 現時点での上流テーブルスキーマ、`schema-U`として識別される。
* 現在DMによって消費されているバイナリイベントのテーブルスキーマ、`schema-B`として識別される。このスキーマは上流テーブルスキーマの過去の時点に対応します。
* DMが内部的に維持しているテーブルスキーマ（スキーマトラッカーコンポーネント）、`schema-I`として識別される。
* 下流のTiDBクラスターのテーブルスキーマ、`schema-D`として識別される。

ほとんどの場合、前述の4つのテーブルスキーマは一致しています。

上流データベースがテーブルスキーマを変更するDDL操作を実行すると、`schema-U`が変更されます。DDL操作を内部スキーマトラッカーコンポーネントと下流のTiDBクラスターに適用することにより、DMはこれらを順番に更新して`schema-U`と一致するように`schema-I`と`schema-D`を維持します。そのため、DMは通常、`schema-B`テーブルスキーマに対応するバイナリイベントを正常に消費できます。つまり、DDL操作が正常に移行された後、`schema-U`、`schema-B`、`schema-I`、および`schema-D`は依然として一致しています。

次のような状況に注意してください。

- [楽観的モードのシャーディングDDLサポート](/dm/feature-shard-merge-optimistic.md)を有効にして移行中、下流テーブルの`schema-D`が上流のシャーディングされたテーブルの`schema-B`および`schema-I`と一致しない場合があります。このような場合も、DMは引き続き`schema-I`と`schema-B`を一致させ、DMLに対応するバイナリイベントを正常にパースできるようにします。
- 下流テーブルに上流テーブルよりも多くの列がある場合、`schema-D`が`schema-B`および`schema-I`と一致しない場合があります。全データ移行(`task-mode=all`)の場合、DMは不一致を自動的に処理します。増分移行(`task-mode=incremental`)の場合、タスクが最初に開始されるため、内部スキーマ情報がなく、DMは自動的に下流のスキーマ(`schema-D`)を読み取り、`schema-I`を更新します（この動作はDMのバージョンによって異なります）。その後、DMは`schema-I`を使用して`schema-B`のバイナリログをパースしようとすると、「列の数が値の数と一致しない」エラーが発生します。詳細については、[上流TiDBテーブルにデータを移行する - 列数が多い下流テーブルに移行する](/migrate-with-more-columns-downstream.md)を参照してください。

`binlog-schema`コマンドを実行して、DMで維持されている`schema-I`テーブルスキーマを取得、変更、または削除できます。

> **注意:**
>
> - データ移行中にテーブルスキーマが変更される可能性があるため、予測可能なテーブルスキーマを取得するためには、現在`binlog-schema`コマンドはデータ移行タスクが`Paused`の状態にある場合にのみ使用できます。
> - 取り扱いの誤りによるデータ損失を回避するために、スキーマを変更する前にテーブルスキーマを取得してバックアップしておくことを**強くお勧め**します。

## コマンド

{{< copyable "shell-regular" >}}

```bash
help binlog-schema
```

```
テーブルスキーマを管理または表示する

使用法:
  dmctl binlog-schema [command]

利用可能なコマンド:
  delete      テーブルスキーマ構造の削除
  list        テーブルスキーマ構造の表示
  update      テーブルスキーマ構造の更新

フラグ:
  -h, --help   binlog-schemaのヘルプ

グローバルフラグ:
  -s, --source strings   MySQLソースID

コマンドの詳細については、 "dmctl binlog-schema [command] --help"を使用してください
```

> **注意:**
>
> - データ移行中にテーブルスキーマが変更される可能性があるため、現在`binlog-schema`コマンドはデータ移行タスクが`Paused`の状態にある場合にのみ使用できます。
> - 取り扱いの誤りによるデータ損失を回避するために、スキーマを変更する前にテーブルスキーマを取得してバックアップしておくことを**強くお勧め**します。

## パラメータ

* `delete`: テーブルスキーマを削除します。
* `list`: テーブルスキーマをリストします。
* `update`: テーブルスキーマを更新します。
* `-s`または`--source`:
    - 必須です。
    - 適用される操作のMySQLソースを指定します。

## 使用例

### テーブルスキーマの取得

テーブルスキーマを取得するには、`binlog-schema list`コマンドを実行します。

```bash
help binlog-schema list
```

```
テーブルスキーマ構造の表示

使用法:
  dmctl binlog-schema list <task-name> <database> <table> [flags]

フラグ:
  -h, --help   listのヘルプ

グローバルフラグ:
  -s, --source strings   MySQLソースID
```

`db_single`.`t1`テーブルのテーブルスキーマを`mysql-replica-01`のMySQLソースで`db_single`タスクで取得する場合は、次のコマンドを実行します:

{{< copyable "shell-regular" >}}

```bash
binlog-schema list -s mysql-replica-01 task_single db_single t1
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "CREATE TABLE `t1` ( `c1` int(11) NOT NULL, `c2` int(11) DEFAULT NULL, PRIMARY KEY (`c1`)) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_bin",
            "source": "mysql-replica-01",
            "worker": "127.0.0.1:8262"
        }
    ]
}
```

### テーブルスキーマの更新

テーブルスキーマを更新するには、`binlog-schema update`コマンドを実行します。

{{< copyable "shell-regular" >}}

```bash
help binlog-schema update
```

```
テーブルスキーマ構造を更新

使用法:
  dmctl binlog-schema update <task-name> <database> <table> [schema-file] [flags]

フラグ:
      --flush         テーブル情報とチェックポイントを即座にフラッシュします（デフォルトはtrue）
      --from-source   上流データベースからスキーマを指定のテーブルのスキーマとして使用します
      --from-target   下流データベースからスキーマを指定のテーブルのスキーマとして使用します
  -h, --help          updateのヘルプ
      --sync          シャードddlロックを解決するためにテーブル情報をマスターに同期します（現在のところ楽観的モードのみ）（デフォルトはtrue）

グローバルフラグ:
  -s, --source strings   MySQLソースID
```

`db_single`.`t1`テーブルのテーブルスキーマを以下のように設定したい場合：

```sql
CREATE TABLE `t1` (
    `c1` int(11) NOT NULL,
    `c2` int(11) DEFAULT NULL,
    PRIMARY KEY (`c1`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_bin
```

上記の`CREATE TABLE`ステートメントをファイル（たとえば`db_single.t1-schema.sql`）として保存し、次のコマンドを実行します:

{{< copyable "shell-regular" >}}

```bash
operate-schema set -s mysql-replica-01 task_single -d db_single -t t1 db_single.t1-schema.sql
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "127.0.0.1:8262"
        }
    ]
}
```

### テーブルスキーマの削除

テーブルスキーマを削除するには、`binlog-schema delete`コマンドを実行します:

```bash
help binlog-schema delete
```

```
テーブルのスキーマ構造を削除

使用法：
  dmctl binlog-schema delete <task-name> <database> <table> [flags]

フラグ：
  -h、--help   削除のヘルプ
  -s、--source strings   MySQLのソースID

グローバルフラグ：
  -s、--source strings   MySQLのソースID。
```

> **注意:**
>
> DMで維持されているテーブルスキーマが削除された場合、このテーブルに関連するDDL/DMLステートメントを下流に移行する必要がある場合、DMは次の3つのソースからテーブルスキーマを順番に取得しようとします:
>
> * チェックポイントテーブル内の`table_info`フィールド
> * 楽観的なシャーディングDDL内のメタ情報
> * 下流のTiDBに対応するテーブル

`db_single`.`t1`テーブルのテーブルスキーマを`db_single`タスク内の`mysql-replica-01` MySQLソースに削除したい場合は、次のコマンドを実行してください:

{{< copyable "shell-regular" >}}

```bash
binlog-schema delete -s mysql-replica-01 task_single db_single t1
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "127.0.0.1:8262"
        }
    ]
}
```