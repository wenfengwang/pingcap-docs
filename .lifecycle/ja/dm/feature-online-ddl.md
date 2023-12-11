---
title: GH-ost/PT-oscを使用するデータベースからの移行
summary: このドキュメントはDMの`online-ddl/online-ddl-scheme`機能を紹介します。
aliases: ['/docs/tidb-data-migration/dev/online-ddl-scheme/','tidb-data-migration/dev/feature-online-ddl-scheme']
---

# GH-ost/PT-oscを使用するデータベースからの移行

本番環境では、DDLの実行中にテーブルロックが発生し、データベースへの読み取りまたは書き込みがある程度ブロックされる場合があります。そのため、オンラインDDLツールを使用して、読み取りや書き込みへの影響を最小限に抑えながらDDLを実行することがよくあります。一般的なDDLツールには、[gh-ost](https://github.com/github/gh-ost)や[pt-osc](https://www.percona.com/doc/percona-toolkit/3.0/pt-online-schema-change.html)があります。

MySQLからTiDBへデータを移行する際には、DMを使用してオンラインDDLを有効にし、DMとgh-ostまたはpt-oscの連携を可能にすることができます。オンラインDDLを有効にする方法や、このオプションを有効にした後のワークフローの詳細については、[gh-ostまたはpt-oscを使用した継続的なレプリケーション](/migrate-with-pt-ghost.md)を参照してください。このドキュメントでは、DMとオンラインDDLツールの連携の詳細に焦点を当てています。

## DMとオンラインDDLツールの動作詳細

このセクションでは、オンラインDDLツール[gh-ost](https://github.com/github/gh-ost)および[pt-osc](https://www.percona.com/doc/percona-toolkit/3.0/pt-online-schema-change.html)を使用したDMの動作詳細について説明します。

## オンラインスキーマ変更：gh-ost

gh-ostがオンラインスキーマ変更を実装する際、3種類のテーブルが作成されます。

- gho: DDLを適用するためのテーブル。データが完全にレプリケートされ、ghoテーブルが元のテーブルと一貫性があるときに、元のテーブルは名前変更によって置き換えられます。
- ghc: オンラインスキーマ変更に関連する情報を格納するためのテーブル。
- del: 元のテーブルを名前変更して作成されたテーブル。

移行の過程では、DMは上記のテーブルを3つのカテゴリに分けます。

- ghostTable: `_*_gho`
- trashTable: `_*_ghc`, `_*_del`
- realTable: オンラインDDLを実行する元のテーブル。

gh-ostで主に使用されるSQLステートメントと、DMの対応操作は次のとおりです。

1. `_ghc`テーブルの作成:

    ```sql
    Create /* gh-ost */ table `test`.`_test4_ghc` (
                            id bigint auto_increment,
                            last_update timestamp not null DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                            hint varchar(64) charset ascii not null,
                            value varchar(4096) charset ascii not null,
                            primary key(id),
                            unique key hint_uidx(hint)
                    ) auto_increment=256 ;
    ```

    DMは`_test4_ghc`テーブルを作成しません。

2. `_gho`テーブルの作成:

    ```sql
    Create /* gh-ost */ table `test`.`_test4_gho` like `test`.`test4` ;
    ```

    DMは`_test4_gho`テーブルを作成しません。DMは、`ghost_schema`、`ghost_table`、および`dm_worker`の`server_id`に基づいて、ダウンストリームの`dm_meta.{task_name}_onlineddl`レコードを削除し、関連する情報をメモリから削除します。

    ```
    DELETE FROM dm_meta.{task_name}_onlineddl WHERE id = {server_id} and ghost_schema = {ghost_schema} and ghost_table = {ghost_table};
    ```

3. `_gho`テーブルで実行するDDLの適用:

    ```sql
    Alter /* gh-ost */ table `test`.`_test4_gho` add column cl1 varchar(20) not null ;
    ```

    DMは`_test4_gho`のDDL操作を実行しません。代わりに、このDDLを`dm_meta.{task_name}_onlineddl`およびメモリに記録します。

    ```sql
    REPLACE INTO dm_meta.{task_name}_onlineddl (id, ghost_schema , ghost_table , ddls) VALUES (......);
    ```

4. `_ghc`テーブルにデータを書き込み、元のテーブルのデータを `_gho` テーブルにレプリケートする:

    ```sql
    INSERT /* gh-ost */ INTO `test`.`_test4_ghc` VALUES (......);
    INSERT /* gh-ost `test`.`test4` */ ignore INTO `test`.`_test4_gho` (`id`, `date`, `account_id`, `conversion_price`, `ocpc_matched_conversions`, `ad_cost`, `cl2`)
      (SELECT `id`, `date`, `account_id`, `conversion_price`, `ocpc_matched_conversions`, `ad_cost`, `cl2` FROM `test`.`test4` FORCE INDEX (`PRIMARY`)
        WHERE (((`id` > _binary'1') OR ((`id` = _binary'1'))) AND ((`id` < _binary'2') OR ((`id` = _binary'2')))) lock IN share mode
      )   ;
    ```

    DMは**realtable**でないDMLステートメントを実行しません。

5. 移行が完了した後、元のテーブルと `_gho` テーブルの両方が名前変更され、オンラインDDL操作が完了します:

    ```sql
    Rename /* gh-ost */ table `test`.`test4` to `test`.`_test4_del`, `test`.`_test4_gho` to `test`.`test4`;
    ```

    DMは次の2つの操作を実行します:

    * DMは上記の`rename`操作を2つのSQLステートメントに分割します。

        ```sql
        rename test.test4 to test._test4_del;
        rename test._test4_gho to test.test4;
        ```

    * DMは`rename to _test4_del`を実行しません。`ghost_table`を元のテーブルに置き換える際、DMは次の手順を踏みます:

        - ステップ3でメモリに記録されたDDLを読み込む
        - `ghost_table`および`ghost_schema`を`origin_table`およびそれに対応するスキーマに置き換える
        - 置き換えられたDDLを実行する

        ```sql
        alter table test._test4_gho add column cl1 varchar(20) not null;
        -- 以下に置き換え:
        alter table test.test4 add column cl1 varchar(20) not null;
        ```

> **注意:**
>
> gh-ostの具体的なSQLステートメントは、実行に使用されるパラメータによって異なります。このドキュメントでは主要なSQLステートメントのみを示しています。詳細については、[gh-ostドキュメント](https://github.com/github/gh-ost#gh-ost)を参照してください。

## オンラインスキーマ変更: pt

pt-oscがオンラインスキーマ変更を実装する際、2種類のテーブルが作成されます。

- `new`: DDLを適用するためのテーブル。データが完全にレプリケートされ、`new`テーブルが元のテーブルと一貫性があるときに、元のテーブルは名前変更によって置き換えられます。
- `old`: 元のテーブルを名前変更して作成されたテーブル。
- Triggerが3種類: `pt_osc_*_ins`, `pt_osc_*_upd`, `pt_osc_*_del`。pt-oscの過程で、元のテーブルで生成された新しいデータがTriggerによって`new`にレプリケートされます。

移行の過程では、DMは上記のテーブルを3つのカテゴリに分けます。

- ghostTable: `_*_new`
- trashTable: `_*_old`
- realTable: オンラインDDLを実行する元のテーブル。

pt-oscで主に使用されるSQLステートメントと、DMの対応操作は次のとおりです:

1. `_new`テーブルの作成:

    ```sql
    CREATE TABLE `test`.`_test4_new` ( id int(11) NOT NULL AUTO_INCREMENT,
    date date DEFAULT NULL, account_id bigint(20) DEFAULT NULL, conversion_price decimal(20,3) DEFAULT NULL, ocpc_matched_conversions bigint(20) DEFAULT NULL, ad_cost decimal(20,3) DEFAULT NULL,cl2 varchar(20) COLLATE utf8mb4_bin NOT NULL,cl1 varchar(20) COLLATE utf8mb4_bin NOT NULL,PRIMARY KEY (id) ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin ;
    ```

    DMは`_test4_new`テーブルを作成しません。DMは、`ghost_schema`、`ghost_table`、および`dm_worker`の`server_id`に基づいて、ダウンストリームの`dm_meta.{task_name}_onlineddl`レコードを削除し、関連する情報をメモリから削除します。

    ```sql
    DELETE FROM dm_meta.{task_name}_onlineddl WHERE id = {server_id} and ghost_schema = {ghost_schema} and ghost_table = {ghost_table};
    ```

2. `_new` テーブルでDDLを実行:

    ```sql
    ALTER TABLE `test`.`_test4_new` add column c3 int;
    ```

    DMは`_test4_new`のDDL操作を実行しません。代わりに、このDDLを`dm_meta.{task_name}_onlineddl`およびメモリに記録します。

    ```sql
    REPLACE INTO dm_meta.{task_name}_onlineddl (id, ghost_schema , ghost_table , ddls) VALUES (......);
    ```

3. データ移行に使用される3つのトリガの作成:

    ```sql
```sql
    CREATE TRIGGER `pt_osc_test_test4_del` AFTER DELETE ON `test`.`test4` ...... ;
    CREATE TRIGGER `pt_osc_test_test4_upd` AFTER UPDATE ON `test`.`test4` ...... ;
    CREATE TRIGGER `pt_osc_test_test4_ins` AFTER INSERT ON `test`.`test4` ...... ;
    ```

    DMはTiDBでサポートされていないトリガー操作を実行しません。

4. 元のテーブルのデータを`_new` テーブルにレプリケートします:

    ```sql
    INSERT LOW_PRIORITY IGNORE INTO `test`.`_test4_new` (`id`, `date`, `account_id`, `conversion_price`, `ocpc_matched_conversions`, `ad_cost`, `cl2`, `cl1`) SELECT `id`, `date`, `account_id`, `conversion_price`, `ocpc_matched_conversions`, `ad_cost`, `cl2`, `cl1` FROM `test`.`test4` LOCK IN SHARE MODE /*pt-online-schema-change 3227 copy table*/
    ```

    DMは**実テーブル**ではないDMLステートメントを実行しません。 

5. データの移行が完了したら、元のテーブルと`_new` テーブルを名前変更し、オンラインDDL操作を完了します:

    ```sql
    RENAME TABLE `test`.`test4` TO `test`.`_test4_old`, `test`.`_test4_new` TO `test`.`test4`
    ```

    DMは以下の2つの操作を実行します:

    * DMは上記の `rename` 操作を2つのSQLステートメントに分割します:

        ```sql
        rename test.test4 to test._test4_old;
        rename test._test4_new to test.test4;
        ```

    * DMは`rename to _test4_old` を実行しません。 `rename ghost_table to origin_table` を実行する際、DMは以下の手順を実行します:

        - Step 2でメモリに記録されたDDLを読み取る
        - `ghost_table` と `ghost_schema` を `origin_table` とそれに対応するスキーマで置き換える
        - 置き換えられたDDLを実行する

        ```sql
        ALTER TABLE `test`.`_test4_new` add column c3 int;
        -- 置き換えられたもの:
        ALTER TABLE `test`.`test4` add column c3 int;
        ```

6. `_old` テーブルとオンラインDDL操作の3つのトリガーを削除します:

    ```sql
    DROP TABLE IF EXISTS `test`.`_test4_old`;
    DROP TRIGGER IF EXISTS `pt_osc_test_test4_del` AFTER DELETE ON `test`.`test4` ...... ;
    DROP TRIGGER IF EXISTS `pt_osc_test_test4_upd` AFTER UPDATE ON `test`.`test4` ...... ;
    DROP TRIGGER IF EXISTS `pt_osc_test_test4_ins` AFTER INSERT ON `test`.`test4` ...... ;
    ```

    DMは`_test4_old` とトリガーを削除しません。
```