---
title: TiDBでTPC-Cテストを実行する方法
aliases: ['/ja/docs/dev/benchmark/benchmark-tidb-using-tpcc/','/ja/docs/dev/benchmark/how-to-run-tpcc/']
---

# TiDBでTPC-Cテストを実行する方法

この文書では、[TPC-C](http://www.tpc.org/tpcc/)を使用してTiDBをテストする方法について説明します。

TPC-Cはオンライントランザクション処理（OLTP）ベンチマークです。異なるタイプの5つの取引を含む流通販売モデルを使用してOLTPシステムをテストします：

* NewOrder
* Payment
* OrderStatus
* Delivery
* StockLevel

## 準備

テストの前に、TPC-Cベンチマークはデータベースの初期状態を指定します。これはデータベースでのデータ生成の規則です。 `ITEM`テーブルには固定数の10万アイテムが含まれており、倉庫の数を調整できます。 `WAREHOUSE`テーブルにWのレコードがある場合：

* `STOCK`テーブルにはW \* 100,000レコードが含まれます（倉庫ごとに10万アイテムの在庫データが対応）
* `DISTRICT`テーブルにはW \* 10のレコードが含まれます（各倉庫が10の地区にサービスを提供）
* `CUSTOMER`テーブルにはW \* 10 \* 3,000のレコードが含まれます（各地区に3,000人の顧客がいます）
* `HISTORY`テーブルにはW \* 10 \* 3,000のレコードが含まれます（各顧客には1つの取引履歴があります）
* `ORDER`テーブルにはW \* 10 \* 3,000のレコードが含まれます（各地区に3,000の注文があり、最後の900の注文は`NEW-ORDER`テーブルに追加されます。各注文はランダムに5〜15のORDER-LINEレコードを生成します。

この文書では、テストにTiDBを使用して1,000の倉庫を使用する例を示します。

TPC-CはtpmC（1分あたりのトランザクション数）を使用して最大正味スループット（MQTh、最大正資格スループット）を測定します。取引はNewOrder取引で、最後の単位は1分間に処理された新しい注文の数です。

この文書のテストは、[go-tpc](https://github.com/pingcap/go-tpc)に基づいて実装されています。TiUPコマンドを使用してテストプログラムをダウンロードできます。

{{< copyable "shell-regular" >}}

```shell
tiup install bench
```

TiUP Benchコンポーネントの詳細な使用法については、[TiUP Bench](/tiup/tiup-bench.md)を参照してください。

TiDBクラスターをデプロイし、TiDBサーバーが172.16.5.140および172.16.5.141に配置され、両方のサーバーがポート4000でリッスンしていると仮定します。次の手順でTPC-Cテストを実行できます。

## データのロード

**データのロードは通常、TPC-Cテスト全体の時間を要し、問題のある段階です。**このセクションでは、次のコマンドを使用してデータをロードするための以下のコマンドを提供します。

以下のTiUPコマンドをシェルで実行します。

{{< copyable "shell-regular" >}}

```shell
tiup bench tpcc -H 172.16.5.140,172.16.5.141 -P 4000 -D tpcc --warehouses 1000 --threads 20 prepare
```

異なるマシン構成に基づき、このロードプロセスには数時間かかることがあります。クラスターサイズが小さい場合、テストにはより小さな `WAREHOUSE`値を使用できます。

データがロードされたら、`tiup bench tpcc -H 172.16.5.140 -P 4000 -D tpcc --warehouses 4 check`コマンドを実行してデータの正確性を検証できます。

## テストを実行する

次のコマンドを実行してテストを実行します。

{{< copyable "shell-regular" >}}

```shell
tiup bench tpcc -H 172.16.5.140,172.16.5.141 -P 4000 -D tpcc --warehouses 1000 --threads 100 --time 10m run
```

テスト中、テスト結果は継続してコンソールに出力されます。

```text
[Current] NEW_ORDER - Takes(s): 4.6, Count: 5, TPM: 65.5, Sum(ms): 4604, Avg(ms): 920, 90th(ms): 1500, 99th(ms): 1500, 99.9th(ms): 1500
[Current] ORDER_STATUS - Takes(s): 1.4, Count: 1, TPM: 42.2, Sum(ms): 256, Avg(ms): 256, 90th(ms): 256, 99th(ms): 256, 99.9th(ms): 256
[Current] PAYMENT - Takes(s): 6.9, Count: 5, TPM: 43.7, Sum(ms): 2208, Avg(ms): 441, 90th(ms): 512, 99th(ms): 512, 99.9th(ms): 512
[Current] STOCK_LEVEL - Takes(s): 4.4, Count: 1, TPM: 13.8, Sum(ms): 224, Avg(ms): 224, 90th(ms): 256, 99th(ms): 256, 99.9th(ms): 256
...
```

テストが終了すると、テストの要約結果が出力されます。

```text
[Summary] DELIVERY - Takes(s): 455.2, Count: 32, TPM: 4.2, Sum(ms): 44376, Avg(ms): 1386, 90th(ms): 2000, 99th(ms): 4000, 99.9th(ms): 4000
[Summary] DELIVERY_ERR - Takes(s): 455.2, Count: 1, TPM: 0.1, Sum(ms): 953, Avg(ms): 953, 90th(ms): 1000, 99th(ms): 1000, 99.9th(ms): 1000
[Summary] NEW_ORDER - Takes(s): 487.8, Count: 314, TPM: 38.6, Sum(ms): 282377, Avg(ms): 899, 90th(ms): 1500, 99th(ms): 1500, 99.9th(ms): 1500
[Summary] ORDER_STATUS - Takes(s): 484.6, Count: 29, TPM: 3.6, Sum(ms): 8423, Avg(ms): 290, 90th(ms): 512, 99th(ms): 1500, 99.9th(ms): 1500
[Summary] PAYMENT - Takes(s): 490.1, Count: 321, TPM: 39.3, Sum(ms): 144708, Avg(ms): 450, 90th(ms): 512, 99th(ms): 1000, 99.9th(ms): 1500
[Summary] STOCK_LEVEL - Takes(s): 487.6, Count: 41, TPM: 5.0, Sum(ms): 9318, Avg(ms): 227, 90th(ms): 512, 99th(ms): 1000, 99.9th(ms): 1000
```

テストが終了したら、`tiup bench tpcc -H 172.16.5.140 -P 4000 -D tpcc --warehouses 4 check`コマンドを実行してデータの正確性を検証できます。

## テストデータのクリーンアップ

次のコマンドを実行してテストデータをクリアアップします。

{{< copyable "shell-regular" >}}

```shell
tiup bench tpcc -H 172.16.5.140 -P 4000 -D tpcc --warehouses 4 cleanup
```