---
title: TiDB で CH-benCHmark テストを実行する方法
summary: TiDB で CH-benCHmark テストを実行する方法について学びます。
---

# TiDB で CH-benCHmark テストを実行する方法

このドキュメントでは、CH-benCHmark を使用して TiDB をテストする方法について説明します。

CH-benCHmark は、[TPC-C](http://www.tpc.org/tpcc/) と [TPC-H](http://www.tpc.org/tpch/) の両方のテストを含む混合ワークロードです。これは、HTAP システムをテストする際に最も一般的なワークロードです。詳細については、[The mixed workload CH-benCHmark](https://research.tableau.com/sites/default/files/a8-cole.pdf) を参照してください。

CH-benCHmark テストを実行する前に、まず TiDB の HTAP コンポーネントである [TiFlash](/tiflash/tiflash-overview.md) を展開する必要があります。TiFlash を展開し、[TiFlash レプリカを作成](#create-tiflash-replicas)した後、TiKV はTPC-Cの最新データをリアルタイムでTiFlashにレプリケートします。また、TiDB オプティマイザは、TPC-Hのワークロードからの OLAP クエリを効率的に実行するために、TiFlash の MPP エンジンに自動的にプッシュダウンします。

このドキュメントにある CH-benCHmark テストは、[go-tpc](https://github.com/pingcap/go-tpc) をベースに実装されています。次の [TiUP](/tiup/tiup-overview.md) コマンドを使用してテストプログラムをダウンロードできます。

{{< copyable "shell-regular" >}}

```shell
tiup install bench
```

TiUP Bench コンポーネントの詳細な使用法については、[TiUP Bench](/tiup/tiup-bench.md) を参照してください。

## データをロードする

### TPC-C データをロードする

**データのロードは通常、TPC-C テスト全体で最も時間がかかり、問題が発生しやすい段階です。**

1000の倉庫を取る例として、次のようにTiUPコマンドを実行してデータのロードとテストを行うことができます。なお、このドキュメントの中で `172.16.5.140` および `4000` を TiDB のホストおよびポートの値に置き換える必要があります。

```shell
tiup bench tpcc -H 172.16.5.140 -P 4000 -D tpcc --warehouses 1000 prepare -T 32
```

異なるマシン構成によっては、このローディングプロセスには数時間かかる場合があります。クラスタサイズが小さい場合、テストのためにより小さな倉庫値を使用することができます。

データがロードされたら、`tiup bench tpcc -H 172.16.5.140 -P 4000 -D tpcc --warehouses 1000 check` コマンドを実行してデータの正当性を検証できます。

### TPC-H に必要な追加のテーブルとビューをロードする

次のTiUPコマンドをシェルで実行します。

{{< copyable "shell-regular" >}}

```shell
tiup bench ch -H 172.16.5.140 -P 4000 -D tpcc prepare
```

以下は、ログの出力例です。

```
creating nation
creating region
creating supplier
generating nation table
generate nation table done
generating region table
generate region table done
generating suppliers table
generate suppliers table done
creating view revenue1
```

## TiFlash レプリカを作成する

TiFlash を展開した後、TiFlash は自動的に TiKV データをレプリケートしません。次のSQLステートメントを実行して、`tpcc`データベースのために TiFlash レプリカを作成する必要があります。指定された TiFlash レプリカが作成されると、TiKV は最新データをリアルタイムでTiFlashにレプリケートします。次の例では、クラスタに2つの TiFlash ノードが展開されており、レプリカ数は2に設定されています。

```
ALTER DATABASE tpcc SET tiflash replica 2;
```

`WHERE` 句を使用して、検査するデータベースおよびテーブルを指定するため、すべてのデータベースのレプリケーション状況を確認する場合は、以下のステートメントから `WHERE` 句を削除します。

{{< copyable "sql" >}}

```sql
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = 'tpcc';
```

上記のステートメントの結果では、以下のようになります。

- `AVAILABLE` は、特定のテーブルの TiFlash レプリカが利用可能かどうかを示します。`1` は利用可能で、`0` は利用不可を意味します。レプリカが利用可能になると、このステータスはもう変わりません。
- `PROGRESS` は、レプリケーションの進捗状況を表します。値は `0` から `1` の間です。`1` は TiFlash レプリカのレプリケーションが完了したことを意味します。

## 統計情報を収集する

TiDB オプティマイザが最適な実行プランを生成できるようにするために、事前に次のSQLステートメントを実行して統計情報を収集してください。

```
analyze table customer;
analyze table district;
analyze table history;
analyze table item;
analyze table new_order;
analyze table order_line;
analyze table orders;
analyze table stock;
analyze table warehouse;
analyze table nation;
analyze table region;
analyze table supplier;
```

## テストを実行する

50のTP同時性と1つのAP同時性を取る例として、次のようにコマンドを実行してテストを実行します。

{{< copyable "shell-regular" >}}

```shell
tiup bench ch --host 172.16.5.140 -P4000 --warehouses 1000 run -D tpcc -T 50 -t 1 --time 1h
```

テスト中、テスト結果はコンソールに連続して出力されます。例:

```text
[Current] NEW_ORDER - Takes(s): 10.0, Count: 13524, TPM: 81162.0, Sum(ms): 998317.6, Avg(ms): 73.9, 50th(ms): 71.3, 90th(ms): 100.7, 95th(ms): 113.2, 99th(ms): 159.4, 99.9th(ms): 209.7, Max(ms): 243.3
[Current] ORDER_STATUS - Takes(s): 10.0, Count: 1132, TPM: 6792.7, Sum(ms): 16196.6, Avg(ms): 14.3, 50th(ms): 13.1, 90th(ms): 24.1, 95th(ms): 27.3, 99th(ms): 37.7, 99.9th(ms): 50.3, Max(ms): 52.4
[Current] PAYMENT - Takes(s): 10.0, Count: 12977, TPM: 77861.1, Sum(ms): 773982.0, Avg(ms): 59.7, 50th(ms): 56.6, 90th(ms): 88.1, 95th(ms): 100.7, 99th(ms): 151.0, 99.9th(ms): 201.3, Max(ms): 243.3
[Current] STOCK_LEVEL - Takes(s): 10.0, Count: 1134, TPM: 6806.0, Sum(ms): 31220.9, Avg(ms): 27.5, 50th(ms): 25.2, 90th(ms): 37.7, 95th(ms): 44.0, 99th(ms): 71.3, 99.9th(ms): 117.4, Max(ms): 125.8
[Current] Q11    - Count: 1, Sum(ms): 3682.9, Avg(ms): 3683.6
[Current] DELIVERY - Takes(s): 10.0, Count: 1167, TPM: 7002.6, Sum(ms): 170712.9, Avg(ms): 146.3, 50th(ms): 142.6, 90th(ms): 192.9, 95th(ms): 209.7, 99th(ms): 251.7, 99.9th(ms): 335.5, Max(ms): 385.9
[Current] NEW_ORDER - Takes(s): 10.0, Count: 13238, TPM: 79429.5, Sum(ms): 1010795.3, Avg(ms): 76.4, 50th(ms): 75.5, 90th(ms): 104.9, 95th(ms): 117.4, 99th(ms): 159.4, 99.9th(ms): 234.9, Max(ms): 352.3
[Current] ORDER_STATUS - Takes(s): 10.0, Count: 1224, TPM: 7350.6, Sum(ms): 17874.1, Avg(ms): 14.6, 50th(ms): 13.6, 90th(ms): 23.1, 95th(ms): 27.3, 99th(ms): 37.7, 99.9th(ms): 54.5, Max(ms): 60.8
```
```yaml
      - {R}
      - 電力消費量(s): 3599.7, カウント: 501795, TPM: 8363.9, 合計(ms): 63905178.8, 平均(ms): 127.4, 50%点(ms): 125.8, 90%点(ms): 167.8, 95%点(ms): 184.5, 99%点(ms): 226.5, 99.9%点(ms): 318.8, 最大(ms): 604.0
      - 在庫レベル - 電力消費量(s): 3599.8, カウント: 500467, TPM: 8341.5, 合計(ms): 13208726.4, 平均(ms): 26.4, 50%点(ms): 25.2, 90%点(ms): 37.7, 95%点(ms): 44.0, 99%点(ms): 62.9, 99.9%点(ms): 96.5, 最大(ms): 570.4
      - Q12    - カウント: 11, 合計(ms): 105320.3, 平均(ms): 9574.6
      - Q13    - カウント: 11, 合計(ms): 19199.5, 平均(ms): 1745.4
```