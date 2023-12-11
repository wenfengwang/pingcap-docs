---
title: TiFlashクラスターのメンテナンス
summary: TiFlashクラスターをメンテナンスする際の一般的な操作方法について学びます。
aliases: ['/docs/dev/tiflash/maintain-tiflash/','/docs/dev/reference/tiflash/maintain/']
---

# TiFlashクラスターのメンテナンス

このドキュメントでは、[TiFlash](/tiflash/tiflash-overview.md)クラスターのメンテナンス時によく行われる操作について説明します。TiFlashのバージョンの確認方法やTiFlashの重要なログ、システムテーブルについても紹介します。

## TiFlashのバージョンを確認する

TiFlashのバージョンを確認する方法には2つあります。

- TiFlashのバイナリファイル名が`tiflash`の場合、`./tiflash version`コマンドを実行してバージョンを確認できます。

    ただし、上記のコマンドを実行するには、TiFlashの実行が`libtiflash_proxy.so`ダイナミックライブラリに依存しているため、`LD_LIBRARY_PATH`環境変数に`libtiflash_proxy.so`ダイナミックライブラリを含むディレクトリパスを追加する必要があります。

    例えば、`tiflash`と`libtiflash_proxy.so`が同じディレクトリにある場合、まずこのディレクトリに移動し、次のコマンドを使用してTiFlashのバージョンを確認できます。

    {{< copyable "shell-regular" >}}

    ```shell
    LD_LIBRARY_PATH=./ ./tiflash version
    ```

- TiFlashのバージョンは、TiFlashログを参照しても確認できます。ログのパスについては、[こちらの`tiflash.toml`ファイル](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)の`[logger]`部分を参照してください。例えば：

    ```
    <information>: TiFlash version: TiFlash 0.2.0 master-375035282451103999f3863c691e2fc2
    ```

## TiFlashの重要なログ

| ログ情報 | ログの説明 |
|---------------|-------------------|
| `[INFO] [<unknown>] ["KVStore: Start to persist [region 47, applied: term 6 index 10]"] [thread_id=23]` | データのレプリケーションが開始される（ログの先頭の角かっこ内の数値はスレッドIDを示す） |
| `[DEBUG] [<unknown>] ["CoprocessorHandler: grpc::Status DB::CoprocessorHandler::execute(): Handling DAG request"] [thread_id=30]` | DAG要求の処理が開始される。つまり、TiFlashがCoprocessor要求を処理し始める |
| `[DEBUG] [<unknown>] ["CoprocessorHandler: grpc::Status DB::CoprocessorHandler::execute(): Handle DAG request done"] [thread_id=30]` | DAG要求の処理が完了する。つまり、TiFlashがCoprocessor要求の処理を終える |

Coprocessor要求の開始や終了を見つけ、その後ログの始めに表示されるスレッドIDを使って、Coprocessor要求に関連するログを特定できます。

## TiFlashシステムテーブル

`information_schema.tiflash_replica`システムテーブルの列名と説明は次のとおりです：

| 列名 | 説明 |
|---------------|-----------|
| TABLE_SCHEMA | データベース名 |
| TABLE_NAME | テーブル名 |
| TABLE_ID | テーブルID |
| REPLICA_COUNT | TiFlashレプリカの数 |
|LOCATION_LABELS | リージョン内の複数のレプリカが散らばるPDへのヒント |
| AVAILABLE | 使用可能か否か（0/1）|
| PROGRESS | レプリケーション進行状況 [0.0〜1.0] |