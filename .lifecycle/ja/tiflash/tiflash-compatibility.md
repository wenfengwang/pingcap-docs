---
title: TiFlash 互換性メモ
summary: TiDBと互換性のないTiDB機能を学ぶ。
---

# TiFlash 互換性メモ

TiFlash は以下の状況でTiDBと互換性がありません：

* TiFlashの計算レイヤーにおいて：
    * オーバーフローした数値のチェックがサポートされていません。例えば、`BIGINT` 型の最大値 `9223372036854775807 + 9223372036854775807` を加算する場合です。TiDBでこの計算を行うと、`ERROR 1690 (22003): BIGINT value is out of range` エラーが返ることが期待されます。しかし、TiFlashでこの計算を行うと、エラーなしにオーバーフローした値 `-2` が返されます。
    * ウィンドウ関数はサポートされていません。
    * TiKV からのデータ読み込みがサポートされていません。
    * 現在、TiFlashの `sum` 関数は文字列型の引数をサポートしていません。しかし、TiDBはコンパイル中に文字列型の引数が渡されたかどうかを識別することができません。そのため、`select sum(string_col) from t` のようなステートメントを実行すると、TiFlashは `[FLASH:Coprocessor:Unimplemented] CastStringAsReal is not supported.` エラーが返ります。このような場合のエラーを回避するには、このSQLステートメントを `select sum(cast(string_col as double)) from t` に修正する必要があります。
    * 現在、TiFlashの10進数の除算計算はTiDBと互換性がありません。例えば、10進数を除算する場合、TiFlashは常にコンパイルから推測された型を使用して計算を行います。一方、TiDBはコンパイルから推測された型よりも精度の高い型を使用してこの計算を行います。そのため、10進数の除算を含む一部のSQLステートメントは、TiDB + TiKV で実行した場合とTiDB + TiFlash で実行した場合で異なる実行結果を返します。例えば：

        ```sql
        mysql> create table t (a decimal(3,0), b decimal(10, 0));
        Query OK, 0 rows affected (0.07 sec)
        mysql> insert into t values (43, 1044774912);
        Query OK, 1 row affected (0.03 sec)
        mysql> alter table t set tiflash replica 1;
        Query OK, 0 rows affected (0.07 sec)
        mysql> set session tidb_isolation_read_engines='tikv';
        Query OK, 0 rows affected (0.00 sec)
        mysql> select a/b, a/b + 0.0000000000001 from t where a/b;
        +--------+-----------------------+
        | a/b    | a/b + 0.0000000000001 |
        +--------+-----------------------+
        | 0.0000 |       0.0000000410001 |
        +--------+-----------------------+
        1 row in set (0.00 sec)
        mysql> set session tidb_isolation_read_engines='tiflash';
        Query OK, 0 rows affected (0.00 sec)
        mysql> select a/b, a/b + 0.0000000000001 from t where a/b;
        Empty set (0.01 sec)
        ```

        上記の例では、`a/b` のコンパイルから推測された型は TiDB と TiFlash の両方で `Decimal(7,4)` です。`Decimal(7,4)` に制約されるため、`a/b` の返された型は `0.0000` になるべきです。TiDBでは、`a/b` のランタイム精度は `Decimal(7,4)` よりも高いため、元のテーブルデータは `where a/b` 条件によってフィルタリングされません。しかし、TiFlashでは、`a/b` の計算は結果の型として `Decimal(7,4)` を使用するため、元のテーブルデータは `where a/b` 条件によってフィルタリングされます。