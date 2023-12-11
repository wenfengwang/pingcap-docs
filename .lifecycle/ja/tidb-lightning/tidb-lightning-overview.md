---
title: TiDB Lightningの概要
summary：Lightningとアーキテクチャ全般について学びます。
エイリアス：['/docs/dev/tidb-lightning/tidb-lightning-overview/','/docs/dev/reference/tools/tidb-lightning/overview/','/docs/dev/tidb-lightning/tidb-lightning-tidb-backend/','/docs/dev/reference/tools/tidb-lightning/tidb-backend/','/tidb/dev/tidb-lightning-tidb-backend','/docs/dev/loader-overview/','/docs/dev/reference/tools/loader/','/docs/dev/load-misuse-handling/','/docs/dev/reference/tools/error-case-handling/load-misuse-handling/','/tidb/dev/load-misuse-handling','/tidb/dev/loader-overview/','/tidb/dev/tidb-lightning-backends']
---

# TiDB Lightningの概要

[TiDB Lightning](https://github.com/pingcap/tidb/tree/master/br/pkg/lightning)は、TiDBクラスタにTB単位のデータをインポートするためのツールです。通常、TiDBクラスタへの初期データのインポートに使用されます。

TiDB Lightningは、次のファイル形式をサポートしています。

- [Dumplingによってエクスポートされたファイル](/dumpling-overview.md)
- CSVファイル
- [Amazon Auroraで生成されたApache Parquetファイル](/migrate-aurora-to-tidb.md)

TiDB Lightningは、次のソースからデータを読み取ることができます。

- ローカル
- [Amazon S3](/external-storage-uri.md#amazon-s3-uri-format)
- [Google Cloud Storage](/external-storage-uri.md#gcs-uri-format)
- [Azure Blob Storage](/external-storage-uri.md#azure-blob-storage-uri-format)

## TiDB Lightningのアーキテクチャ

![TiDB Lightningツールセットのアーキテクチャ](/media/tidb-lightning-architecture.png)

TiDB Lightningは、`backend`によって構成される2つのインポートモードをサポートしています。インポートモードは、データがTiDBにインポートされる方法を決定します。

- [物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode.md)：TiDB Lightningはまずデータをキーと値のペアにエンコードし、これらをローカルの一時ディレクトリに格納し、それらのキーと値のペアを各TiKVノードにアップロードし、最後にTiKV Ingestインターフェースを呼び出してデータをTiKVのRocksDBに挿入します。初期インポートを行う必要がある場合は、インポート速度が速い物理インポートモードを考慮してください。物理インポートモードのバックエンドは`local`です。

- [論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode.md)：TiDB LightningはまずデータをSQLステートメントにエンコードし、その後これらのSQLステートメントを直接実行してデータをインポートします。インポート対象のクラスタが本番環境にある場合や、インポート対象のテーブルにすでにデータが含まれている場合は、論理インポートモードを使用してください。論理インポートモードのバックエンドは`tidb`です。

| インポートモード | 物理インポートモード | 論理インポートモード |
|:---|:---|:---|
| バックエンド | `local` | `tidb` |
| 速度 | 高速（100〜500 GiB/時間） | 低速（10〜50 GiB/時間） |
| リソース消費 | 高 | 低 |
| ネットワーク帯域幅消費 | 高 | 低 |
| インポート中のACID準拠 | なし | あり |
| インポート対象のテーブル | 空である必要があります | データを含んでいても構いません |
| TiDBクラスタのバージョン | >= 4.0.0 | すべて |
| インポート中にTiDBクラスタがサービスを提供できるか | [サービスが制限されます](/tidb-lightning/tidb-lightning-physical-import-mode.md#limitations) | はい |

<Note>

前述のパフォーマンスデータは、2つのモード間のインポートパフォーマンスの違いを比較するために使用されます。実際のインポート速度は、ハードウェア構成、テーブルスキーマ、およびインデックスの数など、さまざまな要因に影響を受けます。

</Note>