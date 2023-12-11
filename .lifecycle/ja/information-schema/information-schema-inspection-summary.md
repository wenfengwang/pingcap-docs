---
title: INSPECTION_SUMMARY
summary: `INSPECTION_SUMMARY` インスペクションサマリーテーブルを学びます。
aliases: ['/docs/dev/system-tables/system-table-inspection-summary/','/docs/dev/reference/system-databases/inspection-summary/','/tidb/dev/system-table-inspection-summary/']
---

# INSPECTION_SUMMARY

特定のリンクやモジュールの監視サマリーにのみ注意を払う必要があるシナリオでは、Coprocessorのスレッドプールのスレッド数が8に設定されている場合を考えてみましょう。CoprocessorのCPU使用率が750%に達した場合、リスクが存在し、Coprocessorが事前にボトルネックになる可能性があります。しかし、異なるユーザーワークロードにより、いくつかの監視メトリクスは大きく異なるため、特定のしきい値を定義することが困難です。このようなシナリオで問題をトラブルシューティングすることは重要ですので、TiDBはリンクサマリーのために `inspection_summary` テーブルを提供しています。

> **注意:**
>
> このテーブルは、TiDB セルフホステッドにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/) では利用できません。

`information_schema.inspection_summary` インスペクションサマリーテーブルの構造は次のようになります。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC inspection_summary;
```

```sql
+--------------+--------------+------+------+---------+-------+
| Field        | Type         | Null | Key  | Default | Extra |
+--------------+--------------+------+------+---------+-------+
| RULE         | varchar(64)  | YES  |      | NULL    |       |
| INSTANCE     | varchar(64)  | YES  |      | NULL    |       |
| METRICS_NAME | varchar(64)  | YES  |      | NULL    |       |
| LABEL        | varchar(64)  | YES  |      | NULL    |       |
| QUANTILE     | double       | YES  |      | NULL    |       |
| AVG_VALUE    | double(22,6) | YES  |      | NULL    |       |
| MIN_VALUE    | double(22,6) | YES  |      | NULL    |       |
| MAX_VALUE    | double(22,6) | YES  |      | NULL    |       |
| COMMENT      | varchar(256) | YES  |      | NULL    |       |
+--------------+--------------+------+------+---------+-------+
9 rows in set (0.00 sec)
```

フィールドの説明:

* `RULE`: サマリーのルール。新しいルールが継続的に追加されているため、最新のルールリストをクエリするには `select * from inspection_rules where type='summary'` 文を実行できます。
* `INSTANCE`: 監視されているインスタンス。
* `METRICS_NAME`: 監視メトリクス名。
* `QUANTILE`: `QUANTILE` を含む監視テーブルに影響を与えます。プレディケートをプッシュダウンして複数のパーセンタイルを指定できます。たとえば、`select * from inspection_summary where rule='ddl' and quantile in (0.80, 0.90, 0.99, 0.999)` を実行して、DDL関連の監視メトリクスをまとめ、P80/P90/P99/P999 の結果をクエリできます。 `AVG_VALUE`、`MIN_VALUE`、`MAX_VALUE` はそれぞれ集計の平均値、最小値、最大値を表します。
* `COMMENT`: 対応する監視メトリクスに関するコメント。

> **注意:**
>
> すべての結果をサマリーするとオーバーヘッドが発生するため、オーバーヘッドを減らすためにSQLプレディケートで具体的な `rule` を表示することを推奨します。たとえば、`select * from inspection_summary where rule in ('read-link', 'ddl')` を実行すると、読み込みリンクとDDL関連の監視メトリクスをまとめることができます。

使用例:

診断結果テーブルと診断監視サマリーテーブルの両方とも、`hint` を使用して診断期間を指定できます。`select /*+ time_range('2020-03-07 12:00:00','2020-03-07 13:00:00') */* from inspection_summary` は、`2020-03-07 12:00:00` から `2020-03-07 13:00:00` までの監視サマリーです。監視サマリーテーブルと同様に、異なる期間のデータを比較して大きな差異を持つ監視項目をすばやく見つけるために、 `inspection_summary` テーブルを使用できます。

以下の例は、2つの時間帯の読み込みリンクの監視メトリクスを比較します:

* `(2020-01-16 16:00:54.933, 2020-01-16 16:10:54.933)`
* `(2020-01-16 16:10:54.933, 2020-01-16 16:20:54.933)`

{{< copyable "sql" >}}

```sql
SELECT
  t1.avg_value / t2.avg_value AS ratio,
  t1.*,
  t2.*
FROM
  (
    SELECT
      /*+ time_range("2020-01-16 16:00:54.933", "2020-01-16 16:10:54.933")*/ *
    FROM information_schema.inspection_summary WHERE rule='read-link'
  ) t1
  JOIN
  (
    SELECT
      /*+ time_range("2020-01-16 16:10:54.933", "2020-01-16 16:20:54.933")*/ *
    FROM information_schema.inspection_summary WHERE rule='read-link'
  ) t2
  ON t1.metrics_name = t2.metrics_name
  and t1.instance = t2.instance
  and t1.label = t2.label
ORDER BY
  ratio DESC;
```