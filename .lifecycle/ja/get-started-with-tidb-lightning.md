---
title: TiDB Lightningのはじめかた
summary: TiDB Lightningの展開方法とフルバックアップデータのインポート方法を学びます。
aliases: ['/docs/dev/get-started-with-tidb-lightning/','/docs/dev/how-to/get-started/tidb-lightning/']
---

# TiDB Lightningのはじめかた

このチュートリアルでは、いくつか新しくてクリーンなCentOS 7のインスタンスを使用することを前提としています。VMware、VirtualBox、またはその他のツールを使用して、ローカルに仮想マシンをデプロイするか、ベンダー提供のプラットフォーム上で小規模なクラウド仮想マシンをデプロイできます。TiDB Lightningは大量のコンピュータリソースを消費するため、最高のパフォーマンスで実行するには少なくとも16 GBのメモリと32コアのCPUを割り当てることをお勧めします。

> **注意:**
>
> このチュートリアルでの展開方法はテストやトライアルにのみ推奨されます。本番環境や開発環境に適用しないでください。

## フルバックアップデータの準備

まず、[`dumpling`](/dumpling-overview.md)を使用してMySQLからデータをエクスポートします。

{{< copyable "shell-regular" >}}

```sh
tiup dumpling -h 127.0.0.1 -P 3306 -u root -t 16 -F 256MB -B test -f 'test.t[12]' -o /data/my_database/
```

上記のコマンドでは:

- `-B test`: `test`データベースからデータをエクスポートすることを意味します。
- `-f test.t[12]`: `test.t1`と`test.t2`テーブルのみをエクスポートすることを意味します。
- `-t 16`: 16スレッドを使用してデータをエクスポートすることを意味します。
- `-F 256MB`: テーブルをチャンクに分割し、1つのチャンクのサイズが256 MBであることを意味します。

このコマンドを実行すると、フルバックアップデータは`/data/my_database`ディレクトリにエクスポートされます。

## TiDB Lightningの展開

### ステップ1: TiDBクラスタの展開

データのインポートの前に、TiDBクラスタを展開する必要があります。このチュートリアルでは、TiDB v5.4.0を例として使用しています。展開方法については、[TiUPを使用してTiDBクラスタを展開](/production-deployment-using-tiup.md)を参照してください。

### ステップ2: TiDB Lightningインストールパッケージのダウンロード

TiDB LightningインストールパッケージはTiDB Toolkitに含まれています。TiDB Toolkitをダウンロードするには、[TiDBツールのダウンロード](/download-ecosystem-tools.md)を参照してください。

> **ノート:**
>
> TiDB Lightningは古いバージョンのTiDBクラスタと互換性があります。TiDB Lightningの最新安定版をダウンロードすることをお勧めします。

### ステップ3: `tidb-lightning`の起動

1. パッケージ内の`bin/tidb-lightning`と`bin/tidb-lightning-ctl`をTiDB Lightningを展開したサーバにアップロードします。
2. [準備したデータソース](#prepare-full-backup-data)をサーバにアップロードします。
3. 次のように`tidb-lightning.toml`を設定します:

    ```toml
    [lightning]
    # ログ
    level = "info"
    file = "tidb-lightning.log"

    [tikv-importer]
    # インポートモードの設定
    backend = "local"
    # ソートされたキー値ペアを一時的に格納するディレクトリを設定します。
    # ターゲットディレクトリは空である必要があります。
    sorted-kv-dir = "/mnt/ssd/sorted-kv-dir"

    [mydumper]
    # ローカルソースデータディレクトリ
    data-source-dir = "/data/my_datasource/"

    # ワイルドカードルールを設定します。デフォルトでは、mysql、sys、INFORMATION_SCHEMA、PERFORMANCE_SCHEMA、METRICS_SCHEMA、およびINSPECTION_SCHEMAシステムデータベースのすべてのテーブルがフィルタされます。
    # この項目が設定されていない場合、システムテーブルのインポート時に「スキーマが見つかりません」というエラーが発生します。
    filter = ['*.*', '!mysql.*', '!sys.*', '!INFORMATION_SCHEMA.*', '!PERFORMANCE_SCHEMA.*', '!METRICS_SCHEMA.*', '!INSPECTION_SCHEMA.*']
    [tidb]
    # ターゲットクラスタの情報
    host = "172.16.31.2"
    port = 4000
    user = "root"
    password = "rootroot"
    # テーブルスキーマ情報は、このstatus-port経由でTiDBから取得されます。
    status-port = 10080
    # クラスタのPDアドレス
    pd-addr = "172.16.31.3:2379"
    ```

4. パラメータを適切に設定した後、`nohup`コマンドを使用して`tidb-lightning`プロセスを起動します。コマンドを直接コマンドラインで実行すると、SIGHUPシグナルを受信したためプロセスが終了する可能性があります。その代わりに、`nohup`コマンドを含むbashスクリプトを実行することをお勧めします:

    {{< copyable "shell-regular" >}}

    ```sh
    #!/bin/bash
    nohup tiup tidb-lightning -config tidb-lightning.toml > nohup.out &
    ```

### ステップ4: データの整合性を確認

インポートが完了すると、TiDB Lightningは自動的に終了します。インポートが成功すると、ログファイルの最終行に`tidb lightning exit`が見つかります。

エラーが発生した場合は、[TiDB Lightning FAQs](/tidb-lightning/tidb-lightning-faq.md)を参照してください。

## 概要

このチュートリアルでは、TiDB Lightningの概要とTiDBクラスタにフルバックアップデータをインポートする方法について簡単に紹介しました。

TiDB Lightningの詳細な機能や使用方法については、[TiDB Lightning概要](/tidb-lightning/tidb-lightning-overview.md)を参照してください。