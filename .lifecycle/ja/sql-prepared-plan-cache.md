---
title: SQL実行計画のキャッシュ
summary: TiDBのSQL実行計画のキャッシュについて学びます。
aliases: ['/tidb/dev/sql-prepare-plan-cache']
---

# SQL実行計画のキャッシュ

TiDBは`Prepare`および`Execute`クエリの実行計画キャッシュをサポートしています。これには、次のような準備された文の両方の形式が含まれます。

- `COM_STMT_PREPARE`および`COM_STMT_EXECUTE`プロトコル機能を使用する。
- SQL文の`PREPARE`および`EXECUTE`を使用する。

TiDBの最適化機能は、これら2種類のクエリを同じ方法で処理します。準備段階では、パラメータ化されたクエリはAST(Abstract Syntax Tree)にパースされてキャッシュされます。後の実行時には、保存されたASTと特定のパラメータ値に基づいて実行計画が生成されます。

実行計画キャッシュが有効になっている場合、最初の実行時には、すべての`Prepare`文が現在のクエリが実行計画キャッシュを使用できるかどうかをチェックし、クエリがそれを使用できる場合は生成された実行計画をLRU(Least Recently Used)連結リストによって実装されたキャッシュに入れます。後続の`Execute`クエリでは、実行計画はキャッシュから取得され、可用性がチェックされます。チェックが成功すると、実行計画生成のステップはスキップされます。それ以外の場合、実行計画は再生成され、キャッシュに保存されます。

TiDBはまた、一部の`PREPARE`文以外の非`PREPARE`文の実行計画キャッシュもサポートしており、これは`Prepare`/`Execute`文と同様です。詳細については、[非準備実行計画のキャッシュ](/sql-non-prepared-plan-cache.md)を参照してください。

TiDBの現行バージョンでは、`Prepare`文が次の条件のいずれかを満たす場合、クエリまたは計画はキャッシュされません。

- クエリに`SELECT`、`UPDATE`、`INSERT`、`DELETE`、`Union`、`Intersect`、および`Except`以外のSQL文が含まれている。
- クエリがパーティションされたテーブル、一時テーブル、または生成された列を含むテーブルにアクセスしている。
- クエリに非相関サブクエリが含まれている（たとえば、`SELECT * FROM t1 WHERE t1.a > (SELECT 1 FROM t2 WHERE t2.b < 1)`）。
- クエリに`PhysicalApply`演算子を持つ相関サブクエリが含まれている（たとえば、`SELECT * FROM t1 WHERE t1.a > (SELECT a FROM t2 WHERE t1.b > t2.b)`）。
- クエリに`ignore_plan_cache`または`set_var`ヒントが含まれている（たとえば、`SELECT /*+ ignore_plan_cache() */ * FROM t`または`SELECT /*+ set_var(max_execution_time=1) */ * FROM t`）。
- クエリに`?`以外の変数が含まれている（システム変数またはユーザー定義変数を含む）、たとえば、`select * from t where a>? and b>@x`。
- キャッシュできない関数が含まれている：`database()`、`current_user`、`current_role`、`user`、`connection_id`、`last_insert_id`、`row_count`、`version`、`like`。
- `LIMIT`パラメータとして変数が使用されている（`LIMIT ?`や`LIMIT 10, ?`など）が、変数の値が10000より大きい。
- `Order By`の後に`?`が含まれている（`Order By ?`）。このようなクエリは`?`で指定された列に基づいてデータをソートします。異なる列を対象とするクエリが同じ実行計画を使用すると、結果が正しくありません。したがって、このようなクエリはキャッシュされません。ただし、`Order By a+?`のような一般的なクエリの場合はキャッシュされます。
- `Group By`の後に`?`が含まれている（`Group By?`）。このようなクエリは`?`で指定された列に基づいてデータをグループ化します。異なる列を対象とするクエリが同じ実行計画を使用すると、結果が正しくありません。したがって、このようなクエリはキャッシュされません。ただし、`Group By a+?`のような一般的なクエリの場合はキャッシュされます。
- `Window Frame`ウィンドウ関数の定義に`?`が含まれている（たとえば、`(partition by year order by sale rows ? preceding)`）。もしウィンドウ関数の他の場所に`?`が現れる場合は、クエリはキャッシュされます。
- `int`と`string`の比較を行うパラメータが含まれている（`c_int >= ?`や`c_int in (?, ?)`のような）が、`?`が`set @x='123'`のように文字列型を示す場合。クエリ結果がMySQLと互換性があるようにするには、クエリごとにパラメータを調整する必要があるため、このようなクエリはキャッシュされません。
- 計画が`TiFlash`にアクセスしようとしている。
- ほとんどの場合、`TableDual`を含む計画はキャッシュされませんが、現在の`Prepare`文にパラメータが含まれていない場合は除きます。
- `information_schema.columns`などのTiDBシステムビューへのアクセスが含まれている。システムビューにアクセスするために`Prepare`および`Execute`文を使用することはお勧めできません。

TiDBには、クエリ内の`?`の数に制限があります。クエリに65535個を超える`?`が含まれている場合、`Prepared statement contains too many placeholders`というエラーが報告されます。

LRU連結リストはセッションレベルのキャッシュとして設計されています。なぜなら`Prepare`/`Execute`はセッション間で実行できないからです。LRUリストの各要素はキーと値のペアです。値は実行計画で、キーは以下の部分で構成されます。

- `Execute`が実行されるデータベース名
- `PREPARE`キーワードの後の名前である`Prepare`文の識別子
- 成功裏に実行されたDDL文ごとに更新される現在のスキーマバージョン
- `Execute`を実行するときのSQLモード
- `time_zone`システム変数の値である現在のタイムゾーン
- `sql_select_limit`システム変数の値

前記の情報の変更（たとえば、データベースの切り替え、`Prepare`文の名前の変更、DDL文の実行、SQLモード/`time_zone`の値の変更）またはLRUキャッシュ除外メカニズムによって、実行計画キャッシュミスが発生します。

キャッシュから実行計画キャッシュを取得した後、TiDBはまず実行計画がまだ有効かどうかをチェックします。もし現在の`Execute`文が明示的トランザクションで実行され、そのトランザクションの事前命令で参照されるテーブルが`UnionScan`演算子を含まない、キャッシュされた実行計画がこのテーブルへのアクセスを含まない場合、そのキャッシュされた実行計画は実行できません。

検証テストに合格した後、実行計画のスキャン範囲は現在のパラメータ値に応じて調整され、データクエリを実行するために使用されます。

実行計画キャッシュとクエリパフォーマンスに関連するいくつかの重要なポイントがあります。

- 実行計画がキャッシュされているかどうかにかかわらず、SQLバインディングによって影響を受けます。キャッシュされていない実行計画（最初の`Execute`）は既存のSQLバインディングに影響を受けます。キャッシュされた実行計画では、新しいSQLバインディングが作成されるとそれらの計画は無効になります。
- キャッシュされた実行計画は、統計情報、最適化ルール、および式によるブロックリストのプッシュダウンの変更に影響を受けません。
- `Execute`のパラメータが異なる場合を考慮すると、特定のパラメータ値と密接に関連する攻撃的なクエリ最適化手法は禁止されます。これは適応性を確保するためです。クエリプランは特定のパラメータ値に対して最適でない場合があります。たとえば、クエリのフィルタ条件が`where a > ? And a < ?`の場合、最初の`Execute`文のパラメータがそれぞれ`2`と`1`であるとします。次回の実行時にはこれら2つのパラメータが`1`と`2`になるかもしれないと考えると、オプティマイザは現在のパラメータ値に特化した最適な`TableDual`実行計画を生成しません。
- キャッシュが無効化および除外されないかぎり、実行計画キャッシュは理論上もさまざまなパラメータ値に適用されます。これにより、ある値に対して最適でない実行計画が発生します。たとえば、フィルタ条件が`where a < ?`で、最初の実行に使用されるパラメータ値が`1`である場合、オプティマイザは最適な`IndexScan`実行計画を生成し、キャッシュに入れます。後続の実行で、値が`10000`になった場合、`TableScan`の計画の方がより良いかもしれません。しかし、実行計画キャッシュのために、以前に生成された`IndexScan`が実行に使用されます。したがって、実行計画キャッシュはクエリが単純で（コンパイルの割合が高い）かつ実行計画が比較的固定であるアプリケーションシナリオに適しています。

v6.1.0以降、実行計画キャッシュはデフォルトで有効になっています。`tidb_enable_prepared_plan_cache`というシステム変数を使用して、準備実行計画キャッシュを制御できます。

> **Note:**
>
> 実行計画キャッシュ機能は`Prepare`/`Execute`クエリにのみ適用され、通常のクエリには適用されません。

実行計画キャッシュ機能を有効にした後は、セッションレベルのシステム変数`last_plan_from_cache`を使用して、以前の`Execute`文がキャッシュされた実行計画を使用したかどうかを確認できます。たとえば:

{{< copyable "sql" >}}

```sql
MySQL [test]> create table t(a int);
Query OK, 0 rows affected (0.00 sec)
MySQL [test]> prepare stmt from 'select * from t where a = ?';
Query OK, 0 rows affected (0.00 sec)
MySQL [test]> set @a = 1;
Query OK, 0 rows affected (0.00 sec)

-- 最初の実行で実行計画が生成され、キャッシュに保存されます。
MySQL [test]> execute stmt using @a;
Empty set (0.00 sec)
```sql
MySQL [test]> select @@last_plan_from_cache;
+------------------------+
| @@last_plan_from_cache |
+------------------------+
| 0                      |
+------------------------+
1 row in set (0.00 sec)

-- 二回目の実行はキャッシュをヒットします。
MySQL [test]> execute stmt using @a;
Empty set (0.00 sec)
MySQL [test]> select @@last_plan_from_cache;
+------------------------+
| @@last_plan_from_cache |
+------------------------+
| 1                      |
+------------------------+
1 row in set (0.00 sec)
```

特定の`Prepare`/`Execute`セットが実行プランキャッシュの影響で予期しない動作をする場合、現在の文に対して実行プランキャッシュを使用せずスキップするために、`ignore_plan_cache()`SQLヒントを使用できます。先行する文を例として以下に示します:

{{< copyable "sql" >}}

```sql
MySQL [test]> prepare stmt from 'select /*+ ignore_plan_cache() */ * from t where a = ?';
Query OK, 0 rows affected (0.00 sec)
MySQL [test]> set @a = 1;
Query OK, 0 rows affected (0.00 sec)
MySQL [test]> execute stmt using @a;
Empty set (0.00 sec)
MySQL [test]> select @@last_plan_from_cache;
+------------------------+
| @@last_plan_from_cache |
+------------------------+
| 0                      |
+------------------------+
1 row in set (0.00 sec)
MySQL [test]> execute stmt using @a;
Empty set (0.00 sec)
MySQL [test]> select @@last_plan_from_cache;
+------------------------+
| @@last_plan_from_cache |
+------------------------+
| 0                      |
+------------------------+
1 row in set (0.00 sec)
```

## Prepared Plan Cacheの診断

一部のクエリまたはプランはキャッシュできません。クエリまたはプランがキャッシュされているかどうかを確認するには、`SHOW WARNINGS`文を使用します。キャッシュされていない場合、結果で失敗の理由を確認できます。例:

```sql
mysql> PREPARE st FROM 'SELECT * FROM t WHERE a > (SELECT MAX(a) FROM t)';  -- クエリにサブクエリが含まれ、キャッシュできません。

Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> show warnings;  -- クエリプランがキャッシュできない理由を確認します。

+---------+------+-----------------------------------------------+
| Level   | Code | Message                                       |
+---------+------+-----------------------------------------------+
| Warning | 1105 | skip plan-cache: sub-queries are un-cacheable |
+---------+------+-----------------------------------------------+
1 row in set (0.00 sec)

mysql> prepare st from 'select * from t where a<?';

Query OK, 0 rows affected (0.00 sec)

mysql> set @a='1';

Query OK, 0 rows affected (0.00 sec)

mysql> execute st using @a;  -- 最適化により、非INT型がINT型に変換され、パラメータの変更に伴い実行プランが変化する可能性があるため、TiDBはプランをキャッシュしません。

Empty set, 1 warning (0.01 sec)

mysql> show warnings;

+---------+------+----------------------------------------------+
| Level   | Code | Message                                      |
+---------+------+----------------------------------------------+
| Warning | 1105 | skip plan-cache: '1' may be converted to INT |
+---------+------+----------------------------------------------+
1 row in set (0.00 sec)
```

## Prepared Plan Cacheのメモリ管理

<CustomContent platform="tidb">

Prepared Plan Cacheを使用すると、メモリオーバーヘッドが発生します。各TiDBインスタンスの全セッションにキャッシュされた実行プランの総メモリ消費量を表示するには、Grafanaの[**Plan Cache Memory Usage**モニタリングパネル](/grafana-tidb-dashboard.md)を使用できます。

> **注記:**
>
> Golangのメモリ回収メカニズムといくつかのカウントされないメモリ構造のため、Grafanaに表示されるメモリは実際のヒープメモリ使用量とは一致しません。Grafanaに表示されるメモリと実際のヒープメモリ使用量の間には約±20%の偏差があることが確認されています。

各TiDBインスタンスでキャッシュされた実行プランの合計数を表示するには、Grafanaの[**Plan Cache Plan Num**パネル](/grafana-tidb-dashboard.md)を使用できます。

以下は、Grafanaの**Plan Cache Memory Usage**および**Plan Cache Plan Num**パネルの例です:

![grafana_panels](/media/planCache-memoryUsage-planNum-panels.png)

v7.1.0以降では、セッションごとにキャッシュされるプランの最大数を制御することができるようになります。システム変数[`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_p

適切な環境によって値を調整し、監視パネルに合わせて設定できます:

</CustomContent>

<CustomContent platform="tidb-cloud">

Prepared Plan Cacheを使用すると、メモリオーバーヘッドが発生します。内部テストでは、各キャッシュされたプランが平均で100 KiBのメモリを使用します。Plan Cacheは現在`SESSION`レベルであるため、合計メモリ消費はおよそ`セッション数 * セッションあたりの平均キャッシュプラン数 * 100 KiB`となります。

例えば、現在のTiDBインスタンスでは並行して50のセッションがあり、それぞれのセッションにはおよそ100のキャッシュプランがあります。合計メモリ消費はおよそ`50 * 100 * 100 KiB` = `512 MB`となります。

システム変数[`tidb_session_plan_cache_size`](/system-variables.md#tidb_session_plan_cache_size-new-in-v710)を設定することで、それぞれのセッションにキャッシュされるプランの最大数を制御できます。異なる環境に対して、推奨される値は以下の通りです:

</CustomContent>

- TiDBサーバーインスタンスのメモリ閾値が<= 64 GiBの場合、`tidb_session_plan_cache_size`を`50`に設定します。
- TiDBサーバーインスタンスのメモリ閾値が64 GiBより大きい場合、`tidb_session_plan_cache_size`を`100`に設定します。

v7.1.0以降では、システム変数[`tidb_plan_cache_max_plan_size`](/system-variables.md#tidb_plan_cache_max_plan_size-new-in-v710)を使用することで、キャッシュできるプランの最大サイズを制御できます。デフォルト値は2 MBです。プランのサイズがこの値を超える場合、プランはキャッシュされません。

TiDBサーバーの未使用メモリが一定の閾値以下になると、実行プランキャッシュのメモリ保護メカニズムがトリガーされ、一部のキャッシュされたプランが削除されます。

閾値はシステム変数`tidb_prepared_plan_cache_memory_guard_ratio`を設定することで制御できます。デフォルトでは閾値は0.1であり、TiDBサーバーの未使用メモリが総メモリの10%未満（メモリの90%が使用中）になると、メモリ保護メカニズムがトリガーされます。

<CustomContent platform="tidb">

メモリ制限のため、プランキャッシュは時折ミスされることがあります。Grafanaダッシュボードで[`Plan Cache Miss OPS`メトリクス](/grafana-tidb-dashboard.md)を表示することで、ステータスを確認できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

メモリ制限のため、プランキャッシュは時折ミスされることがあります。

</CustomContent>

## 実行プランキャッシュのクリア

`ADMIN FLUSH [SESSION | INSTANCE] PLAN_CACHE`文を実行して実行プランキャッシュをクリアできます。

この文で、`[SESSION | INSTANCE]`は現在のセッションまたはTiDBインスタンス全体のプランキャッシュがクリアされるかを指定します。スコープが指定されていない場合、デフォルトで`SESSION`キャッシュに適用されます。

以下は、`SESSION`実行プランキャッシュをクリアする例です:

{{< copyable "sql" >}}

```sql
MySQL [test]> create table t (a int);
Query OK, 0 rows affected (0.00 sec)

MySQL [test]> prepare stmt from 'select * from t';
Query OK, 0 rows affected (0.00 sec)

MySQL [test]> execute stmt;
Empty set (0.00 sec)

MySQL [test]> execute stmt;
Empty set (0.00 sec)

MySQL [test]> select @@last_plan_from_cache; -- キャッシュされたプランを選択
+------------------------+
| @@last_plan_from_cache |
+------------------------+
|                      1 |
+------------------------+
1 row in set (0.00 sec)

MySQL [test]> admin flush session plan_cache; -- 現在のセッションのキャッシュプランをクリア
Query OK, 0 rows affected (0.00 sec)

MySQL [test]> execute stmt;
Empty set (0.00 sec)

MySQL [test]> select @@last_plan_from_cache; -- キャッシュされたプランは再度選択できません。クリアされたためです。
+------------------------+
| @@last_plan_from_cache |
+------------------------+
|                      0 |
+------------------------+
1 row in set (0.00 sec)
```

現在TiDBでは`GLOBAL`実行プランキャッシュのクリアはサポートされていません。つまり、TiDBクラスタ全体のキャッシュプランはクリアできません。次のエラーが表示されます:

{{< copyable "sql" >}}

```sql
MySQL [test]> admin flush global plan_cache;
ERROR 1105 (HY000): Do not support the 'admin flush global scope.'
```

## `COM_STMT_CLOSE`コマンドと`DEALLOCATE PREPARE`ステートメントを無視してください

SQLステートメントの構文解析コストを削減するために、`prepare stmt`を1回実行してから`execute stmt`を複数回実行する前に`deallocate prepare`を実行することを推奨します：

{{< copyable "sql" >}}

```sql
MySQL [test]> prepare stmt from '...'; -- 1回だけ準備
MySQL [test]> execute stmt using ...;  -- 1回だけ実行
MySQL [test]> ...
MySQL [test]> execute stmt using ...;  -- 複数回実行
MySQL [test]> deallocate prepare stmt; -- 準備したステートメントを解放
```

実際のプラクティスでは、以下のように`execute stmt`の後に毎回`deallocate prepare`を実行することがあります：

{{< copyable "sql" >}}

```sql
MySQL [test]> prepare stmt from '...'; -- 1回だけ準備
MySQL [test]> execute stmt using ...;
MySQL [test]> deallocate prepare stmt; -- 準備したステートメントを解放
MySQL [test]> prepare stmt from '...'; -- 2回準備
MySQL [test]> execute stmt using ...;
MySQL [test]> deallocate prepare stmt; -- 準備したステートメントを解放
```

このようなプラクティスでは、最初に実行されたステートメントによって得られたプランを2番目に実行されたステートメントで再利用することができません。

この問題に対処するには、システム変数[`tidb_ignore_prepared_cache_close_stmt`](/system-variables.md#tidb_ignore_prepared_cache_close_stmt-new-in-v600)を`ON`に設定して、TiDBが`prepare stmt`を閉じるコマンドを無視するようにすることができます：

{{< copyable "sql" >}}

```sql
mysql> set @@tidb_ignore_prepared_cache_close_stmt=1;  -- 変数を有効にする
Query OK, 0 rows affected (0.00 sec)

mysql> prepare stmt from 'select * from t'; -- 1回だけ準備
Query OK, 0 rows affected (0.00 sec)

mysql> execute stmt;                        -- 1回だけ実行
Empty set (0.00 sec)

mysql> deallocate prepare stmt;             -- 最初の実行後に解放
Query OK, 0 rows affected (0.00 sec)

mysql> prepare stmt from 'select * from t'; -- 2回準備
Query OK, 0 rows affected (0.00 sec)

mysql> execute stmt;                        -- 2回実行
Empty set (0.00 sec)

mysql> select @@last_plan_from_cache;       -- 最後のプランを再利用
+------------------------+
| @@last_plan_from_cache |
+------------------------+
|                      1 |
+------------------------+
1 row in set (0.00 sec)
```

### モニタリング

<CustomContent platform="tidb">

TiDBページのGrafanaダッシュボード[/grafana-tidb-dashboard.md](/grafana-tidb-dashboard.md)の**Executor**セクションには、「Queries Using Plan Cache OPS」と「Plan Cache Miss OPS」のグラフがあります。これらのグラフを使用して、TiDBとアプリケーションの両方が正しく構成されているかどうかを確認し、SQLプランキャッシュが正常に機能するようにします。同じページの**Server**セクションには、「Prepared Statement Count」のグラフがあります。このグラフは、アプリケーションがプリペアドステートメントを使用している場合（これはSQLプランキャッシュが正しく機能するために必要です）、ゼロ以外の値が表示されます。

![`sql_plan_cache`](/media/performance/sql_plan_cache.png)

</CustomContent>

<CustomContent platform="tidb-cloud">

[TiDB Cloudコンソール](https://tidbcloud.com/)の[**Monitoring**](/tidb-cloud/built-in-monitoring.md)ページでは、TiDBインスタンス全体でプランキャッシュを使用するまたはミスするクエリの数を1秒あたりに取得できる`Queries Using Plan Cache OPS`メトリックを確認できます。

</CustomContent>