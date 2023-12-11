---
title: Changefeedの課金
summary: TiDB Cloudにおけるchangefeedの課金について学ぶ。
aliases: ['/tidbcloud/tidb-cloud-billing-tcu']
---

# Changefeedの課金

TiDB Cloudは、TiCDCレプリケーションキャパシティユニット（RCU）で[changefeed](/tidb-cloud/changefeed-overview.md)のキャパシティを測定します。クラスターに[changefeedを作成](/tidb-cloud/changefeed-overview.md#create-a-changefeed)する際に、適切な仕様を選択することができます。RCUが高いほど、レプリケーションパフォーマンスが向上します。これらのTiCDC changefeed RCUに対して料金が発生します。

## TiCDC RCUの数

以下の表には、changefeedの仕様とそれに対応するレプリケーションパフォーマンスが示されています。

| 仕様            | 最大レプリケーションパフォーマンス  |
|---------------|---------------------------------|
| 2 RCU        | 5,000 rows/s                    |
| 4 RCU        | 10,000 rows/s                   |
| 8 RCU        | 20,000 rows/s                   |
| 16 RCU       | 40,000 rows/s                   |
| 24 RCU       | 60,000 rows/s                   |
| 32 RCU       | 80,000 rows/s                   |
| 40 RCU       | 100,000 rows/s                  |

> **Note:**
>
> 上記のパフォーマンスデータは参考用であり、異なるシナリオによって異なる場合があります。

## 価格

各TiCDC RCUごとのサポートされている地域とTiDBCloudの価格については、[Changefeed Cost](https://www.pingcap.com/tidb-cloud-pricing-details/#changefeed-cost)を参照してください。