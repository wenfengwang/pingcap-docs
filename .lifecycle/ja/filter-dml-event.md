---
title: SQL 式を使用したDMLイベントのフィルタリング
summary: SQL 式を使用してDMLイベントをフィルタリングする方法について学びます。

# SQL 式を使用したDMLイベントのフィルタリング

このドキュメントでは、DMを使用して連続する増分データレプリケーションを行う際に、SQL 式を使用してbinlogイベントをフィルタリングする方法について紹介します。詳細なレプリケーション手順については、次のドキュメントを参照してください：

- [MySQLからTiDBへの小規模データセットの移行](/migrate-small-mysql-to-tidb.md)
- [MySQLからTiDBへの大規模データセットの移行](/migrate-large-mysql-to-tidb.md)
- [MySQLからTiDBへの小規模データセットのシャードの移行とマージ](/migrate-small-mysql-shards-to-tidb.md)
- [MySQLからTiDBへの大規模データセットのシャードの移行とマージ](/migrate-large-mysql-shards-to-tidb.md)

増分データレプリケーションを行う際に、[Binlog イベントフィルタ](/filter-binlog-event.md)を使用して特定のタイプのbinlogイベントをフィルタリングできます。たとえば、アーカイブや監査などの目的で`DELETE`イベントを下流にレプリケートしないように選択することができます。ただし、Binlog イベントフィルタは、より細かい粒度が必要な行の`DELETE`イベントをフィルタリングすることはできません。

この問題に対処するために、v2.0.5以降、DMは増分データレプリケーションで`binlog value filter`を使用してデータをフィルタリングすることをサポートしています。DMがサポートする`ROW`形式のbinlogの中で、binlogイベントはすべての列の値を持ち、これらの値に基づいてSQL 式を構成できます。式が行の変更を`TRUE`と評価した場合、DMはこの行変更を下流にレプリケートしません。

[Binlog イベントフィルタ](/filter-binlog-event.md)と同様に、タスク構成ファイルで`binlog value filter`を構成する必要があります。詳細については、次の構成例を参照してください。高度なタスク構成および説明については、[DM高度なタスク構成ファイル](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)を参照してください。

```yaml
name: test
task-mode: all

mysql-instances:
  - source-id: "mysql-replica-01"
    expression-filters: ["even_c"]

expression-filter:
  even_c:
    schema: "expr_filter"
    table: "tbl"
    insert-value-expr: "c % 2 = 0"
```

上記の構成例では、`even_c`規則が構成され、これをデータソース`mysql-replica-01`が参照します。この規則によれば、`expr_filter`スキーマの`tbl`テーブルにおいて、偶数が`c`列に挿入される場合（`c % 2 = 0`）、この`insert`ステートメントは下流にレプリケートされません。次の例は、この規則の効果を示しています。

上流データソースに次のデータを増分的に挿入します：

```sql
INSERT INTO tbl(id, c) VALUES (1, 1), (2, 2), (3, 3), (4, 4);
```

次に、下流の`tb1`テーブルをクエリします。`c`に奇数が含まれる行のみがレプリケートされていることがわかります。

```sql
MySQL [test]> select * from tbl;
+------+------+
| id   | c    |
+------+------+
|    1 |    1 |
|    3 |    3 |
+------+------+
2 rows in set (0.001 sec)
```

## 構成パラメータと説明

- `schema`: 一致する上流スキーマの名前。ワイルドカード一致や正規表現一致はサポートされていません。
- `table`: 一致する上流テーブルの名前。ワイルドカード一致や正規表現一致はサポートされていません。
- `insert-value-expr`: `INSERT`タイプのbinlogイベント（WRITE_ROWS_EVENT）に適用される式を構成します。同じ構成項目内で`update-old-value-expr`、`update-new-value-expr`、`delete-value-expr`と同時に使用することはできません。
- `update-old-value-expr`: `UPDATE`タイプのbinlogイベント（UPDATE_ROWS_EVENT）に適用される古い値に適用される式を構成します。同じ構成項目内で`insert-value-expr`または`delete-value-expr`と同時に使用することはできません。
- `update-new-value-expr`: `UPDATE`タイプのbinlogイベント（UPDATE_ROWS_EVENT）に適用される新しい値に適用される式を構成します。同じ構成項目内で`insert-value-expr`または`delete-value-expr`と同時に使用することはできません。
- `delete-value-expr`: `DELETE`タイプのbinlogイベント（DELETE_ROWS_EVENT）に適用される式を構成します。同じ構成項目内で`insert-value-expr`、`update-old-value-expr`、`update-new-value-expr`と同時に使用することはできません。

> **注意:**
>
> - `update-old-value-expr`と`update-new-value-expr`を一緒に構成することができます。
> - `update-old-value-expr`と`update-new-value-expr`を一緒に構成すると、「更新 + 古い値」が`update-old-value-expr`を満たし、かつ「更新 + 新しい値」が`update-new-value-expr`を満たす行がフィルタリングされます。
> - `update-old-value-expr`または`update-new-value-expr`のいずれかを構成すると、構成された式が**全体の行の変更**をフィルタリングするかどうかが決まります。つまり、古い値の削除と新しい値の挿入がまとめてフィルタリングされます。

1つの列または複数の列に対してSQL式を使用できます。TiDBがサポートするSQL関数（ `c % 2 = 0`、`a*a + b*b = c*c`、`ts > NOW()`など）も使用できます。

`TIMESTAMP`のデフォルトのタイムゾーンは、タスク構成ファイルで指定されたタイムゾーンです。デフォルト値は下流のタイムゾーンです。`c_timestamp = '2021-01-01 12:34:56.5678+08:00'`のように、明示的にタイムゾーンを指定することもできます。

`expression-filter`構成項目の下に複数のフィルタリング規則を構成できます。上流データソースは、`expression-filters`で必要な規則を参照して効果を発揮します。複数のルールを使用する場合、**いずれか**のルールが一致すると、全体の行の変更がフィルタリングされます。

> **注意:**
>
> 多くの式フィルタリングルールを構成すると、DMの計算オーバーヘッドが増加し、データレプリケーションが遅くなります。