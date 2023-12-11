---
title: ターゲットデータベースのTiDB Lightning要件
summary: TiDB Lightningの実行前提条件を学ぶ
---

# ターゲットデータベースのTiDB Lightning要件

TiDB Lightningを使用する前に、環境が要件を満たしているかどうかを確認する必要があります。これにより、インポート中のエラーが減少し、インポートの成功が保証されます。

## ターゲットデータベースの権限

インポートモードと有効になっている機能に基づいて、ターゲットデータベースのユーザーには異なる権限が付与される必要があります。次の表は参考情報を提供します。

<table>
   <tr>
      <td></td>
      <td>機能</td>
      <td>範囲</td>
      <td>必要な権限</td>
      <td>備考</td>
   </tr>
   <tr>
      <td rowspan="2">必須</td>
      <td rowspan="2">基本機能</td>
      <td>ターゲットテーブル</td>
      <td>CREATE, SELECT, INSERT, UPDATE, DELETE, DROP, ALTER</td>
      <td>tidb-lightning-ctlがcheckpoint-destroy-allコマンドを実行する場合、DROPが必要です</td>
   </tr>
   <tr>
      <td>ターゲットデータベース</td>
      <td>CREATE</td>
      <td></td>
   </tr>
   <tr>
      <td rowspan="4">必須</td>
      <td>論理インポートモード</td>
      <td>information_schema.columns</td>
      <td>SELECT</td>
      <td></td>
   </tr>
   <tr>
      <td rowspan="3">物理インポートモード</td>
      <td>mysql.tidb</td>
      <td>SELECT</td>
      <td></td>
   </tr>
   <tr>
      <td>-</td>
      <td>SUPER</td>
      <td></td>
   </tr>
   <tr>
      <td>-</td>
      <td>RESTRICTED_VARIABLES_ADMIN,RESTRICTED_TABLES_ADMIN</td>
      <td>ターゲットのTiDBがSEMを有効にした場合に必要です</td>
   </tr>
   <tr>
      <td>推奨</td>
      <td>競合検出、max-error</td>
      <td>lightning.task-info-schema-nameに構成されたスキーマ</td>
      <td>SELECT, INSERT, UPDATE, DELETE, CREATE, DROP</td>
      <td>必要ない場合、値は "" に設定する必要があります</td>
   </tr>
   <tr>
      <td>任意</td>
      <td>並列インポート</td>
      <td>lightning.meta-schema-nameに構成されたスキーマ</td>
      <td>SELECT, INSERT, UPDATE, DELETE, CREATE, DROP</td>
      <td>必要ない場合、値は "" に設定する必要があります</td>
   </tr>
   <tr>
      <td>任意</td>
      <td>checkpoint.driver = "mysql"</td>
      <td>checkpoint.schema設定</td>
      <td>SELECT,INSERT,UPDATE,DELETE,CREATE,DROP</td>
      <td>チェックポイント情報がファイルではなくデータベースに保存されている場合に必要です</td>
   </tr>
</table>

## ターゲットデータベースのストレージスペース

ターゲットのTiKVクラスターは、インポートされたデータを格納するための十分なディスク容量を持っている必要があります。[標準のハードウェア要件](/hardware-and-software-requirements.md)に加えて、ターゲットのTiKVクラスターのストレージスペースは、**データソースのサイズ x レプリカ数 x 2** よりも大きくなければなりません。たとえば、クラスターがデフォルトで3つのレプリカを使用している場合、ターゲットのTiKVクラスターは、データソースのサイズの6倍よりも大きなストレージスペースを持っている必要があります。この式には x 2 があります。なぜならば：

- インデックスは余分なスペースを取る可能性があります。
- RocksDBにはスペース増幅効果があります。

MySQLからDumplingによって正確なデータ容量を計算するのは難しいです。ただし、次のSQLステートメントを使用して、information_schema.tablesテーブルの`DATA_LENGTH`フィールドを集計することで、データ容量を見積もることができます：

```sql
-- すべてのスキーマのサイズを計算する
SELECT
  TABLE_SCHEMA,
  FORMAT_BYTES(SUM(DATA_LENGTH)) AS 'データサイズ',
  FORMAT_BYTES(SUM(INDEX_LENGTH)) AS 'インデックスサイズ'
FROM
  information_schema.tables
GROUP BY
  TABLE_SCHEMA;

-- 最も大きな5つのテーブルを計算する
SELECT 
  TABLE_NAME,
  TABLE_SCHEMA,
  FORMAT_BYTES(SUM(data_length)) AS 'データサイズ',
  FORMAT_BYTES(SUM(index_length)) AS 'インデックスサイズ',
  FORMAT_BYTES(SUM(data_length+index_length)) AS '合計サイズ'
FROM
  information_schema.tables
GROUP BY
  TABLE_NAME,
  TABLE_SCHEMA
ORDER BY
  SUM(DATA_LENGTH+INDEX_LENGTH) DESC
LIMIT
  5;
```