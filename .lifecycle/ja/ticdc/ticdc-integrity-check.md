---
title: 単一行データのためのTiCDCデータ整合性検証
summary: TiCDCデータ整合性検証機能の実装原理と使用法を紹介します。
---

# 単一行データのためのTiCDCデータ整合性検証

v7.1.0から、TiCDCは単一行データの整合性検証機能を導入しました。この機能は、チェックサムアルゴリズムを使用して単一行データの整合性を検証します。この機能を使用すると、TiDBからデータを書き込み、TiCDCを介してレプリケートし、Kafkaクラスタに書き込む過程でエラーが発生していないかを検証するのに役立ちます。データ整合性検証機能は、Kafkaを下流として使用し、現在Avroプロトコルをサポートしています。

## 実装原理

単一行データのためのチェックサム整合性検証機能を有効にすると、TiDBはCRC32アルゴリズムを使用して行のチェックサムを計算し、データと一緒にTiKVに書き込みます。TiCDCはTiKVからデータを読み取り、同じアルゴリズムを使用してチェックサムを再計算します。2つのチェックサムが等しい場合、TiDBからTiCDCへの伝送中にデータが整合していることを示します。

その後、TiCDCはデータを特定の形式にエンコードしてKafkaに送信します。Kafka Consumerがデータを読み取った後、TiDBと同じアルゴリズムを使用して新しいチェックサムを計算します。新しいチェックサムがデータ内のチェックサムと等しい場合、TiCDCからKafka Consumerへの伝送中にデータが整合していることを示します。

チェックサムのアルゴリズムの詳細については、[チェックサム計算のためのアルゴリズム](#algorithm-for-checksum-calculation)を参照してください。

## 機能の有効化

TiCDCはデフォルトでデータ整合性検証を無効にしています。有効にするには、次の手順を実行してください。

1. チェックサム整合性検証機能を、アップストリームのTiDBクラスタで行のチェックサム整合性検証を有効に設定することで、次のグローバルSQLステートメントを実行します：

    ```sql
    SET GLOBAL tidb_enable_row_level_checksum = ON;
    ```

    この設定は新しく作成されたセッションにのみ影響しますので、TiDBに再接続する必要があります。

2. changefeedを作成する際の`--config`パラメータで指定された[設定ファイル](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters)に、以下の設定を追加します：

    ```toml
    [integrity]
    integrity-check-level = "correctness"
    corruption-handle-level = "warn"
    ```

3. データエンコーディングフォーマットとしてAvroを使用する場合、[`sink-uri`](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka)で`enable-tidb-extension=true`を設定する必要があります。ネットワーク転送中に数値の精度損失が発生し、チェックサム検証の失敗を引き起こすことを防ぐために、[`avro-decimal-handling-mode=string`](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka)と[`avro-bigint-unsigned-handling-mode=string`](/ticdc/ticdc-sink-to-kafka.md#configure-sink-uri-for-kafka)を設定する必要があります。次は例です：

    ```shell
    cdc cli changefeed create --server=http://127.0.0.1:8300 --changefeed-id="kafka-avro-checksum" --sink-uri="kafka://127.0.0.1:9092/topic-name?protocol=avro&enable-tidb-extension=true&avro-decimal-handling-mode=string&avro-bigint-unsigned-handling-mode=string" --schema-registry=http://127.0.0.1:8081 --config changefeed_config.toml
    ```

    上記の設定により、changefeedによってKafkaに書き込まれる各メッセージに対して、対応するデータのチェックサムが含まれます。これらのチェックサムの値に基づいてデータの整合性を検証できます。

    > **注意：**
    >
    > 既存のchangefeedに対して`avro-decimal-handling-mode`と`avro-bigint-unsigned-handling-mode`が設定されていない場合、チェックサム検証機能を有効にするとスキーマの互換性に問題が発生する可能性があります。この問題を解決するには、Schema Registryの互換性タイプを`NONE`に変更できます。詳細については、[Schema Registry](https://docs.confluent.io/platform/current/schema-registry/fundamentals/avro.html#no-compatibility- checking)を参照してください。

## 機能の無効化

TiCDCはデフォルトでデータ整合性検証機能を無効にしています。有効にした後にこの機能を無効にするには、次の手順を実行してください。

1. changefeedの`--config`パラメータで指定された設定ファイルからすべての`[integrity]`設定を削除し、「タスク一時停止→構成の変更→タスク再開」の手順に従います。具体的な手順については、[タスク構成の更新](/ticdc/ticdc-manage-changefeed.md#update-task-configuration)を参照してください。

    ```toml
    [integrity]
    integrity-check-level = "none"
    corruption-handle-level = "warn"
    ```

2. アップストリームのTiDBで次のSQLステートメントを実行して、チェックサム整合性検証機能（[`tidb_enable_row_level_checksum`](/system-variables.md#tidb_enable_row_level_checksum-new-in-v710)）を無効にします：

    ```sql
    SET GLOBAL tidb_enable_row_level_checksum = OFF;
    ```

    上記の設定は新しく作成されたセッションにのみ影響します。 TiDBに書き込むすべてのクライアントが再接続した後、changefeedによってKafkaに書き込まれるメッセージには、もはや対応するデータのチェックサムが含まれなくなります。

## チェックサム計算のためのアルゴリズム

チェックサム計算アルゴリズムの擬似コードは次のようになります：

```
fn チェックサム(カラム) {
    let 結果 = 0
    for カラム in スキーマ順に並べ替える(カラム) {
        結果 = crc32.update(結果, エンコード(カラム))
    }
    return 結果
}
```

* `カラム`は列IDで並べ替えられなければなりません。Avroスキーマでは、フィールドはすでに列IDで並べられているため、`カラム`の順序を直接使用できます。

* `エンコード(カラム)`関数は、列の値をバイトにエンコードします。エンコーディングルールは、列のデータ型に基づいて異なります。具体的なルールは以下の通りです：

    * TINYINT、SMALLINT、INT、BIGINT、MEDIUMINT、YEARのタイプはUINT64に変換され、リトルエンディアンでエンコードされます。例えば、`0x0123456789abcdef`という数字は、`hex'0x0123456789abcdef'`とエンコードされます。
    * FLOATとDOUBLEのタイプはDOUBLEに変換され、その後IEEE754フォーマットのUINT64としてエンコードされます。
    * BIT、ENUM、SETのタイプはUINT64に変換されます。

        * BITタイプはバイナリ形式のUINT64に変換されます。
        * ENUMおよびSETタイプは、それぞれのINT値をUINT64に変換されます。例えば、`SET('a','b','c')`タイプの列のデータ値が`'a,c'`の場合、その値は`0b101`であり、10進数では`5`にエンコードされます。

    * TIMESTAMP、DATE、DURATION、DATETIME、JSON、DECIMALのタイプはまずSTRINGに変換され、次にバイトに変換されます。
    * CHAR、VARCHAR、VARSTRING、STRING、TEXT、BLOBのタイプ（TINY、MEDIUM、LONGを含む）は直接バイトに変換されます。
    * NULLおよびGEOMETRYのタイプは、チェックサムの計算から除外され、この関数は空のバイトを返します。

Golangを使用したデータ消費とチェックサム検証の実装についての詳細については、[TiCDC行データチェックサム検証](/ticdc/ticdc-avro-checksum-verification.md)を参照してください。

> **注意：**
>
> - チェックサム検証機能を有効にした場合、DECIMALおよびUNSIGNED BIGINTタイプのデータはSTRINGタイプに変換されます。そのため、下流のコンシューマコードで、チェックサムの値を計算する前に、それらを対応する数値タイプに変換する必要があります。
> - チェックサム検証プロセスにはDELETEイベントは含まれません。これは、DELETEイベントにはハンドルキーカラムのみが含まれるためであり、一方、チェックサムはすべての列に基づいて計算されるからです。