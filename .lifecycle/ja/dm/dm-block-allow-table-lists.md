---
title: TiDBデータ移行ブロックリストと許可リスト
summary: DMブロックと許可リスト機能の使い方を学ぶ
---

# TiDBデータ移行ブロックリストと許可リスト

TiDBデータ移行(DM)を使用してデータを移行する際、ブロックリストと許可リストを構成して、特定のデータベースやテーブルの操作のみをフィルターしたり、移行したりできます。

## ブロックリストと許可リストを構成する

タスク構成ファイルに、次の構成を追加します：

```yaml
block-allow-list:             # DMのバージョンがv2.0.0-beta.2より古い場合、black-white-listを使用してください。
  rule-1:
    do-dbs: ["test*"]         # 「〜」以外の文字から始まる場合はワイルドカードを表し、v1.0.5以降のバージョンで正規表現ルールをサポートしています。
    do-tables:
    - db-name: "test[123]"    # test1、test2、test3に一致します。
      tbl-name: "t[1-5]"      # t1、t2、t3、t4、t5に一致します。
    - db-name: "test"
      tbl-name: "t"
  rule-2:
    do-dbs: ["~^test.*"]      # 「〜」から始まる場合は正規表現を表します。
    ignore-dbs: ["mysql"]
    do-tables:
    - db-name: "~^test.*"
      tbl-name: "~^t.*"
    - db-name: "test"
      tbl-name: "t*"
    ignore-tables:
    - db-name: "test"
      tbl-name: "log"
```

単純なシナリオでは、スキーマとテーブルの一致をワイルドカードで行うことをお勧めしますが、次のバージョンの違いに注意してください：

- `*`、`?`、および`[]`を含むワイルドカードがサポートされています。ワイルドカード一致には1つの`*`シンボルのみが含まれることができ、末尾に配置する必要があります。「tbl-name: "t*"」では、`"t*"`は`t`で始まるすべてのテーブルを表します。詳細については、[ワイルドカード一致](https://en.wikipedia.org/wiki/Glob_(programming)#Syntax)を参照してください。

- 正規表現は`〜`文字で始まる必要があります。

## パラメータの説明

- `do-dbs`：移行するスキーマの許可リスト。MySQLの[`replicate-do-db`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-do-db)と類似しています。
- `ignore-dbs`：移行するスキーマのブロックリスト。MySQLの[`replicate-ignore-db`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-ignore-db)と類似しています。
- `do-tables`：移行するテーブルの許可リスト。MySQLの[`replicate-do-table`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-do-table)と類似しています。`db-name`と`tbl-name`の両方を指定する必要があります。
- `ignore-tables`：移行するテーブルのブロックリスト。MySQLの[`replicate-ignore-table`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-ignore-table)と類似しています。`db-name`と`tbl-name`の両方を指定する必要があります。

上記のパラメータの値が`〜`文字で始まる場合、この値の後続する文字は[正規表現](https://golang.org/pkg/regexp/syntax/#hdr-syntax)として扱われます。このパラメータを使用してスキーマまたはテーブル名を一致させることができます。

## フィルタリング処理

- `do-dbs`および`ignore-dbs`に対応するフィルタリングルールは、MySQLの[データベースレベルの複製とバイナリロギングオプションの評価](https://dev.mysql.com/doc/refman/8.0/en/replication-rules-db-options.html)に類似しています。
- `do-tables`および`ignore-tables`に対応するフィルタリングルールは、MySQLの[テーブルレベルの複製オプションの評価](https://dev.mysql.com/doc/refman/8.0/en/replication-rules-table-options.html)に類似しています。

> **注意:**
>
> DMとMySQLでは、ブロックリストと許可リストのフィルタリングルールには以下のような違いがあります：
>
> - MySQLでは、[`replicate-wild-do-table`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-wild-do-table)および[`replicate-wild-ignore-table`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-wild-ignore-table)はワイルドカード文字をサポートしています。一方、DMでは`~`文字で始まる一部のパラメータ値が直接正規表現をサポートしています。
> - DMは現在、`ROW`形式のバイナリログのみをサポートし、`STATEMENT`や`MIXED`形式のバイナリログはサポートしていません。そのため、DMのフィルタリングルールはMySQLの`ROW`形式に対応しています。
> - MySQLでは、ステートメントの`USE`セクションで明示的に指定されたデータベース名のみをDDLステートメントとして判断します。一方、DMではDDLステートメントをまずDDLステートメントのデータベース名セクションに基づいて判断します。DDLステートメントにそのようなセクションが含まれていない場合、DMは`USE`セクションに基づいてステートメントを判断します。DDLステートメントが`USE test_db_2; CREATE TABLE test_db_1.test_table (c1 INT PRIMARY KEY)`であり、MySQLに`replicate-do-db=test_db_1`の設定がされている場合、DMには`do-dbs: ["test_db_1"]`の設定がされます。このルールはMySQLには適用されず、DMにのみ適用されます。

`test`.`t`テーブルのフィルタリングプロセスは次のとおりです：

1. **スキーマ**レベルでのフィルタリング：

    - `do-dbs`が空でない場合、一致するスキーマが`do-dbs`に存在するかどうかを確認します。

        - はいの場合、**テーブル**レベルでのフィルタリングを続行します。
        - いいえの場合、`test`.`t`をフィルタリングします。

    - `do-dbs`が空であり`ignore-dbs`が空でない場合、一致するスキーマが`ignore-dbs`に存在するかどうかを確認します。

        - はいの場合、`test`.`t`をフィルタリングします。
        - いいえの場合、**テーブル**レベルでのフィルタリングを続行します。

    - `do-dbs`および`ignore-dbs`がどちらも空である場合、**テーブル**レベルでのフィルタリングを続行します。

2. **テーブル**レベルでのフィルタリング：

    1. `do-tables`が空でない場合、一致するテーブルが`do-tables`に存在するかどうかを確認します。

        - はいの場合、`test`.`t`を移行します。
        - いいえの場合、`test`.`t`をフィルタリングします。

    2. `ignore-tables`が空でない場合、一致するテーブルが`ignore-tables`に存在するかどうかを確認します。

        - はいの場合、`test`.`t`をフィルタリングします。
        - いいえの場合、`test`.`t`を移行します。

    3. `do-tables`および`ignore-tables`がどちらも空である場合、`test`.`t`を移行します。

> **注意:**
>
> スキーマ`test`をフィルタリングする必要があるかどうかを調べるには、スキーマレベルでのみフィルタリングすればよいです。

## 使用例

上流のMySQLインスタンスには、次のテーブルが含まれているとします：

```
`logs`.`messages_2016`
`logs`.`messages_2017`
`logs`.`messages_2018`
`forum`.`users`
`forum`.`messages`
`forum_backup_2016`.`messages`
`forum_backup_2017`.`messages`
`forum_backup_2018`.`messages`
```

構成は次のとおりです：

```yaml
block-allow-list:  # DMのバージョンがv2.0.0-beta.2より古い場合、black-white-listを使用してください。
  bw-rule:
    do-dbs: ["forum_backup_2018", "forum"]
    ignore-dbs: ["~^forum_backup_"]
    do-tables:
    - db-name: "logs"
      tbl-name: "~_2018$"
    - db-name: "~^forum.*"
​      tbl-name: "messages"
    ignore-tables:
    - db-name: "~.*"
​      tbl-name: "^messages.*"
```

`bw-rule`ルールを適用した後：

| テーブル | フィルタリングするか | フィルタリングの理由 |
|:----|:----|:--------------|
| `logs`.`messages_2016` | はい | スキーマ`logs`が`do-dbs`に一致しないため。 |
| `logs`.`messages_2017` | はい | スキーマ`logs`が`do-dbs`に一致しないため。 |
| `logs`.`messages_2018` | はい | スキーマ`logs`が`do-dbs`に一致しないため。 |
| `forum_backup_2016`.`messages` | はい | スキーマ`forum_backup_2016`が`do-dbs`に一致しないため。 |
| `forum_backup_2017`.`messages` | はい | スキーマ`forum_backup_2017`が`do-dbs`に一致しないため。 |
| `forum`.`users` | はい | 1. `forum`スキーマは`do-dbs`と一致し、テーブルレベルでフィルタリングを続行します。<br/> 2. スキーマとテーブルが`do-tables`および`ignore-tables`のいずれにも一致せず、かつ`do-tables`が空でない場合、`forum`スキーマと`users`テーブルは一致しません。 |
| `forum`.`messages` | いいえ | 1. `forum`スキーマは`do-dbs`と一致し、テーブルレベルでフィルタリングを続行します。<br/> 2. `messages`テーブルは`db-name: "~^forum.*",tbl-name: "messages"`の`do-tables`にあります。 |
| `forum_backup_2018`.`messages` | いいえ | 1. `forum_backup_2018`スキーマは`do-dbs`と一致し、テーブルレベルでフィルタリングを続行します。<br/> 2. スキーマとテーブルは`db-name: "~^forum.*",tbl-name: "messages"`の`do-tables`に一致します。 |