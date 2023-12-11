---
title: AUTO_RANDOM
summary: AUTO_RANDOM属性の学習
aliases: ['/docs/dev/auto-random/', '/docs/dev/reference/sql/attributes/auto-random/']
---

# AUTO_RANDOM <span class="version-mark">v3.1.0で新規追加</span>

## ユーザシナリオ

`AUTO_RANDOM`の値はランダムかつユニークであるため、TiDBは連続したIDを割り当てることによる単一のストレージノードでのライトホットスポットを避けるために、`AUTO_INCREMENT`の代わりとしてよく用いられます。現在の`AUTO_INCREMENT`列がプライマリキーであり、その型が`BIGINT`である場合、`ALTER TABLE t MODIFY COLUMN id BIGINT AUTO_RANDOM(5);`ステートメントを実行して、`AUTO_INCREMENT`から`AUTO_RANDOM`に切り替えることができます。

<CustomContent platform="tidb">

TiDBにおける高並列の書き込み重視のワークロードの取り扱い方の詳細については、[高並列の書き込み最適化手法](/best-practices/high-concurrency-best-practices.md)を参照してください。

</CustomContent>

`CREATE TABLE`ステートメントにおける`AUTO_RANDOM_BASE`パラメーターは、`auto_random`の初期増分部分値を設定するために使用されます。このオプションは内部インタフェースの一部として考えることができます。このパラメーターは無視してもかまいません。

## 基本概念

`AUTO_RANDOM`は、`BIGINT`列に値を自動割り当てるための列属性です。自動的に割り当てられる値は**ランダム**かつ**ユニーク**です。

`AUTO_RANDOM`列を持つテーブルを作成するには、以下のステートメントを使用できます。`AUTO_RANDOM`列はプライマリキーに含まれている必要があり、`AUTO_RANDOM`列はプライマリキー内の最初の列である必要があります。

```sql
CREATE TABLE t (a BIGINT AUTO_RANDOM, b VARCHAR(255), PRIMARY KEY (a));
CREATE TABLE t (a BIGINT PRIMARY KEY AUTO_RANDOM, b VARCHAR(255));
CREATE TABLE t (a BIGINT AUTO_RANDOM(6), b VARCHAR(255), PRIMARY KEY (a));
CREATE TABLE t (a BIGINT AUTO_RANDOM(5, 54), b VARCHAR(255), PRIMARY KEY (a));
CREATE TABLE t (a BIGINT AUTO_RANDOM(5, 54), b VARCHAR(255), PRIMARY KEY (a, b));
```

キーワード`AUTO_RANDOM`を実行可能なコメントで囲むことができます。詳細については、[TiDB専用コメント構文](/comment-syntax.md#tidb-specific-comment-syntax)を参照してください。

```sql
CREATE TABLE t (a bigint /*T![auto_rand] AUTO_RANDOM */, b VARCHAR(255), PRIMARY KEY (a));
CREATE TABLE t (a bigint PRIMARY KEY /*T![auto_rand] AUTO_RANDOM */, b VARCHAR(255));
CREATE TABLE t (a BIGINT /*T![auto_rand] AUTO_RANDOM(6) */, b VARCHAR(255), PRIMARY KEY (a));
CREATE TABLE t (a BIGINT  /*T![auto_rand] AUTO_RANDOM(5, 54) */, b VARCHAR(255), PRIMARY KEY (a));
```

`INSERT`ステートメントを実行する場合:

- `AUTO_RANDOM`列の値を明示的に指定した場合、そのままテーブルに挿入されます。
- `AUTO_RANDOM`列の値を明示的に指定しない場合、TiDBはランダムな値を生成し、テーブルに挿入します。

```sql
tidb> CREATE TABLE t (a BIGINT PRIMARY KEY AUTO_RANDOM, b VARCHAR(255));
Query OK, 0 rows affected, 1 warning (0.01 sec)

tidb> INSERT INTO t(a, b) VALUES (1, 'string');
Query OK, 1 row affected (0.00 sec)

tidb> SELECT * FROM t;
+---+--------+
| a | b      |
+---+--------+
| 1 | string |
+---+--------+
1 row in set (0.01 sec)

tidb> INSERT INTO t(b) VALUES ('string2');
Query OK, 1 row affected (0.00 sec)

tidb> INSERT INTO t(b) VALUES ('string3');
Query OK, 1 row affected (0.00 sec)

tidb> SELECT * FROM t;
+---------------------+---------+
| a                   | b       |
+---------------------+---------+
|                   1 | string  |
| 1152921504606846978 | string2 |
| 4899916394579099651 | string3 |
+---------------------+---------+
3 rows in set (0.00 sec)
```

TiDBによって自動的に割り当てられた`AUTO_RANDOM(S, R)`列の値は、合計64ビットあります:

- `S`がシャードビットの数です。その値の範囲は`1`から`15`です。デフォルト値は`5`です。
- `R`が自動割り当て範囲の合計長さです。その値の範囲は`32`から`64`です。デフォルト値は`64`です。

`AUTO_RANDOM`値の構造は以下の通りです:

| 合計ビット数 | 符号ビット | 予約ビット | シャードビット | 自動増分ビット |
|---------|---------|-------------|--------|--------------|
| 64ビット | 0/1ビット | (64-R)ビット | Sビット | (R-1-S)ビット |

- 符号ビットの長さは`UNSIGNED`属性の存在によって決まります。`UNSIGNED`属性がある場合、その長さは`0`です。そうでなければ、長さは`1`です。
- 予約ビットの長さは`64-R`です。予約ビットは常に`0`です。
- シャードビットの内容は、現在のトランザクションの開始時間のハッシュ値を計算することで取得されます。シャードビットの異なる長さ（たとえば10）を使用する場合は、テーブルを作成する際に`AUTO_RANDOM(10)`を指定できます。
- 自動増分ビットの値はストレージエンジンに保存され、順次割り当てられます。新しい値が割り当てられるたびに値が1増加します。自動増分ビットは`AUTO_RANDOM`の値がグローバルにユニークであることを保証します。自動増分ビットが枯渇した場合は、「ストレージエンジンからの自動増分値の読み取りに失敗」というエラーが再度値が割り当てられる際に報告されます。

> **注記:**
>
> シャードビット（`S`）の選択:
>
> - 64ビット全体の使用可能なビット数から、シャードビットの長さが自動増分ビットの長さに影響を与えます。つまり、シャードビットの長さが増加すると、自動増分ビットの長さが減少し、その逆も然りです。そのため、割り当て値のランダム性と利用可能な空間をバランスさせる必要があります。
> - 最適なプラクティスは、シャードビットを`log(2, x)`と設定することです。ここで、`x`は現在のストレージエンジンの数です。たとえば、TiDBクラスターに16個のTiKVノードがある場合は、シャードビットを`log(2, 16)`すなわち`4`に設定できます。すべてのリージョンが均等に各TiKVノードにスケジュールされた後、大容量書き込みの負荷は異なるTiKVノードに均等に分散され、リソースの利用を最大化できます。
>
> 範囲（`R`）の選択:
>
> - 一般的に、アプリケーションの数値型が完全な64ビット整数を表現できない場合は、`R`パラメーターを設定する必要があります。
> - たとえば、JSON数値の範囲は`[-2^53+1, 2^53-1]`です。TiDBは`AUTO_RANDOM(5)`の列に対してこの範囲外の整数を容易に割り当てられ、アプリケーションが列を読み取る際に予期しない動作が発生する可能性があります。この場合、`AUTO_RANDOM(5, 54)`に置き換えることができ、TiDBは列に`9007199254740991`（2^53-1）より大きな整数を割り当てなくなります。

`AUTO_RANDOM`列に暗黙的に割り当てられた値は`last_insert_id()`に影響を与えます。TiDBが最後に暗黙的に割り当てるIDを取得するには、`SELECT last_insert_id()`ステートメントを使用できます。

`AUTO_RANDOM`列を持つテーブルのシャードビット数を表示するには、`SHOW CREATE TABLE`ステートメントを実行できます。`information_schema.tables`システムテーブルの`TIDB_ROW_ID_SHARDING_INFO`列に含まれる`PK_AUTO_RANDOM_BITS=x`モードの値を確認することもできます。ここで`x`はシャードビットの数です。

`AUTO_RANDOM`列を持つテーブルを作成した後は、`SHOW WARNINGS`を使用して最大の暗黙的割り当て回数を表示することができます:

```sql
CREATE TABLE t (a BIGINT AUTO_RANDOM, b VARCHAR(255), PRIMARY KEY (a));
SHOW WARNINGS;
```

出力は以下のようになります:

```sql
+-------+------+---------------------------------------------------------+
| Level | Code | Message                                                 |
+-------+------+---------------------------------------------------------+
| Note  | 1105 | Available implicit allocation times: 288230376151711743 |
+-------+------+---------------------------------------------------------+
1 row in set (0.00 sec)
```

## 制限

`AUTO_RANDOM`を使用する際に以下の制限に注意してください:

- 値を明示的に挿入する場合、`@@allow_auto_random_explicit_insert`システム変数の値を`1`に設定する必要があります（デフォルトでは`0`）。データを挿入する際に`AUTO_RANDOM`属性を持つ列の値を明示的に指定することは**推奨されません**。そうしないと、このテーブルに自動的に割り当てられることができる数値値が事前に使い果たされる可能性があります。
- 主キー列の属性には、`BIGINT`型のみを指定してください。それ以外の場合はエラーが発生します。また、主キーの属性が`NONCLUSTERED`の場合、整数主キーでも`AUTO_RANDOM`はサポートされません。`CLUSTERED`タイプの主キーの詳細については、[クラスタ化インデックス](/clustered-indexes.md)を参照してください。
- `AUTO_RANDOM`属性の修正は`ALTER TABLE`を使用して行うことはできません。つまり、この属性を追加したり削除したりすることはできません。
- 列の最大値が列のタイプの最大値に近い場合、`AUTO_INCREMENT`から`AUTO_RANDOM`に変更することはできません。
- `AUTO_RANDOM`属性が指定された主キー列の列型を変更することはできません。
- 同じ列に`AUTO_RANDOM`と`AUTO_INCREMENT`を同時に指定することはできません。
- 同じ列に`AUTO_RANDOM`と`DEFAULT`（列のデフォルト値）を同時に指定することはできません。
- 列に`AUTO_RANDOM`が使用されている場合、列属性を`AUTO_INCREMENT`に戻すことが困難です。なぜなら、自動生成される値が非常に大きい可能性があるためです。