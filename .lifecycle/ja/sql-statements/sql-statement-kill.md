---
title: KILL
summary: TiDBデータベースでのKILLの使用の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-kill/', '/docs/dev/reference/sql/statements/kill/']
---

# KILL

`KILL`ステートメントは、現在のTiDBクラスタ内の任意のTiDBインスタンスで接続を終了するために使用されます。

## 概要

```ebnf+diagram
KillStmt ::= 'KILL' 'TIDB'? ( 'CONNECTION' | 'QUERY' )? CONNECTION_ID
```

## 例

次の例は、現在のクラスタ内のすべてのアクティブなクエリを取得し、それらのいずれかを終了する方法を示しています。

{{< copyable "sql" >}}

```sql
SELECT ID, USER, INSTANCE, INFO FROM INFORMATION_SCHEMA.CLUSTER_PROCESSLIST;
```

```
+---------------------+------+-----------------+-----------------------------------------------------------------------------+
| ID                  | USER | INSTANCE        | INFO                                                                        |
+---------------------+------+-----------------+-----------------------------------------------------------------------------+
| 8306449708033769879 | root | 127.0.0.1:10082 | select sleep(30), 'foo'                                                     |
| 5857102839209263511 | root | 127.0.0.1:10080 | select sleep(50)                                                            |
| 5857102839209263513 | root | 127.0.0.1:10080 | SELECT ID, USER, INSTANCE, INFO FROM INFORMATION_SCHEMA.CLUSTER_PROCESSLIST |
+---------------------+------+-----------------+-----------------------------------------------------------------------------+
```

{{< copyable "sql" >}}

```sql
KILL 5857102839209263511;
```

```
Query OK, 0 rows affected (0.00 sec)
```

## MySQL互換性

- MySQLの`KILL`ステートメントは現在接続しているMySQLインスタンスでのみ接続を終了できますが、TiDBの`KILL`ステートメントはクラスタ全体の任意のTiDBインスタンスで接続を終了できます。
- v7.2.0およびそれ以前のバージョンでは、TiDBにおいてMySQLのコマンドラインの<kbd>Control+C</kbd>を使用してクエリまたは接続を終了することはサポートされていません。

## 動作の変更内容

<CustomContent platform="tidb">

v7.3.0から、TiDBではデフォルトで32ビット接続IDを生成するようになり、これは[`enable-32bits-connection-id`](/tidb-configuration-file.md#enable-32bits-connection-id-new-in-v730)設定項目で制御されます。グローバルキル機能と32ビット接続IDが両方有効な場合、TiDBは32ビット接続IDを生成し、MySQLコマンドラインで<kbd>Control+C</kbd>を使用してクエリまたは接続を終了できます。

> **警告:**
>
> クラスタ内のTiDBインスタンス数が2048を超えた場合、または単一のTiDBインスタンスの同時接続数が1048576を超えた場合、32ビット接続IDのスペースが不足し、自動的に64ビット接続IDにアップグレードされます。このアップグレードプロセス中、既存のビジネスと確立した接続に影響はありません。ただし、それ以降の新しい接続はMySQLコマンドラインで<kbd>Control+C</kbd>を使用して終了できません。

v6.1.0から、TiDBはグローバルキル機能をサポートし、デフォルトで有効になり、[`enable-global-kill`](/tidb-configuration-file.md#enable-global-kill-new-in-v610)設定で制御されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

v7.3.0から、TiDBではデフォルトで32ビット接続IDを生成するようになります。グローバルキル機能と32ビット接続IDが両方有効な場合、MySQLコマンドラインで<kbd>Control+C</kbd>を使用してクエリまたは接続を終了できます。

v6.1.0から、TiDBはグローバルキル機能をサポートし、デフォルトで有効になります。

</CustomContent>

グローバルキル機能が有効になっている場合、`KILL`および`KILL TIDB`ステートメントはインスタンス間でのクエリまたは接続を終了できるため、誤ってクエリまたは接続を終了する心配はありません。クライアントが任意のTiDBインスタンスに接続し、`KILL`または`KILL TIDB`ステートメントを実行すると、そのステートメントは対象のTiDBインスタンスに転送されます。クライアントとTiDBクラスタの間にプロキシがある場合、`KILL`および`KILL TIDB`ステートメントも対象のTiDBインスタンスに転送されて実行されます。

グローバルキル機能が無効になっている場合、またはv6.1.0より前のTiDBを使用している場合は、次のことに注意してください。

- デフォルトでは、`KILL`はMySQLと互換性がありません。これは、複数のTiDBサーバーをロードバランサーの後ろに配置することが一般的であるため、間違ったTiDBサーバーによって接続が終了するケースを防ぐためです。現在接続しているTiDBインスタンスで他の接続を終了するには、`KILL TIDB`ステートメントを実行する際に明示的に`TIDB`接尾辞を追加する必要があります。

<CustomContent platform="tidb">

- クライアントが常に同じTiDBインスタンスに接続することが確実である場合を除いては、[`compatible-kill-query = true`](/tidb-configuration-file.md#compatible-kill-query)を設定ファイルに設定しないでください。これは、デフォルトのMySQLクライアントで<kbd>Control+C</kbd>を押すと新しい接続が開かれ、その新しい接続で`KILL`が実行されるためです。クライアントとTiDBクラスタの間にプロキシがある場合、新しい接続が異なるTiDBインスタンスにルーティングされる可能性があるため、誤って別のセッションを終了する可能性があります。

</CustomContent>

- `KILL TIDB`ステートメントはTiDBの拡張機能です。このステートメントの機能はMySQLの`KILL [CONNECTION|QUERY]`コマンドやMySQLコマンドラインの<kbd>Control+C</kbd>と類似しています。同じTiDBインスタンスで`KILL TIDB`を使用するのは安全です。

## 関連項目

* [SHOW \[FULL\] PROCESSLIST](/sql-statements/sql-statement-show-processlist.md)
* [CLUSTER_PROCESSLIST](/information-schema/information-schema-processlist.md#cluster_processlist)