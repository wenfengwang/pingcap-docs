---
title: TiDBデータ移行テーブルルーティング
summary: DMのテーブルルーティングの使用法と注意事項を学ぶ
---

# TiDBデータ移行テーブルルーティング

TiDBデータ移行（DM）を使用してデータを移行する場合、テーブルルーティングを構成することで、上流のMySQLまたはMariaDBインスタンスの特定のテーブルを指定した下流のテーブルに移行することができます。

> **注意:**
>
> - 1つのテーブルに複数の異なるルーティングルールを構成することはサポートされていません。
> - スキーマの一致ルールは別途構成する必要があります。これは、[テーブルルーティングを構成する](#configure-table-routing)セクションの`rule-2`に示されているように、`CREATE/DROP SCHEMA xx`を移行するために使用されます。

## テーブルルーティングの構成

```yaml
routes:
  rule-1:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    target-schema: "test"
    target-table: "t"
    # extract-table, extract-schema, and extract-source are optional and
    # are required only when you need to extract information about sharded
    # tables, sharded schemas, and source datatabase information.
    extract-table:
      table-regexp: "t_(.*)"
      target-column: "c_table"
    extract-schema:
      schema-regexp: "test_(.*)"
      target-column: "c_schema"
    extract-source:
      source-regexp: "(.*)"
      target-column: "c_source"
  rule-2:
    schema-pattern: "test_*"
    target-schema: "test"
```

正規表現とワイルドカードを使用してデータベース名とテーブル名を一致させることができます。シンプルなシナリオでは、スキーマとテーブルの一致にはワイルドカードを使用することをお勧めします。ただし、次の点に注意してください。

- `*`、`?`、`[]`を含むワイルドカードがサポートされています。ワイルドカード一致には`*`記号が1つだけ含まれることができ、その記号は末尾に配置する必要があります。例えば、`table-pattern: "t_*"`において、`"t_*"`は`t_`で始まる全てのテーブルを表します。詳細は[ワイルドカード一致](https://en.wikipedia.org/wiki/Glob_(programming)#Syntax)を参照してください。

- `table-regexp`、`schema-regexp`、`source-regexp`は正規表現のみをサポートし、`~`記号で始まることはできません。

- `schema-pattern`と`table-pattern`はワイルドカードと正規表現の両方をサポートします。正規表現は`~`記号で始まる必要があります。

## パラメータの説明

- DMは、テーブルセレクタが提供する[`schema-pattern`/`table-pattern`ルールに一致する上流のMySQLまたはMariaDBインスタンスのテーブルを、下流の`target-schema`/`target-table`に移行します。
- `schema-pattern`/`table-pattern`ルールに一致するシャードされたテーブルについて、DMは`extract-table`.`table-regexp`正規表現を使用してテーブル名を抽出し、`extract-schema`.`schema-regexp`正規表現を使用してスキーマ名を抽出し、`extract-source`.`source-regexp`正規表現を使用してソース情報を抽出します。その後、DMは下流の統合されたテーブルの対応する`target-column`に抽出された情報を書き込みます。

## 使用例

このセクションでは、4つの異なるシナリオでの使用例を示します。

MySQLの小さなデータセットのシャードをTiDBにマージし移行する場合は、[このチュートリアル](/migrate-small-mysql-shards-to-tidb.md)を参照してください。

### シャードされたスキーマとテーブルをマージする

シャードされたスキーマとテーブルのシナリオでは、上流のMySQLインスタンスの2つの`test_{1,2,3...}`.`t_{1,2,3...}`テーブルを、下流のTiDBインスタンスの`test`.`t`テーブルに移行したいとします。

上流のインスタンスを下流の`test`.`t`に移行するために、次のルーティングルールを作成する必要があります。

- `rule-1`は、`schema-pattern: "test_*"`および`table-pattern: "t_*"`に一致するテーブルのDMLまたはDDLステートメントを下流の`test`.`t`に移行するために使用されます。
- `rule-2`は、`schema-pattern: "test_*"`に一致するスキーマのDDLステートメント（`CREATE/DROP SCHEMA xx`など）を移行するために使用されます。

> **注意:**
>
> - 下流の`schema: test`が既に存在し、削除しない場合、`rule-2`を省略することができます。
> - 下流の`schema: test`が存在せず、`rule-1`のみを構成する場合、移行中に`schema test doesn't exist`エラーが報告されます。

```yaml
  rule-1:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    target-schema: "test"
    target-table: "t"
  rule-2:
    schema-pattern: "test_*"
    target-schema: "test"
```

### マージされたテーブルにテーブル、スキーマ、ソース情報を抽出して書き込み

シャードされたスキーマとテーブルのシナリオでは、上流のMySQLインスタンスの2つの`test_{1,2,3...}`.`t_{1,2,3...}`テーブルを、下流のTiDBインスタンスの`test`.`t`テーブルに移行したいとします。同時に、シャードされたテーブルのソース情報を抽出し、これを下流の統合テーブルに書き込みたいとします。

上流のインスタンスを下流の`test`.`t`に移行するためには、前述の[シャードされたスキーマとテーブルをマージする](#merge-sharded-schemas-and-tables)セクションと同様のルーティングルールを作成する必要があります。さらに、`extract-table`、`extract-schema`、`extract-source`構成を追加する必要があります。

- `extract-table`: `schema-pattern`および`table-pattern`に一致するシャードされたテーブルの場合、DMは`table-regexp`を使用してシャードされたテーブル名を抽出し、統合テーブルの`target-column`に抽出された名前の`t_`部分を除いた接尾辞を書き込みます。
- `extract-schema`: `schema-pattern`および`table-pattern`に一致するシャードされたスキーマの場合、DMは`schema-regexp`を使用してシャードされたスキーマ名を抽出し、統合テーブルの`target-column`に抽出された名前の`test_`部分を書き込みます。
- `extract-source`: `schema-pattern`および`table-pattern`に一致するシャードされたテーブルの場合、DMはソースインスタンス情報を統合テーブルの`target-column`に書き込みます。

```yaml
  rule-1:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    target-schema: "test"
    target-table: "t"
    extract-table:
      table-regexp: "t_(.*)"
      target-column: "c_table"
    extract-schema:
      schema-regexp: "test_(.*)"
      target-column: "c_schema"
    extract-source:
      source-regexp: "(.*)"
      target-column: "c_source"
  rule-2:
    schema-pattern: "test_*"
    target-schema: "test"
```

上流のシャードされたテーブルのソース情報を下流の統合テーブルに抽出するために、**移行を開始する前に下流に合併テーブルを手動で作成する必要があります**。統合テーブルには、シャードされたテーブルのソース情報を指定するための3つの`target-column`（`c_table`、`c_schema`、`c_source`）が含まれている必要があります。さらに、これらの列は**最後の列である必要があり、[文字列型](/data-type-string.md)**である必要があります。

```sql
CREATE TABLE `test`.`t` (
    a int(11) PRIMARY KEY,
    c_table varchar(10) DEFAULT NULL,
    c_schema varchar(10) DEFAULT NULL,
    c_source varchar(10) DEFAULT NULL
);
```

上流には次の2つのデータソースがあると仮定します:

データソース`mysql-01`:

```sql
mysql> select * from test_11.t_1;
+---+
| a |
+---+
| 1 |
+---+
mysql> select * from test_11.t_2;
+---+
| a |
+---+
| 2 |
+---+
mysql> select * from test_12.t_1;
+---+
| a |
+---+
| 3 |
+---+
```

データソース`mysql-02`:

```sql
mysql> select * from test_13.t_3;
+---+
| a |
+---+
| 4 |
+---+
```

DMを使用して移行した後の統合テーブルのデータは次のようになります:

```sql
mysql> select * from test.t;
+---+---------+----------+----------+
| a | c_table | c_schema | c_source |
+---+---------+----------+----------+
| 1 | 1       | 11       | mysql-01 |
| 2 | 2       | 11       | mysql-01 |
| 3 | 1       | 12       | mysql-01 |
| 4 | 3       | 13       | mysql-02 |
+---+---------+----------+----------+
```

#### 合併テーブルの誤った作成例

> **注意:**
>
> 以下のいずれかのエラーが発生した場合、シャードされたテーブルおよびスキーマのソース情報が統合テーブルに書き込まれない可能性があります。

- `c-table`が最後の3つの列に含まれていない場合:

```sql
CREATE TABLE `test`.`t` (
    c_table varchar(10) DEFAULT NULL,
    a int(11) PRIMARY KEY,
    c_schema varchar(10) DEFAULT NULL,
    c_source varchar(10) DEFAULT NULL
);
```

- `c-source`が欠落している場合:

```sql
CREATE TABLE `test`.`t` (
    a int(11) PRIMARY KEY,
    c_table varchar(10) DEFAULT NULL,
    c_schema varchar(10) DEFAULT NULL
);
```sql
    a int(11) PRIMARY KEY,
    c_table varchar(10) DEFAULT NULL,
    c_schema varchar(10) DEFAULT NULL,
);

- `c_schema` is not a string type:

CREATE TABLE `test`.`t` (
    a int(11) PRIMARY KEY,
    c_table varchar(10) DEFAULT NULL,
    c_schema int(11) DEFAULT NULL,
    c_source varchar(10) DEFAULT NULL,
);

### Merge sharded schemas

シャードされたスキーマのシナリオを想定して、2つのアップストリームMySQLインスタンスの`test_{1,2,3...}`.`t_{1,2,3...}`テーブルを、ダウンストリームのTiDBインスタンスの`test`.`t_{1,2,3...}`テーブルにマイグレーションしたいとします。

上流のスキーマを下流の`test`.`t_[1,2,3]`にマイグレーションする場合、ルーティングルールを1つだけ作成すればよいです。

```yaml
  rule-1:
    schema-pattern: "test_*"
    target-schema: "test"
```

### Incorrect table routing

次の2つのルーティングルールが構成されていると仮定し、`test_1_bak`.`t_1_bak`が`rule-1`と`rule-2`の両方に一致する場合、テーブルルーティングの構成が番号の制限に違反するため、エラーが報告されます。

```yaml
  rule-1:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    target-schema: "test"
    target-table: "t"
  rule-2:
    schema-pattern: "test_1_bak"
    table-pattern: "t_1_bak"
    target-schema: "test"
    target-table: "t_bak"
```