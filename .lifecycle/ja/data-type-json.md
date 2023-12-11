---
title: TiDBデータ型
summary: TiDBのJSONデータ型について学びます。
aliases: ['/docs/dev/data-type-json/','/docs/dev/reference/sql/data-types/json/']
---

# JSON型

TiDBは、半構造化データを保存するのに便利な、`JSON`（JavaScript Object Notation）データ型をサポートしています。`JSON`データ型は、以下に示すような利点を提供します。

- 直列化のためにバイナリ形式を使用します。内部形式により、`JSON`ドキュメントの要素に素早くアクセスできます。
- `JSON`列に保存されたJSONドキュメントが自動的に検証されます。有効なドキュメントのみが保存できます。

`JSON`列は、他のバイナリ型の列と同様に直接インデックス化されませんが、生成された列の形式で`JSON`ドキュメント内のフィールドにインデックスを付けることができます。

```sql
CREATE TABLE city (
    id INT PRIMARY KEY,
    detail JSON,
    population INT AS (JSON_EXTRACT(detail, '$.population')),
    index index_name (population)
    );
INSERT INTO city (id,detail) VALUES (1, '{"name": "Beijing", "population": 100}');
SELECT id FROM city WHERE population >= 100;
```

詳細については、[JSON関数](/functions-and-operators/json-functions.md)と[生成列](/generated-columns.md)を参照してください。

## 制限事項

- 現在、TiDBではTiFlashに限定された`JSON`関数の押し下げのみがサポートされています。詳細については、[押し下げ式](/tiflash/tiflash-supported-pushdown-calculations.md#push-down-expressions)を参照してください。
- TiDB バックアップ＆リストア（BR）のバージョンv6.3.0より前は、JSON列を含むデータのリカバリをサポートしていません。BRのどのバージョンも、JSON列を含むデータをTiDBクラスターv6.3.0より前にリカバリすることをサポートしていません。
- `DATE`、`DATETIME`、`TIME`などの標準でない`JSON`データ型を含むデータをレプリケーションツールでレプリケーションしないでください。

## MySQL互換性

- `BINARY`タイプのデータでJSON列を作成する場合、MySQLはデータを現在`STRING`タイプと誤って表示しますが、TiDBは正しく`BINARY`タイプとして処理します。

    ```sql
    CREATE TABLE test(a json);
    INSERT INTO test SELECT json_objectagg('a', b'01010101');

    -- TiDBでは、次のSQLステートメントを実行すると `0, 0` が返されます。MySQLでは、次のSQLステートメントを実行すると `0, 1` が返されます。
    mysql> SELECT JSON_EXTRACT(JSON_OBJECT('a', b'01010101'), '$.a') = "base64:type15:VQ==" AS r1, JSON_EXTRACT(a, '$.a') = "base64:type15:VQ==" AS r2 FROM test;
    +------+------+
    | r1   | r2   |
    +------+------+
    |    0 |    0 |
    +------+------+
    1 row in set (0.01 sec)
    ```

    詳細については、issue [#37443](https://github.com/pingcap/tidb/issues/37443)を参照してください。

- `ENUM`や`SET`から`JSON`にデータ型を変換する場合、TiDBはデータ形式の正しさをチェックします。たとえば、以下のSQLステートメントをTiDBで実行するとエラーが返されます。

    ```sql
    CREATE TABLE t(e ENUM('a'));
    INSERT INTO t VALUES ('a');
    mysql> SELECT CAST(e AS JSON) FROM t;
    ERROR 3140 (22032): Invalid JSON text: The document root must not be followed by other values.
    ```

    詳細については、issue [#9999](https://github.com/pingcap/tidb/issues/9999)を参照してください。

- TiDBでは、JSON配列やJSONオブジェクトを`ORDER BY`でソートすることができます。

    MySQLでは、JSON配列やJSONオブジェクトを`ORDER BY`でソートすると、MySQLは警告を返し、ソート結果が比較演算の結果と一致しません。

    ```sql
    CREATE TABLE t(j JSON);
    INSERT INTO t VALUES ('[1,2,3,4]');
    INSERT INTO t VALUES ('[5]');

    mysql> SELECT j FROM t WHERE j < JSON_ARRAY(5);
    +--------------+
    | j            |
    +--------------+
    | [1, 2, 3, 4] |
    +--------------+
    1 row in set (0.00 sec)

    -- TiDBでは、次のSQLステートメントを実行すると正しいソート結果が返されます。MySQLでは、次のSQLステートメントを実行すると「このMySQLのバージョンでは 'スカラでないJSON値のソート' をまだサポートしていません。」という警告が返され、ソート結果が`<`の比較結果と矛盾します。
    mysql> SELECT j FROM t ORDER BY j;
    +--------------+
    | j            |
    +--------------+
    | [1, 2, 3, 4] |
    | [5]          |
    +--------------+
    2 rows in set (0.00 sec)
    ```

    詳細については、issue [#37506](https://github.com/pingcap/tidb/issues/37506)を参照してください。

- JSON列にデータを挿入する際、TiDBはデータの値を`JSON`型に暗黙的に変換します。

    ```sql
    CREATE TABLE t(col JSON);

    -- TiDBでは、次のINSERTステートメントが正常に実行されます。MySQLでは、次のINSERTステートメントを実行すると「不正なJSONテキスト」エラーが返されます。
    INSERT INTO t VALUES (3);
    ```

`JSON`データ型に関する詳細については、[JSON functions](/functions-and-operators/json-functions.md)と[Generated Columns](/generated-columns.md)を参照してください。