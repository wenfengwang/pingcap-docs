---
title: チェンジフィードログフィルター
summary: TiCDCのテーブルフィルターとイベントフィルターの使用方法を学びます。
---

# チェンジフィードログフィルター

TiCDCは、テーブルとイベントによるデータのフィルタリングをサポートしています。このドキュメントでは、2種類のフィルターの使用方法について紹介します。

## テーブルフィルター

テーブルフィルターは、次の構成を指定することで特定のデータベースやテーブルを保持したりフィルタリングしたりする機能です。

```toml
[filter]
# フィルタールール
rules = ['*.*', '!test.*']
```

一般的なフィルタールール:

- `rules = ['*.*']`
    - すべてのテーブルをレプリケートする (システムテーブルは含まれません)
- `rules = ['test1.*']`
    - `test1`データベース内のすべてのテーブルをレプリケートする
- `rules = ['*.*', '!scm1.tbl2']`
    - `scm1.tbl2`テーブルを除くすべてのテーブルをレプリケートする
- `rules = ['scm1.tbl2', 'scm1.tbl3']`
    - `scm1.tbl2`および`scm1.tbl3`テーブルのみをレプリケートする
- `rules = ['scm1.tidb_*']`
    - `tidb_`から始まる名前を持つ`scm1`データベース内のすべてのテーブルをレプリケートする

詳細については、[テーブルフィルターシンタックス](/table-filter.md#syntax)を参照してください。

## イベントフィルタールール

v6.2.0から、TiCDCはイベントフィルターをサポートしています。指定された条件を満たすDMLおよびDDLイベントをフィルタリングするためのイベントフィルタールールを構成できます。

以下は、イベントフィルタールールの例です。

```toml
[filter]
# イベントフィルタールールは`[filter]`構成の下に配置する必要があります。同時に複数のイベントフィルターを構成できます。

[[filter.event-filters]]
matcher = ["test.worker"] # matcherは許可リストであり、このルールはtestデータベースのworkerテーブルにのみ適用されることを意味します。
ignore-event = ["insert"] # insertイベントを無視します。
ignore-sql = ["^drop", "add column"] # "drop"で始まるまたは"add column"を含むDDLを無視します。
ignore-delete-value-expr = "name = 'john'" # "name = 'john'"条件を含む削除DMLを無視します。
ignore-insert-value-expr = "id >= 100" # "id >= 100"条件を含む挿入DMLを無視します。
ignore-update-old-value-expr = "age < 18 or name = 'lili'" # 古い値が"age < 18"または"name = 'lili'"を含む更新DMLを無視します。
ignore-update-new-value-expr = "gender = 'male' and age > 18" # 新しい値が"gender = 'male'"かつ"age > 18"を含む更新DMLを無視します。
```

構成パラメーターの説明:

- `matcher`: このイベントフィルタールールが適用されるデータベースとテーブル。構文は[テーブルフィルター](/table-filter.md)と同じです。
- `ignore-event`: 無視するイベントタイプ。このパラメーターは文字列の配列を受け入れます。複数のイベントタイプを構成できます。現在サポートされているイベントタイプは次のとおりです:

| イベント | タイプ | エイリアス | 説明 |
| ---------- | ---- | -|--------------------------|
| すべてのDML | | |すべてのDMLイベントに一致  |
| すべてのDDL | | |すべてのDDLイベントに一致         |
| insert      | DML | |`insert`DMLイベントに一致       |
| update      | DML | |`update`DMLイベントに一致       |
| delete      | DML | |`delete`DMLイベントに一致       |
| create schema   | DDL  | create database |`create database`イベントに一致 |
| drop schema     | DDL  | drop database  |`drop database`イベントに一致 |
| create table    | DDL  | |`create table`イベントに一致   |
| drop table      | DDL  | |`drop table`イベントに一致     |
| rename table    | DDL  | |`rename table`イベントに一致   |
| truncate table  | DDL  | |`truncate table`イベントに一致 |
| alter table     | DDL  | |`alter table`イベントに一致、`alter table`、`create index`および`drop index`のすべての節を含む   |
| add table partition    | DDL  | |`add table partition`イベントに一致     |
| drop table partition    | DDL  | |`drop table partition`イベントに一致     |
| truncate table partition    | DDL  | |`truncate table partition`イベントに一致     |
| create view     | DDL  | |`create view`イベントに一致     |
| drop view     | DDL  | |`drop view`イベントに一致     |

- `ignore-sql`: 無視するDDLステートメント。このパラメーターは正規表現の配列を受け入れ、複数の正規表現を構成できます。このルールはDDLイベントにのみ適用されます。
- `ignore-delete-value-expr`: このパラメーターはSQL式を受け入れます。このルールは指定された値を持つ削除DMLイベントにのみ適用されます。
- `ignore-insert-value-expr`: このパラメーターはSQL式を受け入れます。このルールは指定された値を持つ挿入DMLイベントにのみ適用されます。
- `ignore-update-old-value-expr`: このパラメーターはSQL式を受け入れます。このルールは古い値に指定された値を含む更新DMLイベントにのみ適用されます。
- `ignore-update-new-value-expr`: このパラメーターはSQL式を受け入れます。このルールは新しい値に指定された値を含む更新DMLイベントにのみ適用されます。

> **注意:**
>
> - TiDBがクラスタ化されたインデックスの列の値を更新する場合、TiDBは`UPDATE`イベントを`DELETE`イベントと`INSERT`イベントに分割します。TiCDCではこのようなイベントを`UPDATE`イベントとして識別せず、これらのイベントを正しくフィルタリングできません。
> - SQL式を構成する場合、`matcher`と一致するすべてのテーブルにSQL式で指定されたすべての列が含まれていることを確認してください。そうしないと、レプリケーションタスクを作成できません。さらに、レプリケーション中にテーブルスキーマが変更され、必要な列が含まれなくなった場合、レプリケーションタスクは失敗し、自動的に再開されません。そのような状況では、構成を手動で修正してタスクを再開する必要があります。