---
title: SHARD_ROW_ID_BITS
summary: SHARD_ROW_ID_BITS属性について学ぶ。

# SHARD_ROW_ID_BITS

このドキュメントは、テーブル属性`SHARD_ROW_ID_BITS`を紹介し、バラバラになった`_tidb_rowid`の後のシャードのビット数を設定するために使用されます。

## コンセプト

非クラスタ化されたプライマリキーまたはプライマリキーが存在しないテーブルの場合、TiDBは`暗黙的な自動増分の行ID`を使用します。大量の`INSERT`操作が実行されると、データは単一のリージョンに書き込まれ、書き込みホットスポットが発生します。

ホットスポットの問題を緩和するために、`SHARD_ROW_ID_BITS`を構成できます。行IDは散らばり、データは複数の異なるリージョンに書き込まれます。

- `SHARD_ROW_ID_BITS = 4` は16のシャードを示します
- `SHARD_ROW_ID_BITS = 6` は64のシャードを示します
- `SHARD_ROW_ID_BITS = 0` はデフォルトの1つのシャードを示します

<CustomContent platform="tidb">

使用方法の詳細については、[Hotspot Issuesのトラブルシューティングガイド](/troubleshoot-hot-spot-issues.md#use-shard_row_id_bits-to-process-hotspots) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

使用方法の詳細については、[Hotspot Issuesのトラブルシューティングガイド](https://docs.pingcap.com/tidb/stable/troubleshoot-hot-spot-issues#use-shard_row_id_bits-to-process-hotspots) を参照してください。

</CustomContent>

## 例

```sql
CREATE TABLE t (
    id INT PRIMARY KEY NONCLUSTERED
) SHARD_ROW_ID_BITS = 4;
```

```sql
ALTER TABLE t SHARD_ROW_ID_BITS = 4;
```