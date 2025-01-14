---
title：`tidb_read_staleness`システム変数を使用した履歴データの読み込み
summary：`tidb_read_staleness`システム変数を使用して履歴データを読み込む方法について学びます。

# `tidb_read_staleness`システム変数を使用した履歴データの読み込み

v5.4 で、履歴データの読み込みをサポートするために、TiDBは新しい`tidb_read_staleness`システム変数を導入しています。このドキュメントでは、このシステム変数を使用して履歴データを読み込む方法について詳細な手順が記載されています。

## 機能の説明

`tidb_read_staleness`システム変数は、現在のセッションでTiDBが読み取れる履歴データの時間範囲を設定するために使用されます。この変数のデータ型は int 型であり、そのスコープは`SESSION`です。値を設定すると、TiDB はこの変数によって許可される範囲から、できるだけ新しいタイムスタンプを選択し、その以降のすべての読み取り操作はこのタイムスタンプに対して実行されます。たとえば、この変数の値が`-5`に設定されている場合、TiKVに対応する過去のバージョンのデータがある状況下で、TiDB は 5 秒以内の範囲内でできるだけ新しいタイムスタンプを選択します。

`tidb_read_staleness`を有効にした後も、次の操作を実行することができます：

- 現在のセッションでデータの挿入、変更、削除、または DML 操作を行う。これらのステートメントは`tidb_read_staleness`の影響を受けません。
- 現在のセッションでインタラクティブトランザクションを開始する。これらのトランザクション内のクエリは依然として最新のデータを読み取ります。

履歴データを読み取った後、最新のデータを次の2つの方法で読み取ることができます：

- 新しいセッションを開始する。
- `SET`ステートメントを使用して`tidb_read_staleness`変数の値を`""`に設定する。

> **注意：**
>
> ステールリードデータの待ち時間を短縮し、タイムリネスを向上させるためには、TiKVの`advance-ts-interval`構成項目を変更することができます。詳細については、[ステールリードの待ち時間を短縮する](/stale-read.md#reduce-stale-read-latency)を参照してください。

## 使用例

このセクションでは、例を使用して`tidb_read_staleness`の使用方法について説明します。

1. テーブルを作成し、テーブルに数行のデータを挿入します：

    {{< copyable "sql" >}}

    ```sql
    create table t (c int);
    ```

    ```
    Query OK, 0 rows affected (0.01 sec)
    ```

    {{< copyable "sql" >}}

    ```sql
    insert into t values (1), (2), (3);
    ```

    ```
    Query OK, 3 rows affected (0.00 sec)
    ```

2. テーブル内のデータを確認します：

    {{< copyable "sql" >}}

    ```sql
    select * from t;
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

3. 1行のデータを更新します：

    {{< copyable "sql" >}}

    ```sql
    update t set c=22 where c=2;
    ```

    ```
    Query OK, 1 row affected (0.00 sec)
    ```

4. データが更新されたことを確認します：

    {{< copyable "sql" >}}

    ```sql
    select * from t;
    ```

    ```
    +------+
    | c    |
    +------+
    |    1 |
    |   22 |
    |    3 |
    +------+
    3 rows in set (0.00 sec)
    ```

5. `tidb_read_staleness`システム変数の値を設定します。

    この変数のスコープは`SESSION`です。その値を設定した後、TiDB はその値に設定された時間よりも前の最新バージョンデータを読み取ります。

    以下の設定は、TiDB が 5 秒前から現在までの時間範囲内でできるだけ新しいタイムスタンプを選択し、その時点を履歴データのタイムスタンプとして使用することを意味します：

    {{< copyable "sql" >}}

    ```sql
    set @@tidb_read_staleness="-5";
    ```

    ```
    Query OK, 0 rows affected (0.00 sec)
    ```

    > **注意：**
    >
    > - `tidb_read_staleness`の前に`@`ではなく`@@`を使用します。`@@`はシステム変数を意味し、`@`はユーザー変数を意味します。
    > - ステップ3とステップ4で費やした時間に応じて、履歴データを表示するために履歴時間範囲（`tidb_read_staleness`の値）を設定する必要があります。そうしない場合、クエリの結果には最新のデータが表示されます。したがって、操作に費やした時間に応じてこの時間範囲を調整する必要があります。たとえば、この例では、設定した時間範囲が5秒なので、ステップ3とステップ4を5秒以内に完了する必要があります。

    ここで読み取られるデータは、更新前のデータであり、つまり履歴データです：

    {{< copyable "sql" >}}

    ```sql
    select * from t;
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

6. 次のようにこの変数をunsetすると、TiDBは最新のデータを読み込めます：

    {{< copyable "sql" >}}

    ```sql
    set @@tidb_read_staleness="";
    ```

    ```
    Query OK, 0 rows affected (0.00 sec)
    ```

    {{< copyable "sql" >}}

    ```sql
    select * from t;
    ```

    ```
    +------+
    | c    |
    +------+
    |    1 |
    |   22 |
    |    3 |
    +------+
    3 rows in set (0.00 sec)
    ```