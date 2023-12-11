---
title: SQL Plan Management (SPM)
summary: TiDBのSQLプラン管理について学ぶ。
aliases: ['/docs/dev/sql-plan-management/','/docs/dev/reference/performance/execution-plan-bind/','/docs/dev/execution-plan-binding/']
---

# SQLプラン管理（SPM）

SQLプラン管理（SPM）は、SQLバインディングを実行してSQLの実行プランに手動で干渉するための一連の機能です。これらの機能には、SQLバインディング、ベースラインのキャプチャ、ベースラインの進化が含まれます。

## SQLバインディング

SQLバインディングはSPMの基礎です。[オプティマイザーヒント](/optimizer-hints.md)ドキュメントでは、ヒントを使用して特定の実行プランを選択する方法について説明しています。ただし、SQLステートメントを変更せずに実行選択に干渉する必要がある場合もあります。SQLバインディングを使用すると、SQLステートメントを変更することなく指定の実行プランを選択できます。

<CustomContent platform="tidb">

> **注意:**
>
> SQLバインディングを使用するには、`SUPER`権限が必要です。TiDBが必要な権限を持っていないと表示される場合は、[権限管理](/privilege-management.md)を参照して必要な権限を追加してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> SQLバインディングを使用するには、`SUPER`権限が必要です。TiDBが必要な権限を持っていないと表示される場合は、[権限管理](https://docs.pingcap.com/tidb/stable/privilege-management)を参照して必要な権限を追加してください。

</CustomContent>

### バインディングの作成

SQLステートメントまたは過去の実行プランに従って、SQLステートメントのバインディングを作成できます。

#### SQLステートメントに従ってバインディングを作成する

{{< copyable "sql" >}}

```sql
CREATE [GLOBAL | SESSION] BINDING FOR BindableStmt USING BindableStmt
```

このステートメントは、`GLOBAL`または`SESSION`レベルでSQLの実行プランにバインドします。TiDBでサポートされているバインド可能なSQLステートメント（BindableStmt）には、`SELECT`、`DELETE`、`UPDATE`、および`INSERT` / `REPLACE`のサブクエリを含む`SELECT`があります。

> **注意:**
>
> バインディングは手動で追加されたヒントよりも優先されます。そのため、対応するバインディングが存在する場合にヒントを含むステートメントを実行すると、最適化プランの動作を制御するヒントは効果を持ちません。ただし、他の種類のヒントは引き続き有効です。

具体的には、次のような構文の衝突があるため、これらのステートメントの実行プランにはバインドできません。

```sql
-- タイプ1: `JOIN`キーワードを使用してデカルト積を取得し、`USING`キーワードで関連する列を指定しないステートメント。
CREATE GLOBAL BINDING for
    SELECT * FROM t t1 JOIN t t2
USING
    SELECT * FROM t t1 JOIN t t2;

-- タイプ2: `USING`キーワードを含む`DELETE`ステートメント。
CREATE GLOBAL BINDING for
    DELETE FROM t1 USING t1 JOIN t2 ON t1.a = t2.a
USING
    DELETE FROM t1 USING t1 JOIN t2 ON t1.a = t2.a;
```

等価なステートメントを使用して構文の衝突を回避できます。たとえば、次のような方法で上記のステートメントを書き直すことができます。

```sql
-- タイプ1ステートメントの最初の書き換え: `JOIN`キーワードに`USING`句を追加します。
CREATE GLOBAL BINDING for
    SELECT * FROM t t1 JOIN t t2 USING (a)
USING
    SELECT * FROM t t1 JOIN t t2 USING (a);

-- タイプ1ステートメントの2番目の書き換え: `JOIN`キーワードを削除します。
CREATE GLOBAL BINDING for
    SELECT * FROM t t1, t t2
USING
    SELECT * FROM t t1, t t2;

-- タイプ2ステートメントの書き換え: `DELETE`ステートメントから`USING`キーワードを削除します。
CREATE GLOBAL BINDING for
    DELETE t1 FROM t1 JOIN t2 ON t1.a = t2.a
using
    DELETE t1 FROM t1 JOIN t2 ON t1.a = t2.a;
```

> **注意:**
>
> `INSERT` / `REPLACE`ステートメントに`SELECT`サブクエリを持つ実行プランバインディングを作成する場合、`INSERT` / `REPLACE`キーワードの後でバインドしたい最適化ヒントを指定する必要があります。そうしないと、最適化ヒントは意図したとおりに効果を持ちません。

次の例をご覧ください。

```sql
-- 次のステートメントでヒントが効果を発揮します。
CREATE GLOBAL BINDING for
    INSERT INTO t1 SELECT * FROM t2 WHERE a > 1 AND b = 1
using
    INSERT INTO t1 SELECT /*+ use_index(@sel_1 t2, a) */ * FROM t2 WHERE a > 1 AND b = 1;

-- 次のステートメントではヒントが効果を発揮しません。
CREATE GLOBAL BINDING for
    INSERT INTO t1 SELECT * FROM t2 WHERE a > 1 AND b = 1
using
    INSERT /*+ use_index(@sel_1 t2, a) */ INTO t1 SELECT * FROM t2 WHERE a > 1 AND b = 1;
```

実行計画バインディングを作成する際にスコープを指定しない場合、デフォルトのスコープはSESSIONになります。TiDB最適化プランは、バインディングされたSQLステートメントを正規化し、システムテーブルに保存します。SQLクエリの処理中に、正規化されたステートメントがシステムテーブル内のバインドされたSQLステートメントのいずれかに一致し、システム変数`tidb_use_plan_baselines`が`on`に設定されている場合（デフォルト値は`on`）、TiDBはこのステートメントに対して対応する最適化ヒントを使用します。一致する実行プランが複数ある場合、最適化プランはコストの少ないものをバインドします。

`正規化`は、SQLステートメントの定数を変数パラメータに変換し、クエリ内で参照されるテーブルのデータベースを明示的に指定し、SQLステートメント内のスペースと改行を標準化するプロセスです。次の例をご覧ください。

```sql
SELECT * FROM t WHERE a >    1
-- 正規化後、上記のステートメントは次のようになります:
SELECT * FROM test . t WHERE a > ?
```

> **注意:**
>
> 正規化プロセスでは、`IN`述語内の`?`は`...`に正規化されます。
>
> 例:
>
> ```sql
> SELECT * FROM t WHERE a IN (1)
> SELECT * FROM t WHERE a IN (1,2,3)
> -- 正規化後、上記のステートメントは次のようになります:
> SELECT * FROM test . t WHERE a IN ( ... )
> SELECT * FROM test . t WHERE a IN ( ... )
> ```
>
> 正規化後、異なる長さの`IN`述語が同じステートメントとして認識されるため、これらの述語全体に適用される1つのバインディングを作成するだけでよくなります。
>
> 例:
>
> ```sql
> CREATE TABLE t (a INT, KEY(a));
> CREATE BINDING FOR SELECT * FROM t WHERE a IN (?) USING SELECT /*+ use_index(t, a) */ * FROM t WHERE a in (?);
> 
> SELECT * FROM t WHERE a IN (1);
> SELECT @@LAST_PLAN_FROM_BINDING;
> +--------------------------+
> | @@LAST_PLAN_FROM_BINDING |
> +--------------------------+
> |                        1 |
> +--------------------------+
>
> SELECT * FROM t WHERE a IN (1, 2, 3);
> SELECT @@LAST_PLAN_FROM_BINDING;
> +--------------------------+
> | @@LAST_PLAN_FROM_BINDING |
> +--------------------------+
> |                        1 |
> +--------------------------+
> ```
>
> v7.4.0より前のTiDBクラスターで作成されたバインディングには、`IN (?)`が含まれる場合があります。v7.4.0以降のバージョンにアップグレードすると、これらのバインディングは`IN (...)`に変更されます。
>
> 例:
>
> ```sql
> -- v7.3.0でバインディングを作成する
> mysql> CREATE GLOBAL BINDING FOR SELECT * FROM t WHERE a IN (1) USING SELECT /*+ use_index(t, a) */ * FROM t WHERE a IN (1);
> mysql> SHOW GLOBAL BINDINGS;
> +-----------------------------------------------+--------------------------------------------------------------------+------------+---------+-------------------------+-------------------------+---------+-----------------+--------+------------------------------------------------------------------+-------------+
> | Original_sql                                  | Bind_sql                                                           | Default_db | Status  | Create_time             | Update_time             | Charset | Collation       | Source | Sql_digest                                                       | Plan_digest |
> +-----------------------------------------------+--------------------------------------------------------------------+------------+---------+-------------------------+-------------------------+---------+-----------------+--------+------------------------------------------------------------------+-------------+
> | select * from `test` . `t` where `a` in ( ? ) | SELECT /*+ use_index(`t` `a`)*/ * FROM `test`.`t` WHERE `a` IN (1) | test       | enabled | 2023-10-20 14:28:10.093 | 2023-10-20 14:28:10.093 | utf8    | utf8_general_ci | manual | 8b9c4e6ab8fad5ba29b034311dcbfc8a8ce57dde2e2d5d5b65313b90ebcdebf7 |             |
> +-----------------------------------------------+--------------------------------------------------------------------+------------+---------+-------------------------+-------------------------+---------+-----------------+--------+------------------------------------------------------------------+-------------+
>
> -- v7.4.0またはそれ以降のバージョンにアップグレード後
> mysql> SHOW GLOBAL BINDINGS;
> +-------------------------------------------------+--------------------------------------------------------------------+------------+---------+-------------------------+-------------------------+---------+-----------------+--------+------------------------------------------------------------------+-------------+
```
> | Original_sql                                    | Bind_sql                                                           | Default_db | Status  | Create_time             | Update_time             | Charset | Collation       | Source | Sql_digest                                                       | Plan_digest |
> +-------------------------------------------------+--------------------------------------------------------------------+------------+---------+-------------------------+-------------------------+---------+-----------------+--------+------------------------------------------------------------------+-------------+
> | `test` . `t` の `a` が ( ... ) である場合の `test` . `t` のすべてを選択 | SELECT /*+ use_index(`t` `a`)*/ * FROM `test`.`t` WHERE `a` IN (1) | test       | enabled | 2023-10-20 14:28:10.093 | 2023-10-20 14:28:10.093 | utf8    | utf8_general_ci | manual | 8b9c4e6ab8fad5ba29b034311dcbfc8a8ce57dde2e2d5d5b65313b90ebcdebf7 |             |
> ```

SQLステートメントにGLOBALおよびSESSIONスコープのバインド実行プランがあり、セッションバインディングが存在する場合、最適化プログラムはセッションバインディングが遭遇された際にGLOBALスコープのバインド実行プランを無視するため、このステートメントのバインド実行プランがSESSIONスコープにあるバインド実行プランに対して優先します。

例：

```sql
-- GLOBALバインドを作成し、このバインドで`sort merge join`を使用するように指定します。
CREATE GLOBAL BINDING for
    SELECT * FROM t1, t2 WHERE t1.id = t2.id
USING
    SELECT /*+ merge_join(t1, t2) */ * FROM t1, t2 WHERE t1.id = t2.id;

-- このSQLステートメントの実行計画はGLOBALバインドで指定された`sort merge join`を使用します。
explain SELECT * FROM t1, t2 WHERE t1.id = t2.id;

-- 別のSESSIONバインドを作成し、このバインドで`hash join`を使用するように指定します。
CREATE BINDING for
    SELECT * FROM t1, t2 WHERE t1.id = t2.id
USING
    SELECT /*+ hash_join(t1, t2) */ * FROM t1, t2 WHERE t1.id = t2.id;

-- このステートメントの実行計画では、GLOBALバインドで指定された`sort merge join`の代わりにSESSIONバインドで指定された`hash join`が使用されます。
explain SELECT * FROM t1, t2 WHERE t1.id = t2.id;
```

最初の`SELECT`ステートメントが実行されているとき、最適化プログラムはGLOBALスコープのバインディングを介してステートメントに`sm_join(t1, t2)`ヒントを追加します。`explain`結果の実行計画のトップノードはMergeJoinです。2番目の`SELECT`ステートメントが実行されているとき、最適化プログラムはGLOBALスコープのバインディングではなくSESSIONスコープのバインディングを使用し、ステートメントに`hash_join(t1, t2)`ヒントを追加します。`explain`結果の実行計画のトップノードはHashJoinです。

標準化されたSQLステートメントごとに、`CREATE BINDING`を使用して作成されるバインディングは1つのみです。同じ標準SQLステートメントに対して複数のバインディングを作成すると、最後に作成されたバインディングが保持され、すべての以前のバインディング（作成および進化）は削除されます。ただし、セッションバインディングとグローバルバインディングは共存し、このロジックの影響を受けません。

さらに、バインディングを作成する際、TiDBはクライアントが接続されている場合、または`use ${database}`が実行された場合、セッションがデータベースコンテキストにあることを要求します。

元のSQLステートメントとバウンドステートメントは、正規化およびヒントの削除後に同じテキストを持たなければならず、そうでない場合はバインディングが失敗します。次の例をご覧ください。

- パラメータ化とヒントの除去前後のテキストが同じであるため、このバインディングは正常に作成されます：`SELECT * FROM test . t WHERE a > ?`

    ```sql
    CREATE BINDING FOR SELECT * FROM t WHERE a > 1 USING SELECT * FROM t use index  (idx) WHERE a > 2
    ```

- バインドSQLステートメントは`SELECT * FROM test . t WHERE b > ?`とは異なりますが、元のSQLステートメントは`SELECT * FROM test . t WHERE a > ?`と処理されているため、このバインディングは失敗します。

    ```sql
    CREATE BINDING FOR SELECT * FROM t WHERE a > 1 USING SELECT * FROM t use index(idx) WHERE b > 2
    ```

      BINARY_PLAN: 6QOYCuQDCg1UYWJsZVJlYWRlcl83Ev8BCgtTZWxlY3Rpb25fNhKOAQoPBSJQRnVsbFNjYW5fNSEBAAAAOA0/QSkAAQHwW4jDQDgCQAJKCwoJCgR0ZXN0EgF0Uh5rZWVwIG9yZGVyOmZhbHNlLCBzdGF0czpwc2V1ZG9qInRpa3ZfdGFzazp7dGltZTo1NjAuOMK1cywgbG9vcHM6MH1w////CQMEAXgJCBD///8BIQFzCDhVQw19BAAkBX0QUg9lcSgBfCAudC5hLCAxKWrmYQAYHOi0gc6hBB1hJAFAAVIQZGF0YTo9GgRaFAW4HDQuMDVtcywgCbYcMWKEAWNvcF8F2agge251bTogMSwgbWF4OiA1OTguNsK1cywgcHJvY19rZXlzOiAwLCBycGNfBSkAMgkMBVcQIDYwOS4pEPBDY29wcl9jYWNoZV9oaXRfcmF0aW86IDAuMDAsIGRpc3RzcWxfY29uY3VycmVuY3k6IDE1fXCwAXj///////////8BGAE=

    この例では、`plan_digest` に対応する実行プランが `4e3159169cc63c14b139a4e7d72eae1759875c9a9581f94bb2079aae961189cb` であることがわかります。

2. `plan_digest` を使用してバインディングを作成します:

    ```sql
    CREATE BINDING FROM HISTORY USING PLAN DIGEST '4e3159169cc63c14b139a4e7d72eae1759875c9a9581f94bb2079aae961189cb';
    ```

作成したバインディングが有効かどうかを確認するには、[バインディングを表示](#view-bindings)できます:

```sql
SHOW BINDINGS\G;
```

```
*************************** 1. 行 ***************************
Original_sql: select * from `test` . `t` where `a` = ?
    Bind_sql: SELECT /*+ use_index(@`sel_1` `test`.`t` ) ignore_index(`t` `a`)*/ * FROM `test`.`t` WHERE `a` = 1
       ...........
  Sql_digest: 6909a1bbce5f64ade0a532d7058dd77b6ad5d5068aee22a531304280de48349f
 Plan_digest:
1 行 in set (0.01 秒)

ERROR:
クエリーが指定されていません
```

```sql
SELECT * FROM t WHERE a = 1;
SELECT @@LAST_PLAN_FROM_BINDING;
```

```
+--------------------------+
| @@LAST_PLAN_FROM_BINDING |
+--------------------------+
|                        1 |
+--------------------------+
1 行 in set (0.00 秒)
```

### バインディングを削除する

SQL ステートメントまたは `sql_digest` に従ってバインディングを削除できます。

#### SQL ステートメントに従ってバインディングを削除する

{{< copyable "sql" >}}

```sql
DROP [GLOBAL | SESSION] BINDING FOR BindableStmt;
```

このステートメントは、指定された実行計画バインディングを GLOBAL または SESSION レベルで削除します。デフォルトのスコープは SESSION です。

通常、SESSION スコープのバインディングはテストや特別な状況で主に使用されます。バインディングをすべての TiDB プロセスで有効にするには、GLOBAL バインディングを使用する必要があります。作成した SESSION バインディングは SESSION の終了まで、対応する GLOBAL バインディングを遮断します。この場合、セッションが終了する前に SESSION バインディングが削除されても、バインディングは有効にならず、プランはオプティマイザによって選択されます。

次の例は、[バインディングを作成](#create-a-binding)内の例に基づいたもので、SESSION バインディングが GLOBAL バインディングを遮断します:

```sql
-- SESSION スコープで作成したバインディングを削除します。
drop session binding for SELECT * FROM t1, t2 WHERE t1.id = t2.id;

-- SQL 実行プランを再度表示します。
explain SELECT * FROM t1,t2 WHERE t1.id = t2.
```

上記の例では、SESSION スコープの削除されたバインディングが対応する GLOBAL スコープのバインディングを遮断します。オプティマイザは `sm_join(t1, t2)` ヒントをステートメントに追加しません。`explain` 結果の実行プランのトップノードは、このヒントによって MergeJoin に固定されるのではなく、オプティマイザによってコスト推定に従って独立して選択されます。

#### `sql_digest` に従ってバインディングを削除する

SQL ステートメントに従ってバインディングを削除するだけでなく、`sql_digest` に従ってバインディングを削除することもできます。

```sql
DROP [GLOBAL | SESSION] BINDING FOR SQL DIGEST 'sql_digest';
```

このステートメントは、GLOBAL または SESSION レベルで `sql_digest` に対応する実行プランバインディングを削除します。デフォルトのスコープは SESSION です。`sql_digest` は[バインディングを表示](#view-bindings)で入手できます。

> **注意:**
>
> `DROP GLOBAL BINDING` を実行すると、現行の tidb-server インスタンスのキャッシュからバインディングを削除し、システムテーブルの対応する行の状態を '削除済み' に変更します。このステートメントは、他の tidb-server インスタンスが '削除済み' の状態を読み取り、対応するバインディングをキャッシュから削除するために必要です。'削除済み' の状態のこれらのシステムテーブルのレコードに対しては、100 `bind-info-lease` ごと（デフォルト値は `3s` で、合計で `300s` です）、バックグラウンドスレッドが、10 `bind-info-lease` 前に `update_time` のバインディングを回収およびクリアする動作をトリガーします（すべての tidb-server インスタンスが '削除済み' の状態を読み取り、キャッシュを更新したことを保証するため）。

### バインディング状態を変更する

#### SQL ステートメントに従ってバインディング状態を変更する

{{< copyable "sql" >}}

```sql
SET BINDING [ENABLED | DISABLED] FOR BindableStmt;
```

このステートメントを実行してバインディングの状態を変更できます。デフォルトの状態は ENABLED です。有効なスコープはデフォルトで GLOBAL で、変更できません。

このステートメントを実行する際には、バインディングの状態を `Disabled` から `Enabled` に、または `Enabled` から `Disabled` に変更できます。状態変更用のバインディングが利用できない場合は、`There are no bindings can be set the status. Please check the SQL text` という警告メッセージが返されます。`Disabled` の状態のバインディングはいかなるクエリーにも使用されません。

#### `sql_digest` に従ってバインディング状態を変更する

SQL ステートメントに従ってバインディング状態を変更するだけでなく、`sql_digest` に従ってバインディング状態を変更することもできます:

```sql
SET BINDING [ENABLED | DISABLED] FOR SQL DIGEST 'sql_digest';
```

`sql_digest` によって変更できるバインディングの状態、およびその効果は、[SQL ステートメントによって変更する場合](#change-binding-status-according-to-a-sql-statement)と同じです。状態変更用のバインディングが利用できない場合、`can't find any binding for 'sql_digest'` という警告メッセージが返されます。

### バインディングを表示する

{{< copyable "sql" >}}

```sql
SHOW [GLOBAL | SESSION] BINDINGS [ShowLikeOrWhere]
```

このステートメントは、GLOBAL または SESSION レベルでバインディング更新時刻の順に、実行計画バインディングを出力します。デフォルトのスコープは SESSION です。現在 `SHOW BINDINGS` は11列を出力しますが、以下に示します:

| 列名 | メモ |
| :-------- | :------------- |
| original_sql  |  パラメータ化後の元の SQL ステートメント |
| bind_sql | ヒント付きのバインドされた SQL ステートメント |
| default_db | デフォルトのデータベース |
| status | `enabled`（v6.0 から `using` ステータスに置き換えられました）、`disabled`、`deleted`、`invalid`、`rejected`、`pending verify` を含むステータス |
| create_time |  作成時刻 |
| update_time |  更新時刻 |
| charset |  キャラクタセット |
| collation |  整列順 |
| source | バインディングが作成された方法。`manual`（SQL ステートメントに従って作成）、`history`（過去の実行計画に従って作成）、`capture`（TiDB によって自動的にキャプチャ）、`evolve`（TiDB によって自動的に進化） |
| sql_digest | 正規化された SQL ステートメントのダイジェスト |
| plan_digest | 実行プランのダイジェスト |

### バインディングのトラブルシューティング

バインディングのトラブルシューティングには、次のいずれかの方法を使用できます。

- システム変数[`last_plan_from_binding`](/system-variables.md#last_plan_from_binding-new-in-v40)を使用して、最後に実行されたステートメントによって使用された実行プランがバインディングからであるかどうかを表示します。

     {{< copyable "sql" >}}

    ```sql
    -- グローバル バインディングを作成
    CREATE GLOBAL BINDING for
        SELECT * FROM t
    USING
        SELECT /*+ USE_INDEX(t, idx_a) */ * FROM t;

    SELECT * FROM t;
    SELECT @@[SESSION.]last_plan_from_binding;
    ```

    ```sql
    +--------------------------+
    | @@last_plan_from_binding |
    +--------------------------+
    |                        1 |
    +--------------------------+
    1 row in set (0.00 sec)
    ```

- SQLステートメントのクエリプランを表示するために`explain format = 'verbose'`ステートメントを使用します。SQLステートメントがバインディングを使用する場合は、`show warnings`を実行して、SQLステートメントで使用されるバインディングを確認できます。

    ```sql
    -- グローバルバインディングを作成します

    CREATE GLOBAL BINDING for
        SELECT * FROM t
    USING
        SELECT /*+ USE_INDEX(t, idx_a) */ * FROM t;

    -- `explain format = 'verbose'` を使用してSQLステートメントの実行プランを表示します

    explain format = 'verbose' SELECT * FROM t;

    -- `show warnings`を実行してクエリで使用されているバインディングを表示します。

    show warnings;
    ```

    ```sql
    +-------+------+--------------------------------------------------------------------------+
    | Level | Code | Message                                                                  |
    +-------+------+--------------------------------------------------------------------------+
    | Note  | 1105 | Using the bindSQL: SELECT /*+ USE_INDEX(`t` `idx_a`)*/ * FROM `test`.`t` |
    +-------+------+--------------------------------------------------------------------------+
    1 row in set (0.01 sec)

    ```

### バインディングのキャッシュ

各TiDBインスタンスには、バインディング用の最近使用された（LRU）キャッシュがあります。キャッシュ容量はシステム変数[`tidb_mem_quota_binding_cache`](/system-variables.md#tidb_mem_quota_binding_cache-new-in-v600)で制御されます。TiDBインスタンスでキャッシュされているバインディングを表示できます。

バインディングのキャッシュステータスを表示するには、`SHOW binding_cache status` ステートメントを実行します。このステートメントでは、効果範囲はデフォルトでGLOBALであり、変更できません。このステートメントは、キャッシュされている結合の数、システムで利用可能なすべての結合の合計数、キャッシュされているすべての結合のメモリ使用量、およびキャッシュの総メモリを返します。

{{< copyable "sql" >}}

```sql

SHOW binding_cache status;
```

```sql
+-------------------+-------------------+--------------+--------------+
| bindings_in_cache | bindings_in_table | memory_usage | memory_quota |
+-------------------+-------------------+--------------+--------------+
|                 1 |                 1 | 159 Bytes    | 64 MB        |
+-------------------+-------------------+--------------+--------------+
1 row in set (0.00 sec)
```

## ベースラインの取得

[アップグレード中の実行計画の逆行を防止](#prevent-regression-of-execution-plans-during-an-upgrade)するために使用され、この機能は取得条件を満たすクエリを取得し、これらのクエリに対してバインディングを作成します。

プランのベースラインとは、オプティマイザがSQLステートメントを実行するために使用できる受け入れられたプランのコレクションを指します。通常、TiDBは、プランが良好に実行されていることを確認した後に、プランベースラインにプランを追加します。このコンテキストでのプランは、オプティマイザが実行計画を再現するために必要なすべての必要なプラン関連の詳細（SQLプラン識別子、ヒントセット、バインド値、およびオプティマイザ環境など）を含みます。

### 取得の有効化

ベースラインの取得を有効にするには、`tidb_capture_plan_baselines`を`on`に設定します。デフォルト値は`off`です。

> **ノート：**
>
> 自動結合作成機能は[Statement Summary](/statement-summary-tables.md)に依存しているため、自動バインディングを使用する前に、必ずステートメントの要約を有効にしてください。

自動バインディング作成が有効になると、ステートメントサマリーに記録された履歴上のSQLステートメントは、デフォルトで`3s`（デフォルト値）ごとに走査され、少なくとも2回表示されたSQLステートメントに自動的にバインドが作成されます。これらのSQLステートメントに対して、TiDBはステートメントサマリーに記録された実行プランを自動的にバインディングします。

ただし、TiDBは次のタイプのSQLステートメントに対して自動的にバインディングを取得しません。

- `EXPLAIN`および`EXPLAIN ANALYZE`ステートメント。
- TiDB内部で実行されるSQLステートメント、たとえば、統計情報を自動的にローディングするための`SELECT`クエリ。
- `Enabled`または`Disabled`バインディングを含むステートメント。
- 取得条件でフィルタリングされたステートメント。

> **ノート：**
>
> 現在、バインディングは問い合わせステートメントによって生成された実行計画を修正するための一連のヒントを生成します。これにより、同じ問い合わせに対して実行計画が変更されません。ほとんどのOLTPクエリに対して、同じインデックスやJoinアルゴリズム（HashJoinおよびIndexJoinなど）を使用するクエリを含むクエリについて、TiDBはバインディングを明示する前後のプランの一貫性を保証します。ただし、ヒントの制限により、一部の複雑なクエリ（2つ以上のテーブルのJoin、MPPクエリ、および複雑なOLAPクエリなど）についてTiDBはプランの一貫性を保証できません。

`PREPARE` / `EXECUTE`ステートメントおよびバイナリプロトコルで実行されるクエリに対しては、実際のクエリステートメントのためにバインディングが自動的に取得されます。

> **ノート：**
>
> TiDBは、いくつかの機能の正確性を保証するために埋め込まれたSQLステートメントを実行しているため、ベースラインの取得はデフォルトでこれらのSQLステートメントを自動的に遮蔽します。

### バインディングのフィルタリング

この機能を使用すると、フィルタリング条件を構成してバインディングの取得を行いたくないクエリをブロックリストに設定できます。ブロックリストには、テーブル名、頻度、およびユーザー名の3つの次元があります。

#### 使用法

フィルタリング条件をシステムテーブル`mysql.capture_plan_baselines_blacklist`に挿入します。そして、フィルタリング条件は、クラスタ全体で即座に有効になります。

```sql
-- テーブル名でフィルタリング
 INSERT INTO mysql.capture_plan_baselines_blacklist(filter_type, filter_value) VALUES('table', 'test.t');

-- ワイルドカードを使用してデータベース名およびテーブル名でフィルタリング
 INSERT INTO mysql.capture_plan_baselines_blacklist(filter_type, filter_value) VALUES('table', 'test.table_*');
 INSERT INTO mysql.capture_plan_baselines_blacklist(filter_type, filter_value) VALUES('table', 'db_*.table_*');

-- 頻度でフィルタリング
 INSERT INTO mysql.capture_plan_baselines_blacklist(filter_type, filter_value) VALUES('frequency', '2');

-- ユーザー名でフィルタリング
 INSERT INTO mysql.capture_plan_baselines_blacklist(filter_type, filter_value) VALUES('user', 'user1');
```

| **次元名** | **説明**                                                     | 備考                                                         |
| :----------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| table        | テーブル名でフィルタリングします。各フィルタリングルールは`db.table`の形式です。サポートされるフィルタリングシンタックスには、[Plain table names](/table-filter.md#plain-table-names)および[Wildcards](/table-filter.md#wildcards)が含まれます。 | 大文字と小文字を区別しません。テーブル名に不正な文字が含まれている場合、ログには警告メッセージ`[sql-bind] failed to load mysql.capture_plan_baselines_blacklist`が表示されます。 |
| frequency    | 頻度でフィルタリングします。複数回実行されるSQLステートメントはデフォルトで取得されます。頻度を高く設定すると、頻繁に実行されるステートメントを取得できます。 | 頻度を1未満の値に設定することは不正と見なされ、ログには警告メッセージ`[sql-bind] frequency threshold is less than 1, ignore it`が表示されます。複数の頻度フィルタルールが挿入された場合、最も高い頻度の値が有効になります。 |
| user         | ユーザー名でフィルタリングします。ブロックリストに含まれるユーザーが実行したステートメントは取得されません。                           | 複数のユーザーが同じステートメントを実行し、そのユーザー名がすべてブロックリストに含まれている場合、このステートメントは取得されません。 |

> **ノート：**
>
> - ブロックリストを変更するには、スーパー権限が必要です。
>
> - ブロックリストに不正なフィルタが含まれている場合、TiDBはログに`[sql-bind] unknown capture filter type, ignore it`という警告メッセージを返します。

### アップグレード中の実行計画の逆行を防止

TiDBクラスタをアップグレードする前に、以下の手順でベースラインの取得を使用して実行計画の逆行を防止できます。

1. ベースライン取得を有効にして、動作させておきます。

    > **ノート：**
    >
    > テストデータによると、長期間のベースライン取得はクラスタ負荷に僅かな影響を与えます。したがって、重要なプラン（2回以上表示）が取得されるようにするために、できるだけ長くベースライン取得を有効にすることをお勧めします。

2. TiDBクラスタをアップグレードします。アップグレード後、TiDBはこれらの取得されたバインディングを使用して実行計画の一貫性を確保します。

3. アップグレード後、必要に応じてバインディングを削除します。

    - [`SHOW GLOBAL BINDINGS`](#view-bindings)ステートメントを実行してバインディングソースをチェックします。

        出力で`Source`フィールドを確認して、バインディングが取得（`capture`）されたか、手動で作成（`manual`）されたかを確認します。

    - 次のいずれかを確認してバインディングを保持するかどうかを決定します。

        ```
        -- バインディングを有効にしたプランを表示する
        SET @@SESSION.TIDB_USE_PLAN_BASELINES = true;
        EXPLAIN FORMAT='VERBOSE' SELECT * FROM t1 WHERE ...;

        -- バインディングを無効にしたプランを表示する
        SET @@SESSION.TIDB_USE_PLAN_BASELINES = false;
        EXPLAIN FORMAT='VERBOSE' SELECT * FROM t1 WHERE ...;
        ```

        - 実行計画が一貫している場合、安全にバインディングを削除できます。

        - 実行計画に不一致がある場合、例えば、統計情報をチェックすることで原因を特定する必要があります。この場合、計画の一貫性を確保するためにバインディングを保持する必要があります。

## ベースライン進化

ベースライン進化は、TiDB v4.0で導入されたSPMの重要な機能です。

データが更新されると、以前にバインドされた実行プランが最適ではなくなる可能性があります。ベースライン進化機能を使用すると、バインドされた実行プランを自動的に最適化できます。
また、基準線進化は一定程度、統計情報の変更による実行計画の揺れを回避することができます。

### 使用方法

次のステートメントを使用して、自動バインディング進化を有効にします。

```sql
SET GLOBAL tidb_evolve_plan_baselines = ON;
```

`tidb_evolve_plan_baselines` のデフォルト値は `off` です。

<CustomContent platform="tidb">

> **警告:**
>
> + 基準線進化は実験的な機能です。未知のリスクが存在するかもしれませんので、本番環境で使用することは**推奨されません**。
> + この変数は、基準線進化機能が一般利用可能（GA）になるまで `off` に強制設定されます。この機能を有効にしようとすると、エラーが返されます。本番環境で既にこの機能を使用している場合は、できるだけ早く無効にしてください。バインディング状態が期待通りでない場合は、PingCAPかコミュニティの[support](/support.md)を受けてください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **警告:**
>
> + 基準線進化は実験的な機能です。未知のリスクが存在するかもしれませんので、本番環境で使用することは**推奨されません**。
> + この変数は、基準線進化機能が一般利用可能（GA）になるまで `off` に強制設定されます。この機能を有効にしようとすると、エラーが返されます。本番環境で既にこの機能を使用している場合は、できるだけ早く無効にしてください。バインディング状態が期待通りでない場合は、[TiDB Cloud サポート](/tidb-cloud/tidb-cloud-support.md)に問い合わせてください。

</CustomContent>

自動バインディング進化機能を有効にした後、オプティマイザが選択した最適な実行計画がバインディング実行計画の中に含まれていない場合、オプティマイザはその計画を検証待ちの実行計画としてマークします。`bind-info-lease` インターバル（デフォルト値は `3s`）ごとに、検証待ちの実行計画が選択され、実際の実行時間のコストが最も低いバインディング実行計画と比較されます。検証待ちの計画の実行時間が（比較のための現在の基準は、検証待ちの計画の実行時間がバインディング実行計画の2/3未満であること）短い場合、この計画は使用可能なバインディングとしてマークされます。以下の例は上記のプロセスを説明しています。

表 `t` が以下のように定義されていると仮定します：

```sql
CREATE TABLE t(a INT, b INT, KEY(a), KEY(b));
```

表 `t` 上で次のクエリを実行します：

```sql
SELECT * FROM t WHERE a < 100 AND b < 100;
```

上記の定義された表では、`a < 100` 条件を満たす行がほとんどありません。しかし何らかの理由で、オプティマイザが適切な実行計画である `a` インデックスを使用する最適な実行計画の代わりにフルテーブルスキャンを誤って選択しています。以下のステートメントを使用してまずバインディングを作成することができます：

```sql
CREATE GLOBAL BINDING for SELECT * FROM t WHERE a < 100 AND b < 100 using SELECT * FROM t use index(a) WHERE a < 100 AND b < 100;
```

上記のクエリを再度実行すると、オプティマイザはバインディングに影響を受けて `a` インデックスを選択してクエリ時間を短縮します。

仮に表 `t` に挿入と削除が行われ、`a < 100` 条件を満たす行が増え、`b < 100` 条件を満たす行が減少するような状況になったとします。この時、バインディング進化はこの種の問題に対処することができます。オプティマイザは表のデータ変更を認識すると、`b` インデックスを使用するクエリの実行計画を生成します。しかし、現在の計画のバインディングが存在するため、このクエリ計画は採用されず実行されません。その代わりに、この計画はバックエンド進化リストに保存されます。進化プロセス中、この計画が現在の `a` インデックスを使用する実行計画よりも明らかに短い実行時間であることが確認された場合、`b` インデックスは使用可能なバインディングリストに追加されます。その後、同じクエリが再度実行されると、オプティマイザはまず `b` インデックスを使用する実行計画を生成し、この計画がバインディングリストにあることを確認してから、データ変更後のクエリ時間を短縮するためにこの計画を採用し実行します。

自動進化がクラスタへ与える影響を減らすために、以下の構成を使用してください：

- `tidb_evolve_plan_task_max_time` を設定して、各実行計画の最大実行時間を制限します。デフォルトの値は `600s` です。実際の検証プロセスでは、最大実行時間は検証された実行計画の2倍を超えないように制限されます。
- `tidb_evolve_plan_task_start_time`（デフォルトは `00:00 +0000`）と `tidb_evolve_plan_task_end_time`（デフォルトは `23:59 +0000`）を設定して、時間ウィンドウを制限します。

### 注意

基準線進化は、少なくとも1つのグローバルバインディングを持つ標準化されたSQLステートメントのみを進化させます。

標準化されたSQLステートメントに新しいバインディングを作成すると、以前のバインディングはすべて削除されるため、自動的に進化させたバインディングは、手動で新しいバインディングを作成すると削除されます。

進化に関連する計算プロセスはすべて保持されます。これらのヒントは以下の通りです。

    | ヒント | 説明            |
    | :-------- | :------------- |
    | `memory_quota` | クエリに使用できる最大メモリ。 |
    | `use_toja` | オプティマイザがサブクエリを結合に変換するか。 |
    | `use_cascades` | カスケードオプティマイザを使用するか。 |
    | `no_index_merge` | オプティマイザがテーブルを読み取るオプションとしてIndex Mergeを使用するか。 |
    | `read_consistent_replica` | テーブルを読み取る際にFollower Readを強制的に有効にするか。 |
    | `max_execution_time` | クエリの最長実行時間。 |

`read_from_storage` は、テーブルを読み取る際にTiKVからデータを読み取るかTiFlashから読み取るかを指定する特別なヒントです。TiDBは独自の分離読み取りを提供するため、分離条件が変更されると、このヒントは進化実行計画に大きな影響を与えます。そのため、このヒントが最初に作成されたバインディングに存在する場合、TiDBは進化したすべてのバインディングを無視します。

## アップグレードチェックリスト

クラスタをアップグレードする際、SQL Plan Management (SPM) は互換性の問題を引き起こし、アップグレードに失敗する可能性があります。アップグレードの成功を確実にするために、アップグレードの事前チェックに以下のリストを含める必要があります。

* 現在のバージョンが v5.2.0 よりも古いバージョン（つまり、v4.0、v5.0、v5.1）から現在のバージョンにアップグレードする場合は、アップグレード前に `tidb_evolve_plan_baselines` が無効になっていることを確認してください。この変数を無効にするには、次の手順を実行します。

    ```sql
    -- 以前のバージョンで `tidb_evolve_plan_baselines` が無効になっているかを確認します。

    SELECT @@global.tidb_evolve_plan_baselines;

    -- `tidb_evolve_plan_baselines` がまだ有効な場合、無効にします。

    SET GLOBAL tidb_evolve_plan_baselines = OFF;
    ```

* v4.0 から現在のバージョンにアップグレードする場合は、利用可能なSQLバインディングに対応するすべてのクエリの構文が新しいバージョンで正しいかどうかをチェックする必要があります。構文エラーが存在する場合は、対応するSQLバインディングを削除してください。そのためには、次の手順を実行します。

    ```sql
    -- 使用中のステータスにある利用可能なSQLバインディングに対応するクエリを、新しいバージョンにおけるテスト環境で確認します。

    mysql.bind_info から status = 'using' のバインディングに対応するクエリを確認します。

    -- 上記のSQLクエリからの結果を新しいバージョンのテスト環境で確認します。

    bind_sql_0;
    bind_sql_1;
    ...

    -- 構文エラーの場合（ERROR 1064 (42000): You have an error in your SQL syntax）、対応するバインディングを削除してください。
    -- その他のエラー（例えば、テーブルが見つからない）の場合、構文が互換性があることを意味します。それ以外の操作は必要ありません。
    ```