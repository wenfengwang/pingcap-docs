---
title: TiDBデータ移行Binlogイベントフィルター
summary: DMのbinlogイベントフィルター機能の使用方法を学ぶ
---

# TiDBデータ移行Binlogイベントフィルター

TiDBデータ移行（DM）は、一部のスキーマやテーブルに対して特定の種類のbinlogイベントのフィルタリング、ブロックおよびエラーの報告、または特定の種類のみのbinlogイベントの受信を行うbinlogイベントフィルター機能を提供します。たとえば、すべての`TRUNCATE TABLE`または`INSERT`イベントをフィルタリングできます。binlogイベントフィルター機能は、[ブロックと許可リスト](/dm/dm-block-allow-table-lists.md)機能よりも細かい制御が可能です。

## binlogイベントフィルターを構成する

タスク構成ファイルに、次の構成を追加します：

```yaml
filters:
  rule-1:
    schema-pattern: "test_*"
    ​table-pattern: "t_*"
    ​events: ["truncate table", "drop table"]
    sql-pattern: ["^DROP\\s+PROCEDURE", "^CREATE\\s+PROCEDURE"]
    ​action: Ignore
```

DM v2.0.2からは、ソース構成ファイルでbinlogイベントフィルターを構成することができます。詳細については、[アップストリームデータベース構成ファイル](/dm/dm-source-configuration-file.md)を参照してください。

スキーマやテーブルのマッチングにワイルドカードを使用する場合は、次の点に注意してください：

- `schema-pattern`および`table-pattern`はワイルドカード（`*`、`?`、`[]`を含む）のみをサポートしています。ワイルドカードマッチで`*`記号は1つだけであり、末尾に配置する必要があります。たとえば、`table-pattern: "t_*"`の場合、`"t_*"`は「t_」で始まるすべてのテーブルを指します。詳細については、[ワイルドカードマッチング](https://en.wikipedia.org/wiki/Glob_(programming)#Syntax)を参照してください。

- `sql-pattern`は正規表現のみをサポートしています。

## パラメータの説明

- [`schema-pattern`/`table-pattern`](/dm/table-selector.md): アップストリームのMySQLやMariaDBインスタンスのテーブルのbinlogイベントまたはDDL SQLステートメントで、`schema-pattern`/`table-pattern`に一致するものは、以下のルールによってフィルタリングされます。

- `events`: binlogイベントの配列です。以下の表から1つまたは複数の`Event`を選択できます。

    | イベント        | タイプ | 説明                   |
    | ---------------   | ---- | ----------------------------- |
    | `all`             |      | 以下のすべてのイベントを含む |
    | `all dml`         |      | 以下のすべてのDMLイベントを含む |
    | `all ddl`         |      | 以下のすべてのDDLイベントを含む |
    | `incompatible ddl changes` |      | 互換性のないすべてのDDLイベントを含む。ここでの「互換性のないDDL」はデータ損失を引き起こす可能性があるDDL操作を意味します   |
    | `none`            |      | 以下のいずれも含まない |
    | `none ddl`        |      | 以下のいずれも含まないDDLイベントを含む |
    | `none dml`        |      | 以下のいずれも含まないDMLイベントを含む |
    | `insert`          | DML  | `INSERT` DMLイベント              |
    | `update`          | DML  | `UPDATE` DMLイベント              |
    | `delete`          | DML  | `DELETE` DMLイベント              |
    | `create database` | DDL  | `CREATE DATABASE` DDLイベント         |
    | `drop database`   | 互換性のないDDL  | `DROP DATABASE` DDLイベント           |
    | `create table`    | DDL  | `CREATE TABLE` DDLイベント      |
    | `create index`    | DDL  | `CREATE INDEX` DDLイベント          |
    | `drop table`      | 互換性のないDDL  | `DROP TABLE` DDLイベント              |
    | `truncate table`  | 互換性のないDDL  | `TRUNCATE TABLE` DDLイベント          |
    | `rename table`    | 互換性のないDDL  | `RENAME TABLE` DDLイベント            |
    | `drop index`      | 互換性のないDDL  | `DROP INDEX` DDLイベント           |
    | `alter table`     | DDL  | `ALTER TABLE` DDLイベント           |
    | `value range decrease` | 互換性のないDDL  | 列フィールドの値範囲を減少させるDDLステートメント。たとえば、`VARCHAR(20)`を`VARCHAR(10)`に変更する`ALTER TABLE MODIFY COLUMN`ステートメント  |
    | `precision decrease` | 互換性のないDDL  | 列フィールドの精度を減少させるDDLステートメント。たとえば、`Decimal(10, 2)`を`Decimal(10, 1)`に変更する`ALTER TABLE MODIFY COLUMN`ステートメント  |
    | `modify column` | 互換性のないDDL  | 列フィールドの型を変更するDDLステートメント。たとえば、`INT`を`VARCHAR`に変更する`ALTER TABLE MODIFY COLUMN`ステートメント |
    | `rename column` | 互換性のないDDL  | 列の名前を変更するDDLステートメント |
    | `rename index` | 互換性のないDDL  | インデックス名を変更するDDLステートメント |
    | `drop column` | 互換性のないDDL  | テーブルから列を削除するDDLステートメント |
    | `drop index` | 互換性のないDDL  | テーブルのインデックスを削除するDDLステートメント |
    | `truncate table partition` | 互換性のないDDL  | 指定されたパーティションからすべてのデータを削除するDDLステートメント |
    | `drop primary key` | 互換性のないDDL  | 主キーを削除するDDLステートメント |
    | `drop unique key` | 互換性のないDDL  | 一意のキーを削除するDDLステートメント |
    | `modify default value` | 互換性のないDDL  | 列のデフォルト値を変更するDDLステートメント |
    | `modify constraint` | 互換性のないDDL  | 制約を修正するDDLステートメント |
    | `modify columns order` | 互換性のないDDL  | 列の順序を変更するDDLステートメント |
    | `modify charset` | 互換性のないDDL  | 列の文字セットを変更するDDLステートメント |
    | `modify collation` | 互換性のないDDL  | 列の照合順序を変更するDDLステートメント |
    | `remove auto increment` | 互換性のないDDL  | 自動インクリメントキーを削除するDDLステートメント |
    | `modify storage engine` | 互換性のないDDL  | テーブルのストレージエンジンを変更するDDLステートメント |
    | `reorganize table partition` | 互換性のないDDL  | テーブルのパーティションを再編成するDDLステートメント |
    | `rebuild table partition` | 互換性のないDDL  | テーブルのパーティションを再構築するDDLステートメント |
    | `exchange table partition` | 互換性のないDDL  | 2つのテーブル間でパーティションを交換するDDLステートメント |
    | `coalesce table partition` | 互換性のないDDL  | テーブルのパーティション数を減らすDDLステートメント |

- `sql-pattern`: 指定されたDDL SQLステートメントをフィルタリングするために使用されます。一致ルールは正規表現を使用できます。たとえば、`"^DROP\\s+PROCEDURE"`。

- `action`: 文字列（`Do`/`Ignore`/`Error`）です。次のようにルールに基づいて判断されます：

    - `Do`: 許可リスト。以下のいずれかの条件に該当する場合に、binlogはフィルタリングされます：
        - イベントのタイプがルールの`event`リストに含まれていない場合。
        - イベントのSQLステートメントが、ルールの`sql-pattern`に一致しない場合。
    - `Ignore`: ブロックリスト。以下のいずれかの条件に該当する場合に、binlogはフィルタリングされます：
        - イベントのタイプがルールの`event`リストに含まれている場合。
        - イベントのSQLステートメントが、ルールの`sql-pattern`に一致する場合。
    - `Error`: エラーリスト。以下のいずれかの条件に該当する場合に、binlogはエラーを報告します：
        - イベントのタイプがルールの`event`リストに含まれている場合。
        - イベントのSQLステートメントが、ルールの`sql-pattern`に一致する場合。
    - 同じテーブルに複数のルールがマッチする場合、ルールは順次適用されます。ブロックリストの優先度が最も高く、エラーリストの優先度がその次で、許可リストの優先度が最も低くなります。たとえば：
        - 同じテーブルに`Ignore`および`Error`ルールが適用される場合、`Ignore`ルールが適用されます。
- もし`Error`ルールと`Do`ルールが同じテーブルに適用されている場合、`Error`ルールが適用されます。

## 使用例

このセクションでは、シャーディング（シャーディングされたスキーマやテーブル）のシナリオでの使用例を示します。

### すべてのシャーディング削除操作をフィルタリングする

すべての削除操作をフィルタリングするには、次の2つのフィルタリングルールを構成します。

- `filter-table-rule`は、`test_*`.`t_*`パターンに一致するすべてのテーブルの`TRUNCATE TABLE`、`DROP TABLE`、`DELETE STATEMENT`の操作をフィルタリングします。
- `filter-schema-rule`は、`test_*`パターンに一致するすべてのスキーマの`DROP DATABASE`の操作をフィルタリングします。

```yaml
filters:
  filter-table-rule:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    events: ["truncate table", "drop table", "delete"]
    action: Ignore
  filter-schema-rule:
    schema-pattern: "test_*"
    events: ["drop database"]
    action: Ignore
```

### シャーディングDMLステートメントのみを移行する

シャーディングDMLステートメントのみを移行するには、次の2つのフィルタリングルールを設定します。

- `do-table-rule`は、`test_*`.`t_*`パターンに一致するすべてのテーブルの`CREATE TABLE`、`INSERT`、`UPDATE`、`DELETE`ステートメントのみを移行します。
- `do-schema-rule`は、`test_*`パターンに一致するすべてのスキーマの`CREATE DATABASE`ステートメントのみを移行します。

> **注意:**
>
> `CREATE DATABASE/TABLE`ステートメントを移行する理由は、スキーマとテーブルが作成された後にのみDMLステートメントを移行できるためです。

```yaml
filters:
  do-table-rule:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    events: ["create table", "all dml"]
    action: Do
  do-schema-rule:
    schema-pattern: "test_*"
    events: ["create database"]
    action: Do
```

### TiDBがサポートしていないSQLステートメントをフィルタリングする

TiDBがサポートしていない`PROCEDURE`ステートメントをフィルタリングするには、次の`filter-procedure-rule`を構成します。

```yaml
filters:
  filter-procedure-rule:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    sql-pattern: ["^DROP\\s+PROCEDURE", "^CREATE\\s+PROCEDURE"]
    action: Ignore
```

`filter-procedure-rule`は、`test_*`.`t_*`パターンに一致するすべてのテーブルの`^CREATE\\s+PROCEDURE`および`^DROP\\s+PROCEDURE`ステートメントをフィルタリングします。

### TiDBパーサーがサポートしていないSQLステートメントのフィルタリング

TiDBパーサーが解析できないSQLステートメントについては、DMはそれらを解析して`schema`/`table`情報を取得できません。そのため、グローバルフィルタリングルール`schema-pattern: "*"`を使用する必要があります。

> **注意:**
>
> 移行が必要なデータをフィルタリングしないためには、グローバルフィルタリングルールを可能な限り厳密に構成する必要があります。

TiDBパーサー（一部のバージョン）でサポートされていない`PARTITION`ステートメントをフィルタリングするには、次のフィルタリングルールを構成します。

```yaml
filters:
  filter-partition-rule:
    schema-pattern: "*"
    sql-pattern: ["ALTER\\s+TABLE[\\s\\S]*ADD\\s+PARTITION", "ALTER\\s+TABLE[\\s\\S]*DROP\\s+PARTITION"]
    action: Ignore
```

### 特定のDDLステートメントでエラーを報告する

DMがTiDBにレプリケートする前に、いくつかの上流操作によって生成されたDDLステートメントに対してブロックしてエラーを報告する必要がある場合は、次の設定を使用できます。

```yaml
filters:
  filter-procedure-rule:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    events: ["truncate table", "truncate table partition"]
    action: Error
```