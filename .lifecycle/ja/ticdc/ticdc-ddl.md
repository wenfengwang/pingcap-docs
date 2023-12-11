---
title: DDLレプリケーション
summary: TiCDCがサポートするDDLステートメントおよび特別なケースについて学ぶ。
---

# DDLレプリケーション

このドキュメントでは、TiCDCにおけるDDLレプリケーションのルールと特別なケースについて説明します。

## DDL allow list

現在、TiCDCは許可リストを使用してDDLステートメントをレプリケーションするかどうかを判断します。許可リストに含まれるDDLステートメントのみがダウンストリームにレプリケーションされます。許可リストに含まれないDDLステートメントはレプリケーションされません。

TiCDCがサポートするDDLステートメントの許可リストは次のとおりです。

- create database
- drop database
- create table
- drop table
- add column
- drop column
- create index / add index
- drop index
- truncate table
- modify column
- rename table
- alter column default value
- alter table comment
- rename index
- add partition
- drop partition
- truncate partition
- create view
- drop view
- alter table character set
- alter database character set
- recover table
- add primary key
- drop primary key
- rebase auto id
- alter table index visibility
- exchange partition
- reorganize partition
- alter table ttl
- alter table remove ttl

## DDLレプリケーションの考慮事項

### 「ADD INDEX」と「CREATE INDEX」DDLの非同期実行

ダウンストリームがTiDBの場合、TiCDCは「ADD INDEX」と「CREATE INDEX」DDL操作を非同期で実行し、チェンジフィードのレプリケーション遅延に影響を最小限に抑えます。つまり、TiCDCは「ADD INDEX」と「CREATE INDEX」DDLをダウンストリームのTiDBにレプリケーションしてから、DDLの実行が完了するのを待たずにすぐ返します。これにより、後続のDML実行がブロックされるのを避けることができます。

> **注意:**
>
> - ダウンストリームの特定のDMLの実行が未完成のインデックスに依存している場合、これらのDMLは遅延して実行される可能性があり、TiCDCのレプリケーション遅延に影響を与えることがあります。
> - ダウンストリームにDDLをレプリケーションする前に、TiCDCノードがクラッシュしたり、ダウンストリームで他の書き込み操作が実行されている場合、DDLのレプリケーションは非常に低い確率で失敗する可能性があります。これが発生していないかどうかをダウンストリームで確認できます。

### テーブル名の変更に関するDDLレプリケーションの考慮事項

レプリケーションプロセス中に一部のコンテキストが欠落するため、TiCDCは「RENAME TABLE」DDLのレプリケーションにいくつかの制約があります。

#### DDLステートメントで単一のテーブルを名前変更する場合

DDLステートメントで単一のテーブルの名前を変更する場合、古いテーブル名がフィルタルールと一致する場合のみ、TiCDCはDDLステートメントをレプリケーションします。以下は例です。

変更フィードの構成ファイルが次のような場合を想定します。

```toml
[filter]
rules = ['test.t*']
```

TiCDCはこのタイプのDDLを次のように処理します。

| DDL | レプリケーションするかどうか | 処理の理由 |
| --- | --- | --- |
| `RENAME TABLE test.t1 TO test.t2` | レプリケーションする | `test.t1`がフィルタルールに一致 |
| `RENAME TABLE test.t1 TO ignore.t1` | レプリケーションする | `test.t1`がフィルタルールに一致 |
| `RENAME TABLE ignore.t1 TO ignore.t2` | 無視 | `ignore.t1`がフィルタルールに一致しない |
| `RENAME TABLE test.n1 TO test.t1` | エラーを報告してレプリケーションを終了 | `test.n1`がフィルタルールに一致しないが、`test.t1`がフィルタルールに一致します。この操作は不正です。この場合は、処理についてのエラーメッセージを参照してください |
| `RENAME TABLE ignore.t1 TO test.t1` | エラーを報告してレプリケーションを終了 | 上記と同じ理由 |

#### DDLステートメントで複数のテーブルを名前変更する場合

DDLステートメントで複数のテーブルの名前を変更する場合、TiCDCは古いデータベース名、古いテーブル名、および新しいデータベース名がすべてフィルタルールと一致する場合のみ、DDLステートメントをレプリケーションします。

また、TiCDCはテーブル名を交換する「RENAME TABLE」DDLをサポートしていません。以下は例です。

変更フィードの構成ファイルが次のような場合を想定します。

```toml
[filter]
rules = ['test.t*']
```

TiCDCはこのタイプのDDLを次のように処理します。

| DDL | レプリケーションするかどうか | 処理の理由 |
| --- | --- | --- |
| `RENAME TABLE test.t1 TO test.t2, test.t3 TO test.t4` | レプリケーションする | すべてのデータベース名とテーブル名がフィルタルールに一致 |
| `RENAME TABLE test.t1 TO test.ignore1, test.t3 TO test.ignore2` | レプリケーションする | 古いデータベース名、古いテーブル名、新しいデータベース名がフィルタルールに一致 |
| `RENAME TABLE test.t1 TO ignore.t1, test.t2 TO test.t22;` | エラーを報告 | 新しいデータベース名`ignore`がフィルタルールに一致しない |
| `RENAME TABLE test.t1 TO test.t4, test.t3 TO test.t1, test.t4 TO test.t3;` | エラーを報告 | 「RENAME TABLE」DDLが1つのDDLステートメントで`test.t1`と`test.t3`の名前を交換しているため、TiCDCが正しく処理できません。この場合は、処理についてのエラーメッセージを参照してください |

### SQLモード

デフォルトでは、TiCDCはDDLステートメントを解析するためにTiDBのデフォルトSQLモードを使用します。上流のTiDBクラスターがデフォルトでないSQLモードを使用している場合は、TiCDCの構成ファイルでSQLモードを指定する必要があります。そうしないと、TiCDCはDDLステートメントの解析に失敗する可能性があります。TiDBのSQLモードについての詳細は、[SQL Mode](/sql-mode.md)を参照してください。

例えば、上流のTiDBクラスターが`ANSI_QUOTES`モードを使用している場合、変更フィードの構成ファイルで次のようにSQLモードを指定する必要があります。

```toml
# 値の中で、"ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"はTiDBのデフォルトSQLモードです。
# "ANSI_QUOTES"は上流のTiDBクラスターに追加されたSQLモードです。

sql-mode = "ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,ANSI_QUOTES"
```

SQLモードが構成されていない場合、TiCDCは一部のDDLステートメントを正しく解析できない可能性があります。例えば以下のような場合です。

```sql
CREATE TABLE "t1" ("a" int PRIMARY KEY);
```

TiDBのデフォルトSQLモードでは、二重引用符は識別子ではなく文字列として扱われるため、TiCDCはこのDDLステートメントを正しく解析できません。

そのため、レプリケーションタスクを作成する際には、上流のTiDBクラスターで使用されているSQLモードを構成ファイルで指定することをお勧めします。