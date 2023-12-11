---
title: 日付と時刻の関数
summary: 日付と時刻の関数の使い方を学びます。
aliases: ['/docs/dev/functions-and-operators/date-and-time-functions/','/docs/dev/reference/sql/functions-and-operators/date-and-time-functions/']
---

# 日付と時刻の関数

TiDBはMySQL 5.7で利用可能な[日付と時刻の関数](https://dev.mysql.com/doc/refman/5.7/en/numeric-functions.html)をすべてサポートしています。

> **注意:**
>
> - MySQLはしばしば誤ってフォーマットされた日付と時刻の値を受け入れます。例えば、`'2020-01-01\n\t01:01:01'`や`'2020-01_01\n\t01:01'`は有効な日付と時刻の値として扱われます。
> - TiDBはMySQLの動作に最善の努力をして一致させますが、すべてのケースで一致しない場合があります。誤ってフォーマットされた値の意図された動作は文書化されておらず、しばしば一貫性がありませんので、正しくフォーマットされた日付を推奨します。

**日時関数:**

| 名前                                     | 説明                              |
| ---------------------------------------- | ---------------------------------------- |
| [`ADDDATE()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_adddate) | 日付値に時間値（インターバル）を追加 |
| [`ADDTIME()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_addtime) | 時間を追加                                 |
| [`CONVERT_TZ()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_convert-tz) | 1つのタイムゾーンから別のタイムゾーンに変換    |
| [`CURDATE()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_curdate) | 現在の日付を返す                  |
| [`CURRENT_DATE()`, `CURRENT_DATE`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_current-date) | CURDATE()の同義語                   |
| [`CURRENT_TIME()`, `CURRENT_TIME`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_current-time) | CURTIME()の同義語                   |
| [`CURRENT_TIMESTAMP()`, `CURRENT_TIMESTAMP`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_current-timestamp) | NOW()の同義語                       |
| [`CURTIME()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_curtime) | 現在の時刻を返す                  |
| [`DATE()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_date) | 日付または日時式の日付部分を抽出 |
| [`DATE_ADD()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_date-add) | 日付値に時間値（インターバル）を追加 |
| [`DATE_FORMAT()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_date-format) | 指定された形式で日付をフォーマット                 |
| [`DATE_SUB()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_date-sub) | 日付から時間値（インターバル）を減算 |
| [`DATEDIFF()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_datediff) | 2つの日付を減算                       |
| [`DAY()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_day) | DAYOFMONTH()の同義語                |
| [`DAYNAME()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_dayname) | 曜日の名前を返す           |
| [`DAYOFMONTH()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_dayofmonth) | 月の日（0-31）を返す       |
| [`DAYOFWEEK()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_dayofweek) | 引数の曜日インデックスを返す |
| [`DAYOFYEAR()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_dayofyear) | 年の日（1-366）を返す       |
| [`EXTRACT()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_extract) | 日付の一部を抽出                   |
| [`FROM_DAYS()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_from-days) | 日数を日付に変換           |
| [`FROM_UNIXTIME()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_from-unixtime) | Unixタイムスタンプを日付にフォーマット          |
| [`GET_FORMAT()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_get-format) | 日付の形式文字列を返す              |
| [`HOUR()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_hour) | 時を抽出                         |
| [`LAST_DAY`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_last-day) | 引数の月の最終日を返す |
| [`LOCALTIME()`, `LOCALTIME`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_localtime) | NOW()の同義語                     |
| [`LOCALTIMESTAMP`, `LOCALTIMESTAMP()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_localtimestamp) | NOW()の同義語                     |
| [`MAKEDATE()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_makedate) | 年と年の日から日付を作成 |
| [`MAKETIME()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_maketime) | 時間を時、分、秒から作成    |
| [`MICROSECOND()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_microsecond) | 引数からマイクロ秒を返す    |
| [`MINUTE()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_minute) | 引数から分を返す      |
| [`MONTH()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_month) | 渡された日付から月を返す    |
| [`MONTHNAME()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_monthname) | 月の名前を返す             |
| [`NOW()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_now) | 現在の日付と時刻を返す         |
| [`PERIOD_ADD()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_period-add) | 年-月に期間を追加       |
| [`PERIOD_DIFF()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_period-diff) | 期間間の月数を返す     |
| [`QUARTER()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_quarter) | 日付引数から四半期を返す  |
| [`SEC_TO_TIME()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_sec-to-time) | 秒を'HH:MM:SS'形式に変換    |
| [`SECOND()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_second) | 秒（0-59）を返す         |
| [`STR_TO_DATE()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_str-to-date) | 文字列を日付に変換       |
| [`SUBDATE()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_subdate) | 3つの引数で呼び出された場合のDATE_SUB()の同義語 |
| [`SUBTIME()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_subtime) | 時間を減算                     |
| [`SYSDATE()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_sysdate) | 関数が実行された時刻を返す |
| [`TIME()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_time) | 渡された式の時間部分を抽出 |
| [`TIME_FORMAT()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_time-format) | 時刻としてフォーマット         |
| [`TIME_TO_SEC()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_time-to-sec) | 秒に変換                   |
| [`TIMEDIFF()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_timediff) | 時間を減算             |
| [`TIMESTAMP()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_timestamp) | 引数が1つの場合、この関数は日付または日時の表現を返します。引数が2つの場合、引数の合計を返します。 |
| [`TIMESTAMPADD()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_timestampadd) | 日時の表現に間隔を追加します |
| [`TIMESTAMPDIFF()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_timestampdiff) | 日時の表現から間隔を減算します |
| [`TO_DAYS()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_to-days) | 日付引数を日数に変換して返します |
| [`TO_SECONDS()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_to-seconds) | 年0からの秒数に変換した日付または日時引数を返します |
| [`UNIX_TIMESTAMP()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_unix-timestamp) | Unixタイムスタンプを返します                  |
| [`UTC_DATE()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_utc-date) | 現在のUTC日付を返します              |
| [`UTC_TIME()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_utc-time) | 現在のUTC時間を返します              |
| [`UTC_TIMESTAMP()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_utc-timestamp) | 現在のUTC日付と時間を返します     |
| [`WEEK()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_week) | 週番号を返します                   |
| [`WEEKDAY()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_weekday) | 曜日インデックスを返します                 |
| [`WEEKOFYEAR()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_weekofyear) | 日付のカレンダー週を返します（1-53） |
| [`YEAR()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_year) | 年を返します                          |
| [`YEARWEEK()`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_yearweek) | 年と週を返します                 |

詳細については、[日付および時刻関数](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html)を参照してください。

## MySQL互換性

`str_to_date()` 関数はTiDBでサポートされていますが、すべての日付と時刻の値を解析することはできません。さらに、次の日付および時刻の書式オプションは**実装されていません**:

| 書式 | 説明                                                                           |
|--------|---------------------------------------------------------------------------------------|
| "%a"   | 省略された曜日の名前 (日曜日..土曜日)                                                   |
| "%D"   | 英語の接尾辞付き月の日 (0th, 1st, 2nd, 3rd)                             |
| "%U"   | 週 (00..53)、日曜日を週の最初の日とする; WEEK() mode 0               |
| "%u"   | 週 (00..53)、月曜日を週の最初の日とする; WEEK() mode 1               |
| "%V"   | 週 (01..53)、日曜日を週の最初の日とする; WEEK() mode 2; %X と一緒に使用 |
| "%v"   | 週 (01..53)、月曜日を週の最初の日とする; WEEK() mode 3; %x と一緒に使用 |
| "%W"   | 曜日名 (日曜日..土曜日)                                                       |
| "%w"   | 曜日 (0=日曜日..6=土曜日)                                                |
| "%X"   | 週の年、日曜日を週の最初の日とする、数字、4桁    |
| "%x"   | 週の年、月曜日を週の最初の日とする、数字、4桁   |

詳細については、[issue #30082](https://github.com/pingcap/tidb/issues/30082) を参照してください。

## 関連するシステム変数

`default_week_format` 変数は `WEEK()` 関数に影響を与えます。