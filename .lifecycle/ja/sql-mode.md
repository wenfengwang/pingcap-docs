---
title: SQLモード
summary: SQLモードを学ぶ
aliases: ['/docs/dev/sql-mode/','/docs/dev/reference/sql/sql-mode/']
---

# SQLモード

TiDBサーバーは異なるSQLモードで動作し、これらのモードを異なるクライアントに適用します。SQLモードはTiDBがサポートするSQL構文と、実行するデータ検証チェックの種類を定義します。以下に説明します。

TiDBを起動した後、`SET [ SESSION | GLOBAL ] sql_mode='modes'`を変更してSQLモードを設定します。

SQLモードを`GLOBAL`レベルで設定する場合は、`SUPER`権限が必要であり、このレベルでの設定はその後に確立された接続にのみ影響します。`SESSION`レベルでのSQLモードの変更は現在のクライアントにのみ影響します。

`Modes`はコンマ(',')で区切られた異なるモードのシリーズです。現在のSQLモードを確認するには`SELECT @@sql_mode`ステートメントを使用できます。SQLモードのデフォルト値: `ONLY_FULL_GROUP_BY, STRICT_TRANS_TABLES, NO_ZERO_IN_DATE, NO_ZERO_DATE, ERROR_FOR_DIVISION_BY_ZERO, NO_AUTO_CREATE_USER, NO_ENGINE_SUBSTITUTION`。

## 重要な`sql_mode`の値

* `ANSI`: このモードは標準SQLに準拠しています。このモードでは、データがチェックされます。データが定義されたタイプや長さに準拠していない場合、データ型が調整されたりトリムされ、`warning`が返されます。
* `STRICT_TRANS_TABLES`: データが厳密にチェックされる厳密モードです。データに問題があると、テーブルに挿入できず、エラーが返されます。
* `TRADITIONAL`: このモードでは、TiDBは「traditional」なSQLデータベースシステムのように振る舞います。列に不正な値が挿入された場合、警告ではなくエラーが返されます。その後、`INSERT`または`UPDATE`ステートメントは直ちに停止されます。

## SQLモードテーブル

| 名前 | 説明 |
| :--- | :--- |
| `PIPES_AS_CONCAT` | "\|\|"を文字列連結演算子 (`+`) として扱います（`CONCAT()`と同じ）。`OR`として扱われません（完全なサポート） |
| `ANSI_QUOTES` | `"`を識別子として扱います。 `ANSI_QUOTES`が有効になっていると、単一引用符は文字列リテラルとして扱われ、二重引用符は識別子として扱われます。したがって、二重引用符で文字列を引用することはできません（完全なサポート）|
| `IGNORE_SPACE` | このモードが有効になっていると、システムは空白を無視します。例: "user"と"user "は同じです（完全なサポート）|
| `ONLY_FULL_GROUP_BY` | `SELECT`、`HAVING`、または`ORDER BY`で参照される非集約列が`GROUP BY`に存在しない場合、このSQLステートメントは無効です。`GROUP BY`に存在しない列がクエリで表示されるのは異常ですから。 (完全なサポート) |
| `NO_UNSIGNED_SUBTRACTION` | 減算でオペランドに記号がない場合、結果を`UNSIGNED`とマークしません（完全なサポート）|
| `NO_DIR_IN_CREATE` | テーブルが作成されるとき、すべての`INDEX DIRECTORY`および`DATA DIRECTORY`指示を無視します。このオプションは、セカンダリ複製サーバーにのみ有用です（構文サポートのみ） |
| `NO_KEY_OPTIONS` | `SHOW CREATE TABLE`ステートメントを使用する場合、`ENGINE`などのMySQL固有の構文はエクスポートされません。mysql_dumpを使用してDBタイプ間を移行する際にはこのオプションを検討してください（構文サポートのみ）|
| `NO_FIELD_OPTIONS` | `SHOW CREATE TABLE`ステートメントを使用する場合、`ENGINE`などのMySQL固有の構文はエクスポートされません。mysql_dumpを使用してDBタイプ間を移行する際にはこのオプションを検討してください（構文サポートのみ） |
| `NO_TABLE_OPTIONS` | `SHOW CREATE TABLE`ステートメントを使用する場合、`ENGINE`などのMySQL固有の構文はエクスポートされません。mysql_dumpを使用してDBタイプ間を移行する際にはこのオプションを検討してください（構文サポートのみ）|
| `NO_AUTO_VALUE_ON_ZERO` | このモードが有効になっている場合、`AUTO_INCREMENT`列に渡された値が`0`または特定の値の場合、システムは直接この値をこの列に書き込みます。`NULL`が渡された場合、システムは次の連番を自動的に生成します（完全なサポート）|
| `NO_BACKSLASH_ESCAPES` | このモードが有効になっている場合、`\`バックスラッシュ記号は単なる自身を表します（完全なサポート）|
| `STRICT_TRANS_TABLES` | 不正な値が挿入された後、トランザクションストレージエンジンに対する厳密モードを有効にし、全文をロールバックします（完全なサポート） |
| `STRICT_ALL_TABLES` | トランザクションテーブルでは、不正な値が挿入された後、全文をロールバックします（完全なサポート） |
| `NO_ZERO_IN_DATE` | 月や日の部分が`0`の日付は厳密モードでは受け入れられません。`IGNORE`オプションを使用すると、TiDBは類似の日付に'0000-00-00'を挿入します。非厳密モードではこの日付は受け入れられますが、警告が返されます（完全なサポート）
| `NO_ZERO_DATE` | 厳密モードでは、'0000-00-00'を合法な日付として使用しません。`IGNORE`オプションでゼロ日付を挿入できます。非厳密モードではこの日付は受け入れられますが、警告が返されます（完全なサポート）|
| `ALLOW_INVALID_DATES` | このモードでは、すべての日付の有効性をチェックしません。月の値は`1`から`12`の範囲、日の値は`1`から`31`の範囲だけをチェックします。このモードは`DATE`と`DATATIME`列にのみ適用されます。すべての`TIMESTAMP`列は完全な有効性チェックが必要です（完全なサポート） |
| `ERROR_FOR_DIVISION_BY_ZERO` | このモードが有効になっている場合、データ変更操作（`INSERT`または`UPDATE`）で`0`による除算を処理する際にシステムはエラーを返します。<br/>このモードが有効になっていない場合、システムは警告とともに`NULL`を使用します（完全なサポート） |
| `NO_AUTO_CREATE_USER` | 任意のパスワードを除いて`GRANT`が新規ユーザーを自動的に作成しないようにします（完全なサポート）|
| `HIGH_NOT_PRECEDENCE` | NOT演算子の優先度は、`NOT a BETWEEN b AND c`のような式が`NOT (a BETWEEN b AND c)`として解析されます。古いバージョンのMySQLでは、この式は`(NOT a) BETWEEN b AND c`として解析されます（完全なサポート） |
| `NO_ENGINE_SUBSTITUTION` | 必要なストレージエンジンが無効になっているかコンパイルされていない場合、ストレージエンジンの自動置換を防止します（構文サポートのみ）|
| `PAD_CHAR_TO_FULL_LENGTH` | このモードが有効になっている場合、システムは`CHAR`型のトレーリングスペースをトリミングしません。（構文サポートのみ。このモードは[MySQL 8.0で非推奨](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_pad_char_to_full_length).） |
| `REAL_AS_FLOAT` | `REAL`を`DOUBLE`の同義語ではなく`FLOAT`の同義語として扱います（完全なサポート）|
| `POSTGRESQL` | `PIPES_AS_CONCAT`、`ANSI_QUOTES`、`IGNORE_SPACE`、`NO_KEY_OPTIONS`、`NO_TABLE_OPTIONS`、`NO_FIELD_OPTIONS`に相当（構文サポートのみ）|
| `MSSQL` | `PIPES_AS_CONCAT`、`ANSI_QUOTES`、`IGNORE_SPACE`、`NO_KEY_OPTIONS`、`NO_TABLE_OPTIONS`、`NO_FIELD_OPTIONS`に相当（構文サポートのみ）|
| `DB2` | `PIPES_AS_CONCAT`、`ANSI_QUOTES`、`IGNORE_SPACE`、`NO_KEY_OPTIONS`、`NO_TABLE_OPTIONS`、`NO_FIELD_OPTIONS`に相当（構文サポートのみ）|
| `MAXDB` | `PIPES_AS_CONCAT`、`ANSI_QUOTES`、`IGNORE_SPACE`、`NO_KEY_OPTIONS`、`NO_TABLE_OPTIONS`、`NO_FIELD_OPTIONS`、`NO_AUTO_CREATE_USER`に相当（完全なサポート）|
| `MySQL323` | `NO_FIELD_OPTIONS`、`HIGH_NOT_PRECEDENCE`に相当（構文サポートのみ）|
| `MYSQL40` | `NO_FIELD_OPTIONS`、`HIGH_NOT_PRECEDENCE`に相当（構文サポートのみ）|
| `ANSI` | `REAL_AS_FLOAT`、`PIPES_AS_CONCAT`、`ANSI_QUOTES`、`IGNORE_SPACE`に相当（構文サポートのみ）|
| `TRADITIONAL` | `STRICT_TRANS_TABLES`、`STRICT_ALL_TABLES`、`NO_ZERO_IN_DATE`、`NO_ZERO_DATE`、`ERROR_FOR_DIVISION_BY_ZERO`、`NO_AUTO_CREATE_USER`に相当（構文サポートのみ） |
| `ORACLE` | `PIPES_AS_CONCAT`、`ANSI_QUOTES`、`IGNORE_SPACE`、`NO_KEY_OPTIONS`、`NO_TABLE_OPTIONS`、`NO_FIELD_OPTIONS`、`NO_AUTO_CREATE_USER`に相当（構文のみ）|