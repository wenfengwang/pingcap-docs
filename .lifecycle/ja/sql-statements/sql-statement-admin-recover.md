---
title: ADMIN RECOVER INDEX
summary: TiDBデータベースでのADMIN RECOVER INDEXの使用についての概要。

# ADMIN RECOVER INDEX

行データとインデックスデータが一貫性がない場合、冗長なインデックスに基づいて一貫性を回復するために`ADMIN RECOVER INDEX`ステートメントを使用できます。この構文は現在[外部キー制約](/foreign-key.md)をサポートしていません。

## 概要

```ebnf+diagram
AdminCleanupStmt ::=
    'ADMIN' 'RECOVER' 'INDEX' TableName IndexName
```

## 例

データベースの`tbl`テーブルに一貫性のない行データとインデックスがあると仮定します（たとえば、クラスタでの災害復旧シナリオにおいて一部の行データが失われた場合など）：

```sql
SELECT * FROM tbl;
ERROR 1105 (HY000): inconsistent index idx handle count 2 isn't equal to value count 3

ADMIN CHECK INDEX tbl idx ;
ERROR 1105 (HY000): handle &kv.CommonHandle{encoded:[]uint8{0x1, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0xf8}, colEndOffsets:[]uint16{0xa}}, index:types.Datum{k:0x5, decimal:0x0, length:0x0, i:0, collation:"utf8mb4_bin", b:[]uint8{0x0}, x:interface {}(nil)} != record:<nil>
```

`SELECT`クエリのエラーメッセージからわかるように、`tbl`テーブルには3つの行データと2つのインデックスデータが含まれているため、行データとインデックスデータに一貫性がないことがわかります。同時に、少なくとも1つの行データに対応するインデックスがありません。この場合、`ADMIN RECOVER INDEX`ステートメントを使用して不足しているインデックスを補完することができます：

```sql
ADMIN RECOVER INDEX tbl idx;
```

実行結果は次のとおりです：

```sql
ADMIN RECOVER INDEX tbl idx;
+-------------+------------+
| ADDED_COUNT | SCAN_COUNT |
+-------------+------------+
|           1 |          3 |
+-------------+------------+
1 row in set (0.00 sec)
```

データの一貫性を確認するために再度`ADMIN CHECK INDEX`ステートメントを実行して、データが正常な状態に戻ったかどうかを確認できます：

```sql
ADMIN CHECK INDEX tbl idx;
Query OK, 0 rows affected (0.01 sec)
```

<CustomContent platform="tidb">

> **注意:**
>
> レプリカの損失によりデータとインデックスに一貫性がない場合：
>
> - 行データとインデックスデータの両方が失われている場合があります。この問題に対処するには、[`ADMIN CLEANUP INDEX`](/sql-statements/sql-statement-admin-cleanup.md)ステートメントと`ADMIN RECOVER INDEX`ステートメントを併用して行データとインデックスデータの一貫性を回復します。
> - `ADMIN RECOVER INDEX`ステートメントは常に単一スレッドで実行されます。テーブルデータが大きい場合は、インデックスデータを再構築することをお勧めします。
> - `ADMIN RECOVER INDEX`ステートメントを実行すると、対応するテーブルまたはインデックスはロックされず、TiDBは他のセッションが同時にテーブルレコードを変更することを許可します。ただし、この場合、`ADMIN RECOVER INDEX`はすべてのテーブルレコードを正しく処理することができない場合があります。そのため、`ADMIN RECOVER INDEX`を実行する際は、同時にテーブルデータを変更しないようにしてください。
> - TiDBのエンタープライズ版を使用している場合は、[サポートエンジニアに連絡するためのリクエスト](/support.md)を送信することができます。
>
> `ADMIN RECOVER INDEX`ステートメントはアトミックではありません：ステートメントの実行中に中断された場合は、成功するまでステートメントを再度実行することをお勧めします。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> レプリカの損失によりデータとインデックスに一貫性がない場合：
>
> - 行データとインデックスデータの両方が失われている場合があります。この問題に対処するには、[`ADMIN CLEANUP INDEX`](/sql-statements/sql-statement-admin-cleanup.md)ステートメントと`ADMIN RECOVER INDEX`ステートメントを併用して行データとインデックスデータの一貫性を回復します。
> - `ADMIN RECOVER INDEX`ステートメントは常に単一スレッドで実行されます。テーブルデータが大きい場合は、インデックスデータを再構築することをお勧めします。
> - `ADMIN RECOVER INDEX`ステートメントを実行すると、対応するテーブルまたはインデックスはロックされず、TiDBは他のセッションが同時にテーブルレコードを変更することを許可します。ただし、この場合、`ADMIN RECOVER INDEX`はすべてのテーブルレコードを正しく処理することができない場合があります。そのため、`ADMIN RECOVER INDEX`を実行する際は、同時にテーブルデータを変更しないようにしてください。
> - TiDBのエンタープライズ版を使用している場合は、[サポートエンジニアに連絡するためのリクエスト](https://support.pingcap.com/hc/en-us)を送信することができます。
>
> `ADMIN RECOVER INDEX`ステートメントはアトミックではありません：ステートメントの実行中に中断された場合は、成功するまでステートメントを再度実行することをお勧めします。

</CustomContent>

## MySQL互換性

このステートメントはMySQL構文のTiDB拡張です。

## 関連情報

* [`ADMIN CHECK TABLE/INDEX`](/sql-statements/sql-statement-admin-check-table-index.md)
* [`ADMIN CLEANUP INDEX`](/sql-statements/sql-statement-admin-cleanup.md)