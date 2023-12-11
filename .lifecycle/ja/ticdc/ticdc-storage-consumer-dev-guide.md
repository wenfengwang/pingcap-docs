---
title: ストレージ・シンク・コンシューマの開発ガイド
summary: ストレージ・シンク内のデータ変更を消費するためのコンシューマの設計と実装方法を学びます。

# ストレージ・シンク・コンシューマの開発ガイド

このドキュメントでは、TiDBデータ変更コンシューマの設計と実装方法について説明します。

> **注意：**
>
> ストレージ・シンクは `DROP DATABASE` DDL を処理できません。したがって、このDDLを実行しないでください。実行が必要な場合は、ダウンストリームのMySQLで手動で実行してください。

TiCDCにはコンシューマを実装するための標準的な方法が提供されていません。このドキュメントでは、Golangで記述されたコンシューマの例プログラムが提供されています。このプログラムはストレージサービスからデータを読み取り、そのデータをMySQL互換のデータベースに書き込むことができます。この例で提供されているデータ形式と手順に従って、独自のコンシューマを実装するための参考にしてください。

[Golangで記述されたコンシューマプログラム](https://github.com/pingcap/tiflow/tree/master/cmd/storage-consumer)

## コンシューマの設計

以下の図は、コンシューマの全体的な消費プロセスを示しています：

![TiCDCのストレージコンシューマの概要](/media/ticdc/ticdc-storage-consumer-overview.png)

コンシューマの構成要素とその機能は次のように説明されています：

```go
type StorageReader struct {
}
// ストレージからファイルを読み取ります。
// 新しいファイルを追加し、ストレージに存在しないファイルを削除します。
func (c *StorageReader) ReadFiles() {}

// 新しく追加されたファイルとストレージからの最新のチェックポイントをクエリします。1つのファイルは1度だけ返されます。
func (c *StorageReader) ExposeNewFiles() (int64, []string) {}

// ConsumerManagerはTableConsumerにタスクを割り当てる責任があります。
// 異なるコンシューマはデータを同時に消費できますが、1つのテーブルのデータは同じTableConsumerによって処理されなければなりません。
type ConsumerManager struct {
  // StorageCheckpointはメタデータファイルに記録され、`StorageReader.ExposeNewFiles()`を呼び出すことで取得できます。
  // このチェックポイントは、トランザクションのコミット時刻がこのチェックポイントよりも小さいデータがストレージに保存されたことを示します。
  StorageCheckpoint int64
  // このチェックポイントはコンシューマが消費した場所を示します。
  // ConsumerManagerは定期的にTableConsumer.Checkpointを収集し、
  // それからCheckpointをすべてのTableConsumer.Checkpointの最小値に更新します。
  Checkpoint int64

  tableFiles[schema][table]*TableConsumer
}

// StorageReaderから新しく追加されたファイルをクエリします。
// 新しく作成されたテーブルの場合、それに対してTableConsumerを作成します。
// 対応する場合は、新しいファイルを対応するTableConsumerに送信します。
func (c *ConsumerManager) Dispatch() {}
type TableConsumer struct {
  // このチェックポイントはこのTableConsumerが消費した場所を示します。
  // 初期値はConsumerManager.Checkpointです。
  // TableConsumer.CheckpointはTableVersionConsumer.Checkpointと等しいです。
  Checkpoint int64

  schema,table string
  // テーブルバージョンの順序に従って順次消費しなければなりません。
  verConsumers map[version int64]*TableVersionConsumer
  currentVer, previousVer int64
}

// 新しく追加されたファイルを対応するTableVersionConsumerに送信します。
// DDLがある場合は、新しいテーブルバージョンのためにTableVersionConsumerを割り当てます。
func (tc *TableConsumer) Dispatch() {}

// DDLクエリが空であるか、そのtableVersionがTableConsumer.Checkpointよりも小さい場合、
// - このDDLを無視し、テーブルバージョン以下のデータを消費します。
// それ以外の場合、
// - まずDDLを実行し、その後テーブルバージョン以下のデータを消費します。
// - 削除されたテーブルについては、テーブルを削除した後に自動的にリサイクルされます。
func (tc *TableConsumer) ExecuteDDL() {}

type TableVersionConsumer struct {
  // このチェックポイントはTableVersionConsumer が消費した場所を示します。
  // 初期値はTableConsumer.Checkpointです。
  Checkpoint int64

  schema,table,version string
  // 同じテーブルバージョンでは、異なるパーティションのデータを同時に消費できます。
  # partitionNum int64
  // データファイル番号に従って順次消費しなければなりません。
  fileSet map[filename string]*TableVersionConsumer
  currentVersion
}
// データコミット時刻がTableConsumer.Checkpointよりも小さいかConsumerManager.StorageCheckpointよりも大きい場合、
// - このデータを無視します。
// それ以外の場合、
// - このデータを処理し、MySQLに書き込みます。
func (tc *TableVersionConsumer) ExecuteDML() {}
```

## DDLイベントの処理

コンシューマはディレクトリを初めてトラバースします。以下はその例です：

```
├── metadata
└── test
    ├── tbl_1
    │   └── 437752935075545091
    │       ├── CDC000001.json
    │       └── schema.json
```

コンシューマは `schema.json` ファイルのテーブルスキーマを解析し、DDLクエリステートメントを取得します：

- クエリステートメントが見つからないかつ `TableVersion` がコンシューマのチェックポイントよりも小さい場合、コンシューマはこのステートメントをスキップします。
- クエリステートメントが存在するかつ `TableVersion` がコンシューマのチェックポイントと等しいまたはそれ以上の場合、コンシューマはダウンストリームのMySQLでDDLステートメントを実行します。

その後、コンシューマは `CDC000001.json` ファイルの複製を開始します。

次の例では、`test/tbl_1/437752935075545091/schema.json` ファイルのDDLクエリステートメントが空でないことが示されています：

```json
{
    "Table":"test",
    "Schema":"tbl_1",
    "Version": 1,
    "TableVersion":437752935075545091,
    "Query": "create table tbl_1 (Id int primary key, LastName char(20), FirstName varchar(30), HireDate datetime, OfficeLocation Blob(20))",
    "TableColumns":[
        {
            "ColumnName":"Id",
            "ColumnType":"INT",
            "ColumnNullable":"false",
            "ColumnIsPk":"true"
        },
        {
            "ColumnName":"LastName",
            "ColumnType":"CHAR",
            "ColumnLength":"20"
        },
        {
            "ColumnName":"FirstName",
            "ColumnType":"VARCHAR",
            "ColumnLength":"30"
        },
        {
            "ColumnName":"HireDate",
            "ColumnType":"DATETIME"
        },
        {
            "ColumnName":"OfficeLocation",
            "ColumnType":"BLOB",
            "ColumnLength":"20"
        }
    ],
    "TableColumnsTotal":"5"
}
```

コンシューマが再度ディレクトリをトラバースすると、テーブルの新しいバージョンディレクトリを見つけます。コンシューマは `test/tbl_1/437752935075545091` ディレクトリ内のすべてのファイルを消費した後に、新しいディレクトリ内のデータを消費します。

```
├── metadata
└── test
    ├── tbl_1
    │   ├── 437752935075545091
    │   │   ├── CDC000001.json
    │   │   └── schema.json
    │   └── 437752935075546092
    │   │   └── CDC000001.json
    │   │   └── schema.json
```

消費ロジックは一貫しています。具体的には、コンシューマは `schema.json` ファイルのテーブルスキーマを解析し、DDLクエリステートメントを取得および処理します。その後、コンシューマは `CDC000001.json` ファイルの複製を開始します。

## DMLイベントの処理

DDLイベントが適切に処理された後、特定のファイル形式（CSVまたはCanal-JSON）およびファイル番号に基づいて、`{schema}/{table}/{table-version-separator}/` ディレクトリ内のDMLイベントを処理できます。

TiCDCはデータが少なくとも1回複製されることを保証します。したがって、重複するデータがある可能性があります。変更データのコミット時刻とコンシューマのチェックポイントを比較する必要があります。コミット時刻がコンシューマのチェックポイントよりも小さい場合、重複除外を行う必要があります。
```