---
title: AUTO_INCREMENT
summary: TiDBの`AUTO_INCREMENT`カラム属性の学習。
aliases: ['/docs/dev/auto-increment/']
---

# AUTO_INCREMENT

このドキュメントでは、`AUTO_INCREMENT`カラム属性について、その概念、実装原則、オートインクリメントに関連する機能、および制限について紹介します。

<TiDBのカスタムコンテンツ>

> **注記:**
>
> `AUTO_INCREMENT`属性は、本番環境でホットスポットを引き起こす可能性があります。詳細については、[トラブルシューティング ホットスポットの問題](/troubleshoot-hot-spot-issues.md) を参照してください。[`AUTO_RANDOM`](/auto-random.md)を使用することをお勧めします。

</TiDBのカスタムコンテンツ>

<TiDBクラウドのカスタムコンテンツ>

> **注記:**
>
> `AUTO_INCREMENT`属性は、本番環境でホットスポットを引き起こす可能性があります。詳細については、[トラブルシューティング ホットスポットの問題](https://docs.pingcap.com/tidb/stable/troubleshoot-hot-spot-issues#handle-auto-increment-primary-key-hotspot-tables-using-auto_random) を参照してください。[`AUTO_RANDOM`](/auto-random.md)を使用することをお勧めします。

</TiDBクラウドのカスタムコンテンツ>

また、[`CREATE TABLE`](/sql-statements/sql-statement-create-table.md) 文で`AUTO_INCREMENT`パラメータを使用して、増分フィールドの初期値を指定することもできます。

## 概念

`AUTO_INCREMENT`は、デフォルトのカラム値を自動的に補完するためのカラム属性です。`INSERT`文で`AUTO_INCREMENT`カラムの値が指定されない場合、システムはこのカラムに自動的に値を割り当てます。

パフォーマンス上の理由から、TiDBサーバーごとに`AUTO_INCREMENT`番号が値のバッチ（デフォルトでは3万）で割り当てられます。つまり、`AUTO_INCREMENT`番号は一意であることが保証されていますが、`INSERT`文に割り当てられる値は各TiDBサーバー単位でのみ単調になります。

> **注記:**
>
> もし、`AUTO_INCREMENT`番号をすべてのTiDBサーバー上で単調にしたい場合は、TiDBのバージョンがv6.5.0以降であれば、[MySQL互換モード](#mysql-compatibility-mode)を有効にすることをお勧めします。

以下は、`AUTO_INCREMENT`の基本的な例です:

{{< copyable "sql" >}}

```sql
CREATE TABLE t(id int PRIMARY KEY AUTO_INCREMENT, c int);
```

{{< copyable "sql" >}}

```sql
INSERT INTO t(c) VALUES (1);
INSERT INTO t(c) VALUES (2);
INSERT INTO t(c) VALUES (3), (4), (5);
```

```sql
mysql> SELECT * FROM t;
+----+---+
| id | c |
+----+---+
| 1  | 1 |
| 2  | 2 |
| 3  | 3 |
| 4  | 4 |
| 5  | 5 |
+----+---+
5 rows in set (0.01 sec)
```

さらに、`AUTO_INCREMENT`は明示的にカラムの値を指定する`INSERT`文もサポートしています。このような場合、TiDBは明示的に指定された値を保存します:

{{< copyable "sql" >}}

```sql
INSERT INTO t(id, c) VALUES (6, 6);
```

```sql
mysql> SELECT * FROM t;
+----+---+
| id | c |
+----+---+
| 1  | 1 |
| 2  | 2 |
| 3  | 3 |
| 4  | 4 |
| 5  | 5 |
| 6  | 6 |
+----+---+
6 rows in set (0.01 sec)
```

上記の使用法は、MySQLの`AUTO_INCREMENT`と同じです。ただし、暗黙的に割り当てられる具体的な値については、TiDBはMySQLとは大きく異なります。

## 実装原則

TiDBでは、`AUTO_INCREMENT`の暗黙的な割り当てを以下の方法で実装しています:

各自動増分カラムについて、割り当てられた最大IDを記録するために、グローバルに可視なキーと値のペアが使用されます。分散環境では、ノード間の通信にはいくつかのオーバーヘッドがあります。そのため、書き込み増幅の問題を回避するために、各TiDBノードはIDを割り当てる際に、IDのバッチをキャッシュとして申請し、最初のバッチが割り当てられた後に次のバッチのIDを申請します。したがって、TiDBノードはIDを割り当てるときに、毎回ストレージノードにIDを申請することはありません。例:

```sql
CREATE TABLE t(id int UNIQUE KEY AUTO_INCREMENT, c int);
```

クラスターには、`A`と`B`の2つのTiDBインスタンスがあると仮定します。`A`と`B`それぞれの`t`テーブルに`INSERT`文を実行する場合:

```sql
INSERT INTO t (c) VALUES (1)
```

インスタンス`A`は`[1,30000]`のオートインクリメントIDをキャッシュし、インスタンス`B`は`[30001,60000]`のオートインクリメントIDをキャッシュするかもしれません。各インスタンスのキャッシュされたIDは`INSERT`文が実行される際、`AUTO_INCREMENT`カラムにデフォルトの値として割り当てられます。

## 基本的な機能

### 一意性

> **警告:**
>
> クラスターに複数のTiDBインスタンスがある場合、テーブルスキーマにオートインクリメントIDが含まれている場合、デフォルトの値とカスタムの値を同時に使用することは避けることをお勧めします。つまり、オートインクリメントカラムのデフォルト値とカスタム値を使用することは避けることをお勧めします。そうしないと、暗黙的に割り当てられる値の一意性が壊れる可能性があります。

上記の例で、以下の操作を順番に実行します:

1. クライアントはインスタンス`B`に`INSERT INTO t VALUES (2, 1)`という文を挿入し、`id`を`2`に設定します。この文は正常に実行されます。

2. クライアントはインスタンス`A`に`INSERT INTO t (c) (1)`という文を送信します。この文では`id`の値が指定されていません。したがって、`A`がIDを割り当てます。現在、`A`は`[1, 30000]`のIDをキャッシュしているため、`2`をオートインクリメントIDの値として割り当て、ローカルカウンタを`1`増やすかもしれません。この時点で、IDが`2`であるデータはすでにデータベースに存在しているため、「重複エラー」が返されます。

### 単調性

TiDBは、`AUTO_INCREMENT`の値がサーバーごとに常に単調増加することを保証します。次の例では、`1`から`3`までの連続した`AUTO_INCREMENT`の値が生成される状況を考えてください:

{{< copyable "sql" >}}

```sql
CREATE TABLE t (a int PRIMARY KEY AUTO_INCREMENT, b timestamp NOT NULL DEFAULT NOW());
INSERT INTO t (a) VALUES (NULL), (NULL), (NULL);
SELECT * FROM t;
```

```sql
Query OK, 0 rows affected (0.11 sec)

Query OK, 3 rows affected (0.02 sec)
Records: 3  Duplicates: 0  Warnings: 0

+---+---------------------+
| a | b                   |
+---+---------------------+
| 1 | 2020-09-09 20:38:22 |
| 2 | 2020-09-09 20:38:22 |
| 3 | 2020-09-09 20:38:22 |
+---+---------------------+
3 rows in set (0.00 sec)
```

単調増加は連続と同じ保証ではありません。次の例を考えてください:

{{<copyable "sql" >}}

```sql
CREATE TABLE t (id INT NOT NULL PRIMARY KEY auto_increment, a VARCHAR(10), cnt INT NOT NULL DEFAULT 1, UNIQUE KEY (a));
INSERT INTO t (a) VALUES ('A'), ('B');
SELECT * FROM t;
INSERT INTO t (a) VALUES ('A'), ('C') ON DUPLICATE KEY UPDATE cnt = cnt + 1;
SELECT * FROM t;
```

```sql
Query OK, 0 rows affected (0.00 sec)

Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

+----+------+-----+
| id | a    | cnt |
+----+------+-----+
|  1 | A    |   1 |
|  2 | B    |   1 |
+----+------+-----+
2 rows in set (0.00 sec)

Query OK, 3 rows affected (0.00 sec)
Records: 2  Duplicates: 1  Warnings: 0

+----+------+-----+
| id | a    | cnt |
+----+------+-----+
|  1 | A    |   2 |
|  2 | B    |   1 |
|  4 | C    |   1 |
+----+------+-----+
3 rows in set (0.00 sec)
```

この例では、`INSERT INTO t (a) VALUES ('A'), ('C') ON DUPLICATE KEY UPDATE cnt = cnt + 1;`のキー`A`を挿入する場合に`3`の`AUTO_INCREMENT`値が割り当てられますが、この`INSERT`文には重複キー`A`が含まれているため、連続したシーケンスが非連続となるギャップが発生します。この動作は、MySQLとは異なるものの、合法的と見なされます。MySQLでもトランザクションが中止されてロールバックされるなど、別のシナリオでシーケンスにギャップが生じます。

## AUTO_ID_CACHE
```
                           AUTO_INCREMENT シーケンスは、別の TiDB サーバー上で `INSERT` 操作が実行されると、 `AUTO_INCREMENT` 値のキャッシュがそれぞれのサーバーごとに存在するため、_急激に_増加する場合があります。

{{< copyable "sql" >}}

```sql
CREATE TABLE t (a int PRIMARY KEY AUTO_INCREMENT, b timestamp NOT NULL DEFAULT NOW());
INSERT INTO t (a) VALUES (NULL), (NULL), (NULL);
INSERT INTO t (a) VALUES (NULL);
SELECT * FROM t;
```

```sql
Query OK, 1 row affected (0.03 sec)

+---------+---------------------+
| a       | b                   |
+---------+---------------------+
|       1 | 2020-09-09 20:38:22 |
|       2 | 2020-09-09 20:38:22 |
|       3 | 2020-09-09 20:38:22 |
| 2000001 | 2020-09-09 20:43:43 |
+---------+---------------------+
4 rows in set (0.00 sec)
```

初めの TiDB サーバーに対する新しい `INSERT` 操作では、 `AUTO_INCREMENT` 値が `4` 生成されます。これは初めの TiDB サーバーはまだ `AUTO_INCREMENT` キャッシュの割り当てに空きがあるためです。この場合、値 `4` は `2000001` の値の後に挿入されるため、値のシーケンスをグローバルに単調と見なすことはできません。

```sql
mysql> INSERT INTO t (a) VALUES (NULL);
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM t ORDER BY b;
+---------+---------------------+
| a       | b                   |
+---------+---------------------+
|       1 | 2020-09-09 20:38:22 |
|       2 | 2020-09-09 20:38:22 |
|       3 | 2020-09-09 20:38:22 |
| 2000001 | 2020-09-09 20:43:43 |
|       4 | 2020-09-09 20:44:43 |
+---------+---------------------+
5 rows in set (0.00 sec)
```

`AUTO_INCREMENT` キャッシュは TiDB サーバーの再起動後でも維持されません。初めの TiDB サーバーが再起動された後に次の `INSERT` 文が実行されます。

```sql
mysql> INSERT INTO t (a) VALUES (NULL);
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM t ORDER BY b;
+---------+---------------------+
| a       | b                   |
+---------+---------------------+
|       1 | 2020-09-09 20:38:22 |
|       2 | 2020-09-09 20:38:22 |
|       3 | 2020-09-09 20:38:22 |
| 2000001 | 2020-09-09 20:43:43 |
|       4 | 2020-09-09 20:44:43 |
| 2030001 | 2020-09-09 20:54:11 |
+---------+---------------------+
6 rows in set (0.00 sec)
```

高頻度の TiDB サーバーの再起動は `AUTO_INCREMENT` 値の枯渇に寄与する可能性があります。上記の例では、初めの TiDB サーバーには、キャッシュ内で `[5-30000]` の値が空きがあります。これらの値は失われ再割り当てされません。

`AUTO_INCREMENT` 値が連続していることに頼ることは推奨されません。例として、TiDB サーバーが `[2000001-2030000]` の値のキャッシュを持っている場合、値 `2029998` を手動で挿入することで、新しいキャッシュ範囲の動作を確認できます。

```sql
mysql> INSERT INTO t (a) VALUES (2029998);
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO t (a) VALUES (NULL);
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO t (a) VALUES (NULL);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO t (a) VALUES (NULL);
Query OK, 1 row affected (0.02 sec)

mysql> INSERT INTO t (a) VALUES (NULL);
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM t ORDER BY b;
+---------+---------------------+
| a       | b                   |
+---------+---------------------+
|       1 | 2020-09-09 20:38:22 |
|       2 | 2020-09-09 20:38:22 |
|       3 | 2020-09-09 20:38:22 |
| 2000001 | 2020-09-09 20:43:43 |
|       4 | 2020-09-09 20:44:43 |
| 2030001 | 2020-09-09 20:54:11 |
| 2029998 | 2020-09-09 21:08:11 |
| 2029999 | 2020-09-09 21:08:11 |
| 2030000 | 2020-09-09 21:08:11 |
| 2060001 | 2020-09-09 21:08:11 |
| 2060002 | 2020-09-09 21:08:11 |
+---------+---------------------+
11 rows in set (0.00 sec)
```

`2030000` の値が挿入された後、次の値は `2060001` となります。このシーケンスのジャンプは、別の TiDB サーバーが `[2030001-2060000]` の中間キャッシュ範囲を取得したためです。複数の TiDB サーバーが展開されると、キャッシュ要求が交互に行われるため、`AUTO_INCREMENT` シーケンスにはギャップが生じます。

### キャッシュ サイズの制御

以前の TiDB バージョンでは、オートインクリメント ID のキャッシュサイズはユーザーには透過的でした。v3.0.14、v3.1.2、および v4.0.rc-2 から、TiDB は `AUTO_ID_CACHE` テーブルオプションを導入し、ユーザーがオートインクリメント ID の割り当てのためのキャッシュサイズを設定できるようにしました。

```sql
mysql> CREATE TABLE t(a int AUTO_INCREMENT key) AUTO_ID_CACHE 100;
Query OK, 0 rows affected (0.02 sec)

mysql> INSERT INTO t values();
Query OK, 1 row affected (0.00 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t;
+---+
| a |
+---+
| 1 |
+---+
1 row in set (0.01 sec)
```

この時点で、この列のオートインクリメント キャッシュを無効にし、暗黙の挿入をやり直すと、以下の結果が得られます。

```sql
mysql> DELETE FROM t;
Query OK, 1 row affected (0.01 sec)

mysql> RENAME TABLE t to t1;
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO t1 values()
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM t;
+-----+
| a   |
+-----+
| 101 |
+-----+
1 row in set (0.00 sec)
```

再割り当てされた値は `101` です。これは、オートインクリメント ID の割り当てのためのキャッシュサイズが `100` であることを示しています。

さらに、バッチ `INSERT` ステートメント内の連続する ID の長さが `AUTO_ID_CACHE` の長さを超える場合、TiDB はステートメントが正常に挿入されるようにキャッシュサイズを適宜増やします。

### オートインクリメントのステップ サイズとオフセット

v3.0.9 および v4.0.0-rc.1 から、MySQL の動作に類似して、オートインクリメント列に対して暗黙の割り当てられる値は、`@@auto_increment_increment` および `@@auto_increment_offset` セッション変数によって制御されます。

オートインクリメント列に暗黙的に割り当てられる値 (ID) は、次の式を満たします。

`(ID - auto_increment_offset) % auto_increment_increment == 0`

## MySQL 互換モード

TiDB v6.4.0 では、中央集権型のオートインクリメント ID 割り当てサービスが導入されました。各リクエストで、TiDB のインスタンスにデータをキャッシュするのではなく、このサービスからオートインクリメント ID が割り当てられます。

現在のところ、中央集権型割り当てサービスは TiDB プロセスに存在し、DDL Owner のように動作します。1 つの TiDB インスタンスがプライマリ ノードとして ID を割り当て、他の TiDB インスタンスがセカンダリ ノードとして機能します。高可用性を確保するために、プライマリ インスタンスが故障した場合、TiDB は自動フェールオーバーを開始します。

MySQL 互換モードを使用する場合、テーブルを作成する際に `AUTO_ID_CACHE` を `1` に設定できます。

```sql
CREATE TABLE t(a int AUTO_INCREMENT key) AUTO_ID_CACHE 1;
```

> **注意:**
>
> TiDB では、`AUTO_ID_CACHE` を `1` に設定することで、TiDB はもはやキャッシュを保持しないことを意味します。ただし、実装は TiDB のバージョンによって異なります:
>```
```
- TiDB v6.4.0以前では、IDの割り当てにはTiKVトランザクションが必要で、各リクエストの`AUTO_INCREMENT`値を永続化するために`AUTO_ID_CACHE`を`1`に設定すると、パフォーマンスが低下します。
- TiDB v6.4.0以降、`AUTO_INCREMENT`値の変更は、中央集権化された割り当てサービスが導入されたことでTiDBプロセス内のメモリ内操作になり、より高速です。
- `AUTO_ID_CACHE`を`0`に設定すると、TiDBはデフォルトのキャッシュサイズ`30000`を使用します。

MySQL互換モードを有効にした後、割り当てられたIDは**ユニーク**であり、**単調増加**であり、振る舞いはほぼMySQLと同様です。TiDBインスタンス間を越えてアクセスしても、IDは単調です。中央サービスのプライマリインスタンスがクラッシュした場合にのみ、連続しないIDがいくつか発生することがあります。これは、フェイルオーバー時にセカンダリインスタンスがプライマリインスタンスによって割り当てられるはずのいくつかのIDを破棄して、IDの一意性を確保するためです。

## 制限事項

現在、TiDBで`AUTO_INCREMENT`を使用する際は、以下の制限があります:

- TiDB v6.6.0およびそれ以前のバージョンでは、定義された列は主キーまたはインデックスプレフィックスである必要があります。
- `INTEGER`、`FLOAT`、または`DOUBLE`型の列で定義する必要があります。
- 同じ列に`DEFAULT`列値と`AUTO_INCREMENT`属性を指定することはできません。
- `ALTER TABLE`では、`AUTO_INCREMENT`属性を追加または変更することができません。これには、既存の列に`AUTO_INCREMENT`属性を追加するための`ALTER TABLE ... MODIFY/CHANGE COLUMN`を使用することや、`AUTO_INCREMENT`属性を持つ列を追加するための`ALTER TABLE ... ADD COLUMN`を使用することも含まれます。
- `ALTER TABLE`を使用して`AUTO_INCREMENT`属性を削除することができます。ただし、v2.1.18およびv3.0.4以降、TiDBではセッション変数`@@tidb_allow_remove_auto_inc`を使用して、`ALTER TABLE MODIFY`または`ALTER TABLE CHANGE`を使用して列の`AUTO_INCREMENT`属性を削除できるかどうかを制御します。デフォルトでは、`ALTER TABLE MODIFY`または`ALTER TABLE CHANGE`を使用して`AUTO_INCREMENT`属性を削除することはできません。
- `ALTER TABLE`は`FORCE`オプションを必要とします。`AUTO_INCREMENT`の値をより小さな値に設定するには。
- `AUTO_INCREMENT`を`MAX(<auto_increment_column>)`よりも小さい値に設定すると、事前に存在する値がスキップされないため、重複キーが発生します。
```