---
title: TiDB Lightningのデプロイ
summary: TiDB Lightningを使用して大量の新しいデータを迅速にインポートするためのTiDB Lightningのデプロイについて説明します。
aliases: ['/docs/dev/tidb-lightning/deplopy-tidb-lightning/','/docs/dev/reference/tools/tidb-lightning/deployment/']
---

# TiDB Lightningのデプロイ

このドキュメントでは、TiDB Lightningを使用してデータをインポートし、それを手動でデプロイするためのハードウェア要件について説明します。ハードウェアリソースの要件はインポートモードによって異なります。詳細については、以下のドキュメントを参照してください。

- [物理的インポートモードの要件と制限事項](/tidb-lightning/tidb-lightning-physical-import-mode.md#requirements-and-restrictions)
- [論理的インポートモードの要件と制限事項](/tidb-lightning/tidb-lightning-logical-import-mode.md)

## TiUPを使用したオンラインデプロイ（推奨）

1. 以下のコマンドを使用してTiUPをインストールします。

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
    ```

    このコマンドは自動的にTiUPを`PATH`環境変数に追加します。TiUPを使用する前に新しいターミナルセッションを開始するか、`source ~/.bashrc`を実行する必要があります（環境によっては`source ~/.profile`を実行する必要がある場合もあります。具体的なコマンドについては、TiUPの出力を確認してください）。

2. TiUPを使用してTiDB Lightningをインストールします。

    ```shell
    tiup install tidb-lightning
    ```

## 手動でのデプロイ

### TiDB Lightningバイナリのダウンロード

[Download TiDB Tools](/download-ecosystem-tools.md)を参照して、TiDB Lightningバイナリをダウンロードします。TiDB LightningはTiDBの旧バージョンと完全に互換性があります。最新バージョンのTiDB Lightningを使用することをお勧めします。

TiDB Lightningバイナリパッケージを展開して`tidb-lightning`実行ファイルを取得します。

```bash
tar -zxvf tidb-lightning-${version}-linux-amd64.tar.gz
chmod +x tidb-lightning
```

このコマンドでは

- `-B test`: はデータが`test`データベースからエクスポートされることを意味します。
- `-f test.t[12]`: は`test.t1`と`test.t2`テーブルのみがエクスポートされることを意味します。
- `-t 16`: はデータのエクスポートに16スレッドが使用されることを意味します。
- `-F 256MB`: はテーブルをチャンクに分割し、各チャンクが256 MBであることを意味します。

データソースがCSVファイルで構成されている場合は、構成については[CSVサポート](/tidb-lightning/tidb-lightning-data-source.md#csv)を参照してください。

## TiDB Lightningのデプロイ

このセクションでは、[TiDB Lightningを手動でデプロイ](#deploy-tidb-lightning-manually)する方法について説明します。

### TiDB Lightningの手動デプロイ

#### ステップ1: TiDBクラスタのデプロイ

データをインポートする前に、デプロイされたTiDBクラスタが必要です。最新の安定バージョンを使用することを強くお勧めします。

[TiDBクイックスタートガイド](/quick-start-with-tidb.md)にデプロイ手順が記載されています。

#### ステップ2: TiDB Lightningインストールパッケージのダウンロード

[TiDB Toolsをダウンロード](/download-ecosystem-tools.md)ドキュメントを参照して、TiDB Lightningパッケージをダウンロードします。

> **注意:**
>
> TiDB Lightningは以前のバージョンのTiDBクラスタと互換性があります。TiDB Lightningの最新安定バージョンをダウンロードすることをお勧めします。

#### ステップ3: `tidb-lightning`を起動

1. ツールセットから`bin/tidb-lightning`および`bin/tidb-lightning-ctl`をアップロードします。

2. 同じマシンにデータソースをマウントします。

3. `tidb-lightning.toml`を設定します。以下のテンプレートに表示されない設定については、TiDB Lightningは構成エラーをログファイルに書き込んで終了します。

    `sorted-kv-dir`は、ソートされたKey-Valueファイルの一時的な保存ディレクトリを設定します。ディレクトリは空である必要があり、ストレージスペースは**インポートするデータのサイズよりも大きい**必要があります。詳細については[ターゲットデータベースのダウンストリームストレージスペース要件](/tidb-lightning/tidb-lightning-requirements.md#storage-space-of-the-target-database)を参照してください。

    ```toml
    [lightning]
    # コア数と同じ数のデータの並列処理数を設定します。デフォルトでは、論理CPUコア数に設定されます。他のコンポーネントと一緒にデプロイする場合は、CPU使用率を制限するために論理CPUコア数の75%に設定できます。
    # region-concurrency =

    # ロギング
    level = "info"
    file = "tidb-lightning.log"

    [tikv-importer]
    # バックエンドを「local」モードに設定します。
    backend = "local"
    # 一時的なローカルストレージのディレクトリを設定します。
    sorted-kv-dir = "/mnt/ssd/sorted-kv-dir"

    [mydumper]
    # ローカルデータソースディレクトリ
    data-source-dir = "/data/my_database"

    [tidb]
    # クラスタ内のいずれかのTiDBサーバーの構成
    host = "172.16.31.1"
    port = 4000
    user = "root"
    password = ""
    # テーブルスキーマ情報は、このステータスポートを介してTiDBから取得されます。
    status-port = 10080
    # pd-serverのアドレス
    pd-addr = "172.16.31.4:2379"
    ```

    上記は必須の設定のみを示しています。全ての設定のリストについては[Configuration](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-global)セクションを参照してください。

4. `tidb-lightning`を実行します。

    ```sh
    nohup ./tidb-lightning -config tidb-lightning.toml > nohup.out &
    ```

## TiDB Lightningのアップグレード

TiDB Lightningはバイナリの置換のみで簡単にアップグレードできます。アップグレード後にTiDB Lightningを再起動する必要があります。詳細については[TiDB Lightningを正しく再起動する方法](/tidb-lightning/tidb-lightning-faq.md#how-to-properly-restart-tidb-lightning)を参照してください。

インポートタスクが実行中の場合は、TiDB Lightningをアップグレードする前に完了するのを待つことをお勧めします。それ以外の場合、バージョン間でチェックポイントが機能する保証がないため、ゼロから再インポートする必要があります。