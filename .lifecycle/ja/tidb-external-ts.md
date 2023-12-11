---
title: `tidb_external_ts`変数を使用して履歴データを読む
summary: `tidb_external_ts`変数を使用して履歴データを読む方法について学ぶ
---

# `tidb_external_ts`変数を使用して履歴データを読む

履歴データの読み込みをサポートするために、TiDB v6.4.0 はシステム変数 [`tidb_external_ts`](/system-variables.md#tidb_external_ts-new-in-v640) を導入しました。このドキュメントでは、このシステム変数を通じて履歴データを読む方法について、詳細な使用例を含めて説明します。

## シナリオ

特定の時点からの履歴データの読み込みは、TiCDCなどのデータレプリケーションツールにとって非常に役立ちます。データレプリケーションツールが特定の時点までのデータレプリケーションを完了した後、下流のTiDBの`tidb_external_ts`システム変数を設定することで、その時点より前のデータを読むことができます。これにより、データレプリケーションによるデータ不整合が防止されます。

## 機能の説明

システム変数 [`tidb_external_ts`](/system-variables.md#tidb_external_ts-new-in-v640) は、`tidb_enable_external_ts_read` が有効になっている場合に読まれる履歴データのタイムスタンプを指定します。

システム変数 [`tidb_enable_external_ts_read`](/system-variables.md#tidb_enable_external_ts_read-new-in-v640) は、現在のセッションまたはグローバルで履歴データを読むかどうかを制御します。デフォルト値は `OFF` であり、つまり履歴データを読む機能は無効になり、`tidb_external_ts` の値は無視されます。`tidb_enable_external_ts_read` がグローバルで `ON` に設定されると、全てのクエリは `tidb_external_ts` で指定された時点より前の履歴データを読み込みます。特定のセッションだけが `ON` に設定されている場合、そのセッション内のクエリだけが履歴データを読み込みます。

`tidb_enable_external_ts_read` が有効になると、TiDBは読み込み専用になります。全ての書き込みクエリは `ERROR 1836 (HY000): Running in read-only mode` のようなエラーで失敗します。

## 使用例

このセクションでは、例を使用して `tidb_external_ts`変数を使用して履歴データを読む方法について説明します。

1. テーブルを作成し、テーブルにいくつかの行を挿入します:

    ```sql
    CREATE TABLE t (c INT);
    ```

    ```
    クエリが正常終了しました(0.01 sec)
    ```

    ```sql
    INSERT INTO t VALUES (1), (2), (3);
    ```

    ```
    クエリが正常終了しました(0.00 sec)
    ```

2. テーブルのデータを表示します:

    ```sql
    SELECT * FROM t;
    ```

    ```
    +------+
    | c    |
    +------+
    |    1 |
    |    2 |
    |    3 |
    +------+
    3 rows in set (0.00 sec)
    ```

3. `tidb_external_ts`を `@@tidb_current_ts`に設定します:

    ```sql
    START TRANSACTION;
    SET GLOBAL tidb_external_ts=@@tidb_current_ts;
    COMMIT;
    ```

4. 新しい行を挿入し、挿入されたことを確認します:

    ```sql
    INSERT INTO t VALUES (4);
    ```

    ```
    クエリが正常終了しました(0.001 sec)
    ```

    ```sql
    SELECT * FROM t;
    ```

    ```
    +------+
    | id   |
    +------+
    |    1 |
    |    2 |
    |    3 |
    |    4 |
    +------+
    4 rows in set (0.00 sec)
    ```

5. `tidb_enable_external_ts_read`を `ON`に設定し、その後テーブルのデータを表示します:

    ```sql
    SET tidb_enable_external_ts_read=ON;
    SELECT * FROM t;
    ```

    ```
    +------+
    | c    |
    +------+
    |    1 |
    |    2 |
    |    3 |
    +------+
    3 rows in set (0.00 sec)
    ```

    `tidb_external_ts` が新しい行が挿入される前のタイムスタンプに設定されているため、`tidb_enable_external_ts_read`が有効になっても、新しく挿入された行は返されません。