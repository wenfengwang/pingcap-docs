---
title: ADMIN CLEANUP INDEX
summary: TiDBデータベースでのADMIN CLEANUPの使用法の概要。

# ADMIN CLEANUP INDEX

`ADMIN CLEANUP INDEX`ステートメントは、テーブルが不整合なデータとインデックスを持っているときに、テーブルから冗長なインデックスを削除するために使用されます。この構文はまだ [外部キー制約](/foreign-key.md) をサポートしていません。

## 概要

```ebnf+diagram
AdminCleanupStmt ::=
    'ADMIN' 'CLEANUP' ( 'INDEX' TableName IndexName | 'TABLE' 'LOCK' TableNameList )

TableNameList ::=
    TableName ( ',' TableName )*
```

## 例

データベース内の `tbl` テーブルが何らかの理由で不整合なデータとインデックスを持つと仮定します（たとえば、クラスター内での災害復旧シナリオでのいくつかの行データの喪失）：

```sql
SELECT * FROM tbl;
ERROR 1105 (HY000): inconsistent index idx handle count 3 isn't equal to value count 2

ADMIN CHECK INDEX tbl idx ;
ERROR 1105 (HY000): handle &kv.CommonHandle{encoded:[]uint8{0x1, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0xf8}, colEndOffsets:[]uint16{0xa}}, index:types.Datum{k:0x5, decimal:0x0, length:0x0, i:0, collation:"utf8mb4_bin", b:[]uint8{0x0}, x:interface {}(nil)} != record:<nil>
```

`SELECT` クエリのエラーメッセージから、`tbl` テーブルには2行のデータと3行のインデックスデータが含まれており、これは不整合な行とインデックスデータを意味します。同時に、少なくとも1つのインデックスがダングリング状態にあることが分かります。この場合、`ADMIN CLEANUP INDEX`ステートメントを使用してダングリングなインデックスを削除できます：

```sql
ADMIN CLEANUP INDEX tbl idx;
```

実行結果は次のとおりです：

```sql
ADMIN CLEANUP INDEX tbl idx;
+---------------+
| REMOVED_COUNT |
+---------------+
|             1 |
+---------------+
```

データとインデックスの整合性を再度確認し、データが正常な状態に戻ったかどうかを確認するために、`ADMIN CHECK INDEX`ステートメントを再度実行できます：

```sql
ADMIN CHECK INDEX tbl idx;
Query OK, 0 rows affected (0.01 sec)
```

<CustomContent platform="tidb">

> **注意:**
>
> レプリカの損失によりデータとインデックスが不整合になった場合:
>
> - 行データとインデックスデータの両方が喪失している可能性があります。整合性を復元するには、`ADMIN CLEANUP INDEX`ステートメントと[`ADMIN RECOVER INDEX`](/sql-statements/sql-statement-admin-recover.md)ステートメントを一緒に使用してください。
> - `ADMIN CLEANUP INDEX`ステートメントは常に単一スレッドで実行されます。テーブルデータが大きい場合、インデックスデータを再構築することを推奨します。
> - `ADMIN CLEANUP INDEX`ステートメントを実行すると、対応するテーブルまたはインデックスはロックされず、TiDBは同時に他のセッションがテーブルレコードを変更することを許可します。ただし、この場合、`ADMIN CLEANUP INDEX`はすべてのテーブルレコードを正しく処理できない可能性があります。そのため、`ADMIN CLEANUP INDEX`を実行する際は、同時にテーブルデータを変更しないようにしてください。
> - TiDBのエンタープライズエディションを使用している場合、[リクエストを提出](/support.md)してサポートエンジニアに連絡することができます。
>
> `ADMIN CLEANUP INDEX`ステートメントはアトミックではありません: ステートメントの実行中に中断された場合、成功するまで再度実行することが推奨されています。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> レプリカの損失によりデータとインデックスが不整合になった場合:
>
> - 行データとインデックスデータの両方が喪失している可能性があります。整合性を復元するには、`ADMIN CLEANUP INDEX`ステートメントと[`ADMIN RECOVER INDEX`](/sql-statements/sql-statement-admin-recover.md)ステートメントを一緒に使用してください。
> - `ADMIN CLEANUP INDEX`ステートメントは常に単一スレッドで実行されます。テーブルデータが大きい場合、インデックスデータを再構築することを推奨します。
> - `ADMIN CLEANUP INDEX`ステートメントを実行すると、対応するテーブルまたはインデックスはロックされず、TiDBは同時に他のセッションがテーブルレコードを変更することを許可します。ただし、この場合、`ADMIN CLEANUP INDEX`はすべてのテーブルレコードを正しく処理できない可能性があります。そのため、`ADMIN CLEANUP INDEX`を実行する際は、同時にテーブルデータを変更しないようにしてください。
> - TiDBのエンタープライズエディションを使用している場合、[リクエストを提出](https://support.pingcap.com/hc/en-us)してサポートエンジニアに連絡することができます。
>
> `ADMIN CLEANUP INDEX`ステートメントはアトミックではありません: ステートメントの実行中に中断された場合、成功するまで再度実行することが推奨されています。

</CustomContent>

## MySQL 互換性

このステートメントは、MySQLの構文に対するTiDBの拡張です。

## 関連情報

* [`ADMIN CHECK TABLE/INDEX`](/sql-statements/sql-statement-admin-check-table-index.md)
* [`ADMIN RECOVER INDEX`](/sql-statements/sql-statement-admin-recover.md)