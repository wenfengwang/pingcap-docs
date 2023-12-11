---
title: TiDB 専用の制限とクォータ
summary: TiDB Cloud の制限とクォータについて学ぶ。
---

# TiDB 専用の制限とクォータ

TiDB Cloud では、[TiDB 専用](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタ内で作成できる各種コンポーネントの数量や、TiDB の一般的な使用制限を設けています。さらに、必要以上のリソースの作成を防ぐため、組織レベルでのクォータが設定されています。これらの表には、制限とクォータがまとめられています。

> **注意:**
>
> これらの制限やクォータが貴社の運用に問題をもたらす場合は、[TiDB Cloud サポート](/tidb-cloud/tidb-cloud-support.md)にご連絡ください。

## クラスタの制限

| コンポーネント | 制限 |
|:-|:-|
| データレプリカの数 | 3 |
| クロスゾーン展開のための使用可能ゾーン数 | 3 |

> **注意:**
>
> TiDB の一般的な使用制限について詳しくは、[TiDB の制限](https://docs.pingcap.com/tidb/stable/tidb-limitations)を参照してください。

## クラスタのクォータ

| コンポーネント | クォータ (デフォルト) |
|:-|:-|
| 組織全体のすべてのクラスタの TiDB ノードの最大数 | 10 |
| 組織全体のすべてのクラスタの TiKV ノードの最大数 | 15 |
| 組織全体のすべてのクラスタの TiFlash ノードの最大数 | 5 |