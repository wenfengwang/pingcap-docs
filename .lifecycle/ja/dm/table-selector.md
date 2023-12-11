---
title: TiDBデータ移行のテーブルセレクタ
summary: データ移行のテーブルルーティング、バイナリログイベントのフィルタリング、およびカラムマッピングルールで使用されるテーブルセレクタについて学びます。
aliases: ['/docs/tidb-data-migration/dev/table-selector/']
---

# TiDBデータ移行のテーブルセレクタ

テーブルセレクタは、スキーマ/テーブルに基づいた[ワイルドカード文字](https://ja.wikipedia.org/wiki/%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89%E6%96%87%E5%AD%97)による一致ルールを提供します。特定のテーブルに一致させるには、`schema-pattern`/`table-pattern`を構成します。

## ワイルドカード文字

テーブルセレクタでは、次の2つのワイルドカード文字を`schema-pattern`/`table-pattern`で使用します:

+ アスタリスク文字（`*`、または "スター" とも呼ばれる）

    - `*` は0文字以上の文字に一致します。例えば、 `doc*` は `doc` と `document` に一致しますが、`dodo`には一致しません。
    - `*` は単語の末尾にのみ配置できます。 例えば、 `doc*` はサポートされていますが、 `do*c` はサポートされていません。

+ ワイルドカード文字の `?`（疑問符）

    `?` は空の文字を除く1文字に正確に一致します。

## 一致ルール

- `schema-pattern` は空にできません。
- `table-pattern` は空にできます。空に構成すると、 `schema` は `schema-pattern`に従って一致します。
- `table-pattern` が空でない場合、 `schema` は `schema-pattern`に従って一致し、 `table` は `table-pattern`に従って一致します。両方の `schema` と `table` が正常に一致する場合のみ、一致結果を取得できます。

## 使用例

- スキーマ名に `schema_` 接頭辞があるすべてのスキーマとテーブルに一致する:

    ```yaml
    schema-pattern: "schema_*"
    table-pattern: ""
    ```

- スキーマ名に `schema_` 接頭辞があり、テーブル名に `table_` 接頭辞があるすべてのテーブルに一致する:

    ```yaml
    schema-pattern = "schema_*"
    table-pattern = "table_*"
    ```