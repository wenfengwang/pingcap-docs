---
title: シーケンスの作成
summary: TiDBデータベースでCREATE SEQUENCEの使用方法の概要
aliases: ['/docs/dev/sql-statements/sql-statement-create-sequence/','/docs/dev/reference/sql/statements/create-sequence/']
---

# CREATE SEQUENCE

`CREATE SEQUENCE` ステートメントは、TiDBでシーケンスオブジェクトを作成します。シーケンスは、テーブルと `View` オブジェクトと同じくらい重要なデータベースオブジェクトです。シーケンスは、カスタマイズされた方法で直列化されたIDを生成するために使用されます。

## シノプシス

```ebnf+diagram
CreateSequenceStmt ::=
    'CREATE' 'SEQUENCE' IfNotExists TableName CreateSequenceOptionListOpt CreateTableOptionListOpt

IfNotExists ::=
    ('IF' 'NOT' 'EXISTS')?

TableName ::=
    Identifier ('.' Identifier)?

CreateSequenceOptionListOpt ::=
    SequenceOption*

SequenceOptionList ::=
    SequenceOption

SequenceOption ::=
    ( 'INCREMENT' ( '='? | 'BY' ) | 'START' ( '='? | 'WITH' ) | ( 'MINVALUE' | 'MAXVALUE' | 'CACHE' ) '='? ) SignedNum
|   'NOMINVALUE'
|   'NO' ( 'MINVALUE' | 'MAXVALUE' | 'CACHE' | 'CYCLE' )
|   'NOMAXVALUE'
|   'NOCACHE'
|   'CYCLE'
|   'NOCYCLE'
```

## 構文

{{< copyable "sql" >}}

```sql
CREATE [TEMPORARY] SEQUENCE [IF NOT EXISTS] シーケンス名
    [ INCREMENT [ BY | = ] 増分 ]
    [ MINVALUE [=] 最小値 | NO MINVALUE | NOMINVALUE ]
    [ MAXVALUE [=] 最大値 | NO MAXVALUE | NOMAXVALUE ]
    [ START [ WITH | = ] 開始値 ]
    [ CACHE [=] キャッシュ | NOCACHE | NO CACHE]
    [ CYCLE | NOCYCLE | NO CYCLE]
    [table_options]
```

## パラメータ

|パラメータ | デフォルト値 | 説明 |
| :-- | :-- | :--|
| `TEMPORARY` | `false` | TiDBは現在 `TEMPORARY` オプションをサポートしておらず、構文の互換性のためにのみ提供しています。 |
| `INCREMENT` | `1` | シーケンスの増分を指定します。その正負の値でシーケンスの増加方向を制御できます。 |
| `MINVALUE` | `1` または `-9223372036854775807` | シーケンスの最小値を指定します。`INCREMENT` > `0` のとき、デフォルト値は `1` です。 `INCREMENT` < `0` のとき、デフォルト値は `-9223372036854775807` です。 |
| `MAXVALUE` | `9223372036854775806` または `-1` | シーケンスの最大値を指定します。`INCREMENT` > `0` のとき、デフォルト値は `9223372036854775806` です。 `INCREMENT` < `0` のとき、デフォルト値は `-1` です。 |
| `START` | `MINVALUE` または `MAXVALUE`| シーケンスの初期値を指定します。`INCREMENT` > `0` のとき、デフォルト値は `MINVALUE` です。 `INCREMENT` < `0` のとき、デフォルト値は `MAXVALUE` です。 |
| `CACHE` | `1000` | TiDBでシーケンスの局所的なキャッシュサイズを指定します。 |
| `CYCLE` | `NO CYCLE` | シーケンスが最小値（または降順シーケンスの場合は最大値）から再起動するかどうかを指定します。`INCREMENT` > `0` のとき、デフォルト値は `MINVALUE` です。 `INCREMENT` < `0` のとき、デフォルト値は `MAXVALUE` です。 |

## `SEQUENCE` 関数

以下の式関数を使用してシーケンスを制御できます。

+ `NEXTVAL` または `NEXT VALUE FOR`

    どちらも、シーケンスオブジェクトの次の有効な値を取得する `nextval()` 関数です。 `nextval()` 関数のパラメータはシーケンスの `identifier` です。

+ `LASTVAL`

    この関数は、このセッションで前回にシーケンスオブジェクトによって生成された値を取得します。値が存在しない場合は `NULL` が使用されます。この関数のパラメータはシーケンスの `identifier` です。

+ `SETVAL`

    この関数は、シーケンスの現在の値の進行を設定します。この関数の第一パラメータはシーケンスの `identifier` で、第二パラメータは `num` です。

> **注記:**
>
> TiDBでのシーケンスの実装において `SETVAL` 関数は、このシーケンスの初期進行または周期進行を変更することはできません。この関数は、この進行に基づいて次の有効な値を返すだけです。

## 例

+ デフォルトのパラメータでシーケンスオブジェクトを作成します:

    {{< copyable "sql" >}}

    ```sql
    CREATE SEQUENCE seq;
    ```

    ```
    クエリは成功しました。変更された行: 0 (0.06 秒)
    ```

+ `nextval()` 関数を使用して、シーケンスオブジェクトの次の値を取得します:

    {{< copyable "sql" >}}

    ```sql
    SELECT nextval(seq);
    ```

    ```
    +--------------+
    | nextval(seq) |
    +--------------+
    |            1 |
    +--------------+
    1 行が返されました (0.02 秒)
    ```

+ `lastval()` 関数を使用して、このセッションでシーケンスオブジェクトによって生成された値を取得します:

    {{< copyable "sql" >}}

    ```sql
    SELECT lastval(seq);
    ```

    ```
    +--------------+
    | lastval(seq) |
    +--------------+
    |            1 |
    +--------------+
    1 行が返されました (0.02 秒)
    ```

+ `setval()` 関数を使用して、シーケンスオブジェクトの現在の値（または現在の位置）を設定します:

    {{< copyable "sql" >}}

    ```sql
    SELECT setval(seq, 10);
    ```

    ```
    +-----------------+
    | setval(seq, 10) |
    +-----------------+
    |              10 |
    +-----------------+
    1 行が返されました (0.01 秒)
    ```

+ また、`next value for` 文法を使用して、シーケンスの次の値を取得できます:

    {{< copyable "sql" >}}

    ```sql
    SELECT next value for seq;
    ```

    ```
    +--------------------+
    | next value for seq |
    +--------------------+
    |                 11 |
    +--------------------+
    1 行が返されました (0.00 秒)
    ```

+ デフォルトのカスタムパラメータを使用して、シーケンスオブジェクトを作成します:

    {{< copyable "sql" >}}

    ```sql
    CREATE SEQUENCE seq2 start 3 increment 2 minvalue 1 maxvalue 10 cache 3;
    ```

    ```
    クエリは成功しました。変更された行: 0 (0.01 秒)
    ```

+ このセッションでシーケンスオブジェクトが使用されていない場合、`lastval()` 関数は `NULL` の値を返します。

    {{< copyable "sql" >}}

    ```sql
    SELECT lastval(seq2);
    ```

    ```
    +---------------+
    | lastval(seq2) |
    +---------------+
    |          NULL |
    +---------------+
    1 行が返されました (0.01 秒)
    ```

  + シーケンスオブジェクトの `nextval()` 関数の最初の有効な値は `START` パラメータの値です。

    {{< copyable "sql" >}}

    ```sql
    SELECT nextval(seq2);
    ```

    ```
    +---------------+
    | nextval(seq2) |
    +---------------+
    |             3 |
    +---------------+
    1 行が返されました (0.00 秒)
    ```

  + `setval()` 関数はシーケンスオブジェクトの現在の値を変更できますが、次の値の算術進行ルールを変更することはできません。

    {{< copyable "sql" >}}

    ```sql
    SELECT setval(seq2, 6);
    ```

    ```
    +-----------------+
    | setval(seq2, 6) |
    +-----------------+
    |               6 |
    +-----------------+
    1 行が返されました (0.00 秒)
    ```

  + `nextval()` を使用して次の値を取得すると、次の値はシーケンスによって定義された算術進行ルールに従います。

    {{< copyable "sql" >}}

    ```sql
    SELECT next value for seq2;
    ```

    ```
    +---------------------+
    | next value for seq2 |
    +---------------------+
    |                   7 |
    +---------------------+
    1 行が返されました (0.00 秒)
    ```

  + 次の例のように、シーケンスの次の値を列のデフォルト値として使用できます。

    {{< copyable "sql" >}}

    ```sql
    CREATE table t(a int default next value for seq2);
    ```

    ```
    クエリは成功しました。変更された行: 0 (0.02 秒)
    ```

  + 次の例のように、値が指定されていない場合、`seq2` のデフォルト値が使用されます。

    {{< copyable "sql" >}}

    ```sql
    INSERT into t values();
    ```

    ```
    クエリは成功しました。変更された行: 1 (0.00 秒)
    ```

    {{< コピー可能 "sql" >}}
    
    ```sql
    SELECT * from t;
    ```
    
    ```
    +------+
    | a    |
    +------+
    |    9 |
    +------+
    1 行がセットされました (0.00 秒)
    ```

+ 次の例では、値が指定されていないため、`seq2`のデフォルト値が使用されます。しかし、`seq2`の次の値が上記の例で定義された範囲内にないため (`CREATE SEQUENCE seq2 start 3 increment 2 minvalue 1 maxvalue 10 cache 3;`)、エラーが返されます。

    {{< コピー可能 "sql" >}}

    ```sql
    INSERT into t values();
    ```

    ```
    ERROR 4135 (HY000): Sequence 'test.seq2' has run out
    ```

## MySQL 互換性

この文はTiDBの拡張機能です。実装はMariaDBで利用可能なシーケンスをモデルとしています。

`SETVAL` 関数を除き、他のすべての関数はMariaDBと同じ _progressions_ を持っています。ここでいう「progression」は、シーケンス内の数値がシーケンスによって定義された特定の等差数列に従うことを意味します。`SETVAL` を使用してシーケンスの現在の値を設定できますが、シーケンスの後続の値は依然として元の進行規則に従います。

例:

```
1, 3, 5, ...            // シーケンスは1から始まり、2ずつ増加します。
select setval(seq, 6)   // シーケンスの現在の値を6に設定します。
7, 9, 11, ...           // 後続の値は依然として進行規則に従います。
```

`CYCLE` モードでは、最初のラウンドでのシーケンスの初期値は `START` パラメータの値であり、次回以降のラウンドの初期値は `MinValue` (`INCREMENT` > 0) または `MaxValue` (`INCREMENT` < 0) の値です。

## 関連情報

* [DROP SEQUENCE](/sql-statements/sql-statement-drop-sequence.md)
* [SHOW CREATE SEQUENCE](/sql-statements/sql-statement-show-create-sequence.md)