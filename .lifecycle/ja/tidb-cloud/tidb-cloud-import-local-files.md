---
title: TiDB Cloudへのローカルファイルのインポート
summary: ローカルファイルをTiDB Cloudにインポートする方法について学びます。

# TiDB Cloudへのローカルファイルのインポート

TiDB Cloudには、ローカルファイルを直接インポートできます。タスクの構成を完了するために数回のクリックのみで、ローカルのCSVデータをTiDBクラスタに迅速にインポートできます。この方法を使用すると、クラウドストレージバケットパスやロールARNを提供する必要はありません。インポートプロセス全体が迅速でスムーズです。

現在、この方法は、1つのタスクにつき1つのCSVファイルを既存のテーブルまたは新しいテーブルにインポートすることをサポートしています。

## 制限事項

- 現在、TiDB Cloudは1つのタスクあたり50 MiB以内のCSV形式のローカルファイルをサポートしています。
- ローカルファイルのインポートはTiDBサーバーレスクラスタにのみ対応しており、TiDB専用クラスタには対応していません。
- 同時に複数のインポートタスクを実行することはできません。
- TiDB Cloudで既存のテーブルにCSVファイルをインポートし、対象のテーブルにソースファイルよりも多くの列がある場合、追加の列は状況に応じて異なる方法で処理されます：
    - 追加の列が主キーまたは一意のキーでない場合、エラーは報告されません。代わりに、これらの追加の列はそのデフォルト値で埋められます。
    - 追加の列が主キーまたは一意のキーであり、`auto_increment`または`auto_random`属性を持っていない場合、エラーが報告されます。この場合、次の戦略のいずれかを選択することをお勧めします：
        - 主キーまたは一意のキーカラムを含むソースファイルを提供します。
        - 主キーまたは一意のキーカラムの属性を`auto_increment`または`auto_random`に設定します。
- カラム名がTiDBで予約されている[キーワード](/keywords.md)の場合、TiDB Cloudは自動的に列名をバッククォート `` ` `` で囲みます。たとえば、列名が `order`の場合、TiDB Cloudは自動的にバッククォート `` ` `` を追加して `` `order` `` に変更し、データを対象のテーブルにインポートします。

## ローカルファイルのインポート

1. ターゲットクラスタの**インポート**ページを開きます。

    1. [TiDB Cloudコンソール](https://tidbcloud.com/)にログインし、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動します。

        > **ヒント：**
        >
        > 複数のプロジェクトを持っている場合、左下隅の <MDSvgIcon name="icon-left-projects" /> をクリックして別のプロジェクトに切り替えることができます。

    2. ターゲットクラスタの名前をクリックして概要ページに移動し、左側のナビゲーションペインで **インポート**をクリックします。

2. **インポート**ページで、ローカルファイルを直接ドラッグアンドドロップしてアップロードエリアにドロップするか、アップロードエリアをクリックして対象のローカルファイルを選択してアップロードします。1つのタスクにつき50 MiB未満のCSVファイルを1つだけアップロードできることに注意してください。

3. **ターゲット**エリアで、ターゲットデータベースとターゲットテーブルを選択するか、直接名前を入力して新しいデータベースまたは新しいテーブルを作成します。名前は文字（a-zおよびA-Z）または数字（0-9）で始まり、文字（a-zおよびA-Z）、数字（0-9）、およびアンダースコア（_）文字を含めることができます。**プレビュー**をクリックします。

4. テーブルを確認します。

    構成可能なテーブルの列のリストが表示されます。各行には、TiDB Cloudによって推測されたテーブルの列名、推測されたテーブルの列タイプ、およびCSVファイルからのプレビューデータが表示されます。

    - TiDB Cloudで既存のテーブルにデータをインポートする場合、列のリストはテーブル定義から抽出され、プレビューされたデータは列名によって対応づけられます。

    - 新しいテーブルを作成したい場合、列のリストはCSVファイルから抽出され、列タイプはTiDB Cloudによって推測されます。たとえば、プレビューされたデータがすべて整数である場合、推測された列タイプは**int**（整数）になります。

5. 列名とデータ型を構成します。

    CSVファイルの最初の行が列名を記録している場合は、**最初の行を列名として使用**がデフォルトで選択されていることを確認してください。

    CSVファイルに列名の行がない場合は、**最初の行を列名として使用**を選択しないでください。この場合：

    - 対象のテーブルが既に存在する場合、CSVファイルの列は対象のテーブルに順序通りにインポートされます。追加の列は切り捨てられ、不足している列はデフォルト値で埋められます。

    - TiDB Cloudに対象のテーブルを作成する必要がある場合は、各列の名前を入力します。列名は以下の要件を満たす必要があります：

        * 名前は文字（a-zおよびA-Z）、数字（0-9）、文字（中国語および日本語など）、アンダースコア（`_`）文字のみで構成されている必要があります。
        * 他の特殊文字はサポートされていません。
        * 名前の長さは65文字未満である必要があります。

        必要に応じてデータ型を変更することもできます。

6. 新しい対象のテーブルの場合、主キーを設定できます。列を主キーとして選択するか、複合主キーを作成するために複数の列を選択できます。複合主キーは、列名を選択した順に形成されます。

    > **注意：**
    >
    > - テーブルの主キーはクラスタ化されたインデックスであり、作成後に削除することはできません。
    > - 主キー項目に対応するデータが一意でありかつ空でないことを確認してください。それ以外の場合、インポートタスクはデータの不整合を引き起こします。

7. 必要に応じてCSVの構成を編集します。

   **CSV構成を編集**をクリックして、バックスラッシュエスケープ、セパレータ、デリミタを設定して、より細かな制御を行うこともできます。CSV構成についての詳細は、[データのインポートに対するCSV構成](/tidb-cloud/csv-config-for-import-data.md)を参照してください。

8. **プレビュー**ページで、データのプレビューを確認できます。**インポート開始**をクリックします。

    **インポートタスクの詳細**ページでインポートの進行状況を表示できます。警告や失敗したタスクがある場合、詳細を確認して解決できます。

9. インポートタスクが完了したら、**AI-Powered Chat2Query**を使用してインポートされたデータをクエリするために**データを探索する**をクリックできます。Chat2Queryの使用方法についての詳細は、[AI-Powered Chat2Queryでデータを探索](/tidb-cloud/explore-data-with-chat2query.md)を参照してください。

10. **インポート**ページで、**アクション**列の **表示**をクリックして、インポートタスクの詳細を確認できます。

## よくある質問

### TiDB Cloudのインポート機能を使用して指定された列のみをインポートできますか？

いいえ。現在、インポート機能を使用してCSVファイルのすべての列を既存のテーブルにのみインポートできます。

特定の列のみをインポートするには、MySQLクライアントを使用してTiDBクラスタに接続し、[`LOAD DATA`](https://docs.pingcap.com/tidb/stable/sql-statement-load-data)を使用してインポートする列を指定します。たとえば：

```sql
CREATE TABLE `import_test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(64) NOT NULL,
  `address` varchar(64) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;
LOAD DATA LOCAL INFILE 'load.txt' INTO TABLE import_test FIELDS TERMINATED BY ',' (name, address);
```

`mysql`を使用して`ERROR 2068 (HY000): LOAD DATA LOCAL INFILE file request rejected due to restrictions on access.`と遭遇した場合は、接続文字列に `--local-infile=true` を追加できます。

### TiDB Cloudにデータをインポートした後、予約されたキーワードを持つ列をクエリできないのはなぜですか？

TiDBの列名が予約された[キーワード](/keywords.md)である場合、TiDB Cloudは列名を自動的にバッククォート `` ` `` で囲み、それを対象のテーブルにインポートします。列をクエリする際は、列名を囲むためにバッククォート `` ` `` を追加する必要があります。たとえば、列名が `order` の場合は、`` `order` `` で列をクエリする必要があります。