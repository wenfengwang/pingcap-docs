---
title: ステートメントサマリーテーブル
summary: TiDBのステートメントサマリーテーブルについて学ぶ
aliases: ['/docs/dev/statement-summary-tables/','/docs/dev/reference/performance/statement-summary/']
---

# ステートメントサマリーテーブル

SQLパフォーマンスの問題をよりよく扱うために、MySQLは統計を伴うSQLを監視するための[ステートメントサマリーテーブル](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-summary-tables.html)を`performance_schema`に提供しています。これらのテーブルの中で`events_statements_summary_by_digest`は、遅延時間、実行回数、スキャンされた行、そしてテーブル全体のスキャンなど、豊富なフィールドを持つため、SQLの問題を特定するのに非常に役立ちます。

そのため、v4.0.0-rc.1から、TiDBは`performance_schema`ではなく、`information_schema`で`events_statements_summary_by_digest`に似たシステムテーブルを提供しています。

- [`statements_summary`](#ステートメントサマリー)
- [`statements_summary_history`](#ステートメントサマリーの履歴)
- [`cluster_statements_summary`](#statements_summary_evicted)
- [`cluster_statements_summary_history`](#statements_summary_evicted)
- [`statements_summary_evicted`](#statements_summary_evicted)

> **Note:**
>
> 上記のテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

このドキュメントでは、これらのテーブルの詳細を説明し、SQLパフォーマンスの問題をトラブルシューティングする方法を紹介しています。

## `statements_summary`

`statements_summary`は`information_schema`のシステムテーブルです。`statements_summary`はSQL文をSQLダイジェストとプランダイジェストによってグループ化し、各SQLカテゴリーの統計情報を提供します。

ここでの「SQLダイジェスト」とは、遅いログで使用されるものと同じものであり、正規化されたSQL文を通じて計算されるユニークな識別子です。正規化プロセスは定数と空白文字を無視し、大文字小文字の区別をしません。したがって、構文が一貫した文は同じダイジェストを持ちます。例えば:

{{< copyable "sql" >}}

```sql
SELECT * FROM employee WHERE id IN (1, 2, 3) AND salary BETWEEN 1000 AND 2000;
select * from EMPLOYEE where ID in (4, 5) and SALARY between 3000 and 4000;
```

正規化後、以下のカテゴリーの文になります:

{{< copyable "sql" >}}

```sql
select * from employee where id in (...) and salary between ? and ?;
```

ここでの「プランダイジェスト」は、正規化された実行プランによって計算されるユニークな識別子を指します。正規化プロセスでは定数を無視します。同じSQL文でも異なる実行プランを持つことで、異なるカテゴリーにグループ化されることがあります。同じカテゴリーのSQL文は同じ実行プランを持ちます。

`statements_summary`はSQL監視メトリクスの集計結果を保存しています。一般的に、監視メトリクスは最大値と平均値を含んでいます。例えば、実行遅延メトリクスには2つのフィールドが対応しています: `AVG_LATENCY` (平均遅延時間) および `MAX_LATENCY` (最大遅延時間)。

監視メトリクスが最新の状態であることを確認するためには、`statements_summary`テーブル内のデータが定期的にクリアされ、最新の集計結果のみが保持および表示されます。データの周期的なクリアは`tidb_stmt_summary_refresh_interval`システム変数によって制御されます。クリア後すぐにクエリを行う場合、表示されるデータが非常に少ない場合があります。

次は`statements_summary`をクエリした際のサンプル出力です:

```
   SUMMARY_BEGIN_TIME: 2020-01-02 11:00:00
     SUMMARY_END_TIME: 2020-01-02 11:30:00
            STMT_TYPE: Select
          SCHEMA_NAME: test
               DIGEST: 0611cc2fe792f8c146cc97d39b31d9562014cf15f8d41f23a4938ca341f54182
          DIGEST_TEXT: select * from employee where id = ?
          TABLE_NAMES: test.employee
          INDEX_NAMES: NULL
          SAMPLE_USER: root
           EXEC_COUNT: 3
          SUM_LATENCY: 1035161
          MAX_LATENCY: 399594
          MIN_LATENCY: 301353
          AVG_LATENCY: 345053
    AVG_PARSE_LATENCY: 57000
    MAX_PARSE_LATENCY: 57000
  AVG_COMPILE_LATENCY: 175458
  MAX_COMPILE_LATENCY: 175458
  ...........
              AVG_MEM: 103
              MAX_MEM: 103
              AVG_DISK: 65535
              MAX_DISK: 65535
    AVG_AFFECTED_ROWS: 0
           FIRST_SEEN: 2020-01-02 11:12:54
            LAST_SEEN: 2020-01-02 11:25:24
    QUERY_SAMPLE_TEXT: select * from employee where id=3100
     PREV_SAMPLE_TEXT:
          PLAN_DIGEST: f415b8d52640b535b9b12a9c148a8630d2c6d59e419aad29397842e32e8e5de3
                 PLAN:  Point_Get_1     root    1       table:employee, handle:3100
```

> **Note:**
>
> TiDBでは、ステートメントサマリーテーブルのフィールドの時間単位はナノ秒（ns）であり、MySQLではピコ秒（ps）です。

## `statements_summary_history`

`statements_summary_history`のテーブル構造は`statements_summary`と同じです。`statements_summary_history`は時間範囲の履歴データを保存します。履歴データを確認することで、異常をトラブルシューティングし、異なる時間範囲の監視メトリクスを比較することができます。

`SUMMARY_BEGIN_TIME` および `SUMMARY_END_TIME` のフィールドは、履歴時間範囲の開始時間と終了時間を表します。

## `statements_summary_evicted`

`tidb_stmt_summary_max_stmt_count` 変数は`statement_summary`テーブルがメモリ内に格納するSQL文の最大数を制御します。`statement_summary`テーブルはLRUアルゴリズムを使用します。SQL文の数が`tidb_stmt_summary_max_stmt_count`の値を超えると、最長間使用されていないレコードがテーブルから削除されます。各期間ごとに削除されたSQL文の数が`statements_summary_evicted`テーブルに記録されます。

`statements_summary_evicted`テーブルは、`statement_summary`テーブルからSQLレコードが削除された場合のみ更新されます。`statements_summary_evicted`は削除が発生した期間と削除されたSQL文の数のみを記錍します。

## ステートメントサマリーのクラスターテーブル

`statements_summary`、`statements_summary_history`、および`statements_summary_evicted`テーブルは単一のTiDBサーバーのステートメントサマリーを表示します。クラスター全体のデータをクエリするには、`cluster_statements_summary`、`cluster_statements_summary_history`、または`cluster_statements_summary_evicted`テーブルをクエリする必要があります。

`cluster_statements_summary` は、各TiDBサーバーの`statements_summary`データを表示します。`cluster_statements_summary_history` は、各TiDBサーバーの`statements_summary_history`データを表示します。`cluster_statements_summary_evicted`は、各TiDBサーバーの`statements_summary_evicted`データを表示します。これらのテーブルは`INSTANCE`フィールドを使用してTiDBサーバーのアドレスを表します。その他のフィールドは`statements_summary`、`statements_summary_history`、および`statements_summary_evicted`と同じです。

## パラメータ設定

以下のシステム変数はステートメントサマリーを制御するために使用されます:

- `tidb_enable_stmt_summary`: ステートメントサマリー機能を有効にするかどうかを決定します。 `1`は`有効`を、`0`は`無効`を表します。この機能はデフォルトで有効になっています。この機能が無効になると、システムテーブルの統計情報がクリアされます。この機能が再び有効になると、統計情報は再計算されます。この機能を有効にすることがパフォーマンスに与える影響は少ないことがテストで示されています。
- `tidb_stmt_summary_refresh_interval`: `statements_summary`テーブルのリフレッシュ間隔。時間単位は秒（s）です。デフォルト値は`1800`です。
- `tidb_stmt_summary_history_size`: `statements_summary_history`テーブルに保存される各SQL文カテゴリーのサイズ。また、`statements_summary_evicted`テーブルの最大レコード数でもあります。デフォルト値は`24`です。

<CustomContent platform="tidb">

- `tidb_stmt_summary_max_stmt_count`: ステートメントサマリーテーブルに保存できるSQLステートメントの数を制限します。デフォルト値は`3000`です。制限を超えると、TiDBは最近使用されなくなったSQLステートメントをクリアします。これらのクリアされたSQLステートメントは`DIGEST`が`NULL`に設定された行として表され、`statements_summary_evicted`テーブルに記録されます。TiDBダッシュボードの[SQLステートメントページ](/dashboard/dashboard-statement-list.md#others)では、これらの行の情報が`Others`として表示されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- `tidb_stmt_summary_max_stmt_count`: ステートメントサマリーテーブルに保存できるSQLステートメントの数を制限します。デフォルト値は`3000`です。制限を超えると、TiDBは最近使用されなくなったSQLステートメントをクリアします。これらのクリアされたSQLステートメントは`DIGEST`が`NULL`に設定された行として表され、`statements_summary_evicted`テーブルに記録されます。TiDBダッシュボードの[SQLステートメントページ](https://docs.pingcap.com/tidb/stable/dashboard-statement-list#others)では、これらの行の情報が`Others`として表示されます。

</CustomContent>
- `tidb_stmt_summary_max_sql_length`: `DIGEST_TEXT`および`QUERY_SAMPLE_TEXT`の最長表示長を指定します。デフォルト値は`4096`です。
- `tidb_stmt_summary_internal_query`: TiDB SQL文を数えるかどうかを決定します。 `1`は数えることを意味し、`0`は数えないことを意味します。デフォルト値は`0`です。

> **注意:**
>
> `tidb_stmt_summary_max_stmt_count`の制限を超えたためにSQL文カテゴリを削除する必要がある場合、TiDBは`statement_summary_history`テーブルからそのSQL文カテゴリのデータをすべて削除します。したがって、特定の時間範囲内のSQL文カテゴリの数が制限に達しなくても、`statement_summary_history`テーブルに格納されるSQL文の数は実際のSQL文の数よりも少なくなります。このような状況が発生し、パフォーマンスに影響を与える場合、`tidb_stmt_summary_max_stmt_count`の値を増やすことがお勧めされます。

次のようにステートメントの概要構成の例が示されています:

{{< copyable "sql" >}}

```sql
set global tidb_stmt_summary_max_stmt_count = 3000;
set global tidb_enable_stmt_summary = true;
set global tidb_stmt_summary_refresh_interval = 1800;
set global tidb_stmt_summary_history_size = 24;
```

上記の構成が有効になると、`statements_summary`テーブルは30分ごとにクリアされ、`statements_summary_history`テーブルは最大3000種類のSQL文を保存します。各タイプごとに、`statements_summary_history`テーブルは直近の24期間のデータを保存します。`statements_summary_evicted`テーブルには、ステートメントの概要から削除された最近の24期間のSQL文が記録されます。`statements_summary_evicted`テーブルは30分ごとに更新されます。

> **注意:**
>
> - SQLタイプが毎分表示される場合、`statements_summary_history`は最新の12時間のデータを保存します。SQLタイプが毎日00:00から00:30までしか現れない場合、`statements_summary_history`は最新の24期間のデータを保存し、各期間が1日です。したがって、`statements_summary_history`はこのSQLタイプの最新の24日間のデータを保存します。
> - `tidb_stmt_summary_history_size`、`tidb_stmt_summary_max_stmt_count`、`tidb_stmt_summary_max_sql_length`の構成項目はメモリ使用量に影響します。これらの構成は、必要に応じて調整し、SQLサイズ、SQLカウント、およびマシン構成に基づいて調整することがお勧めされます。これらの値をあまり大きな値に設定することはお勧めしません。メモリ使用量は、`tidb_stmt_summary_history_size` \* `tidb_stmt_summary_max_stmt_count` \* `tidb_stmt_summary_max_sql_length` \* `3`を使用して計算できます。

### ステートメントの概要に適切なサイズを設定する

システムが一定の期間（システム負荷に応じる）実行された後、`statement_summary`テーブルを確認してSQLの削除が発生したかどうかを確認できます。例:

```sql
select @@global.tidb_stmt_summary_max_stmt_count;
select count(*) from information_schema.statements_summary;
```

```sql
+-------------------------------------------+
| @@global.tidb_stmt_summary_max_stmt_count |
+-------------------------------------------+
| 3000                                      |
+-------------------------------------------+
1 row in set (0.001 sec)

+----------+
| count(*) |
+----------+
|     3001 |
+----------+
1 row in set (0.001 sec)
```

`statements_summary`テーブルがレコードでいっぱいであることがわかります。その後、`statements_summary_evicted`テーブルから削除されたデータを確認します:

```sql
select * from information_schema.statements_summary_evicted;
```

```sql
+---------------------+---------------------+---------------+
| BEGIN_TIME          | END_TIME            | EVICTED_COUNT |
+---------------------+---------------------+---------------+
| 2020-01-02 16:30:00 | 2020-01-02 17:00:00 |            59 |
+---------------------+---------------------+---------------+
| 2020-01-02 16:00:00 | 2020-01-02 16:30:00 |            45 |
+---------------------+---------------------+---------------+
2 row in set (0.001 sec)
```

上記の結果から、最大59種類のSQLカテゴリが削除されたことがわかります。この場合、`statement_summary`テーブルのサイズを少なくとも59レコード増やすことがお勧めされます。つまり、サイズを少なくとも3059レコードに増やすことがお勧めされます。

## 制限

デフォルトでは、ステートメントの概要テーブルはメモリに保存されます。TiDBサーバーが再起動されると、すべてのデータが失われます。

<CustomContent platform="tidb">

この問題を解決するために、TiDB v6.6.0では [ステートメントの概要の永続化](#persist-statements-summary) 機能が実験的に導入されており、デフォルトでは無効になっています。この機能を有効にした後、履歴データはもはやメモリに保存されず、直接ディスクに書き込まれます。このようにして、履歴データがTiDBサーバーを再起動した場合でも利用可能になります。

</CustomContent>

## ステートメントの概要を永続化

<CustomContent platform="tidb-cloud">

このセクションは TiDB Self-Hosted にのみ適用されます。TiDB Cloudの場合、`tidb_stmt_summary_enable_persistent`パラメータの値はデフォルトで`false`であり、動的な変更はサポートされていません。

</CustomContent>

> **警告:**
>
> ステートメントの概要永続化は実験的な機能です。本番環境で使用することはお勧めできません。この機能は、事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告することができます。

<CustomContent platform="tidb">

[制限](#制限)セクションで述べたように、デフォルトではステートメントの概要テーブルはメモリに保存されます。TiDB v6.6.0以降、TiDBはステートメントの概要永続化を有効または無効にするための新しい設定項目 [`tidb_stmt_summary_enable_persistent`](/tidb-configuration-file.md#tidb_stmt_summary_enable_persistent-new-in-v660) を実験的に提供しています。

</CustomContent>

<CustomContent platform="tidb-cloud">

[制限](#制限)セクションで述べたように、デフォルトではステートメントの概要テーブルはメモリに保存されます。TiDB v6.6.0以降、TiDBはステートメントの概要永続化を有効または無効にするための新しい設定項目 `tidb_stmt_summary_enable_persistent` を提供しています。

</CustomContent>

ステートメントの概要永続化を有効にするには、次の構成項目をTiDBの設定ファイルに追加できます:

```toml
[instance]
tidb_stmt_summary_enable_persistent = true
# 次のエントリはデフォルト値を使用しており、必要に応じて変更できます。
# tidb_stmt_summary_filename = "tidb-statements.log"
# tidb_stmt_summary_file_max_days = 3
# tidb_stmt_summary_file_max_size = 64 # MiB
# tidb_stmt_summary_file_max_backups = 0
```

ステートメントの概要永続化が有効になると、メモリには現在のリアルタイムデータのみが保存され、履歴データは書き込まれます。この履歴データは、[パラメータの設定](#parameter-configuration)セクションで説明されている`tidb_stmt_summary_refresh_interval`の間隔でディスクに書き込まれます。`statements_summary_history`や`cluster_statements_summary_history`テーブルのクエリは、メモリとディスクの両方のデータを組み合わせて返します。

<CustomContent platform="tidb">

> **注意:**
>
> - ステートメントの概要永続化が有効になると、[パラメータの設定](#parameter-configuration)セクションで説明されている`tidb_stmt_summary_history_size`構成はもはや影響を与えません。なぜならば、メモリは履歴データを保持しないからです。代わりに、次の3つの構成が永続性のための履歴データの保持期間とサイズを制御するために使用されます: [`tidb_stmt_summary_file_max_days`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_days-new-in-v660), [`tidb_stmt_summary_file_max_size`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_size-new-in-v660)、および[`tidb_stmt_summary_file_max_backups`](/tidb-configuration-file.md#tidb_stmt_summary_file_max_backups-new-in-v660)。
> - `tidb_stmt_summary_refresh_interval`の値が小さいほど、より即座にデータがディスクに書き込まれます。ただし、これはディスクに余分なデータが書き込まれることを意味します。

</CustomContent>

## トラブルシューティングの例

このセクションでは、ステートメントの概要機能を使用してSQLパフォーマンスの問題をトラブルシューティングする方法を示す2つの例を提供しています。

### 高いSQL遅延はサーバーエンドの原因かもしれませんか？

この例では、クライアントは`employee`テーブルに対するポイントクエリのパフォーマンスが遅いことを示しています。SQLのテキストをぼかし検索を実行できます:

{{< copyable "sql" >}}

```sql
SELECT avg_latency, exec_count, query_sample_text
    FROM information_schema.statements_summary
    WHERE digest_text LIKE 'select * from employee%';
```

`1ms`および`0.3ms`は`avg_latency`の正常範囲内と見なされます。したがって、サーバーエンドが原因ではないと結論付けることができます。クライアントまたはネットワークでトラブルシューティングできます。
```sql
+-------------+------------+------------------------------------------+
| 平均待機時間 | 実行回数   | クエリのサンプルテキスト                       |
+-------------+------------+------------------------------------------+
|    1042040  |          2 | select * from employee where name='eric' |
|     345053  |          3 | select * from employee where id=3100     |
+-------------+------------+------------------------------------------+
2 行がセットされました (0.00 秒)

### どの種類のSQLステートメントが最も時間を消費していますか？

QPSが10:00から10:30にかけて大幅に低下した場合、履歴テーブルから最も時間を消費する3つのSQLステートメントのカテゴリを見つけることができます。

```sql
SELECT sum_latency, avg_latency, exec_count, query_sample_text
    FROM information_schema.statements_summary_history
    WHERE summary_begin_time='2020-01-02 10:00:00'
    ORDER BY sum_latency DESC LIMIT 3;
```

結果では、以下の3つのSQLステートメントカテゴリが総合的に最も時間を消費していることが示され、高い優先度で最適化が必要です。

```sql
+-------------+-------------+------------+-----------------------------------------------------------------------+
| sum_latency | avg_latency | exec_count | query_sample_text                                                     |
+-------------+-------------+------------+-----------------------------------------------------------------------+
|     7855660 |     1122237 |          7 | select avg(salary) from employee where company_id=2013                |
|     7241960 |     1448392 |          5 | select * from employee join company on employee.company_id=company.id |
|     2084081 |     1042040 |          2 | select * from employee where name='eric'                              |
+-------------+-------------+------------+-----------------------------------------------------------------------+
3 行がセットされました (0.00 秒)

## フィールドの説明

### `statements_summary` フィールドの説明

以下は `statements_summary` テーブル内のフィールドの説明です。

基本フィールド:

- `STMT_TYPE`：SQLステートメントのタイプ
- `SCHEMA_NAME`：このカテゴリのSQLステートメントが実行される現在のスキーマ
- `DIGEST`：このカテゴリのSQLステートメントのダイジェスト
- `DIGEST_TEXT`：標準化されたSQLステートメント
- `QUERY_SAMPLE_TEXT`：SQLカテゴリの元のSQLステートメント。元のステートメントは1つだけ取得されます。
- `TABLE_NAMES`：SQLステートメントに関与するすべてのテーブル。複数のテーブルがある場合、それぞれがコンマで区切られます。
- `INDEX_NAMES`：SQLステートメントで使用されているすべてのSQLインデックス。複数のインデックスがある場合、それぞれがコンマで区切られます。
- `SAMPLE_USER`：このカテゴリのSQLステートメントを実行するユーザー。1つだけのユーザーが取得されます。
- `PLAN_DIGEST`：実行プランのダイジェスト
- `PLAN`：元の実行プラン。複数のステートメントがある場合、1つのステートメントのプランが取得されます。
- `BINARY_PLAN`：バイナリ形式でエンコードされた元の実行プラン。複数のステートメントがある場合、1つのステートメントのプランが取得されます。特定の実行プランを解析するには、`SELECT tidb_decode_binary_plan('xxx...')` ステートメントを実行します。
- `PLAN_CACHE_HITS`：このカテゴリのSQLステートメントがプランキャッシュにヒットした回数の合計。
- `PLAN_IN_CACHE`：以前のこのカテゴリのSQLステートメントの実行がプランキャッシュにヒットしたかどうかを示します。

実行時間に関連するフィールド:

- `SUMMARY_BEGIN_TIME`：現在のサマリ期間の開始時間
- `SUMMARY_END_TIME`：現在のサマリ期間の終了時間
- `FIRST_SEEN`：このカテゴリのSQLステートメントが最初に見られた時刻
- `LAST_SEEN`：このカテゴリのSQLステートメントが最後に見られた時刻

TiDBサーバーに関連するフィールド:

- `EXEC_COUNT`：このカテゴリのSQLステートメントの合計実行回数
- `SUM_ERRORS`：実行中に発生したエラーの合計数
- `SUM_WARNINGS`：実行中に発生した警告の合計数
- `SUM_LATENCY`：このカテゴリのSQLステートメントの合計実行待ち時間
- `MAX_LATENCY`：このカテゴリのSQLステートメントの最大実行待ち時間
- `MIN_LATENCY`：このカテゴリのSQLステートメントの最小実行待ち時間
- `AVG_LATENCY`：このカテゴリのSQLステートメントの平均実行待ち時間
- `AVG_PARSE_LATENCY`：パーサの平均待ち時間
- `MAX_PARSE_LATENCY`：パーサの最大待ち時間
- `AVG_COMPILE_LATENCY`：コンパイラの平均待ち時間
- `MAX_COMPILE_LATENCY`：コンパイラの最大待ち時間
- `AVG_MEM`：平均使用メモリ容量（バイト）
- `MAX_MEM`：最大使用メモリ容量（バイト）
- `AVG_DISK`：平均ディスク使用量（バイト）
- `MAX_DISK`：最大ディスク使用量（バイト）

TiKV Coprocessorタスクに関連するフィールド:

- `SUM_COP_TASK_NUM`：送信されたCoprocessorリクエストの合計数
- `MAX_COP_PROCESS_TIME`：Coprocessorタスクの最大実行時間
- `MAX_COP_PROCESS_ADDRESS`：最大実行時間を持つCoprocessorタスクのアドレス
- `MAX_COP_WAIT_TIME`：Coprocessorタスクの最大待ち時間
- `MAX_COP_WAIT_ADDRESS`：最大待ち時間を持つCoprocessorタスクのアドレス
- `AVG_PROCESS_TIME`：TiKV内のSQLステートメントの平均処理時間
- `MAX_PROCESS_TIME`：TiKV内のSQLステートメントの最大処理時間
- `AVG_WAIT_TIME`：TiKV内のSQLステートメントの平均待ち時間
- `MAX_WAIT_TIME`：TiKV内のSQLステートメントの最大待ち時間
- `AVG_BACKOFF_TIME`：SQLステートメントがリトライを必要とするエラーに遭遇した際のリトライ前の平均待ち時間
- `MAX_BACKOFF_TIME`：SQLステートメントがリトライを必要とするエラーに遭遇した際のリトライ前の最大待ち時間
- `AVG_TOTAL_KEYS`：Coprocessorがスキャンした平均キー数
- `MAX_TOTAL_KEYS`：Coprocessorがスキャンした最大キー数
- `AVG_PROCESSED_KEYS`：Coprocessorが処理した平均キー数。`avg_total_keys`と比較すると、`avg_processed_keys`にはMVCCの古いバージョンは含まれません。`avg_total_keys`と`avg_processed_keys`の大きな差は多くの古いバージョンが存在することを示します。
- `MAX_PROCESSED_KEYS`：Coprocessorが処理した最大キー数

トランザクションに関連するフィールド:

（略）
### `statements_summary_evicted` フィールドの説明

- `BEGIN_TIME`：開始時刻を記録
- `END_TIME`：終了時刻を記録
- `EVICTED_COUNT`：記録期間中に追い出されたSQLカテゴリの数
```