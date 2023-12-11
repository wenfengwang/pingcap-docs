---
title: 制約
summary: SQLの制約がTiDBに適用される方法について学びます。
aliases: ['/docs/dev/constraints/','/docs/dev/reference/sql/constraints/']
---

# 制約

TiDBはほぼMySQLと同じ制約をサポートしています。

## NOT NULL

TiDBでサポートされているNOT NULL制約は、MySQLでサポートされているものと同じです。

例:

```sql
CREATE TABLE users (
 id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
 age INT NOT NULL,
 last_login TIMESTAMP
);
```

```sql
INSERT INTO users (id,age,last_login) VALUES (NULL,123,NOW());
```

```
Query OK, 1 row affected (0.02 sec)
```

```sql
INSERT INTO users (id,age,last_login) VALUES (NULL,NULL,NOW());
```

```
ERROR 1048 (23000): Column 'age' cannot be null
```

```sql
INSERT INTO users (id,age,last_login) VALUES (NULL,123,NULL);
```

```
Query OK, 1 row affected (0.03 sec)
```

* 最初の`INSERT`ステートメントは、`AUTO_INCREMENT`カラムに`NULL`を割り当てることが可能なため成功します。TiDBはシーケンス番号を自動的に生成します。
* 2番目の`INSERT`ステートメントは、`age`カラムが`NOT NULL`と定義されているため失敗します。
* 3番目の`INSERT`ステートメントは、`last_login`カラムが明示的に`NOT NULL`と定義されていないため成功します。NULLの値はデフォルトで許可されています。

## CHECK

> **注意:**
>
> `CHECK`制約機能はデフォルトで無効になっています。有効にするには、[`tidb_enable_check_constraint`](/system-variables.md#tidb_enable_check_constraint-new-in-v720)変数を`ON`に設定する必要があります。

`CHECK`制約は、テーブルの列の値を特定の条件を満たすように制限します。`CHECK`制約がテーブルに追加されると、TiDBはテーブルへのデータの挿入や更新時に制約が満たされているかどうかをチェックします。制約が満たされていない場合はエラーが返されます。

TiDBにおける`CHECK`制約の構文はMySQLと同じです:

```sql
[CONSTRAINT [symbol]] CHECK (expr) [[NOT] ENFORCED]
```

構文の説明:

- `[]`: `[]`内の内容はオプションです。
- `CONSTRAINT [symbol]`: `CHECK`制約の名前を指定します。
- `CHECK (expr)`: `expr`はブール式で、各行の計算結果は`TRUE`、`FALSE`、または（`NULL`の値の場合は）`UNKNOWN`のいずれかでなければなりません。行の計算結果が`FALSE`の場合、制約が違反されたことを示します。
- `[NOT] ENFORCED`: 制約のチェックを実施するかどうかを指定します。この設定を使用して`CHECK`制約を有効または無効にできます。

### `CHECK`制約の追加

TiDBでは、`CREATE TABLE`または`ALTER TABLE`ステートメントを使用して、テーブルに`CHECK`制約を追加することができます。

- `CREATE TABLE`ステートメントを使用して`CHECK`制約を追加する例:

    ```sql
    CREATE TABLE t(a INT CHECK(a > 10) NOT ENFORCED, b INT, c INT, CONSTRAINT c1 CHECK (b > c));
    ```

- `ALTER TABLE`ステートメントを使用して`CHECK`制約を追加する例:

    ```sql
    ALTER TABLE t ADD CONSTRAINT CHECK (1 < c);
    ```

`CHECK`制約を追加または有効にする際、TiDBはテーブルの既存データをチェックします。データに制約が違反している場合、`CHECK`制約の追加操作は失敗し、エラーが返されます。

`CHECK`制約を追加する際、制約名を指定するか未指定のままにすることができます。制約名を指定しない場合、TiDBは自動的に`<tableName>_chk_<1, 2, 3...>`の形式で制約名を生成します。

### `CHECK`制約の表示

[`SHOW CREATE TABLE`](/sql-statements/sql-statement-show-create-table.md)ステートメントを使用して、テーブルの制約情報を表示できます。例:

```sql
SHOW CREATE TABLE t;
+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                     |
+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  `c` int(11) DEFAULT NULL,
CONSTRAINT `c1` CHECK ((`b` > `c`)),
CONSTRAINT `t_chk_1` CHECK ((`a` > 10)) /*!80016 NOT ENFORCED */,
CONSTRAINT `t_chk_2` CHECK ((1 < `c`))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin |
+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### `CHECK`制約の削除

`CHECK`制約を削除する際は、削除する制約の名前を指定する必要があります。例:

```sql
ALTER TABLE t DROP CONSTRAINT t_chk_1;
```

### `CHECK`制約の有効化または無効化

テーブルに[CHECK`制約を追加](#add-check-constraints)する際、TiDBがデータの挿入または更新時に制約チェックを実施するかどうかを指定することができます。

- `NOT ENFORCED`が指定されている場合、TiDBはデータの挿入または更新時に制約条件をチェックしません。
- `NOT ENFORCED`が指定されておらず、または`ENFORCED`が指定されている場合、TiDBはデータの挿入または更新時に制約条件をチェックします。

制約を追加する際に`[NOT] ENFORCED`を指定する他に、`ALTER TABLE`ステートメントを使用して`CHECK`制約を有効または無効にすることもできます。例:

```sql
ALTER TABLE t ALTER CONSTRAINT c1 NOT ENFORCED;
```

### MySQLとの互換性

- 列を追加する際に`CHECK`制約を追加することはサポートされていません（例: `ALTER TABLE t ADD COLUMN a CHECK(a > 0)`）。この場合、列の追加のみが成功し、TiDBは`CHECK`制約を無視してエラーを報告しません。
- `ALTER TABLE t CHANGE a b int CHECK(b > 0)`を使用して`CHECK`制約を追加することはサポートされていません。このステートメントを実行すると、TiDBはエラーを報告します。

## UNIQUE KEY

ユニーク制約は、ユニークインデックスおよびプライマリキー列のすべての非NULL値が一意であることを意味します。

### 楽観的トランザクション

デフォルトでは、楽観的トランザクションにおいて、TiDBは一意制約を[実行フェーズ](/transaction-overview.md#lazy-check-of-constraints)で怠惰にチェックし、コミットフェーズでは厳密にチェックしています。これにより、ネットワークオーバーヘッドを削減し、パフォーマンスを向上させることができます。

例:

```sql
DROP TABLE IF EXISTS users;
CREATE TABLE users (
 id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
 username VARCHAR(60) NOT NULL,
 UNIQUE KEY (username)
);
INSERT INTO users (username) VALUES ('dave'), ('sarah'), ('bill');
```

楽観的ロックおよび`tidb_constraint_check_in_place=OFF`の場合:

```sql
BEGIN OPTIMISTIC;
INSERT INTO users (username) VALUES ('jane'), ('chris'), ('bill');
```

```
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

```sql
INSERT INTO users (username) VALUES ('steve'),('elizabeth');
```

```
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0
```

```sql
COMMIT;
```

```
ERROR 1062 (23000): Duplicate entry 'bill' for key 'users.username'
```

前述の楽観的な例では、一意制約のチェックがトランザクションがコミットされるまで遅延されました。これにより、既に`bill`という値が存在するため、重複キーのエラーが発生しました。

`tidb_constraint_check_in_place`を`ON`に設定することで、この動作を無効にすることができます。`tidb_constraint_check_in_place=ON`の場合、一意制約はステートメントの実行時にチェックされます。この変数は楽観的トランザクションにのみ適用されることに注意してください。悲観的トランザクションについては、この動作を制御するために[`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630)変数を使用できます。

例:

```sql
DROP TABLE IF EXISTS users;
CREATE TABLE users (
 id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
 username VARCHAR(60) NOT NULL,
 UNIQUE KEY (username)
);
INSERT INTO users (username) VALUES ('dave'), ('sarah'), ('bill');
```

```sql
SET tidb_constraint_check_in_place = ON;
```

```
Query OK, 0 rows affected (0.00 sec)
```

```sql
BEGIN OPTIMISTIC;
```

```
Query OK, 0 rows affected (0.00 sec)
```

```sql
INSERT INTO users (username) VALUES ('jane'), ('chris'), ('bill');
```

```
ERROR 1062 (23000): ユーザー名「bill」の重複エントリがキー「users.username」にあります。

最初の `INSERT` 文により、重複キーエラーが発生しました。これにより追加のネットワーク通信オーバーヘッドが発生し、挿入操作のスループットが低下する可能性があります。

### 悲観的トランザクション

悲観的トランザクションにおいて、デフォルトでは、TiDB では、一意制約を実行する SQL 文が実行されるとき、一意なインデックスを挿入または更新する必要があるときに、`UNIQUE` 制約を確認します。

```sql
DROP TABLE IF EXISTS users;
CREATE TABLE users (
 id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
 username VARCHAR(60) NOT NULL,
 UNIQUE KEY (username)
);
INSERT INTO users (username) VALUES ('dave'), ('sarah'), ('bill');

BEGIN PESSIMISTIC;
INSERT INTO users (username) VALUES ('jane'), ('chris'), ('bill');
```

```
ERROR 1062 (23000): ユーザー名「bill」の重複エントリがキー「users.username」にあります。
```

悲観的トランザクションの性能を向上させるためには、[`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630) 変数を `OFF` に設定することができます。これにより、TiDB は一意なインデックスの一意制約のチェックを（このインデックスがロックが必要な場合やトランザクションがコミットされるときに次回まで）保留し、対応する悲観的ロックをスキップすることができます。この変数を使用する場合は、以下に注意してください。

- 保留された一意な制約チェックにより、TiDB は一意な制約を満たさない結果を読み取り、悲観的トランザクションをコミットすると `Duplicate entry` エラーが返されることがあります。このエラーが発生すると、TiDB は現在のトランザクションをロールバックします。

    次の例では、`bill` へのロックがスキップされるため、TiDB は一意な制約を満たさない結果を取得する可能性があります。

    ```sql
    SET tidb_constraint_check_in_place_pessimistic = OFF;
    BEGIN PESSIMISTIC;
    INSERT INTO users (username) VALUES ('jane'), ('chris'), ('bill'); -- クエリは OK、3 行が影響を受けました
    SELECT * FROM users FOR UPDATE;
    ```

    以下の例の出力のように、TiDB のクエリ結果には 2 つの `bill` が含まれており、一意な制約を満たさない状態です。

    ```sql
    +----+----------+
    | id | username |
    +----+----------+
    | 1  | dave     |
    | 2  | sarah    |
    | 3  | bill     |
    | 7  | jane     |
    | 8  | chris    |
    | 9  | bill     |
    +----+----------+
    ```

    この時点で、トランザクションがコミットされると、TiDB は一意制約チェックを実行し、`Duplicate entry` エラーを報告し、トランザクションをロールバックします。

    ```sql
    COMMIT;
    ```

    ```
    ERROR 1062 (23000): ユーザー名「bill」の重複エントリがキー「users.username」にあります。
    ```

- この変数が無効にされている場合、データを書き込む必要のある悲観的トランザクションをコミットすると、`Write conflict` エラーが返されることがあります。このエラーが発生すると、TiDB は現在のトランザクションをロールバックします。

    以下の例のように、同じテーブルにデータを挿入する必要がある 2 つの並列トランザクションがある場合、悲観的ロックをスキップすると、トランザクションをコミットする際に TiDB が `Write conflict` エラーを返すことがあります。そしてトランザクションはロールバックされます。

    ```sql
    DROP TABLE IF EXISTS users;
    CREATE TABLE users (
    id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(60) NOT NULL,
    UNIQUE KEY (username)
    );

    SET tidb_constraint_check_in_place_pessimistic = OFF;
    BEGIN PESSIMISTIC;
    INSERT INTO users (username) VALUES ('jane'), ('chris'), ('bill'); -- クエリは OK、3 行が影響を受けました
    ```

    同じ時間に、別のセッションが同じテーブルに `bill` を挿入します。

    ```sql
    INSERT INTO users (username) VALUES ('bill'); -- クエリは OK、1 行が影響を受けました
    ```

    その後、最初のセッションでトランザクションをコミットすると、TiDB は `Write conflict` エラーを報告します。

    ```sql
    COMMIT;
    ```

    ```
    ERROR 9007 (HY000): Write conflict, txnStartTS=435688780611190794, conflictStartTS=435688783311536129, conflictCommitTS=435688783311536130, key={tableID=74, indexID=1, indexValues={bill, }} primary={tableID=74, indexID=1, indexValues={bill, }}, reason=LazyUniquenessCheck [try again later]
    ```

- この変数が無効にされている場合、複数の悲観的トランザクションの間で書き込み競合が発生すると、他の悲観的トランザクションがコミットされると、悲観的ロックが強制的にロールバックされ、`Pessimistic lock not found` エラーが発生することがあります。このエラーが発生すると、悲観的トランザクションに一意な制約のチェックを保留することがアプリケーションシナリオに適さないことを意味します。この場合は、競合を避けるためにアプリケーションロジックを調整するか、エラーが発生した後にトランザクションを再試行してください。

- この変数が無効にされている場合、悲観的トランザクションで DML 文を実行すると、`8147: LazyUniquenessCheckFailure` エラーが返されることがあります。

    > **注意:**
    >
    > `8147` エラーが発生すると、TiDB は現在のトランザクションをロールバックします。

    たとえば、`INSERT` 文の実行で TiDB がロックをスキップし、その後 `DELETE` 文の実行で TiDB が一意なインデックスをロックして一意な制約をチェックするため、`DELETE` 文でエラーが報告されることがあります。

    ```sql
    SET tidb_constraint_check_in_place_pessimistic = OFF;
    BEGIN PESSIMISTIC;
    INSERT INTO users (username) VALUES ('jane'), ('chris'), ('bill'); -- クエリは OK、3 行が影響を受けました
    DELETE FROM users where username = 'bill';
    ```

    ```
    ERROR 8147 (23000): transaction aborted because lazy uniqueness check is enabled and an error occurred: [kv:1062]Duplicate entry 'bill' for key 'users.username'
    ```

- この変数が無効にされている場合、`1062 Duplicate entry` エラーが現在の SQL 文からは発生していないかもしれません。そのため、複数のテーブルで同じ名前のインデックスを持つトランザクションを操作する場合には、どのインデックスからエラーが実際に発生したかを確認するために `1062` エラーメッセージを確認する必要があります。

## PRIMARY KEY

MySQL と同様に、主キー制約は一意制約を含み、すなわち、主キー制約を作成することは一意制約を持つことと同等です。さらに、TiDB の他の主キー制約も MySQL と同様です。

たとえば:

```sql
CREATE TABLE t1 (a INT NOT NULL PRIMARY KEY);
```

```
Query OK, 0 rows affected (0.12 sec)
```

```sql
CREATE TABLE t2 (a INT NULL PRIMARY KEY);
```

```
ERROR 1171 (42000): PRIMARY KEY のすべての部分は NOT NULL である必要があります; キー内で NULL が必要な場合は、UNIQUE 代わりに使用してください
```

```sql
CREATE TABLE t3 (a INT NOT NULL PRIMARY KEY, b INT NOT NULL PRIMARY KEY);
```

```
ERROR 1068 (42000): 複数の主キーが定義された
```

```sql
CREATE TABLE t4 (a INT NOT NULL, b INT NOT NULL, PRIMARY KEY (a,b));
```

```
Query OK, 0 rows affected (0.10 sec)
```

* テーブル `t2` は作成に失敗しました。なぜならば、列 `a` が主キーとして定義されており、NULL 値を許可していないためです。
* テーブル `t3` は作成に失敗しました。なぜならば、テーブルには一つの主キーしか持てないためです。
* テーブル `t4` は成功裏に作成されました。なぜならば、一意な主キーが一つしかないとしても、TiDB では複数の列を複合主キーとして定義することがサポートされているからです。

上記の規則に加えて、TiDB は現在 `NONCLUSTERED` タイプの主キーの追加と削除のみサポートしています。たとえば:

```sql
CREATE TABLE t5 (a INT NOT NULL, b INT NOT NULL, PRIMARY KEY (a,b) CLUSTERED);
ALTER TABLE t5 DROP PRIMARY KEY;
```

```
ERROR 8200 (HY000): クラスター化インデックスを使用しているテーブルの主キーを削除することはサポートされていません
```

```sql
CREATE TABLE t5 (a INT NOT NULL, b INT NOT NULL, PRIMARY KEY (a,b) NONCLUSTERED);
ALTER TABLE t5 DROP PRIMARY KEY;
```

```
Query OK, 0 rows affected (0.10 sec)
```

`CLUSTERED` タイプの主キーについての詳細については、[クラスター化インデックス](/clustered-indexes.md) を参照してください。

## FOREIGN KEY

> **注意:**
>
> v6.6.0 から、TiDB は実験的な機能として [FOREIGN KEY 制約](/foreign-key.md) をサポートします。v6.6.0 以前では、TiDB は FOREIGN KEY 制約の作成と削除をサポートしていますが、制約は実際には有効ではありません。v6.6.0 にアップグレードした後、無効な FOREIGN KEY を削除し、新しいものを作成して FOREIGN KEY 制約を有効にすることができます。

TiDB では DDL コマンドで `FOREIGN KEY` 制約を作成することがサポートされています。

たとえば:

```sql
CREATE TABLE users (
 id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
 doc JSON
);
CREATE TABLE orders (
 id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
```sql
user_id INT NOT NULL,
 doc JSON,
 FOREIGN KEY fk_user_id (user_id) REFERENCES users(id)
);
```

```sql
SELECT table_name, column_name, constraint_name, referenced_table_name, referenced_column_name
FROM information_schema.key_column_usage WHERE table_name IN ('users', 'orders');
```

```
+------------+-------------+-----------------+-----------------------+------------------------+
| table_name | column_name | constraint_name | referenced_table_name | referenced_column_name |
+------------+-------------+-----------------+-----------------------+------------------------+
| users      | id          | PRIMARY         | NULL                  | NULL                   |
| orders     | id          | PRIMARY         | NULL                  | NULL                   |
| orders     | user_id     | fk_user_id      | users                 | id                     |
+------------+-------------+-----------------+-----------------------+------------------------+
3 rows in set (0.00 sec)
```

TiDB は `ALTER TABLE` コマンドを使って `DROP FOREIGN KEY` と `ADD FOREIGN KEY` の構文もサポートしています。

```sql
ALTER TABLE orders DROP FOREIGN KEY fk_user_id;
ALTER TABLE orders ADD FOREIGN KEY fk_user_id (user_id) REFERENCES users(id);
```