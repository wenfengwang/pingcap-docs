---
title: 楽観モードでシャードされたテーブルからデータをマージおよび移行する
summary: DMが楽観モードでシャードされたテーブルからデータをマージおよび移行する方法を学ぶ
---

# 楽観モードでシャードされたテーブルからデータをマージおよび移行する

このドキュメントでは、Data Migration（DM）が楽観モードで提供するシャーディングサポート機能について紹介します。この機能により、上流のMySQLまたはMariaDBインスタンスの異なるテーブルスキーマを持つまたは同じテーブルのデータを下流のTiDBの同じテーブルにマージおよびマイグレーションできます。

> **注意:**
>
> 楽観モードとその制限について十分な理解がない場合は、このモードを使用しないことをお勧めします。そうしないと、マイグレーションの中断やデータ不整合が発生する可能性があります。

## 背景

DMは、オンラインでシャードされたテーブルに対するDDLステートメントの実行をサポートし、このことをシャーディングDDLと呼びます。デフォルトでは「悲観モード」を使用します。このモードでは、上流のシャードされたテーブルでDDLステートメントが実行されると、このテーブルのデータマイグレーションは他の全てのシャードテーブルで同じDDLステートメントが実行されるまで一時停止します。その後、このDDLステートメントが下流で実行され、データマイグレーションが再開されます。

悲観モードは、下流にマイグレーションされるデータが常に正しいことを保証しますが、データマイグレーションが一時停止するため、上流でA/B変更を行う際には不利です。一部のユーザーは、単一のシャードされたテーブルでDDLステートメントを実行し、他のシャードされたテーブルのスキーマを検証期間の後に変更する場合があります。悲観モードでは、これらのDDLステートメントはデータマイグレーションをブロックし、多くのバイナリログイベントが溜まる原因となります。

そのため、楽観モードが必要です。このモードでは、シャードされたテーブルで実行されたDDLステートメントは自動的に他のシャードされたテーブルと互換性のあるステートメントに変換され、直ちに下流に移行されます。これにより、DDLステートメントはいかなるシャードテーブルもDMLマイグレーションを実行することをブロックしません。

## 楽観モードの構成

楽観モードを使用するには、タスク構成ファイル内で`shard-mode`項目を`optimistic`と指定します。また、`strict-optimistic-shard-mode`構成を有効にすることで、楽観モードの動作を制限できます。詳細なサンプル構成ファイルについては、[DM Advanced Task Configuration File](/dm/task-configuration-file-full.md)を参照してください。

## 制限事項

楽観モードを使用する際にいくつかのリスクが伴います。使用する際には、以下のルールに従ってください。

- 一連のDDLステートメントを実行する前後で、各シャードされたテーブルのスキーマが互いに一致していることを確認します。
- A/Bテストを行う場合は、1つのシャードテーブルのみでテストを実行してください。
- A/Bテストが完了した後は、最も直接的なDDLステートメントのみを最終スキーマにマイグレーションします。テストの正誤の一連のステップを再実行しないでください。

    例えば、シャードされたテーブルで `ADD COLUMN A INT; DROP COLUMN A; ADD COLUMN A FLOAT;` を実行した場合、他のシャードされたテーブルで `ADD COLUMN A FLOAT` のみを実行すればよく、3つのDDLステートメントを再実行する必要はありません。

- DDLステートメントの実行時に、DMマイグレーションの状態を観察します。エラーが報告された場合は、その一連のDDLステートメントがデータ不整合を引き起こす可能性があるかどうかを判断する必要があります。

楽観モードでは、上流で実行される大部分のDDLステートメントは追加の手間なしで直ちに下流にマイグレーションされます。これらのDDLステートメントは「Type 1 DDL」と呼ばれます。

列名、列タイプ、または列のデフォルト値を変更するDDLステートメントは「Type 2 DDL」と呼ばれます。上流でType 2 DDLステートメントを実行する際は、全てのシャードテーブルでDDLステートメントを同じ順序で実行する必要があります。

Type 2 DDLステートメントの例:

- 列のタイプを変更: `ALTER TABLE テーブル名 MODIFY COLUMN 列名 VARCHAR(20)`
- 列の名前を変更: `ALTER TABLE テーブル名 RENAME COLUMN 列1 TO 列2`
- デフォルト値のない`NOT NULL`列の追加: `ALTER TABLE テーブル名 ADD COLUMN 列1 NOT NULL`
- インデックスの名前を変更: `ALTER TABLE テーブル名 RENAME INDEX インデックス1 TO インデックス2`

上記のDDLステートメントがシャードテーブルで実行される際に、`strict-optimistic-shard-mode: true`が設定されていると、タスクは直ちに中断され、エラーが報告されます。`strict-optimistic-shard-mode: false`が設定されているか、指定されていない場合は、シャードされたテーブルでのDDLステートメントの異なる実行順序はマイグレーションを中断します。例:

- シャード1が列名を変更し、その後列タイプを変更する:
    1. 列名を変更: `ALTER TABLE テーブル名 RENAME COLUMN 列1 TO 列2`
    2. 列タイプを変更: `ALTER TABLE テーブル名 MODIFY COLUMN 列3 VARCHAR(20)`
- シャード2が列タイプを変更し、その後列名を変更する:
    1. 列タイプを変更: `ALTER TABLE テーブル名 MODIFY COLUMN 列3 VARCHAR(20)`
    2. 列名を変更: `ALTER TABLE テーブル名 RENAME COLUMN 列1 TO 列2`

なお、楽観モードと悲観モードの両方に以下の制限事項が適用されます:

- `DROP TABLE`または`DROP DATABASE`はサポートされていません。
- `TRUNCATE TABLE`はサポートされていません。
- 各DDLステートメントは1つのテーブルの操作のみを含んでいなければなりません。
- TiDBでサポートされていないDDLステートメントは、DMでもサポートされません。
- 新しく追加される列のデフォルト値には、`current_timestamp`、`rand()`、`uuid()`を含めてはなりません。そうしないと、上流と下流でデータの不整合が発生する可能性があります。

## リスク

楽観モードを使用してマイグレーションタスクを実行する際、DDLステートメントは直ちに下流に移行されます。このモードが誤用されると、上流と下流でデータ不整合が発生する可能性があります。

### データ不整合を引き起こす操作

- 各シャードされたテーブルのスキーマが互いに互換性がない場合。例:
    - それぞれのシャードテーブルに同じ名前の2つの列が追加されているが、列の型が異なる。
    - それぞれのシャードテーブルに同じ名前の2つの列が追加されているが、列のデフォルト値が異なる。
    - それぞれのシャードテーブルで同じ名前の2つの生成列が追加されているが、式が異なる。
    - それぞれのシャードテーブルに同じ名前の2つのインデックスが追加されているが、キーが異なる。
    - 同じ名前の異なるテーブルスキーマ。
- シャードされたテーブルのデータを破壊する可能性のあるDDLステートメントを実行し、その後ロールバックを試みる。

    例: 列 `X` を削除し、その後再度この列を追加する。

### 例

以下の3つのシャードされたテーブルをTiDBにマージおよびマイグレーションします:

![optimistic-ddl-fail-example-1](/media/dm/optimistic-ddl-fail-example-1.png)

`tbl01`に新しい列 `Age` を追加し、列のデフォルト値を `0` に設定します:

```sql
ALTER TABLE `tbl01` ADD COLUMN `Age` INT DEFAULT 0;
```

![optimistic-ddl-fail-example-2](/media/dm/optimistic-ddl-fail-example-2.png)

`tbl00`に新しい列 `Age` を追加し、列のデフォルト値を `-1` に設定します:

```sql
ALTER TABLE `tbl00` ADD COLUMN `Age` INT DEFAULT -1;
```

![optimistic-ddl-fail-example-3](/media/dm/optimistic-ddl-fail-example-3.png)

この時点で、`tbl00`の `Age` 列は矛盾しています。つまり、`DEFAULT 0` と `DEFAULT -1` は互換性がありません。この状況では、DMはエラーを報告しますが、データ不整合を手動で修正する必要があります。

## 実装原理

楽観モードでは、DM-workerが上流からDDLステートメントを受信した後、更新されたテーブルスキーマをDM-masterに転送します。DM-workerは各シャードされたテーブルの現在のスキーマを追跡し、DM-masterはそれらのスキーマを複合スキーマにマージし、すべてのシャードテーブルのDMLステートメントと互換性のあるスキーマに変換します。その後、DM-masterは対応するDDLステートメントを下流にマイグレーションし、DMLステートメントは直ちに下流にマイグレーションされます。

![optimistic-ddl-flow](/media/dm/optimistic-ddl-flow.png)

### 例

上流のMySQLにシャードされた3つのテーブル（`tbl00`、`tbl01`、`tbl02`）があると仮定します。これらのシャードされたテーブルを下流のTiDBの`tbl`テーブルにマージおよびマイグレーションします。以下の画像を参照してください:

![optimistic-ddl-example-1](/media/dm/optimistic-ddl-example-1.png)

上流に`Level`列を追加します:

```sql
ALTER TABLE `tbl00` ADD COLUMN `Level` INT;
```

![optimistic-ddl-example-2](/media/dm/optimistic-ddl-example-2.png)

その後、TiDBは`tbl00`からDMLステートメント（`Level`列付き）と`tbl01`および`tbl02`テーブルからのDMLステートメント（`Level`列なし）を受け取ります。

![optimistic-ddl-example-3](/media/dm/optimistic-ddl-example-3.png)

以下のDMLステートメントは修正なしで下流にマイグレーションされます:

```sql
```
```sql
UPDATE `tbl00` SET `Level` = 9 WHERE `ID` = 1;
INSERT INTO `tbl02` (`ID`, `Name`) VALUES (27, 'Tony');
```

![optimistic-ddl-example-4](/media/dm/optimistic-ddl-example-4.png)

`tbl01`に`Level`列を追加します:

```sql
ALTER TABLE `tbl01` ADD COLUMN `Level` INT;
```

![optimistic-ddl-example-5](/media/dm/optimistic-ddl-example-5.png)

この時点で、ダウンストリームは既に同じ`Level`列を持っているため、DM-masterはテーブルスキーマを比較してから操作を行いません。

`tbl01`から`Name`列を削除します:

```sql
ALTER TABLE `tbl01` DROP COLUMN `Name`;
```

![optimistic-ddl-example-6](/media/dm/optimistic-ddl-example-6.png)

その後、ダウンストリームは`tbl00`と`tbl02`から`Name`列を持つDMLステートメントを受け取るため、この列は直ちに削除されません。

同様に、全てのDMLステートメントは引き続きダウンストリームに移行できます:

```sql
INSERT INTO `tbl01` (`ID`, `Level`) VALUES (15, 7);
UPDATE `tbl00` SET `Level` = 5 WHERE `ID` = 5;
```

![optimistic-ddl-example-7](/media/dm/optimistic-ddl-example-7.png)

`tbl02`に`Level`列を追加します:

```sql
ALTER TABLE `tbl02` ADD COLUMN `Level` INT;
```

![optimistic-ddl-example-8](/media/dm/optimistic-ddl-example-8.png)

その時点で、全てのシャードテーブルに`Level`列が存在しています。

`tbl00`と`tbl02`からそれぞれ`Name`列を削除します:

```sql
ALTER TABLE `tbl00` DROP COLUMN `Name`;
ALTER TABLE `tbl02` DROP COLUMN `Name`;
```

![optimistic-ddl-example-9](/media/dm/optimistic-ddl-example-9.png)

その時点で、全てのシャードテーブルから`Name`列が削除され、ダウンストリームで安全に削除できます:

```sql
ALTER TABLE `tbl` DROP COLUMN `Name`;
```

![optimistic-ddl-example-10](/media/dm/optimistic-ddl-example-10.png)