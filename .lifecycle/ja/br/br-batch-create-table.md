---
title: バッチ作成テーブル
summary: バッチ作成テーブル機能の使用方法を学びます。データをリストアする際に、BRはテーブルの作成をバッチで行い、リストアプロセスを高速化することができます。
---

# バッチ作成テーブル

データをリストアする際、Backup & Restore（BR）はターゲットのTiDBクラスターにデータベースとテーブルを作成し、その後バックアップデータをテーブルに復元します。TiDB v6.0.0より前のバージョンでは、BRはリストアプロセスでテーブルを作成する際に[直列実行](#implementation)の実装を使用していました。しかしながら、BRは多くのテーブル（約50000）をリストアする場合、この実装ではたくさんの時間がかかります。

テーブル作成プロセスを高速化し、データリストアにかかる時間を短縮するために、TiDB v6.0.0でバッチ作成テーブル機能が導入されました。この機能はデフォルトで有効になっています。

> **注記:**
>
> - バッチ作成テーブル機能を使用するには、TiDBとBRの両方がv6.0.0以降である必要があります。TiDBまたはBRのどちらかがv6.0.0より前のバージョンの場合、BRは直列実行の実装を使用します。
> - TiUPなどのクラスター管理ツールを使用し、TiDBとBRがv6.0.0以降のバージョンであるか、またはTiDBとBRがv6.0.0以前のバージョンからv6.0.0以降にアップグレードされている場合を想定してください。

## 使用シナリオ

例えば50000のテーブルを含む大量のデータをリストアする必要がある場合、バッチ作成テーブル機能を使用してリストアプロセスを高速化することができます。

詳細な効果については、[バッチ作成テーブル機能のテスト](#feature-test)を参照してください。

## バッチ作成テーブルの使用

TiDB v6.0.0以降では、BRはデフォルトで`--ddl-batch-size=128`のデフォルト設定でバッチ作成テーブル機能を有効にし、リストアプロセスを高速化します。したがって、このパラメータを設定する必要はありません。`--ddl-batch-size=128`はテーブルをバッチで作成し、各バッチに128のテーブルが含まれることを意味します。

この機能を無効にするには、`--ddl-batch-size`を`1`に設定できます。次の例のコマンドを参照してください:

```shell
br restore full \
--storage local:///br_data/ --pd "${PD_IP}:2379" --log-file restore.log \
--ddl-batch-size=1
```

この機能を無効にした後は、BRは[直列実行の実装](#implementation)を使用します。

## 実装

- v6.0.0以前の直列実行の実装:

    データをリストアする際、BRはターゲットのTiDBクラスターにデータベースとテーブルを作成し、その後バックアップデータをテーブルに復元します。テーブルを作成する際、BRはまずTiDBの内部APIを呼び出し、その後テーブル作成タスクを処理します。これは`Create Table`ステートメントの実行と同様に機能します。TiDBのDDLオーナーは順次テーブルを作成します。DDLオーナーがテーブルを作成すると、DDLスキーマバージョンが対応するように変更され、各バージョンの変更が他のTiDBのDDLワーカー（BRを含む）に同期されます。したがって、大量のテーブルをリストアする際に、直列実行の実装は時間がかかります。

- v6.0.0以降のバッチ作成テーブルの実装:

    デフォルトでは、BRは複数のバッチでテーブルを作成し、各バッチには128のテーブルが含まれます。この実装では、BRが1つのバッチのテーブルを作成する際、TiDBのスキーマバージョンが1回だけ変更されます。この実装により、テーブル作成の速度が大幅に向上します。

## 機能テスト

このセクションでは、バッチ作成テーブル機能に関するテスト情報が記載されています。テスト環境は以下のとおりです:

- クラスターの構成:

    - 15 TiKVインスタンス。各TiKVインスタンスには16 CPUコア、80 GBのメモリ、16スレッドを処理するためのRPCリクエスト（[`import.num-threads`](/tikv-configuration-file.md#num-threads) = 16）が搭載されています。
    - 3 TiDBインスタンス。各TiDBインスタンスには16 CPUコア、32 GBのメモリが搭載されています。
    - 3 PDインスタンス。各PDインスタンスには16 CPUコア、32 GBのメモリが搭載されています。

- リストアするデータのサイズ: 16.16 TB

テスト結果は以下の通りです:

```
'[2022/03/12 22:37:49.060 +08:00] [INFO] [collector.go:67] ["Full restore success summary"] [total-ranges=751760] [ranges-succeed=751760] [ranges-failed=0] [split-region=1h33m18.078448449s] [restore-ranges=542693] [total-take=1h41m35.471476438s] [restore-data-size(after-compressed)=8.337TB] [Size=8336694965072] [BackupTS=431773933856882690] [total-kv=148015861383] [total-kv-size=16.16TB] [average-speed=2.661GB/s]'
```

テスト結果から、1つのTiKVインスタンスのリストア速度が平均181.65 MB/s（`average-speed`/`tikv_count`）という高速なものであることがわかります。