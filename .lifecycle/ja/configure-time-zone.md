---
title: タイムゾーンのサポート
summary: 時刻帯とその形式の設定方法について学ぶ
aliases: ['/docs/dev/configure-time-zone/', '/docs/dev/how-to/configure/time-zone/']
---

# タイムゾーンのサポート

TiDBにおけるタイムゾーンは、グローバル`time_zone`システム変数とセッションの`time_zone`システム変数によって決定されます。`time_zone`のデフォルト値は`SYSTEM`です。`System`に対応する実際のタイムゾーンは、TiDBクラスターのブートストラップが初期化される際に構成されます。詳しいロジックは以下の通りです:

- `TZ`環境変数の使用を優先します。
- もし`TZ`環境変数が失敗した場合は、`/etc/localtime`の実際のシンボリックリンクアドレスからタイムゾーンを抽出します。
- 上記の方法のいずれも失敗した場合は、システムのタイムゾーンとして`UTC`を使用します。

以下の文を使用して、実行時にグローバルサーバー`time_zone`の値を設定できます:

{{< copyable "sql" >}}

```sql
SET GLOBAL time_zone = タイムゾーン;
```

各クライアントはそれぞれ独自のタイムゾーン設定を持ち、それはセッションの`time_zone`変数によって与えられます。初期状態では、セッション変数はグローバル`time_zone`変数から値を取りますが、クライアントは次の文を使用して独自のタイムゾーンを変更できます:

{{< copyable "sql" >}}

```sql
SET time_zone = タイムゾーン;
```

以下の文を使用して、グローバル、クライアント固有、およびシステムの現在のタイムゾーンの値を表示できます：

{{< copyable "sql" >}}

```sql
SELECT @@global.time_zone, @@session.time_zone, @@global.system_time_zone;
```

`time_zone`の値のフォーマットを設定するには:

- 'SYSTEM'という値は、システムのタイムゾーンと同じであることを示します。
- '+10:00'や'-6:00'など、UTCからのオフセットを示す文字列で値を与えることができます。
- 'Europe/Helsinki'、'US/Eastern'、'MET'など、名前付きのタイムゾーンで値を与えることができます。

現在のセッションのタイムゾーン設定は、ゾーンに敏感な時刻値の表示および保存に影響します。これには、`NOW()`や`CURTIME()`といった関数によって表示される値が含まれます。

> **Note:**
>
> タイムゾーンの変更によって影響を受けるのは、Timestampデータ型の値だけです。これはTimestampデータ型がリテラル値+タイムゾーン情報を使用しているためです。Datetime/Date/Timeなどの他のデータ型にはタイムゾーン情報がないため、彼らの値はタイムゾーンの変更によって影響を受けません。

{{< copyable "sql" >}}

```sql
create table t (ts timestamp, dt datetime);
```

```
Query OK, 0 rows affected (0.02 sec)
```

{{< copyable "sql" >}}

```sql
set @@time_zone = 'UTC';
```

```
Query OK, 0 rows affected (0.00 sec)
```

{{< copyable "sql" >}}

```sql
insert into t values ('2017-09-30 11:11:11', '2017-09-30 11:11:11');
```

```
Query OK, 1 row affected (0.00 sec)
```

{{< copyable "sql" >}}

```sql
set @@time_zone = '+8:00';
```

```
Query OK, 0 rows affected (0.00 sec)
```

{{< copyable "sql" >}}

```sql
select * from t;
```

```
+---------------------|---------------------+
| ts                  | dt                  |
+---------------------|---------------------+
| 2017-09-30 19:11:11 | 2017-09-30 11:11:11 |
+---------------------|---------------------+
1 row in set (0.00 sec)
```

この例では、タイムゾーンの値をいかに調整してもDatetimeデータ型の値には影響がありません。しかし、タイムゾーン情報が変化するとTimestampデータ型の表示値が変わります。実際には、ストレージに格納された値は変わらず、異なるタイムゾーン設定に応じて異なる表示がされているだけです。

> **Note:**
>
> - TimestampおよびDatetimeの値の変換中にタイムゾーンが絡むため、これはセッションの現在の`time_zone`に基づいて処理されます。
> - データの移行では、プライマリデータベースとセカンダリデータベースのタイムゾーン設定に特に注意する必要があります。