---
title: UUID ベストプラクティス
summary: TiDB での UUID の使用に関するベストプラクティスとストラテジーを学びます。

# UUID ベストプラクティス

## UUID の概要

ユニバーサルユニーク識別子（UUID）を主キーとして使用すると、`AUTO_INCREMENT` 整数値の代わりに以下の利点が得られます。

- UUID は複数のシステムで生成され、競合のリスクを回避できます。いくつかのケースでは、これにより TiDB へのネットワークトリップの回数が減少し、パフォーマンスが向上することがあります。
- UUID は多くのプログラミング言語とデータベースシステムでサポートされています。
- UUID は URL の一部として使用する場合、列挙攻撃に対して脆弱ではありません。これに対し、`auto_increment` 数値では、請求書 ID やユーザー ID を推測することが可能です。

## ベストプラクティス

### バイナリとして保存

テキスト形式の UUID は次のようになります：`ab06f63e-8fe7-11ec-a514-5405db7aad56`、これは 36 文字の文字列です。`UUID_TO_BIN()` を使用することで、テキスト形式を 16 バイトのバイナリ形式に変換できます。これにより、テキストを `BINARY(16)` 列に保存できます。UUID を取得する際には、`BIN_TO_UUID()` 関数を使用してテキスト形式に戻すことができます。

### UUID フォーマットのバイナリ順序とクラスタ化された主キー

`UUID_TO_BIN()` 関数は、UUID を1つの引数として使用する他、2つの引数を指定する場合もあります。2つ目の引数は `swap_flag` と呼ばれます。

<CustomContent platform="tidb">

[ホットスポット](/best-practices/high-concurrency-best-practices.md) を回避するために TiDB で `swap_flag` を設定しないことを推奨します。

</CustomContent>

<CustomContent platform="tidb-cloud">

ホットスポットを回避するために TiDB で `swap_flag` を設定しないことを推奨します。

</CustomContent>

UUID ベースの主キーに対して明示的に [`CLUSTERED` オプション](/clustered-indexes.md) を設定することもでき、これによりホットスポットを回避できます。

`swap_flag` の効果を示すために、同一構造の2つのテーブルがあります。違いは、`uuid_demo_1` に挿入されるデータでは `UUID_TO_BIN(?, 0)` を使用し、`uuid_demo_2` に挿入されるデータでは `UUID_TO_BIN(?, 1)` を使用している点です。

<CustomContent platform="tidb">

以下の [Key Visualizer](/dashboard/dashboard-key-visualizer.md) のスクリーンショットでは、バイナリ形式でフィールドの順序が入れ替えられている `uuid_demo_2` テーブルの単一のリージョンに書き込みが集中していることがわかります。

</CustomContent>

<CustomContent platform="tidb-cloud">

以下の [Key Visualizer](/tidb-cloud/tune-performance.md#key-visualizer) のスクリーンショットでは、バイナリ形式でフィールドの順序が入れ替えられている `uuid_demo_2` テーブルの単一のリージョンに書き込みが集中していることがわかります。

</CustomContent>

![Key Visualizer](/media/best-practices/uuid_keyviz.png)

```sql
CREATE TABLE `uuid_demo_1` (
  `uuid` varbinary(16) NOT NULL,
  `c1` varchar(255) NOT NULL,
  PRIMARY KEY (`uuid`) CLUSTERED
)
```

```sql
CREATE TABLE `uuid_demo_2` (
  `uuid` varbinary(16) NOT NULL,
  `c1` varchar(255) NOT NULL,
  PRIMARY KEY (`uuid`) CLUSTERED
)
```

## MySQL 互換性

UUID は MySQL でも使用することができます。`BIN_TO_UUID()` 関数と `UUID_TO_BIN()` 関数は MySQL 8.0 で導入されました。`UUID()` 関数はそれ以前の MySQL バージョンでも利用できます。

